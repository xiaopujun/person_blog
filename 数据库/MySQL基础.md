# 1 查询数据

- 使用select语句时，select * 会降低查询效率
- distinct关键字不能部分使用，它应用于所有列而非前置他的列

# 2 多列排序的先后顺序

```sql
select age, id, name
from school.student
order by age, id;
```

# 3 sql语句关键字顺序

```html
[select] [from] [where] [order by] [limit]  
```

# 4 and 与 or的过滤顺序

```sql
select *
from school.student
where name = '鲁班'
   or age = 23 and id > 3;
```

上面sql中 age = 23 and id > 3 会作为首先过滤的条件，其次才是过滤 name = '鲁班'；and在计算次序中优先级最高

~~如上sql所示，整个sql按照age、id进行升序排序；当age中出现大小相同的数据时候，比如出现类3条记录的age相同，则这3条记录中再根据id的大小升序排序；若age全部都不同，则不会按照id排序

# 5 in 与 or

in的执行效率要高于or~~

# 6 group by规则

select 后的列必须都出现在group by后的子句中；反之无此要求；

```sql
select age, grade
from school.student
group by age;
```

如上示例；grade未出现在group by之后的子句中；因此会报错；原因在于，只对age分组后的数据，grade仍然可能有多行数据是不同的；在同一组中无法合并

# 7 联结查询

- 内联结（inner join）
- 自联结（联结表自己，取不同的别名）
- 外联结（left、right）

# MySQL数据类型的length含义

MySQL有几种数据类型可以限制类型的"长度"，有：

- CHAR(Length)
- VARCHAR(Length)
- TINYINT(Length)
- SMALLINT(Length)
- MEDIUMINT(Length)
- INT(Length)
- BIGINT(Length)
- FLOAT(Length, Decimals)
- DOUBLE(Length, Decimals)
- DECIMAL(Length, Decimals)

其中具体来讲：

1. CHAR、VARCAHR的长度是指字符的长度，例如CHAR[3]则只能放字符串"123"，如果插入数据"1234"，则从高位截取，变为"123"。 VARCAHR同理。
2. TINYINT、SMALLINT、MEDIUMINT、INT和BIGINT的长度，其实和数据的大小无关！Length指的是**显示宽度**
3. FLOAT、DOUBLE和DECIMAL的长度指的是全部数位（包括小数点后面的），例如DECIMAL(4,1)指的是**全部位数为4**，小数点后1位，如果插入1234，则查询的数据是999.9。

# 查询数据

![image-20200221230136876](C:\Users\xiaopu\AppData\Roaming\Typora\typora-user-images\image-20200221230136876.png)

表名为customer，以下示例基本参照此表

### 查询单列

```sql
SELECT `cust_id`
FROM customer
```

### 查询多列

```sql
SELECT `cust_id`, `cust_name`
FROM customer
```

### 查询所有列

```
SELECT * FROM customer
```

### 查询不重复记录

```
SELECT DISTINCT * FROM customer
```

### 限制记录条数

从头开始取，限定条数

```
SELECT  * FROM customer LIMIT 5
```

从指定索引开始取，限定条数

```sql
SELECT *
FROM customer LIMIT 5
OFFSET 3
```

意为从第三行开始取，一共取5行

### 使用完全限定的表名

```sql
SELECT customer.cust_name
FROM customer    
```

### 数据排序

按一个列排序

```
SELECT  customer.cust_name FROM customer ORDER BY cust_name
```

按多个列排序

```
SELECT  customer.cust_name,customer.cust_id FROM customer ORDER BY cust_name,cust_id
```

按指定方向排序（DESC是降序排序，默认不写是升序排序，要写的话为ASC）

```
SELECT  cust_name,cust_id FROM customer ORDER BY cust_id DESC
```

### 条件查询

```
select * from customer where cust_id = 1
```

where子句的可选项

| 操作符  | 说明       |
| ------- | ---------- |
| =       | 等于       |
| <>      | 不等       |
| !=      | 不等       |
| <       | 小于       |
| <=      | 小于等于   |
| >       | 大于       |
| > =      | 大于等于   |
| between | 在两者之间 |

### 范围查询

```sql
select *
from customer
where cust_id BETWEEN 1 and 5
```

```sql
select *
from customer
where cust_id IN (1, 2, 3)
```

### 空值查询

```sql
select *
from customer
where cust_phone is NULL
```

### 条件组合查询

同时满足

```sql
select *
from customer
where cust_phone is NULL
  AND cust_id > 3
```

直须满足

```sql
select *
from customer
where cust_phone is NULL
   OR cust_id > 3
```

优先处理的条件可以用（）包起来

范围之外的数据

```sql
select *
from customer
where cust_id NOT IN (1, 2, 3)
```

### 模糊查询（使用通配符）

使用通配符，根据Mysql的配置，可以区分大小写

| 通配符 | 说明                                 |
| ------ | ------------------------------------ |
| %      | 任意字符出现任意次数（不能匹配NULL） |
| _      | 匹配任意字符只匹配一次               |

```sql
SELECT *
FROM customer
WHERE cust_name LIKE '%昭%'
```

```sql
SELECT *
FROM customer
WHERE cust_name LIKE '_白'
```

### 正则表达式查询

使用REGEXP关键字，基本语法和通用正则表达式一致。

基本字符搜索

```sql
SELECT *
FROM customer
WHERE cust_name REGEXP '.信'
```

两者之一搜索（两者以上可以用多个 | 分割

```sql
SELECT *
FROM customer
WHERE cust_phone REGEXP '133|137'
```

匹配几个字符之一

```sql
SELECT *
FROM customer
WHERE cust_phone REGEXP '1[123]'
```

# 计算字段

利用计算字段可以返回我们想要的数据呈现形式

### 字段拼接

#### Concat()函数

```sql
select CONCAT(user_name, '(', user_real_name, ')')
FROM user_info
```

