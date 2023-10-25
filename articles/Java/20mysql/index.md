# 1  MySQL安装

MySQL安装

1. 下载并运行”mysql-5.5.40-win64.msi“；
2. 选择安装类型，“Custom”用户自定义；
3. 修改安装路径，“d:\\MySQL\MySQL Server 5.0”；
4. Install
5. 选择“进入MySQL配置向导”，并单击“Finish”；
6. 选择配置方式，“Detailed Configuration”手动配置；
7. 选择服务器类型，“Developer Machine”开发者电脑，占用内存少；
8. 选择数据库用途，“Multifunctional Database”多用途数据库；
9. 选择网站并发连接数，“Manual Setting”手动设置20个；
10. 选择启用TCP/IP连接；选择标准模式（不允许有语法错误）；
11. 选择编码格式，“Manual Select”手动选择utf8；
12. 选择作为windows服务；选择自动加入windows路径；
13. 设置root用户密码，设成"root"；是否远程访问，随意；创建匿名用户，否；
14. Execute，四个对号，说明安装成功；
15. 否则卸载重装

MySQL服务命令启动与关闭：

1. win命令窗口启动服务：`net start mysql` 
2. win命令窗口关闭服务：`net stop mysql` 

登录MySQL:

1. 在window命令窗口，输入`mysql -uroot -proot`
2. 远程连接：`mysql -h127.0.0.1 -uroot -proot`

卸载：

1. 停止MySQL服务；
2. 卸载MySQL；
3. 删除安装目录下所有文件；
4. 删除c盘ProgramDate目录中MySQL目录（把隐藏文件显示出来）

MySQL安装目录：

1. `bin` 可执行文件
2. `data` 日志、数据
3. `my.ini`  配置文件

MySQL数据目录`C:ProgramData\MySQL\MySQLServer 5\data`:

- 里面的3个文件夹是自带的数据库

安装SQLyog图形客户端：

1. 直接安装就行
2. 安装完成后：新建(名字随便)->设置主机、用户名、密码、端口(其他不用)->连接
3. 输入SQL命令，选中一行，执行，OK.

# 2  MySQL基本语法

SQL是标准，各个数据库语言都有各自的定义。

SQL语句分类：

1. DDL数据定义语言：库、表的增删改查
2. DML数据操纵语言：字段的 增删改
3. DQL数据查询语言：查询表
4. DCL数据控制语言：用户权限设置 GRANT/REVOKE

MySQL语法：

1. 每条语句分好结尾
2. 不区分大小写
3. 三种注释：`--空格`单行注释，`/**/`多行注释，`#`mysql特有注释

## 2.1  DDL语句

**定义数据库：** 查询、创建、修改、删除

查询数据库

1. show databases; 所有
2. show create database test; 查询某个数据库定义信息

创建数据库

1. create database db1;
2. create database if not exists db2;
3. create database db3 default character set gbk; 设置编码

修改数据库

1. alter database db3 character set utf8;

删除数据库

1. drop database db2;

使用数据库

- use db1;

查看正在使用的数据库

- select database();

**定义表：**  

创建表

- `create table table_name(字段名1 类型, 字段名2 类型);`
    - 字段有：int/double/varchar/date/等
- `create table 新表名 like 旧表名` 

查看表

- show tables; 所有
- desc table_name; 查看表结构 
- show create table table_name; 查看创建表的SQL语句

删除表：

- `drop table table_name`;
- `drop table if exists table_name`;

修改表：

- `alter table 表明 add 列名 类型;` 
- `alter table 表明 modify 列名 新的类型;` 
- `alter table 表明 change 列名 新的列名 类型;` 
- `alter table 表明 drop 列名;` 
- `rename table 表明 to 新表名;` 
- `alter table 表明 character set 字符集;` 

## 2.2  DML语句

添加

- `insert into student values (1, '张三', 18, null, null);`  
- `insert into student(id, name) values (2, '李四')`  
- `insert into student select * from student2;` 
- `insert into 表1(列1，列2，) select 列1，列2 from 表2;`  

修改

1. `update student set score=100 at age=18;`  

删除

1. `delete from student where age=100;`   条件删除
2. `delete from student;` 清空表（一条一条删除，速度慢）
3. `truncate table student;` 清空表（删除表并创建一个一模一样的空表） 

## 2.3  DQL语句

简单查询

1. `select distinct 列名 from 表名;` 显示不重复的数据
2. `select score+10 from student;` 将结果运算
3. where语句中的运算符
    - `> < <= >= = <> !=` 后面两个都是不等于
    - `where score between 80 and 100;` 
    - `where age in(18, 19, 20);` 
    - like '张%'  模糊查询%表示任意多个字符，_表示一个字符
    - is null 
    - and or not 逻辑运算符

排序查询

1. `select * from db1 order by score asc;` 升序排列
2. `select * from db1 order by score desc;` 降序排列
3. `select * from db1 order by score asc, sex desc;` 第二排序条件

聚合查询：max, min, avg, count, sum

1. `select ifnull(id, 0) from student;` 如果为null，使用0代替
2. `select count(ifnull(id,0)) from student;` 统计数量，不遗漏null

分组查询

1. `select sex, avg(score)  from student group by sex;`  
2. `select sex, avg(score) from student group by sex having  avg(score)>60;` 分组结果筛选

分页查询

- `select * from student limit offset,length;` 从第offset行开始的length条数据中查询
- `select * from student limit 10, 5;` 从10-15行的数据中查询 

## 2.4  约束(主键/非空)

1. primary key 主键

    - `alter table stu drop primary key;` 删除主键
    - `alter table stu add primary key(id);` 添加主键
    - `create table stu(id int primary key AUTO_INCREMENT);` 自增长主键
    - `create table stu(...) AUTO_INCREMENT=1000;` 从1000开始自增长主键 

