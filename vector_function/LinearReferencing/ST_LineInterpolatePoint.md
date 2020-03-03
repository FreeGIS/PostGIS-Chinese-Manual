## <span id='ST_LineInterpolatePoint'>ST_LineInterpolatePoint</span>
### 方法功能描述
返回沿LineString(Z,M,ZM)上某一插值点位置，{% em color="#ff0000" %}不支持MultiLineString{% endem %}。例如：插值点百分比0代表线的起点，0.5代表线的中点，1代表线的终点。
### 函数定义
```
/*
 *@method ST_LineInterpolatePoint
 *@param{geometry} a_linestring LineString(Z,M,ZM)类型
 *@param{float8} a_fraction 0-1之间的小数百分比
 *@return {geometry} Point类型，线性插值点的位置
*/
geometry ST_LineInterpolatePoint(geometry a_linestring, float8 a_fraction);
```
### API示例
![示意图]({{book.service}}/images/LinearReferencing/ST_LineInterpolatePoint.jpg)

已知线LINESTRING(118 32,119 33)，分别设置fraction=0,0.5,1，返回该线的起点，中点，终点。
* fraction=0.5

```
--中点对应的百分比是0.5，起点对应0，终点对应1
SELECT ST_AsText(ST_LineInterpolatePoint(ST_GeomFromText('LINESTRING(118 32,119 33)'), 0.50));
 
    st_astext
-------------------
 POINT(118.5 32.5)
```
* fraction=0

```
SELECT ST_AsText(ST_LineInterpolatePoint(ST_GeomFromText('LINESTRING(118 32,119 33)'), 0));

   st_astext
-------------------
 POINT(118 32)
```
* fraction=1

```
SELECT ST_AsText(ST_LineInterpolatePoint(ST_GeomFromText('LINESTRING(118 32,119 33)'), 1));
   st_astext
-------------------
 POINT(119 33)
```

ST_LineInterpolatePoint可处理多维线如LineStringZ,LineStringM,LineStringZM三种类型。
需要注意以下几点：

* ST_LineInterpolatePoint本质只是处理二维数据的，先基于二维算位置，基于位置再算附带的z与m，对xyz而言，其实是“伪三维计算”。
* ST_LineInterpolatePoint并不能处理真正的xyz高程数据。
* 对xyz高程数据应当使用[ST_3dLineInterpolatePoint](./ST_3dLineInterpolatePoint.html)。

示例如下：
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

{% em color="#ff0000" %}总结：ST_LineInterpolatePoint位置仅与XY位置相关，与ZM维度无关，不能处理”真三维计算“，如带Z高程值的三维线计算。{% endem %}

### 案例场景
背景：线性参考在道路中应用非常常见，例如高速公路每公里都设置有里程桩，实际每个高速路段都会建立线性参考，也就是任意一个路段的起点都可以描述成：
距离xx入口xx公里。

问题：假设有一段高速线，其geometry值，起点是距离xx入口50公里，该路段长度是10公里，则该路段的终点是距离xx入口（50+10）公里，某次该路段发生交通事故，
事故点描述为xx高速距离xx入口56公里处发生事故，要求返回事故点真实地理坐标，从而能在地图上定位。

解决方案：

```
--(56-50)*1.0/(60-50)是计算事故点在该路段中的百分比
select ST_LineInterpolatePoint(line,(56-50)*1.0/(60-50));
```


