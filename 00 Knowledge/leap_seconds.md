`leap_seconds` = **闰秒差**，这里具体指：

```
GPS time - UTC time
```

GPS 时间是连续数秒，不插闰秒；UTC 会因为地球自转校正插入闰秒。所以同一个真实时刻：

```
GPS time = UTC time + leap_seconds
UTC/Unix time = GPS time - leap_seconds
```