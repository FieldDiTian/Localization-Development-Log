我们要证明两件事：

1. **P1 -> ROS/Unix 映射公式正确**
2. **映射后的 IMU stamp 对下游融合是正确的**

核心公式是：

```
gps_unix = pose.gps_time + 315964800 - leap_seconds
offset   = gps_unix - pose.p1_time
imu_ros_stamp = imu.p1_time + offset
```

为什么这个可验证：`pose.gps_time` 是独立的绝对 GNSS 时间，`pose.p1_time` 是设备内部时间。两者在同一条 Pose 消息里给出，所以它们天然定义了 P1 到真实时间轴的映射。

**第一层：字段级验证**  
用 `/atlas/pose_filtered` 全量统计：

- `gps_unix = gps_time + 315964800 - 18`
- `latency = arrival/header_stamp - gps_unix`
- 合格特征：
    - `latency >= 0`，因为 arrival/header 只能晚于真实测量时间；
    - latency 应该是几十毫秒级或可解释的正延迟；
    - 不能出现大量负 latency；
    - `offset = gps_unix - p1_time` 应平滑，不能突然跳秒。

我们之前抽到一个样本：

```
header       1779816149.935134888
p1_time          20868.944703422
gps_time    1463851367.880000114
gps_unix    1779816149.880000114   # gps + 315964800 - 18
latency           0.055134774 s
```

这很合理：真实测量时间比 ROS arrival 早约 55 ms。所以映射方向是对的。

**第二层：IMU timestep 验证**  
修复后录新 bag，看 `/atlas/imu_calibrated` 和 `/gps_p1/imu`：

合格指标：

```
dt > 0 for all samples
median dt ≈ 0.010 s    # 当前 Putnam 数据约 100 Hz
p95/p99 没有 burst 型 near-zero gap
没有一串 0~1 ms gap 后接一个几十/几百 ms gap
```

注意：不是要求永远精确 10 ms，而是要求 stamp 反映设备 sample time，而不是 driver flush time。

**第三层：arrival 反证**  
同一段数据同时记录：

```
message arrival/log_time
mapped header.stamp
```

检查：

```
arrival - mapped_stamp
```

它应该是非负 transport/processing latency。  
如果 burst 发生，arrival 会挤在一起，但 mapped stamp 不应该挤在一起。

这能直接证明我们没有再把 burst 写进 `header.stamp`。

**第四层：下游敏感性测试**  
用新 driver 录的新 bag 跑 GLIM/GICP，不用 `prep_bag` 的 IMU retime。

然后做一个 time-offset sweep：

```
IMU stamp offset = -50ms, -25ms, -10ms, 0ms, +10ms, +25ms, +50ms
```

合格特征：

- `0ms` 附近 GLIM/GICP 最稳定；
- 人为偏移后轨迹、fitness、GNSS residual 变差；
- 如果 `+/- 某个偏移` 明显更好，说明映射还有固定 latency 没处理干净。

**第五层：端到端验收**  
最终用新 bag 验收：

- `/atlas/imu_calibrated`：单调、约 100 Hz、无 burst stamp；
- `/gps_p1/imu`：继承同样正确 stamp；
- `/gps_p1/filtered_odom`：和 Pose 的 GPS/P1 映射一致；
- GLIM mapping 能直接吃 raw/live topic；
- GICP localization 开 RViz 能跑，odom roughly IMU-rate；
- 不依赖 `prep_bag` 修 IMU 时间。

如果这些都过，我们就能说：**时间映射正确，且 IMU burst 不再进入 DLIO/GICP 的 timestep。**