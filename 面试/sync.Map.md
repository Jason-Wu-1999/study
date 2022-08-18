# 为什么需要sync.Map

原生的map的并发的情况下会存在线程不安全的情况

## 原生的map并发写会存在什么问题？

```go
func main() {
	m := map[int]int {1:1}
	go do(m)
	go do(m)

	time.Sleep(1*time.Second)
	fmt.Println(m)
}

func do (m map[int]int) {
	i := 0
	for i < 10000 {
		m[1]=1
		i++
	}
}

```

**输出报错**

```
fatal error: concurrent map writes
```

==不支持并发写==



解决方法，直接加锁，但是会影响效率

故希望通过  **空间换时间，减小加锁影响的范围**

# sync.Map的使用

```
func main() {
    // 关键人物出场
	m := sync.Map{}
	m.Store(1,1)
	go do(m)
	go do(m)

	time.Sleep(1*time.Second)
	fmt.Println(m.Load(1))
}

func do (m sync.Map) {
	i := 0
	for i < 10000 {
		m.Store(1,1)
		i++
	}
}


输出
1 true
```

# 源码实现

## 基本结构

```
type Map struct {
	mu Mutex
	read atomic.Value // readOnly
	dirty map[interface{}]*entry
	misses int
}

```

- **mu**	Mutex									加锁作用。保护后文的dirty字段
- **read**	atomic.Value				  存读的数据。因为是atomic.Value类型，只读，所以并发是安全的。实际存的是readOnly的数据结构。
- **dirty**	map[interface{}]*entry	包含最新写入的数据。当misses计数达到一定值，将其赋值给read。
- **misses**	int						    计数作用。每次从read中读失败，则计数+1。

**readOnly的数据结构：**

```
type readOnly struct {
    m  map[interface{}]*entry			单纯的map结构
    amended bool 						Map.dirty的数据和这里的 m 中的数据不一样的时候，为true
}
```

**entry的结构**

```
type entry struct {
    //可见value是个指针类型，虽然read和dirty存在冗余情况（amended=false），但是由于是指针类型，存储的空间应该不是问题
    p unsafe.Pointer // *interface{}
}

```

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/Jason-Wu-1999/blog.img/imgs/20190720230959289.jpeg)

**写：直写。
读：先读read，没有再读dirty。**



### 查询

注意**双重检查**：如果read中没有，加锁之后还会找一次

```go
func (m *Map) Load(key interface{}) (value interface{}, ok bool) {
    // 因read只读，线程安全，优先读取
    read, _ := m.read.Load().(readOnly)
    e, ok := read.m[key]
    
    // 如果read没有，并且dirty有新数据，那么去dirty中查找
    if !ok && read.amended {
        m.mu.Lock()
        // 双重检查（原因是前文的if判断和加锁非原子的，害怕这中间发生故事）
        read, _ = m.read.Load().(readOnly)
        e, ok = read.m[key]
        
        // 如果read中还是不存在，并且dirty中有新数据
        if !ok && read.amended {
            e, ok = m.dirty[key]
            // m计数+1
            m.missLocked()
        }
        
        m.mu.Unlock()
    }
    
    if !ok {
        return nil, false
    }
    return e.load()
}

func (m *Map) missLocked() {
    m.misses++
    if m.misses < len(m.dirty) {
        return
    }
    
    // 将dirty置给read，因为穿透概率太大了(原子操作，耗时很小)
    m.read.Store(readOnly{m: m.dirty})
    m.dirty = nil
    m.misses = 0
}

```

<img src="https://cdn.jsdelivr.net/gh/Jason-Wu-1999/blog.img/imgs/20190720235756624.png" alt="在这里插入图片描述" style="zoom: 50%;" />

### 读过程

1. 先读read
   1. 有，直接返回
   2. 没有  再判断 read中的amened属性，判断read和dirty是否有不一样的数据
      1. 没有不一样的数据直接返回无
      2. 有不一样的数据
         1. 加锁，==双重检测==，（因为前文的判断和锁不是原子操作，防止期间发生了变化）
            1. 有直接返回
            2. 无就会 去dirty中读  
               1. **最后会对miss+1 ，**并判断miss是否达到阈值，是就把dirty中的值同步过来

