# MySQL常用查询整理

数据准备
-------

1、创建表格:

=== "MySQL"
    ```mysql
    CREATE TABLE student_info (
        number INT PRIMARY KEY,
        name VARCHAR(5),
        sex ENUM('男', '女'),
        id_number CHAR(18),
        department VARCHAR(30),
        major VARCHAR(30),
        enrollment_time DATE,
        UNIQUE KEY (id_number)
    );

    CREATE TABLE student_score (
        number INT,
        subject VARCHAR(30),
        score TINYINT,
        PRIMARY KEY (number, subject),
        CONSTRAINT FOREIGN KEY(number) REFERENCES student_info(number)
    );
    ```
=== "Django ORM"
    ```python
    class StudentInfo(models.Model):
        number = models.IntegerField(primary_key=True)
        name = models.CharField(max_length=5)
        sex = models.CharField(choices=[('男', '男'),('女', '女')], max_length=2)
        id_number = models.CharField(max_length=18, unique=True)
        department = models.CharField(max_length=30)
        major = models.CharField(max_length=30)
        enrollment_time = models.DateField()

        class Meta:
            db_table = "student_info"
            managed = False


    class StudentScore(models.Model):
        number = models.ForeignKey(StudentInfo, db_column='number', on_delete=models.DO_NOTHING)
        subject = models.CharField(max_length=30)
        score = models.SmallIntegerField()

        class Meta:
            db_table = "student_score"
            managed = False
    ```

2、填充数据:
```mysql
INSERT INTO student_info(number, name, sex, id_number, department, major, enrollment_time) VALUES
(20180101, '杜子腾', '男', '158177199901044792', '计算机学院', '计算机科学与工程', '2018-09-01'),
(20180102, '杜琦燕', '女', '151008199801178529', '计算机学院', '计算机科学与工程', '2018-09-01'),
(20180103, '范统', '男', '17156319980116959X', '计算机学院', '软件工程', '2018-09-01'),
(20180104, '史珍香', '女', '141992199701078600', '计算机学院', '软件工程', '2018-09-01'),
(20180105, '范剑', '男', '181048199308156368', '航天学院', '飞行器设计', '2018-09-01'),
(20180106, '朱逸群', '男', '197995199501078445', '航天学院', '电子信息', '2018-09-01');

INSERT INTO student_score (number, subject, score) VALUES
(20180101, '母猪的产后护理', 78),
(20180101, '论萨达姆的战争准备', 88),
(20180102, '母猪的产后护理', 100),
(20180102, '论萨达姆的战争准备', 98),
(20180103, '母猪的产后护理', 59),
(20180103, '论萨达姆的战争准备', 61),
(20180104, '母猪的产后护理', 55),
(20180104, '论萨达姆的战争准备', 46);
```

3、填充结果:

**student_info表**

|number|&nbsp;name&nbsp;|sex|id_number|department|&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;major&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;|enrollment_time|
|:----:|:----:|:----:|:----:|----|----|----|
|20180101|杜子腾|男|158177199901044792|计算机学院|计算机科学与工程|2018-09-01|
|20180102|杜琦燕|女|151008199801178529|计算机学院|计算机科学与工程|2018-09-01|
|20180103|范统|男|17156319980116959X|计算机学院|软件工程|2018-09-01|
|20180104|史珍香|女|141992199701078600|计算机学院|软件工程|2018-09-01|
|20180105|范剑|男|181048199308156368|航天学院|飞行器设计|2018-09-01|
|20180106|朱逸群|男|197995199501078445|航天学院|电子信息|2018-09-01|

**student_score表**    

|number|subject|score|
|:----:|:----:|:----:|
|20180101|母猪的产后护理|78|
|20180101|论萨达姆的战争准备|88|
|20180102|母猪的产后护理|100|
|20180102|论萨达姆的战争准备|98|
|20180103|母猪的产后护理|59|
|20180103|论萨达姆的战争准备|61|
|20180104|母猪的产后护理|55|
|20180104|论萨达姆的战争准备|46|

基础查询
-------

### 别名

* 方式一:`select number as 学号 from student_score`
* 方式二:`select number 学号 from student_score`    

查询结果:
=== "MySQL"
    ```mysql
    mysql> select number 学号, name 姓名 from student_info;
    ```
=== "Django ORM"
    ```python
    # 方式一
    StudentInfo.objects.annotate(学号=F('number'),姓名=F('name')).values('学号','姓名')
    # 方式二
    StudentInfo.objects.extra(select={'学号':'number', '姓名':'name'}).values('姓名','学号')
    ```
```sh title="result"
+----------+-----------+
| 学号     | 姓名      |
+----------+-----------+
| 20180101 | 杜子腾    |
| 20180102 | 杜琦燕    |
| 20180103 | 范统      |
| 20180104 | 史珍香    |
| 20180105 | 范剑      |
| 20180106 | 朱逸群    |
+----------+-----------+
6 rows in set (0.00 sec)
```

### 去重

单列去重:
```mysql
mysql> select distinct department from student_info;
+-----------------+
| department      |
+-----------------+
| 计算机学院      |
| 航天学院        |
+-----------------+
2 rows in set (0.00 sec)
```

多列去重:
=== "MySQL"
    ```mysql
    mysql> select distinct department,major from student_info;
    ```
=== "Django ORM"
    ```python
    StudentInfo.objects.values('department').distinct()
    StudentInfo.objects.values('department','major').distinct()
    ```
