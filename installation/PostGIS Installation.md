### 1.4.2 PostGIS安装
本教程安装PostGIS 3.0,3.0依赖的库有geos,proj,gdal,libxml,json-c,protobuf，如何要支持三维需安装sfcgal，如果要做路网分析需安装pgrouting。
#### 1.4.2.1 安装依赖
##### 1.4.2.1.1 geos
```
[root@ ~]# wget https://download.osgeo.org/geos/geos-3.8.0.tar.bz2
[root@ ~]# tar -jxvf geos-3.8.0.tar.bz2     
[root@ ~]# cd geos-3.8.0   
#指定目录安装
[root@ geos-3.8.0]# ./configure --prefix=/usr/local/geos-3.8.0
[root@ geos-3.8.0]# make -j 4
[root@ geos-3.8.0]# make install   
```
##### 1.4.2.1.2 proj
```
[root@ ~]# wget http://download.osgeo.org/proj/proj-6.2.1.tar.gz    
[root@ ~]# tar -zxvf proj-6.2.1.tar.gz    
[root@ ~]# cd proj-6.2.1  
#指定目录安装
[root@ proj-6.2.1]# ./configure  --prefix=/usr/local/proj-6.2.1
[root@ proj-6.2.1]# make -j 4    
[root@ proj-6.2.1]# make install    
```
##### 1.4.2.1.3 gdal
```
[root@ ~]# wget https://download.osgeo.org/gdal/3.0.2/gdal-3.0.2.tar.gz
[root@ ~]# tar -zxvf gdal-3.0.2.tar.gz 
[root@ ~]# cd gdal-3.0.2    
#编译时间比较久，指定目录安装，且绑定已安装的pg
[root@  gdal-3.0.2]# ./configure  --prefix=/usr/local/gdal-3.0.2 --with-pg=/home/postgres/bin/pg_config    
[root@  gdal-3.0.2]# make -j 4
[root@  gdal-3.0.2]# make install   
```
##### 1.4.2.1.4 jsonc,libxml
```
[root@ ~]# wget https://github.com/json-c/json-c/archive/json-c-0.13.1-20180305.tar.gz
[root@ ~]# tar -zxvf json-c-0.13.1-20180305.tar.gz    
[root@ ~]# cd json-c-0.13.1-20180305    
[root@ json-c-0.13.1-20180305]# ./configure  --prefix=/usr/local/json-c-0.13.1
[root@ json-c-0.13.1-20180305]# make -j 4
[root@ json-c-0.13.1-20180305]# make install   

[root@ ~]# wget https://github.com/GNOME/libxml2/archive/v2.9.7.tar.gz
[root@ ~]# tar -zxvf libxml2-sources-2.9.7.tar.gz    
[root@ ~]# cd libxml2-2.9.7   
[root@ libxml2-2.9.7]# ./configure  --prefix=/usr/local/libxml2-2.9.7
[root@ libxml2-2.9.7]# make -j 4
[root@ libxml2-2.9.7]# make install   

```
##### 1.4.2.1.5 protobuf,protobuf-c
```
[root@ ~]#  wget https://github.com/protocolbuffers/protobuf/archive/v3.10.1.tar.gz
[root@ ~]# tar -zxvf protobuf-3.10.1.tar.gz    
[root@ ~]# cd protobuf-3.10.1  
[root@ protobuf-3.10.1]# ./configure  --prefix=/usr/local/protobuf-3.10.1
[root@ protobuf-3.10.1]# make -j 4    
[root@ protobuf-3.10.1]# make install   
#配置环境变量，增加下protobuf-3.10.1/bin
[root@ ~]# vi /etc/profile
export PROTOBUF_HOME=/usr/local/protobuf-3.10.1
export PATH=$GCC_HOME/bin:$PROTOBUF_HOME/bin:$PATH
#保存退出
[root@ ~]# source /etc/profile
#验证protobuf执行程序
[root@ ~]# protoc --version
libprotoc 3.10.1
#protobuf安装成功

[root@ ~]# wget https://github.com/protobuf-c/protobuf-c/releases/download/v1.3.2/protobuf-c-1.3.2.tar.gz
[root@ ~]# tar -zxvf protobuf-c-1.3.2.tar.gz    
[root@ ~]# cd protobuf-c-1.3.2  
#导入protobuf的pkgconfig，否则"--No package 'protobuf' found"
[root@ protobuf-c-1.3.2]# export PKG_CONFIG_PATH=/usr/local/protobuf-3.10.1/lib/pkgconfig
[root@ protobuf-c-1.3.2]# ./configure  --prefix=/usr/local/protobuf-c-1.3.2
[root@ protobuf-c-1.3.2]# make -j 4
[root@ protobuf-c-1.3.2]# make install   

#配置环境变量，增加下protobuf-c-1.3.2/bin
[root@ ~]# vi /etc/profile
export PROTOBUF_HOME=/usr/local/protobuf-3.10.1
export PROTOBUFC_HOME=/usr/local/protobuf-c-1.3.2
export PATH=$GCC_HOME/bin:$PROTOBUF_HOME/bin:$PROTOBUFC_HOME/bin:$PATH
#保存退出
[root@ ~]# source /etc/profile
```

