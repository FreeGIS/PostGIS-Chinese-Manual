> 为什么要写这一小节呢？因为本人在使用postgis过程中，发现postgis迭代的非常快，新的版本往往会增加一些让人心动的功能。但是postgis作为postgresql的插件，ppostgis和pg的版本也有着一定的关系，在postgis版本定下来后，想更改并不是一件轻松的事。
所以在我们使用postgis的时候，一开始就需要定下来使用什么版本的postgis，该版本能否最大限度的支持未来可能的需求。当然如果你在一张白纸上进行系统建设，那没得说，选择最新的版本的postgis就好。

> postgis近些年发展比较快，近5年更替的版本有20几个，最近的一个版本为3.0.0。目前postgis官网提供的帮助文档有postgis3.0, 2.5, 2.4,2.3四个版本，这四个版本最近几年迭代中比较大的版本。而在这四个版本之间的版本迭代，则大多是对postgis的bugger的修复或者性能的优化。下面分别对上述4个版本迭代进行一个简单的概括和总结。

---

### Release 3.0.0
###### 发布日期：2019/10/20
###### 相对于版本2.5有哪些大的新功能和特性
- 栅格插件作为单独的扩展
- ST_AsGeoJSON将row直接生成完整的geojson,能够便利的提供geojosn格式的数据
- ST_TileEnvelope瓦片号转换成经纬度的函数，能够更简单的进行矢量切片
- 对矢量切片功能进行加强（多处，多个函数都进行了优化）
- ST_3DLineInterpolatePoint 线性参考增加三维函数
- 不再支持<postgresql9.5
- 提高了对postgresql12.0的兼容
- 以及其他的优化或改进



### Release 2.5.0
###### 发布日期：2018/09/23
###### 相对于版本2.4有哪些大的新功能和特性
- ST_Buffer单面缓冲（这个功能印象比较深刻，之前为了实现这个功能写了一个繁琐的函数）
- ST_LineInterpolatePoints 线性参考计算，增加输入多个点的功能
- ST_OffsetCurve平移函数支持集合计算
- ST_OrientedEnvelope功能
- ST_Scale的可选缩放参考中心点
- ST_Angle函数的增加，以前只能用ST_Azimuth方位角计算
- 支持了PostgreSQL 12分支，所以如果postgresql12，最好使用2.5+以上版本
- PostGIS_Extensions_Upgrade 可以借助sql函数进行postgis版本升级
- 支持最低版本postgresql9.4
- 以及其他的优化或改进


### Release 2.4.0 
###### 发布时间：2017/09/30
###### 相对于版本2.3有哪些大的新功能和特性
- ST_Reverse对曲线进行支持
- ST_Centroid对geography进行支持
- postgis开始对矢量切片进行支持，（这是本人认为最让人高兴的一个更新，太棒了特别棒的一个功能）。
- Covers support for polygon on polygon, line on line, point on line for geography(这个用英文比较好理解)
- 对postgresql9.2及以下的版本不在支持。
- 以及其他的优化或改进



### Release 2.3.0
###### 发布时间：2016/09/26
###### 相对于版本2.2有哪些大的新功能和特性
- ST_VoronoiPolygons、ST_VoronoiLines对泰森多边形的支持
- ST_MinimumBoundingRadius、ST_MinimumBoundingCircle对外接圆的支持
- ST_GeneratePoints 用面生成点
- ST_ClusterDBSCAN、ST_ClusterKMeans这几个功能太棒了，有很大用处
- ST_MakeLine 支持了对multipoint 生成线的支持。


## 总结
- postgis2.3主要增加一些空间分析函数
- postgis2.4开始支持矢量切片
- postgis2.5增加和增强了一些空间分析函数，对矢量切片进一步优化。
- postgis3.0持续对矢量切片进行优化；栅格插件作为单独扩展；对geojson更友好支持
- 如果需要借助postgis进行矢量切片，则最少使用2.4以上版本。
- 如果postgresql为12的话，postgis3.0目前支持度最好。