```sh title="result"
+-----------------+--------------------------+
| department      | major                    |
+-----------------+--------------------------+
| 计算机学院      | 计算机科学与工程         |
| 计算机学院      | 软件工程                 |
| 航天学院        | 飞行器设计               |
| 航天学院        | 电子信息                 |
+-----------------+--------------------------+
4 rows in set (0.00 sec)
```

### 限制查询结果条数

使用`limit 从哪开始，多少条`，`从哪开始`可以省略，省略代表第0行。
=== "MySQL"
    ```mysql
    mysql> select * from student_info limit 3,2;
    ```
=== "Django ORM"
    ```python
    StudentInfo.objects.all()[3:5]
    ```
```sh title="result"
+----------+-----------+------+--------------------+-----------------+-----------------+-----------------+
| number   | name      | sex  | id_number          | department      | major           | enrollment_time |
+----------+-----------+------+--------------------+-----------------+-----------------+-----------------+
| 20180104 | 史珍香    | 女   | 141992199701078600 | 计算机学院      | 软件工程        | 2018-09-01      |
| 20180105 | 范剑      | 男   | 181048199308156368 | 航天学院        | 飞行器设计      | 2018-09-01      |
+----------+-----------+------+--------------------+-----------------+-----------------+-----------------+
2 rows in set (0.00 sec)
```


### 排序

多列排序:
=== "MySQL"
    ```mysql
    mysql> select * from student_info order by name asc, number desc;
    mysql> select * from student_info order by name desc, number desc;
    ```
=== "Django ORM"
    ```python
    StudentInfo.objects.order_by('name', '-number')
    StudentInfo.objects.order_by('-name', '-number')
    ```
```sh title="result"
+----------+-----------+------+--------------------+-----------------+--------------------------+-----------------+
| number   | name      | sex  | id_number          | department      | major                    | enrollment_time |
+----------+-----------+------+--------------------+-----------------+--------------------------+-----------------+
| 20180104 | 史珍香    | 女   | 141992199701078600 | 计算机学院      | 软件工程                 | 2018-09-01      |
| 20180106 | 朱逸群    | 男   | 197995199501078445 | 航天学院        | 电子信息                 | 2018-09-01      |
| 20180101 | 杜子腾    | 男   | 158177199901044792 | 计算机学院      | 计算机科学与工程         | 2018-09-01      |
| 20180102 | 杜琦燕    | 女   | 151008199801178529 | 计算机学院      | 计算机科学与工程         | 2018-09-01      |
| 20180105 | 范剑      | 男   | 181048199308156368 | 航天学院        | 飞行器设计               | 2018-09-01      |
| 20180103 | 范统      | 男   | 17156319980116959X | 计算机学院      | 软件工程                 | 2018-09-01      |
+----------+-----------+------+--------------------+-----------------+--------------------------+-----------------+
6 rows in set (0.00 sec)

+----------+-----------+------+--------------------+-----------------+--------------------------+-----------------+
| number   | name      | sex  | id_number          | department      | major                    | enrollment_time |
+----------+-----------+------+--------------------+-----------------+--------------------------+-----------------+
| 20180103 | 范统      | 男   | 17156319980116959X | 计算机学院      | 软件工程                 | 2018-09-01      |
| 20180105 | 范剑      | 男   | 181048199308156368 | 航天学院        | 飞行器设计               | 2018-09-01      |
| 20180102 | 杜琦燕    | 女   | 151008199801178529 | 计算机学院      | 计算机科学与工程         | 2018-09-01      |
| 20180101 | 杜子腾    | 男   | 158177199901044792 | 计算机学院      | 计算机科学与工程         | 2018-09-01      |
| 20180106 | 朱逸群    | 男   | 197995199501078445 | 航天学院        | 电子信息                 | 2018-09-01      |
| 20180104 | 史珍香    | 女   | 141992199701078600 | 计算机学院      | 软件工程                 | 2018-09-01      |
+----------+-----------+------+--------------------+-----------------+--------------------------+-----------------+
6 rows in set (0.00 sec)

```

带条件查询
-------

### 简单搜索条件

不等于可以使用`<>`或`!=` :
=== "MySQL"
    ```mysql
    mysql> select * from student_info where department <> '计算机学院';
    ```
=== "Django ORM"
    ```python
    StudentInfo.objects.filter(~Q(department='计算机学院'))
    ```
```sh title="result"
+----------+-----------+------+--------------------+--------------+-----------------+-----------------+
| number   | name      | sex  | id_number          | department   | major           | enrollment_time |
+----------+-----------+------+--------------------+--------------+-----------------+-----------------+
| 20180105 | 范剑      | 男   | 181048199308156368 | 航天学院     | 飞行器设计      | 2018-09-01      |
| 20180106 | 朱逸群    | 男   | 197995199501078445 | 航天学院     | 电子信息        | 2018-09-01      |
+----------+-----------+------+--------------------+--------------+-----------------+-----------------+
2 rows in set (0.00 sec)
```

区间内使用`between...and...`，不在某区间使用`not between...and...`:
=== "MySQL"
    ```mysql
    mysql> select * from student_info where number between 20180103 and 20180105;
    mysql> select * from student_info where number not between 20180103 and 20180105;
    ```
=== "Django ORM"
    ```python
    StudentInfo.objects.filter(number__range=(20180103, 20180105))
    StudentInfo.objects.filter(~Q(number__range=(20180103, 20180105)))
    ```
