[TOC]


# 前言
Base64 编码会将字符串编码得到一个含有 A-Za-z0-9+/ 的字符串。
标准的 Base64 并不适合直接放在URL里传输，因为URL编码器会把标准 Base64 中的“/”和“+”字符变为形如 “%XX” 的形式，而这些 “%” 号在存入数据库时还需要再进行转换，因为 ANSI SQL 中已将“%”号用作通配符。


# 一、base64加密与解密

## 1.标准的 base64 有填充的编码与解码
在 Base64 编码中，输出编码字符串的长度必须是 3 的倍数。如果不是 3 的倍数编码器会根据需要在编码结尾添加一个或两个填充字符 "=" 以满足此要求。
在解码时，解码器会丢弃结尾的那些额外的填充字符进行解码。
一般都使用此方法进行base64加密与解密。
```java
        //Base64编码
        String str = "hello!你好,小明！哈/哈哈，你去哪？sgdr56+=/*&yyy测试，base64测试加密解密hgdjuytiytiuyuytiuyirytr";
        String encodedString = Base64.getEncoder().encodeToString(str.getBytes());
        //输出：aGVsbG8h5L2g5aW9LOWwj+aYju+8geWTiC/lk4jlk4jvvIzkvaDljrvlk6rvvJ9zZ2RyNTYrPS8qJnl5eea1i+ivle+8jGJhc2U2NOa1i+ivleWKoOWvhuino+WvhmhnZGp1eXRpeXRpdXl1eXRpdXlpcnl0cg==
        System.out.println(encodedString); 

        //Base64解码
        byte[] decodedBytes = Base64.getDecoder().decode(encodedString);
        String decodedString = new String(decodedBytes);
         //输出：hello!你好,小明！哈/哈哈，你去哪？sgdr56+=/*&yyy测试，base64测试加密解密hgdjuytiytiuyuytiuyirytr
        System.out.println(decodedString);
```

你会发现上面的标准的 base64 编码后的字符串除了大小写字母和数字还会有 +、/ 这两个符号，这不太适合用于 url 传输或作为 url 中的参数，就像前言中说的那样，如果要用于url，可以采用getUrlEncoder和getUrlDecoder 的 base64 编码：

```java
        //Base64编码
        String str = "hello!你好,小明！哈/哈哈，你去哪？sgdr56+=/*&yyy测试，base64测试加密解密hgdjuytiytiuyuytiuyirytr";
        String encodedString = Base64.getUrlEncoder().encodeToString(str.getBytes());
         //输出：aGVsbG8h5L2g5aW9LOWwj-aYju-8geWTiC_lk4jlk4jvvIzkvaDljrvlk6rvvJ9zZ2RyNTYrPS8qJnl5eea1i-ivle-8jGJhc2U2NOa1i-ivleWKoOWvhuino-WvhmhnZGp1eXRpeXRpdXl1eXRpdXlpcnl0cg==
        System.out.println(encodedString);

        //Base64解码
        byte[] decodedBytes = Base64.getUrlDecoder().decode(encodedString);
        String decodedString = new String(decodedBytes);
        //输出：hello!你好,小明！哈/哈哈，你去哪？sgdr56+=/*&yyy测试，base64测试加密解密hgdjuytiytiuyuytiuyirytr
        System.out.println(decodedString); 
```

## 2. base64无填充的编码与解码
无填充的编码编码后的字符串结尾不会添加 "=" 字符。
```java
		//Base64编码
		String str = "hello!你好,小明！";
        String encodedString = Base64.getEncoder().withoutPadding().encodeToString(str.getBytes());
        System.out.println(encodedString); //aGVsbG8h5L2g5aW9LOWwj+aYju+8gQ

        //Base64解码
        byte[] decodedBytes = Base64.getDecoder().decode(encodedString);
        String decodedString = new String(decodedBytes);
        System.out.println(decodedString); //hello!你好,小明！
```

**注意：**

> 除了上面介绍的base64类进行base64加密解密，jdk的BASE64Decoder类也提供了base64加密和解密。
> 但是不推荐使用BASE64Decoder类进行加密解密。
> 因为BASE64Decoder是对MIME友好的，编码后的字符串如果超过76个字符就会换行，所以BASE64Decoder编码后的字符串会后\n\r这样的字符，在一些处理\n\r的代码里可能会有问题。如果你非要使用BASE64Decoder类进行加密解密，请使用replaceAll("\r|\n", "")对编码后的\n和\r进行替换。


# 二、MIME友好型base64加密与解密

MIME友好型base64加密与解密即加密后如果长度每大于76就会加入\r\n这样的换行控制符
```java
		//Base64编码
		String str = "727dced7-15c7-48c6-bb11-416ab51f98bc-2a19434a-3a64-496e-b07b-b51b0445384c-22525be7-82c7-4a72-8594-238712d4d59e";
        byte[] encodedAsBytes = str.getBytes();
        String encodedMime = Base64.getMimeEncoder().encodeToString(encodedAsBytes);
        System.out.println("编码:"+encodedMime);

        //Base64解码
        byte[] decodedBytes = Base64.getMimeDecoder().decode(encodedMime);
        String decodedMime = new String(decodedBytes);
        System.out.println("解码:"+decodedMime); //727dced7-15c7-48c6-bb11-416ab51f98bc-2a19434a-3a64-496e-b07b-b51b0445384c-22525be7-82c7-4a72-8594-238712d4d59e
```