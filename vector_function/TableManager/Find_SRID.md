## Find_SRID
### 方法功能描述
返回一个空间表geom列所定义的空间坐标系编号（srid）。
### 函数定义

```
integer Find_SRID(varchar a_schema_name, varchar a_table_name, varchar a_geomfield_name);
```
入参：
	a_schema_name：空间表所在的schema名称。
	a_table_name：空间表（table）名称。
	a_geomfield_name：空间表对应的图形字段名称。

返回值：srid编号，integer类型，如wgs84对应的epsg:4326，则返回值为4326。

### 应用定义
新建测试表：

```
create table mytest(
    gid serial primary key,
    name text,
    geom geometry(Point,4326)  --定义该表为4326坐标系
);
```
查询mytest表的坐标系编号：

```
select Find_SRID('public','mytest','geom');
 find_srid 
-----------
      4326
(1 row)
```
其实该function本质仅仅是个封装，PostGIS中所有的图形列信息都在geometry_columns和geographty_columns两个视图里可以查看。


可以通过原生语句即可查询：

```
select srid from geometry_columns where f_table_schema='public' and f_table_name='mytest' and f_geometry_column='geom';
 srid 
-----------
      4326
(1 row)
```

### find_srid与st_srid的区别
find_srid是查询某个空间表定义的空间坐标系：

```
select Find_SRID('public','mytest','geom');
 find_srid 
-----------
      4326
(1 row)
```
st_srid是查询单个geom对象的空间坐标系:

```
select st_srid(st_geomfromtext('Point(118 32)',4326));
 st_srid 
---------
    4326
```