使用此种方式可以将查询结果变成自己想要的形式，上面sql查询结果为

> 你大 爷(葛军)
> 围棋泰斗(聂卫平)
> 天津股侠(于志英)
> 儿科医生(鲍秀兰)
> 古籍(李学勤)

#### RTrim()函数

去掉值右边的所有空格

#### LTrim()函数

去掉值左边的所有空格

#### Trim()函数

去掉值左右两边的所有空格

```sql
select CONCAT(TRIM(user_name), '(', user_real_name, ')')
FROM user_info
```

### 别名

#### AS关键字

```sql
select CONCAT(TRIM(user_name), '(', user_real_name, ')')
           AS userinfo
FROM user_info
```

### 算术计算

```sql
SELECT sku_id,
       amount,
       (price * amount)
           AS totalprice
FROM sku_info
```

mysql可以支持将表中的某些列进行计算合并之后重新成为一个新的列。

支持 + - * / 优先级可以使用（）来控制。

# 函数

Mysql支持很多函数操作，但是函数的可移植性不高，因此使用函数时尽量写好注释

## 文本处理函数

| 函数                                  | 描述                                                         | 实例                                                         |
| :------------------------------------ | :----------------------------------------------------------- | :----------------------------------------------------------- |
| ASCII(s)                              | 返回字符串 s 的第一个字符的 ASCII 码。                       | 返回 CustomerName 字段第一个字母的 ASCII 码：`SELECT ASCII(CustomerName) AS NumCodeOfFirstChar FROM Customers;` |
| CHAR_LENGTH(s)                        | 返回字符串 s 的字符数                                        | 返回字符串 RUNOOB 的字符数`SELECT CHAR_LENGTH("RUNOOB") AS LengthOfString;` |
| CHARACTER_LENGTH(s)                   | 返回字符串 s 的字符数                                        | 返回字符串 RUNOOB 的字符数`SELECT CHARACTER_LENGTH("RUNOOB") AS LengthOfString;` |
| CONCAT(s1,s2...sn)                    | 字符串 s1,s2 等多个字符串合并为一个字符串                    | 合并多个字符串`SELECT CONCAT("SQL ", "Runoob ", "Gooogle ", "Facebook") AS ConcatenatedString;` |
| CONCAT_WS(x, s1,s2...sn)              | 同 CONCAT(s1,s2,...) 函数，但是每个字符串之间要加上 x，x 可以是分隔符 | 合并多个字符串，并添加分隔符：`SELECT CONCAT_WS("-", "SQL", "Tutorial", "is", "fun!")AS ConcatenatedString;` |
| FIELD(s,s1,s2...)                     | 返回第一个字符串 s 在字符串列表(s1,s2...)中的位置            | 返回字符串 c 在列表值中的位置：`SELECT FIELD("c", "a", "b", "c", "d", "e");` |
| FIND_IN_SET(s1,s2)                    | 返回在字符串s2中与s1匹配的字符串的位置                       | 返回字符串 c 在指定字符串中的位置：`SELECT FIND_IN_SET("c", "a,b,c,d,e");` |
| FORMAT(x,n)                           | 函数可以将数字 x 进行格式化 "#,###.##", 将 x 保留到小数点后 n 位，最后一位四舍五入。 | 格式化数字 "#,###.##" 形式：`SELECT FORMAT(250500.5634, 2); -- 输出 250,500.56` |
| INSERT(s1,x,len,s2)                   | 字符串 s2 替换 s1 的 x 位置开始长度为 len 的字符串           | 从字符串第一个位置开始的 6 个字符替换为 runoob：`SELECT INSERT("google.com", 1, 6, "runnob"); -- 输出：runoob.com` |
| LOCATE(s1,s)                          | 从字符串 s 中获取 s1 的开始位置                              | 获取 b 在字符串 abc 中的位置：`SELECT LOCATE('st','myteststring'); -- 5`返回字符串 abc 中 b 的位置：`SELECT LOCATE('b', 'abc') -- 2` |
| LCASE(s)                              | 将字符串 s 的所有字母变成小写字母                            | 字符串 RUNOOB 转换为小写：`SELECT LCASE('RUNOOB') -- runoob` |
| LEFT(s,n)                             | 返回字符串 s 的前 n 个字符                                   | 返回字符串 runoob 中的前两个字符：`SELECT LEFT('runoob',2) -- ru` |
| LOWER(s)                              | 将字符串 s 的所有字母变成小写字母                            | 字符串 RUNOOB 转换为小写：`SELECT LOWER('RUNOOB') -- runoob` |
| LPAD(s1,len,s2)                       | 在字符串 s1 的开始处填充字符串 s2，使字符串长度达到 len      | 将字符串 xx 填充到 abc 字符串的开始处：`SELECT LPAD('abc',5,'xx') -- xxabc` |
| LTRIM(s)                              | 去掉字符串 s 开始处的空格                                    | 去掉字符串 RUNOOB开始处的空格：`SELECT LTRIM("    RUNOOB") AS LeftTrimmedString;-- RUNOOB` |
| MID(s,n,len)                          | 从字符串 s 的 n 位置截取长度为 len 的子字符串，同 SUBSTRING(s,n,len) | 从字符串 RUNOOB 中的第 2 个位置截取 3个 字符：`SELECT MID("RUNOOB", 2, 3) AS ExtractString; -- UNO` |
| POSITION(s1 IN s)                     | 从字符串 s 中获取 s1 的开始位置                              | 返回字符串 abc 中 b 的位置：`SELECT POSITION('b' in 'abc') -- 2` |
| REPEAT(s,n)                           | 将字符串 s 重复 n 次                                         | 将字符串 runoob 重复三次：`SELECT REPEAT('runoob',3) -- runoobrunoobrunoob` |
| REPLACE(s,s1,s2)                      | 将字符串 s2 替代字符串 s 中的字符串 s1                       | 将字符串 abc 中的字符 a 替换为字符 x：`SELECT REPLACE('abc','a','x') --xbc` |
| REVERSE(s)                            | 将字符串s的顺序反过来                                        | 将字符串 abc 的顺序反过来：`SELECT REVERSE('abc') -- cba`    |
| RIGHT(s,n)                            | 返回字符串 s 的后 n 个字符                                   | 返回字符串 runoob 的后两个字符：`SELECT RIGHT('runoob',2) -- ob` |
| RPAD(s1,len,s2)                       | 在字符串 s1 的结尾处添加字符串 s2，使字符串的长度达到 len    | 将字符串 xx 填充到 abc 字符串的结尾处：`SELECT RPAD('abc',5,'xx') -- abcxx` |
| RTRIM(s)                              | 去掉字符串 s 结尾处的空格                                    | 去掉字符串 RUNOOB 的末尾空格：`SELECT RTRIM("RUNOOB     ") AS RightTrimmedString; -- RUNOOB` |
| SPACE(n)                              | 返回 n 个空格                                                | 返回 10 个空格：`SELECT SPACE(10);`                          |
| STRCMP(s1,s2)                         | 比较字符串 s1 和 s2，如果 s1 与 s2 相等返回 0 ，如果 s1>s2 返回 1，如果 s1<s2 返回 -1 | 比较字符串：`SELECT STRCMP("runoob", "runoob"); -- 0`       |
| SUBSTR(s, start, length)              | 从字符串 s 的 start 位置截取长度为 length 的子字符串         | 从字符串 RUNOOB 中的第 2 个位置截取 3个 字符：`SELECT SUBSTR("RUNOOB", 2, 3) AS ExtractString; -- UNO` |
| SUBSTRING(s, start, length)           | 从字符串 s 的 start 位置截取长度为 length 的子字符串         | 从字符串 RUNOOB 中的第 2 个位置截取 3个 字符：`SELECT SUBSTRING("RUNOOB", 2, 3) AS ExtractString; -- UNO` |
| SUBSTRING_INDEX(s, delimiter, number) | 返回从字符串 s 的第 number 个出现的分隔符 delimiter 之后的子串。 如果 number 是正数，返回第 number 个字符左边的字符串。 如果 number 是负数，返回第(number 的绝对值(从右边数))个字符右边的字符串。 | `SELECT SUBSTRING_INDEX('a*b','*',1) -- a SELECT SUBSTRING_INDEX('a*b','*',-1)  -- b SELECT SUBSTRING_INDEX(SUBSTRING_INDEX('a*b*c*d*e','*',3),'*',-1)  -- c` |
| TRIM(s)                               | 去掉字符串 s 开始和结尾处的空格                              | 去掉字符串 RUNOOB 的首尾空格：`SELECT TRIM('    RUNOOB    ') AS TrimmedString;` |
| UCASE(s)                              | 将字符串转换为大写                                           | 将字符串 runoob 转换为大写：`SELECT UCASE("runoob"); -- RUNOOB` |
| UPPER(s)                              | 将字符串转换为大写                                           | 将字符串 runoob 转换为大写：`SELECT UPPER("runoob"); -- RUNOOB` |

