GNSS 是 **Global Navigation Satellite System**，中文通常叫 **全球导航卫星系统**。

它不是某一个系统，而是一类卫星定位系统的总称，包括：

- **GPS**：美国
- **北斗 BeiDou**：中国
- **Galileo**：欧盟
- **GLONASS**：俄罗斯
- **QZSS / NavIC** 等区域系统

简单说，GNSS 通过接收多颗卫星的信号，计算接收机在地球上的位置、速度和时间。

在自动驾驶、机器人、无人机、测绘里，GNSS 常和 **IMU/INS**、轮速计、LiDAR SLAM 等融合使用。普通 GNSS 可能是米级精度；如果有 **RTK** 修正，精度可以到厘米级。你这个 DLIO++ 项目里提到的 `/gps_p1/filtered_odom`、RTK gating、Atlas 之类，就是 GNSS/INS 定位链路的一部分。