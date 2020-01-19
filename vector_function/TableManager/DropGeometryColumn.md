## DropGeometryColumn
### 方法功能描述
为一个数据库中已存在的带有geometry类型的空间表，删除它的图形字段。
### 函数定义
1 假设表在默认的public下，可不设置shema_name：

```
text DropGeometryColumn(varchar table_name, varchar column_name);
```

入参定义：
    table_name: 指定需要修改的表名称
    column_name： 明确指定所删除的geometry类型列的名称

返回值为text类型，描述删除列执行结果，返回值示例：

```
'public.mytest.geom effectively removed.'
```

2 假设表不在默认的public下，需明确指定shema_name：

```
text DropGeometryColumn(varchar schema_name, varchar table_name, varchar column_name);
```
3 假设表不在默认的catalog下，需明确指定catalog_name：

```
text DropGeometryColumn(varchar catalog_name, varchar schema_name, varchar table_name, varchar column_name);
```
### 应用示例
在public schema下新建测试空间表：

```
drop table if exists mytest cascade;
create table mytest(
    gid serial primary key,  --自增主键
    name text,   --名称
    geom geometry(Point,4326)
);
```
插入测试数据：

```
insert into mytest(name,geom) values ('a',ST_SetSRID(ST_MakePoint(118,32),4326));
insert into mytest(name,geom) values ('b',ST_SetSRID(ST_MakePoint(119,33),4326));
```
删除geom字段：

```
#PostgreSQL原生语法
alter table mytest drop column geom;

# PostGIS函数
SELECT DropGeometryColumn ('mytest','geom');


# 查询空间表
select * from mytest;
 gid |   name   
----+---------------
  1 | a
  2 | b
(2 row)
```
倘若表建立在其他schema下，如test，则：

```
#PostgreSQL原生语法
alter table test.mytest drop column geom;

# PostGIS函数
SELECT AddGeometryColumn ('test','mytest','geom');
```
总结：该函数用于给表删除一个图形列。其实熟悉PostgreSQL中新增列等sql语法的，建议直接使用原生sql去管理表，该函数其实并不是必需的。