### SQL语法基础

---
#### 表结构

```
Student(ID,Name,Age,Sex) 学生表   
Course(ID,Name,TeacherID) 课程表   
Score(StudentID,CourseID,Score) 成绩表   
Teacher(ID,Name) 教师表
```

---
#### 查询语文（CourseID为1）比数学（CourseID为2）课程成绩高的所有学生的姓名

这是一个嵌套查询的题目，考察对子查询的使用，子查询结果作为一个集合可以当做一个独立的表来看待，子查询必须用括号括起来：

```
select st.[Name],c1.Score,c2.Score from
(select sc.[Score], sc.StudentID from Score sc  where sc.[CourseID]=1)c1,
(select sc.[Score],sc.StudentID from Score sc where sc.[CourseID]=2)c2
join Student st on st.[ID]= c1.[StudentID]
where c1.[Score]>c2.[Score] and c1.[StudentID]==c2.[StudentID]
```

---
#### 查询平均成绩大于60分的同学的学号和平均成绩
GROUP BY 语句用于结合合计函数，根据一个或多个列对结果集进行分组。GROUP BY子句在SELECT语句的WHERE子句之后并ORDER BY子句之前。WHERE 关键字无法与合计函数一起使用，**GROUP BY后面不能接WHERE条件，使用HAVING代替。**

```
select sc.[StudentID],avg(sc.Score) from Score sc
group by sc.[CourseID] having avg(sc.Score)>60
```

---
#### 查询所有同学的学号、姓名、选课数、总成绩；

```
select st.[ID],st.Name,count(sc.CourseID),sum(sc.Score) from Student st
left outer join Score sc on sc.[StudentID]=st.[ID]
group by st.[ID]
```

---
#### 查询姓“张”的老师的个数

```
select count(t.ID) from Teacher t where t.Name like '张%'
```
> SQL LIKE子句使用通配符运算符比较相似的值。符合LIKE操作符配合使用2个通配符：  
> - 百分号 (%)：百分号代表零个，一个或多个字符  
> - 下划线 (_)：下划线表示单个数字或字符

---
#### 找出教师表中姓名重复的数据，然后删除多余重复的记录，只留ID小的那个

```
select t.Name,count(t.Name) from Teacher t group by t.[Name] having count(t.Name)>1
```

删除多余的记录，写这种稍微复杂一点的sql的时候，要学会拆解，此题可以拆解为三个部分（删除+重复数据+重复数据中ID最小的数据），先分别把3个部分的sql写了，然后再一步步合并，这样就轻松多了。


```
delete from Teacher where Name in
       (select t2.Name from Teacher t2 group by t2.[Name] having count(t2.Name)>1)
and ID not in
       (select min(t3.ID) from Teacher t3 group by t3.Name having count(t3.Name)>1)
```

---
#### 按照成绩分段标示（<60不及格，60-80良，>80优），输出所有学生姓名、课程名、成绩、成绩分段标示


```
select st.Name,c.Name,sc.Score,(case
       when sc.Score > 80 then '优'       
       when sc.Score < 60 then '不及格'
       else '良' end) as 'Remark'
from Score sc
inner join Student st on st.ID=sc.[StudentID]
inner join Course c on c.ID=sc.[CourseID]
```
输出结果：  
| st-Name | c-Name | sc-Score | Remark |
| :---- | :---- | :---: | :---- |
|张三 | 语文 | 50 | 不及格 | 
|李四 | 数学 | 96 | 优 |

---
#### 查询所有课程成绩小于60分的同学的学号、姓名信息
这个比较简单，下面给出了两种方法，使用join链接和子查询：

```
select distinct st.* from Student st
left join Score sc on sc.[StudentID]=st.ID
where sc.[Score]<60
-- --
select * FROM Student st WHERE st.ID in 
       (SELECT s1.ID FROM Student s1,Score s2 WHERE s1.ID=s2.[StudentID] AND s2.Score<60)
```

---
#### 查询各科成绩最高和最低的分：以如下形式显示：课程名称，最高分，最低分

```
select c.Name as 课程,max(sc.Score) as 最高分,min(sc.Score) as 最低分 from Score sc 
left join Course c on c.ID=sc.[CourseID]
group by sc.[CourseID]
```

---
#### 查询不同老师所教不同课程平均分从高到低显示

```
select t.Name,c.Name, avg(sc.Score) from Score sc,Teacher t,Course c
where sc.[CourseID]=c.ID and c.TeacherID=t.ID
group by sc.[CourseID] order by avg(sc.Score) desc
```

---
#### 查询和“1”号的同学学习的课程完全相同的其他同学学号和姓名

```
select s1.StudentID from Score s1 where s1.CourseID in(select s2.CourseID from Score s2 where s2.StudentID=1)
group by s1.StudentID having count(*)=(select count(*) from Score s3 where s3.StudentID=1)
```

---
#### 查询选修“张老师”老师所授课程的学生中，成绩最高的学生姓名及其成绩


```
select t.Name,c.Name,s1.Score from Score s1,Teacher t,Course c 
       where t.ID=c.[TeacherID] and s1.CourseID=c.ID and t.Name='张老师'
       and s1.Score=(select max(Score) from Score where CourseID=c.ID)
```

**注意：多表连接查询有多种写法，比如本题目中多表连接可以有以下两种方式：**

```
select t.Name,c.Name,s1.Score from Score s1,Teacher t,Course c where t.ID=c.[TeacherID] and s1.CourseID=c.ID 
-- 下面的写法和上面的效果是一样的！ --
select t.Name,c.Name,s1.Score from Score s1
join Teacher t on t.ID=c.[TeacherID] 
join Course c on s1.CourseID=c.ID
```
> **FROM TABLE1,TABLE2…效果等效于FROM TABLE1 join TABLE2…，都是内连接inner操作。**

---
#### 查询所有成绩第二名到第四名的成绩


```
select  s.[StudentID],s.Score from Score s order by s.Score desc limit 2 offset 2
-- SQL 2005/2008中的分页函数是ROW_NUMBER() Over (Order by 列...)--
select  t.[StudentID],t.Score from(
        select s2.[StudentID],s2.Score,ROW_NUMBER() OVER (ORDER BY s2.[Score]) AS rn from Score s2) t
where t.rn>=2 and t.rn<=4
```
> 这是一个分页的题目，上面这是Sqlite提供的内置方法limite进行分页，不同数据库的分页方式又有些差别，但都大同小异。基本的过程都是先根据条件查询所需数据（加上行号），然后再此基础上返回指定行区间段的数据。

---
#### 查询各科成绩前2名的记录:(不考虑成绩并列情况)


```
select * from Score s1 where s1.Score 
       in(select s2.Score from Score s2 where s1.[CourseID]=s2.[CourseID] order by s2.Score desc limit 2 offset 0)
order by s1.[CourseID],s1.[Score] desc
-- 上面是sqlite中的语法，sqlite中没有top，使用limit代替，效果是一样的 --
select * from Score s1 where s1.Score 
       in(select Top 2 s2.Score from Score s2 where s1.[CourseID]=s2.[CourseID] order by s2.Score desc)
order by s1.[CourseID],s1.[Score] desc
```
