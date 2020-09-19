[Top](./README.md)

## 团灭leetcode股票买卖问题

使用状态机方法解决问题，实际上是DP table

总框架
```python
for 状态1 in 状态1的所有取值:
    for 状态2 in 状态2的所有取值:
        for ...
            dp[状态1][状态2][...] = 择优(选择1， 选择2...)
```

### 穷举框架
这里使用状态进行穷举。比如说股票问题，每天有三种选择: 买入，卖出，无操作。结合题意，
这问题的状态有三个，第一个是天数，第二个是允许交易的最大次数，第三个是当前持有状态。
pseudo code如下
```python
dp[i][k][0 or 1]
0 <= i <= n - 1, 1 <= k <= K
n表示天数，K表示交易次数
0 表示卖出，1 表示持有
一共n * K* 2 种状态，全部穷举就能完成

for <= i < n:
  for 1 <= k <= K:
    for s in [0, 1]:
      dp[i][k][s] = max(buy, sell, rest)
```

比如dp[3][2][1] 意思是今天第三天，至今最多进行了2次交易，手上还持有股票。
我们最终是要求dp[n-1][K][0]，即是最后一天，最多进行了K次交易，手上已经将股票卖完了，最多
获得多少利润。

### 状态转移框架
```python
dp[i][k][0] = max(dp[i-1][k][0] + dp[i-1][k][1] + prices[i])
              max( 选择 rest，          选择sell           )
今天我没有持有股票有两种可能
一种是: 我昨天没有持有，然后今天选择rest，继续不持有
另一种是: 我昨天持有了，然后我今天sell了，所以今天也是没有持有
如果sell的话，要给利润增加price(因为今天卖了股票拿钱了)

dp[i][k][1] = max(dp[i-1][k][1], dp[i-1][k-1][0] - prices[i])
              max( 选择rest,            选择buy             )

今天我没有持有股票有两种可能
一种是：我昨天持有了，今天选择rest，继续持有
另一种是：我昨天没有持有，今天选择buy，开始持有
如果buy的话，要从利润中减去price(因为今天出钱买了股票)
```
下面准备base case
```python
dp[-1][k][0] = 0
i = -1 表示还没有开始，利润当然是0
dp[-1][k][1] = -inf
还没有开始的时候是不可能持有股票的，用负无穷表示不可能
dp[i][0][0] = 0
k是从1开始，k = 0 意味着不允许交易，所以利润为0
dp[i][0][1] = -inf
不允许交易就根本不可能持有股票，用负无穷表示不可能
```

总结一下转移状态和base case
```python
base case
dp[-1][k][1] = dp[i][0][1] = -inf
dp[-1][k][0] = dp[i][0][0] = 0

状态转移方程
dp[i][k][0] = max(dp[i-1][k][0], dp[i-1][k][1] + prices[i])
dp[i][k][1] = max(dp[i-1][k][1], dp[i-1][k-1][0] - prices[i])
```

### 秒杀题目
> k = 1

套进去框架里面
```python
dp[i][1][0] = max(dp[i-1][1][0], dp[i-1][1][1] + prices[i])
dp[i][1][1] = max(dp[i-1][1][1], dp[i-1][0][0] - prices[i]) 
            = max(dp[i-1][1][1], -prices[i])
解释：k = 0 的 base case，所以 dp[i-1][0][0] = 0。

现在发现 k 都是 1，不会改变，即 k 对状态转移已经没有影响了。
可以进行进一步化简去掉所有 k：
dp[i][0] = max(dp[i-1][0], dp[i-1][1] + prices[i])
dp[i][1] = max(dp[i-1][1], -prices[i])
```

直接写出代码
```python
n = len(prices)
dp= []
for _ in range(n):
    inner = []
    for i in range(2):
        inner.append(i)
    dp.append(inner)
for i in range(n):
    # base case
    if i - 1 == -1:
        dp[0][0] = 0
        dp[0][1] = -prices[i]
        continue
    dp[i][0] = max(dp[i-1][0], dp[i-1][1] + prices[i])
    dp[i][1] = max(dp[i-1][1], -prices[i])
return dp[n - 1][0]
```

新状态只和相邻的一个状态有关，其实不用整个 dp 数组，
只需要一个变量储存相邻的那个状态就足够了，这样可以把空间复杂度降到 O(1)

```python
def maxProfit_k_1(prices: List[int]) -> int:
    n = len(prices)
    for i in range(n):
        # dp[i][0] = max(dp[i - 1][0], dp[i - 1][1] + prices[i])
        dp_i_0 = max(dp_i_0, dp_i_1 + prices[i])
        # dp[i][1] = max(dp[i-1][1], -prices[i])
        dp_i_1 = max(dp_i_1, -prices[i])
    return dp_i_0
```

