1. 将切片 b 的元素追加到切片 a 之后：a = append(a, b...)

2. 复制切片 a 的元素到新的切片 b 上： b = make([]T, len(a)) copy(b, a)

3. -删除位于索引 i 的元素：a = append(a[:i], a[i+1:]...)
4. 切除切片 a 中从索引 i 至 j 位置的元素：a = append(a[:i], a[j:]...)

5. 为切片 a 扩展 j 个元素长度：a = append(a, make([]T, j)...)

6. 在索引 i 的位置插入元素 x：a = append(a[:i], append([]T{x}, a[i:]...)...)

7. 在索引 i 的位置插入长度为 j 的新切片：a = append(a[:i], append(make([]T, j), a[i:]...)...)

8. 在索引 i 的位置插入切片 b 的所有元素：a = append(a[:i], append(b, a[i:]...)...)

9. 取出位于切片 a 最末尾的元素 x：x, a = a[len(a)-1], a[:len(a)-1]

10. 将元素 x 追加到切片 a：a = append(a, x)
    



> 注意事项

当在二维切片后面append一维切片时，要注意，因为他们共用一个底层切片，后续改变一维切片，二维切片也会变

```
func main() {
// 测试append

   path:=[]int{
      1,2,3,
   }

   var res [][]int
   res=append(res,path)
   fmt.Printf("未修改%p\n",path)
   fmt.Println("未修改",res)
   fmt.Println("未修改",path)

   path[0]=10
   fmt.Printf("修改%p\n",path)

   fmt.Println("修改",res)
   fmt.Println("修改",path)
}

输出结果：
未修改0xc000012168
未修改 [[1 2 3]]
未修改 [1 2 3]
修改0xc000012168
修改 [[10 2 3]]
修改 [10 2 3]
```



