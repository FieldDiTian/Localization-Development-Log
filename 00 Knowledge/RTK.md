### **RTK（Real-Time Kinematic，实时动态定位）**

RTK 是一种高精度 GNSS 定位技术。

普通 GPS 的误差通常在 1～5 米左右，而 RTK 通过接收基站的差分修正数据，可以将定位精度提升到：

- 水平：1～2 cm
- 垂直：2～5 cm

RTK 输出的主要是：

- 经纬度（Latitude, Longitude）
- 海拔（Altitude）
- 速度（Velocity）

但 RTK 有缺点：

- 进入隧道、树荫、城市峡谷时容易失锁
- 遮挡严重时定位会跳变
- 姿态信息有限