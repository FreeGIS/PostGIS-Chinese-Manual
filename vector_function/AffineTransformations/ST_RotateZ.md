## ST_RotateZ
### 方法功能描述
对输入图形围绕z轴旋转一定角度（弧度制），输出一个旋转后的新图形。ST_RotateZST_Rotate(geometry geomA, float rotRadians)是完全一样的。
### 函数定义

```
geometry ST_RotateZ(geometry geomA, float rotRadians);
```
参数说明：
    
    geomA:输入图形。
    rotRadians：绕z轴旋转角度，方向为逆时针方向。
### 应用示例
为方便展示，把z轴坐标统一写0，即将xyz三维，降维到xy二维，方便读者理解。（三维人脑不好理解）。

示例：对已知线LINESTRING(0 0 0, 1 1 0)围绕z轴旋转90度。
![]({{book.service}}/images/AffineTransformations/ST_RotateZ.png)

```
SELECT ST_AsEWKT(ST_RotateZ(ST_GeomFromEWKT('LINESTRINGZ(0 0 0, 1 1 0)'), pi()/2));
    st_asewkt
------------------------------
LINESTRINGZ(0 0 0,-1 1 0)
```

**说明:ST_RotateZ(geomA, rotRadians)是SELECT ST_Affine(geomA,
cos(rotRadians), -sin(rotRadians), 0, sin(rotRadians), cos(rotRadians), 0,0, 0, 1, 0, 0, 0)的缩写，是围绕z轴旋转场景下的一种封装和应用简化。**