> 利用了read和amended属性，解决了每次读取都涉及锁的问题，实现了读这一个使用场景的高性能



这边有几个点需要强调一下：

1. 如何设置阀值？


这里采用miss计数和dirty长度的比较，来进行阀值的设定。

2. 为什么dirty可以直接换到read？

因为写操作只会操作dirty，所以保证了dirty是最新的，并且数据集是肯定包含read的。
（可能有同学疑问，dirty不是下一步就置为nil了，为何还包含？后文会有解释。）

3. 为什么dirty置为nil？

我不确定这个原因。猜测：一方面是当read完全等于dirty的时候，读的话read没有就是没有了，即使穿透也是一样的结果，所以存的没啥用。另一方是当存的时候，如果元素比较多，影响插入效率。

### 删除

```
func (m *Map) Delete(key interface{}) {
    // 读出read，断言为readOnly类型
    read, _ := m.read.Load().(readOnly)
    e, ok := read.m[key]
    // 如果read中没有，并且dirty中有新元素，那么就去dirty中去找。这里用到了amended，当read与dirty不同时为true，说明dirty中有read没有的数据。
    
    if !ok && read.amended {
        m.mu.Lock()
        // 再检查一次，因为前文的判断和锁不是原子操作，防止期间发生了变化。
        read, _ = m.read.Load().(readOnly)
        e, ok = read.m[key]
        
        if !ok && read.amended {
            // 直接删除
            delete(m.dirty, key)
        }
        m.mu.Unlock()
    }
    
    if ok {
    // 如果read中存在该key，则将该value 赋值nil（采用标记的方式删除！）
        e.delete()
    }
}

func (e *entry) delete() (hadValue bool) {
    for {
    	// 再次再一把数据的指针
        p := atomic.LoadPointer(&e.p)
        if p == nil || p == expunged {
            return false
        }
        
        // 原子操作
        if atomic.CompareAndSwapPointer(&e.p, p, nil) {
            return true
        }
    }
}
```

<img src="https://cdn.jsdelivr.net/gh/Jason-Wu-1999/blog.img/imgs/20190720235535762.png" alt="在这里插入图片描述" style="zoom: 25%;" />

#### 删除过程

1. 先读read 如果read中存在，使用标志删除，**将entry指针置为nil**，并没有去dirty中真正的删除（**非常高效，不需要进入dirty，不要加锁）**
2. read没有， 并且**利用amended**判断和dirty是否一致
   1. 不一致 操作dirty 加锁  ，再重复进行**双重检查**，若 read 仍然不存在该元素。则调用 delete 方法从 dirty 中标记该元素的删除。



> 注意到如果 key 同时存在于 read 和 dirty 中时，删除只是做了一个标记，将 p 置为 nil；而如果仅在 dirty 中含有这个 key 时，会直接删除这个 key。原因在于，若两者都存在这个 key，仅做标记删除，可以在下次查找这个 key 时，命中 read，提升效率。若只有在 dirty 中存在时，read 起不到“缓存”的作用，直接删除。



### 增(改)

