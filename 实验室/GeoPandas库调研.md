> [官方文档](https://geopandas.org/en/stable/#)
> 作用：帮助处理Python中的**地理空间数据**

## 前置知识

### 坐标参考系
> [Coordinate Reference System](https://docs.qgis.org/3.34/en/docs/gentle_gis_introduction/coordinate_reference_systems.html)

**所有的地理信息数据都必须有一个crs。**

地图投影：试图在一张平面上描绘地球表面，也即将`3D->2D`。crs定义GIS中二维投影地图与地图上真实位置的关系。
![image.png](https://raw.githubusercontent.com/yzh-2002/img-hosting/main/notes/202409081336259.png)

在某些情况下，**crs的坐标和经纬度一致**，此时的crs为`WGS84`，授权代码为`EPSG:4326`



### 几何图形数据
> `GeoPandas`中几何图像数据的处理是依赖`shapely`完成的
> [Geometry参考文档](https://shapely.readthedocs.io/en/stable/geometry.html)

`GeoPandas`扩展了`pandas`使用的数据类型，其核心数据结构为`geopandas.GeoDataFrame`（`pandas.DataFrame`子类），可存储几何数据并执行空间操作。`geopandas.GeoSeries`是`pandas.Series`的子类。

`DataFrame`和`Series`都是`pandas`的概念，不再赘述。其与`GeoDataFrame`以及`GeoSeries`的不同在于：前者存储的是传统数据（数字，布尔值，文本等），后者存储的是**几何图形数据**。

`shapely`中几何图形数据有以下基本类型：
1. 点：`Point`
2. 曲线：`LineString,LinearRing`
3. 曲面：`Polygon`
相应的，点集由`MultiPoint`类实现，曲线集合由`MultiLineString`类实现，曲面集合由`MultiPolygon`类实现。

地理空间数据和几何图形数据的关系：后者的几何形状可以是任意空间中的坐标集合。**前者需要给定`CRS`**，也即坐标参考系来确定坐标与地球上的位置的关系。

## 功能

### 读写文件

> `GeoPandas`中文件读写基于`pyogrio`完成

`GeoPandas`可通过`read_file`读取几乎任何基于矢量的空间数据格式，例如：`ESRI shapefile`，`GeoJSON`...

### 地理数据操作
> [GeoPandas用户指南](https://geopandas.org/en/stable/docs/user_guide.html)

1. `GeoSeries.buffer(distance,resolution=16)`：获取每个几何对象给定距离内的所有点（也即新的几何对象由原先几何对象中每个点给定距离为半径形成的圆形覆盖而成）
2. `GeoSeries.boundary`：返回每个几何图形的边界
3. `GeoSeries.convex_hull`：返回包含所有几何对象的最小凸多边形（也即凸包）
4. ...
5. 多个空间数据集的集合操作
	1. ![image.png](https://raw.githubusercontent.com/yzh-2002/img-hosting/main/notes/202409090825152.png)
6. ...

## 布点 & 划分区域

布点：
1. 自定义数据`points_from_xy()`实现布点：
	1. 需要知道标记点在指定CRS下的坐标（自己收集数据）
	2. 可通过地理编码`geopandas.tools.geocode`获取指定地理名称的地理空间数据
2. 给定区域随机采样一些点：`GeoPadnas.sample_points()`
3. 给定区域的质心（`GeoSeries.centroid`）

除此之外，无法通过指定规则标记一系列点，例如城市，基础设施，交通站点，除非有这些点的坐标。

划分区域：
1. 自定义数据实现区域划分

```python
import geopandas, geodatasets

chicago = geopandas.read_file(geodatasets.get_path("geoda.chicago_commpop"))
print(chicago)

'''
chicago本身的数据便是有多个社区的地理空间数据组成
0           DOUGLAS  ...  MULTIPOLYGON (((-87.60914 41.84469, -87.60915 ...
1           OAKLAND  ...  MULTIPOLYGON (((-87.59215 41.81693, -87.59231 ...
2       FULLER PARK  ...  MULTIPOLYGON (((-87.6288 41.80189, -87.62879 4...
3   GRAND BOULEVARD  ...  MULTIPOLYGON (((-87.60671 41.81681, -87.6067 4...
4           KENWOOD  ...  MULTIPOLYGON (((-87.59215 41.81693, -87.59215 ...
..              ...  ...                                                ...

'''
```

![image.png](https://raw.githubusercontent.com/yzh-2002/img-hosting/main/notes/202409090930591.png)


2. 多个空间数据集的集合操作可划分区域（交集区域，并集区域...）
3. 规则网格区域划分

```python
import geopandas, geodatasets  
import numpy as np  
import matplotlib.pyplot as plt  
from shapely.geometry import Polygon

chicago = geopandas.read_file(geodatasets.get_path("geoda.chicago_commpop"))

# 选定chicago第一个社区作为划分区域
minx, miny, maxx, maxy = chicago.bounds.iloc[0]  
grid_width = 0.001  # 网格边长
grid_height = 0.001  
grid_cells = []  
x_interval = np.arange(minx, maxx, grid_width)  
y_interval = np.arange(miny, maxy, grid_height)  
for x in x_interval:  
    for y in y_interval:  
        grid_cells.append(  
            Polygon([(x, y), (x + grid_width, y), (x + grid_width, y + grid_height), (x, y + grid_height)]))  
grid = geopandas.GeoDataFrame(grid_cells, columns=['geometry'])  
# 划分网格构成的GeoPandas与chicago的第一个社区区域取交集
grid_clipped = geopandas.overlay(grid, geopandas.GeoDataFrame(geometry=[chicago.iloc[0]['geometry']]), how='intersection')
grid_clipped.plot(facecolor='none', edgecolor='k')  
plt.show()

```

![image.png](https://raw.githubusercontent.com/yzh-2002/img-hosting/main/notes/202409090947469.png)
