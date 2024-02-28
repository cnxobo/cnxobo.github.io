---
title: "解决邮箱关闭导致的邮件无法发送问题 550\_5.1.1\_recipient\_is\_not\_exist javax.mail"
tags: []
id: '819'
categories:
  - - Java
date: 2020-12-20 22:21:00
---

业务方反馈系统的定时邮件没有收到。排查日志后发现，是人员离职后，其工作邮箱关闭，与其相关的业务系统的定时邮件通知的收件人列表内没有移除其邮箱，会导致邮件通知发送失败报"550 5.1.1 recipient is not exist"异常，导致整个邮件通知都没办法发出。 经过查阅文档发现可以设置允许发送部分邮件。可以通过设置属性`mail.smtp.sendpartial`或者通过`SMTPMessage`的`sendPartial`属性来实现。

### 方案一 mail.smtp.sendpartial

设置构造属性`mail.smtp.sendpartial`

```Java
Properties prop = new Properties();
prop.put("mail.smtp.sendpartial", "true");
// ... prop.put(xxx, xxx);

Session session = Session.getInstance(prop, new Authenticator() {
    @Override
    protected PasswordAuthentication getPasswordAuthentication() {
        return new PasswordAuthentication(username, password);
    }
});
```

### 方案二 SMTPMessage

`SMTPMessage`有sendPartial属性。把默认的`mimeMessage`使用`SMTPMessage`包装一下，然后设置 `sendPartial` 属性为true。

```Java
Message message = new MimeMessage(session);
// ... message.set(xxx,xxx);
SMTPMessage smtpMessage = new SMTPMessage(message);
smtpMessage.setSendPartial(true); 
```

即使开启了`sendPartial`属性，如果遇到有无效收件人时，依旧会抛出一个`SMTPSendFailedException`异常。需要处理一下这个异常,根据STMP的定义返回码在2xx的时候都可以认为是成功，所以返回码不为2xx时，把异常再次抛出, 同时在异常内可以通过`e.getInvalidAddresses()`获取无效的邮件地址,做进一步处理。。

```Java
 // Send message
    try {
      Transport.send(smtpMessage);
      System.out.println("Sent message successfully....");
    } catch (SMTPSendFailedException e) {
      if (e.getReturnCode() >= 200 && e.getReturnCode() < 300) {
        Address[] addresses = e.getInvalidAddresses();
        for (Address address : addresses) {
          doSth(address);
        }
      } else {
        throw e;
      }
    } catch (MessagingException mex) {
      mex.printStackTrace();
    }
```