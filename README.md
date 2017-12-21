![SQLInject](SQLInject.jpg)

## 代码审计之SQL注入
代码审计主要是通过源码（主要是`Java/PHP/JavaScript`）挖掘Web漏洞，然后构造HTTP请求来验证安全风险......

### 漏洞介绍
SQL注入漏洞是Web常见的漏洞，通过此漏洞可能用来拖库、获取Web管理员账号和密码、权限提升等攻击，是Web安全中最严重的漏洞之一。

**注意：本章节仅讨论`关系型`数据库，不适用`非关系型`数据库。**

### 漏洞原理

SQL注入(SQL Injection)，就是通过把SQL命令插入到`Web表单提交`或`页面请求`的字符串中，最终达到欺骗数据库服务器执行恶意的SQL命令,从而达到攻击的目的。
...

后台验证用户名和密码是否正确的java代码关键如下：
```java
uName = getRequestString("UserName");
uPass = getRequestString("UserPass");
sql = "SELECT * FROM Users WHERE Name ='" + uName + "' AND Pass ='" + uPass + "'"
```
详见：[SQL_Injection](https://websec.ca/kb/sql_injection)

### 测试思路

**PHP:可以使用mysqli、PDO实现预编译**

**Java:可以使用preparestatement预编译**

因此针对SQL注入，强烈推荐使用本文【Web源码分析SQL注入】为主，黑盒测试为辅。源码分析遵从以下规则：

+ 未使用预编译的，必须换为预编译，如果存在不能预编的场景，需要单独check；
+ 使用预编译的，必须使用占位符，不能针对sql拼接。
+ ...

#### 黑盒测试
在测试前，通过与开交流或者源码源码分析，了解Web中使用的所有数据库名称及版本号，包括关系型数据库（MySQL、Oracle、SQLite等）,然后针对性测试，减少测试工作量。黑盒测试思路如下：
>1）Web漏洞扫描
>利用Web漏洞扫描工具如AppScan、AWVS、Burp自动扫描，可以发现部分注入SQL注入漏洞，针对发现的漏洞“点”，在源码分析漏洞产生的原因，然后全面排查类似问题。
>
>2）手工SQL注入
>在Web界面中找到可能存在数据库操作的点，例如用户用户、数据查询、添加用户等场景，然后尝试手工测试，来发现比较隐蔽，以及工具难以覆盖的SQL注入漏洞。

#### Web源码分析
通过分析源码，识别可能被绕过的SQL注入场景，例如绕过正则、关键字过滤、编码等，常见的SQL注入根因如下：

+ 直接拼接SQL语句，导致存在SQL注入；
+ 不正确使用预编译导致存在SQL注入；
+ 通过拼接，但过滤缺陷导致SQL注入；
+ 第三方框架导致SQL注入（如Hibernate、iBatis……）
+ 其它原因导致存在的SQL注入。

### 测试方法
Web漏洞扫描在`【测试工具】`中有详细介绍，因此本章节测试方法重点介绍手工SQL注入和Web源码分析SQL注入。

#### 源码：非预编译大致的SQL注入
**PHP关键字：query 、mysql_query 、mysql_fetch_array ……**

**Java关键字：Statement 、.execute 、.executeQuery、jdbcTemplate、queryForInt、queryForObject、queryForMap、getConnection**

***【技巧：SQL002】***

*产品安全测试过程中测试量大，参数多，因此最高速有效的测试方法就是基于正常报文添加单双引号，然后查看数据库日志。当然如果你非常熟悉手工注入也可以直接构造出SQL来证明是存在漏洞的。*
