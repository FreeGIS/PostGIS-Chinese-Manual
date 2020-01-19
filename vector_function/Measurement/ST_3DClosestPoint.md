## <span id='ST_3DClosestPoint'>ST_3DClosestPoint</span>
### 一、方法功能描述
> 返回g1上最接近g2的三维维点。本质上它返回的是3d最短线的startpoint。ST_3DClosestPoint函数和ST_ClosestPoint的功能类似，但是ST_3DClosestPoint函数比ST_ClosestPoint强大。使用ST_3DClosestPoint计算时，如果输入的是三维的，则返回三维数据。如果是二维的则返回二维数据。而ST_ClosestPoint即使是三维输入，Z值也会被舍弃，不会参与计算，返回的仍是二维数据。

### 二、函数定义
1. 该函数只有执行方式
geometry ST_3DClosestPoint(geometry g1, geometry g2);
参数定义：

```
g1 空间对象（点、线、面）
g2 空间对象（点、线、面）
--返回g1上的点
```

### 三、应用示例
###### 1、三维点到三维曲线的最近点
> 在此示例中利用点a，b,c三点构建三维曲线，并计算三维点D与曲线的最近点。

```
with test_table as (select 
st_geomfromtext('POINT(120.142 30.265 20000)',4326) as pointA,
st_geomfromtext('POINT(120 31 10000)',4326) as pointB,
st_geomfromtext('POINT(121 31 0)',4326) as pointC,
st_geomfromtext('POINT(120.55 30.62 10000)',4326) as pointD
)
select st_astext(ST_3DClosestPoint(line,pointd)) as geom from (
select st_setsrid(
ST_CurveToLine(
replace(st_astext(st_makeline(array[pointa,pointb,pointc])),'LINESTRING','CIRCULARSTRING')
),
4326
)
 as line,pointd 
 from test_table
 ) k
 
geom
-----------
    POINT Z (120.000631125869 31.0009819870587 10000.0000060715)
(1 row)
```
![image]({{book.service}}/images/Measurement/ST_3DClosestPoint1.png)

###### 2、二维点到三维弧线的最近点
> 在这种情况下，该函数其实只能当作ST_ClosestPoint函数使用，唯一的区别是ST_ClosestPoint返回的二维点，ST_3DClosestPoint返回的是三维点，这两个结果都都投影到平面，其实是同一个点。在这中情况Z值是不参与计算的。而Z值的取值则是2D结果点在3D弧线的投射。

```
with test_table as (select 
st_geomfromtext('POINT(120.142 30.265 20000)',4326) as pointA,
st_geomfromtext('POINT(120 31 10000)',4326) as pointB,
st_geomfromtext('POINT(121 31 0)',4326) as pointC,
st_geomfromtext('POINT(120.348 30.895)',4326) as pointD
)
select 
st_astext(ST_3DClosestPoint(line,pointd)) point1,
st_astext(ST_ClosestPoint(line,pointd)) point2 from (
select st_setsrid(
ST_CurveToLine(
replace(st_astext(st_makeline(array[pointa,pointb,pointc])),'LINESTRING','CIRCULARSTRING')
),
4326
)
 as line,pointd 
 from test_table
 ) k;
 
point1|point2
-----------
  POINT Z (120.134709079275 31.1596818998198 8266.44949070754)	POINT(120.134709073393 31.1596818950798)
(1 row)
```
![image]({{book.service}}/images/Measurement/ST_3DClosestPoint2.png)

###### 3、二维点与三维弧线
> 在这种情况下，ST_3DClosestPoint与ST_ClosestPoint的计算结果没有任何差别。

```
with test_table as (select 
st_geomfromtext('POINT(120.142 30.265)',4326) as pointA,
st_geomfromtext('POINT(120 31)',4326) as pointB,
st_geomfromtext('POINT(121 31)',4326) as pointC,
st_geomfromtext('POINT(120.348 30.895)',4326) as pointD
)
select 
st_astext(ST_3DClosestPoint(line,pointd)) point1,
st_astext(ST_ClosestPoint(line,pointd)) point2 from (
select st_setsrid(
ST_CurveToLine(
replace(st_astext(st_makeline(array[pointa,pointb,pointc])),'LINESTRING','CIRCULARSTRING')
),
4326
)
 as line,pointd 
 from test_table
 ) k;
 point1|point2
-----------
 POINT(120.134709073393 31.1596818950798)|	POINT(120.134709073393 31.1596818950798)
(1 row)
```



###### 4、三维点到2D线的最近点
> 这个例子相当有意思，一个2d线输入和一个3D点的输入的情况下，ST_3DClosestPoint的计算结果等于ST_ClosestPoint的计算的结果投射到3D点所在的平面。

```
with test_table as (select 
st_geomfromtext('POINT(120.142 30.265)',4326) as pointA,
st_geomfromtext('POINT(120 31)',4326) as pointB,
st_geomfromtext('POINT(121 31)',4326) as pointC,
st_geomfromtext('POINT(120.348 30.895 1000)',4326) as pointD
)
select 
st_astext(ST_3DClosestPoint(line,pointd)) point1,
st_astext(ST_ClosestPoint(line,pointd)) point2 from (
select st_setsrid(
ST_CurveToLine(
replace(st_astext(st_makeline(array[pointa,pointb,pointc])),'LINESTRING','CIRCULARSTRING')
),
4326
)
 as line,pointd 
 from test_table
 ) k;
  point1|point2
-----------
POINT Z (120.134709073393 31.1596818950798 1000) |	POINT(120.134709073393 31.1596818950798)
(1 row)
```



### 四、总结
> 从以上示例，ST_3DClosestPoint函数看似很简单，但是如果真的细究里面的细节会发现有很大出入。所以在使用该函数的时候要严抓输入，既然该函数针对的场景是三维计算，那尽量让这个函数的入参是3d数据，不然真得出现什么意外得计算结果，很难发现是哪里出了错。