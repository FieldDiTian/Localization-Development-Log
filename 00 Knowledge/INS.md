INS = **Inertial Navigation System，惯性导航系统**。

它用 **IMU** 的加速度计和陀螺仪来连续估计车辆状态：

- 位置
- 速度
- 姿态，也就是 roll / pitch / yaw
- 有时还融合 GNSS、轮速、磁力计等外部信息

在你这个仓库语境里，**Point One Atlas INS** 指的是 Atlas 设备内部做了融合导航：它接收 GNSS/RTK + IMU，然后发布：

- `/gps_p1/imu`：校准后的 IMU 数据
- `/gps_p1/filtered_odom`：融合后的 odometry，也就是 INS 估计出来的车辆位姿/速度

简单区分：

- **IMU**：原始惯性传感器，输出加速度和角速度
- **GNSS/RTK**：卫星定位，给绝对位置
- **INS**：把 IMU 和 GNSS/RTK 等融合起来，输出连续、平滑、带姿态/速度的导航解