## 时间日期函数

| 函数名                            | 描述                                                         | 实例                                                         |
| :-------------------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- |
| ADDDATE(d,n)                      | 计算起始日期 d 加上 n 天的日期                               | `SELECT ADDDATE("2017-06-15", INTERVAL 10 DAY); ->2017-06-25` |
| ADDTIME(t,n)                      | 时间 t 加上 n 秒的时间                                       | `SELECT ADDTIME('2011-11-11 11:11:11', 5) ->2011-11-11 11:11:16 (秒)` |
| CURDATE()                         | 返回当前日期                                                 | `SELECT CURDATE(); -> 2018-09-19`                            |
| CURRENT_DATE()                    | 返回当前日期                                                 | `SELECT CURRENT_DATE(); -> 2018-09-19`                       |
| CURRENT_TIME                      | 返回当前时间                                                 | `SELECT CURRENT_TIME(); -> 19:59:02`                         |
| CURRENT_TIMESTAMP()               | 返回当前日期和时间                                           | `SELECT CURRENT_TIMESTAMP() -> 2018-09-19 20:57:43`          |
| CURTIME()                         | 返回当前时间                                                 | `SELECT CURTIME(); -> 19:59:02`                              |
| DATE()                            | 从日期或日期时间表达式中提取日期值                           | `SELECT DATE("2017-06-15"); -> 2017-06-15`               |
| DATEDIFF(d1,d2)                   | 计算日期 d1->d2 之间相隔的天数                               | `SELECT DATEDIFF('2001-01-01','2001-02-02') -> -32`          |
| DATE_ADD(d，INTERVAL expr type)   | 计算起始日期 d 加上一个时间段后的日期                        | `SELECT ADDDATE('2011-11-11 11:11:11',1) -> 2011-11-12 11:11:11  (默认是天) SELECT ADDDATE('2011-11-11 11:11:11', INTERVAL 5 MINUTE) -> 2011-11-11 11:16:11 (TYPE的取值与上面那个列出来的函数类似)` |
| DATE_FORMAT(d,f)                  | 按表达式 f的要求显示日期 d                                   | `SELECT DATE_FORMAT('2011-11-11 11:11:11','%Y-%m-%d %r') -> 2011-11-11 11:11:11 AM` |
| DATE_SUB(date,INTERVAL expr type) | 函数从日期减去指定的时间间隔。                               | Orders 表中 OrderDate 字段减去 2 天：`SELECT OrderId,DATE_SUB(OrderDate,INTERVAL 2 DAY) AS OrderPayDate FROM Orders` |
| DAY(d)                            | 返回日期值 d 的日期部分                                      | `SELECT DAY("2017-06-15"); -> 15`                          |
| DAYNAME(d)                        | 返回日期 d 是星期几，如 Monday,Tuesday                       | `SELECT DAYNAME('2011-11-11 11:11:11') ->Friday`             |
| DAYOFMONTH(d)                     | 计算日期 d 是本月的第几天                                    | `SELECT DAYOFMONTH('2011-11-11 11:11:11') ->11`              |
| DAYOFWEEK(d)                      | 日期 d 今天是星期几，1 星期日，2 星期一，以此类推            | `SELECT DAYOFWEEK('2011-11-11 11:11:11') ->6`                |
| DAYOFYEAR(d)                      | 计算日期 d 是本年的第几天                                    | `SELECT DAYOFYEAR('2011-11-11 11:11:11') ->315`              |
| EXTRACT(type FROM d)              | 从日期 d 中获取指定的值，type 指定返回的值。 type可取值为： <br />MICROSECOND<br />SECOND<br />MINUTE<br />HOUR<br />DAY<br />WEEK<br />MONTH<br />QUARTER<br />YEAR<br />SECOND_MICROSECOND<br />MINUTE_MICROSECOND<br />MINUTE_SECOND<br />HOUR_MICROSECOND<br />HOUR_SECOND<br />HOUR_MINUTE<br />DAY_MICROSECOND<br />DAY_SECOND<br />DAY_MINUTE<br />DAY_HOUR<br />YEAR_MONTH | `SELECT EXTRACT(MINUTE FROM '2011-11-11 11:11:11')  -> 11`   |
| FROM_DAYS(n)                      | 计算从 0000 年 1 月 1 日开始 n 天后的日期                    | `SELECT FROM_DAYS(1111) -> 0003-01-16`                       |
| HOUR(t)                           | 返回 t 中的小时值                                            | `SELECT HOUR('1:2:3') -> 1`                                  |
| LAST_DAY(d)                       | 返回给给定日期的那一月份的最后一天                           | `SELECT LAST_DAY("2017-06-20"); -> 2017-06-30`               |
| LOCALTIME()                       | 返回当前日期和时间                                           | `SELECT LOCALTIME() -> 2018-09-19 20:57:43`                  |
| LOCALTIMESTAMP()                  | 返回当前日期和时间                                           | `SELECT LOCALTIMESTAMP() -> 2018-09-19 20:57:43`             |
| MAKEDATE(year, day-of-year)       | 基于给定参数年份 year 和所在年中的天数序号 day-of-year 返回一个日期 | `SELECT MAKEDATE(2017, 3); -> 2017-01-03`                    |
| MAKETIME(hour, minute, second)    | 组合时间，参数分别为小时、分钟、秒                           | `SELECT MAKETIME(11, 35, 4); -> 11:35:04`                    |
| MICROSECOND(date)                 | 返回日期参数所对应的微秒数                                   | `SELECT MICROSECOND("2017-06-20 09:34:00.000023"); -> 23`    |
| MINUTE(t)                         | 返回 t 中的分钟值                                            | `SELECT MINUTE('1:2:3') -> 2`                                |
| MONTHNAME(d)                      | 返回日期当中的月份名称，如 November                          | `SELECT MONTHNAME('2011-11-11 11:11:11') -> November`        |
| MONTH(d)                          | 返回日期d中的月份值，1 到 12                                 | `SELECT MONTH('2011-11-11 11:11:11') ->11`                   |
| NOW()                             | 返回当前日期和时间                                           | `SELECT NOW() -> 2018-09-19 20:57:43`                        |
| PERIOD_ADD(period, number)        | 为 年-月 组合日期添加一个时段                                | `SELECT PERIOD_ADD(201703, 5); -> 201708`                 |
| PERIOD_DIFF(period1, period2)     | 返回两个时段之间的月份差值                                   | `SELECT PERIOD_DIFF(201710, 201703); -> 7`                   |
| QUARTER(d)                        | 返回日期d是第几季节，返回 1 到 4                             | `SELECT QUARTER('2011-11-11 11:11:11') -> 4`                 |
| SECOND(t)                         | 返回 t 中的秒钟值                                            | `SELECT SECOND('1:2:3') -> 3`                                |
| SEC_TO_TIME(s)                    | 将以秒为单位的时间 s 转换为时分秒的格式                      | `SELECT SEC_TO_TIME(4320) -> 01:12:00`                       |
| STR_TO_DATE(string, format_mask)  | 将字符串转变为日期                                           | `SELECT STR_TO_DATE("August 10 2017", "%M %d %Y"); -> 2017-08-10` |
| SUBDATE(d,n)                      | 日期 d 减去 n 天后的日期                                     | `SELECT SUBDATE('2011-11-11 11:11:11', 1) ->2011-11-10 11:11:11 (默认是天)` |
| SUBTIME(t,n)                      | 时间 t 减去 n 秒的时间                                       | `SELECT SUBTIME('2011-11-11 11:11:11', 5) ->2011-11-11 11:11:06 (秒)` |
| SYSDATE()                         | 返回当前日期和时间                                           | `SELECT SYSDATE() -> 2018-09-19 20:57:43`                    |
| TIME(expression)                  | 提取传入表达式的时间部分                                     | `SELECT TIME("19:30:10"); -> 19:30:10`                       |
| TIME_FORMAT(t,f)                  | 按表达式 f 的要求显示时间 t                                  | `SELECT TIME_FORMAT('11:11:11','%r') 11:11:11 AM`            |
| TIME_TO_SEC(t)                    | 将时间 t 转换为秒                                            | `SELECT TIME_TO_SEC('1:12:00') -> 4320`                      |
| TIMEDIFF(time1, time2)            | 计算时间差值                                                 | `SELECT TIMEDIFF("13:10:11", "13:10:10"); -> 00:00:01`       |
| TIMESTAMP(expression, interval)   | 单个参数时，函数返回日期或日期时间表达式；有2个参数时，将参数加和 | `SELECT TIMESTAMP("2017-07-23",  "13:10:11"); -> 2017-07-23 13:10:11` |
| TO_DAYS(d)                        | 计算日期 d 距离 0000 年 1 月 1 日的天数                      | `SELECT TO_DAYS('0001-01-01 01:01:01') -> 366`               |
| WEEK(d)                           | 计算日期 d 是本年的第几个星期，范围是 0 到 53                | `SELECT WEEK('2011-11-11 11:11:11') -> 45`                   |
| WEEKDAY(d)                        | 日期 d 是星期几，0 表示星期一，1 表示星期二                  | `SELECT WEEKDAY("2017-06-15"); -> 3`                         |
| WEEKOFYEAR(d)                     | 计算日期 d 是本年的第几个星期，范围是 0 到 53                | `SELECT WEEKOFYEAR('2011-11-11 11:11:11') -> 45`             |
| YEAR(d)                           | 返回年份                                                     | `SELECT YEAR("2017-06-15"); -> 2017`                         |
| YEARWEEK(date, mode)              | 返回年份及第几周（0到53），mode 中 0 表示周天，1表示周一，以此类推 | `SELECT YEARWEEK("2017-06-15"); -> 201724`                   |

