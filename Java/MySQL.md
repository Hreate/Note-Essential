# MySQL

## 1 索引数据结构

* 索引是帮助 MySQL 高效获取数据的排好序的数据结构

* Hash 哈希散列法

  * 哈希算法（也叫散列），就是把任意长度值（Key）通过散列算法变换成固定长度的 Key 地址，通过这个地址进行访问的数据结构
  * 它通过把关键码值映射到表中一个位置来访问记录，以加快查找的速度
  * 这个映射函数叫做散列函数，存放记录的数组叫做散列表
  * 不足：
    1. 不支持范围查询
    2. 不支持排序

* 二叉树

  * 二叉搜索树又叫二叉查找树、二叉排序树
  * 如果它的左子树不为空，则左子树上结点的值都小于根结点
  * 如果它的右子树不为空，则右子树上结点的值都大于根结点
  * 子树同样也要遵循以上两点

  * 不足：
    1. 根据插入的顺序，极端情况下会不平衡

* 红黑树

  1. 性质

  * 每个结点不是红色就是黑色
  * 不可能有连在一起的红色结点
  * 根结点都是黑色：入度为0
  * 每个红色结点的两个子结点都是黑色
  * 叶子结点都是黑色：出度为0

  2. 旋转和颜色变换规则：所有插入的点默认为红色

  * 变颜色的情况：当前结点的父亲是红色，且它的祖父结点的另一个子结点（叔叔结点）也是红色：

    (1) 把父结点设为黑色

    (2) 把叔叔结点也设为黑色

    (3) 把祖父结点设为红色

    (4) 把指针移动到祖父结点，设为当前要操作的

  * 左旋：当前父结点是红色，叔叔是黑色的时候，且当前的结点是右子树。左旋以父结点作为旋转结点

  * 右旋：当前父结点是红色，叔叔是黑色的时候，且当前的结点是左子树。右旋

    (1) 把父结点变为黑色

    (2) 把祖父结点变为红色

    (3) 以祖父结点作为旋转结点

* B-Tree

  * 每个结点包含
    1. 本结点所含关键字的个数
    2. 指向父结点的指针
    3. 关键字
    4. 指向子结点的指针
  * M阶的B-Tree的几个重要特性：
    1. 结点最多含有m棵子树（指针），m-1个关键字（存的数据，空间）（m>=2）
    2. 除根结点和叶子结点外，其他每个结点至少有 ceil（m/2）个子结点，ceil 向上取整
    3. 若根结点不是叶子结点，则至少有两棵子树

* B+树（MySQL索引使用）

  * 特征

    1. 有m个子树的结点包含有m个元素
    2. 根结点和分支结点中不保存数据，只用于索引，所有数据都保存在叶子结点中
    3. 所有分支结点和根结点都同时存在于子结点中，在子结点元素中是最大或者最小的元素
    4. 叶子结点会包含所有的关键字，以及指向数据记录的指针，并且叶子结点本身是根据关键字的大小从小到大顺序连接

  * 查询 MySQL 索引结点大小

    ```mysql
    show variables like 'innodb_page_size';
    # 或者
    show global status like 'Innodb_page_size';
    ```

    

## 2  MySQL 索引查询原则

最左原则

## 3 MySQL 引擎分类

### 3.1 MyISAM 引擎（非聚集索引方式）

* MyISAM 是 MySQL 的默认数据库引擎（5.5版之前），由早期的 ISAM（Indexed Sequential Access Method：有索引的顺序访问方法）所改良。虽然性能极佳，但却有一个缺点：不支持事务处理（Transaction）
* MyISAM 索引文件和数据文件是分离的（非聚集）
* `data` 文件夹中对应数据库中的文件
  * `.frm` 表的数据定义相关信息
  * `.MYD` 表的数据文件
  * `.MYI` 表的索引文件

### 3.2 InnoDB 引擎（聚集索引方式）

