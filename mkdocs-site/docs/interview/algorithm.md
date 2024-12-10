# 一些小的算法问题

如何写代码逼近圆周率$\pi$？

当时的作答是：把半径为1的圆内切在边长为2的正方形里面，用采样的方式逼近圆周率。
```python
import random
iterations = 10000
inside_count = 0
for i in range(iterations):
    x = random.uniform(0,2)
    y = random.uniform(0,2)
    if (x-1)**2 + (y-1)**2 <= 1:
        inside_count += 1
print("\pi = ", inside_count / iterations * 4)
```

作答时候还提出了一个思路，利用$\pi = \arccos(-1)$，对arccos函数泰勒展开取前面几项，后来发现自己蠢了，这么做不行。


DP：输入一个m*n的0-1矩阵，求出全部都是1的正方形的最大边长；

DP从右下角开始
