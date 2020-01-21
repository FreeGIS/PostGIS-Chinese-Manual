## <span id='ST_LineInterpolatePoint'>ST_LineInterpolatePoint</span>
### 方法功能描述
已知单义线以及0-1之间任意一个百分比，返回线上一个点的位置，例如：百分比为0代表线的起点，0.5代表线的中点，1代表线的终点。
### 函数定义
```
/*
 *@method ST_LineInterpolatePoint
 *@param{geometry} a_linestring 一条LineString类型的单义线，不支持MultiLineString。
 *@param{float8} a_fraction 0-1之间的小数百分比。
 *@return {geometry} Point类型，线性插值点的位置。
*/
geometry ST_LineInterpolatePoint(geometry a_linestring, float8 a_fraction);
```
### 应用示例
![示意图]({{book.service}}/images/LinearReferencing/ST_LineInterpolatePoint.jpg)

已知线LINESTRING(118 32,119 33)，分别设置fraction=0,0.5,1，返回该线的起点，中点，终点。
* 返回线的中点位置：

```
--中点对应的百分比是0.5，起点对应0，终点对应1
SELECT ST_AsText(ST_LineInterpolatePoint(ST_GeomFromText('LINESTRING(118 32,119 33)'), 0.50));
 
    st_astext
-------------------
 POINT(118.5 32.5)
```
* 返回线的起点位置：

```
SELECT ST_AsText(ST_LineInterpolatePoint(ST_GeomFromText('LINESTRING(118 32,119 33)'), 0));

   st_astext
-------------------
 POINT(118 32)
```
* 返回线的终点位置：

```
SELECT ST_AsText(ST_LineInterpolatePoint(ST_GeomFromText('LINESTRING(118 32,119 33)'), 1));
   st_astext
-------------------
 POINT(119 33)
```


### ST_LineInterpolatePointd的多维应用
对于多维线，通常分为LineStringZ,LineStringM,LineStringZM三种类型，分别代表经纬度+高程，经纬度+测量值，经纬度+高程+测量值，举例如下：

* LineStringZ：

```
SELECT ST_AsText(ST_LineInterpolatePoint(ST_GeomFromText('LINESTRINGZ(1 1 6, 3 1 3, 3 3 8)'), 0.5));
 st_astext
-------------------
 POINT Z(3 1 3)
```

* LineStringM：

```
SELECT ST_AsText(ST_LineInterpolatePoint(ST_GeomFromText('LINESTRINGM(1 1 6, 3 1 3, 3 3 8)'), 0.5));
    st_astext
------------------
 POINT M(3 1 3)
```

* LineStringZM：

```
SELECT ST_AsText(ST_LineInterpolatePoint(ST_GeomFromText('LINESTRINGZM(1 1 6 8, 3 1 3 6, 3 3 8 7)'), 0.5));
    st_astext
-------------------
 POINT ZM(3 1 3 6)
```

多维原理：ST_LineInterpolatePoint先对多维数据降维只含有xy的LineString，基于这个计算出线上某个点位置，然后根据该位置再线性计算出对应该点的z与m值。

总结：

