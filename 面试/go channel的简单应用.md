## 使用三个Goroutine 交替打印 abc

使用`channel`和`WaitGroup`

```
package main

import (
	"fmt"
	"sync"
)
func main() {
	var ch1, ch2, ch3 = make(chan struct{}), make(chan struct{}), make(chan struct{})
	var wg sync.WaitGroup
	wg.Add(3)
	go func(s string) {
		defer wg.Done()
		for i := 1; i <= 10; i++ {
			<- ch1
			fmt.Print(s)
			ch2 <- struct{}{}
		}
		<- ch1   //从ch1中接收但是忽略结果
	}("A")
	go func(s string) {
		defer wg.Done()
		for i := 1; i <= 10; i++ {
			<- ch2
			fmt.Print(s)
			ch3 <- struct{}{}
		}
	}("B")
	go func(s string) {
		defer wg.Done()
		for i := 1; i <= 10; i++ {
			<- ch3
			fmt.Println(s)
			ch1 <- struct{}{} 
		}
	}("C")
	ch1 <- struct{}{}
	wg.Wait()
}
```



## 控制并发数量

多个并发的情况下控制最大的并发数 

> 往里面写

```go

package main
 
import (
  "fmt"
  "sync"
  "time"
)
 //最大限度结构体
type Glimit struct {
  n int                  //并发的最大数 用来初始化 chan的缓存空间
  c chan struct{}		//channel
}
 
// initialization Glimit struct
func New(n int) *Glimit {
  return &Glimit{
    n: n,
    c: make(chan struct{}, n),
  }
}
// Run f in a new goroutine but with limit.
func (g *Glimit) Run(f func()) {
  g.c <- struct{}{}
  go func() {
    f()
    <-g.c
  }()
}
 
var wg = sync.WaitGroup{}
 
func main() {
  number := 10
  g := New(2)
  for i := 0; i < number; i++ {
    wg.Add(1)
    value :=i
    goFunc := func() {
    
      // 做一些业务逻辑处理
      fmt.Printf("go func: %dn", value)
      time.Sleep(time.Second)
      wg.Done()
    }
    g.Run(goFunc)
  }
  wg.Wait()
 }

```





## 使用channel实现互斥锁

> 注意使用的是带一个缓存区的channel

```go
type mutex struct {
	ch chan struct{}
}

func newMutex()*mutex{
	c := make(chan struct{}, 1)
	c <- struct{}{}
	return &mutex{
		ch : c,
	}
}

// Lock 请求锁，直到获取到
func (m *mutex)Lock(){
	<- m.ch

}

func (m *mutex)Unlock(){
	select {
		case m.ch <- struct{}{}:
		default:
			panic("Unlock of Unlock mutex")
	}
}
```



## 实现tryLock()

> tryLock()：会立刻返回是否获取到锁， 不会阻塞或者自旋

```
func (m *mutex)tryLock()bool{
	select {
	case <-m.ch:
		return true
	default:

	}
	return false
}
```

