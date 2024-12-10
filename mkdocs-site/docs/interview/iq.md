# 一些被认为是智力测试的题目

一副扑克牌，一共54张，大王小王算“0”，从中随机抽取5张，请**优雅地**判断这5个数字是否组成顺子（“0”可以看成1-13中的任何整数；“A”看成1；“J”，“Q”，“K”分别看成11,12,13）。

也不确定这是不是最优解法：先对序列进行升序排列，把所有0都拿开考虑。计算一个差分序列，观察到在满足条件时候，差分序列的和在$[4-\# 0, 4]$之间。
```python
def check_sequence():
    nums = sorted(map(int, input().split()))  # 读入数字并排序
    zero_count = nums.count(0)  # 计算 0 的个数
    diffs = [nums[i + 1] - nums[i] for i in range(zero_count, 4)]  # 非零部分差分序列
    # 条件判断并分别输出
    if 0 not in diffs and sum(diffs) in range(4 - zero_count, 5):
        print("true")
    else:
        print("false")
```