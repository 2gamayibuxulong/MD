# SQL

### 通配符

| %                          | 替代一个或多个字符         |
| -------------------------- | -------------------------- |
| _                          | 仅替代一个字符             |
| [charlist]                 | 字符列中的任何单一字符     |
| [^charlist]或者[!charlist] | 不在字符列中的任何单一字符 |

### Inner join

在表中存在至少一个匹配时，INNER JOIN 关键字返回行。



### LEFT JOIN

LEFT JOIN 关键字会从左表 (table_name1) 那里返回所有的行，即使在右表 (table_name2) 中没有匹配的行。



### RIGHT JOIN

RIGHT JOIN 关键字会右表 (table_name2) 那里返回所有的行，即使在左表 (table_name1) 中没有匹配的行。



### SQL FULL JOIN 

只要其中某个表存在匹配，FULL JOIN 关键字就会返回行。



### SQL UNION 操作符

UNION 操作符用于合并两个或多个 SELECT 语句的结果集。

请注意，UNION 内部的 SELECT 语句必须拥有相同数量的列。列也必须拥有相似的数据类型。同时，每条 SELECT 语句中的列的顺序必须相同。



### UNION ALL

UNION ALL 命令和 UNION 命令几乎是等效的，不过 UNION ALL 命令会列出所有的值。



### CREATE TABLE 语法

```java
CREATE TABLE 表名称
(
列名称1 数据类型,
列名称2 数据类型,
列名称3 数据类型,
....
)
```



### Constraints

| 约束        | 写法                                        |
| ----------- | ------------------------------------------- |
| NOT NULL    | LastName varchar(255) NOT NULL,             |
|             | Id_P int NOT NULL,                          |
| UNIQUE      | UNIQUE (Id_P)                               |
| PRIMARY KEY | PRIMARY KEY (Id_P)                          |
| FOREIGN KEY | FOREIGN KEY (Id_P) REFERENCES Persons(Id_P) |
| CHECK       | CHECK (Id_P>0)                              |
| DEFAULT     | City varchar(255) DEFAULT 'Sandnes'         |



### SQL Server Date 函数