```sh title="result"
+----------+-----------+------+--------------------+-----------------+-----------------+-----------------+
| number   | name      | sex  | id_number          | department      | major           | enrollment_time |
+----------+-----------+------+--------------------+-----------------+-----------------+-----------------+
| 20180103 | 范统      | 男   | 17156319980116959X | 计算机学院      | 软件工程        | 2018-09-01      |
| 20180104 | 史珍香    | 女   | 141992199701078600 | 计算机学院      | 软件工程        | 2018-09-01      |
| 20180105 | 范剑      | 男   | 181048199308156368 | 航天学院        | 飞行器设计      | 2018-09-01      |
+----------+-----------+------+--------------------+-----------------+-----------------+-----------------+
3 rows in set (0.01 sec)

+----------+-----------+------+--------------------+-----------------+--------------------------+-----------------+
| number   | name      | sex  | id_number          | department      | major                    | enrollment_time |
+----------+-----------+------+--------------------+-----------------+--------------------------+-----------------+
| 20180101 | 杜子腾    | 男   | 158177199901044792 | 计算机学院      | 计算机科学与工程         | 2018-09-01      |
| 20180102 | 杜琦燕    | 女   | 151008199801178529 | 计算机学院      | 计算机科学与工程         | 2018-09-01      |
| 20180106 | 朱逸群    | 男   | 197995199501078445 | 航天学院        | 电子信息                 | 2018-09-01      |
+----------+-----------+------+--------------------+-----------------+--------------------------+-----------------+
3 rows in set (0.00 sec)
```

### 匹配列表中的元素

使用`in (...)`和`not in`筛选出在某个列表中的记录:
=== "MySQL"
    ```mysql
    mysql> select * from student_info where major in ('软件工程',  '电子信息');
    ```
=== "Django ORM"
    ```python
    StudentInfo.objects.filter(major__in=('软件工程', '电子信息'))
    ```
```sh title="result"
+----------+-----------+------+--------------------+-----------------+--------------+-----------------+
| number   | name      | sex  | id_number          | department      | major        | enrollment_time |
+----------+-----------+------+--------------------+-----------------+--------------+-----------------+
| 20180103 | 范统      | 男   | 17156319980116959X | 计算机学院      | 软件工程     | 2018-09-01      |
| 20180104 | 史珍香    | 女   | 141992199701078600 | 计算机学院      | 软件工程     | 2018-09-01      |
| 20180106 | 朱逸群    | 男   | 197995199501078445 | 航天学院        | 电子信息     | 2018-09-01      |
+----------+-----------+------+--------------------+-----------------+--------------+-----------------+
3 rows in set (0.00 sec)
```

使用`is null` 和 `is not null`可筛选出某列是NULL的记录，而不能使用普通的操作符例如等号来进行比较，NULL代表没有值。

### 多个搜索条件

AND优先级高于OR:
=== "MySQL"
    ```mysql
    mysql> SELECT * FROM student_score WHERE score > 95 OR score < 55 AND subject = '论萨达姆的战争准备';
    ```
=== "Django ORM"
    ```python
    StudentScore.objects.filter(Q(score__gt=95) | Q(score__lt=55) & Q(subject='论萨达姆的战争准备'))
    ```
```sh title="result"
+----------+-----------------------------+-------+
| number   | subject                     | score |
+----------+-----------------------------+-------+
| 20180102 | 母猪的产后护理              |   100 |
| 20180102 | 论萨达姆的战争准备          |    98 |
| 20180104 | 论萨达姆的战争准备          |    46 |
+----------+-----------------------------+-------+
3 rows in set (0.00 sec)
```

### 模糊查询

使用`like`和`not like`进行模糊匹配查询，`%`代表任意一个字符串，而`_`代表任意一个字符:
=== "MySQL"
    ```mysql
    mysql> select * from student_info where name like '杜_';
    mysql> select * from student_info where name like '范_';
    mysql> select * from student_info where name like '%杜%';
    ```
=== "Django ORM"
    ```python
    StudentInfo.objects.extra(where=["name LIKE '杜_'"]) # django不支持_字符
    StudentInfo.objects.extra(where=["name LIKE '范_'"])
    StudentInfo.objects.filter(name__contains='杜')
    ```

```sh title="result"
Empty set (0.00 sec)

+----------+--------+------+--------------------+-----------------+-----------------+-----------------+
| number   | name   | sex  | id_number          | department      | major           | enrollment_time |
+----------+--------+------+--------------------+-----------------+-----------------+-----------------+
| 20180103 | 范统   | 男   | 17156319980116959X | 计算机学院      | 软件工程        | 2018-09-01      |
| 20180105 | 范剑   | 男   | 181048199308156368 | 航天学院        | 飞行器设计      | 2018-09-01      |
+----------+--------+------+--------------------+-----------------+-----------------+-----------------+
2 rows in set (0.00 sec)

+----------+-----------+------+--------------------+-----------------+--------------------------+-----------------+
| number   | name      | sex  | id_number          | department      | major                    | enrollment_time |
+----------+-----------+------+--------------------+-----------------+--------------------------+-----------------+
| 20180101 | 杜子腾    | 男   | 158177199901044792 | 计算机学院      | 计算机科学与工程         | 2018-09-01      |
| 20180102 | 杜琦燕    | 女   | 151008199801178529 | 计算机学院      | 计算机科学与工程         | 2018-09-01      |
+----------+-----------+------+--------------------+-----------------+--------------------------+-----------------+
2 rows in set (0.00 sec)
```

函数和表达式
-------

### 表达式

可以将表达式放在查询列表中:
=== "MySQL"
    ```mysql
    mysql> select number,subject,score+100 from student_score;
    ```
=== "Django ORM"
    ```python
    StudentScore.objects.annotate(myscore=F('score')+100).values('number', 'subject', 'myscore')
    ```
