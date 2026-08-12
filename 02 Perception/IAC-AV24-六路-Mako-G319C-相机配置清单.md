---
title: "IAC AV-24 六路 Allied Vision Mako G-319C 相机配置清单"
vehicle: "IAC AV-24"
camera_count: 6
camera_model: "Allied Vision Mako G-319C"
status: "公开资料与公开仓库核对版"
verified_at: "2026-08-10"
race_common_branch: "stable"
race_common_commit: "64735fd3b6cacac3a577221b8fa85eee1249faa2"
race_metadata_branch: "art-jazzy"
race_metadata_commit: "36e8c7bf215be739179fb2e20794ee80ced11637"
tags:
  - IAC/AV-24
  - camera
  - Allied-Vision
  - Mako-G-319C
  - race_common
  - race_metadata
  - calibration
---

# IAC AV-24 六路相机配置清单

> [!summary] 结论速览
> - **硬件/厂家规格**：Allied Vision Mako G-319C；Sony IMX265（C 版技术手册写作 Sony IMX265LQR Exmor）；2064×1544、3.2 MP、3.45 µm 像素、Pregius global shutter、GigE/PoE、全分辨率约 37.6 fps。
> - **实际默认运行模式**：四路环视相机输出 1056×1056，sensor ROI 为 OffsetX=504、OffsetY=244；两路 center 相机通过 4×4 decimation 输出 516×384，偏移为 0。
> - **实际默认帧率**：front/rear/left/right/front_left_center=20 Hz；front_right_center=10 Hz。因此不能把六路概括为统一 20 Hz。
> - **内参**：六个 ROS camera_info_url 指向 race_common/stable 中的固定 YAML；四路使用 equidistant，两路 center 使用 plumb_bob。
> - **外参**：来自 race_metadata/art-jazzy/urdf/iac_car/av24.urdf，相对于 rear_axle_middle；该 link 与 base_link 的 URDF 固定变换为零，因此本清单按 base_link 报告。
> - **镜头**：公开资料只确认 Edmund Optics 是镜头供应方/技术赞助方，公开专利还说六路使用不同镜头；没有找到六路各自的 Edmund Optics 型号、焦距、光圈或序列号。

## 1. 来源分级与核对范围

| 标记 | 含义 |
|---|---|
| [HW] | Allied Vision 官方硬件/厂家规格；不是 AV-24 当前运行值。 |
| [IAC] | IAC 官方网站或公开 IAC/AV-24 专利资料。 |
| [RC] | airacingtech/race_common stable 分支中的实际软件配置、启动链或 calibration YAML。 |
| [RM] | airacingtech/race_metadata art-jazzy 分支中的 AV-24 URDF 外参。 |
| [CALC] | 仅根据 URDF 固定变换计算出的旋转矩阵，不是新的标定结果。 |
| 未确认 | 在本次公开资料范围内没有足够证据，不做推测。 |

本清单核对的是以下固定提交，而不是“仓库当前最新内容”的模糊快照：

