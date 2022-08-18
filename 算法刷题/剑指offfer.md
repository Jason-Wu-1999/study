#  剑指offer

 考察的是什么解决了什么问题

## [剑指 Offer 09. 用两个栈实现队列](https://leetcode-cn.com/problems/yong-liang-ge-zhan-shi-xian-dui-lie-lcof/)

> 双栈 其中栈用数组模拟

注意点

- 在出栈的时候注意判断是否栈中是否为空值    ==数组在删除元素之前一定要判断长度  防止下标越界==

```go
type CQueue struct {
    stack1 stack
    stack2 stack
}

type stack []int
//前面的括号内只能是自定义类型 
func (s *stack)push(a int){
    *s=append(*s,a)
} 

func (s *stack)pop()(res int){
    n:=len(*s)
    res=(*s)[n-1]
    *s=(*s)[:n-1]
    return 
}

func Constructor() CQueue {
    return CQueue{}
}


func (this *CQueue) AppendTail(value int)  {
        this.stack1.push(value)
}

func (this *CQueue) DeleteHead() int {
    
    if len( this.stack2)==0{
        for len( this.stack1)>0{
            this.stack2.push(this.stack1.pop())
        }
    }
    if len( this.stack2)!= 0{
        e:=this.stack2.pop()    
        return e
    }
    return -1
}
```

> 库函数实现栈    *list.List

```go
type CQueue struct {
    stack1,stack2 *list.List
}


func Constructor() CQueue {
    return CQueue{
        stack1:list.New(),
        stack2:list.New(),
    }
}


func (this *CQueue) AppendTail(value int)  {
        this.stack1.PushBack(value)
}


func (this *CQueue) DeleteHead() int {
    if this.stack2.Len()==0{
        for this.stack1.Len()>0{
            this.stack2.PushBack(this.stack1.Remove(this.stack1.Back()))
        }
    }
    if this.stack2.Len() != 0{
        e:=this.stack2.Back()
        this.stack2.Remove(e)
        return  e.Value.(int)
    }
    return -1
}

```

## [剑指 Offer 30. 包含min函数的栈](https://leetcode-cn.com/problems/bao-han-minhan-shu-de-zhan-lcof/)

> 辅助栈  创建一个辅助栈栈顶始终存放最小值

注意：尽管新入栈的值不是栈中的最小值，在辅助栈中栈顶也要压入一个栈的最小值（重复之前的）否则在pop时两个栈对应不上

```go
type MinStack struct {
    stack []int
    min []int 
}


/** initialize your data structure here. */
func Constructor() MinStack {
    return MinStack{
        stack :[]int{},
        min:[]int{},
    }
}


func (this *MinStack) Push(x int)  {
    this.stack=append(this.stack,x)

    if len(this.min)==0 || this.min[len(this.min)-1]>x {
        this.min=append(this.min,x)
    }else{
        this.min=append(this.min,this.min[len(this.min)-1])
    }
}

func (this *MinStack) Pop()  {
    n:=len(this.stack)
    if n==0{
        
    }
    this.stack=this.stack[:n-1]
    this.min=this.min[:n-1]   
}


func (this *MinStack) Top() int {
    n:=len(this.stack)
    if n==0{
        return 0
    }
    return this.stack[n-1]
}


func (this *MinStack) Min() int {
       if len(this.min) > 0 {
        return this.min[len(this.min)-1]
    }
    return 0

}



```





## [剑指 Offer 06. 从尾到头打印链表](https://leetcode-cn.com/problems/cong-wei-dao-tou-da-yin-lian-biao-lcof/)

> 反转数组  只有反向打印没必要去真正的反转数组 

- 先将链表存入数组

- 交换数组元素

  ```
  func reversePrint(head *ListNode) []int {
      if head==nil{
          return nil
      }
      
  // 将链表的值放入切片中，反向遍历切片
      var res []int
      for head!=nil{
          res=append(res,head.Val)
          head=head.Next
      }
      
  //交换数组元素
      for i,j:=0,len(res)-1;i<j;{
          res[i],res[j]=res[j],res[i]
          i++
          j--
      }
  
      return res
  
  
  }
  ```


> 递归（栈的本质是递归）

```go
func reversePrint(head *ListNode) []int {
    if head==nil{
        return nil
    }
   return  append(reversePrint(head.Next),head.Val)
}
```

递归主体前部分是先执行，后部分递归完成后回溯执行



## [剑指 Offer 24. 反转链表](https://leetcode-cn.com/problems/fan-zhuan-lian-biao-lcof/)

> 迭代

<img src="https://pic.leetcode-cn.com/1624782858-oyJziv-008eGmZEly1gnrf1oboupg30gy0c44qp.gif" alt="img" style="zoom: 60%;" />

```go
func reverseList(head *ListNode) *ListNode {
    var prev *ListNode
    curr := head
    for curr != nil {
        next := curr.Next
        curr.Next = prev
        // 下两句为更新指针
        prev = curr
        curr = next
    }
    return prev
}
```

## [剑指 Offer 35. 复杂链表的复制](https://leetcode-cn.com/problems/fu-za-lian-biao-de-fu-zhi-lcof/)

注意点:

- 要注意保证原本的链表不变，改变的话就要还原



> map

```
创建一个map[*node]*node
```

利用key值和value均是*node   key中存放的是原本的节点，value中存放的是新建复制的节点

```
dic[cur].Next=dic[cur.Next]        

dic[cur].Random=dic[cur.Random]
```

可找到next和random

```go
func copyRandomList(head *Node) *Node {
    if head==nil{
        return nil
    }
     dic :=make(map[*Node]*Node)
    cur:=head
    for cur!=nil{
        dic[cur]=&Node{
            Val:cur.Val,
        }
        cur=cur.Next   
    }

    cur=head
    for cur!=nil{
        dic[cur].Next=dic[cur.Next]
        dic[cur].Random=dic[cur.Random]
        cur=cur.Next
    }

    return dic[head]
}
```



> 拼接和拆分

1. 考虑构建 `原节点 1 -> 新节点 1 -> 原节点 2 -> 新节点 2 -> ……` 的拼接链表
2. 可用cur.next.random=cur.random.next   确定新节点的random指针
3. 将链表拆分即可，注意拆分时需要保持原链表不变
4. 注意节点的实时变换

**难点在于要知道拆分时要保持原链表不变**

```go
/**
 * Definition for a Node.
 * type Node struct {
 *     Val int
 *     Next *Node
 *     Random *Node
 * }
 */

func copyRandomList(head *Node) *Node {
    if head==nil{
        return nil
    }
    //对原链表进行复制并插入原链表中,之后指定新插入链表的Next和random指针,最后将其拆分出两个链表即可;
    cur:=head
    for cur!=nil{
      newNode:=&Node{
          Val:cur.Val  ,
      }
        newNode.Next=cur.Next
        cur.Next=newNode
        cur=newNode.Next
    }
 //指定下插入链表的random指针指向
    cur=head
    for cur!=nil{
       if cur.Random!=nil{
           cur.Next.Random=cur.Random.Next
       }
       cur=cur.Next.Next
    }
    //拆分
    cur = head.Next
    ret := cur 
    pre := head 
    for cur.Next != nil{
        pre.Next = pre.Next.Next
        cur.Next = cur.Next.Next 
        pre = pre.Next 
        cur = cur.Next

    }
    pre.Next = nil 
    return ret
}
```

## 

# 第五天

## [剑指 Offer 04. 二维数组中的查找](https://leetcode-cn.com/problems/er-wei-shu-zu-zhong-de-cha-zhao-lcof/)

> 剪枝

- 若是从最小值或者最大值开始查找相对来说会比较慢，故选择从右上角或者左下角开始

```
func findNumberIn2DArray(matrix [][]int, target int) bool {
   if len(matrix) == 0 {
		return false
	}
    m,n:=len(matrix),len(matrix[0])

    i,j:=0,n-1
    for i<m && j>=0{
        if matrix[i][j]< target{
            i++
        }else if matrix[i][j] > target{
            j--
        }else {
            return true
        }
    } 
    return false

}
```



## [剑指 Offer 11. 旋转数组的最小数字](https://leetcode-cn.com/problems/xuan-zhuan-shu-zu-de-zui-xiao-shu-zi-lcof/)

> 二分查找   特别注意临界点的处理

易错点：

- 有可能存在原本就是单调的序列的情况，二分法要选**最右端**元素作为基准
- 当中间值小于最右端值时，不能把他跳过  只能 ` r=mid`
- ==等于时的处理==，出现等于号时无法判断最小值在那一段   如 [2 2 2 2 0 2]和[2 0 2 2 2 2 ],最小值可能在右边也可能在左边，所以出现等于号时，只能将最右端的往左移动一位

```

func minArray(numbers []int) int {
    l,r :=0,len(numbers)-1

    for l<r{
        mid:=l+(r-l)/2
        if numbers[r]<numbers[mid]{
            l=mid+1
        }else if numbers[r]>numbers[mid]{
            r=mid
        }else {
          r--
        }
    }
    return numbers[l]
}
```

## [剑指 Offer 50. 第一个只出现一次的字符](https://leetcode-cn.com/problems/di-yi-ge-zhi-chu-xian-yi-ci-de-zi-fu-lcof/)

> map

注意：若没有将s转换为【】byte类型  for range 出来的结果 v不是 byte类型的

```
func firstUniqChar(s string) byte {
    ss:=[]byte(s)
    if len(ss)==0{
        return ' '
    }
    m:=make(map[byte]bool,len(ss))

    for _,v:=range ss{
        if _,ok:=m[v]; ok{
            m[v]=true
        } else{
              m[v]=false
        }
      
    }
     for _,v:=range ss{
        if a,_:=m[v]; a==false{
            return v
        } 
      
    } 
    return ' '
}
```

# 第六天

## [面试题32 - I. 从上到下打印二叉树](https://leetcode-cn.com/problems/cong-shang-dao-xia-da-yin-er-cha-shu-lcof/)

> 层序遍历