| 函数                                                         | 描述                             |
| :----------------------------------------------------------- | :------------------------------- |
| [GETDATE()](http://www.w3school.com.cn/sql/func_getdate.asp) | 返回当前日期和时间               |
| [DATEPART()](http://www.w3school.com.cn/sql/func_datepart.asp) | 返回日期/时间的单独部分          |
| [DATEADD()](http://www.w3school.com.cn/sql/func_dateadd.asp) | 在日期中添加或减去指定的时间间隔 |
| [DATEDIFF()](http://www.w3school.com.cn/sql/func_datediff.asp) | 返回两个日期之间的时间           |
| [CONVERT()](http://www.w3school.com.cn/sql/func_convert.asp) | 用不同的格式显示日期/时间        |



### MySQL Date 函数

| 函数                                                         | 描述                                |
| :----------------------------------------------------------- | :---------------------------------- |
| [NOW()](http://www.w3school.com.cn/sql/func_now.asp)         | 返回当前的日期和时间                |
| [CURDATE()](http://www.w3school.com.cn/sql/func_curdate.asp) | 返回当前的日期                      |
| [CURTIME()](http://www.w3school.com.cn/sql/func_curtime.asp) | 返回当前的时间                      |
| [DATE()](http://www.w3school.com.cn/sql/func_date.asp)       | 提取日期或日期/时间表达式的日期部分 |
| [EXTRACT()](http://www.w3school.com.cn/sql/func_extract.asp) | 返回日期/时间按的单独部分           |
| [DATE_ADD()](http://www.w3school.com.cn/sql/func_date_add.asp) | 给日期添加指定的时间间隔            |
| [DATE_SUB()](http://www.w3school.com.cn/sql/func_date_sub.asp) | 从日期减去指定的时间间隔            |
| [DATEDIFF()](http://www.w3school.com.cn/sql/func_datediff_mysql.asp) | 返回两个日期之间的天数              |
| [DATE_FORMAT()](http://www.w3school.com.cn/sql/func_date_format.asp) | 用不同的格式显示日期/时间           |











## SQL Server 数据类型

### Character 字符串：

| 数据类型     | 描述                                          | 存储 |
| :----------- | :-------------------------------------------- | :--- |
| char(n)      | 固定长度的字符串。最多 8,000 个字符。         | n    |
| varchar(n)   | 可变长度的字符串。最多 8,000 个字符。         |      |
| varchar(max) | 可变长度的字符串。最多 1,073,741,824 个字符。 |      |
| text         | 可变长度的字符串。最多 2GB 字符数据。         |      |

### Unicode 字符串：

| 数据类型      | 描述                                               | 存储 |
| :------------ | :------------------------------------------------- | :--- |
| nchar(n)      | 固定长度的 Unicode 数据。最多 4,000 个字符。       |      |
| nvarchar(n)   | 可变长度的 Unicode 数据。最多 4,000 个字符。       |      |
| nvarchar(max) | 可变长度的 Unicode 数据。最多 536,870,912 个字符。 |      |
| ntext         | 可变长度的 Unicode 数据。最多 2GB 字符数据。       |      |

### Binary 类型：

| 数据类型       | 描述                                    | 存储 |
| :------------- | :-------------------------------------- | :--- |
| bit            | 允许 0、1 或 NULL                       |      |
| binary(n)      | 固定长度的二进制数据。最多 8,000 字节。 |      |
| varbinary(n)   | 可变长度的二进制数据。最多 8,000 字节。 |      |
| varbinary(max) | 可变长度的二进制数据。最多 2GB 字节。   |      |
| image          | 可变长度的二进制数据。最多 2GB。        |      |

### Number 类型：

| 数据类型     | 描述                                                         | 存储        |
| :----------- | :----------------------------------------------------------- | :---------- |
| tinyint      | 允许从 0 到 255 的所有数字。                                 | 1 字节      |
| smallint     | 允许从 -32,768 到 32,767 的所有数字。                        | 2 字节      |
| int          | 允许从 -2,147,483,648 到 2,147,483,647 的所有数字。          | 4 字节      |
| bigint       | 允许介于 -9,223,372,036,854,775,808 和 9,223,372,036,854,775,807 之间的所有数字。 | 8 字节      |
| decimal(p,s) | 固定精度和比例的数字。允许从 -10^38 +1 到 10^38 -1 之间的数字。p 参数指示可以存储的最大位数（小数点左侧和右侧）。p 必须是 1 到 38 之间的值。默认是 18。s 参数指示小数点右侧存储的最大位数。s 必须是 0 到 p 之间的值。默认是 0。 | 5-17 字节   |
| numeric(p,s) | 固定精度和比例的数字。允许从 -10^38 +1 到 10^38 -1 之间的数字。p 参数指示可以存储的最大位数（小数点左侧和右侧）。p 必须是 1 到 38 之间的值。默认是 18。s 参数指示小数点右侧存储的最大位数。s 必须是 0 到 p 之间的值。默认是 0。 | 5-17 字节   |
| smallmoney   | 介于 -214,748.3648 和 214,748.3647 之间的货币数据。          | 4 字节      |
| money        | 介于 -922,337,203,685,477.5808 和 922,337,203,685,477.5807 之间的货币数据。 | 8 字节      |
| float(n)     | 从 -1.79E + 308 到 1.79E + 308 的浮动精度数字数据。 参数 n 指示该字段保存 4 字节还是 8 字节。float(24) 保存 4 字节，而 float(53) 保存 8 字节。n 的默认值是 53。 | 4 或 8 字节 |
| real         | 从 -3.40E + 38 到 3.40E + 38 的浮动精度数字数据。            | 4 字节      |

### Date 类型：

| 数据类型       | 描述                                                         | 存储       |
| :------------- | :----------------------------------------------------------- | :--------- |
| datetime       | 从 1753 年 1 月 1 日 到 9999 年 12 月 31 日，精度为 3.33 毫秒。 | 8 bytes    |
| datetime2      | 从 1753 年 1 月 1 日 到 9999 年 12 月 31 日，精度为 100 纳秒。 | 6-8 bytes  |
| smalldatetime  | 从 1900 年 1 月 1 日 到 2079 年 6 月 6 日，精度为 1 分钟。   | 4 bytes    |
| date           | 仅存储日期。从 0001 年 1 月 1 日 到 9999 年 12 月 31 日。    | 3 bytes    |
| time           | 仅存储时间。精度为 100 纳秒。                                | 3-5 bytes  |
| datetimeoffset | 与 datetime2 相同，外加时区偏移。                            | 8-10 bytes |
| timestamp      | 存储唯一的数字，每当创建或修改某行时，该数字会更新。timestamp 基于内部时钟，不对应真实时间。每个表只能有一个 timestamp 变量。 |            |

### 其他数据类型：

| 数据类型         | 描述                                                         |
| :--------------- | :----------------------------------------------------------- |
| sql_variant      | 存储最多 8,000 字节不同数据类型的数据，除了 text、ntext 以及 timestamp。 |
| uniqueidentifier | 存储全局标识符 (GUID)。                                      |
| xml              | 存储 XML 格式化数据。最多 2GB。                              |
| cursor           | 存储对用于数据库操作的指针的引用。                           |
| table            | 存储结果集，供稍后处理。                                     |



### HAVING 子句

在 SQL 中增加 HAVING 子句原因是，WHERE 关键字无法与合计函数一起使用。

```
SELECT column_name, aggregate_function(column_name)
FROM table_name
WHERE column_name operator value
GROUP BY column_name
HAVING aggregate_function(column_name) operator value
```

