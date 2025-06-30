---
title: "MySQL经典50道基础练习题（附加答案）"
description: ""
date: 2025-06-16T09:19:32+08:00
Lastmod: 2025-06-16T09:19:32+08:00
draft: false
showComments: true
featureimage: "https://picsum.photos/seed/7d713f/1600/900.webp"
tags: ["数据库软件"]
series: ["数据库教程"]
series_order: 2
---

## 一、环境准备

测试数据库的代码如下：

```sql
create table Student(sid varchar(10),sname varchar(10),sage datetime,ssex nvarchar(10));
insert into Student values('01' , '赵雷' , '1990-01-01' , '男');
insert into Student values('02' , '钱电' , '1990-12-21' , '男');
insert into Student values('03' , '孙风' , '1990-05-20' , '男');
insert into Student values('04' , '李云' , '1990-08-06' , '男');
insert into Student values('05' , '周梅' , '1991-12-01' , '女');
insert into Student values('06' , '吴兰' , '1992-03-01' , '女');
insert into Student values('07' , '郑竹' , '1989-07-01' , '女');
insert into Student values('08' , '王菊' , '1990-01-20' , '女');
create table Course(cid varchar(10),cname varchar(10),tid varchar(10));
insert into Course values('01' , '语文' , '02');
insert into Course values('02' , '数学' , '01');
insert into Course values('03' , '英语' , '03');
create table Teacher(tid varchar(10),tname varchar(10));
insert into Teacher values('01' , '张三');
insert into Teacher values('02' , '李四');
insert into Teacher values('03' , '王五');
create table SC(sid varchar(10),cid varchar(10),score decimal(18,1));
insert into SC values('01' , '01' , 80);
insert into SC values('01' , '02' , 90);
insert into SC values('01' , '03' , 99);
insert into SC values('02' , '01' , 70);
insert into SC values('02' , '02' , 60);
insert into SC values('02' , '03' , 80);
insert into SC values('03' , '01' , 80);
insert into SC values('03' , '02' , 80);
insert into SC values('03' , '03' , 80);
insert into SC values('04' , '01' , 50);
insert into SC values('04' , '02' , 30);
insert into SC values('04' , '03' , 20);
insert into SC values('05' , '01' , 76);
insert into SC values('05' , '02' , 87);
insert into SC values('06' , '01' , 31);
insert into SC values('06' , '03' , 34);
insert into SC values('07' , '02' , 89);
insert into SC values('07' , '03' , 98);
```

> 脚本说明：一共4张表，分别对应学生信息（Student）、课程信息（Course）、教师信息（Teacher）以及成绩信息（SC）

### 二、正文部分

1. 查询"01"课程比"02"课程成绩高的学生的信息及课程分数

```sql
SELECT student.*,t3.sid FROM
(SELECT t1.sid,t1.score FROM (SELECT sid,score FROM sc WHERE cid = "01") as t1
JOIN
(SELECT sid,score FROM sc WHERE cid = "02") as t2
ON 
t1.sid = t2.sid WHERE t1.score > t2.score) as t3 
JOIN student
ON t3.sid = student.sid;
```

结果：

```sql
+-----+-------+---------------------+------+-----+
| sid | sname | sage                | ssex | sid |
+-----+-------+---------------------+------+-----+
| 02  | 钱电  | 1990-12-21 00:00:00 | 男   | 02  |
| 04  | 李云  | 1990-08-06 00:00:00 | 男   | 04  |
+-----+-------+---------------------+------+-----+
2 rows in set
```

**解析**：先将课程为01和02的课程及对应分数筛选出来，再join，on为01.sid = 02.sid，条件为01.score >02.score，结果'存'为新表t3，再将Student表和t3表join

 1. 查询学生选课存在" 01 "课程但可能不存在" 02 "课程的情况（不存在时显示为 null）
SELECT *FROM
(SELECT* FROM sc WHERE cid = "01") as t1
LEFT JOIN
(SELECT * FROM sc WHERE cid = "02") as t2
ON t1.sid = t2.sid;
结果：

```sql
+-----+-----+-------+------+------+-------+
| sid | cid | score | sid  | cid  | score |
+-----+-----+-------+------+------+-------+
| 01  | 01  | 80.0  | 01   | 02   | 90.0  |
| 02  | 01  | 70.0  | 02   | 02   | 60.0  |
| 03  | 01  | 80.0  | 03   | 02   | 80.0  |
| 04  | 01  | 50.0  | 04   | 02   | 30.0  |
| 05  | 01  | 76.0  | 05   | 02   | 87.0  |
| 06  | 01  | 31.0  | NULL | NULL | NULL  |
+-----+-----+-------+------+------+-------+
6 rows in set
```

**解析**：即找出学生选了01课程没有选02课程的情况，用left join即可

1. 查询平均成绩大于等于 60 分的同学的学生编号和学生姓名和平均成绩

```sql
#多表联合查询
SELECT  sc.sid,student.sname,avg(sc.score) FROM sc ,student WHERE sc.sid = student.sid  GROUP BY  sc.sid  HAVING avg(sc.score) > 60;
#多表连接查询
SELECT  sc.sid,student.sname,avg(sc.score) FROM sc JOIN student on sc.sid = student.sid  GROUP BY  sc.sid  HAVING avg(sc.score) > 60;
```

结果：

```sql
+-----+-------+---------------+
| sid | sname | avg(sc.score) |
+-----+-------+---------------+
| 01  | 赵雷  | 89.66667      |
| 02  | 钱电  | 70.00000      |
| 03  | 孙风  | 80.00000      |
| 05  | 周梅  | 81.50000      |
| 07  | 郑竹  | 93.50000      |
+-----+-------+---------------+
5 rows in set
```

