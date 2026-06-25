1. Fixed the IMU burst issue

The current `fusion_engine_driver` still contains an arrival-timestamp path using `this->now()` in the driver code.

`handleFusionMessage()` passes `this->now()` downstream, i.e., the receive/processing time. As a result, the IMU P1 timestamps stored in existing MCAP files are already incorrect. Although `p1_time` exists in the driver’s internal `IMUOutput` payload, the externally published IMU topic uses `sensor_msgs/msg/Imu`, which does not contain a `p1_time` field. Therefore, downstream nodes can no longer access the original payload and can only rely on `header.stamp`.

Fix:

- Decoded the original payload directly from the PCAP file and reconstructed the P1 timestamps.
- Implemented a PCAP replay node to publish PCAP messages, with an adapter subscribing to and converting them.

2. Input topics for GLIM and GICP mapping were inconsistent with the ROS bag topics

Fix:

- Added an adapter node to translate the original ROS bag topics into the topics expected by DLIO.

3. U-turn issue during GLIM reconstruction

https://drive.google.com/file/d/11K9vLuw3IPQcQJMzYOdhJ-tJi_XLOwcI/view?usp=sharing

Updated GLIM online input processing to use a more deterministic live-input pipeline and subscribed to `/luminar_front/points` using reliable QoS.

The direct trigger for the U-turn issue was not corrupted LiDAR timestamps. Instead, the best-effort LiDAR path used during online replay/subscription could drop frames and create gaps. GLIM would then observe:

```text
LiDAR gap
points-IMU time difference
```

These issues could cause parts of the trajectory to degrade suddenly.

Fixed online GLIM result:  
https://drive.google.com/file/d/1geAhkI6NNP-j-tWLmQQ93H08oLUpTF_W/view?usp=drive_link

4. CUDA-based map export caused GPU memory exhaustion

Using CUDA during map export would exhaust the 16 GB GPU memory and crash the system. The final solution was to use CUDA for online mapping and CPU for map export, allowing the export process to utilize the full 64 GB of system memory.

5. Built a baseline using data processed with `prep_bag.py` and developed a frontend for comparison against RTK positions

https://drive.google.com/file/d/1mzVPwL7PjroBqxmkhbHT6A57xxA0BuP-/view?usp=drive_link

6.Current GICP result is still depenging on RTK result. GICP Scan-Matching Acceptance rate is 4 / 3,044 = 0.1314%
Report: https://drive.google.com/file/d/1ZWlw218jEQqQskARcInlIG63XFcwmjRp/view?usp=sharing