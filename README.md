answer for https://stackoverflow.com/questions/68678340/how-to-reply-to-a-mail-using-java-gmail-api-code-so-it-will-be-group-under-the-s

This is what works for me.
I used the following resources.
https://newbedev.com/how-to-send-a-reply-with-gmail-api
https://developers.google.com/gmail/api/guides/sending
https://developers.google.com/gmail/api/guides/threads

The key steps is copieing message-id and macking subject from the original message and using the References (altough it was empty in my case).

```
  public class ReplyMessage{
    

    public static void main(String[] args) {
        ...
        //I need some way to start so let's assume I have the gmail id of some messages.
    
        String gmailId = "gmail id";
        String mailbox = "mailbox@gmail.com";
        Message original = gmail.users().messages().get(mailbox, gmailId)
                .execute();
        List<String> headers = Arrays.asList("Subject", "To", "From", "Message-ID", "References");

        Map<String, String> replyHeaders = original.getPayload().getHeaders()
                .stream()
                .filter((header) -> headers.contains(header.getName())
                ).collect(Collectors.toMap(MessagePartHeader::getName, MessagePartHeader::getValue));


        try {
            Message replayEmail = createReplayEmail(replyHeaders,original.getThreadId());
            gmail.users()
                    .messages()
                    .send("target@gmail.com",replayEmail)
                    .execute();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
    public static Message createReplayEmail(Map<String, String> headers, String threadId)
            throws MessagingException, IOException {
        MimeMessage email = createMimeMessage(headers);

        ByteArrayOutputStream buffer = new ByteArrayOutputStream();
        email.writeTo(buffer);
        byte[] bytes = buffer.toByteArray();
        String encodedEmail = Base64.getEncoder().encodeToString(bytes);
        Message message = new Message();
        message.setThreadId(threadId);
        message.setRaw(encodedEmail);
        return message;
    }

    private static MimeMessage createMimeMessage(Map<String, String> headers) throws MessagingException {
        Properties props = new Properties();
        Session session = Session.getDefaultInstance(props, null);
        MimeMessage email = new MimeMessage(session);

        email.setFrom(new InternetAddress(headers.get("To")));
        email.addRecipient(javax.mail.Message.RecipientType.TO,
                new InternetAddress(headers.get("From")));
        email.setSubject("Re: " + headers.get("Subject"));

        MimeBodyPart mimeBodyPart = new MimeBodyPart();
        mimeBodyPart.setContent("Re Re Re...", "text/plain");

        email.addHeader("In-Reply-To",headers.get("Message-ID"));
        email.addHeader("References",headers.get("References"));

        Multipart multipart = new MimeMultipart();
        multipart.addBodyPart(mimeBodyPart);
        email.setContent(multipart);
        return email;
    }
}
```
  
        