```
/**
 * Definition for a binary tree node.
 * type TreeNode struct {
 *     Val int
 *     Left *TreeNode
 *     Right *TreeNode
 * }
 */
func levelOrder(root *TreeNode) []int {
    // 层序遍历
    if root==nil{
        return []int{}
    }
    queue:=[]*TreeNode{root}
    res:=make([]int,0)
    for len(queue)>0{
        cur:=queue[0]
        queue=queue[1:]
        res=append(res,cur.Val)

        if cur.Left!=nil{
            queue=append(queue,cur.Left)
        }
        if cur.Right!=nil{
            queue=append(queue,cur.Right)
        }

    }
    return res
}
```



## [剑指 Offer 32 - II. 从上到下打印二叉树 II](https://leetcode-cn.com/problems/cong-shang-dao-xia-da-yin-er-cha-shu-ii-lcof/)

> 在层序遍历的基础上  再加一层循环
>
> 先建一个临时的数组保存结果，当一层遍历完之后，再append到结果数组

```
/**
 * Definition for a binary tree node.
 * type TreeNode struct {
 *     Val int
 *     Left *TreeNode
 *     Right *TreeNode
 * }
 */
func levelOrder(root *TreeNode) (res[][]int) {
    if root==nil{
        return [][]int{}
    }
    queue:=[]*TreeNode{root}
  
    for len(queue)>0{
        q1:=make([]*TreeNode,0)
        ans:=make([]int,0)
        for len(queue)>0{
            cur:=queue[0]
            queue=queue[1:]

            ans=append(ans,cur.Val)

            if cur.Left!=nil{
                q1=append(q1,cur.Left)
            }
            if cur.Right!=nil{
                q1=append(q1,cur.Right)
            }
        }
        res=append(res,ans)
        queue=q1
        

    }
    return res
}
```

## [剑指 Offer 32 - III. 从上到下打印二叉树 III](https://leetcode-cn.com/problems/cong-shang-dao-xia-da-yin-er-cha-shu-iii-lcof/)

> 在II的基础上进行分奇偶

- 需要反向从队列中取值
- 根据奇偶 判断 是先遍历左子节点还是右子节点

**判断奇偶常见的两种方法** 

1. num%2
2. num&1

```
/**
 * Definition for a binary tree node.
 * type TreeNode struct {
 *     Val int
 *     Left *TreeNode
 *     Right *TreeNode
 * }
 */
func levelOrder(root *TreeNode) [][]int {
    var res [][]int
    if root==nil{
        return [][]int{}
    }
    queue:=[]*TreeNode{root}
     level:=0
    for len(queue)>0{

        q1:=make([]*TreeNode,0)
        ans:=make([]int,0)
        //反着从队列中取值
        for i:=len(queue)-1;i>=0;i--{
            cur:=queue[i]
            queue=queue[:i]

            ans=append(ans,cur.Val)
			//偶数
            if level&1==0{
                if cur.Left!=nil{
                    q1=append(q1,cur.Left)
                }
                if cur.Right!=nil{
                    q1=append(q1,cur.Right)
                }
            }else{
                 if cur.Right!=nil{
                    q1=append(q1,cur.Right)
                }
                if cur.Left!=nil{
                    q1=append(q1,cur.Left)
                }
            }
        }
        res=append(res,ans)
        level++
        queue=q1
        

    }
    return res
}
```

# 第七天

## [剑指 Offer 26. 树的子结构](https://leetcode-cn.com/problems/shu-de-zi-jie-gou-lcof/)

> 递归 dfs，也可以使用bfs

解题思路：
若树 B 是树 A 的子结构，则子结构的根节点可能为树 A 的任意一个节点。因此，判断树 B 是否是树 A 的子结构，需完成以下两步工作：

- 先序遍历树 A中的每个节点 n_A（对应函数 isSubStructure(A, B)）
- 判断树 A 中 以 n_A为根节点的子树 是否包含树 B 。对应函数 recur(A, B)）

<img src="https://cdn.jsdelivr.net/gh/Jason-Wu-1999/blog.img/imgs/27d9f65b79ae4982fb58835d468c2a23ec2ac399ba5f38138f49538537264d03-Picture1.png" alt="Picture1.png" style="zoom:50%;" />



```go \
func isSubStructure(A *TreeNode, B *TreeNode) bool {
    if B==nil || A==nil{
        return false
    }
    var res bool
    if A.Val==B.Val{
       res= compare(A,B)
    }
    //只要一个条件满足即可  所以使用  &&
    return  res || isSubStructure(A.Left,B)  || isSubStructure(A.Right,B)
}

func compare(A *TreeNode, B *TreeNode) bool{
     if B==nil {
        return true
    }

    if A==nil || A.Val!=B.Val{
        return false
    }
    /需要两个条件同时满足所以使用  &&
    return  compare(A.Left,B.Left) &&  compare(A.Right,B.Right)
}
```



## [剑指 Offer 27. 二叉树的镜像](https://leetcode-cn.com/problems/er-cha-shu-de-jing-xiang-lcof/)

> dfs(递归)：递归在处理链表和二叉树时很妙，因为可以回溯，可以让他不需要定义临时节点遍历

```go
func mirrorTree(root *TreeNode) *TreeNode {
    if root == nil {
        return nil
    }  
    
    mirrorTree(root.Left)                          //递归左子树 (压栈压到底部)
    mirrorTree(root.Right)                         //递归右子树
    root.Left, root.Right = root.Right, root.Left //交换左右子树
                         
    return root
}
--------------------------------------------------------------------------------------
func mirrorTree(root *TreeNode) *TreeNode {
    if root == nil {
        return nil
    }  
    
    root.Left, root.Right = root.Right, root.Left //交换左右子树
    mirrorTree(root.Left)                          //递归左子树 (压栈压到底部)
    mirrorTree(root.Right)                         //递归右子树
  
                         
    return root
}


```

> bfs需要栈辅助 

```go
func mirrorTree(root *TreeNode) *TreeNode {
    if root == nil {
        return nil
    }  
    // bfs 定义辅助栈遍历
    //  var queue [] *TreeNode 
    // queue=append(queue,root)

    queue:= [] *TreeNode{
        root,
    } 

    for len(queue)!=0{
        tmp:=queue[0]
        tmp.Left, tmp.Right = tmp.Right, tmp.Left //交换左右子树
        if tmp.Left!=nil{
            queue=append(queue,tmp.Left)
        }

        if tmp.Right!=nil{
            queue=append(queue,tmp.Right)
        }

        queue=queue[1:]
    }
                     
    return root
}
```

## [剑指 Offer 28. 对称的二叉树](https://leetcode-cn.com/problems/dui-cheng-de-er-cha-shu-lcof/)

> 递归

**对称二叉树：**对于树中 任意两个对称节点 L和 R ，一定有：
L.val = R.val ：即此两对称节点值相等。
L.left.val = R.right.val ：即 L的 左子节点 和 R 的 右子节点 对称；
L.right.val = R.left.val：即 L 的 右子节点 和 R 的 左子节点 对称。



<img src="https://i.loli.net/2021/08/06/YML7GAxCFe2ogua.png" alt="Picture1.png" style="zoom:50%;" />

==递归三部曲：==

1. **确定参数和返回值**

   ```go
   compare（left,right *TreeNode）bool
   ```

2. **终止条件：**

- 若 **l 和 r 均为空**则说明递归到底了，返回true
- 若 一个为空，一个不为显然不对称返回false
- 都不为空且值不相等，不对称返回false


3. **递推工作：**

- 判断两节点 L.left 和 R.right 是否对称，即 compare(L.left, R.right) ；
- 判断两节点 L.right 和 R.left 是否对称，即 compare(L.right, R.left) ；
- 返回值： 两对节点都对称时，才是对称树，因此用与逻辑符 && 连接。

==易错点：==

`if (left==nil &&right!=nil)  || (left!=nil &&right==nil) || (left.Val != right.Val)`    ==这句话 判断不为空必须在前否则会空指针==

```
func isSymmetric(root *TreeNode) bool {
   if root==nil{
       return true
   }

   return compare(root.Left,root.Right)
}

func compare(left,right *TreeNode)bool{
    if left==nil &&right==nil {
        return true
    }

    if (left==nil &&right!=nil)  || (left!=nil &&right==nil) || (left.Val != right.Val){
        return false
    }
    
    //需要两个条件同时满足所以使用  &&
    return compare(left.Left,right.Right) && compare(right.Left ,left.Right)

}
```

# 第八天

## [剑指 Offer 10- I. 斐波那契数列](https://leetcode-cn.com/problems/fei-bo-na-qi-shu-lie-lcof/)

> for循环实现  dp
>
> 注意是否取等号，细节



```go
func fib(n int) int {
    a,b:=0,1
    for i:=1;i<=n;i++{
        a,b=b,(a+b)%1000000007
    }
    return a
}
```

## [剑指 Offer 10- II. 青蛙跳台阶问题](https://leetcode-cn.com/problems/qing-wa-tiao-tai-jie-wen-ti-lcof/)

> 同上，只是初始值不一样
>
> 每加一级台阶，可以分解为 跳一级 f(n-1), 或者跳两级  f(n-2)
>
> 若是可以跳m 阶   转化为背包问题

```go
func numWays(n int) int {
     a,b:=1,1
    for i:=1;i<=n;i++{
        a,b=b,(a+b)%1000000007
    }
    return a
}
```

### 拓展题

==若是每次最多可以跳m阶     转化为完全背包问题==

n表示容量  1--m表示物品    每个物品可以无限次数    装满n容量的次数

```
func climbStairs(n int) int {
	//定义
	dp := make([]int, n+1)
	//初始化
	dp[0] = 1
	// 本题物品只有两个1,2
	m := 2
	// 遍历顺序
	for j := 1; j <= n; j++ {	//先遍历背包
		for i := 1; i <= m; i++ {	//再遍历物品
			if j >= i {
				dp[j] += dp[j-i]
			}
			//fmt.Println(dp)
		}
	}
	return dp[n]
}
```

## [剑指 Offer 63. 股票的最大利润](https://leetcode-cn.com/problems/gu-piao-de-zui-da-li-run-lcof/)

>  动态规划  dp[i]=max( dp[i-1] , prices[i]-minPrices )

```go
func maxProfit(prices []int) (res int) {
    if len(prices)<=1{
        return
    }
  minPrices:=prices[0]
    dp:=make([]int,len(prices))
    dp[0]=0
    for i:=1;i<len(prices);i++{
        dp[i]=max(dp[i-1],prices[i]-minPrices)
        if res<dp[i]{
            res=dp[i]
        }
        if minPrices>prices[i]{
            minPrices=prices[i]
        }
    }
    return 
}

func max(a,b int)int{
    if a>b{
        return a
    }
    return b
}
```

