## <span id='ST_LineInterpolatePoints'>ST_LineInterpolatePoints</span>
### 方法功能描述
[ST_LineInterpolatePoint](./ST_LineInterpolatePoint.html)的复数(repeat)形式。
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


### API示例

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
**说明：当repeat=false时，与ST_LineInterpolatePoint一致。**

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
* ST_LineInterpolatePoints本质是ST_LineInterpolatePoint的复数形式。
* 当repeat=false时，ST_LineInterpolatePoints与ST_LineInterpolatePoint结果一样。