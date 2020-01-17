## ST_TransScale
### 方法功能描述
对输入图形先根据平移量平移，再根据缩放因子进行缩放，返回一个新图形，仅仅用于二维图形。
### 函数定义

```
geometry ST_TransScale(geometry geomA, float deltaX, float deltaY, float XFactor, float YFactor);
```
参数说明：
    
    geomA:输入图形。
    deltaX：x方向偏移量。
    deltaY：y方向偏移量。
    XFactor：x轴缩放因子。
    YFactor：y轴缩放因子。
### 应用示例

对LINESTRING(0 0, 3 3)先平移1 1，再缩放0.5，0.25：

```
--平移后(0 0, 3 3)变成(1 1,4 4)，xy值分别再缩放放0.5，0.25
SELECT ST_AsEWKT(ST_TransScale(ST_GeomFromEWKT('LINESTRING(0 0, 3 3)'), 1,1,0.5,0.25));
    st_Asewkt
-------------------------
LINESTRING(0.5 0.25,2 1)
```

**ST_TransScale(geomA, deltaX, deltaY, XFactor, YFactor)是ST_Affine(geomA, XFactor, 0, 0, 0, YFactor, 0, 0, 0, 1, deltaX*XFactor,deltaY*YFactor, 0)的简写。**