> 维护一个最小值

- 用当前的值前去当前维护的最小值，就是当前时候卖出的最大利润
- 最小值从左往右实时维护
- 从i的利润里面找出最大值

```go
func maxProfit(prices []int) int {
    
    if len(prices)==1{
        return 0
    }
  
    min:=prices[0]
    ans:=0
    for i:=1;i<len(prices);i++{
        tmp:=prices[i]-min
        if tmp>ans{
           ans=tmp
        }
        if min>prices[i]{
            min=prices[i]
        }
    }
    return ans
}
```



# 第九天

## [剑指 Offer 42. 连续子数组的最大和](https://leetcode-cn.com/problems/lian-xu-zi-shu-zu-de-zui-da-he-lcof/)

> 动态规划

注意：可以直接在原数组上改动，进一步减小空间复杂度。

```
func maxSubArray(nums []int) int {
    // dp:=make([]int,len(nums)) 
    // dp[0]=nums[0]
    max:=nums[0]
    for i:=1;i<len(nums);i++{
        if nums[i-1]>0{
            nums[i]+=nums[i-1]
        }
        
        if nums[i]>max{
            max=nums[i]
        }

    }

    return max

}
```



## [剑指 Offer 47. 礼物的最大价值](https://leetcode-cn.com/problems/li-wu-de-zui-da-jie-zhi-lcof/)

> 动态规划 +改原数组

当前礼物最大值：

```go
 grid[i][j]+=max(grid[i-1][j],grid[i][j-1])
```

注意点：

- **在原数组上改动**
- ==需要考虑第一行第一列单独赋值，否则越界==

```go 
func maxValue(grid [][]int) int {
    m,n:=len(grid),len(grid[0])
    // 取较大值
    max := func(a int, b int)int{
        if a>b{
            return a
        }
        return b
    }
    for i:=0;i<m;i++{
        for j:=0;j<n;j++{
            if i==0 && j==0{
                continue
            }else if i==0{
                grid[i][j]+=grid[i][j-1]
            }else if j==0{
                grid[i][j]+=grid[i-1][j]
            }else {
                grid[i][j]+=max(grid[i-1][j],grid[i][j-1])
            }
        }
    }
    return grid[m-1][n-1]
}
```

# 第十天

## [剑指 Offer 46. 把数字翻译成字符串](https://leetcode-cn.com/problems/ba-shu-zi-fan-yi-cheng-zi-fu-chuan-lcof/)

> 动态规划
>
> 类似青蛙跳台阶 只能跳一个或者跳两个
>
> 这里只能一个数字或者两个数字组合，而且需要判断是否满足大于9小于26



==注意点：== 

- **需要取出最后两位数字比较，但不能使其相加** 故有两种做法 
  - 一种是转化为字符串进行截取比较
  - 一种是 将数字对100取余，直接保留最后两位
- 需要两位数大于9是因为两位数  例如 05、06不能翻译

==难点：如何取后两位而使其不相加==

```go
func translateNum(num int) int {
    str :=strconv.Itoa(num)
    if len(str)<2{
        return 1
    }
    f:=[]int{1}
    if str[:2]>="10" && str[:2]<="25"{
         f=append(f,2)
    }else{
        f=append(f,1)
    }
    for i:=2;i<len(str);i++{
        if str[i-1:i+1]>="10" && str[i-1:i+1]<="25"{
            f=append(f,f[i-1]+f[i-2])
        }else{
            f=append(f,f[i-1])
        }
    }

    return f[len(str)-1]

}
```

> 动态规划+递归

```go
func translateNum(num int) int {
   
   if num<10{
       return 1
   }
//判断最好两位是否 > 9 && <26
//是 就等于去掉 fn-1 + fn-2
//否 就等于 fn-1
   if num%100>=10 && num%100<=25{
       return translateNum(num/100)+translateNum(num/10)
   }else{
       return translateNum(num/10)
   }

}
```



## [剑指 Offer 48. 最长不含重复字符的子字符串](https://leetcode-cn.com/problems/zui-chang-bu-han-zhong-fu-zi-fu-de-zi-zi-fu-chuan-lcof/)

> map+滑动窗口

**难点**：主要是要判断    ==当前字符在窗口中是否已经存在==



- 前后指针组成窗口
- map中存放 字符的最慢出现的下标值
- 遍历字符串，判断map中是否已经存在
  - 存在，判断左指针和map中存在值的大小，
    - i大则说明，这个出现过的值已经不在窗口中，左指针不需要更新,		
    - 反之则更新左指针	
  - 不存在  右指针后移，并算出当前窗口的长度

```

func lengthOfLongestSubstring(s string) int {
    if len(s)<=1{
        return len(s)
    }
    ss:=[]byte(s)
    // 使用map来存放下标，通过判断下标是否在窗口中，判断是否重复
    indexMap:=make(map[byte]int,0)
    max:=0
    i,j:=0,0
    for; j<len(ss);j++{
        v,ok:=indexMap[ss[j]]
        // map中存在，并且在窗口中
        if ok && v>=i{
            i=v+1
          
        }
        indexMap[ss[j]]=j
        res:=j-i+1
        if res>max{
            max=res
        }
    }
    return max

}

```



# 11

## [剑指 Offer 18. 删除链表的节点](https://leetcode-cn.com/problems/shan-chu-lian-biao-de-jie-dian-lcof/)

```
/**
 * Definition for singly-linked list.
 * type ListNode struct {
 *     Val int
 *     Next *ListNode
 * }
 */
func deleteNode(head *ListNode, val int) *ListNode {
    
    // 要删除第一个元素怎么办
    if head!=nil && head.Val==val{
        head=head.Next
    }

    pre,cur:=head,head.Next
    for cur!=nil{
        if cur.Val==val{
            pre.Next=cur.Next
            cur=cur.Next
        }else{
            pre=pre.Next
            cur=cur.Next 
        }
        

    }
    return head
}
```



## [[剑指 Offer 22. 链表中倒数第k个节点](https://leetcode-cn.com/problems/lian-biao-zhong-dao-shu-di-kge-jie-dian-lcof/)

> 双指针

使用快慢指针，两者相距k，向后遍历当快指针指向nil时，慢指针刚好指向倒数第k个

<img src="https://cdn.jsdelivr.net/gh/Jason-Wu-1999/blog.img/img/d14cd267e7a0fe71efbb6106f4ccb1fcc3c68faf30c3ce3ee87b14371781436f-Picture4.png" alt="img" style="zoom: 33%;" />

```go
func getKthFromEnd(head *ListNode, k int) *ListNode {
    fast,low :=head,head
    for ;k>0;k--{
        if fast==nil{
            return nil
        }
        fast=fast.Next
    }

    for fast!=nil{
        low=low.Next
        fast=fast.Next
    }
    return low

}
```



# 12

## 剑指 Offer 25. 合并两个排序的链表](https://leetcode-cn.com/problems/he-bing-liang-ge-pai-xu-de-lian-biao-lcof/)

> 递归

回溯将n连起来了

```go
func mergeTwoLists(l1 *ListNode, l2 *ListNode) *ListNode {
    if l1==nil {return l2}
    if l2==nil {return l1}

    var n *ListNode
    if l1.Val<l2.Val {
        n = l1
        n.Next = mergeTwoLists(l1.Next,l2)
    } else {
        n = l2
        n.Next = mergeTwoLists(l1,l2.Next)
    }

    return n
}

```

> 伪头节点    双指针

两个指针分别指向两链表

比较大小，将小的放到新链表中

```go
func mergeTwoLists(l1 *ListNode, l2 *ListNode) *ListNode {
 newListNode :=&ListNode{0,nil,}
    cur:=newListNode

    for l1!=nil && l2!=nil{
        if l1.Val<=l2.Val{
            cur.Next=l1 
            l1=l1.Next
        }else{
            cur.Next=l2
            l2=l2.Next
        }
        cur=cur.Next
    }
    for l1!=nil{
        cur.Next=l1 
        l1=l1.Next
        cur=cur.Next
    }
    for l2!=nil{
        cur.Next=l2 
        l2=l2.Next
        cur=cur.Next
    }
    return newListNode.Next
}
```

## [剑指 Offer 52. 两个链表的第一个公共节点](https://leetcode-cn.com/problems/liang-ge-lian-biao-de-di-yi-ge-gong-gong-jie-dian-lcof/)

> 双指针：让两个指针走相同的路

我们使用两个指针 node1，node2 分别指向两个链表 headA，headB 的头结点，然后同时分别逐结点遍历，当 node1 到达链表 headA 的末尾时，重新定位到链表 headB 的头结点；当 node2 到达链表 headB 的末尾时，重新定位到链表 headA 的头结点。

这样，当它们相遇时，所指向的结点就是第一个公共结点、

==重点：循环的退出条件==    `curA!=curB`

**假设没有相交的节点循环也会在一轮之后退出，因为他们同时指向nil   所以返回nil**

```go
func getIntersectionNode(headA, headB *ListNode) *ListNode {
    if headB==nil || headA==nil{
        return nil
    }

    curA,curB:=headA,headB
    for curA!=curB{

       if curA==nil{
           curA=headB
       }else{
           curA=curA.Next
       }

        if curB==nil{
           curB=headA
       }else{
           curB=curB.Next
       }
    } 

    return curA
}
```



# 13

## [剑指 Offer 21. 调整数组顺序使奇数位于偶数前面](https://leetcode-cn.com/problems/diao-zheng-shu-zu-shun-xu-shi-qi-shu-wei-yu-ou-shu-qian-mian-lcof/)

> 双指针

类似快排的一轮

==注意每次移动都要判断下标==

```
func exchange(nums []int) []int {
    // 快排思想
    n:=len(nums)-1
    i,j:=0,n 
    for i<j{
    
        for j>i &&  nums[j]%2==0{
            j--
        }
        for i<j &&  nums[i]%2==1{
            i++
        }
        nums[i],nums[j]=nums[j], nums[i]
    }
    return nums
}

```



## [剑指 Offer 57. 和为s的两个数字](https://leetcode-cn.com/problems/he-wei-sde-liang-ge-shu-zi-lcof/)

> 双指针 若不是排序用map

