## ST_Azimuth
### 一、方法功能描述
> 该方法返回两点之间弧度方位角，如果两个点重合，则返回NULL。方位角是从北方参考的角度，顺时针为正：North = 0; 东方=π/ 2; 南=π; 西方=3π/ 2。方位角是数学概念，定义为参考平面和点之间的角度，单位为弧度。如示例所示，可以使用内置的PostgreSQL函数degree（）将单位转换为度。**在postgis2.5之前，是postgis中唯一一个能计算角度的函数**。
![image]({{book.service}}/images/Measurement/ST_Azimuth1.png)

### 二、函数定义
1. 该函数的参数支持geometry与geography两种数据类型 
```
float ST_Azimuth(geometry pointA, geometry pointB);
float ST_Azimuth(geography pointA, geography pointB);
```

参数定义：
```
    pointA: 点A（geometry或geography）
    pointB：点B（geometry或geography）
```
### 三、应用示例
> 计算A，B两点之间的方位角

![image]({{book.service}}/images/Measurement/ST_Azimuth2.png)
###### 使用geomtry参与计算

```
with test_table as (select 
st_geomfromtext('POINT(120.142 30.265)',4326) as pointA,
st_geomfromtext('POINT(120.166 30.286)',4326) as pointb
)
select degrees(ST_Azimuth(pointa,pointb)) from test_table;
degrees
-----------
    48.8140748342904
(1 row)
```
###### 使用geography参与计算，计算球面角度

```
with test_table as (select 
st_geomfromtext('POINT(120.142 30.265)',4326) as pointA,
st_geomfromtext('POINT(120.166 30.286)',4326) as pointb)
select degrees(ST_Azimuth(pointa::geography,pointb::geography)) from test_table;
-----------
    44.7622686457229
(1 row)
```

###### 两个重合的点进行角度计算

```
with test_table as (select 
st_geomfromtext('POINT(120.142 30.265)',4326) as pointA,
st_geomfromtext('POINT(120.166 30.286)',4326) as pointb)
select degrees(ST_Azimuth(pointa::geography,pointa::geography)) from test_table;
-----------
    null
(1 row)
```

###### 一个和官网说法不附的例子
> 从这个例子可以看到，大于180度的方位角，计算结果并不是和官网里面描述的一样，这时候的计算结果理应360+（-134）.

```
with test_table as (select 
st_geomfromtext('POINT(120.142 30.265)',4326) as pointA,
st_geomfromtext('POINT(120.134652299788 30.2586212374289)',4326) as pointB
)
select degrees(ST_Azimuth(pointa::geography ,pointb::geography )) from test_table
-----------
    -134.999999998456
(1 row)
```
![image]({{book.service}}/images/Measurement/ST_Azimuth3.png)

### 四、总结
> 下图使用qgis软件对AB两点的方位角计算结果。
![image]({{book.service}}/images/Measurement/ST_Azimuth4.png)
> 从函数使用结果来看ST_Azimuth(geometry ,geometry)的结果为48.81，ST_Azimuth(geography, geography)结果44.762。第二种计算方式更接近qgis软件的量测结果。
###### 思路拓展
- ST_Azimuth函数可以角度量测
- ST_Azimuth函数如果参与查询，则可以实现朝某个角度进行定向查询。比如查询A点60到120度之间范围所有对象。
- 位置相同的两个点，返回结果为null。这个特性可以用来判断2个点是否重合。
- 官网的说法和函数本身的结果也是有出入的。