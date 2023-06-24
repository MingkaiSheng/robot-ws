# 激光雷达密封环境的地图的构建与对比

## 1. 激光雷达构建环境地图
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