* InnoDB，是 MySQL 的数据库引擎之一，为 MySQL AB 发布 binary 的标准之一，InnoDB 由 Innobase Oy 公司所开发，2006年五月时由甲骨文公司并购。与传统的 MyISAM 相比，InnoDB的最大特色就是支持了 ACID 兼容的事务 （Transaction）功能，类似于PostgreSQL。目前 InnoDB 采用双轨制授权，一是 GPL 授权，另一是专有软件授权
* 聚集索引——叶子结点包含了完整的数据记录
* `data` 文件夹中对应数据库中的文件
  * `.frm` 表的数据定义相关信息
  * `.ibd` 表的数据 + 索引文件



## 4 操作数据库

> 创建数据库

```mysql
create database [if not exists] 数据库名;
```



> 删除数据库

```mysql
drop database [if exists] 数据库名;
```



> 使用数据库

```mysql
use 数据库名;
```



> 查看数据库

```mysql
show databases;
```



> 查看创建数据库的语句

```mysql
show create database 数据库名;
```



> 数据类型

* 数值

  1. 整型

  * TINYINT[(*M*)] [UNSIGNED] [ZEROFILL] M默认为4，占1个字节

    很小的整数。带符号的范围是-128到127。无符号的范围是0到255。

  * SMALLINT[(*M*)] [UNSIGNED] [ZEROFILL] M默认为6，占2个字节

    小的整数。带符号的范围是-32768到32767。无符号的范围是0到65535。

  * MEDIUMINT[(*M*)] [UNSIGNED] [ZEROFILL] M默认为9，占3个字节

    中等大小的整数。带符号的范围是-8388608到8388607。无符号的范围是0到16777215。

  *  INT[(*M*)] [UNSIGNED] [ZEROFILL]  M默认为11，占4个字节

    普通大小的整数。带符号的范围是-2147483648到2147483647。无符号的范围是0到4294967295。
    
  * BIGINT[(*M*)] [UNSIGNED] [ZEROFILL] M默认为20，占8个字节

    大整数。带符号的范围是-9223372036854775808到9223372036854775807。无符号的范围是0到18446744073709551615。

  2. 浮点型

  * float(m,d)，单精度浮点型，8位精度（4个字节）

    m 表示最大长度，d 表示显示的小数位数。
    
  * double(m,d)，双精度浮点型，16位精度（8个字节）

    m 表示最大长度，d 表示显示的小数位数。

  3. 定点数（字符串形式的浮点数）

  * decimal(m,d)
  * numeric 与 decimal 同义
  * m 是最大位数（精度），范围是1到65，默认值是10
  * d 是小数点右边的位数（小数位）。范围是0到30，并且不能大于m，默认值是0
  * float、double等非标准类型，在 DB 中保存的是近似值，而 decimal 则以字符串的形式保存数值

* 字符串

  * char(n)，固定长度，最多255个字符
  * varchar(n)，固定长度，最多65535个字符
  * tinytext，可变长度，最多255个字符
  * text，可变长度，最多65535个字符
  * mediumtext，可变长度，最多2de24次方-1个字符
  * longtext，可变长度，最多2的32次方-1个字符

  

  > char 和 varchar

  1. char(n) 若存入字符数小于 n，则以空格补于其后，查询之时再将空格去掉。所以 char 类型存储的字符串末尾不能有空格，varchar 不限于此
  2. char(n) 固定长度，char(4) 不管是存入几个字符，都将占用4个字节，varchar 是存入的实际字符数 + 1个字节（n <= 255）或2个字节(n > 255)，所以 varchar(4) ，存入3个字符将占用4个字节
  3. char 类型的字符串检索速度要比 varchar 类型快

  

  > varchar 和 text

  1. varchar 可指定 n，text 不能指定，内部存储 varchar 是存入的实际字符数 + 1个字节（n <=255）或2个字节（n > 255），text 是实际字符数 +2 个字节
  2. text 类型不能有默认值
  3. varchar 可直接创建索引，text 创建索引要指定前多少个字符。varchar 查询速度快于 text，在都创建索引的情况下，text 的索引似乎不起作用。

* 时间日期

  * date，yyyy-MM-dd，日期格式
  * time，HH:mm:ss，时间格式
  * datetime，yyyy-MM-dd HH:mm:ss
  * timestamp，时间戳，1970.1.1 到现在的毫秒数
  * year，年份

* null



> 字段属性

* Unsigned：无符号数
* zerofill：0填充，不足的位数使用0进行填充
* auto _increment：自增
* null：可空
* not null：非空
* default：设置默认值