**解析**：首先确定的是两张表，student和sc，这里使用多表联合查询和多表连接查的方式都可以，关联条件是sid，然后分组，最后加一个having函数，条件是平均成绩大于60，即可查询出来

1. 查询在 SC 表存在成绩的学生信息

```sql
#多表联合查询方式
SELECT  t1.*,t2.score FROM student t1, sc t2 WHERE t1.sid = t2.sid  GROUP BY t1.sid;
#多表连接查询方式
SELECT a.*,b.score FROM student as a 
    JOIN sc AS b 
    ON a.sid = b.sid 
        GROUP BY a.sid;
```

结果

```sql
+-----+-------+---------------------+------+-------+
| sid | sname | sage                | ssex | score |
+-----+-------+---------------------+------+-------+
| 01  | 赵雷  | 1990-01-01 00:00:00 | 男   | 80.0  |
| 02  | 钱电  | 1990-12-21 00:00:00 | 男   | 70.0  |
| 03  | 孙风  | 1990-05-20 00:00:00 | 男   | 80.0  |
| 04  | 李云  | 1990-08-06 00:00:00 | 男   | 50.0  |
| 05  | 周梅  | 1991-12-01 00:00:00 | 女   | 76.0  |
| 06  | 吴兰  | 1992-03-01 00:00:00 | 女   | 31.0  |
| 07  | 郑竹  | 1989-07-01 00:00:00 | 女   | 89.0  |
+-----+-------+---------------------+------+-------+
7 rows in set
```

**解析**：确定是两个表,student和sc，关联条件还是sid消除笛卡尔积，然后再group by，最后select 取需要的信息

1. 查询所有同学的学生编号、学生姓名、选课总数、所有课程的成绩总和

```sql
#多表联合查询方式
SELECT t1.sid as 学生编号,t1.sname as 学生姓名,COUNT(t2.cid) as 选课总数,SUM(t2.score) as 课程成绩总和  FROM student t1, sc t2 WHERE t1.sid = t2.sid  GROUP BY t1.sid;
#多表连接查询
SELECT t1.sid as 学生编号,t1.sname as 学生姓名,COUNT(t2.cid) as 选课总数,SUM(t2.score) as 课程成绩总和  FROM student t1 JOIN sc t2 ON t1.sid = t2.sid  GROUP BY t1.sid;
```

结果

```sql
+----------+----------+----------+--------------+
| 学生编号 | 学生姓名 | 选课总数 | 课程成绩总和 |
+----------+----------+----------+--------------+
| 01       | 赵雷     |        3 | 269.0        |
| 02       | 钱电     |        3 | 210.0        |
| 03       | 孙风     |        3 | 240.0        |
| 04       | 李云     |        3 | 100.0        |
| 05       | 周梅     |        2 | 163.0        |
| 06       | 吴兰     |        2 | 65.0         |
| 07       | 郑竹     |        2 | 187.0        |
+----------+----------+----------+--------------+
7 rows in set
```

**解析**：两个聚合函数（统计函数）一个count(cid),一个sum(score),同样join student表和sc表，再group by sid即可

1. 查询「李」姓老师的数量
SELECT COUNT(t.tid) FROM teacher t WHERE t.tname like "%李%";
结果：

```sql
+--------------+
| COUNT(t.tid) |
+--------------+
|            1 |
+--------------+
1 row in set
```

**解析**：count加条件函数加通配符即可

1. 查询学过「张三」老师授课的同学的信息

```sql
SELECT f.*,e.tname FROM 
    (SELECT d.sid,c.tname FROM 
        (SELECT a.tname,b.cid FROM teacher AS a
            JOIN course AS b 
            ON a.tid = b.tid
                WHERE a.tname = '张三') AS c
        JOIN sc AS d
        ON c.cid = d.cid) AS e
JOIN student AS f
ON e.sid = f.sid
```

结果

```sql
+-----+-------+---------------------+------+-------+
| sid | sname | sage                | ssex | tname |
+-----+-------+---------------------+------+-------+
| 01  | 赵雷  | 1990-01-01 00:00:00 | 男   | 张三  |
| 02  | 钱电  | 1990-12-21 00:00:00 | 男   | 张三  |
| 03  | 孙风  | 1990-05-20 00:00:00 | 男   | 张三  |
| 04  | 李云  | 1990-08-06 00:00:00 | 男   | 张三  |
| 05  | 周梅  | 1991-12-01 00:00:00 | 女   | 张三  |
| 07  | 郑竹  | 1989-07-01 00:00:00 | 女   | 张三  |
+-----+-------+---------------------+------+-------+
6 rows in set
```

**解析**：四表连接，teacher表里的tid与course表里的tid，条件为tname=‘张三’，再course表里的cid与sc表里的cid，最后sc表里的sid与student里的sid

1. 查询没有学全所有课程的同学的信息

```sql
SELECT a.*,count(b.cid) AS 所学课程数
FROM student AS a
    LEFT JOIN sc AS b
    ON a.sid = b.sid
        GROUP BY b.sid
            HAVING COUNT(b.cid)< (SELECT COUNT(c.cid) FROM course as c);
```

结果

```sql
+-----+-------+---------------------+------+------------+
| sid | sname | sage                | ssex | 所学课程数 |
+-----+-------+---------------------+------+------------+
| 05  | 周梅  | 1991-12-01 00:00:00 | 女   |          2 |
| 06  | 吴兰  | 1992-03-01 00:00:00 | 女   |          2 |
| 07  | 郑竹  | 1989-07-01 00:00:00 | 女   |          2 |
| 08  | 王菊  | 1990-01-20 00:00:00 | 女   |          0 |
+-----+-------+---------------------+------+------------+
```

**解析**：先查询总课程数，再查询所有同学的信息，筛选条件为其所学课程数小于总课程数

1. 查询至少有一门课与学号为" 01 "的同学所学相同的同学的信息

