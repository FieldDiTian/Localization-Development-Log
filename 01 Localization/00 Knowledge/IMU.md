- **Point One Atlas (LG69T) INS** publishing IMU on `/gps_p1/imu` (`imu_calibrated`: sensor-level bias/scale/misalignment removed by FusionEngine firmware, gravity present, no fused orientation) and odometry on `/gps_p1/filtered_odom`. Atlas firmware projects both the IMU and the INS pose to the primary antenna phase centre, so the URDF link `gps_antenna_top` is used as both `base_frame` and `imu_frame` in the localization config. RTK quality is gated on the Atlas-reported pose covariance.

| **设备**    | **IMU状态**              |
| --------- | ---------------------- |
| NovAtel   | raw IMU （现在在用）         |
| Atlas(P1) | calibrated IMU （需要迁移到） |
| VectorNav | calibrated IMU         |
- 静止状态下仍能观察到残余信号。
- 当前 IMU 标定并未完全收敛。
- 当前统一使用 P1 IMU**

正确修法优先级是：
1. **先把 IMU 的 `header.stamp` 改成 P1/设备采样时间**，不要用 `this->now()`。
2. 如果下游真的要求均匀 arrival，再加一个按设备时间或 nominal 120 Hz 的 pacing/republisher queue。
3. 同时检查 P1 端或 `p1_runner` 是否在 TCP relay 时批量 flush。


