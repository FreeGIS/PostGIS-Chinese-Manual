## Populate_Geometry_Columns
### 方法功能描述
该方法将用于确保图形列有明确的类型修饰符或适当的空间约束，主要用来规范空间表的结构。
### 函数定义
1 操作针对全部空间表：

```
text Populate_Geometry_Columns(boolean use_typmod=true);
```
入参定义：

use_typmod=true，会强制该表表结构，例如会将字段类型不明确的geometry改成类型明确的geometry(Point,4326)等。

use_typmod=false，不改变表结构，但是会强制添加维度，坐标系，图形类型的约束，规范写入库的数据。
2 操作仅仅针对指定的某个空间表：

```
int Populate_Geometry_Columns(oid relation_oid, boolean use_typmod=true);
```
### 使用前提
1 保证表不是空表，即要有数据；
2 且保证表中的geom字段非空；
3 且保证表中所有图形都是同一个维度，坐标系，和图形类型。

### 应用示例
1 针对全部空间表：

先创建两个schema,每个schema下再建立两个空的图形表：

```
    create schema test1;
    create schema test2;
    
    create table test1.mytable1(
        gid serial primary key,
        name text,
        geom geometry
    );
    create table test2.mytable2(
        gid serial primary key,
        name text,
        geom geometry
    );

```

请注意，目前的两个图形表，都已知字段类型为geometry，但是，并不清楚图形是二维还是三维，图形是点线还是面，图形坐标系究竟是多少。

分别在两个测试表插入测试数据：

```
INSERT INTO test1.mytable1(geom) VALUES(ST_GeomFromText('Point(118 32)',4326) );
INSERT INTO test2.mytable2(geom) VALUES(ST_GeomFromText('Point(118 32)',4326) );
```
查询现有两张表的数据描述：

```
select * from geometry_columns where f_table_name='mytest1' or f_table_name='mytest2';
 f_table_catalog|f_table_schema|f_table_name|f_geometry_name|coord_dimension|srid|type
----------------+--------------+------------+---------------+---------------+----+-----
    test        | test2        | mytable2   | geom          | 2             | 0  |GEOMETRY
    test        | test1        | mytable1   | geom          | 2             | 0  |GEOMETRY
(2 row)
```
结论：目前的srid和type都是默认的，非具体明确定义的。

执行Populate_Geometry_Columns方法：

```
select Populate_Geometry_Columns();
populate_geometry_columns
 -----------------------
  probed:2 inserted:2
```
然后查询现有两张表的数据描述：

```
select * from geometry_columns where f_table_name='mytest1' or f_table_name='mytest2';
 f_table_catalog|f_table_schema|f_table_name|f_geometry_name|coord_dimension|srid|type
----------------+--------------+------------+---------------+---------------+----+-----
    test        | test2        | mytable2   | geom          | 2             |4326|POINT
    test        | test1        | mytable1   | geom          | 2             |4326|POINT
(2 row)
```
结论：通过执行Populate_Geometry_Columns方法，将索引图形类型等不明确的空间表转换成了定义严格的空间表。

实际上，这样的场景并不常见。绝大部分时候我们只会改某个具体的表。

2 针对指定表
建立测试表并插入测试数据：

```
  create table mytable(
        gid serial primary key,
        name text,
        geom geometry
    );
INSERT INTO mytable(geom) VALUES(ST_GeomFromText('Point(118 32)',4326) );
INSERT INTO mytable(geom) VALUES(ST_GeomFromText('Point(118 32)',4326) );
```
添加强约束，不改表结构：

```
select Populate_Geometry_Columns('public.mytable'::regclass,false);
--查看表结构
test=# \d+ mytable
                                                 Table "public.mytable"
 Column |   Type   | Collation | Nullable |               Default                | Storage  | Stats target | Description
--------+----------+-----------+----------+--------------------------------------+----------+--------------+-------------
 gid    | integer  |           | not null | nextval('mytable_gid_seq'::regclass) | plain    |              |
 name   | text     |           |          |                                      | extended |              |
 geom   | geometry |           |          |                                      | main     |              |
Indexes:
    "mytable_pkey" PRIMARY KEY, btree (gid)
Check constraints:
    "enforce_dims_geom" CHECK (st_ndims(geom) = 2)
    "enforce_geotype_geom" CHECK (geometrytype(geom) = 'POINT'::text)
    "enforce_srid_geom" CHECK (st_srid(geom) = 4326)
Access method: heap

```

如果是希望改变表结构：

```
drop table if exists mytable cascade;
create table mytable(
        gid serial primary key,
        name text,
        geom geometry
    );
INSERT INTO mytable(geom) VALUES(ST_GeomFromText('Point(118 32)',4326) );
INSERT INTO mytable(geom) VALUES(ST_GeomFromText('Point(118 32)',4326) );

select Populate_Geometry_Columns('public.mytable'::regclass,true);
test=# \d+ mytable
                                                       Table "public.mytable"
 Column |         Type         | Collation | Nullable |               Default                | Storage  | Stats target | Description
--------+----------------------+-----------+----------+--------------------------------------+----------+--------------+-------------
 gid    | integer              |           | not null | nextval('mytable_gid_seq'::regclass) | plain    |              |
 name   | text                 |           |          |                                      | extended |              |
 geom   | geometry(Point,4326) |           |          |                                      | main     |              |
Indexes:
    "mytable_pkey" PRIMARY KEY, btree (gid)
Access method: heap
```
### 额外扩展
其实Populate_Geometry_Columns也可以通过原生的pg sql语法实现：
改变表类型：

```
alter table mytable alter column geom type geometry(Point,4326) using geom::geometry(Point,4326);
```
添加图形类型约束：

```
alter table mytable add constraint enforce_geotype_geom CHECK (geometrytype(geom) = 'POINT'::text);
```