```sql
SELECT b.* FROM student AS b
    JOIN sc AS a 
    ON b.sid  = a.sid 
        WHERE a.cid in 
                    (SELECT a.cid FROM sc AS a WHERE a.sid = '01') 
        GROUP BY b.sid 
             HAVING b.sid != '01';
```

结果

```sql
+-----+-------+---------------------+------+
| sid | sname | sage                | ssex |
+-----+-------+---------------------+------+
| 02  | 钱电  | 1990-12-21 00:00:00 | 男   |
| 03  | 孙风  | 1990-05-20 00:00:00 | 男   |
| 04  | 李云  | 1990-08-06 00:00:00 | 男   |
| 05  | 周梅  | 1991-12-01 00:00:00 | 女   |
| 06  | 吴兰  | 1992-03-01 00:00:00 | 女   |
| 07  | 郑竹  | 1989-07-01 00:00:00 | 女   |
+-----+-------+---------------------+------+
6 rows in set
```

**解析**：#先从成绩表里查询学号为01的同学所学的课程编号，筛选条件为sc.cid in 01同学所学编号，再使用学生表和成绩表两表关联，关联字段为sid，并且把课程编号作为子查询的条件，刷选，然后再group by sid 最后通过having筛选sid 不等于01

1. 查询和" 01 "号的同学学习的课程完全相同的其他同学的信息

```sql
SELECT t2.*, count(t3.cid) 
FROM student t2 JOIN sc t3 
ON t2.sid = t3.sid WHERE t2.sid != "01" GROUP BY t2.sid 
HAVING count(t3.cid) = ( SELECT COUNT(*) FROM sc t1 WHERE t1.sid = "01" );
```

结果：

```sql
+-----+-------+---------------------+------+---------------+
| sid | sname | sage                | ssex | count(t3.cid) |
+-----+-------+---------------------+------+---------------+
| 02  | 钱电  | 1990-12-21 00:00:00 | 男   |             3 |
| 03  | 孙风  | 1990-05-20 00:00:00 | 男   |             3 |
| 04  | 李云  | 1990-08-06 00:00:00 | 男   |             3 |
+-----+-------+---------------------+------+---------------+
3 rows in set
```

**解析**：先从成绩表中查询学号为01的总课程数，然后使用学生表和成绩表关联查询，关联字段为sid，消除笛卡尔积，where条件语句过滤学号01，并且用学号字段分组，并且使用having函数，统计课程总数=学号为1的课程总数

1. 查询没学过"张三"老师讲授的任一门课程的学生姓名

```sql
SELECT student.sname FROM student 
    WHERE student.sid NOT IN 
        (SELECT sc.sid FROM sc     
                    JOIN course 
                    ON sc.cid=course.cid
                    JOIN teacher 
                    ON course.tid=teacher.tid 
        WHERE tname='张三' )
```

结果

```sql
+-------+
| sname |
+-------+
| 吴兰  |
| 王菊  |
+-------+
2 rows in set
```

**解析**：先找出所有学生选课信息及sid，再找出张三老师授课课程，将其连接，再用student里的sid not in 前面的sid

1. 查询两门及其以上不及格课程的同学的学号，姓名及其平均成绩

```sql
SELECT c.sname,b.* FROM student as c
JOIN
(
(SELECT sid ,COUNT(cid) FROM sc WHERE score < 60 GROUP BY sid HAVING COUNT(cid) >=2) as a

#找出平均成绩
JOIN
(SELECT sid,avg(score) FROM sc GROUP BY sid ) as b
 ON a.sid = b.sid)  
ON c.sid = b.sid
```

结果

```sql
+-------+-----+------------+
| sname | sid | avg(score) |
+-------+-----+------------+
| 李云  | 04  | 33.33333   |
| 吴兰  | 06  | 32.50000   |
+-------+-----+------------+
2 rows in set
```

**解析**：先查询出不及格两门或两门以上的数据，再查询出不及格的平均成绩，再三张表嵌套关联

1. 查询" 01 "课程分数小于 60，按分数降序排列的学生信息
SELECT b.*,a.score from student b  JOIN (SELECT* FROM sc WHERE cid = "01" AND score < 60  ORDER BY score DESC ) as a  ON a.sid = b.sid ;
结果：

```sql
+-----+-------+---------------------+------+-------+
| sid | sname | sage                | ssex | score |
+-----+-------+---------------------+------+-------+
| 04  | 李云  | 1990-08-06 00:00:00 | 男   | 50.0  |
| 06  | 吴兰  | 1992-03-01 00:00:00 | 女   | 31.0  |
+-----+-------+---------------------+------+-------+
2 rows in set
```

**解析**：先查询出01课程分数小于60的sid ，按照分数降序，然后和学生表关联

1. 按平均成绩从高到低显示所有学生的所有课程的成绩以及平均成绩

```sql
 SELECT a.sid,a.score,a.cid,b.`平均成绩` FROM sc a JOIN 
(SELECT sid,avg(score) as 平均成绩 FROM sc GROUP BY sid ) as b ON a.sid = b.sid ORDER BY b.`平均成绩` DESC;
```

结果：

```sql
+-----+-------+-----+----------+
| sid | score | cid | 平均成绩 |
+-----+-------+-----+----------+
| 07  | 89.0  | 02  | 93.50000 |
| 07  | 98.0  | 03  | 93.50000 |
| 01  | 80.0  | 01  | 89.66667 |
| 01  | 90.0  | 02  | 89.66667 |
| 01  | 99.0  | 03  | 89.66667 |
| 05  | 76.0  | 01  | 81.50000 |
| 05  | 87.0  | 02  | 81.50000 |
| 03  | 80.0  | 01  | 80.00000 |
| 03  | 80.0  | 02  | 80.00000 |
| 03  | 80.0  | 03  | 80.00000 |
| 02  | 70.0  | 01  | 70.00000 |
| 02  | 60.0  | 02  | 70.00000 |
| 02  | 80.0  | 03  | 70.00000 |
| 04  | 50.0  | 01  | 33.33333 |
| 04  | 30.0  | 02  | 33.33333 |
| 04  | 20.0  | 03  | 33.33333 |
| 06  | 31.0  | 01  | 32.50000 |
| 06  | 34.0  | 03  | 32.50000 |
+-----+-------+-----+----------+
18 rows in set
```

