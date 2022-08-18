在go语言中for range是非常常见的循环方式 对slice、 map、channel 特别是后两种类型

但是在使用的过程中有几个坑要注意：

## 案例一

**遍历取所有元素地址？**  代码如下

```
package main
 
import "fmt"
 
func main() {
	arr := []int{1,2}
	res := []*int{}
 
	for _, v := range arr {
		res = append(res, &v)
	}
 
	fmt.Println(*res[0], *res[1])
}
```

输出结果为  

```
2 2 
```

### 原因

for range 在底层是首先 创建 两个变量 v1,v2来接收（以切片为例）切片的下标和值，v1和v2只被创建一次也就是地址是相同的，只是值在不停的覆盖

**底层等效代码如下**

```
newArr:=arr    //将arr复制一份
v1，v2：=0，0
for ;v1<len(newArr);v1++{
		tmp:=arr[v1]
		v1,v2:=v1,tmp
	}
```

**==所以在取地址的时候取得是同一个==**

**正确代码** 添加临时变量

```
package main

import "fmt"

func main() {
	arr := []int{1,2}
	res := []*int{}
 
	for _, v := range arr {
		tmp:=v
		res = append(res, &tmp)
	}
 
	fmt.Println(*res[0], *res[1])
}
```



## 案例二

**在遍历中起协程？**

```
package main
 
import (
	"fmt"
	"sync"
)
 
func main() {
	var m = []int{1, 2, 3}
	var wg sync.WaitGroup
	for i := range m {
		wg.Add(1)
 		go func() {
			fmt.Print(i)
			wg.Done()
    	}()
	}
	wg.Wait()
 
}
```

输出结果：

```undefined
222
```

### 原因有二：

**1，golang是值拷贝传递**；2，for循环很快就执行完了，但是**创建的10个协程需要做初始化**：上下文准备，堆栈，和内核态的线程映射关系的工作，是需要时间的，比for慢，等都准备好了的时候，会同时访问变量temp 。这个时候的temp肯定是for执行完成后的数字10。所以10个协程都打印10。（也可能有个别的协程已经准备好了，取temp的时候，正好是5，或者7，就输出了这些数字

**正确代码**

方法一  以参数的方式传入

```
package main
 
import (
	"fmt"
	"sync"
)
 
func main() {
	var m = []int{1, 2, 3}
	var wg sync.WaitGroup
	for i := range m {
		wg.Add(1)
 		go func(i int) {         //将i以参数的方式传入到协程里面
			fmt.Print(i)
			wg.Done()
    	}(i)
	}
	wg.Wait()
 
}
```

方法二：使用局部变量拷贝

```
package main
 
import (
	"fmt"
	"sync"
)
 
func main() {
	var m = []int{1, 2, 3}
	var wg sync.WaitGroup
	for i := range m {
		wg.Add(1)
		i := i
 		go func() {
			fmt.Print(i)
			wg.Done()
    	}()
	}
	wg.Wait()
 
}
```

