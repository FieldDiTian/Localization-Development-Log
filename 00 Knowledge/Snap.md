`snap` 在这里不是算法名，意思是 **瞬间拉回 / 直接重置**。

在这个 GICP 代码语境里，`GT recovery snap` 指：

当 GICP 连续失败或被拒绝后，系统不再继续相信当前 GICP/IMU 推出来的位置，而是直接把当前定位状态改成 GT/Atlas odom 给出的位姿。

也就是从：

```
current_pose = GICP/IMU 当前估计
```

瞬间改成：

```
current_pose = 当前 scan 时间对应的 GT pose
```