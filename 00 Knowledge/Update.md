Readme 
GLIM(慢慢开车，收集数据，制作地图) 数据不够就慢慢开 wait for right food
RTK?

Dlio
Fixed IMU
Turn off IMU compencation in our code
IMU gained diffenertly  banned Z 高度
GICP在扫描地图的时候很慢，但是比赛的时候需要把限制放开

雷达 点云 时间差 
8bit delta t point cloud


**Why the code "knows" about sensors**

The dependency isn't really about sensors as physical units — it's about **per-point timestamp encoding**.<font color="#ff0000"> To deskew a LiDAR scan, you need to know when each individual point was measured so you can interpolate IMU pose at that exact moment.</font> ROS 2's sensor_msgs/PointCloud2 is a flexible container — fields are self-describing by name, byte offset, and datatype — but the **semantics** of any "time" field (relative vs. absolute, units, anchored to what) are NOT standardized. Vendors picked different conventions:

**Vendor**

**Field name**

**Datatype**

**Units**

**Origin**

Oustert

UINT32

nanoseconds

start of sweep

Velodyne

time

FLOAT32

seconds

start of sweep

Hesai

timestamp

FLOAT64

seconds

Unix epoch (absolute)

Livox

timestamp

FLOAT64

ns × 1e-9 (so seconds, but reached by scaling ns)

Unix epoch (absolute)

Seyond Robin W (coord_mode=3)

time

FLOAT32

seconds

start of sweep

**(removed:)** Luminar

timestamp

FLOAT64 declared, **raw uint64 bits** in practice

hardware clock

not Unix


rebuild glim
验证地图
point one knife