```sh title="result"
+----------+-----------------------------+-----------+
| number   | subject                     | score+100 |
+----------+-----------------------------+-----------+
| 20180101 | 母猪的产后护理              |       178 |
| 20180101 | 论萨达姆的战争准备          |       188 |
| 20180102 | 母猪的产后护理              |       200 |
| 20180102 | 论萨达姆的战争准备          |       198 |
| 20180103 | 母猪的产后护理              |       159 |
| 20180103 | 论萨达姆的战争准备          |       161 |
| 20180104 | 母猪的产后护理              |       155 |
| 20180104 | 论萨达姆的战争准备          |       146 |
+----------+-----------------------------+-----------+
8 rows in set (0.00 sec)
```

也可以把表达式作为搜索的条件:
=== "MySQL"
    ```mysql
    mysql> select * from student_score where score%3=0;
    ```
=== "Django ORM"
    ```python
    StudentScore.objects.annotate(myscore=Mod('score',3)).filter(myscore=0)
    ```
```sh title="result"
+----------+-----------------------+-------+
| number   | subject               | score |
+----------+-----------------------+-------+
| 20180101 | 母猪的产后护理        |    78 |
+----------+-----------------------+-------+
1 row in set (0.00 sec)
```

### 文本处理

连接字符串:
=== "MySQL"
    ```mysql
    mysql> select concat('学号为',number,'的学生在[',subject,']的成绩为',score) from student_score;
    ```
=== "Django ORM"
    ```python
    StudentScore.objects.annotate(mytest=Concat(Value('学号为'), 'number',Value('的学生在['), 'subject', Value(']的成绩为'), 'score', output_field=CharField()))
    ```
```sh title="result"
+--------------------------------------------------------------------------+
| concat('学号为',number,'的学生在[',subject,']的成绩为',score)            |
+--------------------------------------------------------------------------+
| 学号为20180101的学生在[母猪的产后护理]的成绩为78                         |
| 学号为20180101的学生在[论萨达姆的战争准备]的成绩为88                     |
| 学号为20180102的学生在[母猪的产后护理]的成绩为100                        |
| 学号为20180102的学生在[论萨达姆的战争准备]的成绩为98                     |
| 学号为20180103的学生在[母猪的产后护理]的成绩为59                         |
| 学号为20180103的学生在[论萨达姆的战争准备]的成绩为61                     |
| 学号为20180104的学生在[母猪的产后护理]的成绩为55                         |
| 学号为20180104的学生在[论萨达姆的战争准备]的成绩为46                     |
+--------------------------------------------------------------------------+
8 rows in set (0.00 sec)
```

子串:
=== "MySQL"
    ```mysql
    mysql> select substring(number, 4, 5) as 学号尾号 from student_info;
    ```
=== "Django ORM"
    ```python
    StudentScore.objects.annotate(num=Substr('number', 4,5)).values('num')
    ```
```sh title="result"
+--------------+
| 学号尾号     |
+--------------+
| 80104        |
| 80102        |
| 80101        |
| 80103        |
| 80105        |
| 80106        |
+--------------+
6 rows in set (0.00 sec)
```

### 时间处理函数

时间间隔:
```mysql
mysql> select date_add('2015-01-01 10:20:33', interval 2 minute);
+----------------------------------------------------+
| date_add('2015-01-01 10:20:33', interval 2 minute) |
+----------------------------------------------------+
| 2015-01-01 10:22:33                                |
+----------------------------------------------------+
1 row in set (0.00 sec)
```

时间格式化:
```mysql
mysql> select date_format(now(), '%b/%d/%Y %h:%i:%s~%p');
+--------------------------------------------+
| date_format(now(), '%b/%d/%Y %h:%i:%s~%p') |
+--------------------------------------------+
| Dec/02/2019 04:57:31~PM                    |
+--------------------------------------------+
1 row in set (0.00 sec)
```

### 聚集函数

COUNT用来统计行数
* `COUNT(*)`统计表中所有的行，包括NULL
* `COUNT(列名)`统计表中某列的所有行，不包括NULL
=== "MySQL"
    ```mysql
    mysql> select count(*) from student_info;
    mysql> select count(distinct subject) from student_score;
    ```
=== "Django ORM"
    ```python
    StudentInfo.objects.count()
    StudentScore.objects.values('subject').distinct().count()
    ```
```sh title="result"
+----------+
| count(*) |
+----------+
|        6 |
+----------+
1 row in set (0.00 sec)

+-------------------------+
| count(distinct subject) |
+-------------------------+
|                       2 |
+-------------------------+
1 row in set (0.00 sec)
```

SUM和AVG:
=== "MySQL"
    ```mysql
    mysql> select sum(score) from student_score;
    mysql> select avg(score) from student_score where subject="论萨达姆的战争准备";
    ```
=== "Django ORM"
    ```python
    StudentScore.objects.aggregate(Sum('score'))
    StudentScore.objects.filter(subject='论萨达姆的战争准备').aggregate(Avg('score'))
    ```
```sh title="result"
+------------+
| sum(score) |
+------------+
|        585 |
+------------+
1 row in set (0.00 sec)

+------------+
| avg(score) |
+------------+
|    73.2500 |
+------------+
1 row in set (0.00 sec)
```

组合使用:
=== "MySQL"
    ```mysql
    mysql> select count(*) as 成绩记录总数, max(score) as 最高分, min(score) as 最低分,avg(score) as 平均分 from student_score;
    ```
