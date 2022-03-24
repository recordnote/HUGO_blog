---
title: Oracle PLSQL编程
date: 2022-03-24T14:21:26+08:00
lastmod: 2022-03-24T14:21:26+08:00
author: Aaron
avatar: /me/yy.jpg
cover: /img/MySql.png
images:
  - 
categories:
  - 数据库
tags:
  - 数据库
weight: 1
---

	***Procedure Language 实际上是Oracle对SQL语言的能力扩展,让SQL语言拥有了if条件判断 , for循环等处理。***

### 一、PLSQL基本语法

``` mysql
1 DECLARE 
2     -- 声明部分
3     变量名 变量类型 := 初始值 
4     变量名 emp.sal % TYPE  -- 引用类型的变量
5           emp % rowtype -- 记录型变量           
6 BEGIN
7     -- 业务逻辑
8 END ;
```

#### 1、变量的声明与使用

```mysql
 1 -- 已知数据类型的赋值声明 
 2 DECLARE 
 3     i NUMBER := 100 ;
 4 BEGIN
 5     -- 输出语句相当于 System.out.print();
 6     dbms_output.put_line('Hello World!' || i) ;
 7 END ;
 8 
 9 -- 未知数据类型的类型声明
10 -- 输出7369的工资
11 
12 DECLARE 
13     vsal emp.sal % TYPE ;
14 BEGIN
15     -- 给变量赋值
16     SELECT sal INTO vsal FROM emp WHERE empno = 7369 ; 
17     dbms_output.put_line(vsal) ;
18 END ;
19 
20 -- 记录型变量声明与赋值
21 -- 输出7369的所有信息
22 DECLARE 
23     vrow emp % rowtype ;
24 BEGIN    
25     SELECT * INTO vrow FROM emp WHERE empno = 7369 ;
26     dbms_output.put_line(vrow.empno||'   '|| vrow.ename);
27 END ;
```

#### 2、if条件判断语法与使用

```mysql
 1 -- 根据不同年龄输出信息
 2 DECLARE
 3     -- 由客户端输入     
 4     age number := &aaa;
 5 BEGIN
 6     IF age <= 18 THEN
 7         dbms_output.put_line('未成年人');
 8     ELSIF age > 18 AND age <= 24 THEN
 9         dbms_output.put_line('年轻人');
10     ELSIF age > 24 AND age < 48 THEN
11         dbms_output.put_line('中年人');
12     ELSE 
13         dbms_output.put_line('老年人');
14     END IF;
15 END;
```

#### 3、三种循环

```mysql
 1 /*
 2    三种循环
 3    for 变量名 in 起始值..结束值  loop
 4      
 5    end loop; 
 6   ----------------------------------
 7    while 条件 loop
 8      
 9    end loop;
10   -----------------------------------    
11    loop
12      exit when 退出的条件
13      循环体
14    end loop;
15 */
16 
17 -- for 循环
18 -- 输出1-10
19 DECLARE 
20 
21 BEGIN
22     FOR i IN 1..10 LOOP
23         dbms_output.put_line(i);
24     END LOOP;
25 END;
26 -- 输出10-1
27 DECLARE 
28 
29 BEGIN
30     FOR i IN REVERSE 1..10 LOOP
31         dbms_output.put_line(i);
32     END LOOP;
33 END;
34 
35 -- while 循环
36 DECLARE 
37     i NUMBER := 1;
38 BEGIN
39     WHILE i <= 10 loop
40         dbms_output.put_line(i);
41         i := i+1;
42     END LOOP;
43 END;
44 
45 -- 简单循环
46 DECLARE 
47     i NUMBER := 1;
48 BEGIN 
49     LOOP
50         EXIT WHEN i > 10;
51         dbms_output.put_line(i);
52         i := i+1;
53     END LOOP;
54 END;
```

### 二、游标

#### 1、游标概述　　

##### 1.1 游标: (光标/指针) 是对查询结果集的封装, 相当于是jdbc中的ResultSet

##### 1.2 语法:

```mysql
1 -- 声明游标
2     CURSOR 游标名 IS 查询语句;
3     CURSOR 游标名(参数名 参数类型) IS 查询语句 WHERE 列名 = 参数名;
```

##### 1.3 开发步骤:

###### 1.打开游标 open 游标名

###### 2.从游标中提取数据:

> fetch 游标名 into 变量
>
> 　　  游标名%notfound 没有数据
>
> 　　　游标名%found 找到数据

###### 3.关闭游标 close 游标名

#### 2、使用示例：

