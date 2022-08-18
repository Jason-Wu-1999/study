高并发，golang推出的最大卖点之一；那么，在并发中，临界资源的同步又是怎么保护的呢？

今天，我们从浅入深的解析解析golang的互斥锁对象Mutex



注释说明：本文源码基于1.16版本进行分析解读

## 简单使用

```text
var mu Sync.Mutex
mu.Lock()
//do somethings
mu.Unlock()
```

使用非常简单，Lock()进行加锁，Unlock()进行解锁；加锁失败时，Lock()方法堵塞，等待直至获取锁

## 实现原理分析

- 结构定义

走进golang源码，先看下Sync.Lock的定义

```text
type Mutex struct {
	state int32
	sema  uint32
}
```

非常简单，就两个成员：

1. state
2. sema

其中，state采用不同位表示不同含义的方式进行定义的，按从低到高位一次排列：

1. 第一位表示加锁状态：0表示未加锁，1表示已加锁
2. 第二位表示唤醒状态：0表示未设置被唤醒状态，1表示已设置被唤醒状态
3. 第三位表示运行状态：0表示正常状态，1表示饥饿状态
4. 第四位以及其他字节位：表示等待获取锁的数目

另外的sema为信号量，通过其P/V操作实现锁的等待和唤醒

## **加锁逻辑**

1. **快速加锁**

```text
if atomic.CompareAndSwapInt32(&m.state, 0, mutexLocked) {
       return
}
```

采用CAS实现快速加锁

**2. 缓慢加锁（slowLock）**

为了充分提高加锁效率，golang的mutex的运行状态有两种：正常状态和饥饿状态

正常状态下：刚进入锁的协程可通过自旋锁的方式快速获取锁；

饥饿模式下：所有锁的获取者都通过FIFO的方式公平的获取锁资源



- 自旋锁

从操作系统的角度来看，相比睡眠方式的互斥锁，自旋锁通过自旋占用CPU，避免的CPU的上下文切换，因此在持锁时长较短的情况下，性能更优

golang的mutex在实现时，充分考虑了自旋锁跟睡眠锁的取舍；即：在持锁时长较短时，采用自旋锁，在持锁时长较大时，采用睡眠锁

再看看其源码实现

**slowLock part1**

```text
if old&(mutexLocked|mutexStarving) == mutexLocked && runtime_canSpin(iter) {
	if !awoke && old&mutexWoken == 0 && old>>mutexWaiterShift != 0 &&
		atomic.CompareAndSwapInt32(&m.state, old, old|mutexWoken) {
		awoke = true
	}
        runtime_doSpin()
	iter++
	old = m.state
	continue
}
new := old
if old&mutexStarving == 0 {
	new |= mutexLocked
}
if atomic.CompareAndSwapInt32(&m.state, old, new) {
	if old&(mutexLocked|mutexStarving) == 0 {
		break // locked the mutex with CAS
}
```

非饥饿状态下，自旋后获取新的state锁信息，如果发现锁状态为未加锁状态，则使用CAS获取锁

**runtime_canSpin**

```text
//go:linkname sync_runtime_canSpin sync.runtime_canSpin
func sync_runtime_canSpin(i int) bool {
	if i >= active_spin || ncpu <= 1 || gomaxprocs <= int32(sched.npidle+sched.nmspinning)+1 {
		return false
	}
	if p := getg().m.p.ptr(); !runqempty(p) {
		return false
	}
	return true
}
```

**源码解析，在以下情况下不可进入自旋：**

> 1. 迭代次数(入参i) 大于等于4
> 2. cpu为单核
> 3. GOMAXPROCS大于1且至少有一个其他处理运行状态的P
> 4. 当前P没有其他等待运行的G
>

**doSpin**

```text
/go:linkname sync_runtime_doSpin sync.runtime_doSpin
func sync_runtime_doSpin() {
    procyield(active_spin_cnt)
}
```

深入分析，procyield底层采用了循环条用PAUSE和JNZ指令



**饥饿状态**

