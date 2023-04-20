本文使用的AES加密模式是AES-128-CBC。

UsualAES工具类：

```java

import javax.crypto.Cipher;
import javax.crypto.spec.IvParameterSpec;
import javax.crypto.spec.SecretKeySpec;
import java.util.Base64;

public  class AesUtils {

    /**
     * AES_KEY 加密解密用的密钥Key，可以用26个字母和数字组成的16位，此处使用AES-128-CBC加密模式。
     * AES_VECTOR 向量， 普通aes加密解密需要为16位。
     */
    public static final String AES_KEY = "LX%TB*19!@#HPAZW";
    public static final String AES_VECTOR = "123f5678901234ad";

    
    /**
     * 加密
     */
    public static String encrypt(String sSrc, String key, String vector) throws Exception {
        Cipher cipher = Cipher.getInstance("AES/CBC/PKCS5Padding");
        byte[] raw = key.getBytes();
        SecretKeySpec skeySpec = new SecretKeySpec(raw, "AES");
        IvParameterSpec iv = new IvParameterSpec(vector.getBytes());// 使用CBC模式，需要一个向量iv，可增加加密算法的强度
        cipher.init(Cipher.ENCRYPT_MODE, skeySpec, iv);
        byte[] encrypted = cipher.doFinal(sSrc.getBytes("utf-8"));
        return new String(Base64.getEncoder().encode(encrypted));// 此处使用BASE64做转码。
    }

    /**
     * 解密
     */
    public static String decrypt(String sSrc, String key, String ivs) throws Exception {
        try {
            byte[] raw = key.getBytes("ASCII");
            SecretKeySpec skeySpec = new SecretKeySpec(raw, "AES");
            Cipher cipher = Cipher.getInstance("AES/CBC/PKCS5Padding");
            IvParameterSpec iv = new IvParameterSpec(ivs.getBytes());
            cipher.init(Cipher.DECRYPT_MODE, skeySpec, iv);
            byte[] encrypted1 = Base64.getDecoder().decode(sSrc);// 先用base64解密
            byte[] original = cipher.doFinal(encrypted1);
            String originalString = new String(original, "utf-8");
            return originalString;
        } catch (Exception ex) {
            ex.printStackTrace();
            return null;
        }
    }


    public static String encodeBytes(byte[] bytes) {
        StringBuffer strBuf = new StringBuffer();
        for (int i = 0; i < bytes.length; i++) {
            strBuf.append((char) (((bytes[i] >> 4) & 0xF) + ((int) 'a')));
            strBuf.append((char) (((bytes[i]) & 0xF) + ((int) 'a')));
        }
        return strBuf.toString();
    }

     /**
     * 增强加密 密码为原始12位+4位随机数，随机数带入密文中,所以相同的内容每次加密后密文不一样
     */
    public static String encryptAd(String content, String key, String iv) {
        key = key.substring(0,12);
        int num = (int) (Math.random() * 9000 + 1000);
        String newKey= key + num;
        String tt4 = null;
        try {
            tt4 = encrypt(content, newKey, iv);
        } catch (Exception e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }
        tt4 = tt4.replace("=", "!");
        tt4 = tt4.replace('+', '[');
        tt4 = tt4.replace('/', ']');
        String result = tt4 + num;
        return result;
    }

    /**
     * 增强解密 明文[0,15] 位：密文为24 +4=28 位 明文[16,31] 位：密文为44 +4=48 位
     */
    public static String decryptAd(String data, String skey, String iv)  {
        skey = skey.substring(0,12);
        data = data.replace("!", "=");
        data = data.replace('[', '+');
        data = data.replace(']', '/');
        int len = data.length();

        String realkey = skey + data.substring(len - 4, len);
        String realcontent = data.substring(0, len - 4);
        String result = null;
        try {
            result = decrypt(realcontent, realkey, iv);
        } catch (Exception e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }
        return result;
    }
}
```

测试：

```java
public class Test {
    public static void main(String[] args) throws Exception {
        String str="Hello!";//原始字符串

        String encryption = null;
        String decrypt= null;
        try {
            encryption = AesUtils.encrypt(str, AesUtils.AES_KEY, AesUtils.AES_VECTOR);//普通加密
            decrypt = AesUtils.decrypt(encryption, AesUtils.AES_KEY, AesUtils.AES_VECTOR);//普通解密
        } catch (Exception e) {
            e.printStackTrace();
        }
        System.out.println("加密:"+encryption);
        System.out.println("解密:"+decrypt);

        String encryption1 = AesUtils.encryptAd(str, AesUtils.AES_KEY, AesUtils.AES_VECTOR);//增强加密
        String decrypt1= AesUtils.decryptAd(encryption1, AesUtils.AES_KEY, AesUtils.AES_VECTOR);//增强解密
        System.out.println("增强加密:"+encryption1);
        System.out.println("增强解密:"+decrypt1);

    }
}
```
输出结果：

```java
加密:sLo1WdTUW5z8D6qHNFqFrg==
解密:Hello!
增强加密:LO4NVJNWCE556GlVuv1]Pw!!8327
增强解密:Hello!
```
**注意：**
增强加密因为加入了随机数，所以相同的内容每次使用增强加密后会变化。比如，上面的 Hello 增强加密后，这次得到的密文是LO4NVJNWCE556GlVuv1]Pw!!8327   ，  下一次再加密可能就是1QNgS]xcgGO]CTjSSJHeHQ!!2358  。