```mysql
 1 -- 无参 
 2 -- 输出所有员工的信息
 3 DECLARE
 4    -- 声明游标
 5   CURSOR vemps IS SELECT * FROM emp;
 6    -- 声明变量
 7   vrow emp % rowtype;
 8 BEGIN
 9    --1. 打开游标
10    open vemps;
11    --2. 提取数据
12    LOOP
13        FETCH vemps INTO vrow;
14         -- 判断是否有数据
15        EXIT WHEN vemps % notfound;
16         -- 打印数据
17        dbms_output.put_line('姓名:'||vrow.ename||'  工资:'||vrow.sal);
18    END LOOP;
19    
20    -- 关闭游标
21    CLOSE vemps;
22 END;
23 ---------------------------------------------------------------- 
24 -- for 变量游标        
25 DECLARE
26    -- 声明游标
27    CURSOR vemps IS SELECT * FROM emp;
28    -- 声明记录型变量
29    vrow emp % rowtype;
30 BEGIN
31    -- 循环遍历游标
32    FOR vrow IN vemps
33    LOOP
34         dbms_output.put_line('姓名:'||vrow.ename||' 工资:'||vrow.sal);
35    END LOOP;
36 END;
37 
38 -- ===============================================================
39 -- 有参
40 -- 输出指定部门的员工信息
41 DECLARE
42    -- 声明游标
43    CURSOR vemps(vdeptno NUMBER) IS SELECT * FROM emp WHERE deptno = vdeptno;
44    -- 声明记录型变量
45    vrow emp % rowtype;
46 BEGIN
47    -- 1. 打开游标
48    OPEN vemps(20);
49    -- 2.循环遍历游标
50    LOOP
51     FETCH vemps into vrow;
52     EXIT when vemps % notfound;
53       -- 打印数据
54         dbms_output.put_line('姓名:'||vrow.ename||' 工资:'||vrow.sal);
55    END LOOP;
56    -- 3. 关闭游标
57    CLOSE vemps;
58 END;
```

### 三、例外

#### 1、例外概述

   例外 (意外): 相当于是java异常

　　语法: 
　　

```mysql
 1 declare 
 2     声明部分 
 3 begin 
 4     业务逻辑
 5 exception 
 6     处理例外 
 7     when 例外1 then 
 8 
 9     when 例外2 then 
10 
11     when others then 
12 
13 end;
```


　　常见的系统的例外:

  - zero_divide : 除零例外
  - value_error : 类型转换
  - no_data_found : 没有找到数据例外
  - too_many_rows : 查询出多行记录,但是赋值给了单行变量

#### 2、例外使用示例

```mysql
 1 DECLARE
 2     i NUMBER;
 3     vrow emp % rowtype;
 4 BEGIN
 5 --     i := 5/0;
 6 --     i := 'aaa';
 7 --     select * into vrow from emp where empno = 1234566;
 8     select * into vrow from emp;
 9 EXCEPTION
10     WHEN too_many_rows THEN
11         dbms_output.put_line('查询出多行记录,但是赋值给了单行变量');
12     WHEN no_data_found THEN
13         dbms_output.put_line('发生了没有找到数据例外');
14     WHEN value_error THEN
15         dbms_output.put_line('发生类型转换的例外');
16     WHEN zero_divide THEN
17         dbms_output.put_line('发生除零的例外');
18     WHEN others THEN
19         dbms_output.put_line('发生未知的例外');
20 END;
```

#### 3、自定义例外

　　语法：

```mysql
 1 DECLARE 
 2     -- 声明例外
 3     例外名称 EXCEPTION ;
 4 BEGIN
 5     -- 抛出例外
 6     raise 例外名称 ; 
 7 EXCEPTION 
 8     -- 捕获例外
 9     WHEN 例外名称 THEN
10     ....
11 END ;
```

　　使用示例：

```mysql
 1 -- 查询指定编号的员工,若没有找到,则抛出自定义例外
 2 DECLARE 
 3     -- 声明游标
 4     CURSOR vemps IS SELECT * FROM emp WHERE empno = 1234 ; 
 5     -- 记录型变量
 6     vrow vemps % rowtype ; 
 7     -- 定义例外
 8     no_emp_found EXCEPTION ; 
 9 BEGIN 
10     --1.打开游标
11     OPEN vemps ;
12     --2.提取记录 
13     FETCH vemps INTO vrow ;
14     -- 判断是否有数据 
15     IF vemps % notfound THEN 
16         -- 抛出例外
17         raise no_emp_found ; 
18     END IF ; 
19     -- 关闭游标
20     CLOSE vemps ; 
21 EXCEPTION 
22     WHEN no_emp_found THEN 
23         dbms_output.put_line('没有找到对应的员工') ; 
24 END ;
```