- race_common/stable：[64735fd3](https://github.com/airacingtech/race_common/commit/64735fd3b6cacac3a577221b8fa85eee1249faa2)
- race_metadata/art-jazzy：[36e8c7bf](https://github.com/airacingtech/race_metadata/commit/36e8c7bf215be739179fb2e20794ee80ced11637)

## 2. AV-24 与六路相机的公开硬件信息

### 2.1 AV-24 层面的公开信息

| 项目 | 可确认内容 | 来源/结论 |
|---|---|---|
| 车辆 | IAC-built AV-24，基于改装的 Dallara Indy NXT 底盘 | [IAC] [IAC Racecar 页面](https://www.indyautonomouschallenge.com/racecar) |
| 相机数量 | 6 台 Allied Vision camera devices | [IAC] [公开专利 WO2025043253A1，段落 188–190](https://patents.google.com/patent/WO2025043253A1/en) |
| 镜头供应方 | Edmund Optics；公开专利描述为不同镜头组 | [IAC] [IAC Sponsors](https://www.indyautonomouschallenge.com/sponsors)、[公开专利](https://patents.google.com/patent/WO2025043253A1/en) |
| IAC Racecar 页面是否列出 Mako 型号 | 当前页面的文本没有列出具体相机型号、焦距、内参或外参 | 未确认/未公开 |
| 本清单采用的具体型号 | Mako G-319C；stable 的 rear YAML 注释明确写出该型号，用户提供的 AV-24 硬件描述也一致；公开专利只写 Allied Vision，未写 Mako 型号 | [RC] + 外部硬件描述；不把它误标成 IAC Racecar 页面字段 |

### 2.2 Allied Vision Mako G-319 / G-319C 厂家规格

| 参数 | 厂家/公开值 | 标记与说明 |
|---|---:|---|
| 系列/型号 | Mako G-319；彩色型号 G-319C | [HW] 官方产品页/技术手册 |
| C 版 sensor model | Sony IMX265LQR Exmor | [HW] 官方 Mako User Guide；数据表简写为 Sony IMX265 |
| 颜色 | Color | [HW] G-319C 产品/经销商页面 |
| sensor 类型 | CMOS；Pregius global shutter | [HW] |
| sensor format | Type 1/1.8；约 8.9 mm 对角线 | [HW] |
| 有效分辨率 | 2064 (H) × 1544 (V)；3.2 MP | [HW] Allied Vision 官方数据表/技术手册 |
| 像素尺寸 | 3.45 µm × 3.45 µm | [HW] |
| 全分辨率最大帧率 | 37.6 fps；数据表摘要四舍五入写 37 fps | [HW] |
| Burst 帧率 | 39.5 fps burst mode | [HW] 官方技术手册 |
| 光谱范围 | 300–1100 nm | [HW]；若其他资料写成 m，应视为单位错误 |
| 最大图像位深 | 12 bit | [HW] |
| 颜色 RAW 格式 | BayerRG8、BayerRG12、BayerRG12Packed | [HW]；仓库实际选 BayerRG8 |
| 其他输出格式 | YUV411Packed、YUV422Packed、YUV444Packed、RGB8Packed、BGR8Packed | [HW] |
| 数字接口 | IEEE 802.3 1000BASE-T GigE；PoE | [HW] |
| 供电 | 10.8–26.4 VDC AUX 或 PoE；不同官方版本对 PoE 标准写法有 802.3af/802.3at Type 1 差异 | [HW]；以具体相机/电源实物为准 |
| 功耗 | 官方产品页：约 2.3 W（12 VDC）、2.6 W（PoE） | [HW] |
| 硬件同步 | GPIO hardware trigger、software trigger 或 IEEE 1588 PTP | [HW] |
| GPIO | 1 个 opto-isolated input、3 个 outputs | [HW] |
| 曝光范围 | 16 µs–85.89 s，1 µs increments | [HW] 官方技术手册 |
| 增益范围 | 0–40 dB，0.1 dB increments | [HW] |
| Binning 能力 | Horizontal 1–4 pixels；Vertical 1–4 rows | [HW] |
| Decimation 能力 | Horizontal/Vertical factor 1、2、4、8 | [HW] |
| 图像缓存 | 64 MB | [HW] |
| 典型动态范围 | 72.7 dB | [HW]；典型成像性能，不是每台实测保证值 |
| 工作温度 | Housing +5–+45 °C | [HW] |
| 重量 | 80 g（C-Mount 口径） | [HW] |
| 尺寸 | 官方产品页约 61×29×29 mm；数据表为 60.5×29.2×29.2 mm（含 connectors） | [HW] 官方版本/测量口径不同 |
| 镜头接口 | C-Mount、CS-Mount 可选 | [HW] |

> [!warning] 厂家参数和运行参数不是一回事
> 2064×1544 @ 37.6 fps 是相机能力上限/规格，不是 AV-24 默认 ROS 图像流。AV-24 默认配置通过 ROI 或 decimation 把输出改成 1056×1056 或 516×384，并把采集帧率设置为 20/10 Hz。

### 2.3 Edmund Optics 镜头：已知与未知

| 项目 | 结论 |
|---|---|
| 供应/合作关系 | IAC 官方 sponsors 页面把 Edmund Optics 列为 Lens Technology Sponsor；公开专利写六路 Allied Vision 相机配不同的 Edmund Optics lens set。 |
| 六路镜头型号 | 未确认。公开 race_common、race_metadata 中未找到镜头 part number。 |
| 焦距、光圈、视场角 | 未确认。不能从 Mako G-319C 型号或内参反推唯一镜头型号。 |
| focus/zoom 状态 | 未确认。仓库没有公开物理 focus 记录。 |
| 镜头序列号与相机序列号映射 | 未确认；不能把 IP 或 frame 名称当成序列号。 |

## 3. 软件启动链与“实际使用”定义

在 stable 提交中，车载 tmux 配置调用的是：

~~~
ros2 launch isaac_launch vimba.launch.py
        ↓
isaac_launch/components/perception_components/vimba_components.py
        ↓
VIMBA_CAMERA_SOURCES['IAC_CAR']
        ↓
iac_launch/param/cameras_param/vimba_*.param.yaml
iac_launch/param/cameras_param/cam_*_calib.yaml
~~~

VIMBA_CAMERA_SOURCES['IAC_CAR'] 明确列出六路 camera、六个默认参数 YAML 和六个 calibration YAML。以下表格的“实际运行配置”指这条默认启动链会传给 avt_vimba_camera 的 YAML 值，不包含未被默认 launch 引用的 *_1k、*_500p、*_2k 等备份/替代模式。

## 4. 六路默认相机运行配置 [RC]

### 4.1 六路主参数

| camera | IP | frame_id | 默认 calibration | 输出尺寸 | FPS | PixelFormat | OffsetX/Y | Binning | Decimation |
|---|---|---|---|---:|---:|---|---|---|---|
| vimba_front | 10.42.17.52 | vimba_front | cam_front_calib.yaml | 1056×1056 | 20.0 | BayerRG8 | 504 / 244 | 1 / 1 | 1 / 1 |
| vimba_rear | 10.42.17.55 | vimba_rear | cam_rear_calib.yaml | 1056×1056 | 20.0 | BayerRG8 | 504 / 244 | 1 / 1 | 1 / 1 |
| vimba_left | 10.42.17.50 | vimba_left | cam_left_calib.yaml | 1056×1056 | 20.0 | BayerRG8 | 504 / 244 | 1 / 1 | 1 / 1 |
| vimba_right | 10.42.17.54 | vimba_right | cam_right_calib.yaml | 1056×1056 | 20.0 | BayerRG8 | 504 / 244 | 1 / 1 | 1 / 1 |
| vimba_front_left_center | 10.42.17.53 | vimba_front_left_center | cam_front_left_center_calib.yaml | 516×384 | 20.0 | BayerRG8 | 0 / 0 | 1 / 1 | 4 / 4 |
| vimba_front_right_center | 10.42.17.51 | vimba_front_right_center | cam_front_right_center_calib.yaml | 516×384 | 10.0 | BayerRG8 | 0 / 0 | 1 / 1 | 4 / 4 |

> [RC] 这里的 OffsetX/Y、Width/Height、Binning*、Decimation* 是写入 Vimba feature 的参数；它们不是由相机型号页自动推导出来的。

### 4.2 曝光、增益、触发与 PTP

| camera | ExposureAuto | ExposureTimeAbs | GainAuto | Gain | PtpMode | ptp_offset | TriggerMode | TriggerSource |
|---|---|---:|---|---:|---|---:|---|---|
| front | Once | 500 µs | Once | 0.0 | Slave | -37 | On | FixedRate |
| rear | Off | 3000 µs | Off | 14.4 | Slave | -37 | On | FixedRate |
| left | Once | 500 µs | Once | 0.0 | Slave | -37 | On | FixedRate |
| right | Once | 500 µs | Once | 0.0 | Slave | -37 | On | FixedRate |
| front_left_center | Continuous | 500 µs | Continuous | 0.0 | Slave | -37 | On | FixedRate |
| front_right_center | Continuous | 500 µs | Continuous | 0.0 | Slave | -37 | On | FixedRate |

所有六路在默认 YAML 中还写有：

~~~
feature/AcquisitionFrameCount: 1
feature/AcquisitionMode: Continuous
feature/TriggerSelector: FrameStart
feature/TriggerActivation: RisingEdge
feature/TriggerDelayAbs: 0.0
feature/TriggerOverlap: 'Off'
feature/PtpAcquisitionGateTime: 0
feature/ActionDeviceKey: 1
feature/ActionGroupKey: 1
feature/ActionGroupMask: 1
feature/ActionSelector: 0
~~~

曝光/增益自动控制的共同边界值为：

~~~yaml
feature/ExposureAutoAdjustTol: 5
feature/ExposureAutoAlg: FitRange
feature/ExposureAutoMax: 30000
feature/ExposureAutoMin: 29
feature/ExposureAutoOutliers: 1000
feature/ExposureAutoRate: 10
feature/GainAutoAdjustTol: 5
feature/GainAutoMax: 32.0
feature/GainAutoMin: 0.0
feature/GainAutoOutliers: 100
feature/GainAutoRate: 50
feature/GainAutoTarget: 50
~~~

差异项：

- rear 的 BalanceRatioAbs=1.12；其他五路为 2.0。
- rear 的 ExposureAutoTarget=49；其他五路为 50。
- rear 是唯一明确固定 ExposureAuto=Off、ExposureTimeAbs=3000 µs、GainAuto=Off、Gain=14.4 的相机。
- front_right_center 的采集帧率是 10.0，但 trigger 配置文件的 timer 是 20 Hz；仓库没有说明这一差异是有意的还是待修正项。

### 4.3 共同的有效处理/传输参数

下列值在六路默认配置中相同，或除明确例外外相同：

| 类别 | 有效键 | 值 |
|---|---|---|
| 节点 | name | camera |
| 节点 | guid | 空字符串；没有为六路填入设备 GUID |
| 时间戳 | use_measurement_time | true |
| ROS 输出 | publish_compressed | false |
| 白平衡 | BalanceRatioSelector / BalanceWhiteAuto | Red / Off |
| 白平衡 | BalanceWhiteAutoAdjustTol / BalanceWhiteAutoRate | 5 / 100 |
| 带宽 | BandwidthControlMode | StreamBytesPerSecond |
| 黑电平 | BlackLevelSelector / BlackLevel | All / 50.0 |
| chunk | ChunkModeActive | false |
| 颜色变换 | ColorTransformationMode | Off |
| 颜色变换 | ColorTransformationSelector / ValueSelector / Value | RGBtoRGB / Gain00 / 1.0 |
| 温度/用户 ID | DeviceTemperatureSelector / DeviceUserID | Main / 空字符串 |
| 事件 | EventNotification / EventSelector / EventsEnable1 | Off / AcquisitionStart / 0 |
| 曝光模式 | ExposureMode | Timed |
| 增益选择 | GainSelector | All |
| 图像处理 | Gamma / Hue / Saturation | 1.0 / 0.0 / 1.0 |
| 翻转 | ReverseX / ReverseY | false / false |
| 组播 | MulticastEnable | false |
| 组播地址字段 | MulticastIPAddress | 4026470193；该字段在组播关闭时不生效 |
| 缓冲 | StreamBufferHandlingMode | Default |
| 链路速率 | StreamBytesPerSecond | 115000000 |
| 帧率限制 | StreamFrameRateConstrain | true |
| hold | StreamHoldEnable | Off |
| 预录 | RecorderPreEventCount | 0 |
| strobe | StrobeDelay / StrobeDuration / StrobeDurationMode / StrobeSource | 0 / 0 / Source / FrameTrigger |
| sync in | SyncInGlitchFilter / SyncInSelector | 0 / SyncIn1 |
| sync out | SyncOutLevels / SyncOutPolarity / SyncOutSelector / SyncOutSource | 0 / Normal / SyncOut1 / Exposing |
| UserSet | UserSetSelector | Default |

外圈四路的 DSPSubregion* 还各自写有 active 值：

| camera | DSPSubregionLeft/Right | DSPSubregionTop/Bottom | 说明 |
|---|---|---|---|
| front | 0 / 1056 | 306 / 736 | [RC] active feature |
| rear | 0 / 1056 | 246 / 796 | [RC] active feature |
| left | 0 / 1056 | 396 / 756 | [RC] active feature |
| right | 0 / 1056 | 156 / 756 | [RC] active feature |
| 两路 center | 未设置；文件中仅有注释行 | 未设置；文件中仅有注释行 | 不应当当作 active ROI |

DSPSubregion* 与 OffsetX/Y + Width/Height 是不同的 feature 组；本清单不把它们合并成一个“sensor ROI”。

### 4.4 PCAP 与 UserSet 注意项

- front、left、right、front-left-center、front-right-center 的 YAML 中有 enable_pcap=false、pcap_playback_speed=0.667、pcap_loop=false 和一个开发机路径的 pcap_file；因为 enable_pcap=false，默认不是 PCAP 回放。
- rear YAML 没有这组 active PCAP 参数。
- outer 四路和两路 center 写有 UserSetDefaultSelector=Default；rear 只写 UserSetSelector=Default。
- 注释掉的 GVCPCmd*、GVSP* 参数不是 active ROS 参数，不应当当作实际运行值。

### 4.5 触发节点配置与启动不确定项

仓库中存在：

~~~yaml
destination_ip: "10.42.17.255"
trigger_src: "timer"
timer_period: 0.05          # 20 Hz triggering
action_device_key: 1
action_group_key: 1
action_group_mask: 1
~~~

这与六个 camera 的 TriggerMode=On、TriggerSource=FixedRate、action keys 相匹配。但需要区分两件事：

1. iac_launch/launch/cameras_launch/vimba_trigger.launch.py 定义了 trigger node。
2. 同目录 vimba.launch.py 创建了 vimba_trigger_launch 变量，但其 LaunchDescription 返回列表只包含六个 camera launch，没有把该变量加入列表；stable 分支的 tmux 入口又调用 isaac_launch vimba.launch.py，其 IAC 相机节点表也没有加入 trigger component。

因此从公开代码可以确认 **trigger 配置存在**，但不能仅凭默认 launch 文件确认该 trigger node 一定会随这条入口启动。在线实车是否由其他入口、旧进程或硬件网络设备发出 action trigger，需要现场启动日志、ROS graph 或网络抓包确认。

## 5. 六路内参、畸变模型与 ROS CameraInfo [RC]

### 5.1 标准 ROS 字段

ROS 驱动通过各自的 camera_info_url 加载以下标准字段：image_width、image_height、camera_matrix、distortion_model、distortion_coefficients、rectification_matrix、projection_matrix。

内参矩阵统一写作：

~~~
K = [[fx, 0,  cx],
     [0,  fy, cy ],
     [0,  0,  1  ]]
~~~

| camera | image size | distortion model | K（camera_matrix） | D（distortion_coefficients） |
|---|---:|---|---|---|
| front | 1056×1056 | equidistant | [[1026.35682,0,527.48656],[0,1025.85664,520.60822],[0,0,1]] | [0.112560, 0.012172, 0.112262, -0.014034] |
| rear | 1056×1056 | equidistant | [[969.60376,0,514.54172],[0,969.52856,509.21942],[0,0,1]] | [0.128563, -0.050472, 0.183419, -0.044414] |
| left | 1056×1056 | equidistant | [[1017.44460,0,513.77128],[0,1017.13056,525.39936],[0,0,1]] | [0.084982, 0.165945, -0.165464, 0.154104] |
| right | 1056×1056 | equidistant | [[1014.18256,0,516.10560],[0,1013.79152,521.06036],[0,0,1]] | [0.097273, 0.123184, -0.102429, 0.124930] |
| front_left_center | 516×384 | plumb_bob | [[885.4456874262236,0,272.5941691604145],[0,884.2733412910945,188.6284425757264],[0,0,1]] | [-0.208921, 0.003521, -0.005062, 0.002011, 0.0] |
| front_right_center | 516×384 | plumb_bob | [[874.9063570580302,0,251.2557579413245],[0,873.9515194566943,189.08064333258437],[0,0,1]] | [-0.248552, 0.147474, 0.001749, 0.001941, 0.0] |

所有六个 YAML 的 rectification_matrix 都是单位矩阵 I₃。标准 projection_matrix 为：

| camera | 标准 ROS projection_matrix P |
|---|---|
| front | [[1026.35682,0,527.48656,0],[0,1025.85664,520.60822,0],[0,0,1,0]] |
| rear | [[969.60376,0,514.54172,0],[0,969.52856,509.21942,0],[0,0,1,0]] |
| left | [[1017.44460,0,513.77128,0],[0,1017.13056,525.39936,0],[0,0,1,0]] |
| right | [[1014.18256,0,516.10560,0],[0,1013.79152,521.06036,0],[0,0,1,0]] |
| front_left_center | [[1694.7146,0,566.449875,0],[0,1700.180786,286.962465,0],[0,0,1,0]] |
| front_right_center | [[1715.803833,0,623.958957,0],[0,1736.469849,394.38312,0],[0,0,1,0]] |

### 5.2 两路 center YAML 的重复字段

两路 center YAML 还包含非标准/重复字段：

- camera_name=narrow_stereo，而不是 vimba_front_left_center / vimba_front_right_center；实际 URL 由 launch 显式绑定到对应文件名。
- 另有 distortion 字段和 projection 字段，它们与标准 distortion_coefficients / projection_matrix 数值不同。
- 本清单将标准 ROS 字段作为“驱动 CameraInfo 主值”，并把额外字段保留为“文件中存在但用途未确认”；不能把两套数字混合使用。

额外字段原值如下：

~~~yaml
# cam_front_left_center_calib.yaml 中的额外字段
distortion: [-0.25286297655534345, 0.05943606660788565,
             0.002398030252155083, 0.0011937382756416306, 0.0]
projection: [865.727482448658, 0.0, 273.5757995262139, 0.0,
             0.0, 873.580139640985, 188.8715667416139, 0.0,
             0.0, 0.0, 1.0, 0.0]

# cam_front_right_center_calib.yaml 中的额外字段
distortion: [-0.26215974682537396, 0.47151184682535635,
             -0.0010168813144265099, -0.0014191261886042617, 0.0]
projection: [857.4921022966133, 0.0, 250.6986446399015, 0.0,
             0.0, 863.6340642360639, 188.89358212704937, 0.0,
             0.0, 0.0, 1.0, 0.0]
~~~

## 6. 六路外参与 vimba optical-frame 旋转 [RM]

### 6.1 坐标系约定

URDF 中：

~~~
base_link
  └── rear_axle_middle          xyz=0 0 0, identity
       ├── camera_*
       │    └── vimba_*
~~~

因此这里把 rear_axle_middle 当作 base_link 报告。camera_* link 的注释是 “Measured from Cam Sensor Center”；vimba_* 是在该 camera link 下的 optical-style frame。

camera_* -> vimba_* 对六路都相同：

~~~xml
<origin xyz="0 0 0" rpy="-1.5708 0 -1.5708" />
~~~

按 URDF fixed-joint 的 RPY 约定，可写成：

~~~
R_camera_vimba = Rz(-1.5708) * Rx(-1.5708)
               ≈ [[ 0,  0,  1],
                  [-1,  0,  0],
                  [ 0, -1,  0]]
~~~

这里的 vimba_* optical-frame 旋转是 **URDF 坐标约定**，不是重新标定出的相机内参，也不是由相机厂家提供的姿态。

### 6.2 camera link 相对 base_link 的 xyz/rpy

| camera | camera_* xyz (m) | camera_* rpy (rad) | 约合 yaw |
|---|---|---|---:|
| front | (1.365, 0, 0.723) | (0, 0, 0) | 0° |
| rear | (1.215, 0, 0.723) | (0, 0, 3.14159265358979) | 180° |
| left | (2.020, 0.172, 0.418) | (0, 0, 1.74533) | +100° |
| right | (2.020, -0.172, 0.418) | (0, 0, -1.74533) | −100° |
| front_left_center | (2.232, 0.180, 0.442) | (0, 0, 0) | 0° |
| front_right_center | (2.232, -0.180, 0.442) | (0, 0, 0) | 0° |

### 6.3 base_link -> vimba_* 的完整旋转矩阵

以下为 R_base_vimba = R_base_camera · R_camera_vimba 的直接计算结果；平移就是上表 xyz。矩阵保留到约 6 位小数，来源是 URDF 中的 rpy，不代表额外测量精度。

~~~
R_base_vimba_front
= [[ 0,  0,  1],
   [-1,  0,  0],
   [ 0, -1,  0]]

R_base_vimba_rear
= [[ 0,  0, -1],
   [ 1,  0,  0],
   [ 0, -1,  0]]

R_base_vimba_left
≈ [[ 0.984808,  0, -0.173649],
   [ 0.173649,  0,  0.984808],
   [ 0,       -1,  0       ]]

R_base_vimba_right
≈ [[-0.984808,  0, -0.173649],
   [ 0.173649,  0, -0.984808],
   [ 0,       -1,  0       ]]

R_base_vimba_front_left_center
= [[ 0,  0,  1],
   [-1,  0,  0],
   [ 0, -1,  0]]

R_base_vimba_front_right_center
= [[ 0,  0,  1],
   [-1,  0,  0],
   [ 0, -1,  0]]
~~~

若采用齐次变换约定 p_base = T_base_vimba · p_vimba，则例如 front 为：

~~~
T_base_vimba_front =
[[ 0,  0,  1, 1.365],
 [-1,  0,  0, 0    ],
 [ 0, -1,  0, 0.723],
 [ 0,  0,  0, 1    ]]
~~~

其余五路使用同样旋转表和第 6.2 节的平移；不要把 camera_* 的机械 rpy 直接当成 vimba_* optical-frame 的 rpy。

## 7. 文件索引

### 7.1 race_common：启动与六路相机 YAML

- 启动六路列表：[isaac_launch/components/perception_components/vimba_components.py](https://github.com/airacingtech/race_common/blob/64735fd3b6cacac3a577221b8fa85eee1249faa2/src/launch/isaac_launch/components/perception_components/vimba_components.py)
- 默认 launch：[isaac_launch/launch/perception_launch/vimba.launch.py](https://github.com/airacingtech/race_common/blob/64735fd3b6cacac3a577221b8fa85eee1249faa2/src/launch/isaac_launch/launch/perception_launch/vimba.launch.py)
- 传统六路 launch：[iac_launch/launch/cameras_launch/vimba.launch.py](https://github.com/airacingtech/race_common/blob/64735fd3b6cacac3a577221b8fa85eee1249faa2/src/launch/iac_launch/launch/cameras_launch/vimba.launch.py)
- front 参数：[vimba_front.param.yaml](https://github.com/airacingtech/race_common/blob/64735fd3b6cacac3a577221b8fa85eee1249faa2/src/launch/iac_launch/param/cameras_param/vimba_front.param.yaml)
- rear 参数：[vimba_rear.param.yaml](https://github.com/airacingtech/race_common/blob/64735fd3b6cacac3a577221b8fa85eee1249faa2/src/launch/iac_launch/param/cameras_param/vimba_rear.param.yaml)
- left 参数：[vimba_left.param.yaml](https://github.com/airacingtech/race_common/blob/64735fd3b6cacac3a577221b8fa85eee1249faa2/src/launch/iac_launch/param/cameras_param/vimba_left.param.yaml)
- right 参数：[vimba_right.param.yaml](https://github.com/airacingtech/race_common/blob/64735fd3b6cacac3a577221b8fa85eee1249faa2/src/launch/iac_launch/param/cameras_param/vimba_right.param.yaml)
- front-left-center 参数：[vimba_front_left_center.param.yaml](https://github.com/airacingtech/race_common/blob/64735fd3b6cacac3a577221b8fa85eee1249faa2/src/launch/iac_launch/param/cameras_param/vimba_front_left_center.param.yaml)
- front-right-center 参数：[vimba_front_right_center.param.yaml](https://github.com/airacingtech/race_common/blob/64735fd3b6cacac3a577221b8fa85eee1249faa2/src/launch/iac_launch/param/cameras_param/vimba_front_right_center.param.yaml)
- 六路 calibration 目录：[iac_launch/param/cameras_param/](https://github.com/airacingtech/race_common/tree/64735fd3b6cacac3a577221b8fa85eee1249faa2/src/launch/iac_launch/param/cameras_param)
- 六路 calibration 文件：cam_front_calib.yaml、cam_rear_calib.yaml、cam_left_calib.yaml、cam_right_calib.yaml、cam_front_left_center_calib.yaml、cam_front_right_center_calib.yaml，均位于上述目录。
- trigger launch：[vimba_trigger.launch.py](https://github.com/airacingtech/race_common/blob/64735fd3b6cacac3a577221b8fa85eee1249faa2/src/launch/iac_launch/launch/cameras_launch/vimba_trigger.launch.py)
- trigger 参数：[trigger_node.param.yaml](https://github.com/airacingtech/race_common/blob/64735fd3b6cacac3a577221b8fa85eee1249faa2/src/launch/iac_launch/param/cameras_param/trigger_node.param.yaml)
- IAC repos manifest：[repos/iac.jazzy.repos](https://github.com/airacingtech/race_common/blob/64735fd3b6cacac3a577221b8fa85eee1249faa2/repos/iac.jazzy.repos)

### 7.2 race_metadata：AV-24 URDF

- AV-24 URDF：[urdf/iac_car/av24.urdf](https://github.com/airacingtech/race_metadata/blob/36e8c7bf215be739179fb2e20794ee80ced11637/urdf/iac_car/av24.urdf)

### 7.3 厂家与 IAC 公开资料

- [Allied Vision Mako G-319 官方产品页](https://www.alliedvision.com/en/products/area-scan-cameras/mako/mako/view/1051)
- [Allied Vision Mako G-319 Data Sheet PDF](https://www.alliedvision.com/assets/support/Camera-Documentation/Allied-Vision/Cameras/Mako/Data-Sheets/Mako_G-319_DataSheet_en.pdf)
- [Allied Vision Mako User Guide PDF](https://www.alliedvision.com/assets/support/Camera-Documentation/Allied-Vision/User-guides/Mako_TechMan_en.pdf)
- [IAC AV-24 Racecar 页面](https://www.indyautonomouschallenge.com/racecar)
- [IAC Sponsors 页面](https://www.indyautonomouschallenge.com/sponsors)
- [公开专利 WO2025043253A1（AV-24 传感器架构描述）](https://patents.google.com/patent/WO2025043253A1/en)
- [Edmund Optics 的 Mako G-319C 相机页面](https://www.edmundoptics.com/p/allied-vision-mako-g-319-1-18-inch-color-cmos-camera/33094/)；这是相机经销页面，不是 AV-24 六路镜头清单，不能用来推断镜头型号。

## 8. 已知不确定项与使用边界

1. **本文件记录的是公开仓库配置，不等于现场此刻硬件状态。** IP、YAML 和 URDF 能说明软件期望值，不能证明六台相机当前都在线、没有被现场参数覆盖。
2. **镜头型号缺失。** 不能由 K、分辨率或 Mako 型号唯一反推 Edmund Optics lens part number；需要硬件 inventory、采购单、镜头铭牌或现场照片。
3. **相机 serial/GUID 缺失。** rear YAML 注释有 DEV_000F315E7445，但 active guid 仍是空字符串；其他五路没有公开序列号映射。
4. **中心两路 calibration 有重复字段冲突。** 本文以标准 ROS 字段为主，额外字段仅作为原文记录；若应用实际读取的是自定义 parser，应重新核对 parser 源码。
5. **front_right_center 的 10 Hz 与 20 Hz trigger timer 不一致。** 本文不猜测其设计意图；应通过启动日志、topic header 时间戳和相机事件统计验证。
6. **TriggerMode=On 不足以证明 trigger node 正在运行。** 默认 launch 是否把 20 Hz action broadcaster 一起启动，是当前代码路径中的独立问题。
7. **DSPSubregion* 不等于 sensor ROI。** 投影/标定时应使用 calibration YAML 对应的 image_width/image_height 和有效内参，不要用 DSP 字段替换 OffsetX/Y + Width/Height。
8. **改变 imaging geometry 会改变内参适用性。** ROI、binning、decimation、输出尺寸或镜头 focus 变化后，不能默认继续使用当前固定 K,D；应重新确认 calibration 与当前图像坐标是否一致。
9. **公开经销页面存在分辨率文字差异。** Allied Vision 官方资料写 2064×1544；部分 Edmund Optics 页面文字写 2048×1544。本清单以 Allied Vision 官方数据表/产品页为准。
