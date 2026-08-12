整理后的关键结论：

| 项目                   | 结论                                                         |
| -------------------- | ---------------------------------------------------------- |
| 点内时间来源               | Luminar raw 协议的 `packet_ptp_seconds + ray_ptp_nanoseconds` |
| 点内时间格式               | `uint8[8]`，little-endian `uint64 ns`                       |
| 是否是 double           | 不是                                                         |
| 是否是 ROS Unix epoch 秒 | 不是                                                         |
| 是否被驱动改写              | 没有，PCAP 与 MCAP 逐字节一致                                       |
| `header.stamp`       | ROS/bag receive/write time                                 |
| deskew 可用性           | 可用，且是 ray-level 时间                                         |
| deskew 注意点           | 点序不保证时间单调，必须读每个点自己的 timestamp                              |
| 下游适配方式               | `uint8[8] -> uint64 ns -> relative seconds`                |

FAST-LIO/LIO-SAM 适配时，核心就是不要按 `double timestamp` 或 `float time` 读，而要按：

```cpp
uint64_t timestamp_ns;
std::memcpy(&timestamp_ns, point_data + timestamp_offset, sizeof(uint64_t));
// little-endian host 通常可直接用；若需跨平台，显式按字节组装
```

然后转成相对帧时间：

```cpp
double point_time_sec =
    static_cast<double>(timestamp_ns - frame_min_timestamp_ns) * 1e-9;
```

不能用：

```cpp
cloud.header.stamp
```

当作点云采样时钟域；它是 bag/ROS 时间，不是 Luminar 点时间。