=== "Django ORM"
    ```python
    StudentScore.objects.aggregate(Max('score'), Min('score'),Avg('score'), nsum=Count('*'))
    ```
```sh title="result"
+--------------------+-----------+-----------+-----------+
| 成绩记录总数       | 最高分    | 最低分    | 平均分    |
+--------------------+-----------+-----------+-----------+
|                  8 |       100 |        46 |   73.1250 |
+--------------------+-----------+-----------+-----------+
1 row in set (0.00 sec)
```

### 隐式类型转换

Mysql会尽量把值转换为表达式中需要的类型，而不是产生错误:
```mysql
mysql> select 1 + '2';
+---------+
| 1 + '2' |
+---------+
|       3 |
+---------+
1 row in set (0.00 sec)

mysql> select '23sds'+17;
+------------+
| '23sds'+17 |
+------------+
|         40 |
+------------+
1 row in set, 1 warning (0.00 sec)
```

但这种转换不能用于存储数据:
```mysql
mysql> insert into student_score(score,number,subject) values (100,20180101,300);
Query OK, 1 row affected (0.00 sec)

mysql> insert into student_score(score,number,subject) values ('100',20180101,400);
Query OK, 1 row affected (0.00 sec)

mysql> insert into student_score(score,number,subject) values ('asd',20180101,400);
ERROR 1366 (HY000): Incorrect integer value: 'asd' for column 'score' at row 1
```

分组查询
-------

### 基础查询

分组就是针对某个列，将该列的值相同的记录分到一个组中:
=== "MySQL"
    ```mysql
    mysql> select subject, sum(score) from student_score group by subject;
    ```
=== "Django ORM"
    ```python
    StudentScore.objects.values('subject').annotate(myscore=Sum('score'))
    ```
```sh title="result"
+-----------------------------+------------+
| subject                     | sum(score) |
+-----------------------------+------------+
| 母猪的产后护理              |        292 |
| 论萨达姆的战争准备          |        293 |
+-----------------------------+------------+
2 rows in set (0.00 sec)
```

把非分组列放入查询列表中会引起争议，导致结果不确定:
```mysql
mysql> select subject, sum(score),number from student_score group by subject;
ERROR 1055 (42000): Expression #3 of SELECT list is not in GROUP BY clause and contains nonaggregated column 'student.student_score.number' which is not functionally dependent on columns in GROUP BY clause; this is incompatible with sql_mode=only_full_group_by
```

### 分组和过滤条件

是先过滤出符合条件的数据，在进行分组运算的:
=== "MySQL"
    ```mysql
    mysql> select subject, sum(score) from student_score where score>70 group by subject;
    ```
=== "Django ORM"
    ```python
    StudentScore.objects.filter(score__gt=70).values('subject').annotate(myscore=Sum('score'))
    ```
```sh title="result"
+-----------------------------+------------+
| subject                     | sum(score) |
+-----------------------------+------------+
| 母猪的产后护理              |        178 |
| 论萨达姆的战争准备          |        186 |
+-----------------------------+------------+
2 rows in set (0.00 sec)
```

也可以分组后，在筛选出合适的分组:
=== "MySQL"
    ```mysql
    mysql> select subject, sum(score) from student_score group by subject having max(score)>98;
    ```
=== "Django ORM"
    ```python
    StudentScore.objects.values('subject').annotate(sumscore=Sum('score'), mscore=Max('score')).filter(mscore__gt=98)
    ```
```sh title="result"
+-----------------------+------------+
| subject               | sum(score) |
+-----------------------+------------+
| 母猪的产后护理        |        292 |
+-----------------------+------------+
1 row in set (0.00 sec)
```

### 分组和排序
=== "MySQL"
    ```mysql
    mysql> select subject, sum(score) as sum_s from student_score group by subject order by sum_s desc;
    ```
=== "Django ORM"
    ```python
    StudentScore.objects.values('subject').annotate(sum_s=Sum('score')).order_by('-sum_s')
    ```
```sh title="result"
+-----------------------------+-------+
| subject                     | sum_s |
+-----------------------------+-------+
| 论萨达姆的战争准备          |   293 |
| 母猪的产后护理              |   292 |
+-----------------------------+-------+
2 rows in set (0.00 sec)
```

### 嵌套分组

如下例，可先按department分成大组，再按major分为小组:
=== "MySQL"
    ```mysql
    mysql> select department, major, count(*) from student_info group by department, major;
    ```
=== "Django ORM"
    ```python
    StudentInfo.objects.values('department','major').annotate(sum_c=Count('*'))
    ```
```sh title="result"
+-----------------+--------------------------+----------+
| department      | major                    | count(*) |
+-----------------+--------------------------+----------+
| 航天学院        | 电子信息                 |        1 |
| 航天学院        | 飞行器设计               |        1 |
| 计算机学院      | 计算机科学与工程         |        2 |
| 计算机学院      | 软件工程                 |        2 |
+-----------------+--------------------------+----------+
4 rows in set (0.00 sec)
```

### 注意事项

1. 如果分组列中有NULL值，那么NULL会作为一个独立的分组
2. 如果是嵌套分组，聚集函数将作用在最后的分组列上
3. 非分组列不能单独出现在检索列表中(可以被放到聚集函数中)
4. GROUP BY子句后可以跟随表达式(但不能是聚集函数)

简单查询语句中各子句的顺序为:
```mysql
SELECT [DISTINCT] 查询列表
[FROM 表名]
[WHERE 布尔表达式]
[GROUP BY 分组列表 ]
[HAVING 分组过滤条件]
[ORDER BY 排序列表]
[LIMIT 开始行, 限制条数]
```

