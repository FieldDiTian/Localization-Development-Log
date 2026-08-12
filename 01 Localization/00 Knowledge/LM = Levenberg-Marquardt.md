LM = Levenberg-Marquardt。

你可以先把它理解成：

```
一种迭代优化方法，用来一点点调整位姿，让 scan 和 map 更贴合。
```

GICP 要找的是一个 6DoF 位姿：

```
x0 = [roll, pitch, yaw, tx, ty, tz]
```

当然代码里不是欧拉角，而是 `SE(3)` / `SO(3)` 增量。

LM 每轮做的事情是：

```
当前位姿 x0 不够准
算一下应该往哪个方向改一点 delta
如果改完误差变小，就接受
如果改完误差变大，就缩小步子再试
```

所以 LM 比普通 Gauss-Newton 稳一点。`lambda` 就是“保守程度”：

```
lambda 小：更像 Gauss-Newton，步子大胆
lambda 大：更像梯度下降，步子保守
```