---
layout:     post
title:      "HttpUtils 工具类"
subtitle:   "Hello World, Hello Blog"
date:       2020-10-26
author:     "Caiiiiii"
header-img: "img/1174180_screenshots_20201019222158_1.jpg"
tags:
    - 工具
    - Java  
---


# HttpUtils
```
package com.video.ytb_to_bili.httputils;

import com.google.api.client.util.Value;
import com.google.protobuf.ServiceException;
import org.springframework.context.annotation.PropertySource;

import java.io.*;
import java.net.*;

public class HttpUtils {


    public static int port = 23333;

    public HttpUtils() {
    }

    /**
     * 提交json数据格式的post请求
     * @param url
     * @param requstBody
     * @return
     * @throws Exception
     */
    public static String doPost(String url, String requstBody) throws Exception {
        return doPost(url, requstBody, "application/json; charset=UTF-8", "POST", "UTF-8");
    }

    /**
     * 处理post请求
     * @param url
     * @param requstBody
     * @param contentType
     * @param requestMethod
     * @param charset
     * @return
     * @throws Exception
     */
    public static String doPost(String url, String requstBody, String contentType, String requestMethod, String charset) throws Exception {
        StringBuffer buffer = new StringBuffer();
        InputStream inputStream = null;
        OutputStream outputStream = null;
        BufferedReader reader = null;
        OutputStreamWriter writer = null;
        URL urlObject = null;
        HttpURLConnection connection = null;

        try {
            urlObject = new URL(url);
            InetSocketAddress address = new InetSocketAddress("127.0.0.1", port);
            Proxy proxy = new Proxy(Proxy.Type.HTTP, address); // http代理协议类型
            connection = (HttpURLConnection)urlObject.openConnection(proxy);
            connection.setRequestMethod(requestMethod);//设置请求方式为POST
            connection.setRequestProperty("Accept", contentType);
            connection.setRequestProperty("Charset", charset);//设置字符编码
            connection.setRequestProperty("Content-type", contentType);//设置参数类型
            connection.setDoInput(true);//允许写出
            connection.setDoOutput(true);//允许读入
            connection.setConnectTimeout(300000);//连接主机的超时时间（单位：毫秒）
            connection.setReadTimeout(300000);//从主机读取数据的超时时间（单位：毫秒）
            connection.setAllowUserInteraction(true);//为true，则在允许用户交互（例如弹出一个验证对话框）的上下文中对此URL进行检查
            connection.connect();//建立连接
            outputStream = connection.getOutputStream();
            writer = new OutputStreamWriter(outputStream, charset);
            writer.write(requstBody);
            writer.flush();
            writer.close();
            checkResponseCode(connection.getResponseCode());
            inputStream = connection.getInputStream();
            reader = new BufferedReader(new InputStreamReader(inputStream, charset));
            String line = null;

            while((line = reader.readLine()) != null) {
                buffer.append(line);
            }
        } catch (Exception var16) {
            throw var16;
        } finally {
            if (reader != null) {
                reader.close();
            }

            if (writer != null) {
                writer.close();
            }

            if (outputStream != null) {
                outputStream.close();
            }

            if (inputStream != null) {
                inputStream.close();
            }

            if (connection != null) {
                connection.disconnect();
            }

        }

        return buffer.toString();
    }

    /**
     * 提交带参数的get请求
     * @param url
     * @param charset
     * @return
     * @throws Exception
     */
    public static String doGet(String url, String charset) throws Exception {
        return doGet(url, "application/json; charset=" + charset, charset);
    }

    /**
     * 处理get请求
     * @param url
     * @param contentType
     * @param charset
     * @return
     * @throws Exception
     */
    public static String doGet(String url, String contentType, String charset) throws Exception {
        BufferedReader in = null;
        HttpURLConnection conn = null;

        try {
            url = dealUrlParameter(url, charset);
            URL realUrl = new URL(url);
            InetSocketAddress address = new InetSocketAddress("127.0.0.1", port);
            Proxy proxy = new Proxy(Proxy.Type.HTTP, address); // http代理协议类型
            conn = (HttpURLConnection)realUrl.openConnection(proxy);
            conn.setRequestMethod("GET");
            conn.setRequestProperty("Accept", "*/*");
            conn.setRequestProperty("Charset", charset);
            conn.setRequestProperty("Accept-Charset", charset);
            conn.setRequestProperty("Connection", "Keep-Alive");
            conn.setRequestProperty("User-Agent", "Mozilla/5.0 (compatible; MSIE 6.0; Windows NT 5.1;SV1)");
            conn.setRequestProperty("Content-Type", contentType);
            conn.setConnectTimeout(300000);
            conn.setReadTimeout(300000);
            conn.setUseCaches(false);
            checkResponseCode(conn.getResponseCode());
            in = new BufferedReader(new InputStreamReader(conn.getInputStream(), charset));
            StringBuilder builderLine = new StringBuilder();

            String line;
            while((line = in.readLine()) != null) {
                builderLine.append(line);
            }

           // logger.debug(builderLine.toString());
            String var8 = builderLine.toString();
            return var8;
        } catch (Exception var17) {
           // logger.error(var17.toString());
            throw var17;
        } finally {
            try {
                if (conn != null) {
                    conn.disconnect();
                }

                if (in != null) {
                    in.close();
                }
            } catch (IOException var16) {
                var16.printStackTrace();
            }

        }
    }

    /**
     * 处理get请求传递的参数
     * @param url
     * @param charset
     * @return
     * @throws IOException
     */
    public static String dealUrlParameter(String url, String charset) throws IOException {
        int parameterIndex = url.indexOf("?");
        if (parameterIndex==-1) {
            return url;
        } else {
            String parameter = url.substring(parameterIndex + 1);
            String[] parameterArray = parameter.split("&");
            StringBuffer newParameter = new StringBuffer();
            String[] var6 = parameterArray;
            int var7 = parameterArray.length;

            for(int var8 = 0; var8 < var7; ++var8) {
                String kv = var6[var8];
                String[] keyValue = kv.split("=");
                String vlaue = keyValue.length == 2 && keyValue[1] != null ? URLEncoder.encode(keyValue[1], charset) : "";
                if (newParameter.length() == 0) {
                    newParameter.append(keyValue[0]).append("=").append(vlaue);
                } else {
                    newParameter.append("&").append(keyValue[0]).append("=").append(vlaue);
                }
            }

            return url.substring(0, parameterIndex) + "?" + newParameter;
        }
    }

    /**
     * 检验是否交互成功
     * @param responseCode
     */
    public static void checkResponseCode(int responseCode) throws ServiceException {
        if (responseCode < 400) {
           // logger.info("请求交互成功");
        } else if (responseCode < 500) {
            if (responseCode == 429) {
                System.out.println("responseCode:"+responseCode);
                throw new ServiceException(HttpExceptionEnum.REQUEST_NULL.toString());
            } else {
                System.out.println("responseCode:"+responseCode);
                throw new ServiceException(HttpExceptionEnum.REQUEST_NULL.toString());
            }
        } else {
            System.out.println("responseCode:"+responseCode);
            throw new ServiceException(HttpExceptionEnum.SERVER_ERROR.toString());
        }
    }
}
```


# HttpExceptionEnum
```
package com.video.ytb_to_bili.httputils;

public enum  HttpExceptionEnum {
    REQUEST_NULL("请求429"),
    SERVER_ERROR("系统错误")
    ;


    private String msg;
    HttpExceptionEnum(String msg) {
          this.msg = msg;
    }
}

```