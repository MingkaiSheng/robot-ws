# 激光雷达密封环境的地图的构建与对比

## 1. SLAM建图
- 1. gmapping简介

gmapping 是ROS开源社区中较为常用且比较成熟的SLAM算法之一，gmapping可以根据移动机器人里程计数据和激光雷达数据来绘制二维的栅格地图，对应的，gmapping对硬件也有一定的要求：

a.该移动机器人可以发布里程计消息
b.机器人需要发布雷达消息(该消息可以通过水平固定安装的雷达发布，或者也可以将深度相机消息转换成雷达消息)

关于里程计与雷达数据，仿真环境中可以正常获取的，不再赘述，栅格地图如案例所示。

gmapping 安装前面也有介绍，命令如下:
```
sudo apt install ros-<ROS版本>-gmapping
```

- 2. gmapping节点说明
     
gmapping 功能包中的核心节点是:slam_gmapping。为了方便调用，需要先了解该节点订阅的话题、发布的话题、服务以及相关参数。

- 2.1订阅的Topic
  
tf (tf/tfMessage)

    用于雷达、底盘与里程计之间的坐标变换消息。
  
scan(sensor_msgs/LaserScan)

    SLAM所需的雷达信息。
    
- 2.2发布的Topic
  
map_metadata(nav_msgs/MapMetaData)

    地图元数据，包括地图的宽度、高度、分辨率等，该消息会固定更新。
    
map(nav_msgs/OccupancyGrid)

    地图栅格数据，一般会在rviz中以图形化的方式显示。
    
~entropy(std_msgs/Float64)

    机器人姿态分布熵估计(值越大，不确定性越大)。
    
- 2.3服务
  
dynamic_map(nav_msgs/GetMap)

    用于获取地图数据。
    
- 2.4参数

~base_frame(string, default:"base_link")

    机器人基坐标系。
  
~map_frame(string, default:"map")

    地图坐标系。
    
~odom_frame(string, default:"odom")

    里程计坐标系。
    
~map_update_interval(float, default: 5.0)

    地图更新频率，根据指定的值设计更新间隔。
    
~maxUrange(float, default: 80.0)

    激光探测的最大可用范围(超出此阈值，被截断)。
    
~maxRange(float)

    激光探测的最大范围。
    
参数较多，上述是几个较为常用的参数，其他参数介绍可参考官网。

- 2.5所需的坐标变换
  
雷达坐标系→基坐标系

    一般由 robot_state_publisher 或 static_transform_publisher 发布。
    
基坐标系→里程计坐标系

    一般由里程计节点发布。
    
- 2.6发布的坐标变换
  
地图坐标系→里程计坐标系

    地图到里程计坐标系之间的变换。
  
- 3. gmapping使用
     
- 3.1编写gmapping节点相关launch文件
  
