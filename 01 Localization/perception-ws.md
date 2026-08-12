```mermaid
flowchart LR
    A["原始 PCAP"] --> B["decode<br/>时间合并 ROS bag"]

    B --> C["标准 GLIM<br/>LiDAR + IMU"]
    C --> D["/tmp/dump<br/>GLIM world"]
    D --> E["export_map.py"]
    E --> M1["PCD<br/>GLIM 局部世界系"]

    B --> F["build_georeferenced_map.py<br/>RTK/INS 直接投影"]
    F --> M2["PCD + JSON<br/>局部 ENU"]

    M1 --> G["GICP localizer"]
    M2 --> G
    L["/luminar_front/points"] --> G
    I["/gnss 或 CV 预测"] --> G
    G --> O["map → pointonenav<br/>/gicp_localizer/odom"]
```
## 建图路线 A：标准 GLIM LiDAR-IMU SLAM


## 建图路线 B：RTK/INS 直接投影
第一条有效 Atlas fix 被选作 ENU 原点：

```
anchor = (lat₀, lon₀, alt₀)
```

每条 LLA 先转换成 ECEF：

```
p_ecef = f_WGS84(lat, lon, alt)
```

再转换成局部 ENU：

```
p_enu = R_enu_ecef(anchor) · (p_ecef - p_ecef_anchor)
```

因此此时：

```
x = East
y = North
z = Up
origin = 第一条有效 GNSS fix
```

这是当前仓库中真正明确建立 ENU 地图的路径。