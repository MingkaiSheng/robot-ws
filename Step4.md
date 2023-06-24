# 小车障碍走廊动态路径导航

## 1. 定位

所谓定位就是推算机器人自身在全局地图中的位置，当然，SLAM中也包含定位算法实现，不过SLAM的定位是用于构建全局地图的，是属于导航开始之前的阶段，而当前定位是用于导航中，导航中，机器人需要按照设定的路线运动，通过定位可以判断机器人的实际轨迹是否符合预期。在ROS的导航功能包集navigation中提供了 amcl 功能包，用于实现导航中的机器人定位。

- 1.amcl简介

AMCL(adaptive Monte Carlo Localization) 是用于2D移动机器人的概率定位系统，它实现了自适应（或KLD采样）蒙特卡洛定位方法，可以根据已有地图使用粒子滤波器推算机器人位置。
amcl已经被集成到了navigation包，navigation安装前面也有介绍，命令如下:
```
sudo apt install ros-<ROS版本>-navigation
```

- 2. amcl节点说明
 
amcl 功能包中的核心节点是:amcl。为了方便调用，需要先了解该节点订阅的话题、发布的话题、服务以及相关参数。

2.1订阅的Topic

scan(sensor_msgs/LaserScan)

    激光雷达数据。

tf(tf/tfMessage)

    坐标变换消息。

initialpose(geometry_msgs/PoseWithCovarianceStamped)

    用来初始化粒子滤波器的均值和协方差。

map(nav_msgs/OccupancyGrid)

    获取地图数据。

2.2发布的Topic

amcl_pose(geometry_msgs/PoseWithCovarianceStamped)

    机器人在地图中的位姿估计。

particlecloud(geometry_msgs/PoseArray)

    位姿估计集合，rviz中可以被 PoseArray 订阅然后图形化显示机器人的位姿估计集合。

tf(tf/tfMessage)

    发布从 odom 到 map 的转换。

2.3服务

global_localization(std_srvs/Empty)

    初始化全局定位的服务。

request_nomotion_update(std_srvs/Empty)

    手动执行更新和发布更新的粒子的服务。

set_map(nav_msgs/SetMap)

    手动设置新地图和姿态的服务。

2.4调用的服务

static_map(nav_msgs/GetMap)

    调用此服务获取地图数据。

2.5参数

~odom_model_type(string, default:"diff")

    里程计模型选择: "diff","omni","diff-corrected","omni-corrected" (diff 差速、omni 全向轮)

~odom_frame_id(string, default:"odom")

    里程计坐标系。

~base_frame_id(string, default:"base_link")

    机器人极坐标系。

~global_frame_id(string, default:"map")

    地图坐标系。

参数较多，上述是几个较为常用的参数，其他参数介绍可参考官网。
2.6坐标变换

里程计本身也是可以协助机器人定位的，不过里程计存在累计误差且一些特殊情况时(车轮打滑)会出现定位错误的情况，amcl 则可以通过估算机器人在地图坐标系下的姿态，再结合里程计提高定位准确度。

    里程计定位:只是通过里程计数据实现 /odom_frame 与 /base_frame 之间的坐标变换。
    amcl定位: 可以提供 /map_frame 、/odom_frame 与 /base_frame 之间的坐标变换。