**解析**：先求平均成绩，注意，这里的平均成绩一定要取别名，然后取所有人的成绩，再关联，然后按照平均成绩降序排列

1. 查询各科成绩最高分、最低分和平均分
　　以如下形式显示：
　　课程 id，最高分，最低分，平均分，及格率，中等率，
　　优良率，优秀率
　　及格为>=60，中等为：[70,80)，优良为：[80-90)，优秀为：>=90
　　要求输出课程号和选修人数，查询结果按人数降序排列，若人数相同，按课程号升序

```sql
SELECT cid AS 课程id,MAX(score) AS 最高分,MIN(score) AS 最低分 ,AVG(score) AS 平均分, 
SUM(CASE WHEN score >=60 THEN 1 ELSE 0 END)/COUNT(sid) AS 及格率,
SUM(CASE WHEN score >=70 AND score <80 THEN 1 ELSE 0 END)/count(sid) AS 中等率,
SUM(CASE WHEN score >=80 AND score <90 THEN 1 ELSE 0 END)/count(sid) AS 优良率,
SUM(CASE WHEN score >=90 THEN 1 ELSE 0 END)/count(sid) AS 优秀率
FROM sc GROUP BY cid ORDER BY cid ASC;
```

结果

```sql
+--------+--------+--------+----------+--------+--------+--------+--------+
| 课程id | 最高分 | 最低分 | 平均分   | 及格率 | 中等率 | 优良率 | 优秀率 |
+--------+--------+--------+----------+--------+--------+--------+--------+
| 01     | 80.0   | 31.0   | 64.50000 | 0.6667 | 0.3333 | 0.3333 | 0.0000 |
| 02     | 90.0   | 30.0   | 72.66667 | 0.8333 | 0.0000 | 0.5000 | 0.1667 |
| 03     | 99.0   | 20.0   | 68.50000 | 0.6667 | 0.0000 | 0.3333 | 0.3333 |
+--------+--------+--------+----------+--------+--------+--------+--------+
3 rows in set
```

**解析**：重点在case when语句的用法，其实case when 就类似于 if函数 if x>某个值，then 1 else 0。就只用一个表，只是对表头需要做修改，用聚合函数+AS

1. 按各科成绩进行排序，并显示排名， Score 重复时保留名次空缺
select *, rank() over(partition by cid order by score desc) AS ranked from sc;
结果：

```sql
+-----+-----+-------+--------+
| sid | cid | score | ranked |
+-----+-----+-------+--------+
| 01  | 01  | 80.0  |      1 |
| 03  | 01  | 80.0  |      1 |
| 05  | 01  | 76.0  |      3 |
| 02  | 01  | 70.0  |      4 |
| 04  | 01  | 50.0  |      5 |
| 06  | 01  | 31.0  |      6 |
| 01  | 02  | 90.0  |      1 |
| 07  | 02  | 89.0  |      2 |
| 05  | 02  | 87.0  |      3 |
| 03  | 02  | 80.0  |      4 |
| 02  | 02  | 60.0  |      5 |
| 04  | 02  | 30.0  |      6 |
| 01  | 03  | 99.0  |      1 |
| 07  | 03  | 98.0  |      2 |
| 02  | 03  | 80.0  |      3 |
| 03  | 03  | 80.0  |      3 |
| 06  | 03  | 34.0  |      5 |
| 04  | 03  | 20.0  |      6 |
+-----+-----+-------+--------+
18 rows in set
```

**解析**：MySQL可以实现Oracle中的排名公式，一共有三种

