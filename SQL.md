# SQL

## 第一章 数据库概念

### 1.1 数据库的相关概念

- DB（Database）：数据库。存储数据的仓库，保存一系列有组织的数据。
- DBMS（Database Management System）：数据库管理系统。操作和管理数据库的软件。
- SQL（Structured Query Language）：结构化查询语言。操作数据库的语言



### 1.2 常见的数据库管理系统

- Oracle，MySQL，MS SQL Server，PostgreSQL，MongoDB，DB2，Redis等。



### 1.3 关系型数据库与非关系型数据库

- 关系型数据库
  - 本质：采用关系模型（二维表格模型）存储数据
  - 优点
    - 支持复杂查询，灵活性强
    - 支持事务，安全性能高
- 非关系型数据库
  - 本质：采用键值对存储数据
  - 优点
    - 数据模型简单，性能高
    - 数据耦合度低，扩展性强



### 1.4 关系型数据库的设计原则

- 函数依赖
  1. 完全函数依赖：c=f(a,b)，称c完全依赖于a,b
  2. 部分函数依赖：c=f(a)，称c部分依赖于a,b
  3. 传递函数依赖：c=f(b)，b=f(a)，称c传递依赖于a
- 关系型数据库遵循三范式
  1. 列不可拆分：对于数据库中一张表的每一个列（字段），其代表的信息都应细化到不可继续拆分的程度。
  2. 不能存在部分函数依赖：表中的每个的字段完全依赖于主键。
  3. 不能存在传递函数依赖：表中多个字段含义与主键字段含义直接相关，不存在间接关系。



### 1.5 MySQL数据库介绍

- **MySQL**是一种开放源代码的关系型数据库管理系统，开发者为瑞典MySQL AB公司。在2008年1月16号被Sun公司收购。而2009年，SUN又被Oracle收购。
- 目前，MySQL被广泛地应用在Internet上的中小型网站中，分为社区版和商业版。由于其**体积小、速度快、总体拥有成本低，尤其是开放源码这一特点，使得很多互联网公司选择了MySQL作为网站数据库**。



## 第二章 SQL概述

### 2.1 SQL语法规范

1. mysql对SQL语句不区分大小写，SQL关键字尽量大写

2. SQL语句在编写时换行不影响执行结果，为提高可读性需要适当分行

3. 关键字不能分行

4. 除数值型的数据（字符串、日期）均使用单引号''

5. 别名使用双引号""

6. 注释

   ```mysql
   #单行注释
   -- 单行注释
   /*
   多行注释
   */
   ```

   

### 2.2 SQL命名规则

1. 数据库名、表名不能超过30个字符，变量名不能超过29个字符
2. 只能包含A-Z，a-z，0-9，_
3. 不能出现重名
4. 不能与关键字重名，mysql中可使用飘号区分字段与关键字



### 2.3 SQL分类

- DDL：数据定义语言（CREATE，ALTER，DROP）
- DML：数据操作语言（INSERT，DELETE，UPDATE，SELECT）
- DCL：数据控制语言（COMMIT，ROLLBACK，SAVEPOINT，GRANT）



## 第三章 查询

### 3.1 基本查询（select from）

```mysql
#查看表结构
describe emp;

#查询指定表中指定字段
select name, salary from emp;

#查询指定表全表
select * from emp;

#查询指定列,并重命名(as可省略)
select name as emp_name from emp;
#若别名包含空格或需要区分大小写可以使用""
select name as "Emp Name" from emp;

#对查询结果去重
select distinct department_id from emp;
```



### 3.2 过滤查询（where）

- where

  ```mysql
  select id, name, department_id
  from emp
  where department_id = 90;
  ```

- between and

  ```mysql
  select id, name, salary
  from emp
  where salary between 3000 and 5000;
  #包含边界
  ```

- in

  ```mysql
  select id, name, department_id
  from emp
  where department_id in (10, 20, 30);
  ```

