---
title: 全局异常
date: 2019-12-12 14:46:19
tags: [wheels,java]
---
```

import com.bonc.zq.bui.configuration.myexception.*;
import org.springframework.http.converter.HttpMessageNotReadableException;
import org.springframework.validation.BindException;
import org.springframework.web.bind.MethodArgumentNotValidException;
import org.springframework.web.bind.annotation.ControllerAdvice;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.ResponseBody;
import org.springframework.web.multipart.MultipartException;

import java.net.ConnectException;

/**
 * 全局异常处理类，拦截@RequestMapping抛出的异常
 * 只能是controller层抛出的
 * 处理顺序为配置顺序 依次向下
 * @author xxl
 */
@ControllerAdvice
class GlobalExceptionHandler {

    /**
     *  拦截MyAuthException类的异常
     *  认证异常
     * @param e
     * @return
     */
    @ExceptionHandler(MyAuthException.class)
    @ResponseBody
    public ResultMessage exceptionHandler(MyAuthException e){
        e.printStackTrace();
        return  ResultMessage.error(StatusCode.EXCEPTION_DATABASE,e.getErrMsg());
    }
    /**
     *  拦截BindException类的异常 @Valid注解触发
     *  @NotBlank参数非空校验失败的异常
     * @param e
     * @return
     */
    @ExceptionHandler(BindException.class)
    @ResponseBody
    public ResultMessage exceptionHandler(BindException e){
        e.printStackTrace();
        return  ResultMessage.error(StatusCode.PARAMS_VALID_EXCEPTION,e.getMessage());
    }
    /**
     *  拦截MethodArgumentNotValidException类的异常 @Valid注解触发
     *  @NotBlank参数非空校验失败的异常
     * @param e
     * @return
     */
    @ExceptionHandler(MethodArgumentNotValidException.class)
    @ResponseBody
    public ResultMessage exceptionHandler(MethodArgumentNotValidException e){
        e.printStackTrace();
        return  ResultMessage.error(StatusCode.PARAMS_VALID_EXCEPTION,e.getMessage());
    }

    /**
     *  拦截MyDatabaseException类的异常
     *  数据库操作异常
     * @param e
     * @return
     */
    @ExceptionHandler(MyDatabaseException.class)
    @ResponseBody
    public ResultMessage exceptionHandler(MyDatabaseException e){
        e.printStackTrace();
        return  ResultMessage.error(StatusCode.EXCEPTION_DATABASE,e.getErrMsg());
    }

    /**
     *  拦截MyFileException类的异常
     *  文件操作异常
     * @param e
     * @return
     */
    @ExceptionHandler(MyFileException.class)
    @ResponseBody
    public ResultMessage exceptionHandler(MyFileException e){
        e.printStackTrace();
        return  ResultMessage.error(StatusCode.SERVER_EXCEPTION,e.getErrMsg());
    }

    /**
     *  拦截MyHttpException类的异常
     *  http操作异常
     * @param e
     * @return
     */
    @ExceptionHandler(MyHttpException.class)
    @ResponseBody
    public ResultMessage exceptionHandler(MyHttpException e){
        e.printStackTrace();
        return  ResultMessage.error(StatusCode.SERVER_EXCEPTION,e.getErrMsg());
    }

    /**
     * 拦截MyParamsExxception
     * 参数异常
     * @param e
     * @return
     */
    @ExceptionHandler(MyParamsException.class)
    @ResponseBody
    public ResultMessage exceptionHandler(MyParamsException e){
        e.printStackTrace();
        return  ResultMessage.error(StatusCode.EXCEPTION_DATABASE,e.getErrMsg());
    }

    /**
     * 文件上传错误
     * @param e
     * @return
     */
    @ExceptionHandler(MultipartException.class)
    @ResponseBody
    public ResultMessage exceptionHandler(MultipartException e){
        e.printStackTrace();
        return  ResultMessage.error(StatusCode.SERVER_EXCEPTION,"文件上传异常");
    }

    /**
     * 空指针异常拦截
     * @param e
     * @return
     */
    @ExceptionHandler(NullPointerException.class)
    @ResponseBody
    public ResultMessage exceptionHandler(NullPointerException e){
        e.printStackTrace();
        return ResultMessage.error(StatusCode.SERVER_EXCEPTION, "空指针异常");
    }

    /**
     * http异常拦截
     * @param e
     * @return
     */
    @ExceptionHandler(HttpMessageNotReadableException.class)
    @ResponseBody
    public ResultMessage exceptionHandler(HttpMessageNotReadableException e){
        e.printStackTrace();
        return ResultMessage.error(StatusCode.SERVER_EXCEPTION, "http参数异常");
    }

    /**
     * 连接异常拦截
     * @param e
     * @return
     */
    @ExceptionHandler(ConnectException.class)
    @ResponseBody
    public ResultMessage exceptionHandler(ConnectException e){
        e.printStackTrace();
        return ResultMessage.error(StatusCode.SERVER_EXCEPTION, "net连接异常");
    }

    /**
     * 其它异常,最后拦截，所有没匹配到得异常
     * @param e
     * @return
     */
    @ExceptionHandler(Exception.class)
    @ResponseBody
    public ResultMessage exceptionHandler(Exception e){
        e.printStackTrace();
        return  ResultMessage.error(StatusCode.SERVER_EXCEPTION,"发生异常--" + e.getMessage());
    }


}
```
