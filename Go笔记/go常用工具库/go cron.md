> cron(定时任务)，按照约定的时间，定时的执行特定的任务（job），cron 表达式表达了这种约定。
>

**注：最新的v3版本已经不支持秒，需要把cron表达式中的秒去掉**

## cron表达式的基本格式

### 格式一：

cron表达式是一个字符串，字符串分为6个域，中间以空格隔开，每个域代表一个含义

Seconds  Minutes Hours DayofMonth Month DayofWeek

 ![image-20211207111610332](https://gitee.com/the-clock-that-stays/image/raw/master/imgs/image-20211207111610332.png)

注：
1）月(Month)和星期(Day of week)字段的值不区分大小写，如：SUN、Sun和 sun是一样的。
2）星期(Day of week)字段如果没提供，相当于是 *

### 格式二：

@yearly   @annually  每年执行
@monthly 每月执行
@weekly  每周执行
@dail   @midnight  每天执行
@hourly  每小时执行
@every +空格+ *h*m*s(*表示具体的数字，h小时，m分钟，s秒)  例如：@every 2h10m  表示每2小时10分钟执行一次



### 特殊字符说明

1）星号(*)

表示 cron表达式能匹配该字段的所有值。如在第5个字段使用星号(month)，表示每个月

2）斜线(/)

表示增长间隔，如第1个字段(minutes)值是 3-59/15，表示每小时的第3分钟开始执行一次，之后每隔 15 分钟执行一次（即 3、18、33、48这些时间点执行），这里也可以表示为：3/15

例如：spec := "*/5 * * * * *" //每隔5s执行一次

3）逗号(,)

用于枚举值，如第6个字段值是 MON,WED,FRI，表示星期一、三、五执行

例如: spec := "* 52,54 9 * * *" //每天9:52分和9:54分的每秒都执行一次

4）连字号(-)

表示一个范围，如第3个字段的值为 9-17 表示 9am到 5pm直接每个小时（包括9和17）

例如：spec := "15-30 * * * * *" //每分钟的15-30s执行定时任务

5）问号(?)

只用于日(Day of month)和星期(Day of week)，表示不指定值，可以用于代替 *

 

## 主要类型或接口说明

### 1）Cron：

​		包含一系列要执行的实体；支持暂停【stop】；添加【add】实体等

```
type Cron struct {
   entries  []*Entry
   stop     chan struct{}   // 控制 Cron 实例暂停
   add      chan *Entry     // 当 Cron 已经运行了，增加新的 Entity 是通过 add 这个channel 实现的
   snapshot chan []*Entry   // 获取当前所有 entity 的快照
   running  bool            // 当已经运行时为true；否则为false
}
```


注意，Cron结构没有导出任何成员。

注意：有一个成员 stop，类型是struct{}，即空结构体。

### 2）Entry：调度实体

```
type Entry struct {
   // The schedule on which this job should be run.
   // 负责调度当前 Entity 中的 Job 执行
   Schedule Schedule

   // The next time the job will run. This is the zero time if Cron has notbeen
   // started or this entry's schedule is unsatisfiable
   // Job 下一次执行的时间
   Next time.Time

   // The last time this job was run. This is the zero time if the job hasnever
   // been run.
   // 上一次执行时间
   Prev time.Time

   // The Job to run.
   // 要执行的 Job
   Job Job
}
```

### 3）Job：

每一个实体包含一个需要运行的Job

这是一个接口，只有一个方法：run

```
type Job interface {
   Run()
}
```


由于 Entity中需要 Job类型，因此，==我们希望定期运行的任务，就需要实现 Job接口==。同时，由于 Job接口只有一个无参数无返回值的方法，为了使用方便，作者提供了一个类型：

```
type FuncJob func()
```

它通过简单的实现 Run()方法来实现 Job接口：

```
func (f FuncJob) Run() { f() }
```

这样，任何无参数无返回值的函数，**通过强制类型转换为 FuncJob，而FuncJob实现了Job接口的方法，就可以当作 Job来使用了**，AddFunc方法 就是这么做的。

 

### 4）Schedule：

每个实体包含一个调度器（Schedule）

负责调度 Job的执行。它也是一个接口。

```go
type Schedule interface {
   // Return the next activation time, later than the given time.
   // Next is invoked initially, and then each time the job is run.
   // 返回同一 Entity 中的 Job 下一次执行的时间
   Next(time.Time) time.Time
}
```


Schedule 的具体实现通过解析Cron表达式得到。

库中提供了 Schedule的两个具体实现，分别是 SpecSchedule和 ConstantDelaySchedule。

**① SpecSchedule**

```
type SpecSchedule struct {
   Second, Minute, Hour, Dom, Month, Dow uint64
}

```


从开始介绍的 Cron表达式可以容易得知各个字段的意思，同时，对各种表达式的解析也会最终得到一个 SpecSchedule的实例。库中的 Parse返回的其实就是 SpecSchedule的实例（当然也就实现了 Schedule接口）。

**② ConstantDelaySchedule**

```
type ConstantDelaySchedule struct {
   Delay time.Duration // 循环的时间间隔
}
```


这是一个简单的循环调度器，如：每 5分钟。注意，最小单位是秒，不能比秒还小，比如毫秒。

通过 Every函数可以获取该类型的实例，如：

```
constDelaySchedule := Every(5e9)
```

得到的是一个每 5秒执行一次的调度器。

 

## 主要实例化方法

1） 函数

① 实例化 Cron

```go
func New() *Cron {
   return &Cron{
       entries:  nil,
       add:      make(chan *Entry),
       stop:     make(chan struct{}),
       snapshot: make(chan []*Entry),
       running:  false,
    }
}

```


可见实例化时，成员使用的基本是默认值；

② 解析 Cron表达式

```
func Parse(spec string) (_ Schedule, errerror)
```

spec 可以是：

```
Full crontab specs, e.g. “* * * * * ?”

Descriptors, e.g. “@midnight”, “@every1h30m”
```

② 成员方法

```
// 将 job加入 Cron中

// 如上所述，该方法只是简单的通过FuncJob类型强制转换 cmd，然后调用 AddJob方法

func (c *Cron) AddFunc(spec string, cmd func()) error
```

 其中AddFunc的源码如下

```go
func (c *Cron) AddFunc(spec string, cmd func()) error {
   return c.AddJob(spec, FuncJob(cmd))
}
```

==可以看到是将cmd 强转为FuncJob类型的==

==但是注意cmd只能为无参数无返回值的函数类型才能转==

```
// 将 job加入 Cron中

// 通过 Parse函数解析 cron表达式 spec的到调度器实例(Schedule)，之后调用 c.Schedule方法

func (c *Cron) AddJob(spec string, cmd Job)error
```

 

```
// 获取当前 Cron总所有 Entities的快照

func (c *Cron) Entries() []*Entry

 

// 通过两个参数实例化一个Entity，然后加入当前 Cron 中

// 注意：如果当前 Cron未运行，则直接将该 entity加入 Cron中；

// 否则，通过 add这个成员 channel将 entity加入正在运行的 Cron中

func (c *Cron) Schedule(schedule Schedule,cmd Job)


// 新启动一个 goroutine运行当前 Cron

func (c *Cron) Start()

// 通过给 stop成员发送一个 struct{}{}来停止当前 Cron，同时将 running置为 false

// 从这里知道，stop只是通知 Cron停止，因此往 channel发一个值即可，而不关心值是多少

// 所以，成员 stop定义为空 struct

func (c *Cron) Stop()
```

 

5、使用示例

```
package main

import (
   "github.com/robfig/cron"
   "log"
)

func main() {
    i:= 0
    c:= cron.New()
   spec := "*/5 * * * * ?"
   c.AddFunc(spec, func() {
       i++
        log.Println("cron running:", i)
   })
   c.Start()

   select{} //要阻塞住主进程，否则主进程一结束，程序就结束了
}
```

**要阻塞住主进程，否则主进程一结束，程序就结束了**


这是一个简单示例。

输出类似这样：

```
2014/02/22 21:23:40 cron running: 1

2014/02/22 21:23:45 cron running: 2

2014/02/22 21:23:50 cron running: 3

2014/02/22 21:23:55 cron running: 4

2014/02/22 21:24:00 cron running: 5
```

## 定时实现一个有参数的函数

前面讲了AddFunc是将一个**无参数无返回值的函数强转为FuncJob**，而**FuncJob又实现了Job接口**，在接口中调用了最开始的无参数无返回值的函数。

那要如何定时实现一个**有参数的函数**

### 1、创建一个结构体

把需要用的参数，放到结构体中

```
type TestJob struct {
	age int
}
```

###  2、用这个结构体实现Job接口

```
func (this *TestJob)Run() {
   clc(this)
}
```

这样就可以直接使用AddJob来实现，用到**多态性质**

```
 c.AddJob(spec, &TestJob{age: 100})
```



代码如下

```
package main

import (
   "fmt"
   "github.com/robfig/cron"
)

type TestJob struct {
   age int
}

func (this *TestJob)Run() {
   clc(this)
}
func clc(this *TestJob)  {
   this.age--
   fmt.Println("testJob1...",this.age)
}

func main() {
   //i := 0
   c := cron.New()
   spec := "*/3 * * * * ?"

   //AddJob方法
   c.AddJob(spec, &TestJob{age: 100})

   //启动计划任务
   c.Start()

   //关闭着计划任务, 但是不能关闭已经在执行中的任务.
   defer c.Stop()

   select {}
}
```

## Go版 cron源码地址

https://github.com/robfig/cron

源码详解：http://blog.csdn.net/cchd0001/article/details/51076922?locationNum=3&fps=1