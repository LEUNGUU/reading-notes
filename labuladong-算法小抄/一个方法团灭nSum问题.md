[Top](./README.md)

## 一个方法团灭 nSum 问题

如果假设输入一个数组 nums 和一个目标和 target，请你返回 nums 中能够凑出 target 的两个元素的值，
比如输入 nums = [1,3,5,6], target = 9，那么算法返回两个元素 [3,6]。
可以假设只有且仅有一对儿元素可以凑出 target

```python
def twoSum(nums: List, target: int):
    # sort the array
    sorted(nums)
    lo = 0
    hi = len(nums) - 1
    while lo < hi:
        total = nums[lo] + nums[hi]
        if total < target:
            lo += 1
        elif total > target:
            hi -= 1
        elif total == target:
            return [lo, hi]
    return {}
```
修改题目：
nums 中可能有多对儿元素之和都等于 target，请你的算法返回所有和为 target 的元素对儿，
其中不能出现重复

```python
def twoSum(nums: List[int], target: int):
    # sort the array
    sorted(nums)
    lo = 0
    hi = len(nums) - 1
    res = []
    while lo < hi:
        total = nums[lo] + nums[hi]
        left = nums[lo]
        right = nums[hi]
        if total < target:
            while lo < hi and nums[lo] == left:
                lo += 1
        elif total > target:
            while lo < hi and nums[hi] == right:
                hi -= 1
        elif total == target:
            res.append([lo, hi])
            while lo < hi and nums[lo] == left:
                lo += 1
            while lo < hi and nums[hi] == right:
                hi -= 1
    return res
```

### 3sum

三数之和，其实就是确定了第一个数字之后，剩下的两个数字就可以用twosum来解

```python
def twoSum(nums: List[int], start: int, target: int):
    # sort the array
    lo = start
    hi = len(nums) - 1
    res = []
    while lo < hi:
        total = nums[lo] + nums[hi]
        left = nums[lo]
        right = nums[hi]
        if total < target:
            while lo < hi and nums[lo] == left:
                lo += 1
        elif total > target:
            while lo < hi and nums[hi] == right:
                hi -= 1
        elif total == target:
            res.append([lo, hi])
            while lo < hi and nums[lo] == left:
                lo += 1
            while lo < hi and nums[hi] == right:
                hi -= 1
    return res

def threeSum(nums: List[int], target: int):
    sorted(nums)
    res = []
    n = len(nums)
    for i in range(n):
        # 对target - nums[i]计算twoSum
        twosumtarget = twoSum(nums, i+1, target-nums[i])
        for item in twosumtarget:
            item.append(nums[i])
            res.append(item)
        while i < n - 1 and nums[i] == nums[i + 1]:
            i += 1
    return res
```

### 4sum

四数之和，类似三数，穷举第一个数字之和，调用三数之和。。。

```python
def fourSum(nums: List[int], target: int):
    sorted(nums)
    res = []
    n = len(nums)
    for i in range(n):
        threeSumTarget = threeSumTarget(nums, i+1, target-nums[i])
        for item in threeSumTarget:
            item.append(nums[i])
            res.append(item)
        while i < n - 1 and nums[i] == nums[i + 1]:
            i += 1
    return res

def threeSumTarget(nums: List[int], start: int, target: int):
    ...
```

### 100sum

不断分解至twosum就好了

```python
nums = sorted(nums)
def nSumTarget(nums: List[int], n: int, start: int, target: int)
    sz = len(nums)
    res = []
    # 至少是2Sum, 并且数组大小不应该少于n
    if n < 2 or sz < n:
        return res
    # 2Sum 是base case
    if n == 2:
        lo = start
        hi = sz - 1
        while lo < hi:
            total = nums[lo] + nums[hi]
            left = nums[lo]
            right = nums[hi]
            if total < target:
                while lo < hi and nums[lo] == left:
                    lo += 1
            elif total > target:
                while lo < hi and nums[hi] == right:
                    hi -= 1
            elif total == target:
                res.append([left, right])
                while lo < hi and nums[lo] == left:
                    lo += 1
                while lo < hi and nums[hi] == right:
                    hi -= 1
    # n > 2 时，递归计算(n - 1)Sum
    else:
        for i in range(start, sz):
            sub = nSumTarget(nums, n - 1, i + 1, target - nums[i])
            for item in sub:
                item.append(nums[i])
                res.append(item)
        while i < sz - 1 and nums[i] == nums[i + 1]:
            i += 1
    return res
```

> 调用之前需要先进行排序，如果在函数里面排序的话，每次递归就会进行多余的不必要的排序
