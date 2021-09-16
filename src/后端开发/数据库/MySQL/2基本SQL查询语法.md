
# 1 数据库三大范式
1. 1NF：不可分割性，只要字段值还可以继续拆分，就不满足第一范式。不可分割性，只要字段值还可以继续拆分，就不满足第一范式。
1. 2NT：在满足第一范式的前提下，主键外的每一列都必须完全依赖于主键。如果出现不完全依赖，只可能发生在联合主键的情况下。
1. 3NF：在满足第二范式的前提下，除了主键列之外，其他列之间不能有传递依赖关系，即“消除冗余”，各种信息只在一个地方存储，不出现在多张表中。

# 2 MySQL基本命令
```sql
-- 显示所有数据库
show databases;
-- 创建数据库
CREATE DATABASE test;
-- 切换数据库
use test;
-- 显示数据库中的所有表
show tables;
-- 创建数据表
CREATE TABLE pet (
    name VARCHAR(20), --类型要大写
    owner VARCHAR(20),
    species VARCHAR(20),
    sex CHAR(1),
    birth DATE,
    death DATE
);
-- 查看数据表结构
-- describe pet;
desc pet;
-- 查询表
SELECT * from pet;
-- 插入数据
INSERT INTO pet VALUES ('puffball', 'Diane', 'hamster', 'f', '1990-03-30', NULL);
-- 修改数据
UPDATE pet SET name = 'squirrel' where owner = 'Diane';
-- 删除数据
DELETE FROM pet where name = 'squirrel';
-- 删除表
DROP TABLE myorder;
```

# 3 select命令详解

## 3.1 准备练习数据
```sql
-- 创建数据库
CREATE DATABASE select_test;
-- 切换数据库
USE select_test;
-- 创建学生表
CREATE TABLE student (
    no VARCHAR(20) PRIMARY KEY,
    name VARCHAR(20) NOT NULL,
    sex VARCHAR(10) NOT NULL,
    birthday DATE, -- 生日
    class VARCHAR(20) -- 所在班级
);
-- 创建教师表
CREATE TABLE teacher (
    no VARCHAR(20) PRIMARY KEY,
    name VARCHAR(20) NOT NULL,
    sex VARCHAR(10) NOT NULL,
    birthday DATE,
    profession VARCHAR(20) NOT NULL, -- 职称
    department VARCHAR(20) NOT NULL -- 部门
);
-- 创建课程表
CREATE TABLE course (
    no VARCHAR(20) PRIMARY KEY,
    name VARCHAR(20) NOT NULL,
    t_no VARCHAR(20) NOT NULL, -- 教师编号
    -- 表示该 tno 来自于 teacher 表中的 no 字段值
    FOREIGN KEY(t_no) REFERENCES teacher(no)
);
-- 成绩表
CREATE TABLE score (
    s_no VARCHAR(20) NOT NULL, -- 学生编号
    c_no VARCHAR(20) NOT NULL, -- 课程号
    degree DECIMAL,  -- 成绩
    -- 表示该 s_no, c_no 分别来自于 student, course 表中的 no 字段值
    FOREIGN KEY(s_no) REFERENCES student(no),
    FOREIGN KEY(c_no) REFERENCES course(no),
    -- 设置 s_no, c_no 为联合主键
    PRIMARY KEY(s_no, c_no)
);
-- 添加学生表数据
INSERT INTO student VALUES('101', '曾华', '男', '1977-09-01', '95033');
INSERT INTO student VALUES('102', '匡明', '男', '1975-10-02', '95031');
INSERT INTO student VALUES('103', '王丽', '女', '1976-01-23', '95033');
INSERT INTO student VALUES('104', '李军', '男', '1976-02-20', '95033');
INSERT INTO student VALUES('105', '王芳', '女', '1975-02-10', '95031');
INSERT INTO student VALUES('106', '陆军', '男', '1974-06-03', '95031');
INSERT INTO student VALUES('107', '王尼玛', '男', '1976-02-20', '95033');
INSERT INTO student VALUES('108', '张全蛋', '男', '1975-02-10', '95031');
INSERT INTO student VALUES('109', '赵铁柱', '男', '1974-06-03', '95031');
-- 添加教师表数据
INSERT INTO teacher VALUES('804', '李诚', '男', '1958-12-02', '副教授', '计算机系');
INSERT INTO teacher VALUES('856', '张旭', '男', '1969-03-12', '讲师', '电子工程系');
INSERT INTO teacher VALUES('825', '王萍', '女', '1972-05-05', '助教', '计算机系');
INSERT INTO teacher VALUES('831', '刘冰', '女', '1977-08-14', '助教', '电子工程系');
-- 添加课程表数据
INSERT INTO course VALUES('3-105', '计算机导论', '825');
INSERT INTO course VALUES('3-245', '操作系统', '804');
INSERT INTO course VALUES('6-166', '数字电路', '856');
INSERT INTO course VALUES('9-888', '高等数学', '831');
-- 添加添加成绩表数据
INSERT INTO score VALUES('103', '3-105', '92');
INSERT INTO score VALUES('103', '3-245', '86');
INSERT INTO score VALUES('103', '6-166', '85');
INSERT INTO score VALUES('105', '3-105', '88');
INSERT INTO score VALUES('105', '3-245', '75');
INSERT INTO score VALUES('105', '6-166', '79');
INSERT INTO score VALUES('109', '3-105', '76');
INSERT INTO score VALUES('109', '3-245', '68');
INSERT INTO score VALUES('109', '6-166', '81');
```

