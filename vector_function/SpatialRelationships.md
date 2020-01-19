## 拓扑关系
### <span id="ST_Intersects">ST_Intersects,ST_Disjoint</span>
作用：ST_Intersects判断两个Geometry/Geography是否相交，ST_Disjoint判断两个Geometry是否相离，不支持Geography类型。

重点：
- **仅用于二维数据。**
- **ST_Intersects在PostGIS空间关系中是最重要最常用的函数。**
- **空间关系很少用于判定两个图形具体关系，这没意义其实，而是结合gist索引作为检索表达式从空间表中查询数据使用。**
- **ST_Intersects可能会走空间索引，ST_Disjoint一定不会走空间索引，ST_Disjoint绝对不要用于做空间检索。**

精度：对于图形为Geography类型，容差精度到0.00001米，也就是两个图形距离小于0.00001就会被认为是相交。

ST_Intersects是个粗粒度的空间关系，认定两个图形存在交集，但在不同的应用场景下，有时需要更精确的空间关系，如geomA包含geomB等，因此ST_Intersects可以细分为如下更具体的空间关系：ST_Contains（包含）,ST_Within（被包含）,ST_Equals（完全相等），ST_Crosses（穿越）， ST_Covers（覆盖）,ST_Overlaps（压盖），ST_Touches（相连），这些函数其实都隐含了他们一定是相交的。

##### API使用示例

![](/images/SpatialRelationships/ST_Intersestc1.png)

```
--ST_Intersects geometry
SELECT ST_Intersects(ST_GeomFromText('POINT(0 0)',4326), ST_GeomFromText('LINESTRING (2 0, 0 2)',4326));
st_intersects
---------------
   flase
   
SELECT ST_Intersects(ST_GeomFromText('POINT(1 1)',4326), ST_GeomFromText('LINESTRING (2 0, 0 2)',4326));
st_intersects
---------------
   true
   
--ST_Intersects geography
SELECT ST_Intersects(ST_GeomFromText('POINT(0 0)',4326)::geography, ST_GeomFromText('LINESTRING (2 0, 0 2)',4326)::geography);
  st_intersects
---------------
   flase 
   
--ST_Disjoint geometry
SELECT ST_Disjoint(ST_GeomFromText('POINT(0 0)',4326), ST_GeomFromText('LINESTRING (2 0, 0 2)',4326));
  st_disjoint
---------------
    true   
    
SELECT ST_Disjoint(ST_GeomFromText('POINT(0 0)',4326)::geography, ST_GeomFromText('LINESTRING (2 0, 0 2)',4326)::geography);
ERROR:  错误:  函数 st_disjoint(geography, geography) 不存在
```
**结论：ST_Disjoint不支持geography。**

### <span id="ST_Contains">ST_Contains,ST_Within</span>
**前提：都只支持geometry，不支持geography。**

定义：
- ST_Contains(A,B)：当且仅当B的点不位于A的外部且B的内部的至少一个点位于A的内部时，才返回true。
- ST_Within(A,B)：几何A完全在几何B内。与ST_Contains关系相反，也就是ST_Contains(A,B)=true则意味着ST_Within(B,A)=true。


##### API使用示例

ST_Contains(A,B)：

点包含点：

```
--A包含B
--A=Point(0 2) B=Point(0 2)
select ST_Contains(ST_GeomFromText('Point(0 2)',4326),ST_GeomFromText('Point(0 2)',4326));
  ST_Contains
---------------
    true 
```
**判断依据：B点坐标不位于A点外部， 由于A点范围是个点域，B点落在了A的点域范围内（B点至少有一点在A内部），因此，A点是包含B点的。**

![](/images/SpatialRelationships/ST_Contains2.png)

线包含点：
```
--A线 B点
select ST_Contains(ST_GeomFromText('LineString(0 0,0 5)',4326),ST_GeomFromText('Point(0 2)',4326));
  ST_Contains
---------------
    true 
```
**判断依据：B点坐标不位于A线外部， 由于A线范围是个线域，B点落在了A的线域范围内（B点至少有一点在A内部），因此，A线是包含B点的。**


面包含线：

![](/images/SpatialRelationships/ST_Contains1.png)