- like

  ```mysql
  #查询姓名第二位字符为a的所有员工
  select id, name, department_id
  from emp
  where name like '_a%'
  
  #转义字符'\',查询姓名以'_a'开头的所有员工
  select id, name, department_id
  from emp
  where name like '\_a%';
  
  #反转义escape,查询姓名以'$_a'开头的所有员工
  select id, name, department_id
  from emp
  where name like '$_a%' escape '$';
  ```

- null

  ```mysql
  select id, name, department_id
  from emp
  where department_id is null
  ```

- and（且） / or（或） / not（非） / xor（异或） 



### 3.3 排序和分页（order by /  limit）

- order by

  ```mysql
  #升序排列(默认)
  select last_name, job_id, department_id, hire_date
  from employees
  order by hire_date asc;
  
  #降序排列
  select last_name, job_id, department_id, hire_date
  from employees
  order by hire_date desc;
  ```

- limit

  ```mysql
  #显示前三条记录
  select * from emp limit 3;
  
  #显示第1-10条记录
  select * from emp limit 0,10;
  
  #显示第21-30条记录
  select * from emp limit 20,10;
  ```



### 3.4 多表查询（join on）

```mysql
#内连接
select e.ename, d.deptno, d.dname
from emp e
join dept d
on e.deptno = d.deptno;

#左外连接
select e.ename, d.deptno, d.dname
from emp e
left join dept d
on e.deptno = d.deptno;

#右外连接
select e.ename, d.deptno, d.dname
from emp e
right join dept d
on e.deptno = d.deptno;

#满外连接(mysql中不支持,hive中支持)
select e.ename, d.deptno, d.dname
from emp e
full join dept d
on e.deptno = d.deptno;
```



### 3.5 单行函数

#### 3.5.1 字符串函数

| 函数                            | 用法                                                         |
| ------------------------------- | ------------------------------------------------------------ |
| CONCAT(S1,S2,......,Sn)         | 连接S1,S2,......,Sn为一个字符串                              |
| CONCAT_WS(s, S1,S2,......,Sn)   | 同CONCAT(s1,s2,...)函数，但是每个字符串之间要加上s           |
| CHAR_LENGTH(s)                  | 返回字符串s的字符数                                          |
| LENGTH(s)                       | 返回字符串s的字节数，和字符集有关                            |
| INSERT(str, index , len, instr) | 将字符串str从第index位置开始，len个字符长的子串替换为字符串instr |
| UPPER(s) 或 UCASE(s)            | 将字符串s的所有字母转成大写字母                              |
| LOWER(s)  或LCASE(s)            | 将字符串s的所有字母转成小写字母                              |
| LEFT(s,n)                       | 返回字符串s最左边的n个字符                                   |
| RIGHT(s,n)                      | 返回字符串s最右边的n个字符                                   |
| LPAD(str, len, pad)             | 用字符串pad对str最左边进行填充，直到str的长度为len个字符     |
| RPAD(str ,len, pad)             | 用字符串pad对str最右边进行填充，直到str的长度为len个字符     |
| LTRIM(s)                        | 去掉字符串s左侧的空格                                        |
| RTRIM(s)                        | 去掉字符串s右侧的空格                                        |
| TRIM(s)                         | 去掉字符串s开始与结尾的空格                                  |
| TRIM(BOTH s1 FROM s)            | 去掉字符串s开始与结尾的s1                                    |
| TRIM(LEADING s1 FROM s)         | 去掉字符串s开始处的s1                                        |
| TRIM(TRAILING s1 FROM s)        | 去掉字符串s结尾处的s1                                        |
| REPEAT(str, n)                  | 返回str重复n次的结果                                         |
| REPLACE（str, a, b）            | 用字符串b替换字符串str中所有出现的字符串a                    |
| STRCMP(s1,s2)                   | 比较字符串s1,s2                                              |
| SUBSTRING(s,index,len)          | 返回从字符串s的index位置其len个字符                          |

