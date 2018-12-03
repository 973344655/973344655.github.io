---
title: springbootdemo
date: 2018-11-23 15:47:52
tags: [springboot]
---
### springboot架子搭建

#### 1.日志

##### （1）log4j2
- 依赖
```
       <!--引入log4j2-->
       <dependency>
           <groupId>org.springframework.boot</groupId>
           <artifactId>spring-boot-starter-log4j2</artifactId>
       </dependency>
       <!--log4j2依赖于disruptor-->
       <dependency>
           <groupId>com.lmax</groupId>
           <artifactId>disruptor</artifactId>
           <version>3.4.2</version>
       </dependency>
```
- 配置文件log4j2.xml
log4j2默认会在classpath目录下寻找log4j.json、log4j.jsn、log4j2.xml等名称的文件，如果都没有找到，则会按默认配置输出，也就是输出到控制台
```
<?xml version="1.0" encoding="UTF-8"?>
<!-- Configuration后面的status，这个用于设置log4j2自身内部的信息输出，可以不设置，当设置成trace时，
    你会看到log4j2内部各种详细输出。可以设置成OFF(关闭)或Error(只输出错误信息)
  -->
<Configuration status="INFO">
    <!-- 日志文件目录和压缩文件目录配置 -->
    <Properties>
        <Property name="fileName">C:\\Users\\xiong\\Desktop\\log\\log4j2</Property>
        <Property name="fileGz">C:\\Users\\xiong\\Desktop\\log\\log4j2\\7z</Property>
    </Properties>
    <!--appender主要作用就是：①控制打印日志的地方、②打印日志的输出格式-->
    <Appenders>
        <!-- 输出控制台日志的配置 -->
        <Console name="console" target="SYSTEM_OUT">
            <!--控制台只输出level及以上级别的信息（onMatch），其他的直接拒绝（onMismatch）-->
            <ThresholdFilter level="info" onMatch="ACCEPT" onMismatch="DENY"/>
            <!-- 输出日志的格式 -->
            <PatternLayout pattern="%d{HH:mm:ss.SSS} [%t] %-5level %logger{36} - %msg%n"/>
        </Console>

        <!-- 打印出所有的信息，每次大小超过size，则这size大小的日志会自动存入按年份-月份建立的文件夹下面并进行压缩，作为存档 -->
        <RollingRandomAccessFile name="infoFile" fileName="${fileName}/log-info.log" immediateFlush="false"
                                 filePattern="${fileGz}/$${date:yyyy-MM}/app-%d{MM-dd-yyyy}-%i.log-info.gz">
            <PatternLayout pattern="%d{yyyy-MM-dd 'at' HH:mm:ss z} [%t] %-5level %logger{36} %L %M - %msg%xEx%n" />
            <Policies>
                <TimeBasedTriggeringPolicy interval="6" modulate="true" />
                <SizeBasedTriggeringPolicy size="100 MB"/>
            </Policies>
            <Filters>
                <!-- 只记录info和warn级别信息 -->
                <ThresholdFilter level="error" onMatch="DENY" onMismatch="NEUTRAL"/>
                <ThresholdFilter level="info" onMatch="ACCEPT" onMismatch="DENY" />
            </Filters>
            <!-- 指定每天的最大压缩包个数，默认7个，超过了会覆盖之前的 -->
            <DefaultRolloverStrategy max="50"/>
        </RollingRandomAccessFile>

        <!-- 存储所有error信息 -->
        <RollingRandomAccessFile name="errorFile" fileName="${fileName}/log-error.log" immediateFlush="false"
                                 filePattern="${fileGz}/$${date:yyyy-MM}/app-%d{MM-dd-yyyy}-%i.log-error.gz">
            <PatternLayout pattern="%d{yyyy-MM-dd 'at' HH:mm:ss z} [%t] %-5level %logger{36} %L %M - %msg%xEx%n" />
            <Policies>
                <TimeBasedTriggeringPolicy interval="6" modulate="true" />
                <SizeBasedTriggeringPolicy size="100 MB"/>
            </Policies>
            <Filters>
                <!-- 只记录error级别信息 -->
                <ThresholdFilter level="error" onMatch="ACCEPT" onMismatch="DENY" />
            </Filters>
            <!-- 指定每天的最大压缩包个数，默认7个，超过了会覆盖之前的 -->
            <DefaultRolloverStrategy max="50"/>
        </RollingRandomAccessFile>
    </Appenders>

    <!-- 全局配置，默认所有的Logger都继承此配置 -->
    <Loggers>
        <!-- AsyncRoot - 异步记录日志 - 需要LMAX Disruptor的支持 -->
        <AsyncRoot level="info" additivity="false">
            <AppenderRef ref="console"/>
            <AppenderRef ref="infoFile"/>
            <AppenderRef ref="errorFile"/>
        </AsyncRoot>
    </Loggers>
</Configuration>
```
- 使用
```
@Controller
public class TestController {
    Logger logger = LogManager.getLogger(TestController.class);

    @RequestMapping(value = "test", method = RequestMethod.GET)
    @ResponseBody
    public String test(){
        logger.debug("test debug");
        logger.info("test info");
        logger.warn("test warn");
        logger.error("test error");
        logger.fatal("test fatal");
        return "ok";
    }
}
```

