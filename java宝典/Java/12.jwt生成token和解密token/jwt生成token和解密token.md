[TOC]

# 一、引入依赖

```xml
<dependency>
    <groupId>com.auth0</groupId>
    <artifactId>java-jwt</artifactId>
    <version>3.18.2</version>
</dependency>

<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-core</artifactId>
    <version>2.13.2</version>
</dependency>
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>2.13.2.2</version>
</dependency>
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-annotations</artifactId>
    <version>2.13.2</version>
</dependency>
```

# 二、工具类

```java

import com.auth0.jwt.JWT;
import com.auth0.jwt.JWTCreator;
import com.auth0.jwt.JWTVerifier;
import com.auth0.jwt.algorithms.Algorithm;
import com.auth0.jwt.exceptions.JWTVerificationException;
import com.auth0.jwt.interfaces.Claim;
import com.auth0.jwt.interfaces.DecodedJWT;

import java.util.Calendar;
import java.util.Date;
import java.util.HashMap;
import java.util.Map;

public class JwtUtil {
    /**
     * 生成token的秘钥
     */
    private static String TOKEN_KEY = "54j12048-1f56-45c1-a3f1-3c549bc2bb47";
    /**
     * token颁布者
     */
    private static String ISSure = "XiaoHaiTang";
    /**
     * token有效期2小时
     */
    private static int TOKEN_TIMEOUT = 2;


    /**
     * 生成token
     *
     * @param claims 要放入token中的信息
     */
    public static String create(Map<String, String> claims) {
        Map<String, Object> headerMap = new HashMap<>();
        headerMap.put("alg", "HS256");
        headerMap.put("typ", "JWT");
        Calendar c = Calendar.getInstance();
        c.add(Calendar.HOUR_OF_DAY, TOKEN_TIMEOUT);
        JWTCreator.Builder builder = JWT.create()
                .withHeader(headerMap)
                .withIssuer(ISSure)   //发布者
                .withIssuedAt(new Date())   //生成签名的时间
                .withExpiresAt(c.getTime());//token有效期
        //插入值
        for (String key : claims.keySet()) {
            builder.withClaim(key, claims.get(key));
        }
        //使用HS256加密算法生成token
        Algorithm algorithm = Algorithm.HMAC256(TOKEN_KEY);
        String token = builder.sign(algorithm);
        return token;
    }

    /**
     * 验证token是否有效
     */
    public static boolean verifierToken(String token) {
        try {
            Algorithm algorithm = Algorithm.HMAC256(TOKEN_KEY);
            JWTVerifier verifier = JWT.require(algorithm)
                    .withIssuer(ISSure) //匹配指定的token发布者 auth0
                    .build();
            DecodedJWT jwt = verifier.verify(token); //解码JWT ，verifier 可复用
            /*
            System.out.println(jwt.getToken());//打印完整token
            System.out.println(jwt.getHeader());//打印token的头部
            System.out.println(jwt.getPayload());//打印token的荷载
            System.out.println(jwt.getSignature());//打印token的签名
            System.out.println(jwt.getExpiresAt());//打印token的过期时间
            */
            return true;
        } catch (JWTVerificationException e) {
            //无效的签名/声明
            System.out.println("验证的token无效:"+ e.getMessage());
            return false;
        }
    }


    /**
     * 获取token中放入的信息
     */
    public static Map<String, Claim> getTokenInfo(String token) {
        Map<String, Claim> map = null;
        try {
            Algorithm algorithm = Algorithm.HMAC256(TOKEN_KEY);
            JWTVerifier verifier = JWT.require(algorithm)
                    .withIssuer(ISSure) //匹配指定的token发布者 auth0
                    .build();
            DecodedJWT jwt = verifier.verify(token);
            map = jwt.getClaims();
        } catch (JWTVerificationException e) {
            //无效的签名/声明
            System.out.println("验证的token无效");
            e.printStackTrace();
        }
        return map;
    }
}


```

# 三、测试

```java

import com.auth0.jwt.interfaces.Claim;
import com.service.JwtUtil;

import java.util.HashMap;
import java.util.Map;

public class MainServer {
    public static void main(String[] args) {
        //获取token
        Map<String, String> map = new HashMap<>();
        map.put("name", "张三");
        map.put("id", "1024");
        String token = JwtUtil.create(map);
        System.out.println("生成的token:" + token);

        //token中获取信息
        Map<String, Claim> resultMap = JwtUtil.getTokenInfo(token);
        Claim name = resultMap.get("name");
        Claim id = resultMap.get("id");
        System.out.println("获取token中的name:" + name);
        System.out.println("获取token中的id:" + id);

        //验证token是否有效
        boolean isEfficient = JwtUtil.verifierToken(token);
        System.out.println("token是否有效：" + isEfficient);
    }
  }
```
**输出：**

```java
生成的token:eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJYaWFvSGFpVGFuZyIsIm5hbWUiOiLlvKDkuIkiLCJpZCI6IjEwMjQiLCJleHAiOjE2NTA2MDg0OTYsImlhdCI6MTY1MDYwMTI5Nn0.ZwIVouqvUh9gH3o7XjntzaXHadfFLRilEazwSMixmik
获取token中的name:"张三"
获取token中的id:"1024"
token是否有效：true

```

参考：
[使用auth0构建JWT](https://www.jianshu.com/p/0fed399c2561)
[Token插件：Auth0和jjwt对比](https://blog.csdn.net/lizz861109/article/details/104614942/)
[SpringBoot集成JWT实现token验证](https://www.jianshu.com/p/e88d3f8151db)