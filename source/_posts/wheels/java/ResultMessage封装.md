---
title: 返回消息
date: 2019-3-12 16:40:31
tags: [wheels]
---
### 1.状态
```
public class StatusCode {
    private int status;
    private String message;

    public static StatusCode SUCCESS = new StatusCode(200,"SUCCESS");

    public static StatusCode INVALID_REQUEST = new StatusCode(400,"请求有误");
    public static StatusCode UNAUTHORIZED_ERROR = new StatusCode(401,"权限错误");
    public static StatusCode FORBIDDEN  = new StatusCode(403,"禁止访问");
    public static StatusCode NOT_FOUND  = new StatusCode(404,"找不到对应地址");
    public static StatusCode NOT_ACCEPTABLE = new StatusCode(406,"请求参数格式不对,json/xml?");
    public static StatusCode PARAMS_VALID_EXCEPTION = new StatusCode(422,"参数校验异常，未通过");
    public static StatusCode ALEADY_EXIST= new StatusCode(423,"该条数据已存在");
    public static StatusCode USER_INFO_ERROR= new StatusCode(424,"用户信息不匹配");
    public static StatusCode SERVER_EXCEPTION = new StatusCode(500,"服务端处理发生错误");
    public static StatusCode UNEXPECTED_FINISHED = new StatusCode(511,"操作未正常完成");
    public static StatusCode AUTHENTICATION_EXCEPTION = new StatusCode(512,"认证错误");
    public static StatusCode EXPIRED_EXCEPTION = new StatusCode(513,"过期");


    //其它问题 待定
    public static StatusCode EXCEPTION_DATABASE = new StatusCode("数据库操作发生异常");
    public static StatusCode EXCEPTION_DEAL = new StatusCode("处理流程中发生异常");
    public static StatusCode EXCEPTION_HTTP= new StatusCode("HTTP调用异常");


    public static StatusCode FILETYPE_ERROR = new StatusCode(2212, "文件类型错误");
    public static StatusCode FILESIZE_EXCESSIVE = new StatusCode(2213, "文件过大");
    public static StatusCode UPLOAD_FAILED = new StatusCode(2214, "文件上传失败");

    public static StatusCode SAVE_ERROR=new StatusCode(2215,"保存错误");
    public static StatusCode LOGIN_FAILED=new StatusCode(2216,"登录失败");


    private StatusCode(String message){
        this.message = message;
    }
    private StatusCode(int status, String message) {
        this.status = status;
        this.message = message;
    }

    public int getStatus() {
        return this.status;
    }
    public String getMessage() {
        return message;
    }
    //用于提示消息拓展
    public void setMessage(String message) {
        this.message = message;
    }


}


```
### 2.返回
```
public class ResultMessage<T>{
    //提示信息
    private String message;
    //状态
    private int status;
    //返回消息内容
    private T data;

    private ResultMessage(T data) {
        this.status = 200;
        this.message = "SUCCESS";
        this.data = data;
    }
    private ResultMessage(StatusCode statusCode){
        this.status = statusCode.getStatus();
        this.message = statusCode.getMessage();
    }

    /**
     * 成功时候的调用
     * @return
     */
    public static <T> ResultMessage<T> success(T data){
        return new ResultMessage<>(data);
    }


    /**
     * 成功，不需要传入参数
     * @return
     */
    @SuppressWarnings("unchecked")
    public static <T> ResultMessage<T> success(){
        return (ResultMessage<T>) success("");
    }
    /**
     * 失败时候的调用
     * @return
     */
    public static <T> ResultMessage<T> error(StatusCode statusCode){
        return new ResultMessage<T>(statusCode);
    }
    /**
     * 失败时候的调用,扩展消息参数
     * @param statusCode
     * @param msg
     * @return
     */
    public static <T> ResultMessage<T> error(StatusCode statusCode,String msg){
        statusCode.setMessage(statusCode.getMessage().split("--")[0] + "--" + msg);
        return new ResultMessage<>(statusCode);
    }

    public T getData() {
        return data;
    }
    public String getMessage() {
        return message;
    }
    public int getStatus() {
        return this.status;
    }

    @Override
    public String toString(){
        return "status: " + this.status + " | message: " + this.message + " | data: " + this.data;
    }
}


```
