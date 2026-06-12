The configs target an AV-24 Cybertruck instrumented with:

- **3× Luminar Iris LiDAR** — `luminar_front` is the primary sensor; `luminar_left` and `luminar_right` are concatenated into the primary cloud via URDF transforms.
- **Point One Atlas (LG69T) INS** publishing IMU on <font color="#ff0000">`/gps_p1/imu`</font> (`imu_calibrated`: sensor-level bias/scale/misalignment removed by FusionEngine firmware, gravity present, no fused orientation) and odometry on `/gps_p1/filtered_odom`. Atlas firmware projects both the IMU and the INS pose to the primary antenna phase centre, so the URDF link `gps_antenna_top` is used as both `base_frame` and `imu_frame` in the localization config. RTK quality is gated on the Atlas-reported pose covariance.
- **RTK GPS** — the FusionEngine INS itself; no separate raw RTK topic is needed for localization.
- Optional camera (used only by extension modules).

All sensor extrinsics are resolved at runtime from [`av24.urdf`](https://file+.vscode-resource.vscode-cdn.net/home/roar/Documents/DLIO_plusplus/av24.urdf); the `*_frame` strings in the configs are URDF link names, not free-form labels.

Current localization scope is intentionally single-source Point One Atlas. Earlier project notes mention NovAtel and VectorNav GNSS integration, but `gicp_localization` no longer subscribes to either; adding them back is future work and needs a fresh source-selection and fix-status design rather than a topic remap.