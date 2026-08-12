“经纬高转 UTM”就是把 GPS 那种球面坐标，变成地图/机器人算法更好用的平面米制坐标。
原始 GNSS 位姿通常是：

```
latitude   纬度，单位 degree
longitude  经度，单位 degree
altitude   高度，单位 meter
```

UTM，全称 Universal Transverse Mercator，是一种把地球局部区域投影到平面上的坐标系。转换后变成：

```
easting   东向坐标，单位 meter
northing  北向坐标，单位 meter
altitude  高度，单位 meter
```