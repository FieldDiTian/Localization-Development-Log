driver 组装 ROS2 sensor_msgs/msg/PointCloud2
  ↓
发布到：
    /luminar_front/points
    /luminar_left/points
    /luminar_right/points

topic: /luminar_front/points
type:  sensor_msgs/msg/PointCloud2

PointCloud2 只是一个容器格式，它里面有两层时间：

`PointCloud2.header.stamp`

这是整帧点云只有一个时间。

而 Luminar 这个 PointCloud2.data 里面，每个点还有自己的：

`timestamp 字段`

这个才是每个激光点自己的时间。这个 3D 点对应的那束激光，真正打到物体并被 LiDAR 接收到回波的时间。
也就是传感器完成这次测量的时间。

timestamp 是在 Luminar/PTP 时间轴上的时间