```
--A面 B线
select ST_Contains(ST_GeomFromText('Polygon((0 0,0 5,5 5,5 0,0 0))',4326),ST_GeomFromText('LineString(1 1,3 3)',4326));
  ST_Contains
---------------
    true 
```
**判断依据：B线坐标不位于A面外部， 由于A面范围是个面域，B线落在了A的面域范围内（B线至少有一点在A内部），因此，A面是包含B线的。**


说明：在判断依据中，反复强调B不位于A外部，且引入点域线域面域概念，就是为了把A当做一个“**类似面**”的概念，只有这样，B中至少一点在A内部这个定义才比较好理解。

==注意：ST_Contains(A,B)一定会ST_Within(B,A)。==

```
select ST_Contains(A,B),ST_Within(B,A) from
(SELECT ST_GeomFromText('Polygon((0 0,0 5,5 5,5 0,0 0))',4326) As A, ST_GeomFromText('LineString(1 1,3 3)',4326) As B) as test_data;
  ST_Contains  |   ST_Within
--------------------------------------
    true       |     true
```


如果B中不位于A外部，但是不强调B中至少一点在A内部会是什么结果？

![](/images/SpatialRelationships/ST_Contains3.png)

```
select st_contains(st_geomfromtext('Polygon((0 0,0 5,5 5,5 0,0 0))',4326),st_geomfromtext('LineString(0 0,0 5)',4326));
  ST_Contains
---------------
    false 
```
**判断依据：B线坐标不位于A面外部， 但B线所有坐标都在A面的边界上，那么B线没有任何点在A面内部，因此，A面是不包含B线的。**

### <span id="ST_Equals">ST_Equals</span>
**前提：都只支持geometry，不支持geography。**

定义：图形A与图形B“相等”，相等是引号，**意味着两个图形非全等，而只是拓扑相等**，从图形拓扑上看，则ST_Contains(A,B)=true并且ST_Within(A,B)=true，则ST_Equals(A,B)=true。

点的相等：

```
select ST_Contains(A,B),ST_Within(B,A),ST_Equals(A,B) from
(SELECT ST_GeomFromText('Point(0 2)',4326) As A, ST_GeomFromText('Point(0 2)',4326) As B) as test_data;
  ST_Contains  |   ST_Within  | ST_Equals
------------------------------------------
    true       |    true      |   true
```
线的相等：

```
select ST_Contains(A,B),ST_Within(B,A),ST_Equals(A,B) from
(SELECT ST_GeomFromText('LineString(0 0,0 5)',4326) As A, ST_GeomFromText('LineString(0 0,0 3,0 5)',4326) As B) as test_data;
  ST_Contains  |   ST_Within  | ST_Equals
------------------------------------------
    true       |    true      |   true
```
==注意：线A有两个节点，线B有三个节点，如果从节点数量不等来看，其实这两个线肯定是不相等的，但是从图形拓扑看，满足ST_Contains(A,B)=true并且ST_Within(A,B)=true，则ST_Equals(A,B)=true。==



### <span id="ST_ContainsProperly">ST_ContainsProperly</span>
**前提：该函数只支持geometry，不支持geography。**

作用：==ST_Contains的一种特殊情况==，ST_Contains要求B不在A外部且至少有一点在A内部，而ST_ContainsProperly则要求B中所有点都必须在A内部，而不能在A的外部或边界上。

满足ST_Contains而不满足ST_ContainsProperly：
![](/images/SpatialRelationships/ST_ContainsProperly.png)

```
select ST_Contains(A,B),ST_ContainsProperly(A,B) from
(SELECT ST_GeomFromText('Polygon((0 0,0 5,5 5,5 0,0 0))',4326) As A, ST_GeomFromText('LineString(0 0,3 3)',4326) As B) as test_data;
  ST_Contains  |   ST_ContainsProperly
--------------------------------------
    true       |         false
```
满足足ST_Contains又满足ST_ContainsProperly：
![](/images/SpatialRelationships/ST_Contains1.png)

```
select ST_Contains(A,B),ST_ContainsProperly(A,B) from
(SELECT ST_GeomFromText('Polygon((0 0,0 5,5 5,5 0,0 0))',4326) As A, ST_GeomFromText('LineString(1 1,3 3)',4326) As B) as test_data;
  ST_Contains  |   ST_ContainsProperly
--------------------------------------
    true       |         true
```

