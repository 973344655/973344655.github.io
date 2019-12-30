---
title: spring参数验证注解
date: 2019-12-12 14:51:44
tags: [spring]
---

# 一.常用注解


JSR提供的校验注解：  

- @Null   被注释的元素必须为 null    
- @NotNull    被注释的元素必须不为 null     
- @Min(value)     被注释的元素必须是一个数字，其值必须大于等于指定的最小值    
- @Max(value)     被注释的元素必须是一个数字，其值必须小于等于指定的最大值      
- @Size(max=, min=)   被注释的元素的大小必须在指定的范围内      
- @Pattern(regex=,flag=)  被注释的元素必须符合指定的正则表达式    


Hibernate Validator提供的校验注解：

- @NotBlank(message =)   验证字符串非null，且长度必须大于0    
- @Email  被注释的元素必须是电子邮箱地址    
- @Length(min=,max=)  被注释的字符串的大小必须在指定的范围内    
- @NotEmpty   被注释的字符串的必须非空    
- @Range(min=,max=,message=)  被注释的元素必须在合适的范围内


# 二.例子

```
@Data
public class User {

    private String userId;
    private String username;
    @NotBlank(message = "密码不能为空")
    @Length(min = 6,max = 18,message = "密码长度为6-18")
    private String password;
    private String userDesc;
    private String avatarUrl;
    @Pattern(regexp = "^1(3|4|5|7|8)\\d{9}$",message = "手机号码格式错误")
    @NotBlank(message = "手机号码不能为空")
    private String mobile;
    @Email(message = "请输入正常的邮箱")
    private String email;
    private String cityCode;
    private String areaCode;
    private String userAddress;
    private String zoneNum;
    private String connectPhoneNum;
    private String userBak1;
    private String userBak2;
    private String userBak3;
    private String userStatus;
    private String createTime;
    private String updateTime;

}
```

# 三.使用

使用时要加上 @Valid 或 @Validated 注解

```
@ApiOperation(value = "注册",notes = "注册")
 @RequestMapping(value = "register", method = RequestMethod.POST)
 @ResponseBody
 public Object register(@Valid @RequestBody User user){

     return userService.register(user);
 }

```
