---
title: httpclient
date: 2019-01-15 16:21:54
tags: [wheels,java]
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
     public static Map get(String url, Map<String,String> params, Map<String,String>... otherHeader){
         HttpURLConnection connection = null;
         BufferedReader br = null;
         Map<String,Object> res = new HashMap<>();
         StringBuffer sb = new StringBuffer();
         Map<String,List<String>> headers;
         StringBuffer param = new StringBuffer();

         try{
             if(params != null){
                 int i = 1;
                 for(Map.Entry<String,String> entry: params.entrySet()){
                     param.append(entry.getKey()).append("=").append(entry.getValue());
                     if (i < params.size()){
                         i++;
                         param.append("&");
                     }
                 }
             }
             URL send_url = param.toString().equals("")  ? new URL(url) : new URL(url + "?" + param.toString());
             connection = (HttpURLConnection) send_url.openConnection();
             connection.setRequestMethod("GET");
             //一些设置
             connection.setRequestProperty("Connection","keep-alive");
             connection.setRequestProperty("Accept-Charset","utf8, gbk; q=0.6");
             //content-type浏览器会自动解析
             //connection.setRequestProperty("Content-Type","application/json");
             //connection.setDoOutput(true);
             //设置header
             if(otherHeader != null && otherHeader.length > 0){
                 for(Map.Entry<String,String> entry: otherHeader[0].entrySet()){
                     connection.setRequestProperty(entry.getKey(),entry.getValue());
                 }
             }
             connection.setUseCaches(false);
             connection.connect();

             headers = connection.getHeaderFields();

             br = new BufferedReader(new InputStreamReader(connection.getInputStream(),"UTF-8"));
             String line = "";
             while ((line = br.readLine()) != null){
                 sb.append(line);
             }
             res.put("headers",headers);
             res.put("response", sb.toString());
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
      * params为post参数
      * @param url
      * @param params
      */
     public  static  Map post(String url, String params, Map<String,String>... otherHeader){
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
             //其它header
             if(otherHeader != null && otherHeader.length > 0){
                 for(Map.Entry<String,String> entry: otherHeader[0].entrySet()){
                     connection.setRequestProperty(entry.getKey(),entry.getValue());
                 }
             }
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


     /**
      *  使用RestTemplate发送formdaata数据
      * @return
      */
     public static String postForm(String url,  Map<String,String> params, Map<String,String>... otherHeader){
         try{
             RestTemplate restTemplate = new RestTemplate();
             HttpHeaders headers = new HttpHeaders();
             headers.setContentType(MediaType.APPLICATION_FORM_URLENCODED);
             //设置header
             if(otherHeader != null && otherHeader.length > 0){
                 for(Map.Entry<String,String> entry: otherHeader[0].entrySet()){
                     headers.set(entry.getKey(),entry.getValue());
                 }
             }

             MultiValueMap<String, String> map = new LinkedMultiValueMap<>();


             if(params != null){
                 for(Map.Entry<String,String> entry : params.entrySet()){
                     map.add(entry.getKey(),entry.getValue());
                 }
             }
             HttpEntity<MultiValueMap<String, String>> request = new HttpEntity<>(map, headers);
             ResponseEntity<String> response = restTemplate.postForEntity( url, request , String.class );
             System.out.println(response.getBody());
             return response.getBody();
         }catch (Exception e){
             e.printStackTrace();
         }
         return "";

     }

}

```
