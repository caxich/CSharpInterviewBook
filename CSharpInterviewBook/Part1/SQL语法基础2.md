所有sql语句基于SqlServer数据库

#### 分页的几种写法：
查询TMS_Enterprise表的第二页内容，每页10条数据
1. **top + not in**
```
SELECT TOP 10 *
FROM dbo.TMS_Enterprise
WHERE Id NOT IN(SELECT TOP(10 * 2)Id FROM dbo.TMS_Enterprise ORDER BY Id)
ORDER BY Id
```
**注意：前后order by字段及参数需一致**

2. **ROW_NUMBER()**
```
SELECT * 
FROM (SELECT *,ROW_NUMBER() OVER(ORDER BY Id) rown FROM dbo.TMS_Enterprise) t
WHERE t.rown BETWEEN (10 * (2-1) + 1) AND 10 * 2
```
**注意：SqlServer2005以上支持**

3. **offset + fetch next （最佳）**
```
SELECT * FROM dbo.TMS_Enterprise
ORDER BY Id OFFSET (10 * (2-1)) ROWS FETCH NEXT 10 ROWS ONLY
```
**注意：SqlServer2012以上支持**

---

#### with as 用法
```
with
cte1 as
(
    select * from table1 where name like 'abc%'
),
cte2 as
(
    select * from table2 where id > 20
),
cte3 as
(
    select * from table3 where price < 100
)
select a.* from cte1 a, cte2 b, cte3 c where a.id = b.id and a.id = c.id
```
---

#### 时间相关函数
- **datepart()**
```
/* datepart()函数的使用                          
 * datepart()函数可以方便的取到时期中的各个部分
 *如日期：2006-07--02 18：15：36.513
 * yy/year:取年           2006
 * mm/month:取月           7
 * dd/day:取月中的天     2
 * dy/dayofyear:取年中的天     183
 * wk/week:取年中的周     27
 * dw/weekday:取周中的天     1
 * qq/quarter:取年中的季度   3
 * hh/hour:取小时        18
 * mi/minute:取分钟        15
 * ss/second:取秒          36
 * 以下简单的语句可以演示所取到的结果
 */
select getdate()
select datepart(mm,getdate())
select datepart(yy,getDate())
select datepart(dd,getdate())
select datepart(dy,getdate())
select datepart(wk,getdate())
select datepart(dw,getdate())
select datepart(qq,getdate())
select datepart(hh,getdate())
select datepart(mi,getdate())
select datepart(ss,getdate())
```
- **datediff()**
```
select datediff(dd,getdate(),'12/25/2006')　　  --计算从今天到12/25/2006还有多少天
select datediff(mm,getdate(),'12/25/2006')　　--计算从今天到12/25/2006还有多少个月
```

- **dateadd()**
```
select dateadd(dd,30,getdate())           　　　  --在目前的日期日期上加30天
select dateadd(mm,3,getdate())           　　　  --在目前的日期日期上加3个月
select dateadd(yy,1,getdate())           　　　　  --在目前的日期日期上加1年
```
---
#### partition by 用法

**语法**
```
select row_number() over(partition by A order by B ) as rowIndex from table
```
**表结构**
```
create table [OrderInfo](
        [Id] [int] PRIMARY KEY  IDENTITY(1,1) NOT NULL,
        [UserId] [nvarchar](50) NOT NULL,
        [TotalPrice] [float] NOT NULL,
        [OrderTime] [datetime] NOT NULL,
 );
```

1. **所有订单按照客户进行分组，并按照客户下的订单的金额倒序排列：**
```
select Id,UserId,orderTime,ROW_NUMBER() over(partition by UserId order by TotalPrice desc) as rowIndex from OrderInfo
```

2. **筛选出客户第一次下的订单**
```
 with
 baseDate
 as
 (
     select Id,UserId,TotalPrice,orderTime,ROW_NUMBER() over (partition by UserId order by orderTime) as rowIndex from OrderInfo
 )
 select * from baseDate where rowIndex=1
```

#### T-SQL流程控制语句
 **while循环**
```
declare @i int, @temp int;
set @i = 0;
select @temp = MAX(Id) from TMS_Enterprise;
while(@i <= @temp)
begin
	set @i = @i + 1;
	if exists(select * from TMS_Enterprise where Id = @i)
		select * from TMS_Enterprise where Id = @i;
	else
		select 'ID为'+CONVERT(varchar(20),@i)+'记录不存在';
end
```

#### 如何避免全表扫描
1. **应尽量避免在where子句中对字段进行null值判断**
>创建表时NULL是默认值，但大多数时候应该使用NOT NULL，或者使用一个特殊的值，如0，-1作为默认值。null值无法用作索引，任何包含null值的列都将不会被包含在索引中。即使索引有多列这样的情况下，只要这些列中有一列含有null，该列就会从索引中排除。任何在where子句中使用is null或is not null的语句优化器是不允许使用索引的。
2. **应尽量避免在where 子句中使用!=或<>操作符以及全模糊查询（%p%）**
>MySQL只有对以下操作符才使用索引：<，<=，=，>，>=，BETWEEN，IN，以及某些时候的LIKE。 可以在LIKE操作中使用索引的情形是指另一个操作数不是以通配符（%或者_）开头的情形。例如，“SELECT id FROM t WHERE col LIKE 'Mich%';”这个查询将使用索引，但“SELECT id FROM t WHERE col  LIKE '%ike';”这个查询不会使用索引。
3. **应尽量避免在where子句中使用or来连接条件**
>如：select id from t where num=10 or num=20  
    可以这样查询：select id from t where num=10 union all select id from t where num=20
4. **慎用in 和not in，用exists 代替in**
>如：
   select id from t where num in(1,2,3)  
   对于连续的数值，能用between 就不要用in 了：
   select id from t where num between 1 and 3  
select num from a where num in(select num from b)  
用下面的语句替换：
select num from a where exists(select 1 from b where num=a.num)
5. **应尽量避免在where 子句中对字段进行表达式操作**
>如：
  select id from t where num/2=100  
应改为:
 select id from t where num=100*2
6. **应尽量避免在where子句中对字段进行函数操作**
>如：
select id from t where substring(name,1,3)='abc'--name  
select id from t where datediff(day,createdate,'2005-11-30')=0--‘2005-11-30’   
应改为:
select id from t where name like 'abc%'  
select id from t where createdate>='2005-11-30' and createdate<'2005-12-1'
