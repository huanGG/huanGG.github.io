---
title: python 地理数据处理
tags: 'python, gis'
translate_title: python-geodata-processing
date: 2019-06-17 10:31:48
---

关于地理数据处理的工具有很多，目前常用的客户端软件有 MapInfo、QGis 等，从业人员常用这些工具来查看和制图。数据库最常用的是 postgres/postgis，postgres 的强大性能和 postgis 对空间索引和计算的良好支持使其成为空间数据库的主流。另外 echarts、three.js 等前端开源库，也具有丰富的地理数据可视化能力。

不过这都不是我们今天的主角，今天我们要介绍的主角是 GDAL/OGR。GDAL(Geospatial Data Abstraction Library)是一个开源栅格空间数据转换库，利用抽象数据模型来表达所支持的各种地理数据文件格式，同时还有一系列的插件和命令行工具来进行数据转换和处理。OGR 是 GDAL 项目的一个分支，功能与GDAL类似，它提供对矢量数据的支持。今天我们主要讨论如何用 python 和 GDAL/OGR 来处理地理格式数据。

<!--more-->

# 一、OGR 矢量结构简介
地理数据分为矢量数据和栅格数据。由点、线、面等构成的地理数据称为矢量数据。OpenGIS 定义了一组基于数据的服务，而数据的基础是要素 <font color='red'> Feature </font>。所谓要素，简单地说就是一个独立的对象，在地图中可能表现为一个多边形建筑物，在数据库中即一个独立的条目。要素具有两个必要的组成部分，几何信息和属性信息。

OpenGIS 将几何信息分为点、边缘、面和几何集合四种：其中我们熟悉的线 Linestring 属于边缘的一个子类，而多边形 Polygon 是面的一个子类。也就是说 OpenGIS 定义的几何类型并不仅仅是我们常见的点、线、多边形三种，它提供了更复杂更详细的定义，增强了未来的可扩展性。另外，几何类型的设计中采用了组合模式 Composite ，将几何集合 GeometryCollection 也定义为一种几何类型，类似地，要素集合 FeatureCollection 也是一种要素。属性信息没有做太大的限制，可以在实际应用中结合具体的实现进行设置。

相同的几何类型、属性类型的组合成为要素类型 FeatureType ，要素类型相同的要素可以被存放在一个数据源中。而一个数据源只能拥有一个要素类型。因此，可以用要素类型来描述一组属性相似的要素。在面向对象的模型中，完全可以把要素类型理解为一个类，而要素则是类的实例。

实际理解中，可以将数据源理解为一个数据库，有多个图层，每个图层相当于一张数据表，每个数据表中有一个字段存储几何数据( OGR 1.11以后可以有多个几何数据字段)，其他字段中存储该数据的属性。

## OGR类总览

Drivers 驱动： OGRSFDriver 每个驱动可以打开对应类型的数据，转换为数据源。在打开某一格式的文件前首先要通过 OGRSFDriverRegistrar 注册驱动。当数据源打开时，OGRSFDriverRegistrar 将遍历使用所有的驱动，直到成功返回 OGRDataSource 对象。

Data Source 数据源： OGRDataSource 表示存储矢量数据的文件或数据库对象。OGRDataSource 通常拥有 OGRLayer 列表，可以返回它们的引用。OGRDataSource 是抽象基类，将有每个格式的驱动实现， OGRDataSource 对象通常由对应的 OGRSFDriver 实例化，删除 OGRDataSource 对象仅是关闭其对数据源的访问，而非物理删除数据源。

Layer 图层： OGRLayer 表示一个图层，同类型要素的集合。OGRLayer 中也包含从数据源中读写要素的方法。图层主要用于在数据源中读写数据，通常代表一个文件，或者是空间表。OGRLayer 有顺序和随机读写方法， GetNextFeature 方法可以顺序读取所有要素，可以使用 SetSpatialFilter 等滤波器限定该方法获取的要素范围。