## 5 操作数据库表

> 根据阿里巴巴java开发手册建表规约，每张表必须存在以下5个字段：

* id 主键
* version 乐观锁
* is_deleted 逻辑删除
* gmt_create 创建时间
* gmt_modified 修改时间



> 创建表

```mysql
create table [if not exists] 表名 (
	`字段名` 数据类型 [属性] [索引] [注释],
    ...
    `字段名` 数据类型 [属性] [索引] [注释]
)engine = innodb default character set = utf8mb4;
```

示例：

```mysql
CREATE TABLE `student` (
  `id` int NOT NULL AUTO_INCREMENT COMMENT '学号',
  `name` varchar(30) NOT NULL DEFAULT '' COMMENT '姓名',
  `password` varchar(30) NOT NULL COMMENT '密码',
  `gender` varchar(5) NOT NULL COMMENT '性别',
  `birthday` date NOT NULL COMMENT '出生日期',
  `address` varchar(100) NOT NULL COMMENT '家庭住址',
  `email` varchar(50) NOT NULL COMMENT '邮箱',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='学生表';
```



> 查看创建表的语句

```mysql
show create table 表名;
```



> 查看表的具体结构

```mysql
desc [数据库名.]表名;
```



> 数据库引擎

* InnoDB，默认使用
* MyISAM，5.5 前默认

|              | MyISAM             | InnoDB             |
| ------------ | ------------------ | ------------------ |
| 事务         | 不支持             | 支持               |
| 数据行锁定   | 不支持（为表锁定） | 支持               |
| 物理外键约束 | 不支持             | 支持               |
| 全文索引     | 支持               | 不支持             |
| 表空间大小   | 较小               | 较大，约为前者两倍 |

* 物理空间中的位置：data 目录下



> 修改表名

```mysql
ALTER TABLE 原表名 RENAME AS 新表名;
```



>删除表

```mysql
DROP TABLE [IF EXISTS] 表名;
```



> 添加表字段

```mysql
ALTER TABLE 表名 ADD 字段名 ...属性 [AFTER 字段名];
```



> 修改字段属性

```mysql
ALTER TABLE 表名 MODIFY 字段名 ...属性;
```



> 修改字段名以及属性

```mysql
ALTER TABLE 表名 CHANGE 原字段名 [新字段名] [...属性];
```



> 删除字段

```mysql
ALTER TABLE 表名 DROP 字段名;
```



> 物理外键

* 创建表时

  ```mysql
  KEY 索引名 (字段名),
  CONSTRAINT 约束名 FOREIGN KEY (字段名) REFERENCES 引用表名 (引用字段名),
  ```

* 对已有表进行外键操作

  ```mysql
  -- 增加索引
  alter table 表名 add key 索引名 (字段名);
  -- 或者
  create index 索引名 on 表名(字段名);
  -- 增加外键约束
  alter table 表名 add constraint 约束名 foreign key (字段名) references 引用表名 (引用字段名);
  -- 删除外键约束
  alter table 表名 drop foreign key 约束名;
  -- 删除索引
  alter table 表名 drop key 索引名;
  ```

  

* 被引用的表为主表，引用其它表的表为从表（看外键在哪张表，所在表为从表）



## 6 DML 数据操作语言

> insert

```mysql
insert into 表名 (字段名1,字段名2, ...) values (值1,值2, ...)[,(值1,值2, ...), ...];
```



> update

```mysql
update 表名 set 字段名1=值1[, 字段名2=值2, ...] [where 条件]; -- 如果不指定条件，会更新所有表行！！！
```

条件运算符

* =
* <> 或 !=
* \> < >= <=
* and
* or
* between 值1 and 值2
* in (值1,值2, ...)



> delete

```mysql
delete from 表名 [where 条件]; -- 如果不指定条件，会删除所有表行！！！
```



> truncate

```mysql
truncate 表名;
```

* 删除/截断表中的所有数据，表的结构不会改变
* 会使自动增量归零



## 7 DQL 数据查询语言

> select

