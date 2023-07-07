# 动态重置和指定机器人位置(里程计)

## 1. 在网页中动态重置迷你机器人的里程计信息
- 1. 运行
```shell
rostopic list
```
查看机器人当前话题的列表：

![2023-06-23 09-18-06 的屏幕截图](https://github.com/MingkaiSheng/robot-ws/assets/41623939/4185e287-9711-47eb-acf7-53ab6cfc601e)

- 2. 运行
```shell
rostopic echo /tianbot_mini/odom
```
可在ROS终端中看到机器人的里程计数据 ：

![2023-06-23 09-20-43 的屏幕截图](https://github.com/MingkaiSheng/robot-ws/assets/41623939/c77b3090-0bf0-47a6-b783-bc01c0fc9672)
- 3. 在网页中动态重置迷你机器人的里程计信息
在该试验开始前，请确保当前主机连接到机器人的热点
打开主机中的浏览器，输入迷你机器人的手柄控制页面地址：
```shell
192.168.1.1/joystick
```
加载完成后可以看到如下界面：

![2023-06-23 09-22-53 的屏幕截图](https://github.com/MingkaiSheng/robot-ws/assets/41623939/9915d5c6-add0-4ca4-bf17-3b7b0ca1dc65)
点击"Odom里程计复位(0,0)"按钮，便可对里程计信息进行重置。
**重置前**

![2023-06-23 09-23-30 的屏幕截图](https://github.com/MingkaiSheng/robot-ws/assets/41623939/e14db7e2-74c6-4fe2-a160-c0ab8e101348)
**重置后**

![2023-06-23 09-23-37 的屏幕截图](https://github.com/MingkaiSheng/robot-ws/assets/41623939/a3784fb6-1a6f-424f-98c1-afd795585401)

## 2 使用ROS话题来清零和指定迷你机器人的里程计
- 1. 运行
```shell
rostopic list
```
查看机器人当前话题的列表：

![2023-06-23 09-18-06 的屏幕截图](https://github.com/MingkaiSheng/robot-ws/assets/41623939/4185e287-9711-47eb-acf7-53ab6cfc601e)

下面这个命令
```shell
/tianbot_mini/cmd_dbg
```
便是系统预留的机器人的底层接口，机器人里程计的重置也通过这个接口来实现。
- 2. 使用ROS话题来清零迷你机器人的里程计
首先手动移动机器人获得里程数据
然后运行
```shell
rostopic pub /tianbot_mini/cmd_dbg std_msgs/String "data: 'setodom 0 0'" 
```
上述命令中的‘setodom 0 0’表示重置机器人里程计的x和y，同时上述命令也可以主动的让里程计显示在特定的位置上。只需要修改相应的x和y值即可

**重置前**

![2023-06-23 09-24-51 的屏幕截图](https://github.com/MingkaiSheng/robot-ws/assets/41623939/41ef7f95-f395-4386-91fe-3a5a9bc229db)
**重置后**

![2023-06-23 09-24-59 的屏幕截图](https://github.com/MingkaiSheng/robot-ws/assets/41623939/043d1825-1936-4b9e-a4db-dd8bf5917b87)
通过修改steodom后的数值可以指定小车的位置，如
```shell
rostopic pub /tianbot_mini/cmd_dbg std_msgs/String "data: 'setodom 2 3'" 
```
小车的里程计的x值将被置为2，y值将被置为3。
**指定前**

![2023-06-23 09-25-12 的屏幕截图](https://github.com/MingkaiSheng/robot-ws/assets/41623939/14e98715-9628-4969-b322-a90a82529ed0)
**指定后**

![2023-06-23 09-25-15 的屏幕截图](https://github.com/MingkaiSheng/robot-ws/assets/41623939/7fd80151-ba72-4a49-b465-507e5afa56f9)
> 若出现终端显示不完整问题可考虑重新输入上述命令
