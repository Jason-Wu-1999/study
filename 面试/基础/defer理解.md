> ## 规则一 当defer被声明时，其参数就会被实时解析

就是在defer声明的时候，变量的值就是，调用defer时用的值，之后变量值的改变对defer不影响

如例子中声明第一个defer时x=1，y=2 所以在调用这个defer时就将x=1，y=2代入即可

> ## 规则二 defer执行顺序为先进后出

所以例子中先执行”BB“，再执行”AA“



**由于两个defer函数都含有函数型的参数，所以作为参数的函数在其位置展开执行，不需要遵守defer的规则。**

所以先执行cal("A",x,y)

cal("B",x,y)求出两个值

再遵循defer先进后出，将上面求出的值代到下式

cal("BB",x,  ）

cal("AA",x, )

```go
package main

import "fmt"

func cal(index string, a, b int) int {
   ret:=a+b
   fmt.Println(index,a ,b,ret)
   return ret
}

func main() {
   x:=1
   y:=2
   defer cal("AA",x,cal("A",x,y))
    //上面一句话等效于
    //m:=cal("A",x,y)
    //defer cal("AA",x,m)
   x=10
   defer cal("BB",x,cal("B",x,y))
   y=20

}
输出结果
A 1 2 3
B 10 2 12
BB 10 12 22
AA 1 3 4
```





```
func main() {
    for i := 0; i < 5; i++ {
     
        defer func() {
            println(i)
        }()
    }
}

结果 
5
5
5
5
5
```

需要传入参数

```
func main() {
    for i := 0; i < 5; i++ {
        defer func(i int) {
            println(i)
        }(i)
    }
}
结果
4
3
2
1
0
```

# defer和return、panic的执行顺序

defer在panic之前执行

return在defer之前， 但是在defer之后返回值



> return ---  defer  ---  panic
>



## defer和panic的执行顺序

从**代码上来看，协程遇到panic时，遍历本协程的defer链表，并执行defer。在执行defer过程中，遇到recover则停止panic，返回recover处继续往下执行。如果没有遇到recover，遍历完本协程的defer链表后，向stderr抛出panic信息。**



### Go 中 defer 和 return 执行的先后顺序

1. 多个defer的执行顺序为“后进先出”；
2. defer、return、返回值三者的执行逻辑应该是：return最先执行，return负责将结果写入返回值中；接着defer开始执行一些收尾工作；最后函数携带当前返回值退出。

 

如果函数的返回值是无名的（不带命名返回值），则go语言会在执行return的时候会执行一个类似创建一个临时变量作为保存return值的动作，而有名返回值的函数，由于返回值在函数定义的时候已经将该变量进行定义，在执行return的时候会先执行返回值保存操作，而后续的defer函数会改变这个返回值(虽然defer是在return之后执行的，但是由于使用的函数定义的变量，所以执行defer操作后对该变量的修改会影响到return的值



eg1：不带命名返回值的函数

```
package main
 
import "fmt"
 
func main() {
    fmt.Println("return:", test())// defer 和 return之间的顺序是先返回值, i=0，后defer
}
 
func test() int {//这里返回值没有命名
    var i int
    defer func() {
        i++
        fmt.Println("defer1", i) //作为闭包引用的话，则会在defer函数执行时根据整个上下文确定当前的值。i=2
    }()
    defer func() {
        i++
        fmt.Println("defer2", i) //作为闭包引用的话，则会在defer函数执行时根据整个上下文确定当前的值。i=1
    }()
    return i
}　
```

test() 先返回 i=0

defer2先于defer1执行

输出结果为:

defer2 1

defer1 2

return: 0

 

 

eg2：带命名返回值的函数:

```
package main
 
import "fmt"
 
func main() {
    fmt.Println("return:", test())
}
 
func test() (i int) { //返回值命名i
    defer func() {
        i++
        fmt.Println("defer1", i)
    }()
    defer func() {
        i++
        fmt.Println("defer2", i)
    }()
    return i
}
```

输出结果为:

defer2 1

defer1 2

return: 2

# 理解return 返回值的运行机制:

为了弄清上述两种情况的区别，我们首先要理解return 返回值的运行机制:

> ==return 并非原子操作，分为赋值，和返回值两步操作==

**eg1 :** 实际上return 执行了两步操作，因为返回值没有命名，所以return 默认指定了一个返回值（假设为s），首先将i赋值给s,后续的操作因为是针对i,进行的，所以不会影响s, 此后因为s不会更新，所以return s 不会改变相当于：
var i int
s := i
return s

**eg2 :** 同上，s 就相当于 命名的变量i, 因为所有的操作都是基于
命名变量i(s),返回值也是i, 所以每一次defer操作，都会更新
返回值i
