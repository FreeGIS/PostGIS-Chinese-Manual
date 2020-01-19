## DropGeometryTable
### 方法功能描述
从数据库中删除一个带geometry类型的空间表，附带删除geometry相关的引用，如索引，约束等。
### 函数定义
1 假设表在默认的public下，可不设置shema_name：

```
boolean DropGeometryTable(varchar table_name);
```

入参定义：
   
```
    table_name: 指定需要删除的表名称
    column_name： 明确指定所删除的geometry类型列的名称
```

2 假设表不在默认的public下，需明确指定shema_name：

```
boolean DropGeometryTable(varchar schema_name, varchar table_name);
```
3 假设表不在默认的catalog下，需明确指定catalog_name：

```
boolean DropGeometryTable(varchar catalog_name, varchar schema_name, varchar table_name);
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

删除该表：

```
#PostgreSQL原生语法
drop table mytest cascade;

# PostGIS函数
SELECT DropGeometryTable('mytest');
```
倘若表建立在其他schema下，如test，则：
```
#PostgreSQL原生语法
drop table test.mytest cascade;

# PostGIS函数
SELECT DropGeometryTable('test','mytest');
```
总结：该函数用于从数据库中删除一个空间表。其实熟悉PostgreSQL中新增列等sql语法的，建议直接使用原生sql去管理表，该函数其实并不是必需的。