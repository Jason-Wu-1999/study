# new 和make的区别

- make和new都是golang用来==分配内存==的內建函数，且在堆上分配内存，make 即分配内存，==也初始化内存==。new只是将内存清零，并没有初始化内存。
- ==make返回的还是引用类型本身；而new返回的是指向类型的指针。==
- **make只能用来分配及初始化类型为slice，map，channel的数据；new可以分配任意类型的数据**。



make不仅可以开辟一个内存，还能给这个内存的类型==初始化其零值==。



对切片建议使用make，因为new之后返回的是指针，不能append

```go
func main() {
    list := new([]int)
    list = append(list, 1)
    fmt.Println(list)
}
```

==不能通过编译，==new([]int) 之后的 list 是一个 **`*[]int`** 类型的指针，不能对指针执行 append 操作。可以使用 make() 初始化之后再用。同样的，map 和 channel 建议使用 make() 或字面量的方式初始化，不要用 new() 。

