## ST_Distance
### 一、 方法功能描述
对于几何类型，以投影单位（空间参考单位）返回两个几何之间的最小二维笛卡尔（平面）距离。

对于地理类型，默认情况下返回两个地理对象之间的最小测地距离（以米为单位），请计算由SRID确定的椭球体。如果use_spheroid为false，则使用更快的球面计算。

### 二、用法

```
float ST_Distance(geometry g1, geometry g2);
float ST_Distance(geography geog1, geography geog2, >boolean use_spheroid=true);
```

### 三、应用示例
> 该方法得使用示例，打算全部用官网得示例，因为官网得使用示例写个足够详细。
###### 1 几何示例-平面度为4326的单位为WGS 84长纬度，单位为度。

```
SELECT ST_Distance（
		'SRID = 4326; POINT（-72.1235 42.3521）':: geometry，
		'SRID = 4326; LINESTRING（-72.1260 42.45，-72.123 42.1546）':: geometry 
	）; 
st_distance 
----------------- 
0.00150567726382282
```
###### 2 几何示例-单位为米（SRID：3857，与流行的网络地图上的像素成比例）
> 尽管该值不可用，但可以正确比较附近的值，这使其成为KNN或KMeans等算法的不错选择。

```
SELECT ST_Distance（
			ST_Transform（'SRID = 4326; POINT（-72.1235 42.3521）':: geometry，3857），
			ST_Transform（'SRID = 4326; LINESTRING（-72.1260 42.45，-72.123 42.1546）':: geometry，3857）
		）; 
st_distance 
----------------- 
167.441410065196
```

###### 3 几何示例-以米为单位（SRID：如上所述为3857，但通过cos（lat）进行了校正以解决变形）

```
SELECT ST_Distance（
			ST_Transform（'SRID = 4326; POINT（-72.1235 42.3521）':: geometry，3857），
			ST_Transform（'SRID = 4326; LINESTRING（-72.1260 42.45，-72.123 42.1546）':: geometry，3857）
		）* cosd（42.3521）; 
st_distance 
----------------- 
123.742351254151
```

###### 4 几何示例-单位为米（SRID：26986）（对马萨诸塞州最为准确）

```
SELECT ST_Distance（
			ST_Transform（'SRID = 4326; POINT（-72.1235 42.3521）':: geometry，26986），
			ST_Transform（'SRID = 4326; LINESTRING（-72.1260 42.45，-72.123 42.1546）':: geometry，26986）
		）; 
st_distance 
----------------- 
123.797937878454
```

###### 5 几何示例-单位为米（SRID：2163美国国家地图集相等区域）（最不准确）

```
SELECT ST_Distance（
			ST_Transform（'SRID = 4326; POINT（-72.1235 42.3521）':: geometry，2163），
			ST_Transform（'SRID = 4326; LINESTRING（-72.1260 42.45，-72.123 42.1546）':: geometry，2163）
		）; 

st_distance 
------------------ 
126.664256056812
```

###### 6 geography 输入计算，但以米为单位进行注释-使用球体可加快计算速度并降低准确度。

```
SELECT ST_Distance(gg1, gg2) As spheroid_dist, ST_Distance(gg1, gg2, false) As sphere_dist
FROM (SELECT
	'SRID=4326;POINT(-72.1235 42.3521)'::geography as gg1,
	'SRID=4326;LINESTRING(-72.1260 42.45, -72.123 42.1546)'::geography as gg2
	) As foo  ;

  spheroid_dist   |   sphere_dist
------------------+------------------
 123.802076746848 | 123.475736916397
```


### 总结
> 通过以上示例，大家其实能知道ST_Distance其实是一个很简单得函数，比较复杂得点在于坐标系上。对于开发人员来说去记坐标系其实是蛮困难的，即使上学的时候学过，估计一段时间也忘到爪哇国去了。所以正确使用该函数的姿势是第6个示例，这种使用方式屏蔽了坐标系的问题，而且计算结果准确。当然也可以使用第3个示例的方法，这个示例会根据你所在的投影分带，进行结果矫正，但是说实话，本人是不知道里面的数学逻辑的。

