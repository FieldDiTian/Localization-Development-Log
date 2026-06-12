/gps_p1/imu (imu_calibrated)
/gps_p1/filtered_odom

**Point One Atlas (LG69T) INS** publishing IMU on `/gps_p1/imu` (`imu_calibrated`: sensor-level bias/scale/misalignment removed by FusionEngine firmware, gravity present, no fused orientation) and odometry on `/gps_p1/filtered_odom`. Atlas firmware projects both the IMU and the INS pose to the primary antenna phase centre, so the URDF link `gps_antenna_top` is used as both `base_frame` and `imu_frame` in the localization config. RTK quality is gated on the Atlas-reported pose covariance.