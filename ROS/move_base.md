# move_base

### Move_base代码框架

![move_base.h.png](https://i.loli.net/2020/01/09/92ZoVv76Ada4ue5.png)

### Move_base.cpp流程图

![MoveBase P1 Diagram.png](https://i.loli.net/2020/01/10/wz2v5BiVafOENpP.png)

![MoveBase P2 Diagram.png](https://i.loli.net/2020/01/10/zpIa1Y7KcuFxWOt.png)

![MoveBase P3 Diagram.png](https://i.loli.net/2020/01/10/2Q9HmlTMd64nsoV.png)

![MoveBase P4 Diagram.png](https://i.loli.net/2020/01/10/6AFv5QJqBosMZeK.png)

![MoveBase P5 Diagram.png](https://i.loli.net/2020/01/10/ZLrJaxbvhMsdYew.png)

## 流程图逻辑梳理

### part1 初始化部分

从`/move_base_node.cpp`开始看起，main()为入口，init一下move_base_node。

进入到`move_base.cpp`

 MoveBaseActionServer* as_;　初始化这个ＡctionServer，move_base这个节点调用executeCb2；

 初始化move_base的其他参数，其中必要重要的有：

 \- make_plan_srv_ ,做一个服务，make_plan；

 \- clear_costmaps_srv_,做一个服务，clear_costmaps;

Line186执行了as_->start(); 

 然后是动态配置服务开启，也就看到了终端里显示的信息*reconfigureCB,xbot_move_base start!*

### part２ executeCb2

 接着看回as_->start();

 开始执行executeCb2，等待着目标点的进来，move_base_goal；

 有目标点进来　先进行目标点的合法性判断，判断函数是isQuaterionValid，判断目标点的四元数是否合法。

### isQuaternionValid(q)

得到一组四元数，首先判断xyzw中有没有nan或者是inf数据，如果有，直接返回false，并丢弃目标点；

 接下来，我们需要检查四元数的长度是否接近于零;如果小于1e-6，则直接返回false，并丢弃目标点；

 接下来，我们将规范化四元数并检查它是否正确地转换了垂直向量，如果垂直向量转换结果不好，则返回false，这个好与不好是根据fabs(dot-1)>1e-3来判断。

 如果这三个判断都是true，就返回true，是一个合法的目标点。

 －－－－回到主线－－－－

> 如果目标点非法，则as_->setAbort，触发move_base的第一个Abort路径，各类状态取消，发布当前状态后，return;

 目标点合法，则继续再主线进行，主线定义了一个变量，数据类型为PoseStamped，初始化这个变量使用目标点：move_base_goal->target_pose

### goalToGlobalFrame(goal_pose_msg)

这个函数上来定义了两个pose，首先try一下global_frame和goal_pose的tf转换；

结果是成功返回global_pose,失败返回goal_pose_msg.该函数的主要作用是做一个tf转换，获取现在的位姿。

－－－－回到主线－－－－

这边把goal初始化为了global_pose,将hasNewGoal置为了true；

判断costmap是不是打开了，如果没有就start；

获取一下controller_frequency,然后进入循环：

｛

先判断下控制器的频率是不是要改，然后调用一下抢占请求，进入之后再进行判断是否是可用的目标点，判断方法和使用isQuaternionValid函数，如果是不合法的：

> Abort,return;第二条abort路径；

如果合法，继续tf转换，设置hasNewGoal的状态为true;

检查是否转换了global frames，因为我们需要转换我们的goal pose，没转换就在这里转换一下，转换完了就继续执行

判断是不是新的目标hasNewGoal，如果是true：

　　进来先将HasNewGoal的状态给重置为false，然后就是全局规划次数和局部规划次数重置为0,state现在置为PLANNING；发布info我显得接收到了一个目标，并给出坐标，然后将goal给发布出去，planner_goal设置为当前新来的目标点；重置控制、规划、震荡的时间戳，重置规划retry的次数为０；

　　如果不是true：进来就设置wallTime=now.

然后统一进入executeCycle3()

｝

如果主动从上述节点跳出了循环，也会导致abort，这是第三条路径：

> move_base节点被杀掉，导致abort；

退出executeCb2.return;

### part3 executeCycle3()

 

首先加了个递归区域锁，防止出现线程死锁。

更新反馈以适应我们当前的位置，并且随之发布反馈(current_position)

检查costmap是当下的，避免盲目开始运动，如果不是最新的则退出循环，是最新的则进入状态机。

### PLANNING

发布INFO:PLANNING，发布状态NAV_PALNNING;

判断dealPlanning函数的返回结果，进入支线dealPlanning函数

#### dealPlanning(goal,tmpPath)

开始处理全局规划，进来首先将全局规划计数器+1；

然后定义一个变量，来描述全局路径的规划结果

然后进入支线的支线全局规划函数globalPlan

##### globalPlan(goal,tmpPath)

进入函数，定义容忍距离:0.3m

定义找到的结果found_legal，初始化未false，然后将found_legal重新赋值为makePlan(goal,tmpPath)的结果，进入支线的支线的支线(嗨，套娃你好)

###### makePlan(goal,plan)

进入函数，发布info:makePlan,首先将所有的plan给清空，保证目前没有任何plan；

判断costmap是否未空，是空，则发布info makePlan—1，缺少costmap，不能创建出全局规划，返回false；

非空的话肯定就是继续往下走，获取plan开始的pose，也就是global_pose，判断是否获得了robotpose，获取robotpose这里调用了函数**getRobotPose**，不再继续套娃了，因为此函数为官方新增函数，主要逻辑是从给的costmap frame中获得robot pose，会有几个异常的清空可能出现，主要有No Transform可以利用、Connectivity错误、Extrapolation错误，最后检查时间戳,判断当前的时间和global_pose的时间间隔是不是大于costmap tf转换所能忍耐的极限，这个时间忍耐参数是：costmap->getTransformTolerance()，如果是true,就是超过了忍耐极限，这里会报出一个非常常见的warn:

```
Transform timeout for %s. " \"Current time: %.4f, pose stamp: %.4f, tolerance: %.4f
```

然后返回false;

假如都没问题，就成功的获取了global_pose；

回到这里，如果没有成功获取，就爆出错误info makePlan—2，不能获取机器人的初始位姿，无法进行全局规划；

如果成功，则继续判断：

makePlan是否为真，这个makePlan有三个参数start,goal,plan.函数实现再NavfnROS::makePlan()中实现，首先还是先判断是否初始化了，然后清空所有的plan，判断global_frame和start_frame，如果不一致，就会发布info`The goal pose passed to this planner must be in the`并返回false，再判断当前的位置是不是在global　costmap中，如果不在发布info：`Planning will always fail ...`返回false，判断目标是不是再全局代价地图之外，如果也是，则依然发布info:`The goal sent to the navfn planner`返回false,还有一种可能返回false，就是发现了路径，但是没有发现最好的，会发布info:`Failed to get a plan from potential when a legal potential was found.`,但是这种情况应该不会发生，除此之外就是利用全局规划算法找到了路径并返回!plan.empty();

这里失败返回的INFO是makePlan—3,发布出目标点，并返回false；都判断为true，就返回true;

－－回到支线的支线(globalPlan)－－

这时候found_legal应该有了新的值，当为false，就是依靠原有的全局规划逻辑没有找到可用的路径，会进行自己定制的全局规划逻辑，再次进行查找路径，如果成功found_legal会置为true,如果失败，则会发布出INFO`Failed to find a plan to point`,最后会发布INFO`=======exit globalPlan =========`，最后返回found_legal;

－－－回到支线(delPlanning)－－－

这时候就会进入判断：

全局规划失败则进入次数判断，是不是进行了５次以上的全局规划，如果是，则放弃目标点，state_=CLEARING，recovery_trigger_==PLANNING_R;如果没有超过两次，就进行恢复行为：

指针指向dealRecovery(tmp_plan111,1)

恢复结束，进行短暂的sleep，然后状态state_＝PLANNING，继续全局规划；

全局规划成功，则进入局部路径规划，局部规划次数重置为０，mBingoRecovery指向cleanRecovery，state_=CONTROAL

返回success;

－－－－回到主线(状态机)－－－－

处理全局规划如果成功则进行后续操作，不成功就发布0速度。不做任何操作。

处理全局规划成功后，就设置了目标点setPlan,发布INFO:`tc_ setPlanRet = `

继续进行判断，如果没有成功设置目标点，则到达第四条abort的路径

> abort:INFO:"Failed to pass global plan to the controller.

全部走完该状态，然后就break出状态机.继续回到executeCycle3执行状态选择.

### CONTROLLING

进入controlling之后，会发布一个INFO：`In Controlling`，然后进行下一步的判断，判断目标是否到达，使用isGoalReached()函数，进入支线：

#### isGoalReached

该函数使用的是DWA中的函数，代码在dwa_planner_ros.cpp

首先判断是否初始化，未初始化返回false；

其次判断能否获取机器人的位姿，如果不能爆出error:INFO，返回false；

最后判断isGoalReached,源码再base_local_planner/latched_stop_rotate_controller.cpp，主要的判断逻辑就是用目标位置的坐标和现在位置的坐标距离差，小于忍耐值就可以判断为目标已到达，然后确保我们停对了地方之后返回true，否则返回false；

－－－－回到主线－－－－

如果已经到了，就发布信息`Goal reached!`

进入我们唯一的成功路径

> Goal reached!

发布各种的状态，然后return true（executeCycle３）;

假如没有到达目标点，发布状态，NAV_CONTROLLING,进入dealControlling()支线；

#### dealControlling

进入处理局部规划函数，首先照常理，先将规划的次数+1,定义一个速度指令的变量；也定义一个success，用来判断makeLocalPlan是否成功，这里进入支线的支线：

##### makeLocalPlan(cmd_vel)

首先进来也是一个区域递归锁，然后定义一个localPlanResult来表示局部速度指令是否计算出来的结果，也就是表示dwa是否给出了速度，这里进行支线的支线的支线：

###### computeVelocityCommands(cmd_vel)

这里有四个插件可以选择，我们以DWA为例，首先依旧是判断是否能够得到robot pose，得不到，返回false；

再次判断是否得到了局部规划，这里继续深入就是调用getLocalPlan这个函数来判断，函数源码在base_local_planner/local_planner_limits/local_planner_util.cpp中实现。主要逻辑是先获取全局的plan在我们的frame中，这里如果tf出现问题，会报出错误并返回false，然后我们将根据机器人的位置将全局plan进行修剪。再往深处探索就是修剪的具体实现细节。修剪完毕后返回true；

其次判断tf，然后没有tf_plan同样返回false；

全部判断通过后，发布INFO，接收到一个合法的目标点；

更新一下plan

判断是不是到了，如果到了就发布空计划，

如果没有到;这里定义了一个isOk变量，这里是表示真正的dwa计算速度的函数入口;

###### dwaComputeVelocityCommands(current_pose_,cmd_vel)

进入函数，首先还是判断是不是初始化了，没有初始化返回false,初始化了继续进行

然后进入到findBestPath寻找到一个最优的path

如果path.cost<0,报出dwa常见错误INFO:`The dwa local planner failed to find a valid plan`

发布一个空的局部plan，并且返回false；

如果找到了，就填充local plan，最后发布local plan，并且返回true。

至于更深入的findBestPath，就要深入研究dwa的寻路实现了。

这里回到isOk，如果是true就发布出transformed_plan，如果是false，就报出经典错误INFO: `DWA planner failed to produce path`并发布空计划，最终return isOK.

－－返回支线的支线(makeLocalPlan)－－

计算完局部路径时，返回的localPlanResult，会有一个INFO出来`local planner localPlanResult =`

判断，如果是true,则发布速度，如果是false，则发布0速度,最后返回localPlanResult.

－－－回到支线(dealControlling)－－－

局部路径规划是否成功，success现在已经有了值，进入判断

如果失败，则进入10次重复规划的节奏，超过10次局部路径规划失败，则放弃目标点，state_=CLEARING,recovery_trigger==CONTROLLING_R并发布INFO；

如果没有规划10次,则每４次放弃当前全局路径，进入到全局路径规划,state_=PLANNING

如果连续两次局部路径没有给出，则进入恢复行为：mBingoRecovery->dealRecovery(tmp_plan111,0)，state_=CONTROLLING继续规划

如果规划成功，则发布INFO:`localPlan success!`,将计数器重置为０.

处理结束，返回success

－－－－回到主线－－－－

处理完局部，会break;跳出状态机。

### CLEARING

当进入到清除恢复的模块中，就意味着要结束了，这里不同于源码的move_base,这里没有将CLEARING作为恢复行为的逻辑处理，而是作为一个状态发布的最后一环。所以，只要进来了这个模块，就会报出abort

> Failed to find a valid plan. Even afer executing recovery befaviors.

这也是最后一条合理的abort路径，然后就是发布具体的NAVI_STATE，goalState == 1是障碍物的原因，goalState==2是在navi过程中未知的原因，此外的都是局部的原因fail。

最后，所有的状态机执行完，还有一个默认的模块，虽然说了永远不会执行到这里，但是还是有一条abort途径：

> bug abort.

循环走完，executeCycle3会返回false,触发第三条abort路径。

至此，move_base三个模块已经梳理完毕，后续继续梳理recovery的。