```
func twoSum(nums []int, target int) []int {
// 双指针

   n:=len(nums)
   i,j:=0,n-1
   for i<j{
       sum:=nums[i]+nums[j]
       if sum>target{
           j--
       }else if sum<target{
           i++
       }else{
           return []int{nums[i],nums[j]}
       }
   }
   return []int{}
}
```



## [剑指 Offer 58 - I. 翻转单词顺序](https://leetcode-cn.com/problems/fan-zhuan-dan-ci-shun-xu-lcof/)

> 双指针 多次反转 整体反转，再反转每个单词

- 使用额外空间   使用库函数分割

```go
func reverseWords(s string) string {
    str:=strings.Fields(s)
    n:=len(str)
    for i:=0;i<n/2;i++{
        str[i],str[n-i-1]=str[n-i-1],str[i]
    }
    return strings.Join(str," ")
}
```

- **不使用额外空间 且不使用库函数**
- ==难点== 
  - ==在除去中间空格时，不使用额外空间，双指针替换空格==

```go
func reverseWords(ss string) string {
    s:=[]byte(ss)
//    双指针 去掉前后空格和中间的重复空格
    slow,fast:=0,0
    // 去掉前面的
    for fast<len(s) && s[fast]==' ' {
        fast++  
    }
    // 去中间重复的
    for ;fast<len(s);fast++{
        if fast>1 && s[fast]==s[fast-1] && s[fast]==' '{
            continue
        }
        s[slow]=s[fast]
        slow++
    }
    // 去后面的
    for slow-1>0 && s[slow-1]==' '{
        slow-- 
    }
    s=s[0:slow]
    reverse(s,0,len(s)-1)
    i,j:=0,0
    for i<len(s){
        j=i 
        
        //注意这里找空格的时候  不要使用 if s[j]== ' ' 应该使用for
        //用if 的话 最后一个没法找到，for判断的话就可以
        for j<len(s) && s[j]!=' '{
            j++
        }
        reverse(s,i,j-1)
        i=j+1
    }

    return string(s)
}

func reverse(b []byte, left, right int) {
	for left < right {
		(b)[left], (b)[right] = (b)[right], (b)[left]
		left++
		right--
	}
}
```

==注意点==

![image-20220309112606046](https://cdn.jsdelivr.net/gh/Jason-Wu-1999/blog.img/imgs/image-20220309112606046.png)



# 14

## [剑指 Offer 12. 矩阵中的路径](https://leetcode-cn.com/problems/ju-zhen-zhong-de-lu-jing-lcof/)

> dfs+回溯

1. 在dfs过程中，若board【i][j]  !=word[k]直接返回false，找到就往下递归，将找到值进行==格式化==----->赋值为空   ‘ ’，
2. dfs中向该字符的四个方向找（==需要注意判断下标是否越界==），只要找到就继续往下dfs，若四个方向均找不到即此路不通就返回false， 回溯到上一个 找==并将其值恢复==
3. 递归结束的条件；要么找到了最后一个返回true，要么就是找到某个值不能继续往下了返回false

==只要一个true就返回true==

```go
func exist(board [][]byte, word string) bool {
    m,n:=len(board),len(board[0])
    for i:=0;i<m;i++{
        for j:=0;j<n;j++{
            // 循环遍历，找到word第一个元素，开始深度遍历
            if dfs(board,i,j,0,word){
               return true
            }
        }
    }
    return false
}

// k 为word的下标
func dfs(board [][]byte, i,j,k int ,word string) bool{
    // 终止条件
    if k==len(word){
        return true
    }
    // 判断下标是否越界
    if i<0||j<0 || i==len(board) || j== len(board[0]){
        return false
    }


    if board[i][j]==word[k]{
        tmp:=board[i][j]
        board[i][j] = ' '
       
        if dfs(board,i,j+1,k+1,word)||
            dfs(board,i,j-1,k+1,word)||
            dfs(board,i+1,j,k+1,word)||
            dfs(board,i-1,j,k+1,word){
                return true
            }else {
                board[i][j]=tmp
            }
    }
        return false 
}
```

## 

## [剑指 Offer 13. 机器人的运动范围](https://leetcode-cn.com/problems/ji-qi-ren-de-yun-dong-fan-wei-lcof/)

> dfs

注意从00开始可不需要像四个方向，==只要向右和下就行==

```go
func movingCount(m int, n int, k int) int {
    //创建二维数组
    visit:=make([][]bool,m)
    for i:=0;i<m;i++{
        visit[i]=make([]bool,n)
    }
    // 从00开始dfs
    return dfs(visit,0,0,k,m,n)
}
// 求某个数 数位之和
func bitSum(n int)int {
   var sum int
	for n > 0 {
		sum += n % 10
		n = n / 10
	}
	return sum
}

func dfs(visit [][]bool,i,j,k,m,n int)int {

    // 退出条件 是否越界,是否遍历过，是否满足<k
    if i<0 || i>=m || j<0 || j>=n || visit[i][j]==true ||bitSum(i)+bitSum(j) > k{
        return 0
    }

        visit[i][j]=true
    
    //回溯一次加一次1
        return dfs(visit,i,j+1,k,m,n)+dfs(visit,i+1,j,k,m,n)+1
 
}
```



# 15

## [剑指 Offer 34. 二叉树中和为某一值的路径](https://leetcode-cn.com/problems/er-cha-shu-zhong-he-wei-mou-yi-zhi-de-lu-jing-lcof/)

> dfs 回溯  (使用先序遍历)     **必须是根节点到叶子节点**
>
> 类似八皇后

- 使用先序遍历递归

- 先将遍历到的节点加入到路径path中，并将target减去当前值

- 直到叶子节点，判断是否符合target==0

  - 是，则将path加入到结果res中
  - 否，则 将当前节点从path中取出，并回溯

  

==值得注意的是，记录路径时若直接执行 *res=append(*res,path) ，则是将 path 对象加入了 res ；后续 path 改变时， res 中的 path 对象也会随之改变。==

==正确做法：先将path复制到一个临时数组中，再将临时数组加入到res中==     深拷贝

		tmp := make([]int,len(path))
	    copy(tmp,path)
	    *res = append(*res,tmp)

```go
func pathSum(root *TreeNode, target int)  (res [][]int) {
    if root == nil {
        return nil
    } 
    path:=make([]int,0)
    recur(root,target,path,&res)
    return 
}
 
func recur(root *TreeNode,target int,path []int ,res *[][]int) { 
  
    if root ==nil{
        return 
    }
    path=append(path,root.Val)
    target-=root.Val
    // 判断是否为叶子节点
    if root.Left==nil && root.Right ==nil  && target==0{
        tmp := make([]int,len(path))
        copy(tmp,path)
        *res = append(*res,tmp)

        // *res=append(*res,path)
    }

    recur(root.Left,target,path,res)
    recur(root.Right,target,path,res)

    path=path[:len(path)-1]

}
```

## 

## [剑指 Offer 36. 二叉搜索树与双向链表](https://leetcode-cn.com/problems/er-cha-sou-suo-shu-yu-shuang-xiang-lian-biao-lcof/)



```
func treeToDoublyList(root *TreeNode)  {
	if root==nil{
		return
	}
	var pre, head *TreeNode
	var dfs func(node *TreeNode)
	dfs= func(node *TreeNode) {
		if node ==nil{
			return
		}

		dfs(node.Left)
		//执行
		if pre==nil{
			head=node

		}else{
			pre.Right=node
		}
		node.Left=pre
		pre=node

		dfs(node.Right)
	}

	dfs(root)

//首尾相连循环
	head.Left=pre
	pre.Right=head
	
}

```

## [剑指 Offer 54. 二叉搜索树的第k大节点](https://leetcode-cn.com/problems/er-cha-sou-suo-shu-de-di-kda-jie-dian-lcof/)

> 二叉搜索树的中序遍历：必定是升序

中序遍历：左 根 右

反过来遍历   右 根 左就是降序的

只需要在他遍历到最大值之后每回溯一次就 k-1 等于0时此时节点就是要找的

```
func kthLargest(root *TreeNode, k int)(res int ){
     var recur func(root*TreeNode)
     
     recur=func(root*TreeNode){
     	if root==nil||k<=0{
         	return
    	 }
     	recur(root.Right)
     	k--
     	if k==0{
        	 res=root.Val
         	return 
     	}
     	recur(root.Left)
    }
    
    
     recur(root)
     return  
}

```





# 16

## [剑指 Offer 45. 把数组排成最小的数](https://leetcode-cn.com/problems/ba-shu-zu-pai-cheng-zui-xiao-de-shu-lcof/)

> 先转换成字符串切片
>
> 主要点，字符串拼接 x+y >y+x   可得 x > y
>
> 使用切片自定义函数

利用自带的sort包自定义排序规则

```go
func minNumber(nums []int) string {
	s := []string{}
	for _, v := range nums {
		s = append(s, strconv.Itoa(v))
	}
    // 自定义排序
	sort.Slice(s, func(i, j int) bool { return s[i]+s[j] < s[j]+s[i] })
    // 字符串拼接
	return strings.Join(s, "")
}
```

//   快排+重定义比较规则

```go
-----------------------------------------------------------------------------------------------------

func minNumber(nums []int) string {
	s := []string{}
	for _, v := range nums {
		s = append(s, strconv.Itoa(v))
	}

    // 自定义排序
	// sort.Slice(s, func(i, j int) bool { return s[i]+s[j] < s[j]+s[i] })

    quick_sort(s,0,len(s)-1)

    // 字符串拼接
	return strings.Join(s, "")
}

func quick_sort(arr []string, left,right int)  {
	if left<right{
		l,h:=left,right
		pir := arr[l]

		for l < h {
			for l < h && arr[h]+pir >= pir+arr[h] {
				h--
			}

			for l < h && arr[l]+pir <= pir+arr[l] {
				l++
			}

			arr[l],	arr[h] = arr[h],arr[l]
		}
		arr[left], arr[l] =arr[l] ,arr[left]

		quick_sort(arr,left,l-1)
		quick_sort(arr,l+1,right)
	}
}
```

## [剑指 Offer 61. 扑克牌中的顺子](https://leetcode-cn.com/problems/bu-ke-pai-zhong-de-shun-zi-lcof/)

> 只要满足无重复值，且最大值-最小值 <=4必为顺子

