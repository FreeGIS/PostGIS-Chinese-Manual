## ST_Rotate
### 方法功能描述
围绕一个原点，对输入图形进行旋转。

### 函数定义
1. 旋转原点不指定，默认POINT(0 0)：

```
geometry ST_Rotate(geometry geomA, float rotRadians);
```
输入参数：
    
    geomA：输入图形，该图形将被旋转。
    rotRadians：旋转角度，弧度制，例如旋转30°，则该值为pi()*(30.0/180)。旋转基准方向为逆时针。
  
2. 旋转原点为xy声明形式：

```
geometry ST_Rotate(geometry geomA, float rotRadians, float x0, float y0);
```
输入参数：

    x0:旋转原点x坐标。
    y0:旋转原点y坐标。
3. 旋转原点为point形式：

```
geometry ST_Rotate(geometry geomA, float rotRadians, geometry pointOrigin);
```
输入参数：

    pointOrigin:旋转原点，为point对象，如设置旋转原点是POINY(118 32)。
### 应用示例
1. 不指定原点，默认原点是0 0：
![]({{book.service}}/images/AffineTransformations/ST_Affine2.png)

```
select st_astext(geom) geom1,st_astext(ST_Rotate(geom,pi()*30.0/180)) geom2 
from (SELECT ST_GeomFromEWKT('LINESTRING(1 1, 2 2)') As geom) as foo;
 geom1           |           geom2
-------------------------------------------------------------------------------------------------------------------
LINESTRING(1 1,2 2) | LINESTRING(0.366025403784439 1.36602540378444,0.732050807568878 2.73205080756888)
```
2. 指定xy：

![]({{book.service}}/images/AffineTransformations/ST_Affine3.png)

```
select st_astext(geom) geom1,st_astext(ST_Rotate(geom,pi()*30.0/180,118.5,32.5)) geom2 
from (SELECT ST_GeomFromEWKT('LINESTRING(118 32, 119 33)') As geom) as foo;
 geom1           |           geom2
-------------------------------------------------------------------------------------------------------------------
LINESTRING(118 32 10,119 33 20) | LINESTRING(118.316987298108 31.8169872981078,118.683012701892 33.1830127018922)
```
3. 指定point：

```
select st_astext(geom) geom1,st_astext(ST_Rotate(geom,pi()*30.0/180,ST_MakePoint(118.5,32.5))) geom2 
from (SELECT ST_GeomFromEWKT('LINESTRING(118 32, 119 33)') As geom) as foo;
 geom1           |           geom2
-------------------------------------------------------------------------------------------------------------------
LINESTRING(118 32 10,119 33 20) | LINESTRING(118.316987298108 31.8169872981078,118.683012701892 33.1830127018922)
```

说明：以二维举例来说，ST_Rotate其实是ST_Affine(geom, cos(rotRadians), -sin(rotRadians), sin(rotRadians), cos(rotRadians),x-x*cos(rotRadians)+y*sin(rotRadians),y-x*sin(rotRadians)-y*cos(rotRadians))的简写。

ST_Rotate实现的功能，使用ST_Affine都能实现，只是ST_Affine偏底层，使用比ST_Rotate复杂点，但是能实现更多高级的仿射变换。