#### 3.5.2 数值函数

| 函数          | 用法                                 |
| ------------- | ------------------------------------ |
| ABS(x)        | 返回x的绝对值                        |
| CEIL(x)       | 返回大于x的最小整数值                |
| FLOOR(x)      | 返回小于x的最大整数值                |
| MOD(x,y)      | 返回x/y的模                          |
| RAND()        | 返回0~1的随机值                      |
| ROUND(x,y)    | 返回参数x的四舍五入的有y位的小数的值 |
| TRUNCATE(x,y) | 返回数字x截断为y位小数的结果         |
| SQRT(x)       | 返回x的平方根                        |
| POW(x,y)      | 返回x的y次方                         |

#### 3.5.3 日期函数

| 函数                                                         | 用法                                                      |
| ------------------------------------------------------------ | --------------------------------------------------------- |
| **CURDATE()** 或 CURRENT_DATE()                              | 返回当前日期                                              |
| **CURTIME()** 或 CURRENT_TIME()                              | 返回当前时间                                              |
| **NOW()** / SYSDATE() / CURRENT_TIMESTAMP() / LOCALTIME() / LOCALTIMESTAMP() | 返回当前系统日期时间                                      |
| **YEAR(date) / MONTH(date) / DAY(date) / HOUR(time) / MINUTE(time) / SECOND(time)** | 返回具体的时间值                                          |
| WEEK(date) / WEEKOFYEAR(date)                                | 返回一年中的第几周                                        |
| DAYOFWEEK(date)                                              | 返回周几，注意：周日是1，周一是2，。。。周六是7           |
| WEEKDAY(date)                                                | 返回周几，注意，周1是0，周2是1，。。。周日是6             |
| DAYNAME(date)                                                | 返回星期：MONDAY,TUESDAY.....SUNDAY                       |
| MONTHNAME(date)                                              | 返回月份：January，。。。。。                             |
| DATEDIFF(date1,date2) / TIMEDIFF(time1, time2)               | 返回date1 - date2的日期间隔 / 返回time1 - time2的时间间隔 |
| DATE_ADD(datetime, INTERVAL  expr type)                      | 返回与给定日期时间相差INTERVAL时间段的日期时间            |
| DATE_FORMAT(datetime ,fmt)                                   | 按照字符串fmt格式化日期datetime值                         |
| STR_TO_DATE(str, fmt)                                        | 按照字符串fmt对str进行解析，解析为一个日期                |

#### 3.5.4 流程函数

| 函数                                                         | 用法                                         |
| ------------------------------------------------------------ | -------------------------------------------- |
| IF(value,t ,f)                                               | 如果value是真，返回t，否则返回f              |
| IFNULL(value1, value2)                                       | 如果value1不为空，返回value1，否则返回value2 |
| CASE WHEN 条件1 THEN result1 WHEN 条件2 THEN result2 .... [ELSE resultn] END | 相当于Java的if...else if...else...           |
| CASE  expr WHEN 常量值1 THEN 值1 WHEN 常量值1 THEN 值1 .... [ELSE 值n] END | 相当于Java的switch...case...                 |

```mysql
select last_name, job_id, salary,
       case job_id when 'IT_PROG'  then  1.10 * salary
                   when 'ST_CLERK' then  1.15 * salary
                   when 'SA_REP'   then  1.20 * salary
       			  else salary END "REVISED_SALARY"
FROM employees;
```



### 3.6 分组查询与函数

- 分组函数

  ```mysql
  #求总行数count
  select count(*) cnt from emp;
  
  #求最大值max
  select max(sal) max_sal from emp;
  
  #求最小值min
  select min(sal) min_sal from emp;
  
  #求和sum
  select sum(sal) sum_sal from emp;
  
  #求平均值avg
  select avg(sal) avg_sal from emp;
  ```