```go
func isStraight(nums []int) bool {
    // 只要满足无重复值，且最大值-最小值 <=4必为顺子
    numMap:=make(map[int]bool)
    min:=14
    max:=0

    for _,v:=range nums{
        // 判断为0直接跳过
        if v==0{
            continue
        }
        // 判断有无重复值
        if numMap[v]{
            return false
        }
        // 添加到map中
        numMap[v]=true
        // 获取到最大 最小值
        if v>max{
            max=v
        }
        if v<min{
            min=v
        }

    }
    if max-min<5{
        return true
    }

    return false

}
```



# 17

## [剑指 Offer 40. 最小的k个数](https://leetcode-cn.com/problems/zui-xiao-de-kge-shu-lcof/)

> 快排优化

利用快速排序思想，partition函数每次会定位一个元素，并返回其下标。假设该元素为x，位于x之前的都小于等于x，位于x之后的元素都大于等于x。
我们的目的是得到前k小的元素，那么只需利用partition函数定位到下标为k的元素，在该元素之前的就是我们的目标。

==快排注意点：==

1. 注意这里要交换的是数组中的值，而不是tmp 和 arr[l]中的值，所以不能直接使用
   arr[l],tmp=tmp,arr[l]

2. 在找比基值大或者小的值时，==需要取等号==，出现和及值相同值时 进不去内层循环l,h不变，外层循环出不去


```go
func getLeastNumbers(arr []int, k int) []int {
    if k==0{//k=0的情况不能忘记
        return []int{}
    }
	start, end := 0, len(arr)-1
	index := partition(arr, start, end)
	for index != k-1 {//一旦index==k-1，即可停止。
		if index > k-1 {
			end = index - 1
		} else {
			start = index + 1
		}
		index = partition(arr, start, end)
	}
	return arr[:k]//注意：返回的这k个元素不一定是有序的
}

func partition(arr []int, l, h int) int {
    ll:=l
	tmp := arr[l]
	for l < h {
	//arr[h] >= pir要取等号，不然进不去循环，l,h不变，外层循环出不去
		for l < h && arr[h] >= pir {
			h--
		}		
		for l < h && arr[l] <=pir {
			l++
		}
		arr[h],arr[l] = arr[l],arr[h]
	}
	
	//注意这里要交换的是数组中的值，而不是tmp 和 arr[l]中的值，所以不能直接使用 
	//arr[l],tmp=tmp,arr[l]
	
	
	arr[ll]=arr[l]
	arr[l]=tmp
	return l
}


```

> 优先队列 大顶堆   top  k个数首先要想到的就是优先队列

**怎么判断是大顶堆还是小顶堆**

- ==看需要留下的值是什么==：比如最小的k个数  留下的是小值，就是大顶堆，因为大顶堆会把最大值pop出去
- 大顶堆 会把最大值pop出去 留下小值
- 小顶堆 会把最小值pop出去 留下大值

```go
func getLeastNumbers(arr []int, k int) []int {
   var res []int
	n := len(arr)
	if n == 0 || k <= 0 || n < k {
		return res
	}
	pq := &IntHeap{}
	for i := 0; i < k; i++ {
		heap.Push(pq, arr[i])
	}
	// heapify
	heap.Init(pq)
	for i := k; i < n; i++ {
		if pq.Len() > 0 && arr[i] < pq.Peek().(int) {
			heap.Pop(pq)
			heap.Push(pq, arr[i])
		}
	}

	for i := 0; i < k; i++ {
		res = append(res, heap.Pop(pq).(int))
	}
	return res
}

// 实现堆  每一个位置存放一个数组 key和value
type IntHeap []int

func (h IntHeap) Len() int           { return len(h) }
// 为了实现大根堆，Less在大于时返回true
func (h IntHeap) Less(i, j int) bool { return h[i] > h[j] }
func (h IntHeap) Swap(i, j int)      { h[i], h[j] = h[j], h[i] }

func (h *IntHeap) Push(x interface{}) {
	// Push and Pop use pointer receivers because they modify the slice's length,
	// not just its contents.
	*h = append(*h, x.(int))
}
func (h *IntHeap) Pop() interface{} {
	old := *h
	n := len(old)
	x := old[n-1]
	*h = old[0 : n-1]
	return x
}

// 查看堆顶元素
func (h *IntHeap) Peek() interface{} {
	return (*h)[0]
}

```





# 18

## [剑指 Offer 55 - I. 二叉树的深度](https://leetcode-cn.com/problems/er-cha-shu-de-shen-du-lcof/)

> 递归
>
> 递归的最终输出一定是第一层的return，其余的只是返回在函数内部

```go
func maxDepth(root *TreeNode) int {
    if root==nil{
        return 0
    }
  
  return max(maxDepth(root.Left),maxDepth(root.Right))+1
}

func max(a, b int) int {
    if a > b {
        return a
    }
    return b
}
```

> dfs

```go
 var max int
func maxDepth(root *TreeNode) int {
    max=0
    if root==nil{
        return 0
    }
  recur(root,0)
  return max
}
func recur(root *TreeNode,m int){
    if root==nil{
        if m>max{
            max=m
        }
        return 
    } 
    recur(root.Left,m+1)
    recur(root.Right,m+1)
}
```



## [剑指 Offer 55 - II. 平衡二叉树](https://leetcode-cn.com/problems/ping-heng-er-cha-shu-lcof/)

> 递归

==注意==：平衡二叉树是==任意节点的左右子节点==的深度相差不超过1

在第一题的基础上，判断当前节点的左右子节点深度即可

```go
func isBalanced(root *TreeNode) bool {
    if root == nil {
        return true
    }
    return abs(height(root.Left) - height(root.Right)) <= 1 && isBalanced(root.Left) && isBalanced(root.Right)
}

func height(root *TreeNode) int {
    if root == nil {
        return 0
    }
    return max(height(root.Left), height(root.Right)) + 1
}

func max(x, y int) int {
    if x > y {
        return x
    }
    return y
}

func abs(x int) int {
    if x < 0 {
        return -1 * x
    }
    return x
}
```



# 19

## [剑指 Offer 68 - I. 二叉搜索树的最近公共祖先](https://leetcode-cn.com/problems/er-cha-sou-suo-shu-de-zui-jin-gong-gong-zu-xian-lcof/)

> 递归


 利用二叉搜索树的特点  左小右大 ，

**如何最近**：p、q 在一个节点的两侧 （一个大于 一个小于）或者，p q其中一个等于当前节点

-  递归：当两个都在一侧时，向那一侧递归即可，直到pq出现上述情况



    func lowestCommonAncestor(root, p, q *TreeNode) *TreeNode {
    
    	if root.Val>p.Val && root.Val>q.Val{
       		 return root
    	}
     
    	l:= lowestCommonAncestor(root.Left,p,q)
    	r:= lowestCommonAncestor(root.Right,p,q)
    	
    	if l!=nil && r!=nil{
       		 return root
    	}
    	
    	if l==nil{
        	return r
    	}
    	
    	return l
    }






## [剑指 Offer 68 - II. 二叉树的最近公共祖先](https://leetcode-cn.com/problems/er-cha-shu-de-zui-jin-gong-gong-zu-xian-lcof/)

> 递归遍历

二叉树，没有二叉搜索树的特点，故不能通过比大小，只能遍历找

若 root是 p, q的 **最近公共祖先** ，则只可能为以下情况之一：

1. p 和 q 在 root的子树中，且分列 root的 异侧（即分别在左、右子树中）；
2. p = root，且 q在 root 的左或右子树中；
3. q = root ，且 pp 在 root的左或右子树中；



<img src="https://cdn.jsdelivr.net/gh/Jason-Wu-1999/blog.img/imgs/image-20210825222110952.png" alt="image-20210825222110952" style="zoom:80%;" />



```go
func lowestCommonAncestor(root, p, q *TreeNode) *TreeNode {

    if root ==nil{
        return nil
    }
    if root.Val==p.Val ||root.Val==q.Val{
        return root
    }
 
    l:= lowestCommonAncestor(root.Left,p,q)
    r:= lowestCommonAncestor(root.Right,p,q)
    if l!=nil && r!=nil{
        return root
    }
    if l==nil{
        return r
    }
    return l
}
```



## [剑指 Offer 15. 二进制中1的个数](https://leetcode-cn.com/problems/er-jin-zhi-zhong-1de-ge-shu-lcof/)

1. > **方法一：**与1

   ​	循环将==num与1相与==，等于1，计数加一，循环一次就==右移==一次，num==0就跳出循环

2. > **方法二**：**巧用 n \& (n - 1)*n*&(*n*−1)**

- **(n - 1) 解析：** 二进制数字 *n* 最右边的 1 变成 0 ，此 1 右边的 0 都变成 1 。
- **n \& (n - 1) 解析：** 二进制数字 n 最右边的 1 变成0 ，其余不变。

<img src="https://cdn.jsdelivr.net/gh/Jason-Wu-1999/blog.img/img/9bc8ab7ba242888d5291770d35ef749ae76ee2f1a51d31d729324755fc4b1b1c-Picture10.png" alt="Picture10.png" style="zoom:40%;" />

算法流程：

1. 初始化数量统计变量 res

2. 循环消去最右边的1：当n = 0时跳出。
   1. **`res += 1` ：** 统计变量加 1 ；
   2. **`n &= n - 1` ：** 消去数字 *n*最右边的 1 。
   
3. 返回统计数量 res

   ```go
   func hammingWeight(num uint32) (res int) {
       for ;num>0;num &=num-1 {
           res++
       }
       return
   }
   ```

   

## [剑指 Offer 16. 数值的整数次方](https://leetcode-cn.com/problems/shu-zhi-de-zheng-shu-ci-fang-lcof/)

> 快速次幂

**算法流程：**

- 当 x = 0时：直接返回 0（避免后续 x=1/x 操作报错）。
- 初始化 res = 1 ；
- 当 n < 0 时：把问题转化至 n≥0 的范围内，即执行 x=1/x ，n=−n ；
-  循环计算：当 n = 0 时跳出；
  - 当 n \& 1 = 1 时：将当前 x 乘入 res （即 res∗=x ）；
  -  n \& 1 = 0 时 ：执行 x = x^2（即 x *= x ）；
  - 执行 n 右移一位（即 n>>=1）。