```mysql
SELECT [ALL | DISTINCT]
{* | table.* | [table.field[ as alias1][, table.field2[ as alias2]][, ...]]}
FROM table_name[ as table_alias]
[LEFT | RIGHT | INNER JOIN table_name2] -- 联表查询
[ON ...] -- 联表查询需满足的条件
[WHERE ...] -- 指定结果需满足的条件
[GROUP BY ...] -- 指定结果按照哪几个字段来分组
[HAVING ...] -- 过滤分组的记录必须满足的次要条件
[ORDER BY ...] -- 指定查询记录按照一个或多个条件排序
[LIMIT {[offset, ] row_count | row_countOFFSET offset}]; -- 指定查询的记录熊哪条至na'tiao
```

* 起别名，`as` 省略



> 去重

```mysql
select distinct 字段1[, 字段2, ...] from 表名 [where 条件];
```

* 使用 distinct 关键字，放在多个字段最前面，表示多个字段同时不相同
* 使用 group by 也能达到去重的目的



> 表达式

```mysql
select version(); -- 查询系统版本（函数）
select 100*3-1 as 计算结果; -- 计算（表达式）
select @@auto_increment_increment -- 查询自增步长（变量）

-- 查询考试成绩 + 1分
select `id`, `score`+1 as 加分后 from result;
```



### 7.1 where 条件子句

> 逻辑运算符

| 运算符     | 语法               | 描述   |
| ---------- | ------------------ | ------ |
| and    &&  | a and b    a && b  | 逻辑与 |
| or    \|\| | a or b    a \|\| b | 逻辑或 |
| not    !   |                    | 逻辑非 |



> 比较运算符

| 运算符      | 语法                   | 描述                                  |
| ----------- | ---------------------- | ------------------------------------- |
| is null     | a is null              | 若 a 为 null，则结果为真              |
| is not null | a is not null          | 若 a 不为 null，则结果为真            |
| between     | a between b and c      | 若 a 在 b 和 c 之间，则结果为真       |
| like        | a like b               | 若 a 匹配 b，则结果为真               |
| in          | a in (a1, a2, a3, ...) | 如 a 等于括号中的某一个值，则结果为真 |

* `_` ：0到1个字符
* `%`：0到多个字符



### 7.2 联表查询

> 内联接

```mysql
SELECT * FROM a INNER JOIN b ON a.aID =b.bID;
-- 等同于
SELECT * FROM a,b WHERE a.aID = b.bID;
```

只显示出了 A.aID = B.bID 的记录，这说明 inner join 并不以谁为基础，它只显示符合条件的记录



> 左外联接

```mysql
SELECT * FROM a LEFT JOIN b ON a.aID =b.bID;
```

left join 是以 A 表的记录为基础的，A 可以看成左表，B 可以看成右表，left join 是以左表为准的.
换句话说,左表（A）的记录将会全部表示出来，而右表（B）只会显示符合搜索条件的记录（例子中为: A.aID = B.bID）
B表记录不足的地方均为NULL.



> 右外联接

```mysql
SELECT * FROM a RIGHT JOIN b ON a.aID = b.bID;
```

与左外联接相反



> 示例
>
> 查询参加了考试的同学信息：学号、姓名、课程名、分数

```mysql
select r.student_id, s.name, sub.name, r.score
from result r
left join student s
on s.id = r.student_id
left join subject sub
on r.subject_id = sub.id;
```



> 自联接

示例

```mysql
create table `category` (
    `id` int unsigned not null auto_increment comment '主键id',
    `pid` int not null comment '父id',
    `name` varchar(50) not null comment '品牌名',
    primary key (`id`)
) engine=innodb default charset=utf8mb4 collate=utf8mb4_0900_ai_ci comment '品牌表';

insert into `category` (id, pid, name)
values (2, 1, '信息技术'),
       (3, 1, '软件开发'),
       (4, 3, '数据库'),
       (5, 1, '美术设计'),
       (6, 3, 'web开发'),
       (7, 5, 'ps技术'),
       (8, 2, '办公信息');

select a.name 父项目, b.name 子项目
from category a, category b
where a.id = b.pid;
```



### 7.3 排序和分页

>排序

```mysq
order by 字段名 升序/降序
```

* 升序 `asc`
* 降序 `desc`



> 分页

```mysql
limit 偏移量 记录数;
```

第 n 页，偏移量公式：

(n-1) * pagesize