- group by

  ```mysql
  select department_id, avg(salary)
  from employees
  where avg(salary) > 8000
  group by department_id;
  ```

- having

  ```mysql
  select department_id, max(salary)
  from employees
  group by department_id
  having max(salary)>10000 ;
  ```



## 第四章 子查询

### 4.1 基本介绍

- SQL中的查询可以嵌套使用，被嵌套的查询称为子查询，子查询在主查询前被执行，其结果被主查询调用
- 子查询的类型
  - 单行子查询：查询结果为一个字段的一行内容的子查询
  - 多行子查询：查询结果为一个字段的多行内容的子查询
  - 多列子查询：查询结果为多个字段的一或多行内容的子查询
- 格式要求
  1. 子查询需要用()整体括起
  2. 子查询结果作为比较条件时，必须放在右侧
  3. 单行子查询和多行子查询需要使用各自对应的比较符



### 4.2 单行子查询

- 单行子查询的比较符包括：=，>，<，>=，<=，<>

- 示例

  ```mysql
  #查询工资大于Abel的员工姓名和工资
  select last_name, salary
  from employees
  where salary > (select salary 
  				from employees
  				where last_name = 'Abel');
  				
  #查询job_id与141号员工相同,salary比143号员工多的员工姓名,job_id和工资
  select last_name, job_id, salary
  from employees e 
  where job_id = (select job_id
  				from employees e2 
  				where employee_id = 141)
  and salary > (select salary
  			from employees e3
  			where employee_id = 143);
  				
  #查询最低工资大于50号部门最低工资的部门id和其最低工资
  select department_id, min(salary) min_sal
  from employees e 
  group by department_id 
  having min_sal > (select min(salary)
  				from employees e2 
  				where department_id = 50);
  ```



### 4.3 多行子查询

- 多行子查询的比较符

  - IN：等于查询结果中的任意一个
  - ANY：与查询结果中的每一个值比较，仅需一个值满足条件
  - ALL：与查询结果中的每一个值比较，所有值都要满足条件

- 示例

  ```mysql
  #查询比job_id为‘IT_PROG’的员工中任意一人工资低的其他job员工的员工号、姓名、job_id及salary
  select employee_id, last_name, job_id, salary
  from employees e 
  where salary < 	any	(select salary
  					from employees e2 
  					where job_id = 'IT_PROG')
  and job_id != 'IT_PROG';
  
  #查询比job_id为‘IT_PROG’的员工中所有人工资都低的其他job员工的员工号、姓名、job_id及salary
  select employee_id, last_name, job_id, salary
  from employees e 
  where salary < 	all	(select salary
  					from employees e2 
  					where job_id = 'IT_PROG')
  and job_id != 'IT_PROG';
  ```



### 4.4 相关子查询

- 不同于普通的嵌套子查询，相关子查询是在子查询的逻辑中中使用了主查询的字段，因此对于主查询的每一行结果，都需要执行一次子查询的过程

- 示例

  ```mysql
  #查询员工中工资大于本部门平均工资的员工的last_name,salary和其department_id
  select last_name, salary, department_id 
  from employees e 
  where salary > (select avg(salary)
  				from employees e2 
  				where e2.department_id = e.department_id);
  ```

- exists操作符：查询结果不为空时返回true，查询结果为空时返回flase

  ```mysql
  #查询公司管理者的employee_id, last_name, department_id
  select employee_id, last_name, department_id 
  from employees e 
  where exists (select 1
  			from employees e2
  			where e2.manager_id = e.employee_id);
  			
  #查询没有员工的部门号和部门名
  select department_id, department_name 
  from departments d 
  where not exists (select 1
  				from employees e2 
  				where e2.department_id = d.department_id);
  ```



### 4.5 相关更新和相关删除

- 使用相关子查询更新表数据

  ```mysql
  #使用departments表中的部门名更新employees表的部门名
  update employees e
  set department_name = (select department_name 
  						from departments d
  						where d.department_id = e.department_id);
  ```

