## 四个轮速器
wheels encoder
## 两个IMU

## GPS novatail（速度、位置） 里有一个IMU GPS point one knife
信号不稳定

Lidar

第一版 三个sensor一起 得到xyz GPS稳定时速度最准
需要XYZ和orientation

之前一两秒内通过Wheels撑一下，时间长误差很大

第二版 Lidar  Lidar GPS IMU
当GPS没signal，用lidar做Localization
Lidar理论上没有GPS准
赛道标注？
Multi-fri版本，在GPS实效的点位用Lidar

Odometry

Todo
把盘找齐
数据找齐
本地开发与车上对齐
接rosbag和车上对齐
网线接笔记本直接能跑
stack 可以Ping到车里 ssh 
网线能连车
Sensor ip 的document
setup lidar     *lunch calibration*
lunch full stack
rmw和车上对齐
RMW fastDDS 消息类型

Native install(advanced)
TTL_folder=
target trajetory line
Map_name