* ST_LineInterpolatePoint本质只是处理二维数据的，先基于二维算位置，基于位置再算附带的z与m，对xyz而言，其实是“伪三维计算”。
* ST_LineInterpolatePoint并不能处理真正的xyz高程数据。
* 对xyz高程数据应当使用[ST_3dLineInterpolatePoint](#ST_3dLineInterpolatePoint)。


## <span id='ST_3dLineInterpolatePoint'>ST_3dLineInterpolatePoint</span>
该函数使用与ST_LineInterpolatePoint一模一样，但是支持真三维高程数据，下面举几个案例说明：
* 高程z计算：
![二维点与三维点位置示意]({{book.service}}/images/LinearReferencing/ST_3dLineInterpolatePoint.png)

```
SELECT ST_AsText(ST_LineInterpolatePoint(geom, 0.5)) "2d",ST_AsText(ST_3dLineInterpolatePoint(geom, 0.5)) "3d" from 
	(select ST_GeomFromText('LINESTRINGZ(1 1 6, 3 1 3, 3 3 8)') as geom) as foo;
       2d         |                          3d
------------------------------------------------------------------
 POINT Z (3 1 3)  |  POINT Z (3 1.33046593658801 3.82616484147003)
```
**说明：计算结果不同于ST_LineInterpolatePoint，ST_3dLineInterpolatePoint计算结果已经考虑了三维z值对位置的影响，属于“真三维计算”。**



* 测量值m计算：

```
SELECT ST_AsText(ST_LineInterpolatePoint(geom, 0.5)) "2d",ST_AsText(ST_3dLineInterpolatePoint(geom, 0.5)) "3d" from 
	(select ST_GeomFromText('LINESTRINGM(1 1 6, 3 1 3, 3 3 8)') as geom) as foo;
       2d         |        3d
------------------------------------------
 POINT M (3 1 3)  |   POINT M (3 1 3)
```

说明：线型为LINESTRINGM，没有申明z值，此为特殊情况，可以理解成线上的点的z值都是同一个默认值z（比如0），那么在默认值z的维度下（降维打击？？？），这个线其实是个标准的平面二维数据，所以ST_3dLineInterpolatePoint与ST_LineInterpolatePoint结果一致。

例如，当z值相同，可以看成在该维度下是二维的，位置点经纬度结果与二维是一致：

```
SELECT ST_AsText(ST_LineInterpolatePoint(geom, 0.5)) "2d",ST_AsText(ST_3dLineInterpolatePoint(geom, 0.5)) "3d" from 
	(select ST_GeomFromText('LINESTRINGZ(1 1 6, 3 1 6, 3 3 6)') as geom) as foo;
       2d         |        3d
-----------------------------------------
 POINT Z (3 1 6)  |   POINT Z (3 1 6)
```


3. 高程z与测量值m计算：

```
SELECT ST_AsText(ST_3dLineInterpolatePoint(ST_GeomFromText('LINESTRINGZM(1 1 6 5, 3 1 3 4, 3 3 8 9)'), 0.5));
                          st_astext
----------------------------------------------------------------
 POINT ZM (3 1.33046593658801 3.82616484147003 4.82616484147003)
```
说明：ST_3dLineInterpolatePoint函数根据经纬度和高程z计算出三维点位置，再根据点位置反向插值计算出m值。

总结：

* [ST_LineInterpolatePoint](#ST_LineInterpolatePoint)位置点由经纬度参与计算，高程z与测量m值不参与计算。位置点确定后，反向插值计算z，m，仅适合二维。
* ST_3dLineInterpolatePoint位置点有经纬度和高程z参与计算，测量值m不参与计算。位置点确定后，反向插值计算m，仅适合三维。
* 测量值m都不参与位置点计算，而是通过已经计算出的位置点，反向插值推导出该点对应的m值。





## <span id='ST_LineInterpolatePoints'>ST_LineInterpolatePoints</span>
### 方法功能描述
根据百分比返回单义线上一到多个的任意点的位置。
### 函数定义
```
/*
 *@method ST_LineInterpolatePoints
 *@param{geometry} a_linestring 一条LineString类型的单义线，不支持MultiLineString。
 *@param{float8} a_fraction 0-1之间的小数百分比。
 *@param{boolean} repeat false时，等同于ST_LineInterpolatePoint，true时，意味着每隔fraction处便获取一个点位置。
 *@return {geometry} 只有一个点时，返回Point类型，当有多个点时，返回MULTIPOINT类型。
*/
geometry ST_LineInterpolatePoints(geometry a_linestring, float8 a_fraction,boolean repeat);
```


### 应用示例

设置fraction=0.5，即每0.5位置获取一个点，能得到0.5,1.0两个百分比处的位置：
![ST_LineInterpolatePoints]({{book.service}}/images/LinearReferencing/ST_LineInterpolatePoints.png)

```
SELECT ST_AsText(ST_LineInterpolatePoint(geom,0.5)) ST_LineInterpolatePoint,
	   ST_AsText(ST_LineInterpolatePoints(geom,0.5,false)) ST_LineInterpolatePoints_false, 
	   ST_AsText(ST_LineInterpolatePoints(geom,0.5,true)) ST_LineInterpolatePoints_true from 
	(select ST_GeomFromText('LINESTRING(1 1, 3 1, 3 3)') as geom) as foo;
    ST_LineInterpolatePoint  |  ST_LineInterpolatePoints_false  |  ST_LineInterpolatePoints_true
---------------------------------------------------------------------------------------------------
        POINT(3 1)           |          POINT(3 1)              |       MULTIPOINT(3 1,3 3)
```
说明：当repeat=false时，与ST_LineInterpolatePoint一致。

设置fraction=0.6，仅能得到一个位置：

```
SELECT ST_AsText(ST_LineInterpolatePoint(geom,0.6)) ST_LineInterpolatePoint,
	   ST_AsText(ST_LineInterpolatePoints(geom,0.6,false)) ST_LineInterpolatePoints_false, 
	   ST_AsText(ST_LineInterpolatePoints(geom,0.6,true)) ST_LineInterpolatePoints_true from 
	(select ST_GeomFromText('LINESTRING(1 1, 3 1, 3 3)') as geom) as foo;
    ST_LineInterpolatePoint  |  ST_LineInterpolatePoints_false  |  ST_LineInterpolatePoints_true
---------------------------------------------------------------------------------------------------
        POINT(3 1.4)         |          POINT(3 1.4)            |       POINT(3 1.4)
```



### 总结
* ST_LineInterpolatePoint只能处理二维数据，带Z值的三维计算结果是“伪三维”。
* ST_3dLineInterpolatePoint能正确处理带z值的三维，是“真三维”。
* 当只含有m值时，或者ST_3dLineInterpolatePoint处理的线节点的Z值都相等，那么ST_LineInterpolatePoint与ST_3dLineInterpolatePoint其实是一样的，ST_LineInterpolatePoint可以看成是ST_3dLineInterpolatePoint在某同一维度的特殊形式。
* ST_LineInterpolatePoints本质是ST_LineInterpolatePoint的复数形式，当repeat=false时，与ST_LineInterpolatePoint一样。