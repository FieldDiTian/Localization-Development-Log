除了 IMU，live 主要还有这几类阻碍：

1. **`/atlas/pose_filtered` 还没人在线转成 `/gps_p1/filtered_odom`**  
    当前 GLIM/GICP 不直接吃 FusionEngine Pose。`prep_bag.py` 做了 LLA→UTM、`p1_time`→ROS time、ENU rpy→quaternion、covariance/twist 拷贝，并输出 `/gps_p1/filtered_odom` 和 `/gps_p1/filtered_odom_rtk_fixed`。live adapter 也必须做这层转换。见 [prep_bag.py (line 32)](/home/roar/Documents/DLIO_plusplus/scripts/prep_bag.py:32)、[prep_bag.py (line 647)](/home/roar/Documents/DLIO_plusplus/scripts/prep_bag.py:647)。
    
2. **P1 pose 的时间轴也要在线映射，不只是 IMU**  
    `pose_filtered.header.stamp` 也是 arrival time，`prep_bag.py` 用 `p1_time + lower-envelope offset` 重建真实有效时间。live 里如果只修 IMU，不修 pose，GNSS/INS odom 仍然和 LiDAR/IMU 时间轴错。见 [prep_bag.py (line 43)](/home/roar/Documents/DLIO_plusplus/scripts/prep_bag.py:43)、[prep_bag.py (line 150)](/home/roar/Documents/DLIO_plusplus/scripts/prep_bag.py:150)。
    
3. **Luminar 点内 timestamp epoch 仍要处理，尤其 GLIM**  
    LiDAR scan header 是干净的 20 Hz，但点内 `timestamp` 是 Luminar/PTP 轴。`prep_bag.py` 当前把点内时间整体平移到 `header.stamp` 同一 epoch。GICP 现在有 Luminar 特例，会用 `ts - min_ts` anchored at header，所以它相对更抗这个问题；GLIM 路径仍然需要保证不要让点内 PTP epoch 把 frame time 搞飞。见 [prep_bag.py (line 51)](/home/roar/Documents/DLIO_plusplus/scripts/prep_bag.py:51)、[localization.cc (line 2423)](/home/roar/Documents/DLIO_plusplus/gicp_localization/src/localization.cc:2423)。
    
4. **RTK-FIXED gate 要在线存在**  
    GLIM 当前默认吃 `/gps_p1/filtered_odom_rtk_fixed`，不是裸 `/gps_p1/filtered_odom`。`prep_bag.py` 用 `solution_type == RTK_FIXED` 加 covariance 阈值过滤；live 里也要有同等 gate，否则 map 初始锚点可能来自 degraded GNSS。见 [prep_bag.py (line 695)](/home/roar/Documents/DLIO_plusplus/scripts/prep_bag.py:695)、[config_ros.json (line 91)](/home/roar/Documents/DLIO_plusplus/GLIM/glim/config/config_ros.json:91)。
    
5. **UTM origin / map frame 生命周期要明确**  
    mapping 阶段要固定 `utm_origin.txt`，不同 session 要复用同一个 origin。localization 阶段 GICP 默认要 `/gps_p1/filtered_odom_map`，这需要 GLIM dump 的 `T_world_utm.txt` 把 UTM odom 转到 map frame。现在是 `utm_to_map_odom.py` 做的；live C++ 路径要么继续跑这个 node，要么把它并进 adapter。见 [prep_bag.py (line 654)](/home/roar/Documents/DLIO_plusplus/scripts/prep_bag.py:654)、[utm_to_map_odom.py (line 1)](/home/roar/Documents/DLIO_plusplus/gicp_localization/scripts/utm_to_map_odom.py:1)。
    
6. **repo 里没有 Point One / Luminar live driver，只能做 driver 后的 adapter**  
    本仓库现在的消费者默认是 `/gps_p1/*`。raw driver topic 是 `/atlas/*`，而 FusionEngine message 类型和 live driver 本身不在这个 repo 里。C++ live 实现需要依赖 race_common 或对应 `fusion_engine_msgs`，否则连 `/atlas/pose_filtered` 的类型都没法编译订阅。
    
7. **GLIM 当前 mapping 默认是 INS-driven，RTK/INS 覆盖不足会跳帧/暂停**  
    `config_odometry_ins.json` 写得很清楚：LiDAR scan 要被 INS buffer 覆盖才插入地图；没 INS/RTK 覆盖时不会提交 map points。live 开车前必须保证 Atlas aligned + RTK gate 已经放行，或者改 GLIM odometry 策略。见 [config_odometry_ins.json (line 7)](/home/roar/Documents/DLIO_plusplus/GLIM/glim/config/config_odometry_ins.json:7)。
    

所以最小 live 方案不是“只写 IMU retimer”，而是一个 C++ `dlio_input_adapter`：

`/atlas/imu_calibrated` → `/gps_p1/imu`  
`/atlas/pose_filtered` → `/gps_p1/filtered_odom` + `/gps_p1/filtered_odom_rtk_fixed`  
`/luminar_*/points` → normalized `/luminar_*/points`  
可选：`/gps_p1/filtered_odom` + `T_world_utm.txt` → `/gps_p1/filtered_odom_map`

这样 GLIM/GICP 的现有接口基本不用改，rosbag 只是模拟 live driver 的输入源。