```
func (m *Map) Store(key, value interface{}) {
    // 如果m.read存在这个key，并且没有被标记删除，则尝试更新。
    read, _ := m.read.Load().(readOnly)
    if e, ok := read.m[key]; ok && e.tryStore(&value) {
        return
    }
    
    // 如果read不存在或者已经被标记删除
    m.mu.Lock()
    read, _ = m.read.Load().(readOnly)
   
    if e, ok := read.m[key]; ok { // read 存在该key
    // 如果entry被标记expunge，则表明dirty没有key，可添加入dirty，并更新entry。
        if e.unexpungeLocked() { 
            // 加入dirty中，这儿是指针
            m.dirty[key] = e
        }
        // 更新value值
        e.storeLocked(&value) 
        
    } else if e, ok := m.dirty[key]; ok { // dirty 存在该key，更新
        e.storeLocked(&value)
        
    } else { // read 和 dirty都没有
        // 如果read与dirty相同，则触发一次dirty刷新（因为当read重置的时候，dirty已置为nil了）
        if !read.amended { 
            // 将read中未删除的数据加入到dirty中
            m.dirtyLocked() 
            // amended标记为read与dirty不相同，因为后面即将加入新数据。
            m.read.Store(readOnly{m: read.m, amended: true})
        }
        m.dirty[key] = newEntry(value) 
    }
    m.mu.Unlock()
}

// 将read中未删除的数据加入到dirty中
func (m *Map) dirtyLocked() {
    if m.dirty != nil {
        return
    }
    
    read, _ := m.read.Load().(readOnly)
    m.dirty = make(map[interface{}]*entry, len(read.m))
    
    // 遍历read。
    for k, e := range read.m {
        // 通过此次操作，dirty中的元素都是未被删除的，可见标记为expunged的元素不在dirty中！！！
        if !e.tryExpungeLocked() {
            m.dirty[k] = e
        }
    }
}

// 判断entry是否被标记删除，并且将标记为nil的entry更新标记为expunge
func (e *entry) tryExpungeLocked() (isExpunged bool) {
    p := atomic.LoadPointer(&e.p)
    
    for p == nil {
        // 将已经删除标记为nil的数据标记为expunged
        if atomic.CompareAndSwapPointer(&e.p, nil, expunged) {
            return true
        }
        p = atomic.LoadPointer(&e.p)
    }
    return p == expunged
}

// 对entry尝试更新 （原子cas操作）
func (e *entry) tryStore(i *interface{}) bool {
    p := atomic.LoadPointer(&e.p)
    if p == expunged {
        return false
    }
    for {
        if atomic.CompareAndSwapPointer(&e.p, p, unsafe.Pointer(i)) {
            return true
        }
        p = atomic.LoadPointer(&e.p)
        if p == expunged {
            return false
        }
    }
}

// read里 将标记为expunge的更新为nil
func (e *entry) unexpungeLocked() (wasExpunged bool) {
    return atomic.CompareAndSwapPointer(&e.p, expunged, nil)
}

// 更新entry
func (e *entry) storeLocked(i *interface{}) {
    atomic.StorePointer(&e.p, unsafe.Pointer(i))
}

```



![在这里插入图片描述](https://cdn.jsdelivr.net/gh/Jason-Wu-1999/blog.img/imgs/20190721010415938.png)

### 增或改 过程

1. 查看read中是否存在
   1. 存在，
      1. 且没有标记为已删除（expunged）状态，直接改dirty中的值
      2.  但是被标记为已删除，==说明dirty中肯定不存在该元素==，将元素状态从已删除（expunged）更改为 nil。**将元素插入 dirty 中。**
   2. 不存在
      1. dirty中存在，则直接写入更新entry的指向
      2. dirty也不存在，==触发dirty更新==，则从 read 中复制未被标记删除的数据，并向 dirty 中插入该元素，赋予元素值 entry 的指向。更新完成之后，dirty中的元素都是未被删除的，可见标记为expunged的元素不在dirty中！！！

这边有几个点需要强调一下：

> **read中的标记为已删除和nil的区别？**
>
> **标记为nil**，说明是正常的delete操作，此时dirty中不一定存在
> a. dirty赋值给read后，此时dirty不存在
> b. dirty初始化后，肯定存在
>
> **标记为expunged，**说明是在dirty初始化的时候操作的，此时dirty中肯定不存在。
>
> 可能存在性能问题？
> 初始化dirty的时候，虽然都是指针赋值，但read如果较大的话，可能会有些影响。

# 总结

优点：是官方出的，是亲儿子；**通过读写分离，降低锁时间来提高效率；**
缺点：**不适用于大量写的场景**，这样会导致read map读不到数据而进一步加锁读取，同时dirty map也会一直晋升为read map，整体性能较差。
**适用场景：大量读，少量写**

参考博客：https://blog.csdn.net/u011957758/article/details/96633984