**结论：ST_ContainsProperly(A,B)当B哪怕有一个点在A的边界上，也会返回false，而全部节点都在A内部才返回true。**

### <span id="ST_Covers">ST_Covers,ST_CoveredBy</span>

**前提：都支持geometry和geography类型。**

作用：ST_Covers(A,B)表示图形覆盖关系，表达图形B没有点在图形A外部。ST_CoveredBy(A,B)与ST_Covers相反，表达图形A没有点在图形B外部。

说明：ST_Covers和ST_Contains在某些情况下回重叠，但是也有差异。

既符合ST_Contains也符合ST_Covers：
![](/images/SpatialRelationships/ST_ContainsProperly.png)

```
select ST_Contains(A,B),ST_Covers(A,B) from
(SELECT ST_GeomFromText('Polygon((0 0,0 5,5 5,5 0,0 0))',4326) As A, ST_GeomFromText('LineString(0 0,3 3)',4326) As B) as test_data;
  ST_Contains  |   ST_Covers
-------------------------------
    true       |     true
```
**总结：满足B中没有点在图形A外部，所以ST_Covers(A,B)=true，同时，又满足B中至少有个点在A内部，所以ST_Contains(A,B)=true。**

不符合ST_Contains但符合ST_Covers：
![](/images/SpatialRelationships/ST_Contains3.png)

```
select ST_Contains(A,B),ST_Covers(A,B) from
(SELECT ST_GeomFromText('Polygon((0 0,0 5,5 5,5 0,0 0))',4326) As A, ST_GeomFromText('LineString(0 0,0 5)',4326) As B) as test_data;
  ST_Contains  |   ST_Covers
-------------------------------
    false       |     true 
```
**总结：满足B中没有点在图形A外部，所以ST_Covers(A,B)=true，但是，不满足B中至少有个点在A内部，所以ST_Contains(A,B)=false。**
##### 补充说明
ST_Covers支持geography：

```
select ST_Covers(ST_GeomFromText('Polygon((0 0,0 5,5 5,5 0,0 0))',4326)::geography,ST_GeomFromText('LineString(0 0,0 5)',4326)::geography);
    ST_Covers
-----------------
     true 
```

ST_Covers与ST_CoveredBy关系其实很像ST_Contains与St_Within之间的关系：

```
select ST_Covers(A,B),ST_CoveredBy(B,A) from
(SELECT ST_GeomFromText('Polygon((0 0,0 5,5 5,5 0,0 0))',4326) As A, ST_GeomFromText('LineString(0 0,0 5)',4326) As B) as test_data;
  ST_Covers  |   ST_CoveredBy
-------------------------------
    true       |     true 
```


### <span id="ST_Overlaps">ST_Overlaps,ST_Crosses</span>

**前提：仅仅支持geometry类型** 

共同点：ST_Overlaps(A,B),ST_Crosses(A,B)都是表达B中有部分图形在A内部，同时又有部分图形在A外部。

不同点：

- ST_Overlaps(A,B)，压盖关系，A，B及其两者的交集都要求同一几何维度，即如果A是面，则B也要是面，而且交集也一定是面。
- ST_Crosses(A,B)，穿越关系，A,B两者的交集的几何维度小于A，B中最大的几何维度。即如果A是面，B是线，交集是线，线的几何维度小于AB中最大的几何维度（最大的几何维度是A，A是面）。



示例1：两个面相交，类型一致，但是B没有点在A的内部：

![](/images/SpatialRelationships/ST_Overlaps1.png)

```
select ST_Overlaps(A,B),ST_Crosses(A,B) from
(SELECT ST_GeomFromText('Polygon((0 0,0 5,5 5,5 0,0 0))',4326) As A, 
  ST_GeomFromText('Polygon((-1 -1,-1 1,0 1,0 -1,-1 -1))',4326) As B) as test_data;
  ST_Overlaps  |  ST_Crosses
-----------------------------
    false      |    false
```
说明：AB两个都是面，交集是条线，交集的几何维度是线，与输入的AB几何维度是面不一致，因此不是压盖关系。另外，B有部分图形在A外部，但是没有部分图形在A的内部（仅在边界），因此，既不满足压盖也不满足穿越关系的定义，所以结果都是false。

