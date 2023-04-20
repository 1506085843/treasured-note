[TOC]

# 前言
最近在用到了HttpClient发送post和get请求三方接口的需求，所以写了个封装类方便以后用到

# 一、maven依赖
需要用到的一些jar的maven坐标如下：

```xml
		<dependency>
            <groupId>org.apache.httpcomponents</groupId>
            <artifactId>httpclient</artifactId>
            <version>4.5.5</version>
        </dependency>

        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>fastjson</artifactId>
            <version>1.2.7</version>
        </dependency>

        <dependency>
            <groupId>org.apache.commons</groupId>
            <artifactId>commons-lang3</artifactId>
            <version>3.11</version>
        </dependency>

       <dependency>
         <groupId>commons-logging</groupId>
         <artifactId>commons-logging</artifactId>
         <version>1.2</version>
      </dependency>
```
# 二、HttpUtil封装类
HttpUtil类如下：
```java


import com.alibaba.fastjson.JSON;
import org.apache.commons.lang3.StringUtils;
import org.apache.http.*;
import org.apache.http.client.ClientProtocolException;
import org.apache.http.client.HttpClient;
import org.apache.http.client.config.RequestConfig;
import org.apache.http.client.methods.CloseableHttpResponse;
import org.apache.http.client.methods.HttpGet;
import org.apache.http.client.methods.HttpPost;
import org.apache.http.client.methods.HttpUriRequest;
import org.apache.http.entity.ContentType;
import org.apache.http.entity.StringEntity;
import org.apache.http.entity.mime.HttpMultipartMode;
import org.apache.http.entity.mime.MultipartEntityBuilder;
import org.apache.http.entity.mime.content.StringBody;
import org.apache.http.impl.client.CloseableHttpClient;
import org.apache.http.impl.client.HttpClients;
import org.apache.http.protocol.BasicHttpContext;
import org.apache.http.protocol.HttpContext;
import org.apache.http.protocol.HttpCoreContext;
import org.apache.http.util.EntityUtils;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.util.CollectionUtils;
import org.apache.http.client.entity.UrlEncodedFormEntity;
import org.apache.http.message.BasicNameValuePair;
import org.springframework.web.multipart.MultipartFile;

import java.io.*;
import java.nio.charset.StandardCharsets;
import java.util.*;

public class HttpUtil {
    private static final Logger logger = LoggerFactory.getLogger(HttpUtil.class);
    private static RequestConfig requestConfig = RequestConfig.custom().setSocketTimeout(60000).setConnectTimeout(60000).build();
    private static String chaset = "UTF-8";    // 默认编码

    /**
     * 发送 GET 请求（HTTP），无参数
     *
     * @param url
     * @return
     */
    public static String doGet(String url) {
        if (StringUtils.isBlank(url)) {
            return "url不能为空";
        }
        // 创建http GET请求
        logger.info("url:{}", url);
        HttpGet httpGet = new HttpGet(url);
        // 设置请求和传输超时时间
        httpGet.setConfig(requestConfig);
        String result = "";
        try (CloseableHttpClient httpclient = HttpClients.createDefault();// 创建Httpclient对象
             CloseableHttpResponse response = httpclient.execute(httpGet);// 执行请求
        ) {
            int statusCode = response.getStatusLine().getStatusCode();
            if (statusCode == HttpStatus.SC_OK) {
                result = EntityUtils.toString(response.getEntity(), chaset);
                logger.info("HttpGet方式请求成功！返回结果：{}", result);
            } else {
                logger.info("HttpGet方式请求失败！状态码:" + statusCode);
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
        return result;
    }

    /**
     * 发送 GET 请求（HTTP），有参数
     *
     * @param url
     * @param params
     * @return
     */
    public static String doGet(String url, Map<String, Object> params) {
        if (!CollectionUtils.isEmpty(params)) {
            StringBuffer extraUrl = new StringBuffer();
            int temp = 0;
            for (String key : params.keySet()) {
                if (temp == 0) {
                    extraUrl.append("?");
                } else {
                    extraUrl.append("&");
                }
                extraUrl.append(key).append("=").append(params.get(key));
                temp++;
            }
            url += extraUrl;
        }
        return doGet(url);
    }


    /**
     * 发送 GET 请求（HTTP），获取跳转的最终url链接返回
     *
     * @param url
     * @return
     */
    public static String doGetJumpLink(String url) {
        if (StringUtils.isBlank(url)) {
            return "url不能为空";
        }
        // 创建http GET请求
        HttpGet httpGet = new HttpGet(url);
        HttpContext context = new BasicHttpContext();
        String currentUrl = "";
        try (CloseableHttpClient httpclient = HttpClients.createDefault();// 创建Httpclient对象
             CloseableHttpResponse response = httpclient.execute(httpGet, context);// 执行请求
        ) {
            int statusCode = response.getStatusLine().getStatusCode();
            if (statusCode != HttpStatus.SC_OK) {
                return currentUrl = "HttpGet方式请求失败！状态码:" + statusCode;

            }
            //logger.info("HttpGet方式请求成功！返回结果：{}", result);
            HttpUriRequest currentReq = (HttpUriRequest) context.getAttribute(
                    HttpCoreContext.HTTP_REQUEST);
            HttpHost currentHost = (HttpHost) context.getAttribute(
                    HttpCoreContext.HTTP_TARGET_HOST);
            currentUrl = (currentReq.getURI().isAbsolute()) ? currentReq.getURI().toString() : (currentHost.toURI() + currentReq.getURI());
        } catch (IOException e) {
            e.printStackTrace();
        }
        return currentUrl;
    }


    /**
     * 发送 POST 请求（HTTP），无参数
     *
     * @param url
     * @return
     */
    public static String doPost(String url) {
        return doPost(url, null);
    }

    /**
     * 发送 POST 请求（HTTP），有参数
     *
     * @param url
     * @return
     */
    public static String doPost(String url, Map<String, Object> params) {
        // 创建http POST请求
        HttpPost httpPost = new HttpPost(url);
        httpPost.setConfig(requestConfig);
        httpPost.setHeader("Content-Type", "application/json");
        if (params != null) {
            httpPost.setEntity(new StringEntity(JSON.toJSONString(params), ContentType.create("application/json", "utf-8")));
        }
        String result = "";
        try (CloseableHttpClient httpclient = HttpClients.createDefault();// 创建Httpclient对象
             CloseableHttpResponse response = httpclient.execute(httpPost);// 执行请求
        ) {
            logger.info("httpPost:{}", httpPost);
            int statusCode = response.getStatusLine().getStatusCode();
            if (statusCode == HttpStatus.SC_OK) {
                result = EntityUtils.toString(response.getEntity(), chaset);
                logger.info("HttpPost方式请求成功！返回结果：{}", result);
            } else {
                logger.info("HttpPost方式请求失败！状态码:" + statusCode);
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
        return result;
    }


    public static String doPost1(String url, Map<String, Object> paramMap) {
        CloseableHttpClient httpClient = null;
        CloseableHttpResponse httpResponse = null;
        String result = "";
        // 创建httpClient实例
        httpClient = HttpClients.createDefault();
        // 创建httpPost远程连接实例
        HttpPost httpPost = new HttpPost(url);
        // 配置请求参数实例
        RequestConfig requestConfig = RequestConfig.custom().setConnectTimeout(5000)// 设置连接主机服务超时时间
                .setConnectionRequestTimeout(5000)// 设置连接请求超时时间
                .setSocketTimeout(5000)// 设置读取数据连接超时时间
                .build();
        // 为httpPost实例设置配置
        httpPost.setConfig(requestConfig);
        // 设置请求头
        httpPost.addHeader("Content-Type", "application/x-www-form-urlencoded");
        // 封装post请求参数
        if (null != paramMap && paramMap.size() > 0) {
            List<NameValuePair> nvps = new ArrayList<NameValuePair>();
            // 通过map集成entrySet方法获取entity
            Set<Map.Entry<String, Object>> entrySet = paramMap.entrySet();
            // 循环遍历，获取迭代器
            Iterator<Map.Entry<String, Object>> iterator = entrySet.iterator();
            while (iterator.hasNext()) {
                Map.Entry<String, Object> mapEntry = iterator.next();
                nvps.add(new BasicNameValuePair(mapEntry.getKey(), mapEntry.getValue().toString()));
            }
            // 为httpPost设置封装好的请求参数
            try {
                httpPost.setEntity(new UrlEncodedFormEntity(nvps, "UTF-8"));
            } catch (UnsupportedEncodingException e) {
                e.printStackTrace();
            }
        }
        try {
            // httpClient对象执行post请求,并返回响应参数对象
            httpResponse = httpClient.execute(httpPost);
            // 从响应对象中获取响应内容
            HttpEntity entity = httpResponse.getEntity();
            result = EntityUtils.toString(entity);
        } catch (ClientProtocolException e) {
            logger.error("错误信息: ", e);
        } catch (IOException e) {
            logger.error("错误信息: ", e);
        } finally {
            // 关闭资源
            if (null != httpResponse) {
                try {
                    httpResponse.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
            if (null != httpClient) {
                try {
                    httpClient.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
        return result;
    }


    /**
     * 发送 POST 请求（HTTP），有参数和cookie
     *
     * @param url
     * @param params 参数
     * @param cookie 例如：LBSESSION=NGEzODUyNjUtYjZkYy00ZmNkLWJhZTAtMDBmYWNjYzQzZjBi
     * @return
     */
    public static String doPost(String url, Map<String, Object> params, String cookie) {
        // 创建http POST请求
        HttpPost httpPost = new HttpPost(url);
        httpPost.setConfig(requestConfig);
        httpPost.setHeader("Content-Type", "application/json");
        if (params != null) {
            httpPost.setEntity(new StringEntity(JSON.toJSONString(params), ContentType.create("application/json", "utf-8")));
        }
        if (cookie != null) {
            httpPost.addHeader("Cookie", cookie);
        }
        String result = "";
        try (CloseableHttpClient httpclient = HttpClients.createDefault();// 创建Httpclient对象
             CloseableHttpResponse response = httpclient.execute(httpPost);// 执行请求
        ) {
            logger.info("httpPost:{}", httpPost);
            int statusCode = response.getStatusLine().getStatusCode();
            if (statusCode == HttpStatus.SC_OK) {
                result = EntityUtils.toString(response.getEntity(), chaset);
                logger.info("HttpPost方式请求成功！返回结果：{}", result);
            } else {
                logger.info("HttpPost方式请求失败！状态码:" + statusCode);
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
        return result;
    }

    /**
     * post调用三方请求上传单个文件
     *
     * @param url
     * @param file
     * @return
     */
    public static String postUploadFile(String url, MultipartFile file) {
        HttpEntity httpEntity = null;
        String fileName = file.getOriginalFilename();
        HttpPost httpPost = new HttpPost(url);
        MultipartEntityBuilder builder = MultipartEntityBuilder.create();
        builder.setCharset(StandardCharsets.UTF_8);
        builder.setMode(HttpMultipartMode.BROWSER_COMPATIBLE);//加上此行代码解决返回中文乱码问题
        try {
            builder.addBinaryBody("file", file.getInputStream(), ContentType.MULTIPART_FORM_DATA, fileName);// 文件流
        } catch (IOException e) {
            logger.error("错误信息: ", e);
        }

        httpEntity = builder.build();
        httpPost.setEntity(httpEntity);
        String result = "";
        try (CloseableHttpClient httpclient = HttpClients.createDefault();// 创建Httpclient对象
             CloseableHttpResponse response = httpclient.execute(httpPost);// 执行请求
        ) {
            System.out.println("httpPost:" + httpPost);
            int statusCode = response.getStatusLine().getStatusCode();
            if (statusCode == HttpStatus.SC_OK) {
                result = EntityUtils.toString(response.getEntity(), chaset);
                logger.info("HttpPost方式请求成功！返回结果：{}", result);
            } else {
                logger.info("HttpPost方式请求失败！状态码:" + statusCode);
            }
        } catch (IOException e) {
            logger.error("错误信息: ", e);
        }
        return result;
    }

    /**
     * post调用三方请求上传多个文件
     *
     * @param url
     * @param files 多个文件
     * @return params 多个参数
     */
    public static String postUploadFile(String url, List<MultipartFile> files, Map<String, String> params) {
        HttpEntity httpEntity = null;
        HttpPost httpPost = new HttpPost(url);
        MultipartEntityBuilder builder = MultipartEntityBuilder.create();
        builder.setCharset(StandardCharsets.UTF_8);
        builder.setMode(HttpMultipartMode.BROWSER_COMPATIBLE);//加上此行代码解决返回中文乱码问题
        try {
            if (null != files) {
                for (MultipartFile file : files) {
                    String fileName = file.getOriginalFilename();
                    builder.addBinaryBody("files", file.getInputStream(), ContentType.MULTIPART_FORM_DATA, fileName);// 文件流
                }
            }
        } catch (IOException e) {
            logger.error("错误信息: ", e);
        }

        if (null != params) {
            for (Map.Entry<String, String> m : params.entrySet()) {
                builder.addPart(m.getKey(), new StringBody(m.getValue(), ContentType.create("text/plain", StandardCharsets.UTF_8)));
            }
        }

        httpEntity = builder.build();
        httpPost.setEntity(httpEntity);
        String result = "";
        try (CloseableHttpClient httpclient = HttpClients.createDefault();
             CloseableHttpResponse response = httpclient.execute(httpPost);
        ) {
            System.out.println("httpPost:" + httpPost);
            int statusCode = response.getStatusLine().getStatusCode();
            if (statusCode == HttpStatus.SC_OK) {
                result = EntityUtils.toString(response.getEntity(), chaset);
                logger.info("HttpPost方式请求成功！返回结果：{}", result);
            } else {
                logger.info("HttpPost方式请求失败！状态码:" + statusCode);
            }
        } catch (IOException e) {
            logger.error("错误信息: ", e);
        }
        return result;
    }


    /**
     * 通过url下载视频
     *
     * @param url
     * @return
     */
    public static String downVideo(String url) {
        //默认下载到D:\video\
        return downVideo(url, "D:\\video\\");
    }


    /**
     * 通过url下载视频
     *
     * @param url
     * @param filepath 下载到本地的路径
     * @return
     */
    public static String downVideo(String url, String filepath) {
        String filename = "";
        try {
            //获取视频流
            HttpClient client = HttpClients.createDefault();
            HttpGet httpget = new HttpGet(url);
            HttpResponse response = client.execute(httpget);
            HttpEntity entity = response.getEntity();
            InputStream is = entity.getContent();

            //filepath不是斜杠结束的在末尾添加斜杠
            if (filepath.length() - 1 != filepath.lastIndexOf("\\")) {
                filepath += "\\";
            }

            //如果没有文件夹，则创建
            File file = new File(filepath);
            if (!file.exists() && !file.isDirectory()) {
                file.mkdirs();
            }

            //获取文件夹下的文件数量，文件数量加一作为要下载的文件名
            int fileCount = 0;
            File[] list = file.listFiles();
            for (int i = 0; i < list.length; i++) {
                if (list[i].isFile()) {
                    fileCount++;
                }
            }
            fileCount += 1;

            //如果文件不存在则创建文件
            filename = filepath + fileCount + ".mp4";
            File file_name = new File(filename);
            if (!file_name.exists()) {
                //创建文件
                file_name.createNewFile();
            }

            //视频流写入文件
            FileOutputStream fileout = new FileOutputStream(file_name);
            BufferedInputStream buf = new BufferedInputStream(is);
            byte[] buffer = new byte[1024];
            int ch = 0;
            while ((ch = buf.read(buffer)) != -1) {
                fileout.write(buffer, 0, ch);
            }
            is.close();
            fileout.flush();
            fileout.close();
        } catch (Exception e) {
            e.printStackTrace();
        }
        return "下载成功，文件名：" + filename;
    }
}


```