- 返回 res 。

<img src="https://cdn.jsdelivr.net/gh/Jason-Wu-1999/blog.img/img/40a7a874523e26cacae9c502a6e8cf8b58dba878739f17e6bb3ed6be76e97569-Picture1.png" alt="Picture1.png" style="zoom:50%;" />



```go

func myPow(x float64, n int) float64 {
    if x==0{
        return 0
    }
    if n == 0{
        return 1
    }
    if n == 1{
        return x
    }
    if n<0{
        x = 1/x
        n = -n
    }
    ret:=1.0
    for n>=1{
        //最后一位为1  将当前 x 乘入 res 
        if n & 1 == 1{
            ret *= x
            n--
        }else{		//只将x变成平方，不乘入res结果中
            x *= x
            n = n >> 1
        }
    }
    return ret
}

```

> 递归

思想同上：n>>1 就等价于 n/2

- 当 n 为偶数时：==x ^ n = x ^( n / 2 ) * x ^( n / 2 )==
- 当 n 为奇数时：==x ^ n = x ^( n / 2 ) * x ^( n / 2 )*x==     如     ==x ^ 5 = x ^2  * x ^2 * x== 

```go
func myPow(x float64, n int) float64 {
    if n == 0{
        return 1
    }
    if n == 1{
        return x
    }
    if n<0{
        x = 1/x
        n = -n
    }
    ret=myPow(x,n/2)

    if n%2==0{
       return ret*ret
    }else{
        return x*ret*ret
    }
}
```



## [剑指 Offer 17. 打印从1到最大的n位数](https://leetcode-cn.com/problems/da-yin-cong-1dao-zui-da-de-nwei-shu-lcof/)

考虑大数溢出问题；需要将其转换成 []byte 还在string

- > 方法一：使用[]byte模拟数字相加
  >
  
  

初始化一个长度为n的字符串,初始化的时候每个字符都是0, 需要在字符串上模拟数字加法, 从最低位开始,每次加1形成一个数字, 加到10就进1位,
当加到最大的n位的时候停止进位, 循环结束!

> 输入: n=1
> 加1循环停止条件: number[0]=9, 因为再进位就变成两位数了
>
> 输入: n=2
> 加1循环停止条件: number[1]=9, 因为再进位就变成三位数了
>
> 输入: n=3
> 加1循环停止条件: number[2]=9, 因为再进位就变成四位数了

```go
//防止大数字溢出，使用字符串模拟数字加法，
//低位加到等于10就进1，进到最高位的时候停止进位（999进1变成1000，此时停止进位）

func printNumbers(n int) []int {
   res := []int{}
   if n <= 0 {
      return res
   }
    
   number := make([]byte, n)
   for i := 0; i < n; i++ {
      number[i] = '0'
   }
   for !increment(&number, n) {
       //将string转换成int，会自动把"01"--->1
         add, _ := strconv.Atoi(string(number))
         res = append(res, add)
   }
   return res
}

func increment(number *[]byte, length int) bool {
   // 只有最高位进位的时候才会超过最大值
   isOverflow := false

   // 进位
   var takeOver byte
   takeOver = 0

   for i := length - 1; i >= 0; i-- {
      nSum := ((*number)[i] - '0') + takeOver

      // 从低位开始+1
      if i == length-1 {
         nSum++
      }

      if nSum >= 10 {
         // 需要进位

         if i == 0 {
            // 最高位不能进位， 返回进位结束
            isOverflow = true
         } else {
            nSum -= 10
            takeOver = 1
            (*number)[i] = '0' + nSum
         }
      } else {
         // 执行一次+1操作就退出
         (*number)[i] = '0' + nSum
         break
      }
   }
   return isOverflow
}
```

- 方法二：递归全排列

```go
package main

import (
   "fmt"

)
func printNumbers1(n int) []string {
   arr := []string{}
   for j := 0; j < 10; j++ {
      arr = append(arr, fmt.Sprintf("%v", j))
   }

   var delzero func(str string) string
   delzero = func(str string) string {
      if len(str) == 0 {
         return str
      }

      if str[0] == '0' {
         return delzero(str[1:])
      }

      return str
   }

   handle := func(arr []string) []string {
      resarr := []string{}
      for i := 0; i < 10; i++ {
         for _, v := range arr {
            s := fmt.Sprint(i) + v
            resarr = append(resarr, delzero(s))
         }
      }
      return resarr
   }

   for i := 1; i < n; i++ {
      arr = handle(arr)
   }

   return arr[1:]
}
```

## [剑指 Offer 20. 表示数值的字符串](https://leetcode-cn.com/problems/biao-shi-shu-zhi-de-zi-fu-chuan-lcof/)

> 状态机

- 列出所有可以存在的状态，和字符类型
- 定义map 各个状态时下一个状态可以存放的字符类型以及该字符类型对应的状态
- 定义初始状态为起始状态
- 遍历字符串首先判断字符对应的字符类型，再看在目前状态下是否可以放该字符类型的字符，不可以直接返回false，可以将新的字符变量对应的状态赋值给状态
- 遍历完，判断状态是否为可以结束的状态，可以则成功返回true

```go
type State int//自定义类型
type CharType int

const(
    STATE_INITIAL  State=iota
    STATE_INT_SIGN
    STATE_INTEGER
    STATE_POINT
    STATE_POINT_WITHOUT_INT
    STATE_FRACTION
    STATE_EXP
    STATE_EXP_SIGN
    STATE_EXP_NUMBER
    STATE_END
)

const(
    CHAR_NUMBER  CharType = iota
    CHAR_EXP
    CHAR_POINT
    CHAR_SIGN
    CHAR_SPACE
    CHAR_ILLEGA

)

func toCharType(ch byte) CharType {
    switch ch {
    case '0', '1', '2', '3', '4', '5', '6', '7', '8', '9':
        return CHAR_NUMBER
    case 'e', 'E':
        return CHAR_EXP
    case '.':
        return CHAR_POINT
    case '+', '-':
        return CHAR_SIGN
    case ' ':
        return CHAR_SPACE
    default:
        return CHAR_ILLEGA
    }
}

func isNumber(s string) bool{
    //定义map 各个状态时下一个状态可以存放的字符类型以及该字符类型对应的状态
        transfer := map[State]map[CharType]State{
        STATE_INITIAL: map[CharType]State{
            CHAR_SPACE:  STATE_INITIAL,
            CHAR_NUMBER: STATE_INTEGER,
            CHAR_POINT:  STATE_POINT_WITHOUT_INT,
            CHAR_SIGN:   STATE_INT_SIGN,
        },
        STATE_INT_SIGN: map[CharType]State{
            CHAR_NUMBER: STATE_INTEGER,
            CHAR_POINT:  STATE_POINT_WITHOUT_INT,
        },
        STATE_INTEGER: map[CharType]State{
            CHAR_NUMBER: STATE_INTEGER,
            CHAR_EXP:    STATE_EXP,
            CHAR_POINT:  STATE_POINT,
            CHAR_SPACE:  STATE_END,
        },
        STATE_POINT: map[CharType]State{
            CHAR_NUMBER: STATE_FRACTION,
            CHAR_EXP:    STATE_EXP,
            CHAR_SPACE:  STATE_END,
        },
        STATE_POINT_WITHOUT_INT: map[CharType]State{
            CHAR_NUMBER: STATE_FRACTION,
        },
        STATE_FRACTION: map[CharType]State{
            CHAR_NUMBER: STATE_FRACTION,
            CHAR_EXP:    STATE_EXP,
            CHAR_SPACE:  STATE_END,
        },
        STATE_EXP: map[CharType]State{
            CHAR_NUMBER: STATE_EXP_NUMBER,
            CHAR_SIGN:   STATE_EXP_SIGN,
        },
        STATE_EXP_SIGN: map[CharType]State{
            CHAR_NUMBER: STATE_EXP_NUMBER,
        },
        STATE_EXP_NUMBER: map[CharType]State{
            CHAR_NUMBER: STATE_EXP_NUMBER,
            CHAR_SPACE:  STATE_END,
        },
        STATE_END: map[CharType]State{
            CHAR_SPACE: STATE_END,
        },
    }
    state :=STATE_INITIAL
    for i:=0;i<len(s);i++{
        typ:=toCharType(s[i])
        newstate,ok:=transfer[state][typ]
        if !ok{
            return false
        }else{
            state=newstate
        }
    }

   return state == STATE_INTEGER || state == STATE_POINT || state == STATE_FRACTION || state == STATE_EXP_NUMBER || state == STATE_END
}

```





## [剑指 Offer 21. 调整数组顺序使奇数位于偶数前面](https://leetcode-cn.com/problems/diao-zheng-shu-zu-shun-xu-shi-qi-shu-wei-yu-ou-shu-qian-mian-lcof/)

> 双指针

- 一个指向首，一个指向尾，向中间靠拢
- 首指针向右遍历找偶数，尾指针向左遍历找奇数
- 都找到之后进行交换
- 循环退出条件是，首指针到尾指针右边

==注意点：==注意数组越界，中间for也要注意判断

==判断奇偶还可以使用位运算，需要注意位运算的优先级低==

```go
func exchange(nums []int) []int {
     l:=0
     r :=len(nums)-1
     for l<r{
		if nums[l]%2==0&&nums[r]%2!=0{
			nums[l],nums[r]=nums[r],nums[l]
		}
		 for l<len(nums)&&nums[l]%2!=0{
			 l++
		 }
		 for r>=0&&nums[r]%2==0{
			 r--
		 }

	 }
	 return nums
}

```



## [剑指 Offer 22. 链表中倒数第k个节点](https://leetcode-cn.com/problems/lian-biao-zhong-dao-shu-di-kge-jie-dian-lcof/)

> 双指针

使用快慢指针，两者相距k，向后遍历当快指针指向nil时，慢指针刚好指向倒数第k个

<img src="https://cdn.jsdelivr.net/gh/Jason-Wu-1999/blog.img/img/d14cd267e7a0fe71efbb6106f4ccb1fcc3c68faf30c3ce3ee87b14371781436f-Picture4.png" alt="img" style="zoom: 33%;" />

```go
func getKthFromEnd(head *ListNode, k int) *ListNode {
    fast,low :=head,head
    for ;k>0;k--{
        if fast==nil{
            return nil
        }
        fast=fast.Next
    }

    for fast!=nil{
        low=low.Next
        fast=fast.Next
    }
    return low

}
```



