# 1、json序列化（Marshal）

Marshal函数**只有在转换成功**的时候才会返回数据，在转换的过程中我们需要注意几点：

1. JSON对象只支持string作为key，所以要编码一个map，那么必须是map[string]T这种类型(T是Go语言中任意的类型)
2. **Channel,complex和function是不能被编码成JSON的**
3. 嵌套的数据是不能编码的，不然会让JSON编码进入死循环
4. 指针在编码的时候会输出指针指向的内容，而空指针会输出null

- 生成json

```
package main

import (
    "encoding/json"
    "fmt"
    "github.com/bitly/go-simplejson"
)

type Man struct {
    Names string `json:"names"`
    Age   int    `json:"age"`
}

func main() {


    //生成json
    var data Man
    data.Age = 1
    data.Names = "lisa"
    result,_ := json.Marshal(data)
    fmt.Println(string(result))

}
```

# 2 、json反序列化（Unmarshal）

## 2.1、解析已知的固定格式的json

```
package main

import (
    "encoding/json"
    "fmt"
    "github.com/bitly/go-simplejson"
)

type Man struct {
    Names string `json:"names"`
    Age   int    `json:"age"`
}
func main() {
    //解析已知格式的json
    jsonString := `{"names":"json","age":1}`

    jsc := &Man{}

    err := json.Unmarshal([]byte(jsonString), jsc)
    if err != nil {
        fmt.Println(err.Error())
    }
    fmt.Println(*jsc)
//ouput  {json 1}
}
```

## 2.2、解析格式不固定的json

```
package main

import (
    "encoding/json"
    "fmt"
    "github.com/bitly/go-simplejson"
)

type Man struct {
    Names string `json:"names"`
    Age   int    `json:"age"`
}

func main() {


    //解析未知格式

    //使用simple解析json 可以解析已知格式，也可以解析未知格式
    simpleJs, err := simplejson.NewJson([]byte(jsonString))
    if err != nil {
        fmt.Println("simplejson 解析json 解析失败")
    }
    name, _ := simpleJs.Get("names").String()
    age, _ := simpleJs.Get("age").Int()
    fmt.Println(name, age)
}
```

# 3、结构体中tag使用注意点

用于json相关结构体中tag定义的时候需要注意

1. 字段的tag是"-"，那么这个字段不会输出到JSON
2. tag中带有自定义名称，那么这个自定义名称会出现在JSON的字段名中
3. tag中如果带有"omitempty"选项，那么如果该字段值为空，就不会输出到JSON串中
4. 如果字段类型是bool, string, int, int64等，而tag中带有",string"选项，那么这个字段在输出到JSON的时候会把该字段对应的值转换成JSON字符串



## **结构体和map之间可以通过json序列化转化**

struct--- json序列化------>[]byte 、string  ---- json反序列化---->map

```
package main

import (
   "encoding/json"
   "fmt"
)

type people struct {
   Name  string
   Class string
   Mes   string
}


func main() {
   a := people{
      Name:"0.290",
      Class: "111",
      Mes: "xxxnnn",
   }
   ajson,err:=json.Marshal(a)
   if err != nil {
      fmt.Printf("json marshal err:%s\n",err)
      return
   }
   b:=string(ajson)
   fmt.Printf("b :%v \n",b)

   c:=make(map[string]string)

   err = json.Unmarshal([]byte(b), &c)
   if err != nil {
      fmt.Printf(" json unmarshal %s error: %s", b, err.Error())
      return
   }
   fmt.Println(c)

}
```