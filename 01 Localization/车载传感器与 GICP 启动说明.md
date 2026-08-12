### Pane 1：Atlas

```
cd /home/roar/Documents/race_common
source /opt/ros/jazzy/setup.bash
source install/setup.bash
export VEHICLE_NAME=IAC_CAR

taskset -c 6 ros2 launch iac_launch atlas.launch.py use_sim_time:=false
```

### Pane 2：VectorNav

```
cd /home/roar/Documents/race_common
source /opt/ros/jazzy/setup.bash
source install/setup.bash
export VEHICLE_NAME=IAC_CAR

taskset -c 6 ros2 launch iac_launch vectornav.launch.py use_sim_time:=false
```

### Pane 3：NovAtel-A

```
cd /home/roar/Documents/race_common
source /opt/ros/jazzy/setup.bash
source install/setup.bash
export VEHICLE_NAME=IAC_CAR

taskset -c 6 ros2 launch iac_launch oem7_net_a.launch.py use_sim_time:=false
```

### Pane 4：Iris LiDAR

```
cd /home/roar/Documents/race_common
source /opt/ros/jazzy/setup.bash
source install/setup.bash
export VEHICLE_NAME=IAC_CAR

taskset -c 3 ros2 launch iac_launch iris.launch.py
```

### Pane 5：Vehicle Kinematic State

仅在 `/vehicle/state` 没有被 autonomy stack 发布时运行：

```
cd /home/roar/Documents/race_common
source /opt/ros/jazzy/setup.bash
source install/setup.bash
export VEHICLE_NAME=IAC_CAR

taskset -c 18 ros2 launch vehicle_kinematic_state \
  vehicle_kinematic_state.launch.py \
  vehicle_name:=IAC_CAR \
  launch_vks:=true \
  use_sim_time:=false
```

---

## 4. 在线 Driver 主要 Topics

|来源|Topic|类型 / 用途|
|---|---|---|
|Iris|`/luminar_front/points`|`sensor_msgs/PointCloud2`，GICP 直接消费|
|Iris|`/luminar_left/points`|左侧 LiDAR 点云|
|Iris|`/luminar_right/points`|右侧 LiDAR 点云|
|Atlas|`/atlas/pose_filtered`|`fusion_engine_msgs/Pose`|
|Atlas|`/atlas/imu_calibrated`|`sensor_msgs/Imu`|
|Atlas interface|`/gps_p1/filtered_odom`|Atlas 融合里程计|
|Atlas interface|`/gps_p1/filtered_gps`|Atlas 融合 GPS|
|Atlas interface|`/gps_p1/imu`|Atlas 标准化 IMU|
|VectorNav|`/vectornav/raw/gps`|`vectornav_msgs/GpsGroup`，GPS1|
|VectorNav|`/vectornav/raw/gps2`|`vectornav_msgs/GpsGroup`，GPS2|
|VectorNav|`/vectornav/raw/imu`|VectorNav 原始 IMU group|
|VectorNav interface|`/gps_nav/filtered_odom`|VectorNav 融合里程计|
|VectorNav interface|`/gps_nav/imu`|VectorNav 标准化 IMU|
|NovAtel-A|`/novatel_a/bestgnsspos`|`BESTGNSSPOS`，RTK 位置|
|NovAtel-A|`/novatel_a/bestgnssvel`|`BESTGNSSVEL`|
|NovAtel-A|`/novatel_a/heading2`|`HEADING2`|
|NovAtel-A|`/novatel_a/inspva`|Vendor INS，仅用于监控|
|NovAtel interface|`/gps_na/imu`|`sensor_msgs/Imu`，车载 GICP 应消费|
|NovAtel interface|`/gps_na/odom`|GNSS 里程计|
|NovAtel interface|`/gps_na/filtered_odom`|NovAtel INS 里程计|
|VKS|`/vehicle/state`|`race_msgs/VehicleKinematicState`，速度输入|

---

## 5. `cea9a41c` 的 GICP 输入接口

该 commit 中，GICP 实际订阅以下接口：

|      |                         |                                           |
| ---- | ----------------------- | ----------------------------------------- |
| 输入   | 默认或期望接口                 | 类型                                        |
| 前向点云 | `/luminar_front/points` | `sensor_msgs/PointCloud2`                 |
| GNSS | `/gnss`                 | `geometry_msgs/PoseWithCovarianceStamped` |
| IMU  | 由 `imu_topic` 参数指定      | `sensor_msgs/Imu`                         |
| 车辆速度 | 由 `speed_topic` 参数指定，可选 | `race_msgs/VehicleKinematicState`         |

### 5.1 IMU Topic 必须覆盖

默认 Laguna 参数中的 IMU topic 是：

```
/novatel_b/imu_corrected
```

但这个 topic 不是当前车载 driver 的实际输出。

上车运行时必须覆盖为：

```
-p imu_topic:=/gps_na/imu
```

### 5.2 车辆速度 Topic

需要使用原车速度时，增加：

```
-p speed_topic:=/vehicle/state
```

### 5.3 参数组合

至少需要加入：

```
-p imu_topic:=/gps_na/imu
```

同时使用车辆速度时加入：

```
-p imu_topic:=/gps_na/imu \
-p speed_topic:=/vehicle/state
```

---

## 6. GNSS 接口缺口

`cea9a41c` 中没有任何节点负责将 Atlas、VectorNav 或 NovAtel 的输出转换为 GICP 要求的：

```
/gnss
```

GICP 要求的 `/gnss` 类型是：

```
geometry_msgs/PoseWithCovarianceStamped
```


---

## 8. 上车前 Topic 检查

### 检查 LiDAR

```
ros2 topic hz /luminar_front/points
ros2 topic hz /luminar_left/points
ros2 topic hz /luminar_right/points
```

### 检查 NovAtel IMU

```
ros2 topic hz /gps_na/imu
ros2 topic info /gps_na/imu
```

### 检查车辆速度

```
ros2 topic hz /vehicle/state
ros2 topic info /vehicle/state
```

### 检查 `/gnss`

```
ros2 topic info /gnss
```

如果返回 topic 不存在，说明当前没有可供 GICP 使用的 GNSS seed/reseed 输入。

### 查看当前全部相关 Topics

```
ros2 topic list | grep -E "luminar|atlas|gps_p1|vectornav|gps_nav|novatel|gps_na|vehicle|gnss"
```
