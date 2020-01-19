## ST_3DDistance
### 一、 方法功能描述
以投影单位（空间参考单位）返回两个几何之间的3维最小笛卡尔距离。

### 二、用法

```
float ST_3DDistance(geometry g1, geometry g2);
--ST_3DDistance不支持geography对象
```

### 三、应用示例
> 该方法得使用示例，打算全部用官网得示例，因为官网得使用示例写个足够详细。
###### 1 -几何示例-以米为单位的单位（SRID：2163美国国家地图集相等面积）（3D点和线与2D点和线相比）
> 因为ST_3DDistance不支持geography数据类型，所以需要格外注意坐标系的问题，因为在不同的坐标系下计算结果差别是很大的。见下面两条sql,第一条sql是官网示例。第二条为大家增加影响弄了错误坐标系进行计算。

```
--在2163下进行计算
SELECT ST_3DDistance(
			ST_Transform('SRID=4326;POINT(-72.1235 42.3521 4)'::geometry,2163),
			ST_Transform('SRID=4326;LINESTRING(-72.1260 42.45 15, -72.123 42.1546 20)'::geometry,2163)
		) As dist_3d,
		ST_Distance(
			ST_Transform('SRID=4326;POINT(-72.1235 42.3521)'::geometry,2163),
			ST_Transform('SRID=4326;LINESTRING(-72.1260 42.45, -72.123 42.1546)'::geometry,2163)
		) As dist_2d;

     dist_3d      |     dist_2d
------------------+-----------------
 127.295059324629 | 126.66425605671
 
 
 --3857下进行计算，这条sql是在错误的坐标系下进行计算，请注意。
 SELECT ST_3DDistance(
			ST_Transform('SRID=4326;POINT(-72.1235 42.3521 4)'::geometry,3857),
			ST_Transform('SRID=4326;LINESTRING(-72.1260 42.45 15, -72.123 42.1546 20)'::geometry,3857)
		) As dist_3d,
		ST_Distance(
			ST_Transform('SRID=4326;POINT(-72.1235 42.3521)'::geometry,3857),
			ST_Transform('SRID=4326;LINESTRING(-72.1260 42.45, -72.123 42.1546)'::geometry,3857)
		) As dist_2d;
    dist_3d      |     dist_2d
------------------+-----------------
 167.919316304679 | 167.441410065196	
```
###### 2 计算两个平行的平面间的距离
> 在这个示例中是没有坐标系的，因此其实ST_3DDistance可以当成纯几何计算的函数。从该示例里面看，ST_Distance在这样的计算中是没有效果的，Z值是不参与计算的。

```
SELECT ST_3DDistance(poly, mline) As dist3d,
    ST_Distance(poly, mline) As dist2d
        FROM (SELECT  'POLYGON((175 155 50, 20 40 50, 35 45 50, 50 60 50, 100 100 50, 175 155 50))'::geometry as poly,
               'POLYGON((175 150 10, 20 40 10, 35 45 10, 50 60 10, 100 100 10, 175 150 10))'::geometry as mline) as foo;
   dist3d | dist2d 
------------------- + -------- 
 40 | 0
```




### 总结
> 通过以上示例，大家其实能知道ST_3DDistance是比较容易理解的，唯独容易出错的地方就在坐标系上，这点上官网也没有明说，但是各位看官可以参照ST_Distance，在该函数的示例中坐标系使用和ST_3DDistance是一致的，在此就不过多赘述。