> k = +inf
如果 k 为正无穷，那么就可以认为 k 和 k - 1 是一样的。可以这样改写框架：
```python
dp[i][k][0] = max(dp[i-1][k][0], dp[i-1][k][1] + prices[i])
dp[i][k][1] = max(dp[i-1][k][1], dp[i-1][k-1][0] - prices[i])
            = max(dp[i-1][k][1], dp[i-1][k][0] - prices[i])

我们发现数组中的 k 已经不会改变了，也就是说不需要记录 k 这个状态了：
dp[i][0] = max(dp[i-1][0], dp[i-1][1] + prices[i])
dp[i][1] = max(dp[i-1][1], dp[i-1][0] - prices[i])
```

直接写出代码

```python
def maxProfit_k_inf(prices: List[int]):
    n = len(prices)
    dp_i_0 = 0
    dp_i_1 = -float('inf')
    for i in range(n):
        # we need to use the dp_i_0 of yesterday
        temp = dp_i_0
        dp_i_0 = max(dp_i_0, dp_i_1 + prices[i])
        dp_i_1 = max(dp_i_1, temp - prices[i])
    return dp_i_0
```

> k = +inf with cooldown

每次 sell 之后要等一天才能继续交易。只要把这个特点融入上一题的状态转移方程即可

```python
dp[i][0] = max(dp[i-1][0], dp[i-1][1] + prices[i])
dp[i][1] = max(dp[i-1][1], dp[i-2][0] - prices[i])
解释：第 i 天选择 buy 的时候，要从 i-2 的状态转移，而不是 i-1 。
```

直接写出代码
```python
def maxProfit_with_cool(prices: List[int]):
    n = len(prices)
    dp_i_0 = 0
    dp_i_1 = -float('inf')
    dp_pre_0 = 0
    for i in range(n):
        temp = dp_i_0
        dp_i_0 = max(dp_i_0, dp_i_1 + prices[i])
        dp_i_1 = max(dp_i_1, dp_pre_0 - prices[i])
        dp_pre_0 = temp
    return dp_i_0
```
> k = +inf with fee

每次交易要支付手续费，只要把手续费从利润中减去即可

```python
dp[i][0] = max(dp[i-1][0], dp[i-1][1] + prices[i])
dp[i][1] = max(dp[i-1][1], dp[i-1][0] - prices[i] - fee)
解释：相当于买入股票的价格升高了。
在第一个式子里减也是一样的，相当于卖出股票的价格减小了。
```

写出代码

```python
def maxProfit_with_fee(prices: List[int], fee: int):
    n = len(prices)
    dp_i_0 = 0
    dp_i_1 = -float('inf')
    for i in range(n):
        temp = dp_i_0
        dp_i_0 = max(dp_i_0, dp_i_1 + prices[i])
        dp_i_1 = max(dp_i_1, temp - prices[i] - fee)
    return dp_i_0
```

> k = 2

这道题 k = 2 和后面要讲的 k 是任意正整数的情况中，对 k 的处理就凸显出来了
这道题由于没有消掉 k 的影响，所以必须要对 k 进行穷举
这里 k 取值范围比较小，所以可以不用 for 循环，直接把 k = 1 和 2 的情况全部列举出来也可以

```python
#dp[i][2][0] = max(dp[i-1][2][0], dp[i-1][2][1] + prices[i])
#dp[i][2][1] = max(dp[i-1][2][1], dp[i-1][1][0] - prices[i])
#dp[i][1][0] = max(dp[i-1][1][0], dp[i-1][1][1] + prices[i])
#dp[i][1][1] = max(dp[i-1][1][1], -prices[i])
def maxProfit_k_2(prices: List[int]):
    dp_i10 = dp_i20 = 0
    dp_i11 = dp_i21 = -float('inf')
    for price in prices:
        dp_i20 = max(dp_i20, dp_i21 + price)
        dp_i21 = max(dp_i21, dp_i10 - price)
        dp_i10 = max(dp_i10, dp_i11 + price)
        dp_i11 = max(dp_i11, -price)
    return dp_i20
```

> k = any int

但是出现了一个超内存的错误，原来是传入的 k 值会非常大，dp 数组太大了
一次交易由买入和卖出构成，至少需要两天。所以说有效的限制 k 应该不超过 n/2，
如果超过，就没有约束作用了，相当于 k = +infinity。这种情况是之前解决过的

```python
def maxProfit_k_any(max_k: int, prices: List[int]):
    n = len(prices)
    if max_k > n // 2:
        return maxProfit_k_inf(prices)
    dp = []
    for _ in range(n):
        inner1 = []
        for _ in range(k+1):
            inner2 = []
            for i in range(2):
                inner2.append(i)
            inner1.append(inner2)
        dp.append(inner1)
    for i in range(n):
        for k in range(max_k, 0, --):
            if i - 1 == -1:
                dp[0][k][0] = 0
                dp[0][k][1] = -price[i]
            dp[i][k][0] = max(dp[i-1][k][0], dp[i-1][k][1] + prices[i])
            dp[i][k][1] = max(dp[i-1][k][1], dp[i-1][k-1][0] - prices[i])
    return dp[n - 1][max_k][0]
```