示例2：两个面相交，类型一致，但是B既有点在A外部，也有点在A内部：

![](/images/SpatialRelationships/ST_Overlaps2.png)

```
select ST_Overlaps(A,B),ST_Crosses(A,B) from
(SELECT ST_GeomFromText('Polygon((0 0,0 5,5 5,5 0,0 0))',4326) As A, ST_GeomFromText('Polygon((-1 -1,-1 3,3 3,3 -1,-1 -1))',4326) As B) as test_data;
   ST_Overlaps  |  ST_Crosses
-----------------------------
    true      |    false
```
说明：首先，从图形可知，B有部分图形在A的内部，也有部分图形在A的外部，因此满足压盖和穿越关系定义。由于AB都是面，交集也是面，交集的几何维度与输入的AB几何维度一致，因此，这是压盖关系，而不是穿越关系。（穿越关系交集的几何维度小于AB最大的几何维度，即不能是面）

示例3：两个线相交，交集是点：

![](/images/SpatialRelationships/ST_Crosses1.png)

```
select ST_Overlaps(A,B),ST_Crosses(A,B) from
(SELECT ST_GeomFromText('LineString(0 0,3 3)',4326) As A, ST_GeomFromText('LineString(0 3,3 0)',4326) As B) as test_data;
   ST_Overlaps  |  ST_Crosses
-----------------------------
    false      |    true
```
说明：从图形可知，AB线有交集，交集是点，满足B有部分图形在A的内部，也有部分图形在A的外部，因此满足压盖和穿越关系定义。点的几何维度与AB线的几何维度不一致，且点的维度小于AB中最大的几何维度（是线），因此是穿越关系而不是压盖关系。

示例4：两个线相交，交集是线：

![](/images/SpatialRelationships/ST_Crosses2.png)

```
select ST_Overlaps(A,B),ST_Crosses(A,B) from
(SELECT ST_GeomFromText('LineString(0 0,3 3)',4326) As A, ST_GeomFromText('LineString(0 3,1.5 1.5,1.6 1.6,2 4)',4326) As B) as test_data;
   ST_Overlaps  |  ST_Crosses
-----------------------------
    true      |    false
```
说明：AB满足压盖和穿越的图形定义，但是返回维度为线与输入AB一致，所以是压盖关系而不是穿越关系。

应用场景：
现有普通道路表roads，高速公路highways，要求返回被高速公路穿越的道路id。

```
--两个表结构
CREATE TABLE roads (
    gid serial primary key,
    geom geometry(LineString,4326)
);
create index roads_geom_idx on roads using gist(geom);

CREATE TABLE highways (
    gid serial primary key,
    geom geometry(LineString,4326)
);
create index highways_geom_idx on highways using gist(geom);

--求结果：
select a.gid from roads a,highways b where ST_Crosses(a.geom,b.geom);
```

### <span id="ST_Touches">ST_Touches</span>

**前提：仅仅支持geometry类型。** 

定义：ST_Touches(A,B)，相连关系，如果AB之间交集位于AB的边界的并集中，则返回TRUE。

![](/images/SpatialRelationships/ST_Touches1.png)

定义解析：
    1 至少有一个公共点，因此必须相交。
    2 相交部分在AB图形边界的并集中。

示例1：两个面边界相连：
![](/images/SpatialRelationships/ST_Overlaps1.png)

```
select ST_Intersects(A,B),
	ST_Contains(
		ST_Union(ST_Boundary(A),ST_Boundary(B)),
		ST_Intersection(A,B)
	),ST_Touches(A,B) from
(SELECT ST_GeomFromText('Polygon((0 0,0 5,5 5,5 0,0 0))',4326) As A, 
  ST_GeomFromText('Polygon((-1 -1,-1 1,0 1,0 -1,-1 -1))',4326) As B) as test_data;
  
  ST_Intersects | ST_Contains | ST_Touches
-------------------------------------------
    true        |    true     |   true

```
说明：AB相交，A边界ST_Boundary(A)，B边界ST_Boundary(B)，AB边界并集是ST_Union(ST_Boundary(A),ST_Boundary(B))，AB交集是ST_Intersection(A,B)，AB交集在AB边界的并集中，即	ST_Contains(ST_Union(ST_Boundary(A),ST_Boundary(B)),ST_Intersection(A,B))=true，所以，ST_Touches=true。