## 3.2 select查询基本形式
```sql
-- 查询 student 表的所有行
SELECT * FROM student;
-- 查询 student 表中的指定列：name、sex 和 class
SELECT name, sex, class FROM student;
-- 查询 teacher 表中不重复的 department 列
-- DISTINCT命令: 去重查询
SELECT DISTINCT department FROM teacher;

-- 查询 score 表中成绩在60-80之间的所有行（区间查询和运算符查询）
-- BETWEEN xx AND xx: 查询区间
SELECT * FROM score WHERE degree BETWEEN 60 AND 80;
SELECT * FROM score WHERE degree > 60 AND degree < 80;
-- 查询 score 表中成绩为 85, 86 或 88 的行
-- IN: 查询规定中的多个值
SELECT * FROM score WHERE degree IN (85, 86, 88);
-- 查询 student 表中 '95031' 班或性别为 '女' 的所有行
-- or: 表示或者关系
SELECT * FROM student WHERE class = '95031' or sex = '女';
```

## 3.3 函数计算
```sql
-- 查询 "95031" 班的学生人数
-- COUNT: 统计计数，也可以用其他函数计算
SELECT COUNT(*) FROM student WHERE class = '95031';

-- AVG: 平均值
SELECT AVG(degree) FROM score WHERE c_no = '3-105';
-- 分组应用函数计算：先分组，后组内计算
SELECT c_no, AVG(degree) FROM score GROUP BY c_no;

-- 查询 score 表中的最高分的学生学号和课程编号（子查询或排序查询）。
-- MAX：最大值；MIN：最小值
SELECT s_no, c_no FROM score WHERE degree = (SELECT MAX(degree) FROM score);

--YEAR 函数与带 IN 关键字查询
--查询所有和 101 、108 号学生同年出生的 no 、name 、birthday 列
select no, name, birthday from student where year(birthday) in (
select year(birthday) from student where no in (101,108));

--查询 student 表中每个学生的姓名和年龄
select name, year(now())-year(birthday) as age from student;
```

## 3.4 排序查询
```sql
-- 以 class 降序的方式查询 student 表的所有行
-- DESC: 降序，从高到低
-- ASC（默认）: 升序，从低到高
SELECT * FROM student ORDER BY class DESC;
SELECT * FROM student ORDER BY class ASC;
-- 以 c_no 升序、degree 降序查询 score 表的所有行
SELECT * FROM score ORDER BY c_no ASC, degree DESC;

-- LIMIT r, n: 表示从第r行开始，查询n条数据
SELECT s_no, c_no, degree FROM score ORDER BY degree DESC LIMIT 0, 1;
-- LIMIT n offset r: 表示查询n条数据,从第r行开始
SELECT s_no, c_no, degree FROM score ORDER BY degree DESC LIMIT 1 offset 3;
```

## 3.5 模糊查询
```sql
-- 查询score表中至少有 2 名学生选修，并以 3 开头的课程的平均分数
-- 首先把 c_no, AVG(degree) 通过分组查询出来
-- 再查询出至少有 2 名学生选修的课程
-- HAVING: 表示持有
-- 并且是以 3 开头的课程
-- LIKE 表示模糊查询，"%" 是一个通配符，匹配 "3" 后面的任意字符。
-- 把前面的SQL语句拼接起来，
SELECT c_no, AVG(degree), COUNT(*) FROM score GROUP BY c_no HAVING COUNT(c_no) >= 2 AND c_no LIKE '3%';

-- NOT: 取反
-- LIKE: 模糊查询
-- 查询 student 表中不姓 "王" 的同学记录
mysql> SELECT * FROM student WHERE name NOT LIKE '王%';
```

## 3.6 多表查询
```sql
--查询所有学生的name，以及该学生在score表中对应的c_no和degree
-- FROM...: 表示从 student, score 表中查询
-- WHERE 的条件表示为，只有在 student.no 和 score.s_no 相等时才显示出来。
SELECT name, c_no, degree FROM student, score WHERE student.no = score.s_no;

--查询所有学生的 no 、课程名称 ( course 表中的 name ) 和成绩 ( score 表中的degree ) 列。
-- 增加一个查询字段 name，分别从 score、course 这两个表中查询。
-- as 表示取一个该字段的别名。
SELECT s_no, name as c_name, degree FROM score, course WHERE score.c_no = course.no;

--三表关联查询：查询所有学生的 name 、课程名 ( course 表中的 name ) 和 degree 
-- 由于字段名存在重复，使用 "表名.字段名 as 别名" 代替
SELECT student.name as s_name, course.name as c_name, degree
FROM student, score, course
WHERE student.no = score.s_no
AND score.c_no = course.no;

--多重select嵌套：查询计算机系的所有课程的成绩
SELECT * FROM score WHERE c_no IN (
SELECT no FROM course WHERE t_no IN (
SELECT no FROM teacher WHERE department = '计算机系'
));
```

