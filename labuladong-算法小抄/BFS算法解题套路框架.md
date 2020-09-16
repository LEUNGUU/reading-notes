[Top](./README.md)

## BFS 算法解题套路框架

先说一下DFS
> 其实 DFS 算法就是回溯算法

BFS 和 DFS 的区别：BFS 找到的路径一定是最短的，但代价就是空间复杂度比 DFS 大很多

问题的本质就是让你在一幅「图」中找到从起点 start 到终点 target 的最近距离，
这个例子听起来很枯燥，但是 BFS 算法问题其实都是在干这个事儿

直接上代码框架

```python
# 计算从起点 start 到终点 target 的最近距离
def BFS(start: Node, target: Node):
    # 队列
    q = []
    # 避免走回头路
    visited = set()

    # 起点入队
    q.append(start)
    visited.add(start)
    # 记录步数
    step = 0

    while q:
        # 将队列中的所有节点向四周扩散
        for i in range(len(q)):
            cur = q.pop()

            # 判断是否到达终点
            if cur is target:
                return step
            # 将cur的相邻节点加入队列
            for node in cur.adj():
                if node not in visited:
                    q.append(node)
                    visited.add(node)
        # 更新步数
        step += 1
```

### 二叉树最小高度
起点就是root根节点，终点是靠近根节点的那个叶子节点

直接套框架

```python
from collections import deque

def minDepth(root: TreeNode) -> int:
    if root is None:
        return 0
    deque = []
    q.append(root)
    # root 本身就是一层，depth初始化为1
    depth = 1

    while q:
        # 将当前队列中所有节点向四周扩散
        for _ in range(len(q)):
            cur = q.popleft()
            # 判断是否到达终点
            if cur.left is None and cur.right is None:
                return depth
            # 将cur的相邻节点加入队列
            if cur.left is not None:
                q.append(cur.left)
            if cur.right is not None:
                q.append(cur.right)
        # 增加步数
        depth += 1
    return depth
```

### 解开密码锁的最少次数

比如说从 "0000" 开始，转一次，可以穷举出 "1000", "9000", "0100", "0900"... 共 8 种密码。
然后，再以这 8 种密码作为基础，对每个密码再转一下，穷举出所有可能...
仔细想想，这就可以抽象成一幅图，每个节点有 8 个相邻的节点

```python
from collections import deque

def plusOne(s: str, j: int) -> str:
    nums = list(s)
    nums[j] = '0' if nums[j] == '9' else str(int(nums[j]) + 1)
    return ''.join(nums)

def minusOne(s: str, j: int) -> str:
    nums = list(s)
    nums[j] = '9' if nums[j] == '0' else str(int(nums[j]) - 1)
    return ''.join(nums)

# BFS 框架，打印出所有可能的密码
def openlock(deadends: List[str], target: str):
    # 记录需要跳过的密码, 实际上我们可以直接将deads加到visited里面
    # deads = set(deadends)
    # 记录已经穷举过的密码，防止走回头路
    visited = set(deadends)
    q = deque()
    q.append('0000')
    visited.add('0000')
    step = 0

    while q:
        for _ in range(len(q)):
            # 将当前队列中所有节点向四周扩散
            cur = q.popleft()
            # 判断是否到达终点
            if cur in deads:
                continue
            if cur == target:
                return step

            # 将cur的相邻节点加入队列
            for j in range(4):
                up = plusOne(cur, j)
                if up not in visited:
                    q.append(up)
                    visited.add(up)
                down = minusOne(cur, j)
                if down not in visited:
                    q.append(down)
                    visited.add(down)

        # 增加步数
        step += 1
    return -1
```

### 双向BFS

传统的 BFS 框架就是从起点开始向四周扩散，遇到终点时停止；
而双向 BFS 则是从起点和终点同时开始扩散，当两边有交集的时候停止
不过，双向 BFS 也有局限，因为你必须知道终点在哪里

我们可以将上面题目稍加修改
```python
def openlock(deadends: List[str], target: str):
    # 记录需要跳过的密码, 实际上我们可以直接将deads加到visited里面
    # deads = set(deadends)
    # 记录已经穷举过的密码，防止走回头路
    visited = set(deadends)
    # 用集合不用队列，可以快速判断元素是否存在
    q1 = set()
    q2 = set()
    q1.add('0000')
    q2.add(target)
    visited.add('0000')
    step = 0

    while q1 and q2:
        if len(q1) > len(q2):
            #交换q1 和 q2
            temp = q1
            q1 = q2
            q2 = temp
        temp = set()
        # 将当前队列中所有节点向四周扩散
        for cur in q1:
            # 判断是否到达终点
            if cur in deads:
                continue
            if cur in q2:
                return step
            visited.add(cur)

            # 将cur的相邻节点加入队列
            for j in range(4):
                up = plusOne(cur, j)
                if up not in visited:
                    temp.add(up)
                down = minusOne(cur, j)
                if down not in visited:
                    temp.add(down)

        # 增加步数
        step += 1
        # temp 相当于q1
        # 这里交换q1 q2,下一轮while就是扩散q2
        q1 = q2
        q2 = temp
    return -1
```

使用 HashSet 方便快速判断两个集合是否有交集。
另外的一个技巧点就是 while 循环的最后交换 q1 和 q2 的内容，
所以只要默认扩散 q1 就相当于轮流扩散 q1 和 q2。

