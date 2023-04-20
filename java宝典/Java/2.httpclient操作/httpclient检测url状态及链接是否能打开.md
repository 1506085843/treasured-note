**有时候我们需要检测某个url返回的状态码是不是200或者页面能不能正常打开响应可使用如下代码：**


**需要使用到的maven：**

```xml
<dependency>
    <groupId>org.apache.httpcomponents</groupId>
    <artifactId>httpclient</artifactId>
    <version>4.5.5</version>
</dependency>
<dependency>
	<groupId>org.apache.httpcomponents</groupId>
    <artifactId>httpcore</artifactId>
    <version>4.4.14</version>
</dependency>
<dependency>
    <groupId>commons-logging</groupId>
    <artifactId>commons-logging</artifactId>
    <version>1.2</version>
</dependency>
```
**代码：**
```java
    public static String checkUrlConnection(String url) {
        // 创建http POST请求
        HttpGet httpGet = new HttpGet(url);
        httpGet.setHeader("Content-Type", "application/json");
        RequestConfig requestConfig = RequestConfig.custom().setConnectTimeout(600)// 设置连接主机服务超时时间
                .setConnectionRequestTimeout(1000)// 设置连接请求超时时间
                .setSocketTimeout(1000)// 设置读取数据连接超时时间
                .build();
        // 为httpPost实例设置配置
        httpGet.setConfig(requestConfig);
        // 设置请求头
        CloseableHttpClient httpclient = null;
        CloseableHttpResponse response = null;
        int statusCode = 404;
        try {
            httpclient = HttpClients.createDefault();// 创建Httpclient对象
            response = httpclient.execute(httpGet);// 执行请求
            statusCode = response.getStatusLine().getStatusCode();
        }catch (SocketException e) {
            return "404";
        } catch (IOException e) {
            System.out.println("报错");
            return "404";
        }
        return String.valueOf(statusCode);
    }
```

---
**参考：**
[HttpURLConnection timeout settings](https://stackoverflow.com/questions/2799938/httpurlconnection-timeout-settings)
[https://www.baeldung.com/java-check-url-exists](https://www.baeldung.com/java-check-url-exists)
[Java HTTP Client Request with defined timeout](https://stackoverflow.com/questions/3000214/java-http-client-request-with-defined-timeout)