- 使用相关子查询删除表数据

  ```mysql
  #删除employees表中与emp表重复的员工信息
  delete from employees e
  where employee_id = (select employee_id 
  					from emp e2
  					where e.employee_id = e2.employee_id);
  ```



## 第五章 增删改

### 5.1 insert

```mysql
#方式1:向表的指定字段插入数据,未指定字段插入null
insert into departments(department_id, department_name, manager_id, location_id)
values (70, 'Pub', 100, 1700),(80, 'Boss', 200, 2700);

#方式2:不指定字段,为所有字段赋值并插入
insert into departments
values (70, 'Pub', 100, 1700);

#方式3:从其他表查找指定字段插入目标表,需要保证两表字段类型一一对应
insert into emp2 
select * from employees where department_id = 90;

#注意:操作完成后需要进行提交
commit;
```



### 5.2 update

```mysql
#更新指定行数据
update employees
set department_id = 70
where employee_id = 113;

#更新指定字段的所有行的数据
update employees
set department_id = 70;

#注意:操作完成后需要进行提交
commit;
```



### 5.3 delete

```mysql
#删除指定行
delete from departments
where department_name = 'Finance';

#删除全表数据
delete from departments;

#注意:操作完成后需要进行提交
commit;
```



## 第六章 DDL操作

### 6.1 MySQL的数据类型

| 数据类型      | 描述                                                         |
| ------------- | ------------------------------------------------------------ |
| INT           | 从-2^31到2^31-1的整型数据。存储大小为 4个字节                |
| CHAR(size)    | 定长字符数据。若未指定，默认为1个字符，最大长度255           |
| VARCHAR(size) | 可变长字符数据，根据字符串实际长度保存，**必须指定长度**     |
| FLOAT(M,D)    | 单精度，M=整数位+小数位，D=小数位。 D<=M<=255,0<=D<=30，默认M+D<=6 |
| DOUBLE(M,D)   | 双精度。D<=M<=255,0<=D<=30，默认M+D<=15                      |
| DATE          | 日期型数据，格式’YYYY-MM-DD’                                 |
| BLOB          | 二进制形式的长文本数据，最大可达4G                           |
| TEXT          | 长文本数据，最大可达4G                                       |



### 6.2 操作数据库

```mysql
#创建数据库
create database employees;

#查看所有数据库
show databases;

#使用指定的数据库
use employees;

#查看正在使用的数据库
select database() from dual;

#查看指定数据库下的所有表
show tables from employees;

#删除数据库
drop database employees;
```



### 6.2 操作表

#### 6.3.1 创建表

```mysql
#方式1:直接创建,自定义字段类型
CREATE TABLE dept(
deptno INT(2),
dname VARCHAR(14),
loc VARCHAR(13));

#方式2:使用现有表结构创建
create table emp1 
as 
select * from employees where 1=2;
```



#### 6.3.2 修改表

```mysql
#追加列
alter table dept
add job_id varchar(15);

#修改列
alter table dept
modify (salary double(9,2));

#更改字段位置
alter table customers
modify c_contact varchar(50) after c_birth;

#重命名字段,支持修改字段类型
alter table dept
change department_name dept_name varchar(20);

#删除字段
alter table dept 
drop column job_id;

#重命名表
#方式1
rename table emp to myemp;
#方式2
alter table emp rename myemp;
```



#### 6.3.3 删除表

```mysql
#删除表
drop table dept;

#清空表
truncate table dept;
```



## 第七章 约束（constraint）

### 7.1 约束概述