###  四、存储过程

#### 1、概述　　

　　存储过程: 实际上是将一段已经编译好的PLSQL代码片断,封装在数据库中。

　　作用:

##### 1. 提高执行效率

##### 2. 提高代码复用性

　　语法:


```mysql
1 create [or replace] procedure 过程名称[(参数1 in|out 参数类型,参数2 in|out 参数类型)]
2 is | as
3 　　-- 声明　　　　
4 begin
5  　 -- 业务
6 end;
```

#### 2、使用示例


```mysql
 1 -- 给指定员工涨薪,并打印涨薪前和涨薪后的工资
 2 -- 员工编号 : 输入参数
 3 -- 涨多少 : 输入参数
 4 /*
 5    1. 查询当前工
 6    2. 打印涨薪前工资
 7    3. 涨工资
 8    4. 打印涨薪后的工资
 9    5. 提交数据
10 */
11 create or replace procedure proc_updatesal(vempno in number,vcount in number)
12 is
13    -- 声明变量记录当前工资
14    vsal number;    
15 begin
16    --1. 查询当前工资
17    select sal into vsal from emp where empno=vempno;
18    --2. 打印涨薪前工资
19    dbms_output.put_line('涨薪前:'||vsal);
20    --3. 涨工资
21    update emp set sal=vsal+vcount where empno=vempno;
22    -- 4. 打印涨薪后的工资
23    dbms_output.put_line('涨薪后:'||(vsal+vcount));
24    --5. 提交数据
25    commit;
26 end;
27 
28 -- 调用存储过程
29 -- 方式1:
30 call proc_update_sal(7369,100);
31 
32 -- 方式2:
33 declare
34 
35 begin
36    proc_updatesal(7369,100);
37 end;
38 
39 s
40 -- 获取指定编号员工的年薪
41 /*
42    编号: in  输入
43    年薪: out 输出
44 */
45 create or replace procedure proc_getyearsal(vempno in number,vyearsal out number)
46 is
47        
48 begin
49   select sal*12+nvl(comm,0) into vyearsal from emp where empno=vempno;
50 end;
51 
52 -- plsql代码片断中调用
53 declare
54    yearsal number;
55 begin
56    proc_getyearsal(7369,yearsal);
57    dbms_output.put_line(yearsal);
58 end;
59 
60 
61 -- 封装存储过程,输出的是游标类型, 所有员工
62 /*
63    sys_refcursor : 系统引用游标
64 */
65 create or replace procedure proc_getemps(vemps out sys_refcursor)
66 is
67 
68 begin
69     -- 打开游标, 谁调用谁关闭
70     open vemps for select * from emp;   
71 end;
72 
73 declare
74   vemps sys_refcursor;
75   vrow emp%rowtype;
76 begin
77   -- 调用存储过程
78   proc_getemps(vemps);
79   
80   loop
81      fetch vemps into vrow; 
82      exit when vemps%notfound;  
83      dbms_output.put_line(vrow.ename);
84   end loop;
85   -- 关闭游标
86   close vemps;
87 end;五、存储函数
```

### 五、存储函数概述

　　存储函数: 实际上是将一段已经编译好的PLSQL代码片断,封装在数据库中。

#### 1、作用:

1. 提高执行效率

2. 提高代码复用性

　　语法: 


```mysql
1 create [or replace] function 函数名称(参数1 in|out 参数类型) return 返回类型 
2 is 
3 
4 begin 
5 
6 end;
```


　　存储过程和存储函数:

1. 函数有返回值,过程没有返回值

2. 函数可以直接在SQL语句中使用,过程不可以

3. 函数能实现的功能,过程能实现

4. 过程能实现的功能,函数也能实现

5. 函数和过程本质上没有区别 通常情况下,我们自己开发封装的是存储过程

#### 2、使用示例


```mysql
 1 -- 存储函数:获取年薪
 2 create or replace function func_getyearsal(vempno number) return number
 3 is
 4   vyearsal number;
 5 begin
 6   select sal*12+nvl(comm,0) into vyearsal from emp where empno=vempno;
 7   return vyearsal;
 8 end;
 9 
10 -- 调用
11 declare
12    yearsal number;
13 begin
14    yearsal := func_getyearsal(7369);
15    dbms_output.put_line(yearsal);
16 end;
17 
18 select emp.*,func_getyearsal(emp.empno) from emp;
```

###  六、触发器

#### 　1、概述：

