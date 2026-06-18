完整方案我会按“修 driver 源头，保留旧 bag 兼容层”做。

**核心结论**

- 修复点在 `race_common/fusion_engine_driver`，不是 DLIO/GICP。
- 当前问题是 `/atlas/imu_calibrated.header.stamp` 使用 arrival time；TCP/driver burst 后，下游看到错误 timestep。
- 正确修法不是裸写 `p1_time`，因为 P1 Time 不是 Unix/ROS epoch。应使用 `Pose.p1_time + Pose.gps_time` 建立 `P1 -> ROS/UTC` 映射，再给 IMU stamp。
- 不硬编码 100Hz。当前 bag 是约 100Hz/10ms，但 driver 应使用 IMUOutput 的真实 P1 sample delta。

**Driver 改动**

1. 在 [fusion_dispatch.hpp (line 33)](/home/roar/Documents/race_common/src/external/drivers/fusion_engine_driver/fusion_engine_driver/core/fusion_dispatch.hpp:33) 附近新增 `FusionTimeMapper`：
    
    - 输入 `PoseMessage.p1_time` 和 `PoseMessage.gps_time`。
    - 计算：
        
        ```
        ros_time = gps_time + 315964800 - leap_seconds
        p1_to_ros_offset = ros_time - p1_time
        imu_stamp = imu.p1_time + p1_to_ros_offset
        ```
        
    - `leap_seconds` 优先来自 `GNSSInfo.leap_second`，否则用参数默认值，比如当前数据为 18。
    - 对 invalid P1/GPS time 拒绝更新 mapper，并 throttle warn。
2. 扩展参数：
    
    ```
    timestamp_source: "device_time_mapped"  # arrival | device_time_mapped
    gps_leap_seconds_default: 18
    drop_imu_until_time_mapper_ready: true
    ```
    
    IAC Atlas 配置写到 [atlas.param.yaml (line 1)](/home/roar/Documents/race_common/src/launch/iac_launch/param/gps_param/atlas/atlas.param.yaml:1)。
    
3. 修改 Pose 和 IMU handler：
    
    - Pose handler：收到 `PoseMessage` 时先更新 mapper，然后把 `/atlas/pose_filtered.header.stamp` 改成 GPS/P1 映射后的有效时间。
    - IMU handler：`IMU_OUTPUT` 使用 `mapper.map(imu.p1_time)` 作为 `/atlas/imu_calibrated.header.stamp`。
    - 如果 mapper 未 ready：默认 drop IMU，不发布坏 stamp；日志说明等待 P1->ROS mapper。
    - 加 per-topic monotonic guard：`stamp <= last_stamp` 时 drop 并计数，不允许 0/负 dt 进入下游。
4. 保留 backward compatibility：
    
    - `timestamp_source: "arrival"` 仍保持旧行为，方便对比和回滚。
    - 不把 UDP 当修复；当前 `udp` 路径仍走 `TcpListener`，那是另一个问题。

**PointOneNav Interface 改动**

- 在 [raw.cpp (line 54)](/home/roar/Documents/race_common/src/external/interfaces/pointonenav_interface/src/impl/raw.cpp:54) 修 IMU dt 计算：
    - 第一帧不再用 `this->now()` 初始化后直接算频率。
    - `dt <= 0`、NaN、Inf 直接 reject 或使用安全默认 covariance。
    - `/gps_p1/imu` 继续继承修复后的 `/atlas/imu_calibrated.header.stamp`。
- 当前 [pointonenav_interface.param.yaml (line 3)](/home/roar/Documents/race_common/src/launch/iac_launch/param/gps_param/atlas/pointonenav_interface.param.yaml:3) 写的是 120Hz；实际 bag 是 100Hz。低通滤波器参数应改成 100Hz，或更好：用实测 dt 动态更新滤波器采样频率。

**旧数据处理**

- 已有坏 ROS bag 里的 `sensor_msgs/Imu` 没有 P1 sample time，不能精确恢复。
- 如果有原始 FusionEngine PCAP/InputDataWrapper，可以用修好的 driver 重新 decode，得到正确 IMU stamp。
- 如果只有现有 ROS bag，继续用 `prep_bag` 或 DLIO live adapter 的 retime，当作 legacy fallback。

**测试验收**

- Unit tests：
    - 模拟 100Hz P1 IMU，但 arrival 成 burst；输出 stamp dt 应保持约 10ms。
    - 模拟非 100Hz P1 IMU；输出跟随 P1 delta，证明无硬编码。
    - GPS/P1 mapper 测试：`gps_time + epoch - leap - p1_time` 得到稳定 offset。
    - mapper 未 ready、invalid timestamp、out-of-order timestamp 都有明确 drop/warn。
- Build：
    
    ```
    colcon build --packages-select fusion_engine_driver pointonenav_interface iac_launch
    colcon test --packages-select fusion_engine_driver pointonenav_interface
    ```
    
- Live test：
    - 启动 `ros2 launch iac_launch atlas.launch.py`。
    - 录 5-10 分钟 `/atlas/imu_calibrated`、`/atlas/pose_filtered`、`/gps_p1/imu`、`/gps_p1/filtered_odom`、Luminar topics。
    - 验收：IMU stamp 单调，median dt 约 10ms，near-zero burst gaps 消失，无 `dt<=0`。
- DLIO/GICP test：
    - 用新 driver 录的新 bag，**不使用 prep_bag IMU retime**。
    - 跑 GLIM mapping 和 GICP localization。
    - GICP odom roughly IMU-rate；GLIM/GICP 不再因 IMU timestep burst 退化。

这个方案的关键升级是：**用 Pose 的 GPS/P1 时间建立真实时间映射，而不是只靠 arrival lower-envelope 猜 offset**。arrival envelope 只能作为没有 GPS time 时的 fallback，不能作为主修复。