总页数 = (总记录数 + (pagesize - 1)) / pagesize



### 7.4 子查询

本质：在 where 语句中嵌套子查询语句

> 嵌套查询示例

```mysql
select id, name
from student
where id in (
    select student_id 
    from result 
    where score < 80 and subject_id = (
        select id 
        from subject 
        where name='高等数学-1'
        )
    );
```



## 8 MySQL 函数

> 官网

https://dev.mysql.com/doc/refman/8.0/en/sql-function-reference.html



### 8.1 常用函数

```mysql
-- 数学运算
-- 绝对值
select abs(-8);

-- 向上取整
select ceil(9.4);
select ceiling(9.4);

-- 向下取整
select floor(9.6);

-- 随机数
select rand(); -- 返回一个0 ~ 1之间的随机数

-- 正负
select sign(0); -- 返回参数的符号，正数返回1，负数返回-1，0返回0

-- 字符串
-- 字符串长度
select char_length('I am the singer');
select character_length('I am the singer in the band');

-- 拼接字符串
select concat('lying ', 'on ', 'my bed');

-- 替换字符串
select insert('如果没有明天', 1, 2, '真的可以');
select replace('如果没有明天', '如果', '真的可以');

-- 转小写
select lower('ABC');

-- 转大写
select upper('abc');

-- 索引
select instr('wo cao', 'w');

-- 截取子串
select substr('如果没有明天', 4, 2); -- (源字符串, 截取的位置, 截取的长度)
select substring('如果没有明天', 4, 2);

-- 反转
select reverse('如果没有明天');

-- 时间和日期函数
-- 获取当前日期
select curdate();
select current_date();

-- 获取当前时间
select curtime(); -- 参数：int，精确到秒小数点后多少位
select current_time();

-- 获取当前日期及时间
select now(); -- 参数：int，精确到秒小数点后多少位
select current_timestamp();
select localtime();
select localtimestamp();

-- 获取系统时间
select sysdate();

-- 获取年月日时分秒
select year(now());
select month(now());
select day(now());
select hour(now());
select minute(now());
select second(now());

-- 系统
select system_user();
select user();
select version();
```



### 8.2 聚合函数

| 函数名称 | 描述   |
| -------- | ------ |
| count()  | 计数   |
| sum()    | 求和   |
| avg()    | 平均值 |
| max()    | 最大值 |
| min()    | 最小值 |

```mysql
-- 聚合函数
select count(name) from student; -- count(字段)，会忽略 null 值
select count(*) from student; -- count(*)，不会忽略 null 值
select count(1) from student; -- count(1)，不会忽略 null 值

select sum(score) 总分 from result;
select avg(score) 平均分 from result;
select max(score) 最高分 from result;
select min(score) 最低分 from result;
```



示例：

```mysql
-- 查询不同课程的最高分、最低分、平均分 > 80分
select s.name, max(r.score) 最高分, min(r.score) 最低分, avg(r.score) 平均分
from result r, subject s
where r.subject_id = s.id
group by subject_id
having 平均分>80;
```



### 8.3 MD5加密

* md5()



## 9 事务

> 事务原则：ACID 原则

* 原子性（Atomicity）

  一个事务中的所有操作，要么全部完成，要么全部不完成，不会结束在中间某个环节。事务在执行过程中发生错误，会被回滚到事务开始前的状态，就像这个事务从来没有执行过一样

* 一致性（Consistency）

  在事务开始之前和事务结束以后，数据库的完整性没有被破坏。这表示写入的资料必须完全符合所有的预设规则，这包含资料的精确度、串联性以及后续数据库可以自发性地完成预定的工作

* 隔离性（Isolation）

  数据库允许多个并发事务同时对其数据进行读写和修改的能力，隔离性可以防止多个事务并发执行时由于交叉而导致数据的不一致性。事务隔离分为不同级别，包括读未提交（read uncommitted）、读提交（read committed）、可重复读（repeatable read）和串行化（Serializable）

* 持久性（Durability）

  事务处理结束后，对数据的修改就是永久的，即便系统故障也不会丢失



> 事务的并发问题

* 脏读

  是指一个事务读取到了另外一个事务未提交的数据

