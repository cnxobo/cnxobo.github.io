---
title: Java7 HTTPS 支持  TLS 1.1 TLS 1.2
tags:
  - apache http client
  - Java7
  - letencrypt
  - tls
id: '948'
categories:
  - - Java
comments: false
date: 2022-01-23 17:27:39
---

在Java SE 7 中，SunJSSE 支持TLS 1.1 and TLS 1.2。但是考虑到当时(since 2011)很多服务器对这两个协议支持的不好，就把这两个版本的协议禁用，默认使用TLS 1.0。 如果要启用这两个协议的支持也很简单:

## 设置系统属性

### https.protocols

指定Java HTTPS连接(`HttpsURLConnection` 或 `URL.openStream()`)使用的协议版本。

```
-Dhttps.protocols=TLSv1,TLSv1.1,TLSv1.2
```

或

```Java
System.setProperty("https.protocols", "TLSv1,TLSv1.1,TLSv1.2");
```

验证代码

```Java
@Test
public static void testOpenConnection() throws Exception {
    System.setProperty("https.protocols", "TLSv1,TLSv1.1,TLSv1.2");
    URL url = new URL("https://xobo.org");

    HttpsURLConnection connection = (HttpsURLConnection) url.openConnection();
    connection.setConnectTimeout(2000);
    connection.setReadTimeout(2000);
    connection.connect();
    connection.getResponseCode();
    assertEquals("connection failed", 200, connection.getResponseCode());
}
  ```
###  jdk.tls.client.protocols   
指定TLS的默认版本(since Java8,Java7u95,Java6u121)。
```

\-Djdk.tls.client.protocols=TLSv1,TLSv1.1,TLSv1.2

````
或
```Java
System.setProperty("jdk.tls.client.protocols", "TLSv1,TLSv1.1,TLSv1.2");
````

> TLSv1 是不安全的协议，不建议在生产环境中使用。

### TLS 与 HTTPS的关系

HTTPS是传输协议，TLS只是它的加密方式，比如还可以用更不安全的SSL2，SSL3。 TLS也不仅仅使用在HTTPS里，使用JDBC连接数据库时也可以启用TLS加密。

## Apache HttpClient

使用Apache HttpClient 时会发现这个配置并不生效，这是因为它默认不读取Java系统属性，需要显示的指定`useSystemProperties()`。

### 使用系统属性

```Java
System.setProperty("https.protocols", "TLSv1,TLSv1.1,TLSv1.2");
CloseableHttpClient client = HttpClientBuilder.create()
    .useSystemProperties().build();
```

验证代码

```Java
@Test
public static void testSystemProperties() throws Exception {

    System.setProperty("https.protocols", "TLSv1,TLSv1.1,TLSv1.2");

    CloseableHttpClient client = HttpClientBuilder.create().useSystemProperties().build();
    HttpGet request = new HttpGet("https://xobo.org");
    CloseableHttpResponse response = client.execute(request);

    assertEquals("connection failed", 200, response.getStatusLine().getStatusCode());
}

```

### 如何不影响全局

修改系统属性，整个JVM都会受到影响。如果希望把影响范围降到最低，可以通过定制`HttpClients`的方式。

```Java
SSLContext sslContext = SSLContexts.createDefault();
SSLConnectionSocketFactory sslsf = new SSLConnectionSocketFactory(sslContext,
    new String[] {"TLSv1",  "TLSv1.1", "TLSv1.2"}, null, new DefaultHostnameVerifier());

CloseableHttpClient httpclient = HttpClients.custom()
    .setSSLSocketFactory(sslsf)
    .build();
```

验证代码

```Java
@Test
public static void testCustomeHttpClient() throws Exception {
    SSLContext sslContext = SSLContexts.createDefault();
    SSLConnectionSocketFactory sslsf = new SSLConnectionSocketFactory(sslContext,
        new String[] {"TLSv1", "TLSv1.1", "TLSv1.2"}, null, new DefaultHostnameVerifier());

    CloseableHttpClient client = HttpClients.custom()
        .setSSLSocketFactory(sslsf)
        .build();

    HttpGet request = new HttpGet("https://xobo.org");

    CloseableHttpResponse response = client.execute(request);
    assertEquals("connection failed", 200, response.getStatusLine().getStatusCode());
}
```

## unable to find valid certification path to requested target

如果拿我的域名做测试，还会报这个异常。因为我的HTTPS证书是申请LetsEncrypt的。JDK7发布时，LetsEncrypt还不存在，所以JDK7的证书里不识别它，需要手动加载一下。

```Shell
wget https://letsencrypt.org/certs/lets-encrypt-r3.pem
$JAVA_HOME/bin/keytool -trustcacerts -keystore "$JAVA_HOME/jre/lib/security/cacerts" -storepass changeit -noprompt -importcert -alias lets-encrypt-r3 -file  lets-encrypt-r3.pem
```

> \>= 7u151, 8u141 就不需要手动加载证书了。 $JAVA\_HOME 需要指向JDK7或其它低版本JDK。

参考链接: [Java Cryptography Architecture Oracle Providers Documentation for Java Platform Standard Edition 7](https://docs.oracle.com/javase/7/docs/technotes/guides/security/SunProviders.html#tlsprotonote)