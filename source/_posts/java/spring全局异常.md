---
title: spring利用异常来优化代码
date: 2019-08-01 10:52:55
tags: [java,spring]
---

# 一.异常

盗图：https://marchnineteen.github.io/2018/08/08/java/javase/exception/
![io](http://67.216.218.49:8000/file/blogs/java/base/exception/异常结构体系.png)

异常的体系结构为：

  - Throwable

  所有异常的基类， 分为 Error 和 Exception

  - Error

  错误，一般会导致Jvm出错。

  - Exception

  分为 运行时异常(RuntimeException) 和 非运行时异常.<br>
  也叫 未检查的异常  和  检查的异常


Java compiler要求所有的Exception 要么被catch,要么被throw，除非这是一个RuntimeExeption (e instanceof RuntimeException)。也就是说，通常的Exception一定要被处理，也即我们所说的 checked exception，而RuntimeException不强制要求处理，（当然你自己要处理也可以），所以我们称为unchecked exception

# 二.自定义异常

## 1.例子

```
/**
 * 标记文件处理流程中得错误
 */
public class MyFileException extends RuntimeException {
    private String errMsg;
    private Throwable throwable;


    public MyFileException() {
        this("文件相关操作发生异常");
    }


    public MyFileException(String errMsg) {
        this.errMsg = errMsg;
    }

    public MyFileException(Throwable throwable) {
        this.throwable = throwable;
    }
    public MyFileException(Throwable throwable, String msg) {
        this.throwable = throwable;
        this.errMsg = msg;
    }

    public String getErrMsg() {
        if(this.errMsg == null || this.errMsg.equals("")){
            return this.throwable.getMessage();
        }
        if(this.throwable == null){
            return this.errMsg;
        }
        return this.throwable.getMessage() + " -- " + this.errMsg;
    }
}
```
## 2.继承选择

通常我们的自定义异常都继承于 Exception 或 RuntimeException.<br>

两者的区别是:<br>

 继承于 Exception 的  需要我们对异常进行处理，否则编译会不通过。利用这一点，可以强制的告诉别人，这里会发生一个异常，你必须注意到，并对它进行处理.<br>

 继承于 RuntimeException 的， 为 unchecked Exception, 不要求强制处理(throw 时 不需要在方法上 throws)。实际上，这种类型的异常，通常应该交给上级(调用你的程序来进行处理).<br>


选择使用那种方式，完全按照我们的需求与业务逻辑来.<br>

# 三.Spring 全局异常

## 1.@ExceptionHandler

```
@Target({ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface ExceptionHandler {
    Class<? extends Throwable>[] value() default {};
}
```

作用域为 method<br>

如果@ExceptionHandler方法是在 @Controller 内部定义的，那么它会接收并处理由 当前 Controller（或其任何子类）中的@RequestMapping方法抛出的异常。如果你将 @ExceptionHandler 方法定义在 @ControllerAdvice 类中，那么它会处理相关控制器中抛出的异常.<br>
此外,@ExceptionHandler注解还可以接受一个异常类型的数组作为参数值。若抛出了已在列表中声明的异常，那么相应的@ExceptionHandler方法将会被调用。如果没有给注解任何参数值，那么默认处理的异常类型将是方法参数所声明的那些异常。

## 2.@ControllerAdvice

```
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Component
public @interface ControllerAdvice {
    @AliasFor("basePackages")
    String[] value() default {};

    @AliasFor("value")
    String[] basePackages() default {};

    Class<?>[] basePackageClasses() default {};

    Class<?>[] assignableTypes() default {};

    Class<? extends Annotation>[] annotations() default {};
}
```
作用域为 type,包括类、接口和枚举等<br>

通过 @ControllerAdvice 注解可以将对于控制器(@Controller)的全局配置放在同一个位置<br>

注意：  @ControllerAdvice 只能捕捉到Controller层(异常会向上传递，抛到调用层)抛出的异常

## 3.全局异常

由1，2已知，@ExceptionHandler 异常拦截只拦截当前 Controller 里的 @RequestMapping方法抛出的异常, 同时 @ControllerAdvice 可以提供对 Controller 的全局配置。 所以，将这两个注解结合，可以提供对 Controller 的全局异常拦截处理.<br>


例子：
```
/**
 * 全局异常处理类，拦截@RequestMapping抛出的异常
 * 处理顺序为配置顺序 依次向下
 * @author xxl
 */
@ControllerAdvice
class GlobalExceptionHandler {

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
        return  ResultMessage.error(StatusCode.PARAMS_VALID_EXCEPTION,"参数非空校验异常");
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
        return  ResultMessage.error(StatusCode.EXCEPTION_DATABASE,e.getErrMsg());
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
