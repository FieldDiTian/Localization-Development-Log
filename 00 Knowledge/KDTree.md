给地图建 KDTree。

每轮匹配时：

```
target_kdtree_->nearestKSearch(pt, 1, k_indices, k_sq_dists);
```

查变换后的 source 点在地图里的最近点。