数据库触发器是一个与表相关的、存储的PL/SQL程序。每当一个特定的数据操作语句（insert，update，delete）在指定的表上发出是，Oracle自动地执行触发器中定义的语句序列。

#### 　2、作用：

 - 监听表中的数据变化；
 - 对表中的数据进行校验

#### 　3、语法：


```mysql
 1 CREATE [OR REPLACE] TRIGGER 触发器名称 
 2 {BEFORE | AFTER}
 3 {INSERT | UPDATE | DELETE} 
 4 ON 表名 
 5 [ FOR EACH ROW [WHEN(条件)]]
 6 DECLARE
 7     ....
 8 BEGIN
 9     PLSQL块
10 END 触发器名;
```

#### 　　4、触发器的类型

- 行级触发器：一条SQL语句，影响了多少行记录，触发器就会执行多少次；
  - 两个内置对象：
    - :new 新的记录
    - :old 旧的记录
- 语句级触发器：一条SQL语句，无论影响了多少行记录，都只触发一次；

#### 　　5、使用示例


```mysql
  1 -- 若用户向表中插入数据之后, 打印一句话
  2 create or replace trigger tri_test1
  3 after
  4 insert 
  5 on emp
  6 declare
  7 
  8 begin
  9    dbms_output.put_line('有人插入了....');
 10 end;
 11 
 12 insert into emp(empno,ename) values(9527,'华安');
 13 -- 执行一条更新工资的语句
 14 
 15 -- 周二老板不在,不能办理员工入职(不能向员工表中插入数据)
 16 -- 触发器
 17 -- before insert
 18 -- 判断今天是否是周二
 19 select trim(to_char(sysdate,'day')) from dual;
 20 
 21 create or replace trigger tri_checkday
 22 before
 23 insert
 24 on emp
 25 declare
 26    vday varchar2(20);
 27 begin
 28    -- 查询当前周几
 29    select trim(to_char(sysdate,'day')) into vday from dual;
 30    -- 判断是否为周二,若为周二,则需要中断插入操作
 31    if vday = 'tuesday' then
 32      --                   -20000 - -20999
 33      raise_application_error(-20001,'周二老板不在,不能插入');
 34    end if;
 35 end;
 36 
 37 insert into emp(empno,ename) values(9527,'华安');
 38 
 39 select * from emp;
 40 
 41 -- 语句级触发器
 42 create trigger tri_test3
 43 before
 44 update
 45 on emp
 46 declare
 47 
 48 begin
 49   dbms_output.put_line('语句级触发器'); 
 50 end;
 51 
 52 -- 行级触发器
 53 create or replace trigger tri_test4
 54 before
 55 update
 56 on emp
 57 for each row
 58 declare
 59 
 60 begin
 61   dbms_output.put_line('行级触发器,旧的工资:'||:old.sal||'  新的工资:'||:new.sal); 
 62 end;
 63 
 64 update emp set sal=sal+100;
 65 
 66 -- 6个月 ---> 人事 加薪 ---> 加10块钱 ---> 老板签字
 67 -- 校验员工薪资 调整后的工资一定要 大于 薪资调整前的工资
 68 -- 触发器:  before update on emp
 69 -- 行级触发器
 70 create or replace trigger tri_checksal
 71 before
 72 update
 73 on emp
 74 for each row
 75 declare
 76 
 77 begin
 78   -- 调整后的工资 <= 薪资调整前的工资 ,则中断更新操作
 79   -- :new.sal    <= :old.sal
 80   if :new.sal <= :old.sal then
 81      raise_application_error(-20002,'坑爹的,降薪啦!');
 82   end if;
 83 end;
 84 
 85 update emp set sal=sal-100;
 86 
 87 /*
 88      使用触发器模拟类似auto_increment功能
 89      当用户插入的时候,若为sid为null,则给sid赋值一个编号
 90 */
 91 create table stu(
 92      sid number primary key,
 93      name varchar2(20)
 94 );
 95 
 96 -- 创建一个序列
 97 create sequence seq_stu;
 98 
 99 -- 触发器: before insert on stu
100 -- 行级触发器
101 create or replace trigger tri_auto
102 before
103 insert 
104 on stu
105 for each row
106 declare
107 
108 begin
109    -- 从序列中查询一个数字出来,赋值给sid
110    select seq_stu.nextval into :new.sid from dual;
111 end;
112 
113 -- 同样一张表,有时候自己指定id, 有时候需要数据库自动生成id
114 insert into stu values(null,'zs');
115 insert into stu values(4,'zs');
116 select * from stu; 
```