子查询
-------

### 标量子查询

标量子查询单纯的代表一个值，可以作为表达式参与运算或作为搜索条件:
=== "MySQL"
    ```mysql
    mysql> select * from student_score where number=(select number from student_info where name='范统');
    ```
=== "Django ORM"
    ```python
    StudentScore.objects.filter(number=Subquery(StudentInfo.objects.filter(name='范统').values('number')))
    ```
```sh title="result"
+----------+-----------------------------+-------+
| number   | subject                     | score |
+----------+-----------------------------+-------+
| 20180103 | 母猪的产后护理              |    59 |
| 20180103 | 论萨达姆的战争准备          |    61 |
+----------+-----------------------------+-------+
2 rows in set (0.00 sec)
```

### 列子查询

内层查询结果不是一个单独的值，而是一个列:
```mysql
mysql> select * from student_score where number=(select number from student_info where sex='男');
ERROR 1242 (21000): Subquery returns more than 1 row
```
=== "MySQL"
    ```mysql
    mysql> select * from student_score where number in (select number from student_info where sex='男');
    ```
=== "Django ORM"
    ```python
    StudentScore.objects.filter(number__in=Subquery(StudentInfo.objects.filter(sex='男').values('number')))
    ```
```sh title="result"
+----------+-----------------------------+-------+
| number   | subject                     | score |
+----------+-----------------------------+-------+
| 20180101 | 母猪的产后护理              |    78 |
| 20180101 | 论萨达姆的战争准备          |    88 |
| 20180103 | 母猪的产后护理              |    59 |
| 20180103 | 论萨达姆的战争准备          |    61 |
+----------+-----------------------------+-------+
4 rows in set (0.00 sec)
```

而行子查询、表子查询不常用，省略。

### EXISTS和相关子查询

EXISTS和NOT EXISTS单独看很像一个函数，返回查询结果是否为空集:
```mysql
mysql> select exists (select * from student_info where number=20180101);
+-----------------------------------------------------------+
| exists (select * from student_info where number=20180101) |
+-----------------------------------------------------------+
|                                                         1 |
+-----------------------------------------------------------+
1 row in set (0.00 sec)

mysql> select not exists (select * from student_info where number=20180101);
+---------------------------------------------------------------+
| not exists (select * from student_info where number=20180101) |
+---------------------------------------------------------------+
|                                                             0 |
+---------------------------------------------------------------+
1 row in set (0.00 sec)
```

之前我们尝试的都是不相关子查询，而相关子查询就是内层查询语句要用到外层查询语句的值，比如我们查学生的基本信息并要求这些学生有成绩的记录:
```mysql
mysql> select * from student_info where exists(select * from student_score where student_score.number=student_info.number);
+----------+-----------+------+--------------------+-----------------+--------------------------+-----------------+
| number   | name      | sex  | id_number          | department      | major                    | enrollment_time |
+----------+-----------+------+--------------------+-----------------+--------------------------+-----------------+
| 20180101 | 杜子腾    | 男   | 158177199901044792 | 计算机学院      | 计算机科学与工程         | 2018-09-01      |
| 20180102 | 杜琦燕    | 女   | 151008199801178529 | 计算机学院      | 计算机科学与工程         | 2018-09-01      |
| 20180103 | 范统      | 男   | 17156319980116959X | 计算机学院      | 软件工程                 | 2018-09-01      |
| 20180104 | 史珍香    | 女   | 141992199701078600 | 计算机学院      | 软件工程                 | 2018-09-01      |
+----------+-----------+------+--------------------+-----------------+--------------------------+-----------------+
4 rows in set (0.00 sec)
```

这个相关子查询的查询过程是:先执行外层查询获得到student_info表的第一条记录，发现它的number值是20180101。把20180101当作参数传入到子查询，此时子查询的意思是判断student_score表的number字段是否有20180101这个值存在，子查询的结果是该值存在，所以整个EXISTS表达式的值为TRUE，那么student_info表的第一条记录可以被加入到结果集。每条记录依次按这个过程执行。

此外，子查询还可以应用于同一个表，比如我们去查student_score表中分数大于平均分的记录，第一印象可能是如下写法:
```mysql
mysql> select * from student_score where score > avg(score);
ERROR 1111 (HY000): Invalid use of group function
```

实际应该使用子查询来实现:
```mysql
mysql> select * from student_score where score > (select avg(score) from student_score);
+----------+-----------------------------+-------+
| number   | subject                     | score |
+----------+-----------------------------+-------+
| 20180101 | 母猪的产后护理              |    78 |
| 20180101 | 论萨达姆的战争准备          |    88 |
| 20180102 | 母猪的产后护理              |   100 |
| 20180102 | 论萨达姆的战争准备          |    98 |
+----------+-----------------------------+-------+
4 rows in set (0.00 sec)
```
因为聚集函数不能用于WHERE子句，可以把上述写法看做是给student_score做了一个副本。

连接查询
-------

### 基础概念
连接的本质就是将各个表中的记录都拉取出来，依次匹配组合形成一个结果集，也就是笛卡尔积的方式。

我们来看一个示例:
```mysql
mysql> create table t1(m1 int, n1 char(1));
Query OK, 0 rows affected (0.02 sec)

mysql> create table t2(m2 int, n2 char(1));
Query OK, 0 rows affected (0.02 sec)

mysql> insert into t1 values(1, 'a'),(2, 'b'),(3, 'c');
Query OK, 3 rows affected (0.00 sec)
Records: 3  Duplicates: 0  Warnings: 0

mysql> insert into t2 values(2, 'a'),(3, 'b'),(4, 'c');
Query OK, 3 rows affected (0.00 sec)
Records: 3  Duplicates: 0  Warnings: 0
```

