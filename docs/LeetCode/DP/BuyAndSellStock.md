## 声明

本文取自 LeetCode 排名第一的大佬 [@Storm](https://leetcode.cn/u/stormsunshine/) 的文章[股票问题系列通解（转载翻译）](https://leetcode.cn/circle/article/qiAgHn/)，文章本身写已经足够清晰完整，并且评论区有一些优秀的问题和答案，但是，本菜鸡为了加深的自己的理解需要增加些注释信息，所以需要在这里搬运/修改部分内容。

## 前言

LeetCode 上关于**买卖股票的最佳时机**的题目目前有 6 道，每道题中都有优秀的题解，但是这些题解多属于**特解**，而非**通解**，本身关联性不强或没有关联性，本篇文章给出了解决股票买卖的通解以及特定条件下的特解，并且每道题都提供了空间压缩前后的 Java 代码。以下是六道题的链接：

- [121. 买卖股票的最佳时机（只允许一次交易）](https://leetcode-cn.com/problems/best-time-to-buy-and-sell-stock/)
- [122. 买卖股票的最佳时机 II（不限制交易次数）](https://leetcode-cn.com/problems/best-time-to-buy-and-sell-stock-ii/)
- [123. 买卖股票的最佳时机 III（最多进行两次交易）](https://leetcode-cn.com/problems/best-time-to-buy-and-sell-stock-iii/)
- [188. 买卖股票的最佳时机 IV（最多进行 K 次交易）](https://leetcode-cn.com/problems/best-time-to-buy-and-sell-stock-iv/)
- [309. 最佳买卖股票时机含冷冻期（不限制交易次数，但是卖出股票后需要休息一天才能买入股票。）](https://leetcode-cn.com/problems/best-time-to-buy-and-sell-stock-with-cooldown/)
- [714. 买卖股票的最佳时机含手续费（不限制交易次数，但是每笔交易需要支付一笔手续费。）](https://leetcode-cn.com/problems/best-time-to-buy-and-sell-stock-with-transaction-fee/)

## 通用情况

寻求通解这个想法基于如下问题：**给定一个表示每天股票价格的数组，有哪些因素可以左右完成股票交易后获得的最大收益？**

相信很多人可以很快给出一些答案，例如“在哪些天进行交易以及允许多少次交易”，这些因素当然重要，也是描述问题的必要因素，然而还有一个隐藏但是关键的因素也可以影响最大收益，下文会阐述这一关键**隐藏因素**。

### 状态定义

在正式提出通解之前，我们首先定义一些下文可能会用到的符号：

- 用 $n$ 表示股票价格数组的长度，即 $n = prices.length$。
- 用 $i$ 表示第 $i$ 天，$i \in [0, n-1]$。
- 用 $k$ 表示允许的最大交易次数
- 用 $T[i][k]$ 表示在第 $i$ 天结束时，**最多**进行 $k$ 次交易的情况下可以获得的最大收益。

### 基准情况

基准情况是比较容易确定的：

- $T[-1][k]=0$：表示在没有进行股票交易时没有收益（注意前面定义的 $i \in [0, n-1]$，所以 $i=-1$ 表示没有产生交易）。
- $T[i][0]=0$：表示在任意一天均没有进行交易

### 状态转移

在股票买卖问题中，相关子问题有以下几种：

- $T[i-1][k]$：在第 $i-1$ 天结束时，**最多**进行 $k$ 次交易的情况下可以获得的最大收益。
- $T[i][k-1]$：在第 $i$ 天结束时，**最多**进行 $k-1$ 次交易的情况下可以获得的最大收益。
- $T[i-1][k-1]$：在第 $i-1$ 天结束时，**最多**进行 $k-1$ 次交易的情况下可以获得的最大收益。

接下来的问题就是：如何将问题 $T[i][k]$ 与子问题 $T[i-1][k], T[i][k-1], T[i-1][k-1]$ 关联起来？

最直接的办法就是看第 $i$ 天可以进行哪些操作，答案很显然，可行操作有三种：**买入，卖出，休息。**但是具体采用哪一种操作收益最大无法直接得知，因此只能通过计算得到选个每个操作后得到的最大收益。

假设没有其他限制条件，则可以尝试每一种操作，并选择可以最大化收益的那一种。但是，题目中确实存在限制条件：

- 不能同时进行多次交易
- 在买入股票之前必须持有 $0$ 份股票
- 如果决定在第 $i$ 天卖出，则卖出之前必须恰好持有 $1$ 份股票。

> 上文提到的隐藏因素：持有股票的数量
>
> 该因素影响第 $i$ 天可以进行的操作，进行影响股票收益。

因此对 $T[i][k]$ 的定义需要分成两项：

- $T[i][k][0]$：表示在第 $i$ 天结束时，**最多**可以进行 $k$ 次交易且在进行操作后持有 $0$ 份股票的情况下可以获得最大收益。
- $T[i][k][1]$：表示在第 $i$ 天结束时，**最多**可以进行 $k$ 次交易且在进行操作后持有 $1$ 份股票的情况下可以获得最大收益。

使用新的**状态定义**之后，我们再重新回顾一下**基准情况**与**状态转移**。

首先是基准情况：

- $T[-1][k][0]=0$：没有股票交易的情况下持有 $0$ 份股票的收益为 $0$
- $T[-1][k][1] = -\infty$：没有股票交易的情况下持有 $1$ 份股票的收益为“不可能”
- $T[i][0][0]=0$：没有股票交易的情况下持有 $0$ 份股票的收益为 $0$
- $T[i][0][1]=-\infty$：没有股票交易的情况下持有 $1$ 份股票的收益为”不可能“

接下来是状态转移方程：

- $T[i][k][0] = max(T[i-1][k][0], T[i-1][k][1] + prices[i])$：在第 $i$ 天结束时持有 $0$ 份股票
    - 要么是**休息**（第 $i-1$ 天也没有股票）
    - 要么是**卖出**（第 $i-1$ 天有 $1$ 份股票，在第 $i$ 天卖出后收益了 $prices[i]$）。
- $T[i][k][1] = max(T[i-1][k][1], T[i-1][k - 1][0] - prices[i])$：在第 $i$ 天结束时持有 $1$ 份股票
    - 要么是**休息**（第 $i-1$ 天也持有 $1$ 份股票）
    - 要么是**买入**（第 $i-1$ 天没有持有股票，在第 $i$ 天购买了股票，花费 $prices[i]$）

> 需要注意的是，在**卖出**时，我们并没有将股票交易次数 $k$ 减一，而在**买入**股票时，将股票交易次数 $k$ 减一。
>
> 这是因为：我们将一次交易定义为“一次完整的买入+卖出操作”，并且认为“买入是交易的开始”，因此只在买入时对交易次数 $k$ 减一。

接下来要做的，就是遍历股票价格数组，计算 $T[i][k][0], T[i][k][1]$ 得出股票收益数组，最终所求答案为 $T[n-1][k][0]$，因为结束时持有 $0$ 份股票的收益一定大于持有 $1$ 份股票的收益。

> 你也许会有疑问：为什么持有 $0$ 份收益一定大于持有 $1$ 份收益？因为如果结束时持有 $1$ 份股票，可能有两种情况，一是在最后一天买入，而是在之前某一天买入。
>
> - 第一种情况：因为股票价格均为正数，只要购买就一定会减少当前的收益，所以不买股票收益更高，即 $maxProfit_{n-1} \gt maxProfit_{n-1} - prices_{n-1}, prices_{n-1} \in (0, +\infty)$。
> - 第二种情况：因为股票价格均为正数，则最后一天卖出则一定可以获得收益，即 $maxProfit_{n-2} \lt maxProfit_{n-2} + prices_{n-1}$ 。

## 特殊情况

前文提到的六道题目的主要差距都在 $k$ 值的划分上，最后两个问题还有额外的附加限制，即“冷冻期”和“手续费”，对通解稍作修改后也可以解决。

### 只允许一次交易

此种情况对应的是 [121. 买卖股票的最佳时机](https://leetcode-cn.com/problems/best-time-to-buy-and-sell-stock/)，每天的交易结果有两个未知变量：`T[i][1][0]` 和 `T[i][1][1]`，这种情况下的状态转移方程如下所示，其中，$T[i][1][1]$ 的计算使用到了前文提到的 $T[i][0][0] = 0$ 这个条件进行化简：

- $T[i][1][0] = max(T[i-1][1][0], T[i-1][1][1] + prices[i])$
- $T[i][1][1] = max(T[i-1][1][1], T[i-1][0][0] - prices[i]) = max(T[i-1][1][1], -prices[i])$

因为 $k=1$ 恒定不变，不影响结果，所以可以省略。根据前文所述状态定义与状态转移方程，可以写出时间复杂度和空间复杂度均为 $O(n)$ 的原始解法，以及进行空间压缩后的空间复杂度为 $O(1)$ 的解法。

<!-- tabs:start -->

#### **原始解法**

```java
public int maxProfit(int[] prices) {
    if (prices == null || prices.length == 0) {
        return 0;
    }
    // 状态定义
    int[][] dp = new int[prices.length][2];
    // 基准条件
    dp[0][0] = 0; // 休息
    dp[0][1] = -prices[0]; // 买入
    // 状态转移
    for (int i = 1; i < prices.length; i++) {
        dp[i][0] = Math.max(dp[i - 1][0], dp[i - 1][1] + prices[i]); // 休息/卖出
        dp[i][1] = Math.max(dp[i - 1][1], -prices[i]); // 休息/买入
    }
    return dp[prices.length - 1][0];
}
```

#### **空间压缩解法**

```java
public int maxProfit(int[] prices) {
    if (prices == null || prices.length == 0) {
        return 0;
    }
    // 基准条件
	int profit0 = 0; // 休息
    int profit1 = -prices[0]; // 买入
    // 状态转移
    for (int i = 1; i < prices.length; i++) {
        profit0 = Math.max(profit0, profit1 + prices[i]); // 休息/卖出
		profit1 = Math.max(profit1, -prices[i]); // 休息/买入
    }
    return profit0;
}
```

<!-- tabs:end -->

对于循环中的部分，`profit1` 实际上是在表示到第 `i` 天为止，出现过的**最低的股票价格**，`profit0` 表示的是进行**卖出**和**休息**时可以获得的最大收益，如果选择**卖出**，则买入股票时的价格为 `profit1`，即在 `0~i-1` 天之中出现过的最低的股票价格，当天的卖出价格 `prices[i]` 减去最低购买价格 `profit1` 即为最大收益。

> 注意 `profit1` 总为负数，所以 `profit1 + prices[i]` 相当于在 `prices[i]` 减去购买价格 `profit1`。

### 不限制交易次数

此种情况对应的是[122. 买卖股票的最佳时机 II](https://leetcode-cn.com/problems/best-time-to-buy-and-sell-stock-ii/)，每天的交易结果有两个未知变量：`T[i][k][0]` 和 `T[i][k][1]`，因为不限制交易次数，所以交易次数的减少并不影响最后的收益，可以认为 `k ~ k-1`（`k` 等价于 `k - 1`），这种情况下的状态转移方程如下所示：

- $T[i][k][0] = max(T[i-1][k][0], T[i - 1][k][1] + prices[i])$
- $T[i][k][1] = max(T[i-1][k][1], T[i - 1][k - 1][0] - prices[i]) = T[i - 1][k][0] - prices[i])$

对于 `T[i][k][1]` 的计算使用到了 `T[i][k][0]=T[i][k-1][0]` 这个条件，并且因为 `k ~ k-1`，根据上面的状态定义与状态转移方程，可以省略交易次数 `k`，写出时间复杂度和空间复杂度均为 $O(n)$ 的原始解法，以及进行空间压缩后的空间复杂度为 $O(1)$ 的解法。

<!-- tabs:start -->

#### **原始解法**

```java
public int maxProfit(int[] prices) {
    if (prices == null || prices.length == 0) {
        return 0;
    }
    int[][] dp = new int[prices.length][2];
    dp[0][0] = 0; // 休息
    dp[0][1] = -prices[0]; // 买入
    
    for (int i = 1; i < prices.length; i++) {
        dp[i][0] = Math.max(dp[i - 1][0], dp[i - 1][1] + prices[i]); // 休息或卖出
        dp[i][1] = Math.max(dp[i - 1][1], dp[i - 1][0] - prices[i]); // 休息或买入
    }
    
    return dp[prices.length - 1][0];
}
```

#### **空间压缩解法**

```java
public int maxProfit(int[] prices) {
    if (prices == null || prices.length == 0) {
        return 0;
    }
    int profit0 = 0;
    int profit1 = -prices[0];
    
    for (int i = 1; i < prices.length; i++) {
        int newProfit0 = Math.max(profit0, profit1 + prices[i]); // 休息或买入
        int newProfit1 = Math.max(profit1, profit0 - prices[i]); // 休息或卖出
        profit0 = newProfit0;
        profit1 = newProfit1;
    }
    
    return profit0;
}
```

<!-- tabs:end -->

这个解法提供了获得最大收益的贪心策略：在可能的情况下，在每个局部最小值买入股票，然后在之后遇到的第一个局部最大值卖出股票。这个做法等价于找到股票价格中的递增子数组，对于每个递增子数组，在开始位置买入，并在结束位置卖出。可以看到，只要这样的操作收益为正，这和累计收益是相同的。

### 最多交易两次

此种情况对应的是[123. 买卖股票的最佳时机 III](https://leetcode-cn.com/problems/best-time-to-buy-and-sell-stock-iii/)，每天的交易结果有四个未知变量：`T[i][1][0], T[i][1][1], T[i][2][0], T[i][2][1]`，这种情况下的状态转移方程如下所示：

- `T[i][1][0] = max(T[i - 1][1][0], T[i - 1][1][1] + prices[i])`：休息/卖出
- `T[i][1][1] = max(T[i - 1][1][1], T[i - 1][0][0] - prices[i]) = max(T[i - 1][1][1], -prices[i])`：休息/买入
- `T[i][2][0] = max(T[i - 1][2][0], T[i - 1][2][1] + prices[i])`：休息/卖出
- `T[i][2][1] = max(T[i - 1][2][1], T[i - 1][1][0] - prices[i])`：休息/买入

其中，第二个表达式利用了前文提到的 `T[i][0][0] = 0` 这个条件，根据上面的状态定义与状态转移方程，可以写出时间复杂度和空间复杂度均为 $O(n)$ 的原始解法，以及进行空间压缩后的空间复杂度为 $O(1)$ 的解法。

<!-- tabs:start -->

#### **原始解法**

```java
public int maxProfit(int[] prices) {
    if (prices == null || prices.length == 0) {
        return 0;
    }
    // 状态定义
    int[][][] dp = new int[prices.length][3][2];
    // 基准条件
    dp[0][1][0] = 0;
    dp[0][1][1] = -prices[0];
    dp[0][2][0] = 0; // 第 0 天，交易 2 次后手持股票为 0，其实只能是两次交易都是休息。
    dp[0][2][1] = -prices[0]; // 第 0 天，只能是一次休息，一次买入但未卖出。
    // 状态转移
    for (int i = 1; i < prices.length; i++) {
        dp[i][2][0] = Math.max(dp[i - 1][2][0], dp[i - 1][2][1] + prices[i]);
        dp[i][2][1] = Math.max(dp[i - 1][2][1], dp[i - 1][1][0] - prices[i]);
        dp[i][1][0] = Math.max(dp[i - 1][1][0], dp[i - 1][1][1] + prices[i]);
        dp[i][1][1] = Math.max(dp[i - 1][1][1], -prices[i]);
    }
    
    return dp[prices.length - 1][2][0];
}
```

#### **空间压缩解法**

```java
public int maxProfit(int[] prices) {
    if (prices == null || prices.length == 0) {
        return 0;
    }
    int profit10 = 0;
    int profit11 = -prices[0];
    int profit20 = 0;
    int profit21 = -prices[0];
    for (int i = 1; i < prices.length; i++) {
        profit20 = Math.max(profit20, profit21 + prices[i]);
        profit21 = Math.max(profit21, profit10 - prices[i]);
        profit10 = Math.max(profit10, profit11 + prices[i]);
        profit11 = Math.max(profit11, -prices[i]);
    }
    return profit20;
}
```

<!-- tabs:end -->

### 最多交易 K 次

此种情况对应的是[188. 买卖股票的最佳时机 IV](https://leetcode-cn.com/problems/best-time-to-buy-and-sell-stock-iv/)，也是最通用的情况，对于每天持有的 $0$ 份或 $1$ 份股票，都需要使用各个 $k$ 值来更新所能获得的最大收益。

需要注意的是，当 $k$ 超过某个临界值时，最大收益将不再取决于交易次数，而是取决于股票数组的长度，那这个与股票数组长度相关的临界值应该怎么求得呢？

一次有收益的交易至少需要两天（前一天买入，之后的某一天卖出，并且卖出价格高于买入价格。）如果数组长度为 $n$，则有收益的交易的数量最多为 $\frac{n}{2}$ 组，因此 $k \lt \frac{n}{2}$，如果 $k$ 不小于临界值，即 $k >= \frac{n}{2}$，则可以将 $k$ 扩展为正无穷，此时问题等价于不限制交易次数，与[不限制交易次数](#不限制交易次数)同解。

根据前文状态定义与状态转移方程，可以写出时间复杂度为 $O(nk)$ 和空间复杂度为 $O(nk)$ 的原始解法，已经空间压缩后的空间复杂度为 $O(k)$ 的解法。

<!-- tabs:start -->

#### **原始解法**

```java
public int maxProfit(int k, int[] prices) {
    if (prices == null || prices.length == 0) {
        return 0;
    }
    int length = prices.length;
    if (k >= length / 2) {
        return maxProfit(prices);
    }
    // 状态定义
    int[][][] dp = new int[length][k + 1][2];
    // 基准条件
    for (int i = 0; i <= k; i++) {
        // 第 0 天结束时持有 0 支股票，只有一种可能，就是休息。
        dp[0][i][0] = 0;
        // 第 0 天结束时持有 1 支股票，只有一种可能，就是买入。
        dp[0][i][1] = -prices[0];
    }
    
    // 状态转移
    for (int i = 1; i < length; i++) {
        for (int j = k; j > 0; j--) {
            // 休息或卖出
            dp[i][j][0] = Math.max(dp[i - 1][j][0], dp[i - 1][j][1] + prices[i]);
            // 休息或买入
            dp[i][j][1] = Math.max(dp[i - 1][j][1], dp[i - 1][j - 1][0] - prices[i]);
        }
    }
    
    return dp[length - 1][k][0];
}

public int maxProfit(int[] prices) {
    if (prices == null || prices.length == 0) {
        return 0;
    }
    int length = prices.length;
    int[][] dp = new int[length][2];
    dp[0][0] = 0;
    dp[0][1] = -prices[0];
    for (int i = 1; i < length; i++) {
        dp[i][0] = Math.max(dp[i - 1][0], dp[i - 1][1] + prices[i]);
        dp[i][1] = Math.max(dp[i - 1][1], dp[i - 1][0] - prices[i]);
    }
    return dp[length - 1][0];
}
```

#### **空间压缩解法**

```java
public int maxProfit(int k, int[] prices) {
    if (prices == null || prices.length == 0) {
        return 0;
    }
    int length = prices.length;
    if (k >= length / 2) {
        return maxProfit(prices);
    }
    // 状态定义：k 小于临界值时，收益主要取决于交易次数，而不是数组长度。
    int[][] dp = new int[k + 1][2];
    // 基准条件
    for (int i = 1; i <= k; i++) {
        dp[i][0] = 0;
        dp[i][1] = -prices[0];
    }
    for (int i = 1; i < length; i++) {
        for (int j = k; j > 0; j--) {
            dp[j][0] = Math.max(dp[j][0], dp[j][1] + prices[i]);
            dp[j][1] = Math.max(dp[j][1], dp[j - 1][0] - prices[i]);
        }
    }
    return dp[k][0];
}

private int maxProfit(int[] prices) {
    int profit0 = 0;
    int profit1 = -prices[0];
    for (int i = 1; i < prices.length; i++) {
        int newProfit0 = Math.max(profit0, profit1 + prices[i]);
        int newProfit1 = Math.max(profit1, profit0 - prices[i]);
        profit0 = newProfit0;
        profit1 = newProfit1;
    }
    return profit0;
}
```

<!-- tabs:end -->

### 不限制交易次数但有冷冻期

这种情况属于[309. 最佳买卖股票时机含冷冻期](https://leetcode-cn.com/problems/best-time-to-buy-and-sell-stock-with-cooldown/)，与[不限制交易次数](#不限制交易次数)类似，但是多了一个限制条件：**卖出股票后需要等待一天才能重新买入股票**，因此需要对状态转移方程稍作修改。

[不限制交易次数](#不限制交易次数)情况下的转移方程为：

- `T[i][k][0] = max(T[i-1][k][0], T[i-1][k][1] + prices[i])`
- `T[i][k][1] = max(T[i-1][k][1], T[i-1][k][0] - prices[i])`

在存在**冷却时间**的条件下，如果第 `i-1` 天卖出了股票，就不能在第 `i` 天买入股票，因此如果要在第 `i` 天买入股票，第二个转移方程中就不能使用 `T[i-1][k][0]`，而应该使用 `T[i-2][k][0]`，其他各项保持不变。

- `T[i][k][0] = max(T[i-1][k][0], T[i-1][k][1] + prices[i])`
- `T[i][k][1] = max(T[i-1][k][1], T[i-2][k][0] - prices[i])`

根据此状态转移方程，可写出时间复杂度与空间复杂度均为 $O(n)$ 的原始解法以及进行空间压缩后的空间复杂度为 $O(1)$ 的解法。

<!-- tabs:start -->

#### **原始解法**

```java
public int maxProfit(int[] prices) {
    if (prices == null || prices.length == 0) {
        return 0;
    }
    int[][] dp = new int[prices.length][2];
    dp[0][0] = 0;
    dp[0][1] = -prices[0];
    for (int i = 1; i < prices.length; i++) {
        dp[i][0] = Math.max(dp[i - 1][0], dp[i - 1][1] + prices[i]);
        dp[i][1] = Math.max(dp[i - 1][1], (i >= 2 ? dp[i - 2][0] : 0) - prices[i]);
    }
	return dp[prices.length - 1][0];
}
```

#### **空间压缩解法**

```java
public int maxProfit(int[] prices) {
    if (prices == null || prices.length == 0) {
        return 0;
    }
    int profit0Prev = 0;
    int profit0 = 0;
    int profit1 = -prices[0];
    
    for (int i = 1; i < prices.length; i++) {
        int nextProfit0 = Math.max(profit0, profit1 + prices[i]);
        int nextProfit1 = Math.max(profit1, profit0Prev - prices[i]);
        profit0Prev = profit0;
        profit0 = nextProfit0;
        profit1 = nextProfit1;
    }
    return profit0;
}
```

<!-- tabs:end -->

### 不限制交易次数但有手续费

这种情况属于[714. 买卖股票的最佳时机含手续费](https://leetcode-cn.com/problems/best-time-to-buy-and-sell-stock-with-transaction-fee/)，与[不限制交易次数](#不限制交易次数)相同之处是不限制交易次数，不同之处是眉笔交易需要**手续费**，因此需要对状态转移方程做一些修改。

[不限制交易次数](#不限制交易次数)情况下的转移方程为：

- `T[i][k][0] = max(T[i-1][k][0], T[i-1][k][1] + prices[i])`
- `T[i][k][1] = max(T[i-1][k][1], T[i-1][k][0] - prices[i])`

因为需要对每笔交易付手续费，而每次**买入+卖出**这个完整的流程才算是一次交易，所以可以在**买入**时付手续费，也可以在**卖出**时付手续费，所以状态转移方程有两种表示方式。

第一种表示方式，在每次买入股票时扣除手续费：

- `T[i][k][0] = max(T[i-1][k][0], T[i-1][k][1] + prices[i])`
- `T[i][k][1] = max(T[i-1][k][1], T[i-1][k][0] - prices[i] - fee)`

第二种表示方式，在每次卖出股票时扣除手续费：

- `T[i][k][0] = max(T[i-1][k][0], T[i-1][k][1] + prices[i])`
- `T[i][k][1] = max(T[i-1][k][1], T[i-1][k][0] - prices[i] + fee)`

根据状态定义与状态转移方程，可以写出时间复杂度和时间复杂度均为 $O(n)$ 的原始解法以及空间压缩后空间复杂度为 $O(1)$ 的解法。

<!-- tabs:start -->

#### **方案一原始解法**

```java
public int maxProfit(int[] prices, int fee) {
    if (prices == null || prices.length == 0) {
        return 0;
    }
    int[][] dp = new int[prices.length][2];
    dp[0][0] = 0;
    dp[0][1] = -prices[0] - fee;
    for (int i = 1; i < prices.length; i++) {
        dp[i][0] = Math.max(dp[i - 1][0], dp[i - 1][1] + prices[i]);
        dp[i][1] = Math.max(dp[i - 1][1], dp[i - 1][0] - prices[i] - fee);
    }
	return dp[prices.length - 1][0];
}
```

#### **方案一压缩解法**

```java
public int maxProfit(int[] prices, int fee) {
    if (prices == null || prices.length == 0) {
        return 0;
    }
	int profit0 = 0;
    int profit1 = -prices[0] - fee;
    
    for (int i = 1; i < prices.length; i++) {
        int nextProfit0 = Math.max(profit0, profit1 + prices[i]);
        int nextProfit1 = Math.max(profit1, profit0 - prices[i] - fee);
        profit0 = nextProfit0;
        profit1 = nextProfit1;
    }
    
	return profit0;
}
```

#### **方案二原始解法**

```java
public int maxProfit(int[] prices, int fee) {
    if (prices == null || prices.length == 0) {
        return 0;
    }
    int[][] dp = new int[prices.length][2];
    dp[0][0] = 0;
    dp[0][1] = -prices[0]; // 卖出时才付手续费
    for (int i = 1; i < prices.length; i++) {
        dp[i][0] = Math.max(dp[i - 1][0], dp[i - 1][1] + prices[i] - fee);
        dp[i][1] = Math.max(dp[i - 1][1], dp[i - 1][0] - prices[i]);
    }
	return dp[prices.length - 1][0];
}
```

#### **方案二压缩解法**

```java
public int maxProfit(int[] prices, int fee) {
    if (prices == null || prices.length == 0) {
        return 0;
    }
	int profit0 = 0;
    int profit1 = -prices[0];
    
    for (int i = 1; i < prices.length; i++) {
        int nextProfit0 = Math.max(profit0, profit1 + prices[i] - fee);
        int nextProfit1 = Math.max(profit1, profit0 - prices[i]);
        profit0 = nextProfit0;
        profit1 = nextProfit1;
    }
    
	return profit0;
}
```

<!-- tabs:end -->

## 总结

总而言之，股票问题的最通用情况由三个特征决定，即天数 `i`，允许的最大交易次数 `k` 以及每天结束时持有的股票数 `0 OR 1`。这篇文章阐述了最大利润的状态定义、基准条件、以及状态转移方程，并由此编写了原始解法与进行空间压缩后的解法，并在特殊场景下对原始的状态转移方程进行了微调以解决相应的问题。

> 原始内容与其他扩展阅读信息请参考本文声明中所列出的原文
