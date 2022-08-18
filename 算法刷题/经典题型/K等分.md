# 经典K等分

背包的加强版（两等分）

**有两种解法 一种是回溯剪枝，一种是状态压缩**

## [1723. 完成所有工作的最短时间](https://leetcode.cn/problems/find-minimum-time-to-finish-all-jobs/)

> 回溯
>
> 建立k个桶 往桶中放值，尽量最均匀



注意剪枝 ： 

1. 优先分配空闲工人
2. 在for中只需要在 已经分配的工人中回溯， **对于没有分配的工人都是一样的**，不需要使用for遍历
3. 从大到小排序
4. 如果 支路的值已经大于res 直接跳过
5. 确保 第一个job 只分配给 第一个工人，保证递归树的根只有一个，其实第一二已经保证了这一点

```go
func minimumTimeRequired(jobs []int, k int) int {

    // 创建k个桶
    buk := make([]int, k)
    res := math.MaxInt64
    // 从大到小排序
    sort.Sort(sort.Reverse(sort.IntSlice(jobs)))
、

    var backtrace func(int, int)
    backtrace = func (index,used int){
        // 退出条件
        if index == len(jobs){
            // 找到最大值
            tmp := arrMax(buk)
            // 所有最大值中的最小值
            res = min(res, tmp)
            return 
        }
        // 最重要的剪枝 优先给每个工人分配一个 优先分配空闲工人
        if used < k {
            buk[used] += jobs[index]
            backtrace(index+1,used+1)
            buk[used] -= jobs[index]
        }
        // 注意这里一定是  < used 否则超时
        for i := 0; i < used; i++{
            // 因为第一个分配给哪个工人都是一样的结果
            // 剪枝  确定只分给第一个  保证递归树的根只有一个
            if index == 0 && i != 0{
                continue
            }
            // 剪枝 如果已经大于 res 这个支路就没必要继续下去了
            if buk[i] + jobs[index]  > res {
                continue
            }     
            buk[i] += jobs[index]
            backtrace(index+1,used)
            buk[i] -= jobs[index]
        }
    } 

    backtrace(0,0)    
    return res

}

func min(a, b int)int{
    if a > b {
        return b
    }
    return a
}
func max(a, b int)int{
    if a < b {
        return b
    }
    return a
}
func arrMax(arr []int)int{
    res := arr[0]
    for _, v := range arr{
        res = max(res, v)
    }
    return res
}
```



> 状态压缩dp

`dp[i][j]`: 表示 前`i`个工人为了完成作业子集 `j`，需要花费的最大工作时间的最小值

假设 第`i`个工人 分配的工作集合为 `s` 需要的时间是`sum[s]`

- 如果 `sum[s] > dp[i - 1][j - s]`说明给第`i`个工人分配的工作干比前面的孩子多，不公平程度变为`sum[s]`；
- 如果 `sum[s] < dp[i - 1][j - s]`说明给第`i`个工人分配的工作干比前面的孩子少，不公平程度不变；

重难点：

```
 for s := j; s > 0; s = (s - 1) & j {}
```

==从 `j`中找出所有组合可能==  

- 假设`j= 10` 是第2位和第4位  被选中   s 将会 遍历出 三种可能
- s = 10  第2位和第4位 
- s = 8 第4位 
- s = 2 第2位 

```
func minimumTimeRequired(jobs []int, k int) int {

    n := 1 << len(jobs)
    sum := make([]int, n)
    // 初始化 求出所有的组合的值
    for i, v := range jobs {
        for j, bit := 0, 1<<i; j < bit; j++{
            sum[bit|j] = sum[j] + v
        }
    }
    
    dp := append([]int{}, sum...)
    
    for i := 1; i < k; i++{
        for j := n - 1; j > 0; j--{
        //遍历所有的j的集合
            for s := j; s > 0; s = (s - 1) & j {
				dp[j] = min(dp[j], max(dp[j^s], sum[s]))
			}
        }
    }
    return dp[n-1]
}

func min(a, b int) int { if a > b { return b }; return a }
func max(a, b int) int { if b > a { return b }; return a }
```



**类似题目**

## [473. 火柴拼正方形](https://leetcode.cn/problems/matchsticks-to-square/)

> 微小的区别 ： 桶的个数固定为 4
>
> 只需要返回 true / false 

```
func makesquare(matchsticks []int) bool {
    totalLen := 0
    for _, l := range matchsticks {
        totalLen += l
    }
    if totalLen%4 != 0 {
        return false
    }
    sort.Sort(sort.Reverse(sort.IntSlice(matchsticks))) // 减少搜索量

    edges := [4]int{}
    var dfs func(int, int) bool
    dfs = func(idx, used int) bool {
        if idx == len(matchsticks) {
            return true
        }
        if used < 4{
            edges[used] += matchsticks[idx]
            if edges[used] <= totalLen/4 && dfs(idx+1, used+1) {
                return true
            }
            edges[used] -= matchsticks[idx]
        }
        for i := 0; i < used; i ++  {
            edges[i] += matchsticks[idx]
            if edges[i] <= totalLen/4 && dfs(idx+1, used) {
                return true
            }
            edges[i] -= matchsticks[idx]
        }
        return false
    }
    return dfs(0, 0)
}

```



