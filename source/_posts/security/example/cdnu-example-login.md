---
title: "一个渗透小例子"
date: 2018-12-18 15:54:01
tags: [security]
---

看了好久的渗透概念性东西，都没有实践过，今天想试一下，随意找了一个学校官网(自知菜鸟水平，还没入门，不敢找太好的学校)。
![indexPage](./index_page.png)

#### 1.找到薄弱点
官网一般相对安全一点，所以通常从子域名下手。<br>
chrome搜索，输入: inurl:cdnu.edu.cn

![searchPage](./search_page.png)

看到一个身份认证平台<br>
一般这些都是后台登录界面<br>
点进去
![searchPage](./login_page.png)

可以看到，不仅没有验证码，还清楚的告诉你了账号为学号，默认密码为身份证后六位。典型的用于给我们进行爆破。

#### 2.信息收集
为了能进行爆破，先要得到学号。<br>
直接google和百度.
![searchPage](./studentid_search_01.png)

![searchPage](./studentid_search_02.png)

![searchPage](./studentid_search_03.png)

可以看到，在搜索一些国内的网站信息时，百度还是要好用一点。<br>

尝试登录：160003021029/123456<br>
返回:用户名或密码错误。到这还看不出什么<br>
再尝试输入: 160000000000/123456<br>
返回：用户名不存在。(由此可知160003021029用户是存在的，同理，可以知道160003021030等等)<br>

在这一步，由于网站的 <strong>不恰当返回信息处理</strong> (用户名枚举)，我们得到了可用用户名.<br>

接下来该进行爆破了，先生成身份证后六位的密码字典.<br>
因为从学号可看出是2016届的，所以推测生日应该在1997-1999年，这样字典就只有三万条.
```
import os

# ##########################
# 生成身份证后六位的密码字典
# ##########################
year_start = input("请输入开始年份(四位): ")
year_end = input("请输入结束年份(四位):")

if len(year_start) != 4 or len(year_end) != 4:
    print("输入格式有误")
    exit(-1)
else:
    year_count = abs(int(year_end) - int(year_start)) + 1
    file_path = os.getcwd() + "/" + year_start + "-" + year_end + "dict.txt"
    f = open(file_path, "a+")
    for index in range(0, year_count):
        res = []
        if year_start < year_end:
            list_temp = []
            for i in range(0, 10000):
                list_temp.append(str((int(year_start) + index))[-2:] + str(i).zfill(4))
                f.write(str((int(year_start) + index))[-2:] + str(i).zfill(4))
                f.write("\n")
            res.append(list_temp)
        else:
            for i in range(0, 9999):
                list_temp.append(str((int(year_end) + index))[-2:] + str(i).zfill(4))
                f.write(str((int(year_end) + index))[-2:] + str(i).zfill(4))
                f.write("\n")
            res.append(list_temp)
    f.close()
```
#### 3.进行尝试
使用burpsuite抓包，然后进行爆破<br>
没有成功<br>
换弱密码top100,再次尝试。<br>

![searchPage](./password_success.png)

这次可以看到密码：12345不一样，进行手动登录尝试(160003021029/12345).

![searchPage](./password_modify.png)

出现此界面，修改密码。

![searchPage](./login_success.png)

nice,登录成功。<br>

注销登录，换(160003021030/12345)再次尝试<br>
发现再次出现修改密码界面。<br>
160003021030/123 再次尝试，并没此问题.<br>


由此惊讶的猜测测，12345并不是真是密码，而是由于逻辑错误造成的漏洞利用<br>


右击登录按钮，检查元素，查看源代码.<br>
![searchPage](./code_logic.png)

![searchPage](./code_logic_02.png)

可以看到，因为触发了该界面，可以直接更改密码，且不需要输入密码<br>
到此，因为没有挂代理，不敢进行下一步操作。

#### 4.事后分析
- 网站登录没有验证码，给了爆破利用空间。<br>
- 网站登录返回信息处理失误，应统一返回用户或密码出错。<br>
- 网站逻辑处理有漏洞，如更改密码不需要旧密码。<br>
- 网站敏感信息能被看到<br>