- 定义：为了保证表中数据的一致性和完整性，SQL以约束的方式对表数据进行额外的条件限制
- 约束可以在创建表时规定，也可以在创建表后添加
- 约束的分类
  - 根据约束的字段数
    1. 单列约束：约束单个字段
    2. 多列约束：约束多个字段
  - 根据约束的作用范围
    1. 列级约束：作用于单列，可在字段后定义
    2. 表级约束：作用于多列，单独定义
  - 根据约束的具体作用
    1. not null 非空约束，约束字段不能为空
    2. unique 唯一约束，约束字段值在表中唯一，不能重复，但可以多行为空
    3. primary key 主键约束，非空且唯一
    4. foreign key 外键约束
    5. check 检查约束
    6. default 默认值约束



### 7.2 not null

- not null约束是列级约束，被约束的列字段不能为空

- 创建表时添加约束

  ```mysql
  #创建列级非空约束
  create table emp(
  	id int(10) not null,
  	name varchar(20) not null,
  	sex char null
  );
  ```

- 添加和取消约束

  ```mysql
  #添加列级非空约束
  alter table emp
  modify first_name varchar(25) not null
  
  #取消列级非空约束
  alter table emp
  modify first_name varchar(25) null
  ```



### 7.3 unique

- 说明

  - unique约束可以是列级约束（单列唯一），也可以是表级约束（多列组合唯一）
  - **约束名如果不指定，默认使用列名，如果约束了多列，默认使用第一个列的列名**
  - mysql中会为唯一约束的列创建一个唯一索引

- 创建表时添加约束

  ```mysql
  #创建列级唯一约束
  create table emp(
  	id int(10),
  	name varchar(20) unique,
  	sex char
  );
  	
  #创建表级唯一约束(约束名为id_name, 作用列为id, name)
  create table emp(
  	id int(10),
  	name varchar(20),
  	sex char,
      constraint id_name unique(id, name)
  );
  ```

- 添加和取消约束

  ```mysql
  #添加列级唯一约束
  alter table emp
  modify phone_number varchar(20) unique
  #添加表级唯一约束
  alter table emp
  add constraint first_last_name unique(first_name,last_name)
  
  #删除唯一约束(必须通过索引名也就是约束名删除)
  alter table emp 
  drop index first_last_name
  #查看表的索引所有索引
  show index from emp
  ```



### 7.4 primary key

- 说明

  - 主键约束等同于**非空+唯一**约束的效果，一个表只能有1个主键约束
  - 主键约束可以是列级约束（指定单个字段），也可以是表级约束（指定多个字段组合）
  - **当主键约束作用于多个字段时，每个字段都都不允许空值，并且组合值唯一**
  - 主键在创建时会默认创建主键索引，删除主键时索引也会删除

- 创建表时添加约束

  ```mysql
  #创建列级主键约束
  create table emp(
  	id int(10) primary key auto_increment,
  	name varchar(20),
  	sex char
  );
  
  #创建表级主键约束
  create table emp(
  	id int(10),
  	name varchar(20),
  	sex char,
      constraint emp_pk primary key(id, name)
  );
  ```

- 添加和取消约束

  ```mysql
  #添加列级约束
  alter table emp
  modify employee_id int(6) primary key
  #添加表级约束
  alter table emp 
  add primary key(employee_id)
  
  #取消主键约束(主键约束取消后,非空约束依然存在)
  alter table emp 
  drop primary key;
  ```



### 7.5 foreign key

- 说明

  - **外键约束用于关联两个表，保证数据的一致性和完整性，从表中外键约束字段的值在主表中有相应记录时，主表中的该条记录不允许被删除**
  - 一个表可以有多个外键约束
  - **外键约束列的参照列，必须具有主键或唯一约束**
  - 外键列的字段名可以与参照列不同，但**数据类型必须一致**
  - 外键约束在创建时，会默认创建一个索引名与列名相同的索引

- 创建表时添加约束

  ```mysql
  #创建表级外键约束
  create table emp(
      id int(10),
      name varchar(20),
      department_id int(10),
      constraint emp_dept_id_fk foreign key(department_id) references dept(department_id)
  );
  ```