![图片](https://github.com/fqy2333/robot-ws/assets/57582782/fe065210-d46b-43e9-9e70-25e8c153bfa3)

3.amcl使用

3.1编写amcl节点相关的launch文件

关于launch文件的实现，在amcl功能包下的example目录已经给出了示例，可以作为参考，具体实现:

roscd amcl
ls examples

该目录下会列出两个文件: amcl_diff.launch 和 amcl_omni.launch 文件，前者适用于差分移动机器人，后者适用于全向移动机器人，可以按需选择，此处参考前者，新建 launch 文件，复制 amcl_diff.launch 文件内容并修改如下:

<launch>
<node pkg="amcl" type="amcl" name="amcl" output="screen">
  <!-- Publish scans from best pose at a max of 10 Hz -->
  <param name="odom_model_type" value="diff"/><!-- 里程计模式为差分 -->
  <param name="odom_alpha5" value="0.1"/>
  <param name="transform_tolerance" value="0.2" />
  <param name="gui_publish_rate" value="10.0"/>
  <param name="laser_max_beams" value="30"/>
  <param name="min_particles" value="500"/>
  <param name="max_particles" value="5000"/>
  <param name="kld_err" value="0.05"/>
  <param name="kld_z" value="0.99"/>
  <param name="odom_alpha1" value="0.2"/>
  <param name="odom_alpha2" value="0.2"/>
  <!-- translation std dev, m -->
  <param name="odom_alpha3" value="0.8"/>
  <param name="odom_alpha4" value="0.2"/>
  <param name="laser_z_hit" value="0.5"/>
  <param name="laser_z_short" value="0.05"/>
  <param name="laser_z_max" value="0.05"/>
  <param name="laser_z_rand" value="0.5"/>
  <param name="laser_sigma_hit" value="0.2"/>
  <param name="laser_lambda_short" value="0.1"/>
  <param name="laser_lambda_short" value="0.1"/>
  <param name="laser_model_type" value="likelihood_field"/>
  <!-- <param name="laser_model_type" value="beam"/> -->
  <param name="laser_likelihood_max_dist" value="2.0"/>
  <param name="update_min_d" value="0.2"/>
  <param name="update_min_a" value="0.5"/>

  <param name="odom_frame_id" value="odom"/><!-- 里程计坐标系 -->
  <param name="base_frame_id" value="base_footprint"/><!-- 添加机器人基坐标系 -->
  <param name="global_frame_id" value="map"/><!-- 添加地图坐标系 -->

  <param name="resample_interval" value="1"/>
  <param name="transform_tolerance" value="0.1"/>
  <param name="recovery_alpha_slow" value="0.0"/>
  <param name="recovery_alpha_fast" value="0.0"/>
</node>
</launch>

3.2编写测试launch文件

amcl节点是不可以单独运行的，运行 amcl 节点之前，需要先加载全局地图，然后启动 rviz 显示定位结果，上述节点可以集成进launch文件，内容示例如下:

<launch>
    <!-- 设置地图的配置文件 -->
    <arg name="map" default="nav.yaml" />
    <!-- 运行地图服务器，并且加载设置的地图-->
    <node name="map_server" pkg="map_server" type="map_server" args="$(find mycar_nav)/map/$(arg map)"/>
    <!-- 启动AMCL节点 -->
    <include file="$(find mycar_nav)/launch/amcl.launch" />
    <!-- 运行rviz -->
    <node pkg="rviz" type="rviz" name="rviz"/>
</launch>

当然，launch文件中地图服务节点和amcl节点中的包名、文件名需要根据自己的设置修改。

3.3执行

1.先启动 Gazebo 仿真环境(此过程略)；

2.启动键盘控制节点：

rosrun teleop_twist_keyboard teleop_twist_keyboard.py

3.启动上一步中集成地图服务、amcl 与 rviz 的 launch 文件；

4.在启动的 rviz 中，添加RobotModel、Map组件，分别显示机器人模型与地图，添加 posearray 插件，设置topic为particlecloud来显示 amcl 预估的当前机器人的位姿，箭头越是密集，说明当前机器人处于此位置的概率越高；

5.通过键盘控制机器人运动，会发现 posearray 也随之而改变。
![图片](https://github.com/fqy2333/robot-ws/assets/57582782/e1cc349d-41c7-4770-be4b-8977c5b50ecf)

## 2. 路径规划

毋庸置疑的，路径规划是导航中的核心功能之一，在ROS的导航功能包集navigation中提供了 move_base 功能包，用于实现此功能。
1.move_base简介

move_base 功能包提供了基于动作(action)的路径规划实现，move_base 可以根据给定的目标点，控制机器人底盘运动至目标位置，并且在运动过程中会连续反馈机器人自身的姿态与目标点的状态信息。如前所述(7.1)move_base主要由全局路径规划与本地路径规划组成。

move_base已经被集成到了navigation包，navigation安装前面也有介绍，命令如下:

    sudo apt install ros-<ROS版本>-navigation

2.move_base节点说明

move_base功能包中的核心节点是:move_base。为了方便调用，需要先了解该节点action、订阅的话题、发布的话题、服务以及相关参数。

2.1动作

动作订阅

move_base/goal(move_base_msgs/MoveBaseActionGoal)

    move_base 的运动规划目标。

move_base/cancel(actionlib_msgs/GoalID)

    取消目标。

动作发布

move_base/feedback(move_base_msgs/MoveBaseActionFeedback)

    连续反馈的信息，包含机器人底盘坐标。

move_base/status(actionlib_msgs/GoalStatusArray)

    发送到move_base的目标状态信息。

move_base/result(move_base_msgs/MoveBaseActionResult)

    操作结果(此处为空)。

2.2订阅的Topic

move_base_simple/goal(geometry_msgs/PoseStamped)

    运动规划目标(与action相比，没有连续反馈，无法追踪机器人执行状态)。

2.3发布的Topic

cmd_vel(geometry_msgs/Twist)

    输出到机器人底盘的运动控制消息。

2.4服务

~make_plan(nav_msgs/GetPlan)

    请求该服务，可以获取给定目标的规划路径，但是并不执行该路径规划。

~clear_unknown_space(std_srvs/Empty)

    允许用户直接清除机器人周围的未知空间。

~clear_costmaps(std_srvs/Empty)

    允许清除代价地图中的障碍物，可能会导致机器人与障碍物碰撞，请慎用。

2.5参数

3.move_base与代价地图

3.1概念

机器人导航(尤其是路径规划模块)是依赖于地图的，地图在SLAM时已经有所介绍了，ROS中的地图其实就是一张图片，这张图片有宽度、高度、分辨率等元数据，在图片中使用灰度值来表示障碍物存在的概率。不过SLAM构建的地图在导航中是不可以直接使用的，因为：

    SLAM构建的地图是静态地图，而导航过程中，障碍物信息是可变的，可能障碍物被移走了，也可能添加了新的障碍物，导航中需要时时的获取障碍物信息；
    在靠近障碍物边缘时，虽然此处是空闲区域，但是机器人在进入该区域后可能由于其他一些因素，比如：惯性、或者不规则形体的机器人转弯时可能会与障碍物产生碰撞，安全起见，最好在地图的障碍物边缘设置警戒区，尽量禁止机器人进入..

所以，静态地图无法直接应用于导航，其基础之上需要添加一些辅助信息的地图，比如时时获取的障碍物数据，基于静态地图添加的膨胀区等数据。

3.2组成

代价地图有两张:global_costmap(全局代价地图) 和 local_costmap(本地代价地图)，前者用于全局路径规划，后者用于本地路径规划。

两张代价地图都可以多层叠加,一般有以下层级:

    Static Map Layer：静态地图层，SLAM构建的静态地图。

    Obstacle Map Layer：障碍地图层，传感器感知的障碍物信息。

    Inflation Layer：膨胀层，在以上两层地图上进行膨胀（向外扩张），以避免机器人的外壳会撞上障碍物。

    Other Layers：自定义costmap。

多个layer可以按需自由搭配。
![图片](https://github.com/fqy2333/robot-ws/assets/57582782/3517afd7-3ab0-4cc2-a7d9-9b5d98d9f6f6)

3.3碰撞算法

在ROS中，如何计算代价值呢？请看下图:
![图片](https://github.com/fqy2333/robot-ws/assets/57582782/8296c47a-9c0f-4ed1-99f6-6de4e97b0763)

上图中，横轴是距离机器人中心的距离，纵轴是代价地图中栅格的灰度值。

    致命障碍:栅格值为254，此时障碍物与机器人中心重叠，必然发生碰撞；
    内切障碍:栅格值为253，此时障碍物处于机器人的内切圆内，必然发生碰撞；
    外切障碍:栅格值为[128,252]，此时障碍物处于其机器人的外切圆内，处于碰撞临界，不一定发生碰撞；
    非自由空间:栅格值为(0,127]，此时机器人处于障碍物附近，属于危险警戒区，进入此区域，将来可能会发生碰撞；
    自由区域:栅格值为0，此处机器人可以自由通过；
    未知区域:栅格值为255，还没探明是否有障碍物。

膨胀空间的设置可以参考非自由空间。

4.move_base使用

路径规划算法在move_base功能包的move_base节点中已经封装完毕了，但是还不可以直接调用，因为算法虽然已经封装了，但是该功能包面向的是各种类型支持ROS的机器人，不同类型机器人可能大小尺寸不同，传感器不同，速度不同，应用场景不同....最后可能会导致不同的路径规划结果，那么在调用路径规划节点之前，我们还需要配置机器人参数。具体实现如下:

    先编写launch文件模板
    编写配置文件
    集成导航相关的launch文件
    测试

4.1launch文件

关于move_base节点的调用，模板如下:

<launch>

    <node pkg="move_base" type="move_base" respawn="false" name="move_base" output="screen" clear_params="true">
        <rosparam file="$(find 功能包)/param/costmap_common_params.yaml" command="load" ns="global_costmap" />
        <rosparam file="$(find 功能包)/param/costmap_common_params.yaml" command="load" ns="local_costmap" />
        <rosparam file="$(find 功能包)/param/local_costmap_params.yaml" command="load" />
        <rosparam file="$(find 功能包)/param/global_costmap_params.yaml" command="load" />
        <rosparam file="$(find 功能包)/param/base_local_planner_params.yaml" command="load" />
    </node>

</launch>

launch文件解释:

启动了 move_base 功能包下的 move_base 节点，respawn 为 false，意味着该节点关闭后，不会被重启；clear_params 为 true，意味着每次启动该节点都要清空私有参数然后重新载入；通过 rosparam 会载入若干 yaml 文件用于配置参数，这些yaml文件的配置以及作用详见下一小节内容。

4.2配置文件

关于配置文件的编写，可以参考一些成熟的机器人的路径规划实现，比如: turtlebot3，github链接：https://github.com/ROBOTIS-GIT/turtlebot3/tree/master/turtlebot3_navigation/param，先下载这些配置文件备用。

在功能包下新建 param 目录，复制下载的文件到此目录: costmap_common_params_burger.yaml、local_costmap_params.yaml、global_costmap_params.yaml、base_local_planner_params.yaml，并将costmap_common_params_burger.yaml 重命名为:costmap_common_params.yaml。

配置文件修改以及解释:

4.2.1 costmap_common_params.yaml

该文件是move_base 在全局路径规划与本地路径规划时调用的通用参数，包括:机器人的尺寸、距离障碍物的安全距离、传感器信息等。配置参考如下:
```
#机器人几何参，如果机器人是圆形，设置 robot_radius,如果是其他形状设置 footprint
robot_radius: 0.12 #圆形
# footprint: [[-0.12, -0.12], [-0.12, 0.12], [0.12, 0.12], [0.12, -0.12]] #其他形状

obstacle_range: 3.0 # 用于障碍物探测，比如: 值为 3.0，意味着检测到距离小于 3 米的障碍物时，就会引入代价地图
raytrace_range: 3.5 # 用于清除障碍物，比如：值为 3.5，意味着清除代价地图中 3.5 米以外的障碍物


#膨胀半径，扩展在碰撞区域以外的代价区域，使得机器人规划路径避开障碍物
inflation_radius: 0.2
#代价比例系数，越大则代价值越小
cost_scaling_factor: 3.0

#地图类型
map_type: costmap
#导航包所需要的传感器
observation_sources: scan
#对传感器的坐标系和数据进行配置。这个也会用于代价地图添加和清除障碍物。例如，你可以用激光雷达传感器用于在代价地图添加障碍物，再添加kinect用于导航和清除障碍物。
scan: {sensor_frame: laser, data_type: LaserScan, topic: scan, marking: true, clearing: true}
```
4.2.2 global_costmap_params.yaml

该文件用于全局代价地图参数设置:
```
global_costmap:
  global_frame: map #地图坐标系
  robot_base_frame: base_footprint #机器人坐标系
  # 以此实现坐标变换

  update_frequency: 1.0 #代价地图更新频率
  publish_frequency: 1.0 #代价地图的发布频率
  transform_tolerance: 0.5 #等待坐标变换发布信息的超时时间

  static_map: true # 是否使用一个地图或者地图服务器来初始化全局代价地图，如果不使用静态地图，这个参数为false.
```
4.2.3 local_costmap_params.yaml

该文件用于局部代价地图参数设置:
```
local_costmap:
  global_frame: odom #里程计坐标系
  robot_base_frame: base_footprint #机器人坐标系

  update_frequency: 10.0 #代价地图更新频率
  publish_frequency: 10.0 #代价地图的发布频率
  transform_tolerance: 0.5 #等待坐标变换发布信息的超时时间

  static_map: false  #不需要静态地图，可以提升导航效果
  rolling_window: true #是否使用动态窗口，默认为false，在静态的全局地图中，地图不会变化
  width: 3 # 局部地图宽度 单位是 m
  height: 3 # 局部地图高度 单位是 m
  resolution: 0.05 # 局部地图分辨率 单位是 m，一般与静态地图分辨率保持一致
```
4.2.4 base_local_planner_params

基本的局部规划器参数配置，这个配置文件设定了机器人的最大和最小速度限制值，也设定了加速度的阈值。

```
TrajectoryPlannerROS:
# Robot Configuration Parameters
  max_vel_x: 0.5 # X 方向最大速度
  min_vel_x: 0.1 # X 方向最小速速

  max_vel_theta:  1.0 # 
  min_vel_theta: -1.0
  min_in_place_vel_theta: 1.0

  acc_lim_x: 1.0 # X 加速限制
  acc_lim_y: 0.0 # Y 加速限制
  acc_lim_theta: 0.6 # 角速度加速限制

# Goal Tolerance Parameters，目标公差
  xy_goal_tolerance: 0.10
  yaw_goal_tolerance: 0.05

# Differential-drive robot configuration
# 是否是全向移动机器人
  holonomic_robot: false

# Forward Simulation Parameters，前进模拟参数
  sim_time: 0.8
  vx_samples: 18
  vtheta_samples: 20
  sim_granularity: 0.05
```

4.2.5 参数配置技巧

以上配置在实操中，可能会出现机器人在本地路径规划时与全局路径规划不符而进入膨胀区域出现假死的情况，如何尽量避免这种情形呢？

    全局路径规划与本地路径规划虽然设置的参数是一样的，但是二者路径规划和避障的职能不同，可以采用不同的参数设置策略:

        全局代价地图可以将膨胀半径和障碍物系数设置的偏大一些；
        本地代价地图可以将膨胀半径和障碍物系数设置的偏小一些。

    这样，在全局路径规划时，规划的路径会尽量远离障碍物，而本地路径规划时，机器人即便偏离全局路径也会和障碍物之间保留更大的自由空间，从而避免了陷入“假死”的情形。

4.3 launch文件集成

如果要实现导航，需要集成地图服务、amcl 、move_base 与 Rviz 等，集成示例如下:

<launch>
    <!-- 设置地图的配置文件 -->
    <arg name="map" default="nav.yaml" />
    <!-- 运行地图服务器，并且加载设置的地图-->
    <node name="map_server" pkg="map_server" type="map_server" args="$(find mycar_nav)/map/$(arg map)"/>
    <!-- 启动AMCL节点 -->
    <include file="$(find mycar_nav)/launch/amcl.launch" />

    <!-- 运行move_base节点 -->
    <include file="$(find mycar_nav)/launch/path.launch" />
    <!-- 运行rviz -->
    <node pkg="rviz" type="rviz" name="rviz" args="-d $(find mycar_nav)/rviz/nav.rviz" />

</launch>

4.4 测试

1.先启动 Gazebo 仿真环境(此过程略)；

2.启动导航相关的 launch 文件；

3.添加Rviz组件(参考演示结果),可以将配置数据保存，后期直接调用；

全局代价地图与本地代价地图组件配置如下:
![图片](https://github.com/fqy2333/robot-ws/assets/57582782/66150ee8-001e-46ce-bd01-67e9a491c492)

全局路径规划与本地路径规划组件配置如下:
![图片](https://github.com/fqy2333/robot-ws/assets/57582782/1f46fcd2-54d9-49f7-9edc-cbdd590239c1)

4.通过Rviz工具栏的 2D Nav Goal设置目的地实现导航。
![图片](https://github.com/fqy2333/robot-ws/assets/57582782/63fe9c89-1b33-4652-b24f-1fe0af47f73e)

5.也可以在导航过程中，添加新的障碍物，机器人也可以自动躲避障碍物。

## 3. 导航与SLAM建图-最后

上述需求是可行的。虽然可能会有疑问，导航时需要地图信息，之前导航实现时，是通过 map_server 包的 map_server 节点来发布地图信息的，如果不先通过SLAM建图，那么如何发布地图信息呢？SLAM建图过程中本身就会时时发布地图信息，所以无需再使用map_server，SLAM已经发布了话题为 /map 的地图消息了，且导航需要定位模块，SLAM本身也是可以实现定位的。

该过程实现比较简单，步骤如下:

    编写launch文件，集成SLAM与move_base相关节点；
    执行launch文件并测试。

1.编写launc文件

当前launch文件实现，无需调用map_server的相关节点，只需要启动SLAM节点与move_base节点，示例内容如下:

<launch>
    <!-- 启动SLAM节点 -->
    <include file="$(find mycar_nav)/launch/slam.launch" />
    <!-- 运行move_base节点 -->
    <include file="$(find mycar_nav)/launch/path.launch" />
    <!-- 运行rviz -->
    <node pkg="rviz" type="rviz" name="rviz" args="-d $(find mycar_nav)/rviz/nav.rviz" />
</launch>

2.测试

1.首先运行gazebo仿真环境；

2.然后执行launch文件；

3.在rviz中通过2D Nav Goal设置目标点，机器人开始自主移动并建图了；

4.最后可以使用 map_server 保存地图。
![图片](https://github.com/fqy2333/robot-ws/assets/57582782/9f5162cc-54b2-4812-97fa-128cecdf00ad)


