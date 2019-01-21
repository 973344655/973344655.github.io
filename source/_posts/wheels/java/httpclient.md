---
title: httpclient
date: 2019-01-15 16:21:54
tags: wheels
---
```
package util;

import org.omg.CORBA.portable.OutputStream;

import java.io.*;
import java.net.HttpURLConnection;
import java.net.URL;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

/**
 * 发送与接收都是String  接收到String类型的返回后自己用fastJson转一下
 */
public class HttpClientUtil {
    /**
     * url为地址，params 为请求参数 如 id=1&name=xx
     * @param url
     * @param params
     * @return
     */
    private static  Map get(String url, String params){
        HttpURLConnection connection = null;
        BufferedReader br = null;
        Map<String,Object> res = new HashMap<>();
        StringBuffer sb = new StringBuffer();
        Map<String,List<String>> headers;

        try{
            URL send_url = params.equals("") ? new URL(url) : new URL(url + "?" + params);
            connection = (HttpURLConnection) send_url.openConnection();
            connection.setRequestMethod("GET");
            //一些设置
            connection.setRequestProperty("Connection","keep-alive");
            connection.setRequestProperty("Accept-Charset","utf8, gbk; q=0.6");
            //content-type浏览器会自动解析
            //connection.setRequestProperty("Content-Type","application/json");
            //connection.setDoOutput(true);
            connection.setUseCaches(false);

            connection.connect();
            headers = connection.getHeaderFields();

            br = new BufferedReader(new InputStreamReader(connection.getInputStream(),"UTF-8"));
            String line = "";
            while ((line = br.readLine()) != null){
                sb.append(line);
            }
            res.put("headers",headers);
            res.put("response", sb);
        }catch (Exception e){
            System.out.println("http_get 异常！");
            e.printStackTrace();
        }finally {
            try {
                if(connection != null){
                    connection.disconnect();
                }
                if(br != null){
                    br.close();
                }
            }catch (Exception e2){

            }
        }
        return res;
    }

    /**
     * params为postc参数
     * @param url
     * @param params
     */
    private static  Map post(String url, String params){
        HttpURLConnection connection = null;
        BufferedReader br = null;
        OutputStreamWriter out = null;
        Map<String,Object> res = new HashMap<>();
        StringBuffer sb = new StringBuffer();
        Map<String,List<String>> headers;

        try {
            URL send_url = new URL(url);
            connection = (HttpURLConnection)send_url.openConnection();

            connection.setRequestMethod("POST");
            connection.setRequestProperty("Content-Type", "application/json");
            connection.setRequestProperty("Connection", "Keep-Alive");
            connection.setUseCaches(false);
            //以后就可以使用conn.getOutputStream().write()
            connection.setDoOutput(true);
            //以后就可以使用conn.getInputStream().read();
            connection.setDoInput(true);

            out = new OutputStreamWriter(connection.getOutputStream(),"UTF-8");
            out.write(params);
            out.flush();

            headers = connection.getHeaderFields();
            br = new BufferedReader(new InputStreamReader(connection.getInputStream()));
            String line;
            while ((line = br.readLine()) != null){
                sb.append(line);
            }

            res.put("headers",headers);
            res.put("response",sb.toString());
        }catch (Exception e1){
            e1.printStackTrace();
        }finally {
            try {
                if(connection != null){
                    connection.disconnect();
                }
                if(br != null){
                    br.close();
                }
                if(out != null){
                    out.close();
                }
            }catch (Exception e2){
                e2.printStackTrace();
            }
        }
        return res;
    }

    private void xxxx(){

    }

    public static void main(String args[]){
//        Map<String,Object> res = get("http://127.0.0.1:8080/test1","");
//        System.out.println(res.get("response"));
//        System.out.println(res.get("headers"));
//        Map<String,List<String>> headers  = (Map<String, List<String>>)res.get("headers");
//        System.out.println(headers.get("Content-Type"));

        String params = "{\n" +
                "    \"status\": 0,\n" +
                "    \"data\": \"hello world!\"\n" +
                "}";
        Map<String,Object> res = post("http://127.0.0.1:8080/test2",params);
        System.out.println(res.get("response"));


    }
}

```
