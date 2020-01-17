## ST_Scale
### 方法功能描述
根据缩放因子对图形进行缩放，返回一个缩放后的新图形。

### 函数定义

```
--xyz三维缩放
geometry ST_Scale(geometry geomA, float XFactor, float YFactor, float ZFactor);
--xy二维缩放
geometry ST_Scale(geometry geomA, float XFactor, float YFactor);
--xy,xyz,xym,xyzm都可以用这个重载函数实现
geometry ST_Scale(geometry geom, geometry factor);
geometry ST_Scale(geometry geom, geometry factor, geometry origin);
```
说明：ST_Scale支持xy轴，xyz轴，xym轴，xyzm轴等任意2-4维的缩放，缩放函数是基于默认原点0 0的，有些场景需要图形原地缩放，那么就需要使用ST_Scale(geometry geom, geometry factor, geometry origin)重载函数，origin设置为原图形的质心即可。
### 应用示例

二维缩放：

```
select ST_AsEWKT(st_scale(ST_GeomFromEWKT('LINESTRING(0 0, 4 4)'),0.5,0.5));
    st_asewkt
--------------------
LINESTRING(0 0,2 2)
```

三维z缩放：

```
select ST_AsText(st_scale(ST_GeomFromEWKT('LINESTRINGZ(0 0 0, 4 4 4)'),0.5,0.5,0.5));
    st_astext
--------------------------
LINESTRING Z (0 0 0,2 2 2)
```


三维m缩放：

使用xy轴缩放函数：

```
select ST_AsText(st_scale(ST_GeomFromEWKT('LINESTRINGM(0 0 0, 4 4 4)'),0.5,0.5));
    st_astext
--------------------------
LINESTRING M (0 0 0,2 2 4)
```

使用xyz轴缩放函数：

```
select ST_AsText(st_scale(ST_GeomFromEWKT('LINESTRINGM(0 0 0, 4 4 4)'),0.5,0.5,0.5));
    st_astext
--------------------------
LINESTRING M (0 0 0,2 2 4)
```
xy和xyz缩放函数，对m值没任何影响，对m值缩放应使用ST_Scale(geometry geom, geometry factor)重载函数，如下：

```
select ST_AsText(st_scale(ST_GeomFromEWKT('LINESTRINGM(0 0 0, 4 4 4)'),ST_MakePointM(0.5,0.5,0.5)));
    st_astext
--------------------------
LINESTRING M (0 0 0,2 2 2)
```
第二个参数geometry factor其实是个二维或者三维或者四维点，坐标其实都是缩放因子，比如我想对xym轴都缩放0.5，就定义factor=ST_MakePointM(0.5,0.5,0.5)，如果我想定义xyz都缩放0.5，就定义factor=ST_MakePoint(0.5,0.5,0.5)，如下对xyz缩放0.5所示：
ST_MakePoint函数有几个重载函数，如ST_MakePoint(x,y),ST_MakePoint(x,y,z),ST_MakePoint(x,y,z,m)，单独构造xym是ST_MakePointM(x,y,m)。

```
select ST_AsText(st_scale(ST_GeomFromEWKT('LINESTRINGZ(0 0 0, 4 4 4)'),ST_MakePoint(0.5,0.5,0.5)));
    st_astext
--------------------------
LINESTRING Z (0 0 0,2 2 2)
```

四维xyzm：

```
select ST_AsText(st_scale(ST_GeomFromEWKT('LINESTRINGZM(0 0 0 0, 4 4 4 4)'),ST_MakePoint(0.5,0.5,0.5,0.5)));
    st_astext
--------------------------
LINESTRING ZM (0 0 0 0,2 2 2 2)
```

结论：ST_Scale(geometry geom, geometry factor)比较通用，ST_Scale(geometry geomA, float XFactor, float YFactor, float ZFactor)与ST_Scale(geometry geomA, float XFactor, float YFactor)只是ST_Scale(geometry geom, geometry factor)的特殊形式。


指定原点缩放：
![](../../images/AffineTransformations/ST_Affine4.png)

```
--对输入线原地缩放，指定缩放原点为原图形LINESTRING(118 32, 119 33)的几何质心118.5,32.5。
select st_astext(geom) geom1,st_astext(ST_Scale(geom, st_makepoint(0.5,0.5), st_makepoint(118.5,32.5))) geom2 
from (SELECT ST_GeomFromEWKT('LINESTRING(118 32, 119 33)') As geom) as foo;
    geom1                  |            geom2
-------------------------------------------------------------------
LINESTRING(118 32, 119 33) | LINESTRING(118.25 32.25,118.75 32.75)
```

ST_Scale仍然是ST_Affine的一个缩放场景的特例封装和简化。