新建了两个表，并各插入了三条数据，那么连接可以这样做:
```mysql
mysql> select * from t1,t2;
+------+------+------+------+
| m1   | n1   | m2   | n2   |
+------+------+------+------+
|    1 | a    |    2 | a    |
|    2 | b    |    2 | a    |
|    3 | c    |    2 | a    |
|    1 | a    |    3 | b    |
|    2 | b    |    3 | b    |
|    3 | c    |    3 | b    |
|    1 | a    |    4 | c    |
|    2 | b    |    4 | c    |
|    3 | c    |    4 | c    |
+------+------+------+------+
9 rows in set (0.00 sec)
```

使用以下写法连接都是可以的:

* `select t1.m1,t1.n1,t2.m2,t2.n2 from t1, t2`
* `select m1,n1,m2,n2 from t1, t2`
* `select t1.*,t2.* from t1, t2`

### 内外连接

现在我们想通过一条语句既查到学生的基本信息，又查到他的成绩信息:
=== "MySQL"
    ```mysql
    mysql> select student_info.number,name,sex,subject,score from student_info, student_score where student_info.number = student_score.number;
    ```
=== "Django ORM"
    ```python
    StudentScore.objects.values('number__name','number__name','number__sex','subject','score')
    ```
```sh title="result"
+----------+-----------+------+-----------------------------+-------+
| number   | name      | sex  | subject                     | score |
+----------+-----------+------+-----------------------------+-------+
| 20180101 | 杜子腾    | 男   | 母猪的产后护理              |    78 |
| 20180101 | 杜子腾    | 男   | 论萨达姆的战争准备          |    88 |
| 20180102 | 杜琦燕    | 女   | 母猪的产后护理              |   100 |
| 20180102 | 杜琦燕    | 女   | 论萨达姆的战争准备          |    98 |
| 20180103 | 范统      | 男   | 母猪的产后护理              |    59 |
| 20180103 | 范统      | 男   | 论萨达姆的战争准备          |    61 |
| 20180104 | 史珍香    | 女   | 母猪的产后护理              |    55 |
| 20180104 | 史珍香    | 女   | 论萨达姆的战争准备          |    46 |
+----------+-----------+------+-----------------------------+-------+
8 rows in set (0.00 sec)
```

这时候我们发现有两个人没有成绩，所以他们没有显示在查询结果中。为了有办法让其显示出，就有了内连接和外连接的概念:

* 内连接就是我们之前使用的，没有匹配的记录则结果不会加入到最后的结果集
* 对于外连接的两个表，驱动表中的记录即使在被驱动表中没有匹配的记录，也仍然需要加入到结果集

而外连接又分为左外连接，即左侧的表为驱动表。右外连接，即右侧的表为驱动表。

此外，WHERE子句不论内外连接，凡是不符合WHERE条件的记录都不会在最后的结果集中。而对于外连接的驱动表，ON可以将被驱动表中找不到记录的对应驱动表记录加入到结果集，内连接中WHERE子句和ON子句是等价的。所以我们一般会把WHERE用于涉及单表的过滤条件，ON用于涉及多表的过滤条件。

外连接的标准语法为: `select * from t1 left/right [outer] join t2 on 连接条件 [where 普通过滤条件]`，outer和where可省略。

上例中使用外连接的结果为:
=== "MySQL"
    ```mysql
    mysql> select student_info.number,name,sex,subject,score from student_info left join student_score on student_info.number = student_score.number;
    ```
=== "Django ORM"
    ```python
    StudentInfo.objects.values('number', 'name','sex','studentscore__subject','studentscore__score')
    ```
```sh title="result"
+----------+-----------+------+-----------------------------+-------+
| number   | name      | sex  | subject                     | score |
+----------+-----------+------+-----------------------------+-------+
| 20180101 | 杜子腾    | 男   | 母猪的产后护理              |    78 |
| 20180101 | 杜子腾    | 男   | 论萨达姆的战争准备          |    88 |
| 20180102 | 杜琦燕    | 女   | 母猪的产后护理              |   100 |
| 20180102 | 杜琦燕    | 女   | 论萨达姆的战争准备          |    98 |
| 20180103 | 范统      | 男   | 母猪的产后护理              |    59 |
| 20180103 | 范统      | 男   | 论萨达姆的战争准备          |    61 |
| 20180104 | 史珍香    | 女   | 母猪的产后护理              |    55 |
| 20180104 | 史珍香    | 女   | 论萨达姆的战争准备          |    46 |
| 20180105 | 范剑      | 男   | NULL                        |  NULL |
| 20180106 | 朱逸群    | 男   | NULL                        |  NULL |
+----------+-----------+------+-----------------------------+-------+
10 rows in set (0.00 sec)
```

内连接以下的写法是等价的:

* `select * from t1, t2`
* `select * from t1 join t2`
* `select * from t1 inner join t2`
* `select * from t1 cross join t2`

