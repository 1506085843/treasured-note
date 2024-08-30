


# 前言
之前写过一个邮件发送的文章[java发送邮件](https://blog.csdn.net/qq_33697094/article/details/115705236)
不过因为它的实现功能比较简单，不支持附件和抄送，所以本文使用 jakarta.mail 来实现邮件发送功能并支持附件和抄送。

# 一、maven依赖
```java
        <dependency>
            <groupId>com.sun.mail</groupId>
            <artifactId>jakarta.mail</artifactId>
            <version>2.0.1</version>
        </dependency>
        <dependency>
            <groupId>com.microsoft.ews-java-api</groupId>
            <artifactId>ews-java-api</artifactId>
            <version>2.0</version>
        </dependency>
```

# 二、工具类

EmailConfig类

```java
package com.hai.tang.model;

public class EmailConfig {
    //邮箱的类型：0-SMTP 1-IMAP 2-Exchange
    private Integer mailPro;
    //是否开启身份验证
    private Integer auth;
    //TLS加密传输
    private Integer tls;
    //SSL加密传输
    private Integer ssl;
    //发信名称
    private String alias;
    //发件人邮箱
    private String servicedMailAddress;
    //发件人密码
    private String password;
    //发件服务器地址
    private String sendMailHost;
    //发件服务器端口
    private String sendMailPort;

    public Integer getMailPro() {
        return mailPro;
    }

    public void setMailPro(Integer mailPro) {
        this.mailPro = mailPro;
    }

    public Integer getAuth() {
        return auth;
    }

    public void setAuth(Integer auth) {
        this.auth = auth;
    }

    public Integer getTls() {
        return tls;
    }

    public void setTls(Integer tls) {
        this.tls = tls;
    }

    public Integer getSsl() {
        return ssl;
    }

    public void setSsl(Integer ssl) {
        this.ssl = ssl;
    }

    public String getAlias() {
        return alias;
    }

    public void setAlias(String alias) {
        this.alias = alias;
    }


    public String getServicedMailAddress() {
        return servicedMailAddress;
    }

    public void setServicedMailAddress(String servicedMailAddress) {
        this.servicedMailAddress = servicedMailAddress;
    }

    public String getPassword() {
        return password;
    }

    public void setPassword(String password) {
        this.password = password;
    }

    public String getSendMailHost() {
        return sendMailHost;
    }

    public void setSendMailHost(String sendMailHost) {
        this.sendMailHost = sendMailHost;
    }

    public String getSendMailPort() {
        return sendMailPort;
    }

    public void setSendMailPort(String sendMailPort) {
        this.sendMailPort = sendMailPort;
    }
}

```

ExchangeClient类
```java
package com.hai.tang.util;

import microsoft.exchange.webservices.data.core.ExchangeService;
import microsoft.exchange.webservices.data.core.enumeration.misc.ExchangeVersion;
import microsoft.exchange.webservices.data.core.exception.service.local.ServiceLocalException;
import microsoft.exchange.webservices.data.core.service.item.EmailMessage;
import microsoft.exchange.webservices.data.credential.ExchangeCredentials;
import microsoft.exchange.webservices.data.credential.WebCredentials;
import microsoft.exchange.webservices.data.property.complex.MessageBody;

import java.net.URI;
import java.net.URISyntaxException;
import java.util.ArrayList;
import java.util.Arrays;
import java.util.Iterator;
import java.util.List;

public class ExchangeClient {

    private final String hostname;
    private final ExchangeVersion exchangeVersion;
    private final String domain;
    private final String username;
    private final String password;
    private final String subject;
    private final String recipientTo;
    private final List<String> recipientTos;
    private final List<String> recipientCc;
    private final List<String> recipientBcc;
    private final List<String> attachments;
    private final String message;

    private ExchangeClient(ExchangeClientBuilder builder) {
        this.hostname = builder.hostname;
        this.exchangeVersion = builder.exchangeVersion;
        this.domain = builder.domain;
        this.username = builder.username;
        this.password = builder.password;
        this.subject = builder.subject;
        this.recipientTo = builder.recipientTo;
        this.recipientTos = builder.recipientTos;
        this.recipientCc = builder.recipientCc;
        this.recipientBcc = builder.recipientBcc;
        this.attachments = builder.attachments;
        this.message = builder.message;
    }

    public static class ExchangeClientBuilder {

        private String hostname;
        private ExchangeVersion exchangeVersion;
        private String domain;
        private String username;
        private String password;
        private String subject;
        private String recipientTo;
        private List<String> recipientTos;
        private List<String> recipientCc;
        private List<String> recipientBcc;
        private List<String> attachments;
        private String message;

        public ExchangeClientBuilder() {
            this.exchangeVersion = ExchangeVersion.Exchange2010_SP1;
            this.hostname = "";
            this.username = "";
            this.password = "";
            this.subject = "";
            this.recipientTo = "";
            this.recipientTos = new ArrayList<>(0);
            this.recipientCc = new ArrayList<>(0);
            this.recipientBcc = new ArrayList<>(0);
            this.attachments = new ArrayList<>(0);
            this.message = "";
        }

        /**
         * The hostname of the Exchange Web Service. It will be used for
         * connecting with URI https://hostname/ews/exchange.asmx
         *
         * @param hostname the hostname of the MS Exchange Smtp Server.
         * @return the builder for chain usage.
         */
        public ExchangeClientBuilder hostname(String hostname) {
            this.hostname = hostname;
            return this;
        }

        /**
         * The Exchange Web Server version.
         *
         * @param exchangeVersion the Exchange Web Server version.
         * @return the builder for chain usage.
         */
        public ExchangeClientBuilder exchangeVersion(ExchangeVersion exchangeVersion) {
            this.exchangeVersion = exchangeVersion;
            return this;
        }

        /**
         * The domain of the MS Exchange Smtp Server.
         *
         * @param domain the domain of the Active Directory. The first part of
         * the username. For example: MYDOMAIN\\username, set the MYDOMAIN.
         * @return the builder for chain usage.
         */
        public ExchangeClientBuilder domain(String domain) {
            this.domain = domain;
            return this;
        }

        /**
         * The username of the MS Exchange Smtp Server. The second part of the
         * username. For example: MYDOMAIN\\username, set the username.
         *
         * @param username the username of the MS Exchange Smtp Server.
         * @return the builder for chain usage.
         */
        public ExchangeClientBuilder username(String username) {
            this.username = username;
            return this;
        }

        /**
         * The password of the MS Exchange Smtp Server.
         *
         * @param password the password of the MS Exchange Smtp Server.
         * @return the builder for chain usage.
         */
        public ExchangeClientBuilder password(String password) {
            this.password = password;
            return this;
        }

        /**
         * The subject for this send.
         *
         * @param subject the subject for this send.
         * @return the builder for chain usage.
         */
        public ExchangeClientBuilder subject(String subject) {
            this.subject = subject;
            return this;
        }

        /**
         * The recipient for this send.
         *
         * @param recipientTo the recipient for this send.
         * @return the builder for chain usage.
         */
        public ExchangeClientBuilder recipientTo(String recipientTo) {
            this.recipientTo = recipientTo;
            return this;
        }

        /**
         * The recipients for this send.
         *
         * @param recipientTos the recipients for this send.
         * @return the builder for chain usage.
         */
        public ExchangeClientBuilder recipientTos(List<String> recipientTos) {
            this.recipientTos = recipientTos;
            return this;
        }

        /**
         * You can specify one or more email address that will be used as cc
         * recipients.
         *
         * @param recipientCc the first cc email address.
         * @param recipientsCc the other cc email address for this send.
         * @return the builder for chain usage.
         */
        public ExchangeClientBuilder recipientCc(String recipientCc, String... recipientsCc) {
            // Prepare the list.
            List<String> recipients = new ArrayList<>(1 + recipientsCc.length);
            recipients.add(recipientCc);
            recipients.addAll(Arrays.asList(recipientsCc));
            // Set the list.
            this.recipientCc = recipients;
            return this;
        }

        /**
         * You can specify a list with email addresses that will be used as cc
         * for this email send.
         *
         * @param recipientCc the list with email addresses that will be used as
         * cc for this email send.
         * @return the builder for chain usage.
         */
        public ExchangeClientBuilder recipientCc(List<String> recipientCc) {
            this.recipientCc = recipientCc;
            return this;
        }

        /**
         * You can specify one or more email address that will be used as bcc
         * recipients.
         *
         * @param recipientBcc the first bcc email address.
         * @param recipientsBcc the other bcc email address for this send.
         * @return the builder for chain usage.
         */
        public ExchangeClientBuilder recipientBcc(String recipientBcc, String... recipientsBcc) {
            // Prepare the list.
            List<String> recipients = new ArrayList<>(1 + recipientsBcc.length);
            recipients.add(recipientBcc);
            recipients.addAll(Arrays.asList(recipientsBcc));
            // Set the list.
            this.recipientBcc = recipients;
            return this;
        }

        /**
         * You can specify a list with email addresses that will be used as bcc
         * for this email send.
         *
         * @param recipientBcc the list with email addresses that will be used
         * as bcc for this email send.
         * @return the builder for chain usage.
         */
        public ExchangeClientBuilder recipientBcc(List<String> recipientBcc) {
            this.recipientBcc = recipientBcc;
            return this;
        }

        /**
         * You can specify one or more email address that will be used as cc
         * recipients.
         *
         * @param attachment the first attachment.
         * @param attachments the other attachments for this send.
         * @return the builder for chain usage.
         */
        public ExchangeClientBuilder attachments(String attachment, String... attachments) {
            // Prepare the list.
            List<String> attachmentsToUse = new ArrayList<>(1 + attachments.length);
            attachmentsToUse.add(attachment);
            attachmentsToUse.addAll(Arrays.asList(attachments));
            // Set the list.
            this.attachments = attachmentsToUse;
            return this;
        }

        /**
         * You can specify a list with email attachments that will be used for
         * this email send.
         *
         * @param attachments the list with email attachments that will be used
         * for this email send.
         * @return the builder for chain usage.
         */
        public ExchangeClientBuilder attachments(List<String> attachments) {
            this.attachments = attachments;
            return this;
        }

        /**
         * The body of the email message.
         *
         * @param message the body of the email message.
         * @return the builder for chain usage.
         */
        public ExchangeClientBuilder message(String message) {
            this.message = message;
            return this;
        }

        /**
         * Build a mail.
         *
         * @return an EmailApacheUtils object.
         */
        public ExchangeClient build() {
            return new ExchangeClient(this);
        }
    }

    public void sendExchange(ExchangeService exchangeService) throws Exception{
        // The Exchange Server Version.
//        ExchangeService exchangeService = new ExchangeService(exchangeVersion);

        // Credentials to sign in the MS Exchange Server.
        ExchangeCredentials exchangeCredentials = new WebCredentials(username, password, domain);
        exchangeService.setCredentials(exchangeCredentials);

        // URL of exchange web service for the mailbox.
        try {
            exchangeService.setUrl(new URI("https://" + hostname + "/ews/Exchange.asmx"));
        } catch (URISyntaxException ex) {
            System.out.println("An exception occured while creating the uri for exchange service."+ex);
            throw ex;
        }

        // The email.
        EmailMessage emailMessage;
        try {
            emailMessage = new EmailMessage(exchangeService);
            emailMessage.setSubject(subject);
            emailMessage.setBody(MessageBody.getMessageBodyFromText(message));
        } catch (Exception ex) {
            System.out.println("An exception occured while setting the email message."+ex);
            throw ex;
        }

        // TO recipient
        try {
            if(null!=recipientTo&& recipientTo.trim().length()>0){
                emailMessage.getToRecipients().add(recipientTo);
            }
        } catch (ServiceLocalException ex) {
            System.out.println("An exception occured while sstting the TO recipient(" + recipientTo + ")."+ex);
            throw ex;
        }

        // TO recipients
        try {
            if(!recipientTos.isEmpty()){
                Iterator<String> iterator = recipientTos.iterator();
                emailMessage.getToRecipients().addSmtpAddressRange(iterator);
            }
        } catch (ServiceLocalException ex) {
            System.out.println("An exception occured while sstting the TO recipients(" + recipientTos + ")."+ex);
            throw ex;
        }

        // CC recipient.
        for (String recipient : recipientCc) {
            try {
                emailMessage.getCcRecipients().add(recipient);
            } catch (ServiceLocalException ex) {
                System.out.println("An exception occured while sstting the CC recipient(" + recipient + ")."+ex);
                throw ex;
            }
        }

        // BCC recipient
        for (String recipient : recipientBcc) {
            try {
                emailMessage.getBccRecipients().add(recipient);
            } catch (ServiceLocalException ex) {
                System.out.println("An exception occured while sstting the BCC recipient(" + recipient + ")."+ex);
                throw ex;
            }
        }

        // Attachements.
        for (String attachmentPath : attachments) {
            try {
                emailMessage.getAttachments().addFileAttachment(attachmentPath);
            } catch (ServiceLocalException ex) {
                System.out.println("An exception occured while setting the attachment."+ex);
                throw ex;
            }
        }

        try {
            emailMessage.send();
            System.out.println("An email is send.");
        } catch (Exception ex) {
            System.out.println("An exception occured while sending an email."+ex);
            throw ex;
        }

    }
}


```

EmailUtil类
```java
package com.hai.tang.util;

import java.io.File;
import java.io.UnsupportedEncodingException;
import java.security.GeneralSecurityException;
import java.util.ArrayList;
import java.util.Arrays;
import java.util.Date;
import java.util.List;
import java.util.Properties;

import com.hai.tang.model.EmailConfig;
import jakarta.activation.DataHandler;
import jakarta.activation.FileDataSource;
import jakarta.mail.Message;
import jakarta.mail.Address;
import jakarta.mail.Authenticator;
import jakarta.mail.MessagingException;
import jakarta.mail.Multipart;
import jakarta.mail.PasswordAuthentication;
import jakarta.mail.Session;
import jakarta.mail.Transport;
import jakarta.mail.internet.AddressException;
import jakarta.mail.internet.InternetAddress;
import jakarta.mail.internet.MimeBodyPart;
import jakarta.mail.internet.MimeMessage;
import jakarta.mail.internet.MimeMultipart;
import jakarta.mail.internet.MimeUtility;
import com.sun.mail.util.MailSSLSocketFactory;
import microsoft.exchange.webservices.data.core.ExchangeService;
import microsoft.exchange.webservices.data.core.enumeration.misc.ExchangeVersion;

public class EmailUtil {

    // 邮件发送的路径
    public final static String MAIL_URL = "";
    // 附件支持的类型
    public final static String MAIL_TYPE = "xls,xlsx,pdf,zip,txt,rar,ppt,pptx,doc,docx";
    // 最大长度10M
    public static final int MAX_BYTE_NUM = 10 * 1024 * 1024;

    /**
     * mailConfigRsp 发件服务器及发件人配置
     * recipientEmails 收件人邮箱
     * cc 抄送邮箱
     * subject  邮件主题
     * content 邮件内容
     * attachPath 要发送的附件路径
     **/
    public static void sendMail(EmailConfig mailConfigRsp, String[] recipientEmails, String[] cc, String subject, String content, String[] attachPath) {
        if (null != content) {
            content = content.replaceAll("\n", "<br/>");
        }
        //Exchange
        if ("2".equals(mailConfigRsp.getMailPro().toString())) {
            try {
                List<String> attachments = attachPath == null ? new ArrayList<>() : Arrays.asList(attachPath);
                ExchangeService service = new ExchangeService(ExchangeVersion.Exchange2010_SP2);
                ExchangeClient client = new ExchangeClient.ExchangeClientBuilder().hostname(mailConfigRsp.getSendMailHost())
                        .exchangeVersion(ExchangeVersion.Exchange2010_SP2).username(mailConfigRsp.getServicedMailAddress())
                        .password(mailConfigRsp.getPassword()).recipientTos(Arrays.asList(recipientEmails))
                        .subject(subject).message(content)
                        .attachments(attachments)
                        .build();
                //发送邮件
                client.sendExchange(service);
            } catch (Exception e) {
                e.printStackTrace();
            }
        } else {
            try {
                Message mailMessage = createMessage(createSession(mailConfigRsp), mailConfigRsp.getServicedMailAddress(), mailConfigRsp.getAlias(), recipientEmails, cc, subject, content, attachPath);
                if (mailMessage.getSize() > MAX_BYTE_NUM) {
                    int max = MAX_BYTE_NUM / 1024 / 1024;
                    System.out.println("内容+附件最大只能 " + max + " M.....");
                    return;
                }
                // 发送邮件
                Transport.send(mailMessage);
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
        System.out.println("邮件发送成功，内容如下： 发件人" + mailConfigRsp.getServicedMailAddress() + "，接收者" + Arrays.toString(recipientEmails) + ",主题" + subject + ",内容" + content + ",附件" + Arrays.toString(attachPath));
    }


    public static Message createMessage(Session session, String account, String alias, String[] notifyTo, String[] cc, String subject,
                                        String content, String[] attachFileNames) {
        if (attachFileNames == null || attachFileNames.length < 1) {
            return createTextMessage(session, account, alias, notifyTo, cc, subject, content);
        } else {
            return createHtmlMessage(session, account, alias, notifyTo, cc, subject, content, attachFileNames);
        }
    }

    public static Session createSession(EmailConfig mailConfigRsp) {
        Properties properties = new Properties();
        properties.put("mail.smtp.host", mailConfigRsp.getSendMailHost());
        properties.put("mail.smtp.port", mailConfigRsp.getSendMailPort());

        if ("0".equals(mailConfigRsp.getAuth().toString())) {
            properties.put("mail.user", mailConfigRsp.getServicedMailAddress());
            properties.put("mail.password", mailConfigRsp.getPassword());
            properties.put("mail.smtp.auth", "true");
        } else {
            properties.put("mail.smtp.auth", "false");
        }

        if ("1".equals(mailConfigRsp.getSsl().toString())) {
            MailSSLSocketFactory sslSF = null;
            try {
                sslSF = new MailSSLSocketFactory();
            } catch (GeneralSecurityException e) {
                e.printStackTrace();
            }
            if (sslSF != null) {
                sslSF.setTrustAllHosts(true);
            }
            properties.put("mail.smtp.ssl.enable", "true");
            properties.put("mail.smtp.ssl.socketFactory", sslSF);
            properties.put("mail.transport.protocol", "smtps");
            if ("0".equals(mailConfigRsp.getAuth().toString())) {
                properties.put("mail.smtps.auth", "true");
            } else {
                properties.put("mail.smtps.auth", "false");
            }
        } else {
            properties.put("mail.transport.protocol", "smtp");
        }
        if ("1".equals(mailConfigRsp.getTls().toString())) {
            properties.put("mail.smtp.starttls.enable", "true");
        } else {
            properties.put("mail.smtp.starttls.enable", "false");
        }
        Session session;
        if ("0".equals(mailConfigRsp.getAuth().toString())) {
            session = Session.getInstance(properties,
                    new mailAuthenricator(mailConfigRsp.getServicedMailAddress(), mailConfigRsp.getPassword()));
        } else {
            session = Session.getInstance(properties);
        }
        session.setDebug(true);
        return session;
    }

    public static Message createHtmlMessage(Session session, String account, String alias, String[] notifyTo, String[] cc, String subject,
                                            String content, String[] attachFileNames) {
        Message mimeMessage = createNotConnetMessage(session, account, alias, notifyTo, cc, subject);
        try {
            // 容器类，可以包含多个MimeBodyPart对象
            Multipart mp = new MimeMultipart("mixed");

            // MimeBodyPart可以包装文本，图片，附件
            MimeBodyPart body = new MimeBodyPart();
            // HTML正文
            body.setContent(content, "text/html; charset=UTF-8");
            mp.addBodyPart(body);
            // 添加图片&附件
            attachBodyPart(attachFileNames, mp);
            // 设置邮件内容
            mimeMessage.setContent(mp);
            // 仅仅发送文本
            // mimeMessage.setText(content);
            mimeMessage.saveChanges();
        } catch (MessagingException e) {
            e.printStackTrace();
        }
        return mimeMessage;
    }

    public static Message createNotConnetMessage(Session session, String account, String alias, String[] notifyTo, String[] cc, String subject) {
        Message mimeMessage = new MimeMessage(session);
        // 发件人
        try {
            if (null != alias && alias.length() > 0) {
                alias = alias.trim();
            }
            mimeMessage.setFrom(new InternetAddress(account, alias));
            // 收件人
            mimeMessage.addRecipients(Message.RecipientType.TO, InternetAddress(notifyTo));
            // 抄送人
            List<String> ccStr = new ArrayList<>();
            if (null != cc) {
                for (String copy : cc) {
                    if (null != copy && copy.length() > 0) {
                        ccStr.add(copy);
                    }
                }
                String[] copyTo = ccStr.toArray(new String[0]);
                mimeMessage.setRecipients(Message.RecipientType.CC, InternetAddress(copyTo));
            }
            // 主题
            mimeMessage.setSubject(subject);
            // 时间
            mimeMessage.setSentDate(new Date());
        } catch (MessagingException | UnsupportedEncodingException e) {
            e.printStackTrace();
        }
        return mimeMessage;
    }

    public static void attachBodyPart(String[] attachFileNames, Multipart mp) {
        int len = 0;
        if (attachFileNames == null || (len = attachFileNames.length) < 1) {
            return;
        }
        String attachFileName;
        String path;
        int byteNum = 0;
        MimeBodyPart bodyPart;
        for (int i = 0; i < len; i++) {
            attachFileName = attachFileNames[i];
            if (null == attachFileName || attachFileName.length() == 0) {
                continue;
            }
            // 判断附件是否是支持的文件类型
            String[] fileNameArr = attachFileName.split("\\.");
            if (!MAIL_TYPE.contains(fileNameArr[fileNameArr.length - 1])) {
                System.out.println("该文件不支持发送 ，文件名 ：   " + attachFileName);
                continue;
            }
            // 设置附件的完整路径
            path = MAIL_URL + attachFileName;
            // 该文件的大小
            File file = new File(path);
            if (!file.exists() || !file.isFile()) {
                System.out.println("该文件不存在,附件中不发送：   attachFileName =" + attachFileName);
                continue;
            }
            byteNum += file.length();// 每个文件大小开始计算
            FileDataSource fds = new FileDataSource(path);// 得到数据源
            bodyPart = new MimeBodyPart();
            try {
                bodyPart.setContent("", "text/html;charset=UTF-8");
                bodyPart.setDataHandler(new DataHandler(fds));
                bodyPart.setFileName(MimeUtility.encodeText(fds.getName())); // 得到文件名并编码（防止中文文件名乱码）同样放入BodyPart
                mp.addBodyPart(bodyPart);
            } catch (MessagingException | UnsupportedEncodingException e) {
                e.printStackTrace();
            }
        }
    }

    static class mailAuthenricator extends Authenticator {
        String userName = null;
        String passWord = null;

        public mailAuthenricator(String userName, String passWord) {
            super();
            this.userName = userName;
            this.passWord = passWord;
        }

        @Override
        protected PasswordAuthentication getPasswordAuthentication() {
            return new PasswordAuthentication(userName, passWord);
        }
    }

    public static Message createTextMessage(Session session, String account, String alias, String[] notifyTo, String[] cc, String subject,
                                            String content) {
        Message mimeMessage = createNotConnetMessage(session, account, alias, notifyTo, cc, subject);
        try {
            Multipart mp = new MimeMultipart();
            MimeBodyPart bodyPart = new MimeBodyPart();
            bodyPart.setContent(content, "text/html; charset=UTF-8");
            mp.addBodyPart(bodyPart);
            mimeMessage.setContent(mp);
        } catch (MessagingException e) {
            e.printStackTrace();
        }
        return mimeMessage;
    }

    public static Address[] InternetAddress(String[] notifyTos) {
        int length = 0;
        if (notifyTos == null || (length = notifyTos.length) < 1) {
            return null;
        }
        Address[] address = new Address[length];
        for (int i = 0; i < length; i++) {
            try {
                address[i] = new InternetAddress(notifyTos[i]);
            } catch (AddressException e) {
                e.printStackTrace();
            }
        }
        return address;
    }
}

```

# 三、测试
```java
        //邮箱服务器地址和发送方配置
        EmailConfig emailConfig = new EmailConfig();
        emailConfig.setMailPro(0);
        emailConfig.setAuth(0);
        emailConfig.setTls(0);
        emailConfig.setSsl(0);
        emailConfig.setAlias("张三");
        emailConfig.setSendMailHost("smtp.qq.com");
        emailConfig.setSendMailPort("25");
        emailConfig.setServicedMailAddress("11111111@qq.com");
        emailConfig.setPassword("ppgfjgdfgfhg5465");

        //收件方（多个收件方分号分割）
        String recipientEmailsStr = "111@163.com;222@163.com";
        String[] recipientEmails = recipientEmailsStr.split(";");

        //要抄送的邮箱（多个抄送方分号分割）
        String ccEmailsStr = "333@163.com;444@163.com";
        String[] ccEmails = ccEmailsStr.split(";");

        //邮件主题
        String subject = "邮件主题";
        //邮件内容
        String content = "第一行件内容\n第二行件内容\n";
        //要发送的附件文件路径
        String[] attachPath = new String[] {"/root/local/file/a.txt", "/root/local/file/b.zip"};

		//发送邮件
        EmailUtil.sendMail(emailConfig, recipientEmails, ccEmails,subject, content, attachPath);
```