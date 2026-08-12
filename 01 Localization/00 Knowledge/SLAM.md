- State Estimation / Localization
  - Odometry
    - Wheel Odometry
    - Visual Odometry (VO)
      - Visual-Inertial Odometry (VIO)
        - Camera + IMU
    - LiDAR Odometry (LO)
      - LiDAR-Inertial Odometry (LIO)
        - LiDAR + IMU
    - Multi-Sensor Odometry
  - SLAM
    - Localization
    - Mapping
    - Loop Closure
    - Global Optimization
    - Visual SLAM
      - Camera
      - Camera + IMU
    - LiDAR SLAM
      - LiDAR
      - LiDAR + IMU
    - Multi-Modal SLAM


**Simultaneous Localization And Mapping**
## **定位（Localization）**
## **建图（Mapping）**

没有地图
↓
无法定位

不知道自己在哪
↓
无法建图

传感器
↓
估计自己位置
↓
更新地图
↓
利用地图修正位置
↓
更新地图