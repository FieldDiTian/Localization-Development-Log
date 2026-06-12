### **GLIM 已完成 P1 数据重构**

 使用 P1 INS
-使用 P1 IMU

Luminar 时间戳问题（当前最大待办）
### **理解 Luminar 每点时间戳**
LiDAR 点云中每个点的时间信息。
因为：
**要开启 Deskew**
当前还没正式开启：

Deskew 的作用：
在激光雷达扫描期间：
车辆会移动。
如果不校正：
同一帧中的点来自不同时间。
结果：
- 墙会弯
- 路边会扭曲
- 地图会变形
所以：
GLIM 建图质量依赖正确的点级时间戳。

RTK 与建图启动逻辑重大修改**
新逻辑**
GLIM 现在： **必须 RTK FIXED**才允许：
- 建立第一帧
- 启动地图
### **GLIM 和 GICP 不再共用同一组 IMU Gain**

GLIM**

用途：

地图构建阶段。

特点：

- 车速低
- 运动缓慢
- 希望地图稳定

因此：

IMU gain 调得更紧（tighter）。

---

### **GICP**

用途：

运行时定位。

特点：

- 车辆姿态变化更大
- 运动更激烈

因此：

IMU gain 调得更松（looser）。

允许：

- 更大的 pose 变化
- 更大的 orientation 变化

但仍受地图约束。

# **当前整体状态**

### **已完成**

- P1 接入 GLIM
- 停止双重校准
- 停止双重补偿
- RTK FIXED 启动机制
- GLIM/GICP 分离 IMU gain
- 更新 art-jazzy 分支 README

### **正在进行**

- IMU 标定残余误差排查
- Luminar 点级时间戳解析
- Ubuntu 24.04 + ROS2 Jazzy 验证
- Deskew 功能接入

### **尚未完成**

- VectorNav 接入
- Filtered NovAtel IMU 接入
- 完整 LiDAR deskew
- Luminar timestamp 规范确认