- 添加和取消约束

  ```mysql
  #添加表级外键约束
  alter table emp
  add constraint emp_dept_id_fk foreign key(department_id) references dept(department_id)
  
  #取消外键约束
  alter table emp
  drop foreign key emp_dept_id_fk
  #外键索引需要单独删除
  alter table emp
  drop index department_id
  ```



#### 7.5.1 级联删除

- 级联删除：当主表中的参照列被删除时，从表中的外键列也被删除
- 级联置空：当主表中的参照列被删除时，从表中的外键列的值置空

```mysql
#级联删除 on delete cascade
create table emp(
    id int(10),
    name varchar(20),
    department_id int(10),
    constraint emp_dept_id_fk foreign key(department_id) 
    references dept(department_id) on delete cascade
);

#级联置空 on delete set null
create table emp(
    id int(10),
    name varchar(20),
    department_id int(10),
    constraint emp_dept_id_fk foreign key(department_id) 
    references dept(department_id) on delete set null
);
```



#### 7.5.2 级联修改

- 级联修改：当主表中的参考列的值被修改时，从表中外键列的值也做相同的修改
- 级联置空：当主表中的参考列的值被修改时，从表中外键列的值置空

```mysql
#级联修改 on update cascade
create table emp(
    id int(10),
    name varchar(20),
    department_id int(10),
    constraint emp_dept_id_fk foreign key(department_id) 
    references dept(department_id) on update cascade
);

#级联置空 on update set null
create table emp(
    id int(10),
    name varchar(20),
    department_id int(10),
    constraint emp_dept_id_fk foreign key(department_id) 
    references dept(department_id) on update set null
);
```



### 7.6 check

- check约束可以是列级或表级约束，在向表中插入一条数据时，若不满足check条件，则插入失败

- 创建表时添加约束

  ```mysql
  #创建列级检查约束
  create table emp(
  	id int(10),
  	name varchar(20),
  	age int check(age > 20)
  );
  ```



### 7.7 default

- default约束是列级约束，在向表中插入一条数据时，若没有显示赋值约束字段，则赋值为默认值

- 创建表时添加约束

  ```mysql
  #创建列级默认约束
  create table emp(
  	id int(10),
  	name varchar(20),
  	salary double(10,2) default 2000
  );
  ```



## 第八章 其他数据库对象

### 8.1 常见的数据库对象

| 对象            | 描述                             |
| --------------- | -------------------------------- |
| 表(TABLE)       | 基本的数据存储集合，由行和列组成 |
| 视图(VIEW)      | 从表中抽出的逻辑上相关的数据集合 |
| 序列(SEQUENCE)  | 提供有规律的数值                 |
| 索引(INDEX)     | 提高查询的效率                   |
| 同义词(SYNONYM) | 给对象起别名                     |



### 8.2 视图（view）

- 视图的介绍

  - 视图是在已有表的基础上，使用指定的查询逻辑形成的虚表
  - 视图所依赖的表称为**基表**，**对视图的增删改操作会作用到基表**
  - 视图可以理解为持久化的查询语句
  - 视图的作用
    1. 控制数据的访问
    2. 简化查询
    3. 避免数据的重复访问

- 创建视图

  ```mysql
  #创建视图
  create view emp_view
  as
  select employee_id, last_name, salary
  from employees e;
  
  #查看视图结构
  desc emp_view;
  ```

- 修改视图

  ```mysql
  #修改视图(覆盖原视图)
  create or replace view emp_view
  as
  select employee_id, last_name, salary, department_id
  from employees e;
  ```

- 删除视图

  ```mysql
  drop view emp_view;
  ```

- 视图的DML操作

  - select：与操作表的方式完全一致
  - insert：当视图定义时包含以下元素则不可使用
    - 组函数，group by子句，distinct关键字，rounum伪列
    - 自定义表达式类型的字段（在基表中无相应列）
    - 视图中缺少基表中带有非空约束的列
  - delete：当视图定义时包含以下元素则不可使用
    - 组函数，group by子句，distinct关键字，rounum伪列
  - update：当视图定义时包含以下元素则不可使用
    - 组函数，group by子句，distinct关键字，rounum伪列
    - 自定义表达式类型的字段（在基表中无相应列）