综上，我们总结以下三种连接的结果差异:
```mysql
mysql> select * from t1 inner join t2 on t1.m1=t2.m2;
+------+------+------+------+
| m1   | n1   | m2   | n2   |
+------+------+------+------+
|    2 | b    |    2 | a    |
|    3 | c    |    3 | b    |
+------+------+------+------+
2 rows in set (0.00 sec)

mysql> select * from t1 left join t2 on t1.m1=t2.m2;
+------+------+------+------+
| m1   | n1   | m2   | n2   |
+------+------+------+------+
|    2 | b    |    2 | a    |
|    3 | c    |    3 | b    |
|    1 | a    | NULL | NULL |
+------+------+------+------+
3 rows in set (0.00 sec)

mysql> select * from t1 right join t2 on t1.m1=t2.m2;
+------+------+------+------+
| m1   | n1   | m2   | n2   |
+------+------+------+------+
|    2 | b    |    2 | a    |
|    3 | c    |    3 | b    |
| NULL | NULL |    4 | c    |
+------+------+------+------+
3 rows in set (0.00 sec)
```

### 多表连接

我们可以连接任意数量的表，我们再加入一张表试验:
```mysql
mysql> create table t3(m3 int, n3 char(1));
Query OK, 0 rows affected (0.07 sec)

mysql> insert into t3 values(3, 'a'),(3, 'b'),(4, 'c');
Query OK, 3 rows affected (0.01 sec)
Records: 3  Duplicates: 0  Warnings: 0
```

我们使用下面的语法查询是等价的:

* `select * from t1 inner join t2 inner join t3 where t1.m1=t2.m2 and t2.m2=t3.m3;`
* `select * from t1 inner join t2 on t1.m1=t2.m2 inner join t3 on t1.m1=t3.m3;`

查询结果:
```mysql
mysql> select * from t1 inner join t2 on t1.m1=t2.m2 inner join t3 on t1.m1=t3.m3;
+------+------+------+------+------+------+
| m1   | n1   | m2   | n2   | m3   | n3   |
+------+------+------+------+------+------+
|    3 | c    |    3 | b    |    3 | a    |
|    3 | c    |    3 | b    |    3 | b    |
+------+------+------+------+------+------+
2 rows in set (0.00 sec)
```
也可以用伪代码来描述:
```
for each_row in t1{
    for each_row in t2 which t1.m1=t2.m2{
        for each_row in t3 which t2.m2=t3.m3{
            add_result(each_row)
        }
    }
}
```

### 自连接

我们无法直接自连接，但可以通过别名:
```mysql
mysql> select * from t1, t1;
ERROR 1066 (42000): Not unique table/alias: 't1'
mysql> select * from t1 as table1, t1 as table2;
+------+------+------+------+
| m1   | n1   | m1   | n1   |
+------+------+------+------+
|    1 | a    |    1 | a    |
|    2 | b    |    1 | a    |
|    3 | c    |    1 | a    |
|    1 | a    |    2 | b    |
|    2 | b    |    2 | b    |
|    3 | c    |    2 | b    |
|    1 | a    |    3 | c    |
|    2 | b    |    3 | c    |
|    3 | c    |    3 | c    |
+------+------+------+------+
9 rows in set (0.00 sec)
```

而自连接的意义，比如要查询与'范统'的专业相同的同学:
```mysql
mysql> select * from student_info as s1, student_info as s2 where s1.name='范统' and s1.major=s2.major;
+----------+--------+------+--------------------+-----------------+--------------+-----------------+----------+-----------+------+--------------------+-----------------+--------------+-----------------+
| number   | name   | sex  | id_number          | department      | major        | enrollment_time | number   | name      | sex  | id_number          | department      | major        | enrollment_time |
+----------+--------+------+--------------------+-----------------+--------------+-----------------+----------+-----------+------+--------------------+-----------------+--------------+-----------------+
| 20180103 | 范统   | 男   | 17156319980116959X | 计算机学院      | 软件工程     | 2018-09-01      | 20180103 | 范统      | 男   | 17156319980116959X | 计算机学院      | 软件工程     | 2018-09-01      |
| 20180103 | 范统   | 男   | 17156319980116959X | 计算机学院      | 软件工程     | 2018-09-01      | 20180104 | 史珍香    | 女   | 141992199701078600 | 计算机学院      | 软件工程     | 2018-09-01      |
+----------+--------+------+--------------------+-----------------+--------------+-----------------+----------+-----------+------+--------------------+-----------------+--------------+-----------------+
2 rows in set (0.00 sec)
```

### 与子查询转换

有的需求既可以用连接查询，也可以用子查询:
=== "MySQL"
    ```mysql
    mysql> select * from student_score where number in (select number from student_info where major='软件工程');
    mysql> select s2.* from student_score as s2, student_info as s1 where s1.number=s2.number and s1.major='软件工程';
    ```
=== "Django ORM"
    ```python
    StudentScore.objects.filter(number__in=Subquery(StudentInfo.objects.filter(major='软件工程').values('number')))
    StudentScore.objects.select_related('number').filter(number__major='软件工程')
    ```
```sh title="result"
+----------+-----------------------------+-------+
| number   | subject                     | score |
+----------+-----------------------------+-------+
| 20180103 | 母猪的产后护理              |    59 |
| 20180103 | 论萨达姆的战争准备          |    61 |
| 20180104 | 母猪的产后护理              |    55 |
| 20180104 | 论萨达姆的战争准备          |    46 |
+----------+-----------------------------+-------+
4 rows in set (0.00 sec)

+----------+-----------------------------+-------+
| number   | subject                     | score |
+----------+-----------------------------+-------+
| 20180103 | 母猪的产后护理              |    59 |
| 20180103 | 论萨达姆的战争准备          |    61 |
| 20180104 | 母猪的产后护理              |    55 |
| 20180104 | 论萨达姆的战争准备          |    46 |
+----------+-----------------------------+-------+
4 rows in set (0.00 sec)
```