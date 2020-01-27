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