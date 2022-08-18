## sync.Once

### 实现原理

```go
// Once is an object that will perform exactly one action.
type Once struct {
	// 0未执行，1执行了
	done uint32
	// 互斥锁
	m    Mutex
}
```

里面就一个对外的函数

```go
func (o *Once) Do(f func()) {
	// 原子的读取done的值，如果为0代表onec第一次的执行还没有出发
	if atomic.LoadUint32(&o.done) == 0 {
		// 执行
		o.doSlow(f)
	}
}

func (o *Once) doSlow(f func()) {
	// 加锁
	o.m.Lock()
	defer o.m.Unlock()
	// 判断done变量为0表示还没执行第一次
	if o.done == 0 {
		// 计数器原子的加一
		defer atomic.StoreUint32(&o.done, 1)
		// 执行传入的函数
		f()
	}
}
```

### 总结

1、总体上也是很简单一个计数器，一把互斥锁，通过`atomic.LoadUint32`的原子读取技术器中的值；

2、如果计数器中的值为0表示还没有执行；

3、加锁，执行传入的函数，然后通过`atomic.StoreUint32`原子的对计数器的值进行加一操作；

4、完成。



双检,但是是原子操作,==原子读取和原子加一==