launch文件编写可以参考 github 的演示 launch文件：https://github.com/ros-perception/slam_gmapping/blob/melodic-devel/gmapping/launch/slam_gmapping_pr2.launch
复制并修改如下:
```
<launch>
<param name="use_sim_time" value="true"/>
    <node pkg="gmapping" type="slam_gmapping" name="slam_gmapping" output="screen">
      <remap from="scan" to="scan"/>
      <param name="base_frame" value="base_footprint"/><!--底盘坐标系-->
      <param name="odom_frame" value="odom"/> <!--里程计坐标系-->
      <param name="map_update_interval" value="5.0"/>
      <param name="maxUrange" value="16.0"/>
      <param name="sigma" value="0.05"/>
      <param name="kernelSize" value="1"/>
      <param name="lstep" value="0.05"/>
      <param name="astep" value="0.05"/>
      <param name="iterations" value="5"/>
      <param name="lsigma" value="0.075"/>
      <param name="ogain" value="3.0"/>
      <param name="lskip" value="0"/>
      <param name="srr" value="0.1"/>
      <param name="srt" value="0.2"/>
      <param name="str" value="0.1"/>
      <param name="stt" value="0.2"/>
      <param name="linearUpdate" value="1.0"/>
      <param name="angularUpdate" value="0.5"/>
      <param name="temporalUpdate" value="3.0"/>
      <param name="resampleThreshold" value="0.5"/>
      <param name="particles" value="30"/>
      <param name="xmin" value="-50.0"/>
      <param name="ymin" value="-50.0"/>
      <param name="xmax" value="50.0"/>
      <param name="ymax" value="50.0"/>
      <param name="delta" value="0.05"/>
      <param name="llsamplerange" value="0.01"/>
      <param name="llsamplestep" value="0.01"/>
      <param name="lasamplerange" value="0.005"/>
      <param name="lasamplestep" value="0.005"/>
    </node>

    <node pkg="joint_state_publisher" name="joint_state_publisher" type="joint_state_publisher" />
    <node pkg="robot_state_publisher" name="robot_state_publisher" type="robot_state_publisher" />

    <node pkg="rviz" type="rviz" name="rviz" />
    <!-- 可以保存 rviz 配置并后期直接使用-->
    <!--
    <node pkg="rviz" type="rviz" name="rviz" args="-d $(find my_nav_sum)/rviz/gmapping.rviz"/>
    -->
</launch>
```
关键代码解释：
```
<remap from="scan" to="scan"/><!-- 雷达话题 -->
<param name="base_frame" value="base_footprint"/><!--底盘坐标系-->
<param name="odom_frame" value="odom"/> <!--里程计坐标系-->
```
- 3.2执行
- 1.先启动 Gazebo 仿真环境
- 2.然后再启动地图绘制的 launch 文件:
```
roslaunch 包名 launch文件名
```
- 3.启动键盘键盘控制节点，用于控制机器人运动建图
```
rosrun teleop_twist_keyboard teleop_twist_keyboard.py
```
- 4.在 rviz 中添加组件，显示栅格地图
![图片](https://github.com/fqy2333/robot-ws/assets/57582782/59070045-4dfc-4ec7-accb-6ec32fcab2b4)

最后，就可以通过键盘控制gazebo中的机器人运动，同时，在rviz中可以显示gmapping发布的栅格地图数据了，下一步，还需要将地图单独保存。

## 2. 地图服务

上一节我们已经实现通过gmapping的构建地图并在rviz中显示了地图，不过，上一节中地图数据是保存在内存中的，当节点关闭时，数据也会被一并释放，我们需要将栅格地图序列化到的磁盘以持久化存储，后期还要通过反序列化读取磁盘的地图数据再执行后续操作。在ROS中，地图数据的序列化与反序列化可以通过 map_server 功能包实现。

1.map_server简介

map_server功能包中提供了两个节点: map_saver 和 map_server，前者用于将栅格地图保存到磁盘，后者读取磁盘的栅格地图并以服务的方式提供出去。

map_server安装前面也有介绍，命令如下:

    sudo apt install ros-<ROS版本>-map-server

2.map_server使用之地图保存节点(map_saver)
2.1map_saver节点说明

订阅的topic:

map(nav_msgs/OccupancyGrid)

    订阅此话题用于生成地图文件。

2.2地图保存launch文件

地图保存的语法比较简单，编写一个launch文件，内容如下:

<launch>
    <arg name="filename" value="$(find mycar_nav)/map/nav" />
    <node name="map_save" pkg="map_server" type="map_saver" args="-f $(arg filename)" />
</launch>

其中 mymap 是指地图的保存路径以及保存的文件名称。

SLAM建图完毕后，执行该launch文件即可。

测试:

    首先，参考上一节，依次启动仿真环境，键盘控制节点与SLAM节点；

    然后，通过键盘控制机器人运动并绘图；

    最后，通过上述地图保存方式保存地图。

    结果：在指定路径下会生成两个文件，xxx.pgm 与 xxx.yaml
2.3 保存结果解释

xxx.pgm 本质是一张图片，直接使用图片查看程序即可打开。

xxx.yaml 保存的是地图的元数据信息，用于描述图片，内容格式如下:

image: /home/rosmelodic/ws02_nav/src/mycar_nav/map/nav.pgm
resolution: 0.050000
origin: [-50.000000, -50.000000, 0.000000]
negate: 0
occupied_thresh: 0.65
free_thresh: 0.196

解释:

    image:被描述的图片资源路径，可以是绝对路径也可以是相对路径。

    resolution: 图片分片率(单位: m/像素)。

    origin: 地图中左下像素的二维姿势，为（x，y，偏航），偏航为逆时针旋转（偏航= 0表示无旋转）。

    occupied_thresh: 占用概率大于此阈值的像素被视为完全占用。

    free_thresh: 占用率小于此阈值的像素被视为完全空闲。

    negate: 是否应该颠倒白色/黑色自由/占用的语义。

map_server 中障碍物计算规则:

    地图中的每一个像素取值在 [0,255] 之间，白色为 255，黑色为 0，该值设为 x；
    map_server 会将像素值作为判断是否是障碍物的依据，首先计算比例: p = (255 - x) / 255.0，白色为0，黑色为1(negate为true，则p = x / 255.0)；
    根据步骤2计算的比例判断是否是障碍物，如果 p > occupied_thresh 那么视为障碍物，如果 p < free_thresh 那么视为无物。

备注:

    图片也可以根据需求编辑。

3.map_server使用之地图服务(map_server)

3.1map_server节点说明

发布的话题

map_metadata（nav_msgs / MapMetaData）

    发布地图元数据。

map（nav_msgs / OccupancyGrid）

    地图数据。

服务

static_map（nav_msgs / GetMap）

    通过此服务获取地图。

参数

〜frame_id（字符串，默认值：“map”）

    地图坐标系。

3.2地图读取

通过 map_server 的 map_server 节点可以读取栅格地图数据，编写 launch 文件如下:

<launch>
    <!-- 设置地图的配置文件 -->
    <arg name="map" default="nav.yaml" />
    <!-- 运行地图服务器，并且加载设置的地图-->
    <node name="map_server" pkg="map_server" type="map_server" args="$(find mycar_nav)/map/$(arg map)"/>
</launch>

其中参数是地图描述文件的资源路径，执行该launch文件，该节点会发布话题:map(nav_msgs/OccupancyGrid)
3.3地图显示

在 rviz 中使用 map 组件可以显示栅格地图：
![图片](https://github.com/fqy2333/robot-ws/assets/57582782/5cf10816-a7f2-4eda-a6c3-376b2a47576b)


## 3. 激光雷达构建环境地图
- 1. 激光雷达接收器连接电脑上
  


- 2. 检查是否正常被连接：
```
ls /dev/ttyUSB*
```
![image](https://github.com/cztianchao/robot-ws/assets/41623939/fc576ab6-aa23-4240-b9bc-4403407e0db4)

- 3. 启动ROS雷达驱动：
```
roslaunch tianbot_mini lidar.launch
```
![image](https://github.com/cztianchao/robot-ws/assets/41623939/19dcadf6-ab7c-4735-aeee-2a858107faf0)



- 4. 若采样率 >=4k ,方为正常，若不大于等于4k：ctrl c 关闭驱动 ，重启即可
  
  **第一次为3k**
  
![image](https://github.com/cztianchao/robot-ws/assets/41623939/5a3455f7-ba60-4e17-ae65-8427708990e3)

  **第二次为5k（可以使用）**
  
![image](https://github.com/cztianchao/robot-ws/assets/41623939/8ae2e356-b6db-4aac-8b88-2fc75e1dcace)


- 5. 打开新终端输入以下命令以运行SLAM，进行地图采集：
```
roslaunch tianbot_mini slam.launch
```
**成功运行截图：**

![2023-06-23 09-19-21 的屏幕截图](https://github.com/MingkaiSheng/robot-ws/assets/41623939/8318b871-fb6b-4633-bb67-746a9ec4b76a)
运行成功后进入rviz界面：

![2023-06-23 09-19-32 的屏幕截图](https://github.com/MingkaiSheng/robot-ws/assets/41623939/092e95dc-b36c-4a84-86a9-8c3e43b40772)


- 6. 打开新终端输入以下命令以运行键盘遥控，以使用键盘操作小车移动实现地图采集：

```
roslaunch tianbot_mini teleop.launch
```

**成功运行截图：**

![2023-06-24 14-54-43 的屏幕截图](https://github.com/MingkaiSheng/robot-ws/assets/41623939/ff69cbab-53b4-4196-a656-66f8340dc705)

- 7. 采集结束后运行map_save.launch来保存地图，地图将保存在maps文件夹中：
```
roslaunch tianbot_mini map_save.launch
```
**成功保存截图：**

![2023-06-24 15-56-04 的屏幕截图](https://github.com/cztianchao/robot-ws/assets/41623939/ac868122-cf09-4879-bc79-bb237cb141fa)
**map_save.launch中的内容**

![2023-06-24 15-56-18 的屏幕截图](https://github.com/cztianchao/robot-ws/assets/41623939/36202a1e-f8f4-403f-8eea-317404196fcc)


- 8. 查看现实与扫描地图对比

  1）扫描地图
  
   ![image](https://github.com/cztianchao/robot-ws/assets/41623939/dd7c7ca8-844c-42bd-988a-1a2cec82e3d4)

  
  2）真实环境

  ![IMG_20230624_155227](https://github.com/cztianchao/robot-ws/assets/41623939/5e5201e9-d1f1-44d7-ad30-8abeafac76a9)


  3）对比效果

  > 建图后障碍物略有调整
  
  ![image](https://github.com/cztianchao/robot-ws/assets/41623939/afebd735-39d1-4c36-9866-1aa8957f7ed5)

