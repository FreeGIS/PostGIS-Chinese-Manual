## <span id='ST_3dLineInterpolatePoint'>ST_3dLineInterpolatePoint</span>
### 方法功能描述
已知单义线以及0-1之间任意一个百分比，返回线上一个点的位置，例如：百分比为0代表线的起点，0.5代表线的中点，1代表线的终点。
重点说明：{% em color="#ff0000" %}该函数为真三维计算，输入线是LINESTRINGZ类型。{% endem %}
### 函数定义
```
/*
 *@method ST_3dLineInterpolatePoint
 *@param{geometry} a_linestring 一条LineStringZ类型的单义线，不支持MultiLineStringZ。
 *@param{float8} a_fraction 0-1之间的小数百分比。
 *@return {geometry} Point类型，线性插值点的位置。
*/
geometry ST_3dLineInterpolatePoint(geometry a_linestring, float8 a_fraction);
```


### API示例
该函数使用与ST_LineInterpolatePoint一模一样，但是支持真三维高程数据。

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

**说明：线数据类型为LINESTRINGM，没有申明Z值，此时可以理解成线上所有节点的Z值都是同一个默认值（比如0），
那么可以理解成在高程为Z的二维平面内的线，所以ST_3dLineInterpolatePoint与ST_LineInterpolatePoint在这种情况下其实都是处理二维，所以结果一致。**

仅有测量值M时，其实Z值未声明，可以理解成隐式的Z值相等，可以通过显示声明Z值来验证。

* Z值相等：

```
SELECT ST_AsText(ST_LineInterpolatePoint(geom, 0.5)) "2d",ST_AsText(ST_3dLineInterpolatePoint(geom, 0.5)) "3d" from 
	(select ST_GeomFromText('LINESTRINGZ(1 1 6, 3 1 6, 3 3 6)') as geom) as foo;
       2d         |        3d
-----------------------------------------
 POINT Z (3 1 6)  |   POINT Z (3 1 6)
```


* 高程z与测量值m计算：

```
SELECT ST_AsText(ST_3dLineInterpolatePoint(ST_GeomFromText('LINESTRINGZM(1 1 6 5, 3 1 3 4, 3 3 8 9)'), 0.5));
                          st_astext
----------------------------------------------------------------
 POINT ZM (3 1.33046593658801 3.82616484147003 4.82616484147003)
```
**说明：ST_3dLineInterpolatePoint函数根据经纬度和高程z计算出三维点位置，再根据点位置反向插值计算出m值。**

总结：

* [ST_LineInterpolatePoint](./ST_LineInterpolatePoint.html)位置点由经纬度参与计算，高程z与测量m值不参与计算。位置点确定后，反向插值计算z，m，仅适合二维。
* ST_3dLineInterpolatePoint位置点有经纬度和高程z参与计算，测量值m不参与计算。位置点确定后，反向插值计算m，仅适合三维。
* 测量值m都不参与位置点计算，而是通过已经计算出的位置点，反向插值推导出该点对应的m值。