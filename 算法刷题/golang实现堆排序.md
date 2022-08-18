# 使用实例

> 堆排序

==利用heap标准库实现==

## 标准库

```go
package main

import (
	"container/heap"
	"fmt"
)

type IntHeap []int

func (h IntHeap) Len() int           { return len(h) }
func (h IntHeap) Less(i, j int) bool { return h[i] < h[j] }
func (h IntHeap) Swap(i, j int)      { h[i], h[j] = h[j], h[i] }

func (h *IntHeap) Push(x interface{}) {
	*h = append(*h, x.(int))
}

func (h *IntHeap) Pop() interface{} {
	old := *h
	n := len(old)
	x := old[n-1]
	*h = old[0 : n-1]
	return x
}

func main() {
	h := &IntHeap{2, 1, 5, 100, 3, 6, 4, 5}
	heap.Init(h)
	heap.Push(h, 3)
	fmt.Printf("minimum: %d\n", (*h)[0])
	for h.Len() > 0 {
		fmt.Printf("%d ", heap.Pop(h))
	}
}

```

heap是常用的实现优先队列的方法。heap包对任意实现了heap接口的类型提供堆操作。

接口如下：

```go
type Interface interface {
	sort.Interface
	Push(x interface{}) // add x as element Len()
	Pop() interface{}   // remove and return element Len() - 1.
}
//其中sort.Interface为

type Interface interface {	
	Len() int
	Less(i, j int) bool
	Swap(i, j int)
}

```

所以实现这5个方法就是实现了Interface接口，可以通过heap包进行堆操作





# 源码分析

以实现小堆顶为例（小堆顶大队顶只需要在实现Less函数是进行修改即可）

## 插入数据Push

首先，新添加的元素加入末尾。为了保持最小堆的性质，需要沿着其祖先的路径，**自下而上**依次比较和交换该结点与父结点的位置，直到重新满足堆的性质为止。

所以需要==向上比较,==将其和父节点比较

- 若大于父节点则满足最小堆特性，无需交换

- 小于，则需要和父节点交换，并继续向上比较知道符合最小堆

```go
func (h Heap) up(i int) {
	for {
		f := (i - 1) / 2 // 父亲结点
		if i == f || h.less(f, i) {
			break
		}
		h.swap(f, i)
		i = f
	}
}

func (h *Heap) Push(x int) {
	*h = append(*h, x)
	h.up(len(*h) - 1)
}


```

## 移除指定数据Remove

1. 首先把最末端的结点填入要删除节点的位置，
2. 然后删除末端元素，同理，这样做也可能导致破坏最小堆的堆序特性

需要判断是向上比较还是向下比较

- 若当前节点大于父节点，只需要向下比较
- 小于则向上比较

```go
// 删除堆中位置为i的元素
// 返回被删元素的值
func (h *Heap) Remove(i int) (int, bool) {
	if i < 0 || i > len(*h)-1 {
		return 0, false
	}
	n := len(*h) - 1
	h.swap(i, n) // 用最后的元素值替换被删除元素
	// 删除最后的元素
	x := (*h)[n]
	*h = (*h)[0:n]
	// 如果当前元素大于父结点，向下筛选
	if (*h)[i] > (*h)[(i-1)/2] {
		h.down(i)
	} else { // 当前元素小于父结点，向上筛选
		h.up(i)
	}
	return x, true
}

func (h Heap) down(i int) {
	for {
		l := 2*i + 1 // 左孩子
		if l >= len(h) {
			break // i已经是叶子结点了
		}
		j := l
		if r := l + 1; r < len(h) && h.less(r, l) {
			j = r // 右孩子
		}
		if h.less(i, j) {
			break // 如果父结点比孩子结点小，则不交换
		}
		h.swap(i, j) // 交换父结点和子结点
		i = j        //继续向下比较
	}
}
```

## 弹出Pop

其实就是移除  i=0  的数据

只是弹出的是最上面的数据，所以肯定是向下比较

```go
// 弹出堆顶的元素，并返回其值
func (h *Heap) Pop() int {
	n := len(*h) - 1
	h.swap(0, n)
	x := (*h)[n]
	*h = (*h)[0:n]
	h.down(0)
	return x
}
```

### 初始化-Init

在我们讲完了堆的核心操作 `up` 和 `down` 后，我们来讲如何根据一个数组构造一个最小堆。

其实我们可以写个循环，然后将各个元素依次 `push` 进去，但是这次我们利用数学规律，直接由一个数组构造最小堆。

首先，将所有关键码放到一维数组中，此时形成的完全二叉树并不具备最小堆的特征，但是仅包含叶子结点的子树已经是堆。

即在有n个结点的完全二叉树中，当 `i>n/2-1` 时，以i结点为根的子树已经是堆。

```go
func (h Heap) Init() {
	n := len(h)
	// i > n/2-1 的结点为叶子结点本身已经是堆了
	for i := n/2 - 1; i >= 0; i-- {
		h.down(i)
	}
}
```



## 自己实现排序

```

package main

import "fmt"

func main() {
	n:=0
	fmt.Scanln(&n)
	heapSize:=n
	nums:=make([]int,n)
	fmt.Println(n)
	for i := 0; i < n; i++ {
		fmt.Scan(&nums[i])
	}

	fmt.Println("排序之前",nums)
	//初始化堆  构建大顶堆
	buildMaxHeap(nums,n)

// 然后开始把第一个和最后一个元素换，换完再来构建大顶堆
	for i := len(nums)-1; i >=0; i-- {
		//把最大值换到最后
		nums[0], nums[i] = nums[i], nums[0]
		heapSize--
		maxHeapify(nums, 0, heapSize)
	}

	fmt.Println(nums)
}

func buildMaxHeap(nums []int, heapSize int){
	for i:=heapSize/2-1;i>=0;i--{
		//	非叶子节点  下沉
		maxHeapify(nums, i, heapSize)
	}
}

//下沉n
func maxHeapify(nums []int, i int, heapSize int) {
	l,r,largest:=2*i+1,2*i+2,i

	if l<heapSize && nums[l]>nums[largest]{
		//交换
		largest=l
	}
	if r<heapSize && nums[r]>nums[largest]{
		//交换
		largest=r
	}
	//判断他是否和子节点有交换，如果交换了，想要继续下沉
	if largest != i {
		nums[i], nums[largest] = nums[largest], nums[i]
		maxHeapify(nums, largest, heapSize)
	}
}
```





```
func sortArray(nums []int) []int {
    size := len(nums)
    build(nums,size)
    // 开始拿出最大值
    for i:=size -1; i >= 0; i--{
        nums[0], nums[i] = nums[i], nums[0]
        size--
        heap(nums, 0, size)
    } 

    return nums
}

func build(nums []int, size int) {
    for i := size/2; i >= 0 ; i--{
        heap(nums, i, size)
    }
}

func heap(nums []int, start, size int) {
    for {
        child := 2 * start + 1
        if child >= size{
            return 
        } 
        // 从左右子节点中找出最大值 
        if child + 1 <= size-1 && nums[child] < nums[child+1] {
            child++
        } 

        if nums[start] > nums[child] {
			return
		}
        // 拿开始的节点和子节点中找到的最大值比较
        // 下沉
       
        nums[start], nums[child] = nums[child], nums[start]
        start = child
        
    }
    
}
```

