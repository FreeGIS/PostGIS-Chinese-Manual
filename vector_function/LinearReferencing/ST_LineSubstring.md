## ST_LineSubstring
### 方法功能描述
根据输入的0-1之间的两个分数，截取两个分数之间的一个子线，使用方式很像字符串截取的函数。

### 函数定义
```
geometry ST_LineSubstring(geometry a_linestring, float8 startfraction, float8 endfraction);
```
a_linestring:输入线，必须是LineString及其扩展LineStringM，LineStringZ，LineStringZM类型。

startfraction：0-1之间的一个百分比数。

endfraction：0-1之间的一个百分比数。

当startfraction=endfraction，ST_LineSubstring结果等同于ST_LineInterpolatePoint，仅仅返回一个点。


### 应用示例
1. startfraction<endfraction

![]({{book.service}}/images/LinearReferencing/ST_LineSubstring1.png)
```
SELECT ST_AsText(ST_LineSubString(ST_GeomFromText('LINESTRING(118 32,119 33)'), 0.333, 0.666));
            st_astext
----------------------------------------
LINESTRING(118.333 32.333,118.666 32.666)
```
2. startfraction=endfraction

![]({{book.service}}/images/LinearReferencing/ST_LineSubstring2.png)
```
SELECT ST_AsText(ST_LineSubString(ST_GeomFromText('LINESTRING(118 32,119 33)'), 0.333, 0.333));
    st_astext
----------------------
POINT(118.333 32.333)
```
当startfraction=endfraction时，ST_LineSubString结果等同于ST_LineInterpolatePoint函数的要实现的功能。
3. startfraction>endfraction
```
SELECT ST_AsText(ST_LineSubString(ST_GeomFromText('LINESTRING(118 32,119 33)'),0.666, 0.333));
ERROR:  错误:  2nd arg must be smaller then 3rd arg
```
**结论：startfraction一定要小于等于endfraction**。

4. 连续的MULTILINESTRINGs

**说明：ST_LineSubString仅仅支持LineString及其衍生类型，不支持MULTILINESTRING及其衍生类型。**

所以有种特殊情况下，即MULTILINESTRING中的子线都是连续的，那么他们可以合并成一个新的LineString结果。

![]({{book.service}}/images/LinearReferencing/ST_LineSubstring3.png)
```

SELECT ST_AsText(ST_LineSubString(ST_LineMerge(ST_GeomFromText('MULTILINESTRING((118 32,118.5 32.5),(118.5 32.5,119 33))')),0.333, 0.666));
    st_astext
----------------------------------------------------
LINESTRING(118.333 32.333,118.5 32.5,118.666 32.666)

```
由于(118 32,118.5 32.5),(118.5 32.5,119 33)两个子线是连续的，当ST_LineMerge时，会形成一个新的单线LineString(118 32,118.5 32.5,119 33)，以上函数实际是对该LineString做的分段线提取。