## [剑指 Offer 24. 反转链表](https://leetcode-cn.com/problems/fan-zhuan-lian-biao-lcof/)

> 迭代

<img src="https://pic.leetcode-cn.com/1624782858-oyJziv-008eGmZEly1gnrf1oboupg30gy0c44qp.gif" alt="img" style="zoom: 60%;" />

```go
func reverseList(head *ListNode) *ListNode {
    var prev *ListNode
    curr := head
    for curr != nil {
        next := curr.Next
        curr.Next = prev
        // 下两句为更新指针
        prev = curr
        curr = next
    }
    return prev
}


```

## [剑指 Offer 25. 合并两个排序的链表](https://leetcode-cn.com/problems/he-bing-liang-ge-pai-xu-de-lian-biao-lcof/)

> 递归

```go
func mergeTwoLists(l1 *ListNode, l2 *ListNode) *ListNode {
    if l1==nil {return l2}
    if l2==nil {return l1}

    var n *ListNode
    if l1.Val<l2.Val {
        n = l1
        n.Next = mergeTwoLists(l1.Next,l2)
    } else {
        n = l2
        n.Next = mergeTwoLists(l1,l2.Next)
    }

    return n
}

```

> 伪头节点    双指针

```go
func mergeTwoLists(l1 *ListNode, l2 *ListNode) *ListNode {
 newListNode :=&ListNode{0,nil,}
    cur:=newListNode

    for l1!=nil && l2!=nil{
        if l1.Val<=l2.Val{
            cur.Next=l1 
            l1=l1.Next
        }else{
            cur.Next=l2
            l2=l2.Next
        }
        cur=cur.Next
    }
    for l1!=nil{
        cur.Next=l1 
        l1=l1.Next
        cur=cur.Next
    }
    for l2!=nil{
        cur.Next=l2 
        l2=l2.Next
        cur=cur.Next
    }
    return newListNode.Next
}
```



## [剑指 Offer 29. 顺时针打印矩阵](https://leetcode-cn.com/problems/shun-shi-zhen-da-yin-ju-zhen-lcof/)

> 设定边界，逐渐缩小

#### 牛逼写法

```
var dirs = []struct{x, y int}{{0,1}, {1,0}, {0,-1}, {-1,0}}
func spiralOrder(matrix [][]int) []int {
    m,n := len(matrix), len(matrix[0])
    res := make([]int, 0)

    used := make([][]bool, m)
     for i := 0; i < m ; i++{
         used[i] = make([]bool, n)
     }

    x, y , index := 0, 0, 0
    for i := 0; i < n * m; i++{
        res = append(res, matrix[x][y])
        used[x][y] = true
        d := dirs[index & 3]

        xx, yy := x+d.x, y+d.y
        if xx < 0 || xx >= m ||yy < 0 || yy >= n || used[xx][yy]{
            index++
        }
         d = dirs[index & 3]
         x, y = x+d.x, y+d.y

    }
    return res
}
```



外循环

- 从左向右、从上向下、从右向左、从下向上” 四个方向循环
  - 从左向右遍历一次就会遍历完==最上面的一行==，就将上边界下移
  - 从上向下遍历一次就会遍历完==最右面的一行==，就将右边界左移
  - 从右向左遍历一次就会遍历完==最下面的一行==，就将下边界上移
  - 从右向左遍历一次就会遍历完==最下面的一行==，就将左边界右移
- 没缩小一次边界，判断边界是否越界，越界就退出循环

```go
func spiralOrder(matrix [][]int) []int {
    if len(matrix)==0{
        return nil
    }
    var res []int 
    l:=0
    r:=len(matrix[0])-1
    t:=0
    b:=len(matrix)-1
    for{
        // 向右
        for i:=l;i<=r;i++{
            res=append(res,matrix[t][i])
        }
        //遍历完最上面一行向下缩,上边界往下走
        t++
        if t>b{
            break
        }


        for i:=t; i<=b;i++{
            res=append(res,matrix[i][r])
        }
         //遍历完最右边一列向走缩,右边界往左走
        r--
        if l>r{
            break
        }

        for i:=r;i>=l;i--{
            res=append(res,matrix[b][i])
        }
        b--
         if t>b{
            break
        }

        for i:=b;i>=t;i--{
            res = append(res,matrix[i][l])
        }
        l++
        if l>r{
            break
        }

    }

    return res

}

```

## [59. 螺旋矩阵 II](https://leetcode-cn.com/problems/spiral-matrix-ii/)

> 剑指offer29的逆过程

#### 牛逼写法

```
var dirs = []struct{x, y int}{{0,1}, {1,0}, {0,-1}, {-1,0}}
func generateMatrix(n int) [][]int {
    res := make([][]int, n)
    for i := 0; i < n; i++{
        res[i] = make([]int, n)
    }
    x, y := 0, 0
    index := 0
    for i := 1; i <= n*n; i++{
        res[x][y] = i
        d := dirs[index&3]
        xx, yy := x + d.x, y + d.y
        if xx < 0 || xx >= n || yy < 0 || yy >= n || res[xx][yy]!=0{
            index++
        }
        d = dirs[index&3]
        x, y = x + d.x, y + d.y
    }
    return res
}
```



```go
func generateMatrix(n int) [][]int {
    //
    res:=make([][]int,n)
     for i := 0; i < n; i++ {
        res[i] = make([]int, n)
    }
    left,right:=0,n-1
    up,down:=0,n-1
    num:=1
    for {
        for i:=left;i<=right;i++{
            res[up][i]=num
            num++
        }
        up++
        if down<up{
            break
        }

        for i:=up;i<=down;i++{
            res[i][right]=num
            num++
        }
        right--
        if right<left{
            break
        }

         for i:=right;i>=left;i--{
            res[down][i]=num
            num++
        }
        down--
        if down<up{
            break
        }

        for i:=down;i>=up;i--{
            res[i][left]=num
            num++
        }
        left++
        if right<left{
            break
        }
    }
    return res
}
```



## [剑指 Offer 36. 二叉搜索树与双向链表](https://leetcode-cn.com/problems/er-cha-sou-suo-shu-yu-shuang-xiang-lian-biao-lcof/)

> 中序线索化，递归

dfs(cur): 递归法中序遍历；

1. **==终止条件：==** 当节点 cur 为空，代表越过叶节点，直接返回；
2. ==递归左子树==，即 dfs(cur.left) ；
3. ==构建链表：==
   1. 当 pre 为空时： 代表正在访问链表头节点，记为 head ；
   2. 当 pre 不为空时： 修改双向节点引用，即 pre.right = cur ， cur.left = pre ；
   3. 更新 pre ： 更新 pre = cur ，即节点 cur 是后继节点的 pre ；
4. ==递归右子树==，即 dfs(cur.right) ；

treeToDoublyList(root)：

1. 特例处理： 若节点 root 为空，则直接返回；
2. 初始化： 空节点 pre ；
3. 转化为双向链表： 调用 dfs(root) ；
4. 构建循环链表： 中序遍历完成后，head 指向头节点， pre 指向尾节点，因此修改 head 和 pre 的双向节点引用即可；
5. 返回值： 返回链表的头节点 head 即可；

```go
class Solution {  
    Node pre ,head;
    public Node treeToDoublyList(Node root) {
      if (root ==null){
          return null;
      }
        dfs(root);
        head.left=pre;
        pre.right=head;
        return head;
    }

    public void dfs(Node  cur){
        if (cur ==null){
            return ;
        }
    // 先遍历左子节点
        dfs(cur.left);
        if (pre!=null){
            pre.right = cur;
        }else{
            head=cur;
        }
        cur.left=pre;
        pre=cur;
        dfs(cur.right);
    }
}
```

## [剑指 Offer 37. 序列化二叉树](https://leetcode-cn.com/problems/xu-lie-hua-er-cha-shu-lcof/)

> dfs

只要求序列化，故可以使用dfs和bfs

需要注意的是转化为string

```go
/**
 * Definition for a binary tree node.
 * type TreeNode struct {
 *     Val int
 *     Left *TreeNode
 *     Right *TreeNode
 * }
 */

type Codec struct {
    
}

func Constructor() Codec {
    return  Codec{}
}

// Serializes a tree to a single string.
func (this *Codec) serialize(root *TreeNode) string {
	if root == nil {
		return "null"
	}
	return strconv.Itoa(root.Val) + "," + this.serialize(root.Left) + "," + this.serialize(root.Right)
}

// Deserializes your encoded data to tree.
func (this *Codec) deserialize(data string) *TreeNode {    
    sp := strings.Split(data, ",")
    var build func() *TreeNode
    build = func() *TreeNode {
        if sp[0] == "null" {
            sp = sp[1:]
            return nil
        }
        val, _ := strconv.Atoi(sp[0])
        sp = sp[1:]
        root:=&TreeNode{
            Val:val,
        }
        root.Left=build()
        root.Right=build()
        return root
    }
    return build()

}

```



> bfs 一层一层遍历  入队列

```go
//序列化
func (this *Codec) serialize(root *TreeNode) string {
	q := []*TreeNode{root}
	res := []string{}
	for len(q) != 0 {
		node := q[0]
		q = q[1:]
		if node != nil {
			res = append(res, strconv.Itoa(node.Val))
			q = append(q, node.Left)
			q = append(q, node.Right)
		} else {
			res = append(res, "X")
		}
	}
	return strings.Join(res, ",")
}


```