# 三、测试
## 1.get请求测试：
```java
	Map<String, Object> map = new HashMap<>();
        map.put("name", "张三");
        map.put("age", "18");
	String result = HttpUtil.doGet("http://125.113.27.170:8080/query/v1/addition",map);
```

## 2.post请求测试：
```java
        Map<String, Object> map = new HashMap<>();
        map.put("name", "张三");
        map.put("age", "18");
        String result = HttpUtil.doPost("http://125.113.27.170:8080/query/v1/myapi",map);
```

## 3.上传多文件携带多参数测试：
假设要调用三方接口 receiveFiles  然后把多个文件数据和一些参数带过去：

（需要注意的是：receiveFiles 接口中的 @RequestParam 里相关的值，要和 HttpUtil 工具类中postUploadFile 方法里添加到的 addBinaryBody 中的一致。）
```java
    public void upLoadFiles(List<MultipartFile> files) {
        Map<String, String> params = new HashMap<>();
        params.put("name", "张三");
        params.put("age", "18");
        //调用三方receiveFiles接口，并把多个文件和参数传过去
        String result = HttpUtil.postUploadFile("http://localhost:8080/apiList/download/receiveFiles", files, params);
    }
```
这里用 SpringBoot 模拟一个 receiveFiles 接口给上面的postUploadFile方法调用测试：

```java
	@ApiOperation("接收多文件")
    @PostMapping("/download/receiveFiles")
    public String create(@RequestParam(value="files") List<MultipartFile> files, @RequestParam("name") String name, @RequestParam("age") String age) throws IOException {
        for (MultipartFile file : files) {
        	//接收文件保存到F:\reciveFile目录下
            file.transferTo( new File("F:\\reciveFile\\", file.getOriginalFilename()));
        }
        System.out.println("姓名："+name);
        System.out.println("年龄："+age);
        return "接收文件成功！";
    }
```

---
参考：
[HttpClient 4 - how to capture last redirect URL](https://stackoverflow.com/questions/1456987/httpclient-4-how-to-capture-last-redirect-url)
[使用httpclient获得服务器跳转之后的url！](https://my.oschina.net/banxi/blog/69570)