## 数字处理函数

| 函数名                                               | 描述                                                         | 实例                                                         |
| :--------------------------------------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- |
| ABS(x)                                               | 返回 x 的绝对值                                              | 返回 -1 的绝对值：`SELECT ABS(-1) -- 返回1`                  |
| ACOS(x)                                              | 求 x 的反余弦值(参数是弧度)                                  | `SELECT ACOS(0.25);`                                         |
| ASIN(x)                                              | 求反正弦值(参数是弧度)                                       | `SELECT ASIN(0.25);`                                         |
| ATAN(x)                                              | 求反正切值(参数是弧度)                                       | `SELECT ATAN(2.5);`                                          |
| ATAN2(n, m)                                          | 求反正切值(参数是弧度)                                       | `SELECT ATAN2(-0.8, 2);`                                     |
| AVG(expression)                                      | 返回一个表达式的平均值，expression 是一个字段                | 返回 Products 表中Price 字段的平均值：`SELECT AVG(Price) AS AveragePrice FROM Products;` |
| CEIL(x)                                              | 返回大于或等于 x 的最小整数                                  | `SELECT CEIL(1.5) -- 返回2`                                  |
| CEILING(x)                                           | 返回大于或等于 x 的最小整数                                  | `SELECT CEIL(1.5) -- 返回2`                                  |
| COS(x)                                               | 求余弦值(参数是弧度)                                         | `SELECT COS(2);`                                             |
| COT(x)                                               | 求余切值(参数是弧度)                                         | `SELECT COT(6);`                                             |
| COUNT(expression)                                    | 返回查询的记录总数，expression 参数是一个字段或者 * 号       | 返回 Products 表中 products 字段总共有多少条记录：`SELECT COUNT(ProductID) AS NumberOfProducts FROM Products;` |
| DEGREES(x)                                           | 将弧度转换为角度                                             | `SELECT DEGREES(3.1415926535898) -- 180`                     |
| n DIV m                                              | 整除，n 为被除数，m 为除数                                   | 计算 10 除于 5：`SELECT 10 DIV 5; -- 2`                     |
| EXP(x)                                               | 返回 e 的 x 次方                                             | 计算 e 的三次方：`SELECT EXP(3) -- 20.085536923188`          |
| FLOOR(x)                                             | 返回小于或等于 x 的最大整数                                  | 小于或等于 1.5 的整数：`SELECT FLOOR(1.5) -- 返回1`          |
| GREATEST(expr1, expr2, expr3, ...)                   | 返回列表中的最大值                                           | 返回以下数字列表中的最大值：`SELECT GREATEST(3, 12, 34, 8, 25); -- 34`返回以下字符串列表中的最大值：`SELECT GREATEST("Google", "Runoob", "Apple"); -- Runoob` |
| LEAST(expr1, expr2, expr3, ...)                      | 返回列表中的最小值                                           | 返回以下数字列表中的最小值：`SELECT LEAST(3, 12, 34, 8, 25); -- 3`返回以下字符串列表中的最小值：`SELECT LEAST("Google", "Runoob", "Apple"); -- Apple` |
| [LN](https://www.runoob.com/mysql/func_mysql_ln.asp) | 返回数字的自然对数                                           | 返回 2 的自然对数：`SELECT LN(2); -- 0.6931471805599453`    |
| LOG(x)                                               | 返回自然对数(以 e 为底的对数)                                | `SELECT LOG(20.085536923188) -- 3`                           |
| LOG10(x)                                             | 返回以 10 为底的对数                                         | `SELECT LOG10(100) -- 2`                                     |
| LOG2(x)                                              | 返回以 2 为底的对数                                          | 返回以 2 为底 6 的对数：`SELECT LOG2(6); -- 2.584962500721156` |
| MAX(expression)                                      | 返回字段 expression 中的最大值                               | 返回数据表 Products 中字段 Price 的最大值：`SELECT MAX(Price) AS LargestPrice FROM Products;` |
| MIN(expression)                                      | 返回字段 expression 中的最小值                               | 返回数据表 Products 中字段 Price 的最小值：`SELECT MIN(Price) AS LargestPrice FROM Products;` |
| MOD(x,y)                                             | 返回 x 除以 y 以后的余数                                     | 5 除于 2 的余数：`SELECT MOD(5,2) -- 1`                      |
| PI()                                                 | 返回圆周率(3.141593）                                        | `SELECT PI() --3.141593`                                     |
| POW(x,y)                                             | 返回 x 的 y 次方                                             | 2 的 3 次方：`SELECT POW(2,3) -- 8`                          |
| POWER(x,y)                                           | 返回 x 的 y 次方                                             | 2 的 3 次方：`SELECT POWER(2,3) -- 8`                        |
| RADIANS(x)                                           | 将角度转换为弧度                                             | 180 度转换为弧度：`SELECT RADIANS(180) -- 3.1415926535898`   |
| RAND()                                               | 返回 0 到 1 的随机数                                         | `SELECT RAND() --0.93099315644334`                           |
| ROUND(x)                                             | 返回离 x 最近的整数                                          | `SELECT ROUND(1.23456) --1`                                  |
| SIGN(x)                                              | 返回 x 的符号，x 是负数、0、正数分别返回 -1、0 和 1          | `SELECT SIGN(-10) -- (-1)`                                   |
| SIN(x)                                               | 求正弦值(参数是弧度)                                         | `SELECT SIN(RADIANS(30)) -- 0.5`                             |
| SQRT(x)                                              | 返回x的平方根                                                | 25 的平方根：`SELECT SQRT(25) -- 5`                          |
| SUM(expression)                                      | 返回指定字段的总和                                           | 计算 OrderDetails 表中字段 Quantity 的总和：`SELECT SUM(Quantity) AS TotalItemsOrdered FROM OrderDetails;` |
| TAN(x)                                               | 求正切值(参数是弧度)                                         | `SELECT TAN(1.75); -- -5.52037992250933`                    |
| TRUNCATE(x,y)                                        | 返回数值 x 保留到小数点后 y 位的值（与 ROUND 最大的区别是不会进行四舍五入） | `SELECT TRUNCATE(1.23456,3) -- 1.234`                        |

# 数据汇总

| 函数    | 说明               |
| ------- | ------------------ |
| AVG()   | 返回某一列的评论值 |
| COUNT() | 返回某列的行数     |
| MAX()   | 返回某列的最大值   |
| MIN()`` | 返回某列的最小值   |
| SUM()   | 返回某列的和       |

使用ALL和DISTINCT关键字

对于上面的函数，可以使用这两个关键字进行计算时候的筛选，如果是ALL，则计算所有的数据，这个也是默认选项，如果不指定，默认的也是ALL，如果指定DISINCT，那么计算值得时候就会排除值相同得元素，这会对计算结果有一定得影响，如果你确实需要这样做得时候。

实际上，各个函数之间可以进行组合使用。

# 分组查询

使用group by关键字进行分组查询

如下sql，对cust_profession和总的数据进行查询，对查询的结果依照cust_profession进行分组统计。

```sql
SELECT customer.cust_profession, COUNT(*) AS position
FROM customer
GROUP BY cust_profession
```

group by语句必须出现在where之后，order by之前 ，使用group by，如果有null字段，将会被单独分为一组

```sql
SELECT customer.cust_profession, COUNT(*) AS position
FROM customer
GROUP BY cust_profession
WITH ROLLUP
```

如上使用WITH ROLLUP将会得到一个分组数据的汇总

### 分组过滤

group
by中的分组使用having进行过滤，其使用方法和where是一样的，只有关键字的差别，但是这里却不能使用where，因为where是对行的过滤，having才是对分组的过滤。不过基本用where的地方都可以使用having进行代替。

```sql
SELECT cust_id, customer.cust_profession, COUNT(*) AS position
FROM customer
GROUP BY cust_profession
HAVING COUNT (*)>4
```

# select子句的顺序要求

```html
select  ->  from  ->  where  ->  group by  ->  having  ->  order  by  ->  limit
```

使用select查询的时候，子句的组合必须符合上面的顺序要求。

# 子查询

子查询简单来讲就是，从已经查询到的结果中再进行查询。或者说，将一个select查询的结果作为一个临时的表，供其他select进行查询。

```sql
SELECT cust_name
FROM customer
WHERE cust_name
          in (SELECT customer.cust_name FROM customer WHERE cust_id BETWEEN 1 AND 4)
```

如上就是一个典型的子查询，最里面一层select的查询结果作为最外层select查询的数据来源。

# 表的联结（表与表之间的关系）

### 外键

外键为某个表中的一列，它包含了另一个表中的主键值，定义了两个表之间的关系

### 全限定名

当查询的时候可能会出现字段的歧义性，因此这个时候可以使用全限定名，即表名.字段名

如下代码演示联结查询的列子，字段使用全限定名形式

```sql
SELECT customer.cust_name, `order`.orderName
FROM customer,
     `order`
WHERE `order`.orderCustId = customer.cust_id
```

### 笛卡尔积

再查询多个表的时候，如果没有明确的匹配条件，那么查询的结果就会称为笛卡尔积的效果，比如存在A、B两个表，那么结果就是A表中的每一个字段都会把B表中的所有字段都匹配一次，这样的结果显然是没有意义的。如：

```sql
SELECT customer.cust_name, `order`.orderName
FROM customer,
     `order` 
```

### 内部联结查询

```sql
SELECT `order`.orderName, customer.cust_name
FROM customer
         INNER JOIN `order`
                    ON `order`.orderCustId = customer.cust_id
```

使用此种方式和where的效果一样，但是表达的意思更加明确，on后面的条件和where后面的是一样的。

### 联结查询多个表

mysql对联结表的数量是没有限制的，因此可以联结查询多个表，格式和前面的一样，先列出所有的表，后面跟上过滤的条件，但是需要谨慎使用，因为此种方式会极大的降低性能。

# 高级联结

### 表的别名

除了可以给字段起别名，同样的mysql也支持给表起别名。

```sql
SELECT C.cust_name, C.cust_profession
FROM customer
         AS C
WHERE C.cust_id < 10
```

AS关键字是可以省略的。

### 自联结查询

自联结查询表示对一个表起两个不同的别名来方便查询，自联结查询对于子查询而言，有时性能会更加突出。

```sql
SELECT c2.cust_name, c2.cust_profession
FROM customer as c1,
     customer as c2
WHERE c2.email = '22342@qq.com'
  AND c1.cust_id = c2.cust_id
```

### 自然联结查询

首先自然连接查询是等值联结查询的一种，再此基础上，自然联结查询消除了重复列的出现。

```sql
SELECT *
FROM customer
         NATURAL JOIN `order`
WHERE `order`.cust_id = customer.cust_id
```

### 外部联结查询

根据内部联结和自然连接查询的情况来说，他们的查询结果只有在满足等值条件的时候才会出现在结果中，但是如果希望两个表中即使没有关联的数据也出现在结果中，那么就需要使用外部查询了。

```sql
SELECT *
FROM customer
         LEFT OUTER JOIN `order`
                         ON `order`.cust_id = customer.cust_id
```

此时，哪些没有被关联上的数据也会作为结果显示出来，这里有LEFT OUTER JOIN和RIGHT OUTER JOIN两种，表示的是分别以左边的表为基准展示还是以右边的表为基准展示所有结果

### 在联结查询中使用聚合函数

可以在联结查询中使用聚合函数对查询结果进行统计

```sql
SELECT orderName, cust_profession, COUNT(cust_profession) AS profession
FROM customer
         INNER JOIN `order`
                    ON `order`.cust_id = customer.cust_id
GROUP BY cust_profession
```

### 使用联结查询的注意事项

- 我们一般使用内部联结即可
- 应该要记住提供联结查询条件，避免笛卡尔积
- 使用联结查询的时候可以多试几种，测试性能，选取最优

# 组合查询

使用UNION关键字可以将多个select查询语句进行组合

```sql
SELECT c.cust_name, c.cust_phone
FROM customer c
WHERE cust_id < 10
UNION
SELECT c.cust_phone, c.cust_name
FROM customer c
WHERE c.cust_name = '李白拜拜'
```

如上即是一个简单的组合查询，需要注意的是，查询的结果为两个查询语句结果的并集。

使用UNION的注意事项

- 使用UNION必须是2条select语句及其以上的情况
- 每个查询必须包含相同的列、表达式、聚集函数，但是顺序没有要求，否则查询的结果一定会出现冗余。
- 列的数据类型必须兼容，类型不必一定相同，但是之间可以隐式的转化。

### 组合查询开启和取消重复行

UNION在默认情况下是会去掉重复行的，但是如果在有些情况下需要开启显示重复行，则将UNION关键字修改为UNION ALL即可

```sql
SELECT c.cust_name, c.cust_phone
FROM customer c
WHERE cust_id < 10
UNION ALL
SELECT c.cust_phone, c.cust_name
FROM customer c
WHERE c.cust_name = '李白拜拜'
```

### 组合查询的排序

组合查询的排序同样使用order by，不过此条件必须放在语句的最后面

```sql
SELECT customer.cust_name, customer.cust_id
FROM customer
WHERE cust_id < 10
UNION
SELECT customer.cust_name, customer.cust_id
FROM customer
WHERE cust_name = '王昭君'
ORDER BY cust_id DESC
```

# 全文本搜索（未完...）

常见的数据库引擎有MyISAM和InnoDB，前者支持全文本搜索，后者不支持，如果需要做这个功能因该注意这一点。

# 插入数据

插入数据有一下几种形式

- 插入完整的行
- 插入行的一部分
- 插入多行
- 插入某些查询的结果

### 插入完整行

```sql
INSERT INTO customer
VALUES (122, '大哥大', '肉', 112, '123232@qq.com')
```

此种方式并不是很安全，因为他有严格的数据顺序要求

如果想较为安全的插入数据，可以使用给出列名的形式

```sql
INSERT INTO customer (cust_name, cust_profession, cust_phone, email)
VALUES ('大哥大', '肉', 112, '123232@qq.com')
```

此种方式可以明确需要插入哪些列数据，如果没有的会自动填充为null值（前提是本列允许为NULL），同时此种种方式指定的列顺序可以和表中的数据顺序不一致，插入时只需遵循自己写的字段顺序即可，mysql会自动识别填充。

上面两种方式插入数据，强烈建议不要使用第一种。

在第二种方式中，如果省略某些列不写，则应该满足以下条件

- 允许为NULL值
- 在表定义种给出默认值

### 提高mysql的性能

insert语句相对来讲是很耗能的，因此如果你的数据库经常被用来查询而不是更改，则在insert、update、delete中可以降低他们的优先级。

```sql
INSERT
LOW_PRIORITY INTO
```

update和delete同理。

### 插入多行

单独使用多条insert语句是最通俗的方法。

或者可以使用这种方式：提供一次列名，提供多次数据的集合

```sql
INSERT INTO customer (cust_name, cust_profession, cust_phone, email)
VALUES ('大哥大', '肉', 112, '123232@qq.com'),
       ('无时大地', '肉', 112, '123232@qq.com'),
       ('无时大地', '肉', 112, '123232@qq.com'),
       ('无时大地', '肉', 112, '123232@qq.com'),
       ('无时大地', '肉', 112, '123232@qq.com'),
       ('无时大地', '肉', 112, '123232@qq.com');
```

使用下面这种方式可以提高mysql数据的性能

### 插入查询出来的数据

```sql
INSERT INTO customer (cust_name, cust_profession, cust_phone, email)
SELECT cust_name, cust_profession, cust_phone, email
FROM customer
```

使用此种方式可以将查询出来的值重新插入到表中，这里演示的是同一张表，但实际上可以是不同的表，不过要求表之间的结构尽量相同，至少是数据类型上，这里的select语句同样支持where的条件筛选。

# 更新数据

更新数据有以下两种形式

- 更新表中特定的行
- 更新表中所有的行

在使用updata的时候一定要格外注意，因为稍不留神，就会更新表中所有的数据。因此删选条件一定不要忘记，除非你明确的要更新所有的数据。

```sql
UPDATE customer
SET email = '110@qq.com'
WHERE cust_id = 1
```

### 更新多列

```sql
UPDATE customer
SET email     = '110@qq.com',
    cust_name ='望天吧'
WHERE cust_id = 2
```

如果在更新过程中某一条记录的更新出现错误，则整个更新操作就停止，如果不想出现这种情况则可以忽略局部的错误，让整个更新操作继续进行。

使用ignore关键字

```sql
UPDATE IGNORE customer
SET email     = '110@qq.com',
    cust_name ='望天吧'
WHERE cust_id = 2
```

# 删除数据

删除数据同样有两种形式

- 删除特定行
- 删除所有行

如果没有过滤条件，那么也是很容易删除表中的所有数据的，因此在执行delete操作的时候，要时刻注意筛选条件。

```sql
DELETE
FROM customer
WHERE customer.cust_id = 100
```

delete操作删除的是整行数据，如果要删除特定某一列的数据，应该使用updata语句进行操作。

# MySQL存储过程

存储过程是一些列完整sql的集合，可以将很多sql操作进行封装，就像是Java的方法一样。

## 执行存储过程

```sql
CALL productprincing(@pricelow,@pricelhigh,@priceaverage)
```

如上代码，展示如何调用一个存储过程。 这里调用了一个名为productprincing的存储过程。

## 创建存储过程

```sql
CREATE PROCEDURE avageid()
BEGIN
SELECT AVG(customer.cust_id) as avageid
FROM customer;
END;

```

如上代码即创建了一个很简单的存储过程

## 修改分割符

由于mysql默认的语句分隔符为“；”，而MySQL的命令执行分隔符也为“；”，为了避免冲突。可以自己定义分隔符，使用完了之后再修改回来

```sql
DELIMITER
//
CREATE PROCEDURE avageid()
BEGIN
SELECT AVG(customer.cust_id) as avageid
FROM customer;
END
//
DELIMITER ;
```

## 删除存储过程

```sql
DROP PROCEDURE avageid;
```

只有存储过程是存在的时候才能删除，如果删除一个不存在的存储过程则会报错。

## 带参数的存储过程

以下代码展示一个带参数的存储过程，并详细解释如何获取存储过程的结果。

需要注意的是，存储过程的返回结果不能是多列或者多行。只能是一个确切的结果，如果要有多个结果，应该有多个参数来接收这个结果。

```sql
CREATE PROCEDURE get_profession_phone(-- 	OUT代表这两个参数是要从存储过程中传递出去的，phone和profession是记录传递出去数据的临时变量名
--  后面紧跟的是数据类型，这里类似Java中的形参
--  与OUT对应的还有IN（表示传递给存储过程）、INOUT（对存储过程的传入和传出）
    OUT phone BIGINT(13),
    OUT profession VARCHAR (20))
BEGIN
SELECT customer.cust_phone
INTO phone --此处将查询出来的数据封装到phone参数中，和上面的是对应的
FROM customer
WHERE customer.cust_id = 1;
SELECT customer.cust_profession
INTO profession
FROM customer
WHERE customer.cust_id = 1;
END;
```

执行存储过程并获得存储结果。

```sql
CALL get_profession_phone(@phone,@profession);

SELECT @phone;
```

调用有参数的存储过程，其参数必须加上@符号，并且，这里的参数名自定义，不必和定义存储过程的时候的一样。相当于Java中的实参。

执行了存储过程之后并不会返回一个明确的结果并显示，需要我们手动调用获得的结果。也就是后面的select @phone。这样即可获得存储过程封装在phone这个变量中的结果值。

## 智能存储过程

如果我们为存储过程添加一些条件判断，那么存储过程将会具备更加强大的功能。

存储过程中同样支持if条件判断和while循环的使用。具体使用方法菜鸟教程可查。

## 检查存储过程

```sql
show
create procedure <存储过程名>
```

使用以上代码即可查询整个存储过程的详细信息。

# 触发器

触发器就是在数据库中，做了一定条件的操作之后就会触发另外一个操作。比如在删除某一条记录后，另外一条sql语句就会被触发并自动执行。这就是触发器。

MySQL中支持update、insert、delete三种sql语句的触发器

## 创建触发器

创建触发器需要注意以下4点

1. 唯一的触发器名
2. 触发器关联的表
3. 触发器应该响应的活动（insert、delete、update）
4. 触发器在何时执行（处理前还是处理后，before、after）

一张表最多拥有6种触发器，为出发类型和触发时间的组合

### 触发器的基本语法

```sql
Create trigger 触发器名字 出发时间 事件类型 on 表名 for each row
begin
	--触发器内容
UPDATE customer
SET email     = '110@qq.com',
    cust_name ='望天吧'
WHERE cust_id = 2
end;
```

## 查看触发器

查看触发器列表

```sql
SHOW
TRIGGERS
```

查看触发器创建语句（在MySQL视图中使用）

```sql
SHOW
CREATE TRIGGER testTrigger;
```

值得注意的是，创建的所有触发器都会保存到一张表中，具体位置在：Information_schema.triggers

## 使用触发器

触发器不需要手动触发，当某个事件被激活后，触发器会被自动激活并执行

使用触发器只需要对触发器指定的表进行相关操作的时候，他就会自动触发

触发器千万要注意的一点就是，尽量避免触发器指定的表和触发动作操作的表为同一张表，容易陷入死循环

## 删除触发器

触发器不能修改，只能先删除后新增

```sql
drop trigger <触发器名>
```

## 触发器记录

不管触发器是否触发了，只要当某种操作准备执行，系统就会将当前要操作的记录的当前状态和即将执行之后的新状态分别保留下来，供触发器使用。其中，要操作的当前状态保存到old中，操作之后可能的形态保存到new中。

lod表示的是旧纪录，new表示的是新纪录。

删除的时候是没有new的，插入的时候没有old。

使用方式

```sql
old
.
字段名
new.字段名
```

