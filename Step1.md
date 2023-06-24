# 小车开箱使用与基本配置。

## 1. 小车组件介绍
- 1. 小车主体

  ![image](https://github.com/RichardMJT/robot-ws/assets/51042944/bc64fe06-7ccb-4ce8-ab6d-ab2530c6cb7e)
  ```
  TIANBOT迷你机器人配备360°激光雷达传感器，集成差速运动控制系统，只需3步即可实现SLAM建图导航功能，从开机到建图导航仅需数秒钟，快速帮您使用ROS机器人操作系统控制与构建智能机器人，同时可通过预置接口，连接扩展模块，轻松拓展更加丰富的应用场景。
  ```
  ```
  指示灯说明：
  1. 系统状态指示灯
     黄灯闪烁：热点模式启动30S内需要连接到TBMINI-XXX热点
     绿色常亮：已有客户端正常连接到TBMINI-XXX热点
     白灯常亮：进入遥控控制模式，可以在网页端进行遥控控制
  2. 雷达状态指示灯
     红灯常亮，蓝灯常亮，已正常配对连接
     红灯闪烁，此时雷达未连接成功，需检查线路以及接收器是否连接正常
  3. 电量指示灯
     在一格时指示灯闪烁，电量过低，处于该状态时，请立即充电，系统在此电量时，将无法保证正常工作
  ```
- 2.  ROS2GO U 盘系统


![IMG_20230623_095955](https://github.com/MingkaiSheng/robot-ws/assets/41623939/9a1f7bae-ac3e-4bd8-b16f-be8dc9e57e70)

  
```
U盘中预装有基于Ubuntu的深度定制的Ubuntu18.04+ ROS Melodic（我们的系统进行了更新，新版本为Ubuntu20.04+ ROS Noetic），内置完整ROS开发环境，无需配置，可直接插入电脑使用。
```
```
设备要求：
  计算机支持UEFI
```
- 3. 接收天线
     
     ![image](https://github.com/RichardMJT/robot-ws/assets/51042944/11634332-9b4a-4c3f-af30-8ee6968e558c)

```
可以通过天线和接收器与机器人之间进行数据传输
```

## 2 链接启动
- 1. ROS2GO U 盘系统启动

系统盘中已经预置了ROS2以及所需要的驱动，用户只需要将其提供的U盘插入电脑即可使用，具体步骤如下：
```
1. 将ROS2GO插入电脑的USB端口（USB端口协议需要为3.0）
2. 进入BIOS，启动项选择"TIANBOT ROS2GO"（各品牌按键不同，具体参考https://zhuanlan.zhihu.com/p/358043625）
3. 之后选择保存并重启，进入系统
```
**进入 ROS2GO 系统后界面**

![2023-06-23 09-14-54 的屏幕截图](https://github.com/MingkaiSheng/robot-ws/assets/41623939/fd3b1718-b812-4b51-9258-bf3fbd404bf3)


- 2. 小车链接

进入系统后将小车与电脑进行连接，具体步骤如下：
```
1. 长按小车上的电源键3秒，在出现“滴”的一声后小车开启，机关雷达开始转动，系统灯黄灯闪烁，设备进入热点模式（首次使用）
2. 热点模式持续30秒，迷你机器人会创建TBMINI-XXXX热点，该热点未加密，使用电脑连接该热点，系统状态指示灯变为绿色。
```
连接小车成功后界面：

![image](https://github.com/RichardMJT/robot-ws/assets/51042944/7fb86e13-61f9-4208-bfd0-40de7162aa14)


- 3. 初始化

电脑连接小车后，依次执行一下命令，完成系统的初始化

1. 打开终端输入以下命令以运行迷你机器人驱动：
```
roslaunch tianbot_mini bringup.launch
```
**成功运行截图：**

![2023-06-23 09-17-33 的屏幕截图](https://github.com/MingkaiSheng/robot-ws/assets/41623939/9cff83cb-5116-4668-8e6d-e72db9a55ab5)

2. 打开新终端输入以下命令以运行雷达驱动：
```
roslaunch tianbot_mini lidar.launch
```
> 若运行指令后显示的采样率小于4K，则重启该驱动，直至采样率大于等于4K

**成功运行截图：**

![2023-06-23 09-17-58 的屏幕截图](https://github.com/MingkaiSheng/robot-ws/assets/41623939/928513d4-9545-4731-859f-9de373efecf7)

3. 打开新终端输入以下命令以运行SLAM：
```
roslaunch tianbot_mini slam.launch
```
**成功运行截图：**

![2023-06-23 09-19-21 的屏幕截图](https://github.com/MingkaiSheng/robot-ws/assets/41623939/8318b871-fb6b-4633-bb67-746a9ec4b76a)
运行成功后进入rviz界面：

![2023-06-23 09-19-32 的屏幕截图](https://github.com/MingkaiSheng/robot-ws/assets/41623939/092e95dc-b36c-4a84-86a9-8c3e43b40772)


4. 打开新终端输入以下命令以运行键盘遥控：

```
roslaunch tianbot_mini teleop.launch
```

**成功运行截图：**
![2023-06-24 14-54-43 的屏幕截图](https://github.com/MingkaiSheng/robot-ws/assets/41623939/ff69cbab-53b4-4196-a656-66f8340dc705)