#### 2.Restful API
使用swagger2
- 依赖
```
    <!-- Swagger依赖 注意版本一致-->
    <dependency>
			<groupId>io.springfox</groupId>
			<artifactId>springfox-swagger2</artifactId>
			<version>2.2.2</version>
		</dependency>
		<dependency>
			<groupId>io.springfox</groupId>
			<artifactId>springfox-swagger-ui</artifactId>
			<version>2.2.2</version>
		</dependency>
```
- 配置类
```
@Configuration
@EnableSwagger2
public class Swagger2 {
    /**
     * apis()设置监控路径
     * @return
     */
    @Bean
    public Docket createRestApi() {
        return new Docket(DocumentationType.SWAGGER_2)
                .apiInfo(apiInfo())
                .select()
                .apis(RequestHandlerSelectors.basePackage("com.example.springboottemplate.controllers"))
                .paths(PathSelectors.any())
                .build();
    }

    /**
     * 创建api基本信息，会展示在文档页面
     * @return
     */
    private ApiInfo apiInfo() {
        return new ApiInfoBuilder()
                .title("Swagger2 demo")
                .description("Spring Boot中使用Swagger2构建RESTful APIs")
                .version("1.0")
                .build();
    }
}
```
- 使用<br>
  <strong>注意：要使用@RequestParam()接收参数，需要paramType="query"</strong><br>
  具体参数信息:https://amphilicitezh.github.io/2017/12/04/swaggerui-introduce/

```
@Api(value = "测试接口类TestController")
@Controller
@ResponseBody
public class TestController {
    Logger logger = LogManager.getLogger(TestController.class);

    @ApiOperation(value = "测试get不带参数",notes = "测试get不带参数")
    @RequestMapping(value = "test1", method = RequestMethod.GET)
    public String test(){
        logger.debug("test debug");
        logger.info("test info");
        logger.warn("test warn");
        logger.error("test error");
        logger.fatal("test fatal");
        return "ok";
    }

    @ApiOperation(value = "测试get带参数",notes = "测试get带参数")
    @ApiImplicitParam(name = "username", value = "用户名", required = true, dataType = "String",paramType = "query")
    @RequestMapping(value = "test2", method = RequestMethod.GET)
    public String test2(@RequestParam("username") String username){
        if(!username.equals("") && username != null){
            System.out.println(username);
            return username;
        }else {
            return "null";
        }
    }

    @ApiOperation(value = "测试post带实体参数", notes = "测试post带实体参数")
    @ApiImplicitParam(name = "user",value = "用户名",required = true, dataType = "Map<String,Object>")
    @ApiResponses({
           @ApiResponse(code = 400, message = "请求参数没填好,注意为json"),
           @ApiResponse(code = 404, message = "请求路径没有或页面跳转路径不对"),
           @ApiResponse(code = 500, message = "服务器端发生错误")
   })
    @RequestMapping(value = "/test3", method = RequestMethod.POST)
    public String test3(@RequestBody Map<String,Object> user){
        System.out.println(user.get("username").toString());
        System.out.println(user.get("password").toString());
        return "ok";
    }

    @ApiOperation(value = "测试post参数为String", notes = "测试post参数为String")
    @ApiImplicitParams({
            @ApiImplicitParam(name = "username", value = "用户名", required = true, paramType = "query", dataType = "String"),
            @ApiImplicitParam(name = "password", value = "密码", required = true, paramType = "query", dataType = "String")
    })
    @RequestMapping(value = "/test4", method = RequestMethod.POST)
    public String test4(@RequestParam("username") String username,
                        @RequestParam("password") String passwprd){
       return username + ":" + passwprd;
    }

    @ApiOperation(value = "测试restful类型接口", notes = "测试restful类型接口")
    @RequestMapping(value = "/test5/{name}", method = RequestMethod.GET)
    @ApiImplicitParam(name = "username", value = "用户名", required = true, paramType = "query", dataType = "String")
    public String test5(@PathVariable("name") String name,
                        @RequestParam("username") String username){
        return name + " : " + username;
    }
}
```

访问:http://127.0.0.1:8080/swagger-ui.html

#### 3.Health
这是2.0版本：https://blog.csdn.net/alinyua/article/details/80009435<br>
可以直观的看到自己CPU的利用率、内存的利用率、数据库连接是否正常等信息
- 依赖
```
       <dependency>
           <groupId>org.springframework.boot</groupId>
           <artifactId>spring-boot-starter-actuator</artifactId>
           <!--<version>2.1.0.RELEASE</version>-->
       </dependency>
```
- Springboot2版本需要在application.properties加入
```
management.endpoints.web.exposure.include=*
#显示所有健康状态，需要加配置
management.endpoint.health.show-details=always
```
 内置EndPoints：
 >http://127.0.0.1:8080/actuator<br>
 http://127.0.0.1:8080/actuator/health<br>
 http://127.0.0.1:8080/actuator/info<br>
 http://127.0.0.1:8080/actuator/beans<br>
 http://127.0.0.1:8080/actuator/httptrace


 #### 4.Eureka