* 不可重复读

  在一个事务内读取表中的某一行数据，多次读取结果不同

* 虚读（幻读）

  是指在一个事务内读取到了别的事务插入的数据，导致前后读取不一致



> sql

```mysql
-- 事务
-- MySQL 默认开启事务自动提交
-- set autocommit=0 | off | false; -- 关闭自动提交
-- set autocommit=1 | on | true; -- 开启自动提交（默认）
select @@autocommit;

-- 显示开启事务
begin; -- 之后的 sql 都在同一个事务内
start transaction; -- 同上
-- 事务结束，成功 -> 提交，失败 -> 回滚
-- 提交
commit;
commit work; -- 同上
-- 回滚
rollback;
rollback work; -- 同上

-- 事务嵌套
savepoint 保留点名; -- 声明一个保留点
rollback to savepoint 保留点名; -- 回滚到指定保留点
release savepoint 保留点名; -- 删除指定保留点
```



## 10 索引

* 普通索引：key / index
* 主键索引：primary key
* 唯一索引：unique key
* 全文索引：fulltext



> 查看索引

```mysql
show index from 表名;
-- 或者
show keys from 表名;
```



> 创建索引

```mysql
CREATE [UNIQUE|FULLTEXT|SPATIAL] INDEX index_name
 [USING index_type]
 ON table_name (index_col_name,...)
-- 或者
alter table 表名 add index 索引名(字段名);
```



> 删除索引

```mysql
drop index 索引名 on 表名;
-- 或者
alter table 表名 drop index 索引名;
```





> 修改索引

* 对于MySQL 5.7及以上版本,可以执行以下命令（修改索引名）

  ```mysql
  alter table 表名 rename index old_index_name to new_index_name;
  ```

* 对于MySQL 5.7以前的版本，先删除，再创建



> 索引原则

* 索引不是越多越好
* 不要对经常改变的数据创建索引
* 数据量小的表不需要创建索引
* 一般给常用来查询的字段创建索引



## 11 权限管理和备份

### 11.1 DCL

* 用户表：mysql.user
* 本质：对这张表进行增删改查



> 创建用户

```mysql
create user '用户名'@'主机地址' identified with mysql_native_password by '密码';
```



> 修改密码

```mysql
ALTER USER '用户名'@'localhost' IDENTIFIED WITH MYSQL_NATIVE_PASSWORD BY '新密码';
flush privileges;
```



> 给用户重命名

```mysql
rename user 原用户名 to 新用户名;
```



> 授予所有权限

```mysql
GRANT ALL PRIVILEGES ON *.* TO '用户名'@'主机地址';
```



> 查看用户权限

```mysql
show grants for '主机名'@'主机地址';
```



> 撤销全部权限

```mysql
REVOKE ALL PRIVILEGES ON *.* TO '用户名'@'主机地址';
```



> 删除用户

```mysql
drop user 用户名;
```



### 11.2 MySQL 备份

> 为什么要备份

* 保证重要的数据不丢失
* 数据转移 



> 备份方式

* 直接拷贝物理文件
* 利用可视化工具手动导出
* 使用命令行导出：`mysqldump -h主机 -u用户名 -p密码 数据库名 [表名1 表名2 ...] > 物理磁盘位置/文件名`
* 导入
  * 登录的情况下： `source 物理磁盘位置/文件名`
  * 或者 `mysql -u用户名 -p密码 数据库名 < 物理磁盘位置/文件名`



## 12 规范数据库设计

* 分析需求：分析业务和需要处理的数据库的需求
* 概要设计：设计关系图——E-R图



## 13 三大范式

> 第一范式（1NF）

* 要求数据库表的每一列都是不可分割的原子数据项



> 第二范式（2NF）

* 在1NF的基础上，非码属性必须完全依赖于候选码（在1NF基础上消除非主属性对主码的部分函数依赖）
* 第二范式需要确保数据库表中的每一列都和主键相关，而不能只与主键的某一部分相关（主要针对联合主键而言）



> 第三范式（3NF）

* 在2NF基础上，任何非主[属性](https://baike.baidu.com/item/属性)不依赖于其它非主属性（在2NF基础上消除传递依赖）
* 第三范式需要确保数据表中的每一列数据都和主键直接相关，而不能间接相关