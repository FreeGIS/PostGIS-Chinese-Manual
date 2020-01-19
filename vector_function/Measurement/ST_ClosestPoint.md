## ST_ClosestPoint
### 一、方法功能描述
> 返回g1上最接近g2的二维点。本质上它返回的是最近线的startpoint

### 二、函数定义
1. 该函数只有执行方式
geometry ST_ClosestPoint(geometry g1, geometry g2);
参数定义：
```
g1 空间对象（点、线、面）
g2 空间对象（点、线、面）
--返回g1上的点
```

### 三、应用示例
###### 1、点到直线的最近点
> 在这种情况下，因为超出了起点和终点，所以不能计算垂足，只能计算出边界点。所以这种情况该函数不能够运用来作为垂足的计算。

```
with test_table as (select 
st_geomfromtext('POINT(120.142 30.265)',4326) as pointA,
st_geomfromtext('POINT(120.162 30.257)',4326) as pointB,
st_geomfromtext('POINT(120.181 30.259)',4326) as pointC
)
select ST_ClosestPoint(line,pointC),line,pointC,pointA,pointB from  (select 
st_makeline(pointA,pointB) as line ,pointC,pointB,pointA
from test_table) k
```
![image]({{book.service}}/images/Measurement/ST_ClosestPoint1.png)

###### 2、点到弧线的最近点
> 在这种情况下，该函数能够运用来作为垂足的计算

```
with test_table as (select 
st_geomfromtext('POINT(120.142 30.265)',4326) as pointA,
st_geomfromtext('POINT(120.151 30.272)',4326) as pointB,
st_geomfromtext('POINT(120.164 30.270)',4326) as pointC,
st_geomfromtext('POINT(120.155 30.262)',4326) as pointD
)
select ST_ClosestPoint(line,pointd),pointd,line from (
select st_setsrid(ST_CurveToLine(replace(st_astext(st_makeline(array[pointa,pointb,pointc])),'LINESTRING','CIRCULARSTRING')),4326)
 as line,pointd 
 from test_table
 ) k
```
![image]({{book.service}}/images/Measurement/ST_ClosestPoint2.png)

###### 3、点到直线的最近点
> 在这种情况下，该函数能够运用来作为垂足的计算

```
with test_table as (select 
st_geomfromtext('POINT(120.142 30.265)',4326) as pointA,
st_geomfromtext('POINT(120.162 30.257)',4326) as pointB,
st_geomfromtext('POINT(120.164 30.2715)',4326) as pointC
)
select ST_ClosestPoint(pointC,line),line,pointC,pointA,pointB from  (select 
st_makeline(pointA,pointB) as line ,pointC,pointB,pointA
from test_table) k
```
![image]({{book.service}}/images/Measurement/ST_ClosestPoint3.png)


###### 4、圆心到圆弧的最近点
> 在这种情况下返回的是圆弧上的任意点

```
with test_table as (select 
st_geomfromtext('POINT(120.142 30.265)',4326) as pointA
)
select ST_ClosestPoint(line,pointa),line,pointa from (
select ST_ExteriorRing(ST_Buffer(pointa,0.003)) as line,
pointa from test_table 
) k
```
![image]({{book.service}}/images/Measurement/ST_ClosestPoint4.png)

###### 5、两个多边形的最近点

```
with test_table as (select 
st_geomfromtext('POINT(120.142 30.265)',4326) as pointA,
st_geomfromtext('POINT(120.162 30.257)',4326) as pointB,
st_geomfromtext('POINT(120.164 30.271)',4326) as pointC,
st_geomfromtext('POINT(120.181 30.259)',4326) as pointA1,
st_geomfromtext('POINT(120.179 30.271)',4326) as pointB1,
st_geomfromtext('POINT(120.178 30.256)',4326) as pointC1

)
select ST_ClosestPoint(poly1,poly2) as point1 ,ST_ClosestPoint(poly2,poly1) as point2,poly1,poly2 from (select 
ST_MakePolygon(st_makeline(array[pointA,pointB,pointC,pointA]))  as poly1,ST_MakePolygon(st_makeline(array[pointA1,pointB1,pointC1,pointA1]))  as poly2
from test_table) k
```
![image]({{book.service}}/images/Measurement/ST_ClosestPoint4.png)

###### 6、两条平行线之间的最近点
> 在这种情况返回的是两个端点，若两条平行线不等长，返回任意一对

```
with test_table as (select 
st_geomfromtext('POINT(120.142 30.265)',4326) as pointA,
st_geomfromtext('POINT(120.162 30.257)',4326) as pointB
)
select ST_ClosestPoint(line,off_line) as point1,
ST_ClosestPoint(off_line,line) as point2 
from (
select st_makeline(pointa,pointb) as line ,
ST_OffsetCurve(st_makeline(pointa,pointb),0.001) as off_line 
from test_table
) k
```
![image]({{book.service}}/images/Measurement/ST_ClosestPoint5.png)

###### 7、两个重叠的多边形计算
> 在这种情况返回的值估计和大家所想的就不一样了，理论上是边界重合的任意点,这不知道属不属于bug。

```
with test_table as (select 
st_geomfromtext('POINT(120.142 30.265)',4326) as pointA,
st_geomfromtext('POINT(120.146 30.263)',4326) as pointB
)
select ST_ClosestPoint(buf1,buf2),buf1,buf2 from (
select ST_Buffer(pointa,0.003) as buf1,
ST_Buffer(pointb,0.003) as buf2 from test_table
) k
```
![image]({{book.service}}/images/Measurement/ST_ClosestPoint6.png)

### 四、总结
> 从以上示例，ST_ClosestPoint函数是比较容易理解的，但是它对应的情况比较多，大家使用的时候应该注意，在一些特殊情况的计算结果会出乎你的意料。所以如果你的数据比较复杂，可能覆盖的场景比较多的话，该函数谨慎使用。点到线与点到面的最近点计算比较合适，其他情况谨慎使用。