### 8.3 索引（index）

- 索引的介绍

  - 索引是脱离于表存储的的数据库中单独的结构，作用是**加速查询速度**，副作用是降低增删改速度
  - 索引建立后，数据库会自动对其进行维护，在使用时无需声明，自动生效
  - 表被删除时，所有基于该表的索引会被连带删除
  - mysql中索引的类型
    - Hash
    - BTree

- 查看索引

  ```mysql
  show index from emp;
  ```
  
- 创建索引

  ```mysql
  #手动创建索引,可创建在一个或多个列上
  create index emp_name_idx
  on emp(last_name);
  
  #自动创建索引
  #在声明主键,外键,唯一约束时,系统会自动创建索引
  ```

- 删除索引

  ```mysql
  #方式1
  drop index emp_name_idx
  on emp;
  #方式2
  alter table emp 
  drop index emp_name_idx
  ```



## 第九章 数据库事务

### 9.1 事务介绍

- 事务：一组逻辑操作单元，使数据从一种状态变换到另一种状态
- 数据库事务包含以下几种
  - 一或多个DML语句
  - 一个DDL语句
  - 一个DCL语句



### 9.2 事务的处理

- 事务处理：保证所有事务都作为一个工作单元执行。在一个事务中执行多个操作时，要么所有的操作都提交，所有修改得到了永久的保存；要么放弃所做的所有修改，整个事务回滚到最初的状态。
- 事务的过程
  1. 设置自动提交规则 set autocomit = false
  2. 事务以一个DML语句的执行作为开始
  3. 事务以以下操作之一作为结束
     - commit和rollback语句
     - DDL语句
     - 用户会话结束/系统异常终止



### 9.3 事物的ACID属性

1. **原子性（Atomicity）**
   事务是一个不可分割的工作单位，事务中的操作要么都发生，要么都不发生。 

2. **一致性（Consistency）**
   事务必须使数据库从一个一致性状态变换到另外一个一致性状态。

3. **隔离性（Isolation）**
   指一个事务的执行不能被其他事务干扰，即一个事务内部的操作及使用的数据对并发的其他事务是隔离的，并发执行的各个事务之间不能互相干扰。

4. **持久性（Durability）**
   持久性是指一个事务一旦被提交，它对数据库中数据的改变就是永久性的，接下来的其他操作和数据库故障不应该对其有任何影响



### 9.4 数据库的隔离级别

- 对于多个同时运行的事务，若没有正确的设置数据库的隔离机制，会导致以下问题

  1. 脏读：A事务读取了B事务过程中尚未提交的数据，而B事务最终未提交，A读取的到就是临时的数据
  2. 不可重复读：A事务读取数据后，B事务对数据进行了更改，A事务再次读取这部分数据时与上一次结果不同
  3. 幻读：A事务读取数据后，B事务向表中插入了符合A读取条件的数据，A再次读取时比上一次数据量大

- MySQL中的隔离级别

  - READ UNCOMMITTED：三种问题都可能出现
  - READ COMMITTED：可能出现不可重复读和幻读的问题
  - REPEATABLE READ（默认）：可能出现幻读
  - SERIALIZABLE：为数据加锁，不会出现上述问题，但会导致多事务时运行效率低下

- 设置隔离级别

  1. 查看隔离级别

     ```mysql
     #查看当前连接的隔离级别
     select @@tx_isolation;
     #查看系统的全局隔离级别
     select @@global.tx_isolation;
     ```

  2. 设置隔离级别

     ```mysql
     #设置当前连接的隔离级别
     set tx_isolation ='repeatable-read';
     #设置系统的全局隔离级别
     set global tx_isolation ='read-committed';
     ```

     

  