![image.png](https://cdn.jsdelivr.net/gh/Jason-Wu-1999/blog.img/imgs/f93235b0b74fbc481451954ea50a65fab347c602dd55ccb1df36ceed929136d5-image.png)

- 依然先转成list数组，用一个指针 cursor 从第二项开始扫描。
- 起初，用list[0]构建根节点，并让根节点入列。
- 节点出列，此时 cursor 指向它的左子节点值，cursor+1 指向它的右子节点值。
  - 如果子节点值是数值，则创建节点，并认出列的父亲，同时自己也是父亲，入列。
  - 如果子节点值为 'X'，什么都不用做，因为出列的父亲的 left 和 right 本来就是 null
- 可见，所有的真实节点都会在队列里走一遍，出列就带出儿子入列

- 类似序列化，序列化是将其放在string后面，反序列化是添加的节点后面
- 需要一个cursor 指针来从切片中取值，发现==左右子节点是成对出现==的，每出列一个节点，指针后移两位对应的就是下一个节点的左右节点，



```go
//反序列化
func (this *Codec) deserialize(data string) *TreeNode {
	if data == "X" {
		return nil
	}
	list := strings.Split(data, ",")
	Val, _ := strconv.Atoi(list[0])
	root := &TreeNode{Val: Val}
	q := []*TreeNode{root}
	cursor := 1

	for cursor < len(list) {
		node := q[0]
		q = q[1:]
		leftVal := list[cursor]
		rightVal := list[cursor+1]
		if leftVal != "X" {
			v, _ := strconv.Atoi(leftVal)
			leftNode := &TreeNode{Val: v}
			node.Left = leftNode
			q = append(q, leftNode)
		}
		if rightVal != "X" {
			v, _ := strconv.Atoi(rightVal)
			rightNode := &TreeNode{Val: v}
			node.Right = rightNode
			q = append(q, rightNode)
		}
		cursor += 2
	}
	return root
}

```



## [剑指 Offer 38. 字符串的排列](https://leetcode-cn.com/problems/zi-fu-chuan-de-pai-lie-lcof/)

> dfs   难理解，递归中还有循环

```go
func permutation(s string) []string {
	res := []string{}
	bytes := []byte(s)
	var dfs func(x int)
	dfs = func(x int) {
		if x == len(bytes)-1 {
			res = append(res, string(bytes))
		}
		dict := map[byte]bool{}
		for i := x; i < len(bytes); i++ {
			if !dict[bytes[i]] {
                // 交换
				bytes[x], bytes[i] = bytes[i], bytes[x]
				dict[bytes[x]] = true
				dfs(x + 1)
                // 换回来
				bytes[x], bytes[i] = bytes[i], bytes[x]
			}
		}
	}
	dfs(0)
	return res
}
```



## [剑指 Offer 44. 数字序列中某一位的数字](https://leetcode-cn.com/problems/shu-zi-xu-lie-zhong-mou-yi-wei-de-shu-zi-lcof/)





## [剑指 Offer 48. 最长不含重复字符的子字符串](https://leetcode-cn.com/problems/zui-chang-bu-han-zhong-fu-zi-fu-de-zi-zi-fu-chuan-lcof/)

> 窗口滑动+map

- 前后指针组成窗口
- map中存放 字符的最慢出现的下标值
- 遍历字符串，判断map中是否已经存在
  - 存在，判断左指针和map中存在值的大小，
    - i大则说明，这个出现过的值已经不在窗口中，左指针不需要更新,		
    - 反之则更新左指针	
  - 不存在  右指针后移，并算出当前窗口的长度

==计算长度需要注意：按照正常是  右下标 - 左下标+1，为了方便直接将左指针指向窗口的前一个下标==

```go
func lengthOfLongestSubstring(s string) int {
// 窗口滑动
    m := make(map[byte]int)
    i:=-1
    res:=0
    for j:=0;j<len(s);j++{
        if _,ok:=m[s[j]];  ok{
        //判断左指针和map中存在值的大小，i大则说明，这个出现过的值已经不在窗口中，左指针不重要更新			  
        i=max(i,m[s[j]])
        }
                   
        m[s[j]]=j
        res=max(res,j-i)
    }
    return res
}
func max(a,b int)int{
    if a>b {
        return a 
    }
    return b
}
```

 

## [剑指 Offer 50. 第一个只出现一次的字符](https://leetcode-cn.com/problems/di-yi-ge-zhi-chu-xian-yi-ci-de-zi-fu-lcof/)

> 数组保存出现的次数，因为规定是只包含小写字母所以最多26个  下标0--25分别对应a--z

第一次遍历将次数放进数组中

第二次遍历，找对应字符在数组中的值，返回第一个为1的

```go
func firstUniqChar(s string) byte {
    ss:=[26]int{}
    for _,v:=range s{
        ss[v-'a']++
    }

    for i,v:=range s{
        if ss[v-'a']==1{
            return s[i]
        }
    }
    return  ' '
}
```

> map

```go
func firstUniqChar(s string) byte {
    m :=make(map[byte]int)
    for i,_:=range s{
        _,ok:=m[s[i]]   
        if !ok{
            m[s[i]]=i
        }else {
            m[s[i]]=-1  
        }
    }
    for i,_:=range s{
        if m[s[i]]!=-1{
            return s[i]
        }
    }
    return' '
}

```





## [剑指 Offer 56 - I. 数组中数字出现的次数](https://leetcode-cn.com/problems/shu-zu-zhong-shu-zi-chu-xian-de-ci-shu-lcof/)

> 要求时间复杂度O（n），空间复杂度O（1）
>
> 异或

==异或性质：相同的数字异或结果为0，==

故：所有值异或的最终结果为，不同的数字异或的结果

```
4 ^ 3 ^ 1 ^ 3 ^ 4 ^ 6 => 1 ^ 6

6 对应的二进制： 110
1 对应的二进制： 001
1 ^ 6  二进制： 111
```

但是由上可知111不能反推出，1和6

- **分两组：**只需将不同的分开，并且相同的必须分到一组

```
1 ^ 3 ^ 3=1
6 ^ 4 ^ 4=6
```

- **如何分，按什么分**

将所有值异或得到k，k至少有1位为1（否则x与y就是相等的），k为1的位置说明了x,y在此位置不同

我们通过一个辅助变量m来保存z中哪一位为1.（可能有多个位都为1，我们找到最低位的1即可）

遍历数组把中的值和m异或，为0的一组，为1的一组

```cs
func singleNumbers(nums []int) []int {
   
    k:=0
    for _,v1:=range nums{
        k=k^v1
    }
   
    m:=1
    // 可知k至少有一位为1，那两个数字在该位上必不同，找出最低的不为1的，
    for((k & m) == 0){
        m <<=1
    }
    x,y:=0,0
    for _,v2:=range nums{
        if v2 & m ==0{
            x^=v2
        }else{
            y^=v2
        }
    }
    return []int{x,y}
}
```



## [剑指 Offer 56 - II. 数组中数字出现的次数 II](https://leetcode-cn.com/problems/shu-zu-zhong-shu-zi-chu-xian-de-ci-shu-ii-lcof/)

> map

```go
func singleNumber(nums []int) int {
    m:= make(map[int]int,0)
    for _,v:=range nums{
        m[v]++
    }

    for k,v1:=range m{
        if v1==1{
            return k
        }
    }
    return -1
}
```



> 排序





## [剑指 Offer 64. 求1+2+…+n](https://leetcode-cn.com/problems/qiu-12n-lcof/)

==难点：==

1. **怎么不使用 if  来判断终止条件**
2. 当需要全局变量的时候， 可以使用闭包

**逻辑运算符的短路性质。**

以逻辑运算符 && 为例，对于 A && B 这个表达式，如果 A 表达式返回 False ，那么 A && B 已经确定为 False ，此时不会去执行表达式 B。同理，对于逻辑运算符 ||， 对于 A || B 这个表达式，如果 A 表达式返回 True ，那么 A || B 已经确定为True ，**此时不会去执行表达式 B**



> 递归

```go
func sumNums(n int) int {
    _= n > 0 && func () bool {  n+= sumNums(n-1) ; return true }()
	return n 
}
    
```

> 闭包递归

```go
func sumNums(n int) int {
     ans := 0
    var sum func(n int)bool
    sum=func(n int)bool{
        ans+=n
        return n>0 && sum(n-1)
    }
    sum(n)
    return ans
  
}
```



## [剑指 Offer 65. 不用加减乘除做加法](https://leetcode-cn.com/problems/bu-yong-jia-jian-cheng-chu-zuo-jia-fa-lcof/)

> 位运算

计算机内都是补码**补码的优势：正负数统一处理，CPU只有加法器**

<img src="https://pic.leetcode-cn.com/56d56524d8d2b1318f78e209fffe0e266f97631178f6bfd627db85fcd2503205-Picture1.png" alt="Picture1.png" style="zoom:50%;" />

```go
func add(a int, b int) int {
    for b!=0{		//直到进位和为0 退出循环
        tmp:=a^b	//非进位和
        b=(a&b)<<1  //进位和
        a=tmp
    }
    return a
}
```

## [剑指 Offer 66. 构建乘积数组](https://leetcode-cn.com/problems/gou-jian-cheng-ji-shu-zu-lcof/)

<img src="https://cdn.jsdelivr.net/gh/Jason-Wu-1999/blog.img/imgs/1627495578-jXwUKy-image.png" alt="image.png" style="zoom:50%;" />

通过两次遍历：

1. 第一次算出下三角的每行的乘积
2. 第二次算出上三角的每行的乘积，并且和当前行的下三角的积相乘

```go
func constructArr(a []int) []int{
    if len(a)==0{
        return nil
    }
    b:=make([]int,len(a))
    b[0]=1
    tmp:=1 
    // 算出下三角
    for i:=1;i<len(a);i++{
        b[i]=b[i-1] * a[i-1]
    }
    // 算上三角
    for i:=len(a)-2;i>=0;i--{
        tmp*=a[i+1]
        b[i]*=tmp
    }
    return b

}
```

## [剑指 Offer 67. 把字符串转换成整数](https://leetcode-cn.com/problems/ba-zi-fu-chuan-zhuan-huan-cheng-zheng-shu-lcof/)

> 需要注意的是 ‘0’在计算机内为ascii码值，不能直接转换  使用  v-'0'

```go
func strToInt(str string) int {
    //除去空格
    //使用库函数
 	// str = strings.TrimSpace(str)
    
    //不使用库函数
    
    for len(str) > 0 && str[0] == ' '{
        str = str[1:]
       
    } 
	result := 0
	sign := 1
    //遍历字符串  判断是数字还是符号还是其他   考虑是否有效
	for i, v := range str {
  
		if v >= '0' && v <= '9' {
			result = result*10 + int(v-'0')
		} else if v == '-' && i == 0 {
			sign = -1
		} else if v == '+' && i == 0 {
			sign = 1
		} else {
			break
		}

    // 处理越界的情况
        if result > math.MaxInt32 {
            if sign == -1 {
                return math.MinInt32
            }
            return math.MaxInt32
        }

    }
    return result* sign
}
```

