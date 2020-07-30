# costmap_2d

一:摘要

该软件包提供了一个2D成本图的实现，该图可从世界中获取传感器数据，构建数据的2D或3D占用网格（取决于是否使用基于体素的实现），并根据2D成本图来扩大成本。占用网格和用户指定的充气半径。该程序包还支持基于map_server的成本图初始化，基于滚动窗口的成本图以及基于参数的传感器主题订阅和配置。

所述`costmap_2d`包提供了一个可配置的结构，其保持关于其中机器人应的占用网格的形式导航信息。Costmap使用传感器数据和静态地图中的信息通过`costmap_2d :: Costmap2DROS`对象存储和更新有关世界上障碍物的信息。该`costmap_2d :: Costmap2DROS`对象提供了一个纯粹的二维界面，它的用户，这意味着大约障碍查询只能在列中进行。例如，桌子和鞋子在XY平面上的相同位置，但Z位置不同，则将在`costmap_2d :: Costmap2DROS`中生成相应的单元格具有相同成本值的对象的成本图。旨在帮助规划平面空间。

从Hydro版本开始，用于将数据写入成本图的基础方法是完全可配置的。功能的每一点都存在于一个层中。例如，静态地图是一层，障碍物是另一层。默认情况下，障碍层在三个维度上维护信息（请参见[voxel_grid](http://wiki.ros.org/voxel_grid)）。维护3D障碍物数据可使图层更智能地处理标记和清除。

主界面是`costmap_2d :: Costmap2DROS`，它维护了许多与ROS相关的功能。它包含一个`costmap_2d :: LayeredCostmap`，用于跟踪每个图层。使用[pluginlib](http://wiki.ros.org/pluginlib)在`Costmap2DROS中`实例化每个图层，并将其添加到`LayeredCostmap中`。图层本身可以单独编译，从而可以通过C ++接口对成本图进行任意更改。所述`costmap_2d :: Costmap2D`类实现用于存储和访问所述二维costmap基本的数据结构。



二:标记和清除



代价图会自动通过ROS订阅传感器主题并相应地进行更新。每个传感器用于标记（将障碍物信息插入到成本图中），清除（从障碍物图中去除障碍物信息）或两者兼而有之。标记操作只是改变阵列成本的数组索引。但是，清除操作包括对于每次报告的观察从传感器的原点向外穿过网格进行射线跟踪。如果使用三维结构来存储障碍物信息，则将每列中的障碍物信息放到成本图中后，会向下投影成二维.



三:占用,自由和未知的空间



虽然成本图中的每个像元可以具有255个不同成本值之一（请参见[通胀](http://wiki.ros.org/costmap_2d#Inflation)部分），但其使用的基础结构只能表示三个。具体来说，此结构中的每个单元可以是空闲，已占用或未知的。每个状态在投影到成本图中后都会分配一个特殊的成本值。拥有一定数量占用单元格的列（请参见[mark_threshold](http://wiki.ros.org/costmap_2d/hydro/obstacles#VoxelCostmapPlugin)参数）被分配为`costmap_2d :: LETHAL_OBSTACLE`成本，具有一定数量未知单元格的列（请参见[unknown_threshold](http://wiki.ros.org/costmap_2d#Map_type_parameters)参数）被分配为`costmap_2d :: NO_INFORMATION`成本，而其他`列为`分配了`costmap_2d :: FREE_SPACE`费用。



四: 地图更新

costmap以[update_frequency](http://wiki.ros.org/costmap_2d#Rate_parameters)参数指定的速率执行地图更新周期。每个周期都会进入传感器数据，在成本图的基础占用结构中执行标记和清除操作，并将此结构投影到成本图中，在其中[如上所述](http://wiki.ros.org/costmap_2d#Occupied.2C_Free.2C_and_Unknown_Space)分配了适当的成本值。此后，`将以costmap_2d :: LETHAL_OBSTACLE`成本对每个像元执行障碍物膨胀。这包括将成本值从每个占用的单元向外传播到用户指定的充气半径。[下面](http://wiki.ros.org/costmap_2d#Inflation)概述了这一膨胀过程的细节。



五: TF

为了将来自传感器源的数据插入到成本图中，`costmap_2d :: Costmap2DROS`对象大量使用了[tf](http://wiki.ros.org/tf)。具体来说，假设由[global_frame](http://wiki.ros.org/costmap_2d#Coordinate_frame_and_tf_parameters)参数，[robot_base_frame](http://wiki.ros.org/costmap_2d#Coordinate_frame_and_tf_parameters)参数指定的坐标系和传感器源之间的所有变换均已连接且是最新的。所述[transform_tolerance](http://wiki.ros.org/costmap_2d#Coordinate_frame_and_tf_parameters)参数设置允许这些变换之间的延时的最大数量。如果[tf](http://wiki.ros.org/tf)树未按此预期速率更新，则[导航堆栈将](http://wiki.ros.org/navigation)停止机器人。



六: 膨胀层



通货膨胀是从占用的单元中传播成本值的过程，该值随距离而减小。为此，我们为成本图值定义了5个与机器人相关的特定符号。

- “致命”成本意味着单元中存在实际的（工作空间）障碍。因此，如果机器人的中心在该单元中，则机器人显然会发生碰撞。
- “内切”成本是指一个单元距实际障碍物小于机器人的内切半径。因此，如果机器人中心位于一个单元中的成本等于或高于所指示的成本，则机器人肯定会遇到一些障碍。
- “可能限制”的成本与“限制”的成本相似，但是使用机器人的限制半径作为截止距离。因此，如果机器人中心位于该值处或高于该值的单元中，则取决于机器人的方向是否与障碍物碰撞。我们使用“可能”一词是因为它并不是真正的障碍物，而是某些用户的偏好，将特定的成本值放入地图中。例如，如果用户要表达机器人应尝试避开建筑物的特定区域，则他们可以独立于任何障碍将自己的成本插入该区域的成本图中。请注意，尽管在上图中以128为例，[代码](https://github.com/ros-planning/navigation/blob/jade-devel/costmap_2d/include/costmap_2d/inflation_layer.h#L113)。
- 假定“自由空间”成本为零，这意味着没有什么可以阻止机器人继续前进。
- “未知”成本意味着没有有关给定单元的信息。费用映射表的用户可以按照自己认为合适的方式进行解释。
- 根据所有其他成本与“致命”单元的距离以及用户提供的衰减函数，在“自由空间”和“可能外接”之间分配一个值。

这些定义背后的理由是，我们让计划制定者实施该计划来关心或不关心确切的占用空间，但是为他们提供足够的信息，使其仅在定向确实重要的情况下才产生占用占用空间的成本。



七: 地图类型



初始化`costmap_2d :: Costmap2DROS`对象的主要方法有两种。第一种方法是使用用户生成的静态地图为其添加种子（有关构建地图的文档，请参见[map_server](http://wiki.ros.org/map_server)软件包）。在这种情况下，初始化成本图以匹配静态图提供的宽度，高度和障碍物信息。此配置通常与诸如[amcl之](http://wiki.ros.org/amcl)类的定位系统结合使用，该定位系统使机器人可以在地图框架中注册障碍物，并在其穿越环境时从传感器数据更新其成本图。

初始化`costmap_2d :: Costmap2DROS`对象的第二种方法是为其设置宽度和高度，并将[rolling_window](http://wiki.ros.org/costmap_2d#Map_management_parameters)参数设置为true。当机器人在整个世界范围内移动时，[rolling_window](http://wiki.ros.org/costmap_2d#Map_management_parameters)参数将其保持在[成本图](http://wiki.ros.org/costmap_2d#Map_management_parameters)的中心，而当机器人距离给定区域太远时，它会从地图上删除障碍物信息。这种类型的配置最常用于里程表坐标系中，在该坐标系中，机器人仅关心局部区域内的障碍物。



八: 组件API



所述`costmap_2d :: Costmap2DROS`对象是[包装](http://wiki.ros.org/navigation/ROS_Wrappers)为`costmap_2d :: Costmap2D`暴露其作为一个功能对象[C ++包装ROS](http://wiki.ros.org/navigation/ROS_Wrappers)。它在初始化时指定的ROS名称空间（假定为此处的*名称*）内运行。

创建指定*my_costmap*命名空间的`costmap_2d :: Costmap2DROS`对象的*示例*：



[切换行号](http://wiki.ros.org/costmap_2d#)

```
   1个 ＃包括<tf / transform_listener.h>
   2 ＃包括<costmap_2d / costmap_2d_ros.h>
   3 
   4 ... 
   5 
   6 tf :: TransformListener  tf（ros :: 持续时间（10））; 
   7 costmap_2d :: Costmap2DROS  costmap（“ my_costmap ”，tf）;
```

如果你`rosrun`或`roslaunch`的`costmap_2d`节点直接它将在运行`costmap`命名空间。在这种情况下，下面所有对*name的*引用都应替换为*costmap*。



更常见的情况是通过启动`move_base`节点来运行完整的导航堆栈。这将创建2个成本图，每个成本图都有自己的名称空间：*local_costmap*和*global_costmap*。您可能需要两次设置一些参数，每个成本图一次。



#### Subscribed Topics

`~<name>/footprint` ([geometry_msgs/Polygon](http://docs.ros.org/api/geometry_msgs/html/msg/Polygon.html))

- Specification for the footprint of the robot. This replaces the previous parameter specification of the footprint.

#### Published Topics

`~<name>/costmap` ([nav_msgs/OccupancyGrid](http://docs.ros.org/api/nav_msgs/html/msg/OccupancyGrid.html))

- The values in the costmap

`~<name>/costmap_updates` ([map_msgs/OccupancyGridUpdate](http://docs.ros.org/api/map_msgs/html/msg/OccupancyGridUpdate.html))

- The value of the updated area of the costmap

`~<name>/voxel_grid` ([costmap_2d/VoxelGrid](http://docs.ros.org/api/costmap_2d/html/msg/VoxelGrid.html))

- Optionally advertised when the underlying occupancy grid uses voxels and the user requests the voxel grid be published.

参数

默认参数

Hydro和更高版本使用了所有`costmap_2d`图层的插件。如果不提供`plugins`参数，则初始化代码将假定您的配置为Hyddro之前的版本，并将加载具有默认名称空间的默认插件集。您的参数将自动移动到新的名称空间。默认命名空间是`static_layer`，`obstacle_layer`和`inflation_layer`。一些教程（和书籍）仍引用Hyddro之前的参数，因此请密切注意。为了安全起见，请确保提供一个`plugins`参数。

##### 外挂程式



`〜<名称> /插件`（`序列`，默认：Hydro之前的行为）

- 插件规范的顺序，每层一个。每个规范都是带有名称和类型字段的字典。该名称用于定义插件的参数名称空间。有关示例，请参见[教程](http://wiki.ros.org/costmap_2d/Tutorials/Configuring Layered Costmaps)。





##### 坐标框架和tf参数



`〜<名称> / global_frame`（`字符串`，默认值：`“ / map”`）

- 运行成本图的全局框架。

`〜<名称> / robot_base_frame`（`字符串`，默认值：`“ base_link”`）

- 机械手基本链接的框架名称。

`〜<名称> / transform_tolerance`（`double`，默认值：0.2）

- 指定可忍受的转换（tf）数据延迟，以秒为单位。此参数可作为丢失tf树中的链接的保护措施，同时仍允许用户满意的系统中存在一定的延迟时间。例如，可以容忍过时0.2秒的转换，但不能容忍过时8秒的转换。如果由`global_frame`和`robot_base_frame`参数指定的坐标帧之间的[tf](http://wiki.ros.org/tf)转换比`ros :: Time :: now（）`早`transform_tolerance`秒，则[导航堆栈](http://wiki.ros.org/navigation)将停止机器人。





##### 速率参数



`〜<名称> / update_frequency`（`double`，默认值：5.0）

- 要更新的地图的频率（以Hz为单位）。

`〜<名称> / publish_frequency`（`double`，默认值：0.0）

- 要发布的地图显示信息的频率（以Hz为单位）。





##### 地图管理参数



`〜<名称> / rolling_window`（`bool`，默认：`false`）

- 是否使用成本图的滚动窗口版本。如果将`static_map`参数设置为true，则必须将此参数设置为false。

`〜<名称> / always_send_full_costmap`（`bool`，默认：`false`）

- 如果为true，则每次更新都会将完整的成本图发布到“〜<名称> / costmap”。如果为false，则仅在“〜<名称> / costmap_updates”主题上发布成本图的已更改部分。



以下参数可以被某些图层（即静态地图图层）覆盖。

- `〜<名称> / width`（`int`，默认值：10）

  - 地图的宽度（以米为单位）。

  `〜<名称> /高度`（`int`，默认值：10）

  - 地图的高度（以米为单位）。

  `〜<名称> / resolution`（`double`，默认值：0.05）

  - 地图的分辨率，以米/格为单位。

  `〜<名称> / origin_x`（`double`，默认值：0.0）

  - 全局框架中地图的x原点（以米为单位）。

  `〜<名称> / origin_y`（`double`，默认值：0.0）

- - 全局框架中地图的y原点（以米为单位）。

层规格

1. 静态地图层

2. 障碍物地图层

3. 膨胀层

4. 其他层