Feature Class Definition 要素类定义: OGRFeatureDefn 类通常存储整个图层的属性的元数据。一个 OGRFeatureDefn 通常用于表示一个图层的要素属性信息。具有相同要素类定义的要素通过引用计数的方式共享同一个要素类定义。要素ID（FID）在图层中是唯一的，用于标识。单独的要素或者还未写入图层的要素可能有空FID，在OGR中FID用长整型表示，而其他模型中并不一定，例如GML中用字符串表示，而Oracle中行ID比长整型要大。要素类中包含几何类型（使用 OGRFeatureDefn::GetGeomType() 获取 OGRwkbGeometryType ），如果是wkbUnknown类型，那么该要素中可以添加任意类型的几何形状。这意味着一个图层中可以有不同的几何形状。

Feature 要素： OGRFeature 类封装一个要素，包括其几何形状和其他属性信息.

Spatial Reference 空间参考：OGRSpatialReference 类封装了投影和水准面。目前已经支持本地坐标系、地理坐标系、投影坐标系、垂直坐标系、地心坐标系、复合坐标系。不同坐标系下的坐标必须转换到同一坐标系下才可以进行矢量分析。

Geometry 几何形状： OGRGeometry 类以及其子类封装了 OpenGIS 模型中矢量数据，提供了一些几何操作，以及转换为WKB、WKT的操作，每个几何形状中包含有空间参考系统（投影）。它的派生类包括了各种形状，有如下类型： OGRPoint , OGRLineString , OGRPolygon , OGRGeometryCollection , OGRMultiPolygon , OGRMultiPoint , OGRMultiLineString等等。


# 二、利用 OGR 处理矢量数据格式文件
如下例子实现了从一个 MapInfo mif/mid 文件,转换为 MapInfo Tab 格式文件的功能。展示了如何注册驱动，打开创建文件，创建图层、创建要素、设置几何形状和属性等功能。
{% codeblock  lang:python %}
from osgeo import ogr
from gis import ogr_helper


# old mif
mif_path = "/home/map/changhuan/test.mif"
driver = ogr.GetDriverByName("MapInfo File")
datasource = driver.Open(mif_path, 0)
lyr = datasource.GetLayer()

# copy to tab
tab_path = "/home/map/changhuan/test.tab"
datasource2 = driver.CreateDataSource(tab_path)
dest_srs = ogr.osr.SpatialReference()
# 设置投影坐标系
# dest_srs.ImportFromEPSG(4326)
new_lyr = datasource2.CreateLayer("layer", srs=dest_srs, geom_type=ogr.wkbPolygon， options=["BOUNDS= -31.5527340,-24.4471500,135.8789060,52.1065050"])
new_lyr.CreateFields(lyr.schema)
new_defn = new_lyr.GetLayerDefn()

for feature in lyr:
    # create a new feature
    new_feat = ogr.Feature(new_defn)

    # create new geometry
    geometry = feature.GetGeometryRef()
    out_ring = ogr_helper.get_outer_ring(geometry)
    new_out_ring = ogr.Geometry(ogr.wkbLinearRing)

    for i in range(ogr_helper.get_point_count(out_ring)):
        new_out_ring.AddPoint(out_ring.GetX(i), out_ring.GetY(i), z=0)
    new_out_ring.CloseRings()
    new_polygon = ogr.Geometry(ogr.wkbPolygon)
    new_polygon.AddGeometry(new_out_ring)

    # Set new geometry
    new_feat.SetGeometry(new_polygon)

    # copy fields value
    for i in range(feature.GetFieldCount()):
        value = feature.GetField(i)
        new_feat.SetField(i, value)
	
    # Add new feature to new Layer
    new_lyr.CreateFeature(new_feat)
{% endcodeblock %} 

其中 [ogr_helper](https://github.com/huanGG/python_tools/blob/master/gis/ogr_helper.py) 是本人封装的关于 ogr 常用操作的库。当然 GDAL/OGR 的能力远不止于此，更多高阶的功能读者可自行探索。
