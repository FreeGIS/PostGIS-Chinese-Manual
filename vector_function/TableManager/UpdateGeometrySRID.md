## UpdateGeometrySRID
### 方法功能描述
该方法将更新表的srid元数据信息，同时更新geometry列中所有图形的srid。
### 函数定义
1 假设表在默认的public下，可不设置shema_name：

```
text UpdateGeometrySRID(varchar table_name, varchar column_name, integer srid);
```

参数定义：
    table_name: 指定需要修改的表名称
    column_name： 指定新建geometry类型列的名称
    srid：坐标系epsg编号

    


2. 假设表不在默认的public下，需明确指定shema_name：

```
text UpdateGeometrySRID(varchar schema_name, varchar table_name, varchar column_name, integer srid);
```
3. 假设表不在默认的catalog下，需明确指定catalog_name：

```
text UpdateGeometrySRID(varchar catalog_name, varchar schema_name, varchar table_name, varchar column_name, integer
srid);
```
### 应用示例
在public schema下新建测试空间表，但是没有定义图形的坐标系：

```
drop table if exists mytest cascade;
create table mytest(
    gid serial primary key,  --自增主键
    name text,   --名称
    geom geometry(Point)
);
```
插入测试数据：

```
insert into mytest(name,geom) values ('a',ST_MakePoint(118,32));
insert into mytest(name,geom) values ('b',ST_MakePoint(119,33));
```
查询该表的坐标系：

```
select  find_srid('public','mytest','geom');
 find_srid 
-----------
      0
(1 row)

#上面语句等价于下面原生sql语句：
select srid from geometry_columns where f_table_schema='public' and f_table_name='mytest' and f_geometry_column='geom';
 srid 
-----------
      0
(1 row) 

```
查询表信息可知，目前该表没有定义坐标系，默认值是0.
查询该表中图形对象的坐标系：

```
select gid,st_srid(geom) from mytest;
 gid |st_srid  
----+---------------
  1 | 0
  2 | 0
(2 row)
```
查询表中图形对象信息可知，目前所有要素的图形对象都没有定义坐标系，默认值是0.


执行UpdateGeometrySRID函数：

```
SELECT UpdateGeometrySRID('mytest','geom',4326);

 updategeometrysrid 
----------------------------------------
public.mytest.geom SRID changed to 4326
(1 row)
```

检查目前的表与表中数据坐标系：

```
select  find_srid('public','mytest','geom');
 find_srid 
-----------
    4326
(1 row)


select gid,st_srid(geom) from mytest;
 gid |st_srid  
----+---------------
  1 | 4326
  2 | 4326
(2 row)

```
结论：使用UpdateGeometrySRID改变了表的坐标系定义，从geometry(Point)类型改变成了geometry(Point,4326)类型，同时，图形列中所有图形对象也都强制生成了对应的坐标系。

### 使用限制说明
UpdateGeometrySRID函数通常情况下用于更新完善空间表和数据的坐标系，但这有几个前提：**空间表中的数据都必须是同一个坐标系，且函数执行者明确知道表中数据所对应的正确的坐标系。**

前文插入的测试数据是(118,32)，(119,33)两个点，很明显我们能看出这是经纬度坐标，对应epsg:4326，所以我们明确指明将表更新为4326坐标系。假设，(119,33)点替换为(13247019.4043996,3895303.96339389)，那么执行该函数量报错。同样，表中坐标不是常见的经纬度和墨卡托，操作者自己也不知道这个数据是什么srid，那么也就没法去更新表的坐标系了。

实际上，该函数本质就是改变了表的字段类型，我们可以用原生的sql语法表示：

```
ALTER TABLE mytest
ALTER COLUMN geom TYPE geometry(Point, 4326)
USING ST_SetSRID(geom,4326);
```
原生语法其实更加灵活，设置可以修改geom列为其他坐标系。

例如，测试数据是经纬度，但是没定义坐标系，目前因需求需要，需要将该表定义成墨卡托坐标系（3857）：

```
ALTER TABLE mytest
ALTER COLUMN geom TYPE geometry(Point, 3857)
USING ST_TransForm(ST_SetSRID(geom,4326),3857);
```

总结：尽量使用原生sql语法，如果你比较熟悉sql的话。UpdateGeometrySRID在大部分情况下其实是可以使用的，但是不如原生的sql功能强大与灵活。