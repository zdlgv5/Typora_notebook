# ROS机器人恢复机制

### 概述

recovery 机制是当ROS移动机器人认为自己被卡主时,指导机器人进行一系列的恢复行为.

### 恢复机制

![选区_001.png](https://i.loli.net/2019/12/27/EsDjqLnV3ih8GPz.png)

如图所示,为move_base的默认恢复机制,move_base节点可以在机器人认为自己被卡主的时候执行恢复行为的操作.默认执行的顺序为:

- step1: 用户指定区域以外的障碍物从机器人的地图上清除;
- step2: 机器人进行原地旋转以清除空间;
- step3: 若继续失败则机器人将更加激进的清除地图,清除的距离范围更近,任何矩形区域以外的所有障碍均被清除;
- step4: 机器人继续原地旋转清除地图;
- step5: 仍然失效则机器人认为目标不可行,导航终止.

`recovery_behavior`参数配置上述恢复行为; `recovery_behavior_enabled`参数禁用上述恢复行为;

### 源码分析

在navigation包中,有三个包与恢复机制有关,分别为:`clear_costmap_recovery`,`move_slow_and_clear`,`rotate_recovery`.三个包中定义了三个类,均集成了`nav_core`中的接口规范.

![选区_002.png](https://i.loli.net/2019/12/27/BaJUhKRLu8Gdgle.png)

##### nav_core::RecoveryBehavior接口规范

除构造函数外,该虚基类中定义了两个函数,一个负责初始化,另一个负责执行恢复行为.

```
virtual void initialize() //初始化

virtual void runBehavior()//执行恢复行为
```

##### clear_costmap_recovery包

遍历所有层,如果某层在可清理列表里那么就清理掉它的costmap.这里可清理列表中只有障碍物层(obstacleLayer),也就是实时扫描建立的costmap.

###### ClearCostmapRecovery类

```
public:

initialize() //初始化函数,定义一个ros::nodehandle命名空间,读取ros参数,将要清除的layer,插入到std::setstd::string clearable_layers_集合中

runBehavior()//执行清除命令:clear(global_costmap_);clear(local_costmap_).

private:

clear(costmap_2d::Costmap2DROS costmap)//函数,查找std::setstd::string clearable_layers_集合中是否有要清除的图层,然后运行clearMap(costmap,x,y);

clearMap()//将机器人正方形区域之外的所有cost设置为noinfomation(255)
```

##### move_slow_and_clear包

```
public:

initialize() //定义一个ros::nodehandle命名空间,读取ros参数.

runBehavior() //清除机器人周围指定范围内的障碍物层,设置为FREE_SPACE(0).然后调用setRobotSpeed函数,通过动态重置修改配置机器人最大线速度和角速度,并实时调用distanceCheck,移动足够远时,停止监视.
```

##### rotate_recovery包

###### RotateRecovery类

```
initialize() //定义一个ros::nodehandle命名空间,读取ros参数.

runBehavior() //在runBehavior里只需要发指令让小车逆时针旋转180度,有没有可前进的路线是需要在这个过程中,local costmap在转一圈中建立发现.
```

###### 参考链接

[导航recovery机制](https://blog.csdn.net/qq_41986495/article/details/86477409)