2. unique 唯一

    - `create table stu(name varchar(20) unique) ;` 不能重复，但可以为NULL

3. not null 非空

    - `create table stu(age not null);` 不能为空 

4. foreign key 外键

    - 创建外键约束

    ```
    create table employee(
    id int primary key auto_increment,
    dep_id int,
    /*添加dep_id的外键, 外键名字为emp_fk*/
    constraint emp_fk foreign key(dep_id) references department(id)
    );
    ```

    - ``alter table employee add constraint...;` 和上面一样
    - `alter table employee drop foregin key emp_fk;` 删除外键

5. 级联操作：在修改、删除主表主键时，需要更新或删除 副表的外键

    - `alter table employee add constraint emp_fk foreign key (dep_id) references department(id) on update cascade on delete cascade;` 添加修改、删除级联操作 

# 3  MySQL高级

## 3.1  范式

数据设计规则，称为范式，有六种范式，满足第三范式就可以了。

1. 1NF：每一列就是一列 ，不能拆分成两列 （原子性）
2. 2NF：每一列都完全依赖于主键（不产生局部依赖）
3. 3NF：任何非主列不得传递依赖于主键。（不产生传递依赖）

## 3.2  备份和还原

命令行：

1. 备份：`mysqldump -uroot -proot 数据库名 > d://test.sql`  
2. 还原：
    1. 登录数据库 `mysql -uroot -proot`
    2. 创建数据库`create database db1;`
    3. 使用数据库`use db1;`
    4. 执行文件：`source d://test.sql;`

图形工具SQLyog

1. 备份：数据库右键->备份/导出->转存到SQL->导出
2. 还原：root右键->执行脚本->选择文件

## 3.3  多表查询

多表查询

1. 内连接：隐式内连接、显示内连接
2. 外连接：左连接、右连接

笛卡尔积，`select * from emp,dept;` 导致结果太多，所以要联合查询

隐式内连接：

```
不适用JOIN关键字，使用WHERE指定
SELECT * FROM emp, dept WHERE emp.`dept_id`=dept.`id`;
```

显示内连接：

```
使用 JOIN ... ON 语句
SELECT * FROM emp JOIN dept ON emp.`dept_id`=dept.`id`;
```

左连接

```
使用 LEFT JOIN ... ON
SELECT * FROM emp LEFT JOIN dept ON emp.`dept_id`=dept.`id`;
```

右连接

```
SELECT * FROM emp RIGHT JOIN dept ON emp.`dept_id`=dept.`id`;
```

子查询（嵌套查询）

```
/* 一个结果 */
select * from emp where salary = (select max(salary) from emp);
/* 一列结果 */
select * from dept where id in (select dept_id from emp where salary > 5000);
/* 多行多列 */
select * from (子查询) where 条件;
```



## 3.4  事务

事务执行是一个整体，必须保证所有SQL语句执行成功。如转账，小明账号-500，对方账号+500。

事务的四大特性：

1. 原子性：整体，要么成功，要么失败
2. 一致性：数据库在事务执行前后一致
3. 隔离性：事务之间互不影响
4. 持久性：一旦执行成功，数据库永久保存。

MySQL中操作事务的方式：

1. 手动提交事务
    - 成功：开启事务-》执行SQL语句-》成功提交事务
    - 失败：开启事务-》执行SQL语句-》事务的回滚
2. 自动提交事务：每条SQL语句都是一个事务
3. 设置回滚点 `savepoint point_name;` 和`rollback to point_name;`

例，手动提交事务的两种情况：(在win命令窗口)

```
/* 成功 */
start transaction;
update db1 set money=money-500 where name=‘小明’;
update db1 set money=money+500 where name='老王';
commit;
/* 失败 */
start transaction;
update db1 set money=money-500 where name=‘小明’;
update db1 set money=money+500 where name='老王';
rollback;
```

并发访问数据库，可能存在的问题：

1. 脏读：一个事务读取另一个事务尚未提交的数据
2. 不可重复读：一个事务两次读取的内容不一致（update导致）
3. 幻读：一个事务两次读取的数量不一致（insert/delete导致）

为了解决并发问题，MySQL事务有四种隔离级别，级别越高，安全性越高，性能越差：

1. read uncommitted：默认
2. read committed：避免脏读
3. repeatable read: 避免脏读、不可重复读
4. serializable: 串行化，都可以避免

设置事务隔离级别：

1. 查看级别`select @@tx_isolation;` 
2. 设置级别`set global transaction isolation level read committed;` 
3. 设置后需要重启数据库

## 3.5  DCL语句

创建用户

```
// 创建用户，只能本地登录
create user 'user1'@'localhost' identified by '123';
// 创建用户，可以远程登录
create user 'user2'@'%' identified by '123';
```

查看用户权限：

- `show grants for 'user1'@'localhost';`

新创建用户没有权限，需要授权：

1. `grant 权限1,权限2.. on 数据库.表名 to '用户名'@'主机名';`
2. `grant create,insert,update,delete on db1.* to 'user1'@'localhost';` 
3. `grant all on *.* to 'user2'@'%';` 给用户2分配所有权限

撤销授权：

1. `revoke 权限1,权限2.. on 数据库.表名 from '用户'@'主机';` 
2. `revoke all on db1.* from 'user1'@'localhost';` 撤销用户1在db1上的所有权限

删除用户：`drop user 'user1'@'localhost';` 

修改管理员密码：`mysqladmin -uroot -p password 123456` 在未登录mysql情况下修改的密码

修改普通用户密码：`set password for 'user1'@'localhost'=password('新棉');` 