示例2：两个线，交集为线：
![](/images/SpatialRelationships/ST_Crosses2.png)

```
select ST_Intersects(A,B),
	ST_Contains(
		ST_Union(ST_Boundary(A),ST_Boundary(B)),
		ST_Intersection(A,B)
	),ST_Touches(A,B) from
(SELECT ST_GeomFromText('LineString(0 0,3 3)',4326) As A, ST_GeomFromText('LineString(0 3,1.5 1.5,1.6 1.6,2 4)',4326) As B) as test_data;

  ST_Intersects | ST_Contains | ST_Touches
-------------------------------------------
    true        |    false    |   false
```
说明：AB相交，交集不在AB边界的并集中，因此ST_Touches=false。


示例3：两个线，交集为点情况1：

![](/images/SpatialRelationships/ST_Touches2.png)

```
select ST_Intersects(A,B),
	ST_Contains(
		ST_Union(ST_Boundary(A),ST_Boundary(B)),
		ST_Intersection(A,B)
	),ST_Touches(A,B) from
(SELECT ST_GeomFromText('LineString(0 0,3 3)',4326) As A, ST_GeomFromText('LineString(3 3,2 0)',4326) As B) as test_data;
  
  ST_Intersects | ST_Contains | ST_Touches
-------------------------------------------
    true        |    true     |   true
```

示例4：两个线，交集为点情况2：

![](/images/SpatialRelationships/ST_Crosses1.png)

```
select ST_Intersects(A,B),
	ST_Contains(
		ST_Union(ST_Boundary(A),ST_Boundary(B)),
		ST_Intersection(A,B)
	),ST_Touches(A,B) from
(SELECT ST_GeomFromText('LineString(0 0,3 3)',4326) As A, ST_GeomFromText('LineString(0 3,3 0)',4326) As B) as test_data;
  
  ST_Intersects | ST_Contains | ST_Touches
-------------------------------------------
    true        |    false    |   false
```
### <span id="ST_Relate">ST_Relate</span>

**前提：仅仅支持geometry类型。** 

用法：

```
boolean ST_Relate(geometry geomA, geometry geomB, text intersectionMatrixPattern);
text ST_Relate(geometry geomA, geometry geomB);
text ST_Relate(geometry geomA, geometry geomB, integer BoundaryNodeRule);
```
作用：通过测试两个图形的interior，boundary和exterior之间的交集关系，这个关系是一个DE-9IM模型，比对传入的intersectionMatrixPattern，一致则返回true。
如果不传入intersectionMatrixPattern，返回两个图形的最大的intersectionMatrixPattern字符串。

概念：

1 图形维度

几何图形 | 维度编号
---|---
点 | 0
线 | 1
面 | 2
2 名词解释

名词 |说明
---|---
interior | 图形内部
boundary | 图形边界
exterior | 图形外部

两个压盖面(ST_Overlaps)的9IM示例图：
![](/images/SpatialRelationships/ST_Relate1.png)

上图对应的DE-9IM表：

DE-9IM | interior| boundary | exterior
---|---|---|---
interior | 2 | 1 | 2
boundary | 1 | 0 | 1
exterior | 2 | 1 | 2
解释：见上图，当AB两个面是压盖关系，面的内部有交集，交集是个面，所以interior交interior结果是2（面的维度编号）；A面内部与B面边界交集是线，所以结果是1（线的维度编号）；A面内部与B面外部交集是个面，所以结果是2；所以DE-9IM表第一行是 212，ST_Overlaps完整的intersectionMatrixPattern字符是"212101212"。

![](/images/SpatialRelationships/ST_Overlaps2.png)

```
select ST_Relate(A,B),ST_Overlaps(A,B) from
(SELECT ST_GeomFromText('Polygon((0 0,0 5,5 5,5 0,0 0))',4326) As A, ST_GeomFromText('Polygon((-1 -1,-1 3,3 3,3 -1,-1 -1))',4326) As B) as test_data;

 ST_Relate  |  ST_Overlaps
--------------------------
 212101212  |    true
```
**注意：当交集为空，可以用"F"(false简写)代指；当交集不为空如是0,1,2时，可以用"T"(true简写)代指。**

