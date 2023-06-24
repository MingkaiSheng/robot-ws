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


- 5. 运行slam.launch，进行地图采集

  运行成功画面：
  

- 9. 查看现实与扫描地图对比

  1）扫描地图
  
   ![image](https://github.com/cztianchao/robot-ws/assets/41623939/dd7c7ca8-844c-42bd-988a-1a2cec82e3d4)

  
  2）真实环境

  ![IMG_20230624_155227](https://github.com/cztianchao/robot-ws/assets/41623939/5e5201e9-d1f1-44d7-ad30-8abeafac76a9)


  3）对比效果

  > 建图后障碍物略有调整
  
  ![image](https://github.com/cztianchao/robot-ws/assets/41623939/afebd735-39d1-4c36-9866-1aa8957f7ed5)