1. rank() over(order by col_name desc）2.dense_rank() over() 3.row_number() over()
第一个是如果出现了相同排名都为同一排名，下个排名跳过，例如1,1,3,4
第二个是如果出现了相同排名都为同一排名，下个排名不跳过，例如1,1,2,3
第三个是直接对行进行排名不分是否有相同值
此题目要按照各科成绩进行排序 over()中要填partition by col_name order by col_name
第一个colname 为分组的内容，第二个是按什么值排的内容
1. 查询学生的总成绩，并进行排名，总分重复时保留名次空缺
SELECT a.*,rank() over(ORDER BY a.总成绩 DESC) AS Ranked  FROM
(SELECT*, SUM(score) AS 总成绩 FROM sc GROUP BY sid)  AS a ;
结果：

```sql
+-----+-----+-------+--------+--------+
| sid | cid | score | 总成绩 | Ranked |
+-----+-----+-------+--------+--------+
| 01  | 01  | 80.0  | 269.0  |      1 |
| 03  | 01  | 80.0  | 240.0  |      2 |
| 02  | 01  | 70.0  | 210.0  |      3 |
| 07  | 02  | 89.0  | 187.0  |      4 |
| 05  | 01  | 76.0  | 163.0  |      5 |
| 04  | 01  | 50.0  | 100.0  |      6 |
| 06  | 01  | 31.0  | 65.0   |      7 |
+-----+-----+-------+--------+--------+
7 rows in set
```

**解析**：跟上题一样用rank（）over（），只是多了层嵌套

1. 查询学生的总成绩，并进行排名，总分重复时不保留名次空缺
SELECT a.*,dense_rank() over(ORDER BY a.total_socre DESC) AS Ranked FROM
(SELECT*,SUM(score) AS total_socre FROM sc GROUP BY sid) AS a;
结果：

```sql
+-----+-----+-------+-------------+--------+
| sid | cid | score | total_socre | Ranked |
+-----+-----+-------+-------------+--------+
| 01  | 01  | 80.0  | 269.0       |      1 |
| 03  | 01  | 80.0  | 240.0       |      2 |
| 02  | 01  | 70.0  | 210.0       |      3 |
| 07  | 02  | 89.0  | 187.0       |      4 |
| 05  | 01  | 76.0  | 163.0       |      5 |
| 04  | 01  | 50.0  | 100.0       |      6 |
| 06  | 01  | 31.0  | 65.0        |      7 |
+-----+-----+-------+-------------+--------+
7 rows in set
```

**解析**：和上面一样，只是换成dense_rank () over()，只是总分没有重复无法看出区别

1. 统计各科成绩各分数段人数：课程编号，[100-85)，[85-70)，[70-60)，[60-0] 及所占百分比

```sql
SELECT cid AS 课程ID, 
SUM(CASE WHEN score <= 60 THEN 1 ELSE 0 END)/count(sid) AS 百分比1,
SUM(CASE WHEN score >60 AND score <=70 THEN 1 ELSE 0 END)/count(sid) AS 百分比2,
SUM(CASE WHEN score >70 AND score <=85 THEN 1 ELSE 0 END)/count(sid) AS 百分比3,
SUM(CASE WHEN score >85 THEN 1 ELSE 0 END)/count(sid) AS 百分比4
FROM sc GROUP BY cid ORDER BY cid
```

结果

```sql
+--------+---------+---------+---------+---------+
| 课程ID | 百分比1 | 百分比2 | 百分比3 | 百分比4 |
+--------+---------+---------+---------+---------+
| 01     | 0.3333  | 0.1667  | 0.5000  | 0.0000  |
| 02     | 0.3333  | 0.0000  | 0.1667  | 0.5000  |
| 03     | 0.3333  | 0.0000  | 0.3333  | 0.3333  |
+--------+---------+---------+---------+---------+
3 rows in set
```

**解析**：使用case when

1. 查询各科成绩前三名的记录
SELECT *FROM
(SELECT*,rank() over(PARTITION by cid ORDER BY score desc) as ranked FROM sc) as a
WHERE a.ranked <=3;
结果：

```sql
+-----+-----+-------+--------+
| sid | cid | score | ranked |
+-----+-----+-------+--------+
| 01  | 01  | 80.0  |      1 |
| 03  | 01  | 80.0  |      1 |
| 05  | 01  | 76.0  |      3 |
| 01  | 02  | 90.0  |      1 |
| 07  | 02  | 89.0  |      2 |
| 05  | 02  | 87.0  |      3 |
| 01  | 03  | 99.0  |      1 |
| 07  | 03  | 98.0  |      2 |
| 02  | 03  | 80.0  |      3 |
| 03  | 03  | 80.0  |      3 |
+-----+-----+-------+--------+
10 rows in set
```

**解析**：与上面rank一样，用rank（）over（）where ranked <=3
注意！where 的执行顺序在select前，嵌套一个select 语句就好

1. 查询每门课程被选修的学生数
SELECT  cid AS 课程id,COUNT(sid) AS 选修的学生数 FROM sc GROUP BY cid ORDER BY 课程id;
结果：

```sql
+--------+--------------+
| 课程id | 选修的学生数 |
+--------+--------------+
| 01     |            6 |
| 02     |            6 |
| 03     |            6 |
+--------+--------------+
3 rows in set
```

**解析**：单表 查询，使用group by ，order by

1. 查询出只选修两门课程的学生学号和姓名
SELECT student.sname,a.* FROM student JOIN
(SELECT sid,count(cid) as 选修课程数 FROM sc  GROUP BY sid HAVING 选修课程数 = 2) as a ON student.sid = a.sid;
结果：

```sql
+-------+-----+------------+
| sname | sid | 选修课程数 |
+-------+-----+------------+
| 周梅  | 05  |          2 |
| 吴兰  | 06  |          2 |
| 郑竹  | 07  |          2 |
+-------+-----+------------+
3 rows in set
```

**解析**：先从成绩表中查询出只选修两门课程的学生id和课程数，再和学生表进行关联查询
备注：
众所周知SQL的执行顺序应该为
FORM-JOIN ON-WHERE-GROUP BY-HAVING-SELECT-DISTINCT-UNION-ORDER
但是为什么上述语句中的能够在HAVING后面直接更SELECT语句里面的别名
因为mysql在版本迭代中，mysql对having做出了扩展，即having能够'解析'select中的别名，但是执行顺序还是having在select前。但是大多数标准SQL中，having是无法引用别名的。
还有where 后不能用聚合函数，having后可以用聚合函数

1. 查询男生、女生人数
SELECT ssex,COUNT(sid) FROM student GROUP BY ssex;
结果：

```sql
+------+------------+
| ssex | COUNT(sid) |
+------+------------+
| 男   |          4 |
| 女   |          4 |
+------+------------+
2 rows in set
```

**解析**：根据ssex  group by后再count()

1. 查询名字中含有「风」字的学生信息
SELECT * FROM student WHERE sname like "%风%";
结果：

```sql
+-----+-------+---------------------+------+
| sid | sname | sage                | ssex |
+-----+-------+---------------------+------+
| 03  | 孙风  | 1990-05-20 00:00:00 | 男   |
+-----+-------+---------------------+------+
1 row in set
```

**解析**：通配符，%，‘%a’a结尾，‘a%’a开头，‘%a%’含有a

1. 查询同名同性学生名单，并统计同名人数
SELECT *,COUNT(sid) as 同名人数 FROM  
(SELECT  a.* FROM student AS a JOIN student as b WHERE a.sname = b.sname AND a.ssex = b.ssex) as c  GROUP BY sid HAVING 同名人数 >=2;
结果：

 **解析**：连接表student和student on ssname and ssex 在group by sid（因为id唯一，name可能重名），count sid

1. 查询 1990 年出生的学生名单
SELECT * FROM student  WHERE YEAR(sage) = 1990;
结果：

```sql
+-----+-------+---------------------+------+
| sid | sname | sage                | ssex |
+-----+-------+---------------------+------+
| 01  | 赵雷  | 1990-01-01 00:00:00 | 男   |
| 02  | 钱电  | 1990-12-21 00:00:00 | 男   |
| 03  | 孙风  | 1990-05-20 00:00:00 | 男   |
| 04  | 李云  | 1990-08-06 00:00:00 | 男   |
| 08  | 王菊  | 1990-01-20 00:00:00 | 女   |
+-----+-------+---------------------+------+
5 rows in set
```

**解析**：sage一列为datetime类型，用时间函数。MySQL里面能够对datetime类型函数截取年、月、周、日等等 ，用YEAR()来表示年，以此类推

1. 查询每门课程的平均成绩，结果按平均成绩降序排列，平均成绩相同时，按课程编号升序排列
SELECT cid,avg(score) AS 平均成绩 FROM sc GROUP BY cid ORDER BY 平均成绩 DESC,cid ASC;
结果：

```sql
+-----+----------+
| cid | 平均成绩 |
+-----+----------+
| 02  | 72.66667 |
| 03  | 68.50000 |
| 01  | 64.50000 |
+-----+----------+
3 rows in set
```

**解析**：order by x desc,y,z,... 先根据x排序，再根据y，然后z....

1. 查询平均成绩大于等于 85 的所有学生的学号、姓名和平均成绩
SELECT student.sname , a.* FROM student
JOIN
(SELECT sid as 学号, avg(score) as 平均成绩 FROM sc GROUP BY sid HAVING 平均成绩 > 85) as a ON student.sid = a.学号;
结果：

```sql
+-------+------+----------+
| sname | 学号 | 平均成绩 |
+-------+------+----------+
| 赵雷  | 01   | 89.66667 |
| 郑竹  | 07   | 93.50000 |
+-------+------+----------+
2 rows in set
```

**解析**：先从成绩表中查询出平均成绩大于85的学生好和平均成绩（记住，这里需要取别名），然后再和学生表关联，关联字段为sid，获取到学生名字

1. 查询课程名称为「数学」，且分数低于 60 的学生姓名和分数

```sql
SELECT  student.sname,c.* FROM student 
JOIN
(SELECT  t1.cname,t2.score,t2.sid  FROM course as t1 
JOIN sc  as t2 
ON t1.cid = t2.cid 
WHERE  t2.score < 60  AND t1.cname = "数学") as c ON student.sid = c.sid;
```

结果

```sql
+-------+-------+-------+-----+
| sname | cname | score | sid |
+-------+-------+-------+-----+
| 李云  | 数学  | 30.0  | 04  |
+-------+-------+-------+-----+
1 row in set
```

**解析**：先把课程表和成绩表关联，获取到低于60分的学生号、分数和课程名称，作为临时表，然后再和学生表关联，获取到最后一个字段，学生姓名

 1. 查询所有学生的课程及分数情况（存在学生没成绩，没选课的情况）
SELECT  student.sname,c.* FROM student
JOIN
(SELECT a.cname,b.sid,b.score  FROM course as a
LEFT JOIN
sc AS b on a.cid = b.cid) as c on student.sid = c.sid;
结果：

```sql
+-------+-------+-----+-------+
| sname | cname | sid | score |
+-------+-------+-----+-------+
| 赵雷  | 语文  | 01  | 80.0  |
| 赵雷  | 数学  | 01  | 90.0  |
| 赵雷  | 英语  | 01  | 99.0  |
| 钱电  | 语文  | 02  | 70.0  |
| 钱电  | 数学  | 02  | 60.0  |
| 钱电  | 英语  | 02  | 80.0  |
| 孙风  | 语文  | 03  | 80.0  |
| 孙风  | 数学  | 03  | 80.0  |
| 孙风  | 英语  | 03  | 80.0  |
| 李云  | 语文  | 04  | 50.0  |
| 李云  | 数学  | 04  | 30.0  |
| 李云  | 英语  | 04  | 20.0  |
| 周梅  | 语文  | 05  | 76.0  |
| 周梅  | 数学  | 05  | 87.0  |
| 吴兰  | 语文  | 06  | 31.0  |
| 吴兰  | 英语  | 06  | 34.0  |
| 郑竹  | 数学  | 07  | 89.0  |
| 郑竹  | 英语  | 07  | 98.0  |
+-------+-------+-----+-------+
18 rows in set
```

**解析**：先把课程表和成绩表关联，关联字段为cid，获取到课程名称，学生号和学科成绩，作为临时表，然后再和学生表关联，关联字段为sid，获取到学生名字

1. 查询任何一门课程成绩在 70 分以上的姓名、课程名称和分数
SELECT  student.sname,c.* FROM student
JOIN
(SELECT a.cname,b.sid,b.score  FROM course as a
LEFT JOIN
sc AS b on a.cid = b.cid) as c on student.sid = c.sid WHERE c.score > 70;
结果：

```sql
+-------+-------+-----+-------+
| sname | cname | sid | score |
+-------+-------+-----+-------+
| 赵雷  | 语文  | 01  | 80.0  |
| 赵雷  | 数学  | 01  | 90.0  |
| 赵雷  | 英语  | 01  | 99.0  |
| 钱电  | 英语  | 02  | 80.0  |
| 孙风  | 语文  | 03  | 80.0  |
| 孙风  | 数学  | 03  | 80.0  |
| 孙风  | 英语  | 03  | 80.0  |
| 周梅  | 语文  | 05  | 76.0  |
| 周梅  | 数学  | 05  | 87.0  |
| 郑竹  | 数学  | 07  | 89.0  |
| 郑竹  | 英语  | 07  | 98.0  |
+-------+-------+-----+-------+
11 rows in set
```

**解析**：在上一题的基础上增加score > 70,使用where 或and都可以

1. 查询不及格的课程
SELECT cname,a.* FROM course
JOIN
(SELECT score,cid FROM sc WHERE score < 60) as a
ON course.cid = a.cid;
结果：

```sql
+-------+-------+-----+
| cname | score | cid |
+-------+-------+-----+
| 语文  | 50.0  | 01  |
| 数学  | 30.0  | 02  |
| 英语  | 20.0  | 03  |
| 语文  | 31.0  | 01  |
| 英语  | 34.0  | 03  |
+-------+-------+-----+
5 rows in set
```

**解析**：先从成绩表中获取到不及格的课程id和成绩，然后再和课程表关联，关联字典为课程id，获取到课程名称

1. 查询课程编号为 01 且课程成绩在 60 分以上的学生的学号和姓名

```sql
SELECT student.sname,c.* FROM student 
JOIN
(SELECT  b.sid ,b.score,a.cid ,a.cname FROM course as a
JOIN
sc as b
ON a.cid = b.cid WHERE a.cid = "01" AND b.score > 60) as c  ON student.sid = c.sid;
```

结果

```sql
+-------+-----+-------+-----+-------+
| sname | sid | score | cid | cname |
+-------+-----+-------+-----+-------+
| 赵雷  | 01  | 80.0  | 01  | 语文  |
| 钱电  | 02  | 70.0  | 01  | 语文  |
| 孙风  | 03  | 80.0  | 01  | 语文  |
| 周梅  | 05  | 76.0  | 01  | 语文  |
+-------+-----+-------+-----+-------+
4 rows in set
```

**解析**：先从课程表和成绩表中获取到学生号、成绩、课程号和课程名称，关联字段为课程号，作为临时表，然后再和学生表关联，关联字段为学生号，获取到学生名字

1. 求每门课程的学生人数
SELECT course.cname,a.* FROM course
JOIN
(SELECT   count(sid),cid FROM sc  GROUP BY cid) as a ON course.cid = a.cid;
结果：

```sql
+-------+------------+-----+
| cname | count(sid) | cid |
+-------+------------+-----+
| 语文  |          6 | 01  |
| 数学  |          6 | 02  |
| 英语  |          6 | 03  |
+-------+------------+-----+
3 rows in set
```

**解析**：先从成绩表中统计出每门课程的人数，再和课程表关联，关联字段为课程号，获取到课程名称

1. 成绩没有重复的情况下，查询选修「张三」老师所授课程的学生中，成绩最高的学生信息及其成绩

```sql
SELECT student.sname ,e.* FROM student 
JOIN
(SELECT MAX(d.score),c.* ,d.sid FROM sc AS d
JOIN
(SELECT a.tid,a.tname,b.cid,b.cname FROM teacher AS a
JOIN
course AS b ON a.tid = b.tid WHERE a.tname = "张三") as c ON d.cid = c.cid) as e ON student.sid = e.sid;
```

结果

```sql
+-------+--------------+-----+-------+-----+-------+-----+
| sname | MAX(d.score) | tid | tname | cid | cname | sid |
+-------+--------------+-----+-------+-----+-------+-----+
| 赵雷  | 90.0         | 01  | 张三  | 02  | 数学  | 01  |
+-------+--------------+-----+-------+-----+-------+-----+
1 row in set
```

**解析**：教师表和课程表关联，获取到教师编号、教师名称和课程编号和课程名称，关联字段为教师编号
作为临时表再和成绩表关联，关联字段为课程编号
作为临时表再和学生表关联，关联字段为学生号

1. 成绩有重复的情况下，查询选修「张三」老师所授课程的学生中，成绩最高的学生信息及其成绩

```sql
SELECT student.sname ,e.*  FROM student 
JOIN
(SELECT MAX(d.score),c.* ,d.sid,rank() over(ORDER BY MAX(d.score))as Ranked FROM sc AS d
JOIN
(SELECT a.tid,a.tname,b.cid,b.cname FROM teacher AS a
JOIN
course AS b ON a.tid = b.tid WHERE a.tname = "张三") as c ON d.cid = c.cid) as e ON student.sid = e.sid WHERE e.Ranked ;
```

结果

```sql
+-------+--------------+-----+-------+-----+-------+-----+--------+
| sname | MAX(d.score) | tid | tname | cid | cname | sid | Ranked |
+-------+--------------+-----+-------+-----+-------+-----+--------+
| 赵雷  | 90.0         | 01  | 张三  | 02  | 数学  | 01  |      1 |
+-------+--------------+-----+-------+-----+-------+-----+--------+
1 row in set
```

**解析**：用rank函数，然后再嵌套一个select，where rank = 1

1. 查询不同课程成绩相同的学生的学生编号、课程编号、学生成绩
SELECT DISTINCT a.* FROM sc AS a
    JOIN sc AS b
        ON a.score =b.score AND a.cid != b.cid
结果：

```sql
+-----+-----+-------+
| sid | cid | score |
+-----+-----+-------+
| 02  | 03  | 80.0  |
| 03  | 02  | 80.0  |
| 03  | 03  | 80.0  |
| 01  | 01  | 80.0  |
| 03  | 01  | 80.0  |
+-----+-----+-------+
5 rows in set
```

**解析**：sc表自连，distinct去重，cid 不同，score相同

1. 查询每门功成绩最好的前两名
SELECT *FROM
    (SELECT*,dense_rank()over(PARTITION BY cid ORDER BY score DESC) AS ranked FROM sc ) a
WHERE a.ranked <=2
结果：

```sql
+-----+-----+-------+--------+
| sid | cid | score | ranked |
+-----+-----+-------+--------+
| 01  | 01  | 80.0  |      1 |
| 03  | 01  | 80.0  |      1 |
| 05  | 01  | 76.0  |      2 |
| 01  | 02  | 90.0  |      1 |
| 07  | 02  | 89.0  |      2 |
| 01  | 03  | 99.0  |      1 |
| 07  | 03  | 98.0  |      2 |
+-----+-----+-------+--------+
7 rows in set
```

**解析**：我认为最好的前两名是排名的前2个，即第一个排名1 和第二个排名2，如果有两个并列第一，一个第二，那么前两名应该是3个人，用dense_rank，排名不跳过；如果说是最好的前两个人，就用rank，排名跳过

1. 统计每门课程的学生选修人数（超过 5 人的课程才统计）
SELECT  course.cname,a.* FROM course
JOIN
(SELECT  cid,COUNT(sid) as 选修人数 FROM sc GROUP BY cid HAVING COUNT(sid) >5) as a
ON course.cid = a.cid;
结果：

```sql
+-------+-----+----------+
| cname | cid | 选修人数 |
+-------+-----+----------+
| 语文  | 01  |        6 |
| 数学  | 02  |        6 |
| 英语  | 03  |        6 |
+-------+-----+----------+
3 rows in set
```

**解析**：group by，having聚合

1. 检索至少选修两门课程的学生学号
SELECT  student.sname,a.* FROM student
JOIN
(SELECT  sid,COUNT(cid) as 选修课程总数 FROM sc GROUP BY sid HAVING 选修课程总数 >=2) as a on student.sid = a.sid;
结果：

```sql
+-------+-----+--------------+
| sname | sid | 选修课程总数 |
+-------+-----+--------------+
| 赵雷  | 01  |            3 |
| 钱电  | 02  |            3 |
| 孙风  | 03  |            3 |
| 李云  | 04  |            3 |
| 周梅  | 05  |            2 |
| 吴兰  | 06  |            2 |
| 郑竹  | 07  |            2 |
+-------+-----+--------------+
7 rows in set
```

**解析**：

1. 查询选修了全部课程的学生信息

```sql
SELECT  student.*,c.`选修课程总数` from student 
JOIN
(SELECT b.sid,COUNT(a.cid) as 选修课程总数 FROM course a 
JOIN
sc b  ON a.cid = b.cid GROUP BY b.sid HAVING COUNT(a.cid) = (SELECT COUNT(cid) FROM course)) as c ON student.sid = c.sid;
```

结果

```sql
+-----+-------+---------------------+------+--------------+
| sid | sname | sage                | ssex | 选修课程总数 |
+-----+-------+---------------------+------+--------------+
| 01  | 赵雷  | 1990-01-01 00:00:00 | 男   |            3 |
| 02  | 钱电  | 1990-12-21 00:00:00 | 男   |            3 |
| 03  | 孙风  | 1990-05-20 00:00:00 | 男   |            3 |
| 04  | 李云  | 1990-08-06 00:00:00 | 男   |            3 |
+-----+-------+---------------------+------+--------------+
4 rows in set
```

**解析**：从课程表中查询出总的课程数，作为后面子查询的条件
从成绩表中查询出选修了全部课程数的的学生号和选修的课程总数
作为临时表和学生表关联，关联字段为学生号，获取到全部的学生信息

 1. 查询各学生的年龄，只按年份来算
SELECT  sname,YEAR(NOW()) - YEAR(sage) as 年龄 FROM student;
结果：

```sql
+-------+------+
| sname | 年龄 |
+-------+------+
| 赵雷  |   31 |
| 钱电  |   31 |
| 孙风  |   31 |
| 李云  |   31 |
| 周梅  |   30 |
| 吴兰  |   29 |
| 郑竹  |   32 |
| 王菊  |   31 |
+-------+------+
8 rows in set
```

**解析**：使用year函数

1. 按照出生日期来算，当前月日 < 出生年月的月日则，年龄减一

```sql
SELECT sname,
   CASE 
    WHEN (DATE_FORMAT(NOW(),'%m-%d') - DATE_FORMAT(sage,'%m-%d')) < 0 
        THEN YEAR(NOW()) - YEAR(sage) - 1
        ELSE YEAR(NOW()) - YEAR(sage)
    END AS age
FROM student;
```

结果

```sql
+-------+-----+
| sname | age |
+-------+-----+
| 赵雷  |  31 |
| 钱电  |  30 |
| 孙风  |  31 |
| 李云  |  31 |
| 周梅  |  29 |
| 吴兰  |  29 |
| 郑竹  |  32 |
| 王菊  |  31 |
+-------+-----+
8 rows in set
```

**解析**：有两种方法，一种是利用date_format直接截取时间类型中的月日，直接比大小
另外一种是用month()先比大小，相等再用day()比大小

1. 查询本周过生日的学生
SELECT sname FROM student  WHERE week(NOW()) = WEEK(sage);
结果：
Empty set
**解析**：week() 返回的是今年的第几周，即如果本周过生，返回数字相等
1. 查询下周过生日的学生
SELECT sname FROM student  WHERE week(NOW()) + 1 = WEEK(sage);
结果：
Empty set
**解析**：加一就行
1. 查询本月过生日的学生
SELECT sname FROM student  WHERE month(NOW()) = month(sage);
结果：
Empty set
**解析**：使用month函数
1. 查询下月过生日的学生
SELECT sname FROM student  WHERE month(NOW()) + 1 = month(sage);
结果：
Empty set
**解析**：加一即可
1. 49、50是删除的，这里就不罗列
