LiDAR-inertial SLAM. 
Builds a 3D map from IMU + multi-LiDAR + GNSS.

**GLIM dump**  
是 GLIM 建图结束后保存的“工程状态包”，不是给 GICP 直接吃的地图文件。

里面通常有：

- `graph.bin` / `graph.txt`：位姿图
- `values.bin`：优化变量
- `traj_lidar.txt` / `odom_lidar.txt`：轨迹
- `000000/`, `000001/` 这些 submap 目录
- 每个 submap 里是 GLIM 自己的 compact 点云数据，比如 `points_compact.bin`

它的作用是：可以重新加载、检查、优化、导出地图。

**PCD**  
是标准点云地图文件，扩展名 `.pcd`。这是 GICP 真正需要的地图格式。

比如：

```
putnam_run3_map.pcd
```

里面是点云点：

```
x y z intensity
```