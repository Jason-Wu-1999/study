# 怎么优雅地关闭[通道](https://so.csdn.net/so/search?q=通道&spm=1001.2101.3001.7020)

------

许多天前，我写了一篇解释[Go中Channel法则](https://go101.org/article/channel.html)的文章。这文章在reddit和HN上获得了许多赞，但是评论区中也有许多对于Go channel(通道)设计细节的质疑。

我总结了一下，主要有这些：

1. 没有简单通用的方法检查是否一个通道已经关闭，不需要改变通道状态的那种。
2. 关闭一个已经关闭的通道会panic，所以如果你不知道是否通道已经关闭，去关闭它是很危险的。
3. 往一个已经关闭的通道发送值会panic，所以如果你不知道是否通道已经关闭，往通道发送值是很危险的。

这些质疑看起来很有道理(其实并没有)。是的，确实没有一个内建的检查一个通道是否已关闭的函数。

但如果你确定曾经以及未来不会给某个通道发送任何值，其实是有个可以简单地检查一个通道是否已经关闭的方法：

```go
package main

import "fmt"

type T int

func IsClosed(ch <-chan T) bool {
	select {
	case <-ch:
		return true
	default:
	}

	return false
}

func main() {
	c := make(chan T)
	fmt.Println(IsClosed(c)) // false
	close(c)
	fmt.Println(IsClosed(c)) // true
}
12345678910111213141516171819202122
```

如上所述，这不是检查通道是否关闭的通用方法。

事实上，即使有一个简单的内建`closed`函数能检查一个通道是否已关闭，它的用处也会很有限，就像用来检查通道中当前值的个数的内建函数`len`。这是因为被检查的通道的状态可能会在函数返回后瞬间改变，所以返回的那个值并没法反映那个通道的最新状态。尽管当调用`closed(ch)`的返回值是`true`时，停止向通道 `ch`发送值是没问题的。但当返回值是`false`时并不代表你就可以安全地往那个通道发送值或者关闭它。

# 通道的关闭原则

使用Go通道的一个基本原则就是不要在接收方关闭通道，以及如果有多个并发发送方的话在发送方也不行。换句话说，我们仅应该在一个发送goroutine中关闭通道，如果这个发送者是通道唯一的发送方的话。

(之后，我们会把以上原则称为通道关闭原则。)

自然，这不是关闭通道的通用原则。通用原则是不要关闭（或者发送值到）关闭了的通道。如果我们可以保证再也没有其他goroutine会关闭及发送值到一个未关闭非nil的通道，那这个goroutine就可以安全地关闭这个通道了。然而，由通道的一个接收方或许多发送方来实现这一点通常需要付出许多努力，常常会使得代码很复杂。相对地，如果我们遵从上面提到的通道关闭原则，就简单地多了。

# 简单粗暴的方案

如果你无论如何都要从通道的接收方或者多个发送方关闭一个通道的话，你可以使用recover机制防止可能的panic把你的程序搞崩。这里是示例(假设通道的元素类型是`T`)：

```go
func SafeClose(ch chan T) (justClosed bool) {
	defer func() {
		if recover() != nil {
			// The return result can be altered
			// in a defer function call.
			justClosed = false
		}
	}()

	// assume ch != nil here.
	close(ch)   // panic if ch is closed
	return true // <=> justClosed = true; return
}
12345678910111213
```

这个方案显然没有遵守通道关闭原则。

也可以用同一个方法来发送值到也许关闭了的通道。

```go
func SafeSend(ch chan T, value T) (closed bool) {
	defer func() {
		if recover() != nil {
			closed = true
		}
	}()

	ch <- value  // panic if ch is closed
	return false // <=> closed = false; return
}
12345678910
```

这个简单粗暴的方案不仅打破了通道关闭原则，也可能在进程内发生data race。

# 礼貌的方案

许多人喜欢使用`sync.Once`来关闭通道：

```go
type MyChannel struct {
	C    chan T
	once sync.Once
}

func NewMyChannel() *MyChannel {
	return &MyChannel{C: make(chan T)}
}

func (mc *MyChannel) SafeClose() {
	mc.once.Do(func() {
		close(mc.C)
	})
}
1234567891011121314
```

当然，我们也可以使用`sync.Mutex`来避免多次关闭一通道：

```go
type MyChannel struct {
	C      chan T
	closed bool
	mutex  sync.Mutex
}

func NewMyChannel() *MyChannel {
	return &MyChannel{C: make(chan T)}
}

func (mc *MyChannel) SafeClose() {
	mc.mutex.Lock()
	defer mc.mutex.Unlock()
	if !mc.closed {
		close(mc.C)
		mc.closed = true
	}
}

func (mc *MyChannel) IsClosed() bool {
	mc.mutex.Lock()
	defer mc.mutex.Unlock()
	return mc.closed
}
123456789101112131415161718192021222324
```

这些方案可能很有礼貌，但是他们可能没法避免data races。当前，Go规范不保证当关闭通道的同时有发送操作不会发生data races。如果`SafeClose`函数与一个通道发送操作对同个通道并发执行，可能会发生data race(尽管data race通常没有什么危害)

# 优雅的方案

上面`SafeSend`函数的一个缺点是没法将它和`select` `case`语句结合使用来进行发送操作。以上`SafeSend`和`SafeClose`函数的缺点是许多人会觉得上面使用 `panic`/`recover`机制和`sync`包的方案不够优雅，包括我。下面会介绍一些纯粹的方案，不使用 `panic`/`recover`机制和`sync`包，并且适用于各种情况的那种。

(在以下示例中，`sync.WaitGroup`被用来实现这些示例。这也许在实际使用中不总是适用)

1. M 个接收者，一个发送者，发送者通过关闭数据通道来告知"没有更多要发送的了"
   这是最简单的场景，只要让发送者在他不想要发更多的时候直接关闭数据通道就行了。

```go
package main

import (
	"time"
	"math/rand"
	"sync"
	"log"
)

func main() {
	rand.Seed(time.Now().UnixNano())
	log.SetFlags(0)

	// ...
	const Max = 100000
	const NumReceivers = 100

	wgReceivers := sync.WaitGroup{}
	wgReceivers.Add(NumReceivers)

	// ...
	dataCh := make(chan int)

	// 发送者
	go func() {
		for {
			if value := rand.Intn(Max); value == 0 {
				// 仅有一个发送者时可以在任意时候安全地关闭通道
				close(dataCh)
				return
			} else {
				dataCh <- value
			}
		}
	}()

	// 接受者们
	for i := 0; i < NumReceivers; i++ {
		go func() {
			defer wgReceivers.Done()

			// 不断接收值一直到dataCh被关闭，且没有缓存在其中的值了。
			for value := range dataCh {
				log.Println(value)
			}
		}()
	}

	wgReceivers.Wait()
}
1234567891011121314151617181920212223242526272829303132333435363738394041424344454647484950
```

1. 一个接受者，N个发送者，这唯一的接受者通过关闭一个额外的通知通道来告知“请停止发送更多”
   这种场景比上一种稍微复杂些。我们不能让接收方通过关闭数据通道来停止数据传输，因为这会打破通道关闭原则。但是我们可以让接收者关闭一个额外的通知通道来告知发送者停止发送。

```go
package main

import (
	"time"
	"math/rand"
	"sync"
	"log"
)

func main() {
	rand.Seed(time.Now().UnixNano())
	log.SetFlags(0)

	// ...
	const Max = 100000
	const NumSenders = 1000

	wgReceivers := sync.WaitGroup{}
	wgReceivers.Add(1)

	// ...
	dataCh := make(chan int)
	stopCh := make(chan struct{})
		// stopCh是一个额外的通知通道，它的发送者是dataCh通道的接收者，
		// 接收者们则是dataCh通道的发送者们。 

	// 发送者们
	for i := 0; i < NumSenders; i++ {
		go func() {
			for {
				// 这尝试接收操作是为了尽快退出goroutine。
				// 在这个特殊示例中，这并不是必要的。
				select {
				case <- stopCh:
					return
				default:
				}

				// 即使stopCh已经关闭了，如果dataCh未被阻塞住，
				// 这第二个select里的第一个分支也不是必定会被选择的。
				// 但对于这个示例，这是可接受的，所以前面那个select
				// 语句块是可以不要的。
				select {
				case <- stopCh:
					return
				case dataCh <- rand.Intn(Max):
				}
			}
		}()
	}

	// 接收者
	go func() {
		defer wgReceivers.Done()

		for value := range dataCh {
			if value == Max-1 {
				// dataCh通道的接收者同时也是stopch的发送者。
				// 在这里关闭stopCh通道是安全的。T
				close(stopCh)
				return
			}

			log.Println(value)
		}
	}()

	// ...
	wgReceivers.Wait()
}
12345678910111213141516171819202122232425262728293031323334353637383940414243444546474849505152535455565758596061626364656667686970
```

如注释中提到的，对于额外的通知通道，它的发送者是数据通道的接收者。这额外的通知通道仅会被它的唯一的发送者关闭，这符合通道关闭原则。

在这个例子中，通道`dataCh`永远不会关闭。是的，通道并不是必须关闭。当没有goroutine引用它时，一个通道最终会被垃圾回收，不管它是否已关闭。所以这里说的优雅地关闭通道并不是说 close 通道。

1. M个接受者，N个发送者，任意一方通过通知moderator关闭一个额外的通知通道来说“我们结束游戏吧”
   这是最复杂的场景。我们不能让任何接收者或发送者关闭数据通道。我们也不能让接收者之一关闭一个额外的通知通道来通知所有的发送者和接收者退出游戏。这两个方案都会打破通道关闭原则。但是我们可以引入一个moderator来关闭额外的通知通道。下例中的窍门是怎么使用一个try-send操作来通知moderator去关闭额外的通知通道。

```go
package main

import (
	"time"
	"math/rand"
	"sync"
	"log"
	"strconv"
)

func main() {
	rand.Seed(time.Now().UnixNano())
	log.SetFlags(0)

	// ...
	const Max = 100000
	const NumReceivers = 10
	const NumSenders = 1000

	wgReceivers := sync.WaitGroup{}
	wgReceivers.Add(NumReceivers)

	// ...
	dataCh := make(chan int)
	stopCh := make(chan struct{})
		// stopCh是一个额外的通知通道。
		// 它的发送者是下面的moderator goroutine，
		// 它的接收者是datach的所有发送者和接收者
	toStop := make(chan string, 1)
		// toStop是用来通知moderator关闭额外的通知通道(stopCh)的。
		// 它的发送者是dataCh的任意发送者和接收者，它的接收者是下面
		// 的moderator goroutine 。
		// 它必须是个带缓冲的通道

	var stoppedBy string

	// moderator
	go func() {
		stoppedBy = <-toStop
		close(stopCh)
	}()

	// 发送者
	for i := 0; i < NumSenders; i++ {
		go func(id string) {
			for {
				value := rand.Intn(Max)
				if value == 0 {
					// 这里的try-send操作是为了通知moderator
					// 关闭额外的通知通道。
					select {
					case toStop <- "sender#" + id:
					default:
					}
					return
				}

				// 这儿的try-receive操作是为了尽早退出发送者
				// goroutine. 
				// Try-receive和try-send select语句块被标准
				// Go编译器针对优化过，所以十分高效。
				select {
				case <- stopCh:
					return
				default:
				}

				// 即使stopCh关闭了，这个select语句块的第一个分支
				// 也仍然可能在几次循环中(理论上可以永远)不被选择，
				// 如果dataCh的发送也是非阻塞的就有可能。如果无法
				// 接受这一点的话，上面的try-receive操作就是必要的了。
				select {
				case <- stopCh:
					return
				case dataCh <- value:
				}
			}
		}(strconv.Itoa(i))
	}

	// 接收者
	for i := 0; i < NumReceivers; i++ {
		go func(id string) {
			defer wgReceivers.Done()

			for {
				// 与发送者goroutine一样，这儿的try-receive
				// 操作是为了尽早退出接受者 goroutine. 
				select {
				case <- stopCh:
					return
				default:
				}
				
				// 即使stopCh关闭了，这个select语句块的第一个分支
				// 也仍然可能在几次循环中(理论上可以永远)不被选择，
				// 如果dataCh的发送也是非阻塞的就有可能。如果无法
				// 接受这一点的话，上面的try-receive操作就是必要的了。
				select {
				case <- stopCh:
					return
				case value := <-dataCh:
					if value == Max-1 {
						// 这里使用了同样的技巧来通知
						// moderator关闭额外的通知通道。
						select {
						case toStop <- "receiver#" + id:
						default:
						}
						return
					}

					log.Println(value)
				}
			}
		}(strconv.Itoa(i))
	}

	// ...
	wgReceivers.Wait()
	log.Println("stopped by", stoppedBy)
}
123456789101112131415161718192021222324252627282930313233343536373839404142434445464748495051525354555657585960616263646566676869707172737475767778798081828384858687888990919293949596979899100101102103104105106107108109110111112113114115116117118119120121122
```

在这个示例中，仍然遵守了通道关闭原则。

请注意通道`toStop`的缓冲大小(容量)是1。这是为了避免在moderator goroutine准备好接收来自`toStop`的通知前的首个通知被错过。

我们也可以设置`toStop`通道的容量为发送者和接收者的总数，这样我们就不需要try-send`select`语句块来通知moderator了。

```go
...
toStop := make(chan string, NumReceivers + NumSenders)
...
			value := rand.Intn(Max)
			if value == 0 {
				toStop <- "sender#" + id
				return
			}
...
				if value == Max-1 {
					toStop <- "receiver#" + id
					return
				}
...
1234567891011121314
```

4."M个接收者，一个发送者"场景的一种变化：关闭请求是由一个第三方goroutine发起的。
有时，我们需要让第三方goroutine发起关闭通知。对于这种场景，我们可以使用一个额外的通知通道来通知发送者关闭数据通道，比如：

```go
package main

import (
	"time"
	"math/rand"
	"sync"
	"log"
)

func main() {
	rand.Seed(time.Now().UnixNano())
	log.SetFlags(0)

	// ...
	const Max = 100000
	const NumReceivers = 100
	const NumThirdParties = 15

	wgReceivers := sync.WaitGroup{}
	wgReceivers.Add(NumReceivers)

	// ...
	dataCh := make(chan int)
	closing := make(chan struct{}) // 通知通道
	closed := make(chan struct{})
	
	// stop函数可以被安全地调用多次。
	stop := func() {
		select {
		case closing<-struct{}{}:
			<-closed
		case <-closed:
		}
	}
	
	// 一些第三方goroutines
	for i := 0; i < NumThirdParties; i++ {
		go func() {
			r := 1 + rand.Intn(3)
			time.Sleep(time.Duration(r) * time.Second)
			stop()
		}()
	}

	// 发送者
	go func() {
		defer func() {
			close(closed)
			close(dataCh)
		}()

		for {
			select{
			case <-closing: return
			default:
			}

			select{
			case <-closing: return
			case dataCh <- rand.Intn(Max):
			}
		}
	}()

	// 接收者
	for i := 0; i < NumReceivers; i++ {
		go func() {
			defer wgReceivers.Done()

			for value := range dataCh {
				log.Println(value)
			}
		}()
	}

	wgReceivers.Wait()
}
1234567891011121314151617181920212223242526272829303132333435363738394041424344454647484950515253545556575859606162636465666768697071727374757677
```

`stop`函数的点子是从Roger Peppe的评论中来的。

1. “N个发送者”场景的一种变体：必须要关闭数据通道来通知所有接收者说数据发送结束了
   在以上N个发送者场景的方案中，为了遵守通道关闭原则，我们避免了关闭数据通道。但是，有时，数据通道必须最后被关闭来让接收者们知道数据发送结束了。对于这种情况，我们可以通过一个中间通道来翻译N个发送者场景为一个发送者场景。中间通道只有一个发送者，所以我们可以关闭它而不是关闭原始的数据通道。

```go
package main

import (
	"time"
	"math/rand"
	"sync"
	"log"
	"strconv"
)

func main() {
	rand.Seed(time.Now().UnixNano())
	log.SetFlags(0)

	// ...
	const Max = 1000000
	const NumReceivers = 10
	const NumSenders = 1000
	const NumThirdParties = 15

	wgReceivers := sync.WaitGroup{}
	wgReceivers.Add(NumReceivers)

	// ...
	dataCh := make(chan int)     // 将被关闭
	middleCh := make(chan int)   // 永远不会被关闭
	closing := make(chan string) // 通知通道
	closed := make(chan struct{})

	var stoppedBy string

	// stop函数可以安全地被多次调用。
	stop := func(by string) {
		select {
		case closing <- by:
			<-closed
		case <-closed:
		}
	}
	
	// 中间层
	go func() {
		exit := func(v int, needSend bool) {
			close(closed)
			if needSend {
				dataCh <- v
			}
			close(dataCh)
		}

		for {
			select {
			case stoppedBy = <-closing:
				exit(0, false)
				return
			case v := <- middleCh:
				select {
				case stoppedBy = <-closing:
					exit(v, true)
					return
				case dataCh <- v:
				}
			}
		}
	}()
	
	// 一些第三方goroutines
	for i := 0; i < NumThirdParties; i++ {
		go func(id string) {
			r := 1 + rand.Intn(3)
			time.Sleep(time.Duration(r) * time.Second)
			stop("3rd-party#" + id)
		}(strconv.Itoa(i))
	}

	// 发送者
	for i := 0; i < NumSenders; i++ {
		go func(id string) {
			for {
				value := rand.Intn(Max)
				if value == 0 {
					stop("sender#" + id)
					return
				}

				select {
				case <- closed:
					return
				default:
				}

				select {
				case <- closed:
					return
				case middleCh <- value:
				}
			}
		}(strconv.Itoa(i))
	}

	// 接收者
	for range [NumReceivers]struct{}{} {
		go func() {
			defer wgReceivers.Done()

			for value := range dataCh {
				log.Println(value)
			}
		}()
	}

	// ...
	wgReceivers.Wait()
	log.Println("stopped by", stoppedBy)
}
123456789101112131415161718192021222324252627282930313233343536373839404142434445464748495051525354555657585960616263646566676869707172737475767778798081828384858687888990919293949596979899100101102103104105106107108109110111112113114115
```

# 更多场景？