正常模式下，由于刚进入锁的协程有较高的获取锁优势，所以可能会出现某个进程一直获取不到锁的情况发生，golang的Mutex为了避免这种情况，引入了饥饿状态

在某个协程超过1ms依旧没有获取到锁的情况下，mutex进入饥饿模式；进入饥饿模式后，刚进入锁的协程不进行自旋，直接放入到sema信号量的队尾，并将锁的等待着加1；



依旧看看其源码实现

```text
if old&(mutexLocked|mutexStarving) == mutexLocked && runtime_canSpin(iter) {
    //非饥饿模式下进入自旋
	continue
}
new := old

if old&(mutexLocked|mutexStarving) != 0 {//锁等待着+1
	new += 1 << mutexWaiterShift
}

if starving && old&mutexLocked != 0 {
	new |= mutexStarving
}
if awoke {
	if new&mutexWoken == 0 {
		throw("sync: inconsistent mutex state")
	}
	new &^= mutexWoken
}
if atomic.CompareAndSwapInt32(&m.state, old, new) {
	if old&(mutexLocked|mutexStarving) == 0 {
		break // locked the mutex with CAS
	}
	queueLifo := waitStartTime != 0
	if waitStartTime == 0 {
		waitStartTime = runtime_nanotime()
	}
	runtime_SemacquireMutex(&m.sema, queueLifo, 1)
    //如果等待时间超过1ms,则进入饥饿模式
	starving = starving || runtime_nanotime()-waitStartTime > starvationThresholdNs
	old = m.state
	if old&mutexStarving != 0 {
		if old&(mutexLocked|mutexWoken) != 0 || old>>mutexWaiterShift == 0 {
			throw("sync: inconsistent mutex state")
		}
		delta := int32(mutexLocked - 1<<mutexWaiterShift)
		if !starving || old>>mutexWaiterShift == 1 {
			delta -= mutexStarving
		}
		atomic.AddInt32(&m.state, delta)
		break
	}
	awoke = true
	iter = 0
} else {
	old = m.state
}
```

## 解锁逻辑

相对加锁逻辑，解锁逻辑相对简单；

1. **快速解锁**

```text
func (m *Mutex) Unlock() {
	new := atomic.AddInt32(&m.state, -mutexLocked)
	if new != 0 {
		m.unlockSlow(new)
	}
}
```

解锁后，锁状态恢复到初始状态（即没有其他获取锁的协程）

**2. 缓慢解锁（slowUnlock）**

**异常处理**

```text
if (new+mutexLocked)&mutexLocked == 0 {
	throw("sync: unlock of unlocked mutex")
}
```

针对未加锁的情况下调用解锁逻辑，直接抛出异常

**饥饿模式**

```text
if new&mutexStarving == 0 {
} else {
	runtime_Semrelease(&m.sema, true, 1)
}
```

饥饿模式下，直接调用runtime_Semrelease唤醒对头协程

**正常模式**

```text
if new&mutexStarving == 0 {
	old := new
	for {
		if old>>mutexWaiterShift == 0 || old&(mutexLocked|mutexWoken|mutexStarving) != 0 {
			return
		}
		new = (old - 1<<mutexWaiterShift) | mutexWoken
		if atomic.CompareAndSwapInt32(&m.state, old, new) {
			runtime_Semrelease(&m.sema, false, 1)
			return
		}
		old = m.state
	}
}
```

正常模式下，如果锁的等待者为0或者锁已经被其他进入锁的协程获取了，则不唤醒等待协程

## 锁特性总结

1. 总体上，golang的互斥锁实现为一公平锁，不会出现某进程被“饿死”的情况；但正常模式下，被唤醒的协程需要与当前进入锁的协程竞争锁，但通常情况下，当前进入锁的协程拥有更好的获取锁的优势
2. 非并发场景下，mutex通过CAS实现快速加锁、解锁
3. 正常模式下，通过自旋实现快速获取锁
4. 饥饿模式下，采用FIFO方式获取锁
5. 加锁解锁不限制于相同的协程中



https://mp.weixin.qq.com/s/bhze_LcFHk7Y-QB4nEQKnA