{% em color="#ff0000" %}问题：为什么要引入ST_Relate及其DE-9IM？{% endem %}

PostGIS已经提供ST_Overlaps,ST_Crosses,ST_Contains等常用空间关系（这些空间关系都可以通过ST_Relate来实现），但是并不能包含全部的图形关系，对于官方定义函数之外的图形关系，都以通过DE-9IM去表达和查询，因此，单独使用ST_Relate主要是用来实现更多复杂的空间关系。




由于线点的外部内部描述比较抽象，有时不好理解，面比较符合人的思维，以下都以面面来说明。


示例1：用ST_Relate实现ST_Intersects一样的查询：

AB两面内部相交：

![](/images/SpatialRelationships/ST_Overlaps2.png)

当AB两面相交是，只要看interiorA与interiorB交集存在，结果是面，维度编码是2，即可满足AB是相交的，其他关系用 * 表示即可，表达“对此两者关系不关心，不做定义”，因此两面内部相交intersectionMattrixPattern='2********'，在做关系判断时有时不需要明确的维度编码，只需要判别是否有交集，没有就是F，有交集的记过可能是0,1,2之一，他们都可以用T表示存在交集，因此，'2********'在面内部相交是也可以写成'T********'。

DE-9IM | interiorB| boundaryB | exteriorB
---|---|---|---
interiorA | 2 | * | *
boundaryA | * | * | *
exteriorA | * | * | *

```
--AB两面内部相交
select ST_Relate(A,B,'T********'),ST_Intersects(A,B) from
(SELECT ST_GeomFromText('Polygon((0 0,0 5,5 5,5 0,0 0))',4326) As A, ST_GeomFromText('Polygon((-1 -1,-1 3,3 3,3 -1,-1 -1))',4326) As B) as test_data;

 ST_Relate  |  ST_Overlaps
--------------------------
  true      |    true

```



AB两面边界相交：

![](/images/SpatialRelationships/ST_Overlaps1.png)

AB两面边界相交，结果是线，维度编码是1，其他关系不关心，即可表达两个图形是边界相交，因此，DE-9IM表如下：

DE-9IM | interiorB| boundaryB | exteriorB
---|---|---|---
interiorA | * | * | *
boundaryA | * | 1 | *
exteriorA | * | * | *

```
--intersectionMattrixPattern为'****1****'或'****T****'
select ST_Relate(A,B,'****T****'),ST_Intersects(A,B) from
(SELECT ST_GeomFromText('Polygon((0 0,0 5,5 5,5 0,0 0))',4326) As A, 
  ST_GeomFromText('Polygon((-1 -1,-1 1,0 1,0 -1,-1 -1))',4326) As B) as test_data;
 
  ST_Relate  |  ST_Overlaps
--------------------------
   true      |    true
```

示例2：查询两个图形之间的intersectionMattrixPattern具体编码：

```
select ST_Relate(A,B),ST_Intersects(A,B) from
(SELECT ST_GeomFromText('Polygon((0 0,0 5,5 5,5 0,0 0))',4326) As A, 
  ST_GeomFromText('Polygon((-1 -1,-1 1,0 1,0 -1,-1 -1))',4326) As B) as test_data;
   ST_Relate  |  ST_Overlaps
-----------------------------
   FF2F11212  |    true
```

{% em color="#ff0000" %}注意：ST_Relate虽然也能实现St_Intersects等函数，但是该函数不会走空间索引，因为ST_Relate里面也会实现ST_Disjoint这样的相离操作，这是不会走索引的。{% endem %}


### <span id="ST_RelateMatch">ST_RelateMatch</span>


 [ST_Relate](#ST_Relate)






### <span id="ST_LineCrossingDirection">ST_LineCrossingDirection</span>

### <span id="ST_OrderingEquals">ST_OrderingEquals</span>


### <span id="ST_PointInsideCircle">ST_PointInsideCircle</span>

### <span id="ST_3DIntersects">ST_3DIntersects</span>
