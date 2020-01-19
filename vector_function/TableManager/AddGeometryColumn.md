## AddGeometryColumn
### 方法功能描述
为一个数据库中已存在的关系表新建一个geometry类型的字段，用于存储图形，返回值是一个text文本，描述创建列的结果。
### 函数定义
1 假设表在默认的public下，可不设置shema_name：

```
text AddGeometryColumn(varchar table_name, varchar column_name, integer srid, varchar type, integer dimension, boolean
use_typmod=true);
```

参数定义：
 
    table_name: 指定需要修改的表名称
    column_name： 指定新建geometry类型列的名称
    srid：坐标系epsg编号
    type：图形类型，Point或者LineString等Postgis图形类型
    dimension：几何图形的维度，通常是2维
    use_typmod：默认true，不循序旧约束，false是要遵循旧约束

    


2 假设表不在默认的public下，需明确指定shema_name：

```
text AddGeometryColumn(varchar schema_name, varchar table_name, varchar column_name, integer srid, varchar type, integer
dimension, boolean use_typmod=true);
```

3 假设表不在默认的catalog下，需明确指定catalog_name：

```
text AddGeometryColumn(varchar catalog_name, varchar schema_name, varchar table_name, varchar column_name, integer
srid, varchar type, integer dimension, boolean use_typmod=true);
```
### 应用示例
在public schema下新建测试关系表：

```
create table mytest(
    gid serial primary key,  --自增主键
    name text,   --名称
    lon numeric, --经度
    lat numeric  --纬度
);
```
插入测试数据：

```
insert into mytest(name,lon,lat) values ('a',118,32);
insert into mytest(name,lon,lat) values ('b',119,33);
```
测试表是拥有经纬度字段的关系表，通常可以对这样的表转换成空间表，为此，需要给该表新建一个geometry类型的字段。
新建geom字段：

```
#PostgreSQL原生语法
alter table mytest add column geom geometry(Point,4326);

# PostGIS函数
SELECT AddGeometryColumn ('mytest','geom',4326,'Point',2, false);

# 更新数据
update mytest set geom=ST_SetSRID(ST_MakePoint(lon,lat),4326);

# 查询空间表
select gid,ST_AsText(geom) from mytest;
 gid |   st_astext   
----+---------------
  1 | POINT(118 32)
  2 | POINT(119 33)
(2 row)
```
倘若表建立在其他schema下，如test，则：

```
#PostgreSQL原生语法
alter table test.mytest add column geom geometry(Point,4326);

# PostGIS函数
SELECT AddGeometryColumn ('test','mytest','geom',4326,'Point',2, false);
```
总结：该函数用于给表新增一个图形列。其实熟悉PostgreSQL中新增列等sql语法的，建议直接使用原生sql去管理表，该函数其实并不是必需的。