## 3.7 高级逻辑关键字
除了and、or、in等基本关键字，还有下面的高级关键字用于逻辑判断：
```sql
--查询计算机系与电子工程系中的不同职称的教师
-- NOT: 代表逻辑非
-- UNION 合并两个集
SELECT * FROM teacher WHERE department = '计算机系' AND profession NOT IN (
    SELECT profession FROM teacher WHERE department = '电子工程系'
)
UNION
SELECT * FROM teacher WHERE department = '电子工程系' AND profession NOT IN (
    SELECT profession FROM teacher WHERE department = '计算机系'
);

--查询课程 3-105 且成绩至少高于 3-245 的 score 表
-- ANY: 符合SQL语句中的任意条件。
-- 也就是说，在 3-105 成绩中，只要有一个大于从 3-245 筛选出来的任意行就符合条件，
-- 最后根据降序查询结果。
SELECT * FROM score WHERE c_no = '3-105' AND degree > ANY(
    SELECT degree FROM score WHERE c_no = '3-245'
) ORDER BY degree DESC;

--查询课程 3-105 且成绩都高于 3-245 的 score 表
-- ALL: 符合SQL语句中的所有条件。
-- 也就是说，在 3-105 每一行成绩中，都要大于从 3-245 筛选出来全部行才算符合条件。
SELECT * FROM score WHERE c_no = '3-105' AND degree > ALL(
    SELECT degree FROM score WHERE c_no = '3-245'
);
```



# 4 Join查询

## 4.1 准备练习数据
```sql
CREATE DATABASE testJoin;
CREATE TABLE person (
    id INT,
    name VARCHAR(20),
    cardId INT
);
CREATE TABLE card (
    id INT,
    name VARCHAR(20)
);
INSERT INTO card VALUES (1, '饭卡'), (2, '建行卡'), (3, '农行卡'), (4, '工商卡'), (5, '邮政卡');
INSERT INTO person VALUES (1, '张三', 1), (2, '李四', 3), (3, '王五', 6);
```
person表并没有为cardId字段设置一个在card表中对应的 id外键。如果设置了的话，person中cardId字段值为6的行就插不进去，因为该cardId值在card表中并没有。

## 4.2 内连接
```sql
-- INNER JOIN: 表示为内连接，将两张表拼接在一起。
-- on: 表示要执行某个条件。
SELECT * FROM person INNER JOIN card on person.cardId = card.id;
-- 将 INNER 关键字省略掉，结果也是一样的。on 也可以改用where
-- 注意：card 的整张表被连接到了右边。
+------+--------+--------+------+-----------+
| id   | name   | cardId | id   | name      |
+------+--------+--------+------+-----------+
|    1 | 张三   |      1 |    1 | 饭卡      |
|    2 | 李四   |      3 |    3 | 农行卡    |
+------+--------+--------+------+-----------+
```

## 4.3 外连接

### 左外连接
完整显示左边的表 ( person ) ，右边的表如果符合条件就显示，不符合则补 NULL
```sql
-- LEFT JOIN 也叫做 LEFT OUTER JOIN，用这两种方式的查询结果是一样的。
SELECT * FROM person LEFT JOIN card on person.cardId = card.id;
+------+--------+--------+------+-----------+
| id   | name   | cardId | id   | name      |
+------+--------+--------+------+-----------+
|    1 | 张三   |      1 |    1 | 饭卡      |
|    2 | 李四   |      3 |    3 | 农行卡    |
|    3 | 王五   |      6 | NULL | NULL      |
+------+--------+--------+------+-----------+
```

### 右外连接
完整显示右边的表 ( card ) ，左边的表如果符合条件就显示，不符合则补 NULL
```sql
SELECT * FROM person RIGHT JOIN card on person.cardId = card.id;
+------+--------+--------+------+-----------+
| id   | name   | cardId | id   | name      |
+------+--------+--------+------+-----------+
|    1 | 张三   |      1 |    1 | 饭卡      |
|    2 | 李四   |      3 |    3 | 农行卡    |
| NULL | NULL   |   NULL |    2 | 建行卡    |
| NULL | NULL   |   NULL |    4 | 工商卡    |
| NULL | NULL   |   NULL |    5 | 邮政卡    |
+------+--------+--------+------+-----------+
```

### 全外连接
完整显示两张表的全部数据。但是MySQL不支持这种语法的全外连接，会出现错误：
```sql
-- SELECT * FROM person FULL JOIN card on person.cardId = card.id;
-- 出现错误：
-- ERROR 1054 (42S22): Unknown column 'person.cardId' in 'on clause'
```

MySQL中，应该使用 UNION 将两张表合并在一起。
