原始输入
/atlas/imu_calibrated
/atlas/pose_filtered
/luminar_front/points
/luminar_left/points
/luminar_right/points

预处理后输入
/gps_p1/imu
/gps_p1/filtered_odom
/gps_p1/filtered_odom_rtk_fixed
/luminar_front/points
/luminar_left/points
/luminar_right/points

原始输入
`/atlas/imu_calibrated`

类型是 `sensor_msgs/msg/Imu`。里面主要是：

```
angular_velocity      角速度，gyro
linear_acceleration   线加速度，accel，带重力
orientation           原消息里的 orientation 字段
header.stamp          原始 arrival stamp
header.frame_id       原来是 pointonenav
```

问题是它的时间戳是到达时间，不是真正采样时间。Atlas driver 会 bursty 地吐数据，IMU 明明大概 100 Hz，但消息可能一批一批到，所以直接拿它做 IMU preintegration 会把 `dt` 搞坏。

`/atlas/pose_filtered`

类型是 `fusion_engine_msgs/msg/Pose`。这是 Atlas 融合后的 GNSS/INS 位姿，里面有：

```
latitude / longitude / altitude    经纬高
rpy.roll / pitch / yaw              ENU 姿态，单位是 degree
velflu.x/y/z                        车体 FLU frame 速度
position_covariance                 位置协方差
rpy_covariance                      姿态协方差
velflu_covariance                   速度协方差
p1_time                             Atlas 自己的真实 time-of-validity
solution_type                       解类型，比如 RTK_FIXED
```

这个 topic 也有 arrival-time stamp 的问题，但它自己带 `p1_time`，所以可以更可靠地重建时间。

`/luminar_front/points`、`/luminar_left/points`、`/luminar_right/points`

类型是 `sensor_msgs/msg/PointCloud2`。就是三个 Luminar 雷达各自的点云：

```
front lidar scan
left lidar scan
right lidar scan
```

lidar信息直接passthrough 