##### 1.4.2.1.6 sfcgal (三维场景，可选择安装)
sfcgal需要cmkae编译，需先安装下cmake：
```
[root@ ~]# wget https://github.com/Kitware/CMake/releases/download/v3.16.2/cmake-3.16.2.tar.gz
[root@ ~]# tar -zxvf cmake-3.16.2.tar.gz
[root@ ~]# cd cmake-3.16.2
[root@ cmake-3.16.2]# ./configure --prefix=/usr/local/cmake-3.16.2
[root@ cmake-3.16.2]# make -j 4
[root@ cmake-3.16.2]# make install

#配置环境变量
[root@ ~]# vi /etc/profile
export CMAKE_HOME=cmake-3.16.2
export PATH=$GCC_HOME/bin:$CMAKE_HOME/bin:$PROTOBUF_HOME/bin:$PROTOBUFC_HOME/bin:$PATH
#保存退出
[root@ ~]# source /etc/profile

```
sfcgal依赖boost,cgal，需要提前编译，编译默认目录，避免编译sfcgal时各种找不到库的问题。
```
[root@ ~]# yum install boost-devel
[root@ ~]# wget https://github.com/CGAL/cgal/archive/releases/CGAL-4.13.tar.gz
[root@ ~]# tar -zxvf CGAL-4.13.tar.gz   
[root@ ~]# cd CGAL-4.13  
[root@ CGAL-4.13]# mkdir build && cd build 
#cmake不要指定安装路径
[root@ build]# cmake ..
[root@ build]# make    
[root@ build]# make install   
```

编译安装sfcgal：
```
[root@ ~]# wget https://github.com/Oslandia/SFCGAL/archive/v1.3.7.tar.gz
[root@ ~]# tar -zxvf v1.3.7.tar.gz   
[root@ ~]# cd SFCGAL-1.3.7  
[root@ SFCGAL-1.3.7]# mkdir build & cd build 
[root@ build]# cmake -DCMAKE_INSTALL_PREFIX=/usr/local/sfcgal-1.3.7 ..
[root@ build]# make -j 4    
[root@ build]# make install   
```
pgrouting可以单独安装，在之后章目里会单独介绍。

#### 1.4.2.2 PostGIS安装
##### 1.4.2.2.1 配置ld.so.conf
```
[root@ ~]# vim /etc/ld.so.conf
#编辑内容如下
include ld.so.conf.d/*.conf
/home/postgres/lib
/usr/local/proj-6.2.1/lib
/usr/local/gdal-3.0.2/lib
/usr/local/geos-3.8.0/lib
/usr/local//sfcgal-1.3.7/lib64
/usr/local/json-c-0.13.1/lib
/usr/local/libxml2-2.9.7/lib
/usr/local/protobuf-3.10.1/lib
/usr/local/protobuf-c-1.3.2/lib
#编辑完成后wq!保存退出
#保存配置，重启生效
[root@ ~]# ldconfig -v 
```
##### 1.4.2.2.2 安装postgis
```
[root@ ~]# wget http://download.osgeo.org/postgis/source/postgis-3.0.0.tar.gz    
[root@ ~]# tar -zxvf postgis-3.0.0.tar.gz    
[root@ ~]# cd postgis-3.0.0 
#根据安装不同的要求，选择任意一个configure
#基本安装，不带sfcgal
[root@ postgis-3.0.0]# ./configure --prefix=/home/postgres --with-gdalconfig=/usr/local/gdal-3.0.2/bin/gdal-config --with-pgconfig=/home/postgres/bin/pg_config --with-geosconfig=/usr/local/geos-3.8.0/bin/geos-config --with-projdir=/usr/local/proj-6.2.1 --with-xml2config=/usr/local/libxml2-2.9.7/bin/xml2-config --with-jsondir=/usr/local/json-c-0.13.1 --with-protobufdir=/usr/local/protobuf-c-1.3.2
# 带protobuf,sfcgal安装
[root@ postgis-3.0.0]# ./configure --prefix=/home/postgres --with-gdalconfig=/usr/local/gdal-3.0.2/bin/gdal-config --with-pgconfig=/home/postgres/bin/pg_config --with-geosconfig=/usr/local/geos-3.8.0/bin/geos-config --with-projdir=/usr/local/proj-6.2.1 --with-xml2config=/usr/local/libxml2-2.9.7/bin/xml2-config --with-jsondir=/usr/local/json-c-0.13.1 --with-protobufdir=/usr/local/protobuf-c-1.3.2 --with-sfcgal=/usr/local/sfcgal-1.3.7/bin/sfcgal-config

[root@ postgis-3.0.0]# make -j 4
[root@ postgis-3.0.0]# make install
```
可能报错什么can not found lsqlite3等错误：
```
yum instlal sqlite-devel
```

##### 1.4.2.2.3 验证安装
```
[root@ ~]# su - postgres
[postgres@ ~]$ psql
psql (12.1)
Type "help" for help.

postgres=# create database mytest;
CREATE DATABASE
postgres=# \c mytest
You are now connected to database "mytest" as user "postgres".
#验证postgis扩展
mytest=# create extension postgis;
CREATE EXTENSION
#验证栅格类数据需要的raster扩展
mytest=# create extension postgis_raster;
CREATE EXTENSION
#如果安装带有sfcgal，验证下三维sfcgal扩展
mytest=# create extension postgis_sfcgal;
CREATE EXTENSION
```