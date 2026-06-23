1. 修复了IMU burst的问题
现在的`fusion_engine_driver`
驱动代码里仍有 `this->now()` 的 arrival stamp 路径
[fusion_engine_node.cpp (line 17)](/home/roar/Documents/race_common/src/external/drivers/fusion_engine_driver/fusion_engine_driver/core/fusion_engine_node.cpp:17)
`handleFusionMessage()` 传下去的是 `this->now()`，也就是接收/处理时刻，所以当前已有的mcap受到的IMU p1 time已经是错误信息了，并且`p1_time` 在 driver 内部的 `IMUOutput` payload 里存在，但当前对外发布的 IMU topic 类型是 `sensor_msgs/msg/Imu`，这个类型没有 `p1_time` 字段。所以下游节点订阅到消息后已经看不到原始 payload，只能看 `header.stamp`。

修复方法：从pcap中decode了原始payload，复原了p1 time
做了一个pcap replay node发布pcap消息，adapter订阅

2. glim 和gicp建图时的输入与rosbag发布的topics不一致
修复方法：加了一个adapter node,把原始rosbag topics转译为dlio能消费的话题

3.GLIM重建过程中出现uturn
https://drive.google.com/file/d/11K9vLuw3IPQcQJMzYOdhJ-tJi_XLOwcI/view?usp=sharing
GLIM 在线输入改成更确定的 live 输入处理，并用可靠 QoS 消费 `/luminar_front/points`。

之前 u-turn 的直接触发点不是 LiDAR 时间戳本身坏，而是 online replay/subscription 的 best-effort LiDAR 路径会丢帧/形成 gap，GLIM 看到：

```
LiDAR gap
points-IMU time difference
```

这些会让轨迹在某些段突然坏掉。
修复后的online glim:
https://drive.google.com/file/d/1geAhkI6NNP-j-tWLmQQ93H08oLUpTF_W/view?usp=drive_link

4.地图导出时用CUDA会导致显存(16GB)爆炸直接死机，最终方案用了CUDA的在线建图和CPU的导出，这样可以用上全部的内存（64GB）


5.用prep_bag.py后处理的数据建了一个baseline，并做了前端，可以对比与rtk位置的差异
https://drive.google.com/file/d/1mzVPwL7PjroBqxmkhbHT6A57xxA0BuP-/view?usp=drive_link