## ST_Angle
### 一、方法功能描述
> 该方法能够计算3点或四点够成的夹角。也能够计算两条直线构成的夹角。该函数只在postgis2.5之后的版本才能够使用，在此之此案只能使用ST_Azimuth函数。ST_Angle函数在官网中的介绍有点粗略，并不是很容易理解。
### 二、函数定义
1. 该函数有三种执行方式
```
float ST_Angle(geometry point1, geometry point2, geometry point3, geometry point4);
float ST_Angle(geometry point1, geometry point2, geometry point3);
float ST_Angle(geometry line1, geometry line2);
```
参数定义：
```
--第一种方法
    point1: 点1（geometry）
    point2：点2（geometry）
    point3: 点3（geometry）
    point4: 点4（geometry）
--第二种方法
    line1：线1（geometry）
    line2：线2（geometry）
```
### 三、应用示例
> **计算a,b,c,d四个点之间的角度和计算四点构成的线之间的角度关系**。
> 
![image]({{book.service}}/images/Measurement/ST_Angle1.png)
###### 1、计算向量AC(线)与向量BD(线)之间夹角
> 在此处用向量来代表线更容易理解一点。

![image]({{book.service}}/images/Measurement/ST_Angle2.png)

```
with test_table as (select 
st_geomfromtext('POINT(120.142 30.265)',4326) as pointA,
st_geomfromtext('POINT(120.166 30.286)',4326) as pointB,
st_geomfromtext('POINT(120.162 30.257)',4326) as pointC,
st_geomfromtext('POINT(120.164 30.2715)',4326) as pointD
)
select degrees(ST_Angle(line_1,line_2)) from (
select 
st_makeline(pointA,pointC) as line_1,
st_makeline(pointB,pointD) as line_2 
from test_table
)k 

degrees
-----------
    76.0519038156197
(1 row)
```

###### 2、计算向量AC(线)与向量DB(线)之间夹角
> 在此处用向量来代表线更容易理解一点。

![image]({{book.service}}/images/Measurement/ST_Angle3.png)

```
with test_table as (select 
st_geomfromtext('POINT(120.142 30.265)',4326) as pointA,
st_geomfromtext('POINT(120.166 30.286)',4326) as pointB,
st_geomfromtext('POINT(120.162 30.257)',4326) as pointC,
st_geomfromtext('POINT(120.164 30.2715)',4326) as pointD
)
select degrees(ST_Angle(line_1,line_2)) from (
select 
st_makeline(pointA,pointC) as line_1,
st_makeline(pointD,pointB) as line_2 
from test_table
)k 

degrees
-----------
    256.05190381562
(1 row)
```
###### 3、以点A、点C、点B三点为输入
> 该输入可以理解成向量AC与向量CB之间的夹角

![image]({{book.service}}/images/Measurement/ST_Angle4.png)

```
with test_table as (select 
st_geomfromtext('POINT(120.142 30.265)',4326) as pointA,
st_geomfromtext('POINT(120.166 30.286)',4326) as pointB,
st_geomfromtext('POINT(120.162 30.257)',4326) as pointC,
st_geomfromtext('POINT(120.164 30.2715)',4326) as pointD
)
select degrees(ST_Angle(pointA,pointC,pointB)) from test_table;

degrees
-----------
    76.0519038156206
(1 row)
```

###### 4、以点B、点C、点A三点为输入
> 该输入可以理解成向量CA与向量BC之间的夹角

![image]({{book.service}}/images/Measurement/ST_Angle5.png)

```
with test_table as (select 
st_geomfromtext('POINT(120.142 30.265)',4326) as pointA,
st_geomfromtext('POINT(120.166 30.286)',4326) as pointB,
st_geomfromtext('POINT(120.162 30.257)',4326) as pointC,
st_geomfromtext('POINT(120.164 30.2715)',4326) as pointD
)
select degrees(ST_Angle(pointB,pointC,pointA)) from test_table;

degrees
-----------
    283.948096184379
(1 row)
```
###### 5、以点A、点C、点D、点B四点为输入
> 该输入可以理解成向量AC与向量DB之间的夹角

![image]({{book.service}}/images/Measurement/ST_Angle6.png)

```
with test_table as (select 
st_geomfromtext('POINT(120.142 30.265)',4326) as pointA,
st_geomfromtext('POINT(120.166 30.286)',4326) as pointB,
st_geomfromtext('POINT(120.162 30.257)',4326) as pointC,
st_geomfromtext('POINT(120.164 30.2715)',4326) as pointD
)
select degrees(ST_Angle(pointa,pointc,pointd,pointb)) from test_table

degrees
-----------
    256.05190381562
(1 row)
```


###### 6、以点C、点A、点D、点B四点为输入
> 该输入可以理解成向量CA与向量DB之间的夹角

![image]({{book.service}}/images/Measurement/ST_Angle7.png)

```
with test_table as (select 
st_geomfromtext('POINT(120.142 30.265)',4326) as pointA,
st_geomfromtext('POINT(120.166 30.286)',4326) as pointB,
st_geomfromtext('POINT(120.162 30.257)',4326) as pointC,
st_geomfromtext('POINT(120.164 30.2715)',4326) as pointD
)
select degrees(ST_Angle(pointc,pointa,pointd,pointb)) from test_table

degrees
-----------
    76.0519038156197
(1 row)
```



###### 7、计算两条平行线之间的角度
> 在此处借助了ST_OffsetCurve函数构造了AC线的平行线，计算结果为0
![image]({{book.service}}/images/Measurement/ST_Angle8.png)

```
with test_table as (select 
st_geomfromtext('POINT(120.142 30.265)',4326) as pointA,
st_geomfromtext('POINT(120.166 30.286)',4326) as pointB,
st_geomfromtext('POINT(120.162 30.257)',4326) as pointC,
st_geomfromtext('POINT(120.180 30.271)',4326) as pointD
)
select ST_Angle(off_line,line_1) from (
select 
ST_OffsetCurve(st_makeline(pointA,pointC),0.001) as off_line,
st_makeline(pointA,pointC) as line_1  from test_table
) k

degrees
-----------
    0
(1 row)
```


###### 8、计算同一条直线(方向不同)
> 通过该示例能清楚的看到ST_Angle，本质就是一个向量夹角的计算

```
with test_table as (select 
st_geomfromtext('POINT(120.142 30.265)',4326) as pointA,
st_geomfromtext('POINT(120.166 30.286)',4326) as pointB,
st_geomfromtext('POINT(120.162 30.257)',4326) as pointC,
st_geomfromtext('POINT(120.180 30.271)',4326) as pointD
)
select degrees(ST_Angle(off_line,line_1)) from (
select 
st_makeline(pointC,pointA) as off_line,
st_makeline(pointA,pointC) as line_1  from test_table
) k

degrees
-----------
    180
(1 row)
```
### 四、总结
> 从以上示例，大家可以看到ST_Angle的本质是计算两个向量之间的角度。如果你按照官网给的例子和介绍去理解这个函数，只会让自己越来越糊涂，你无法理解角度为0的两条直线的计算结果为180度，也搞不清点的顺序造成结果的差异。使用这个函数的时候只需要抓住该函数是计算2个向量之间的夹角就OK了。不管输入是3个点还是4个点、还是两条线，理解成输入的是两个向量就行。