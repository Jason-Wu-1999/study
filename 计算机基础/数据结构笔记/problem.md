## **1、如何求一个数有多少位数**

```java
int  maxLength =  (max+"").length;//先让他加空字符串，变成string型，在对他取length。
```

## 2、while（）的注意事项

while（判断语句）其中的判断语句若有多个则需要注意顺序问题

如：

```java
 while ( arr[index]>temp &&  index>=0  ){
		index--;
  }
```

​	若按如上顺序写则会报指针越界异常，在判断 index>=0之前已经将越界的index作为下标带入数组。

![image-20210417115913358](https://cdn.jsdelivr.net/gh/Jason-Wu-1999/blog.img/img/image-20210417115913358.png)

应该将index>=0判断写前面

```
while (index>=0 && arr[index]>temp ){
		index--;
  }
```

## 3、统计个数

```java
	// 遍历 bytes , 统计 每一个byte出现的次数->map[key,value]
 String content = "i like like like java do you like a java";
    byte[] contentBytes = content.getBytes();//把字符串拆分成单个，放到数组中

	Map<Byte, Integer> counts = new HashMap<>();
	for (byte b : bytes) {
		Integer count = counts.get(b);//get() 方法获取指定 key 对应对 value。  而key是字符，value是个数
		if (count == null) { // Map还没有这个字符数据,第一次
			counts.put(b, 1);
		} else {
			counts.put(b, count + 1);
		}
	}
```

## 4、HashSet

- hashSet 不会有重复的元素，若添加重复元素，不会成功（能运行，只是不添加）
-  **retainAll函数** ：求出tempSet 和 allAreas 集合的交集, 交集会赋给 tempSet*

```
tempSet.retainAll(allAreas);  // 求出tempSet 和 allAreas 集合的交集, 交集会赋给 tempSet*
```

- **contains函数**：判断hashmap和hashset中是否含有某个值返回boolean型

```
sites.contains("Taobao")//判断hashset   sites中是否有字符串 "Taobao"，有返回true，无false
```

### 遍历

#### 1. Iterator迭代方式遍历

```
Set<String> set = new HashSet<String>();  
set.add("aaa");  
set.add("bbb");  
set.add("ccc");    
Iterator iterator = set.iterator();  
while (iterator.hasNext()) {  
 System.out.println(iterator.next());              
}
```

#### 2. for循环方式遍历

```
Set<String> set = new HashSet<String>();  
set.add("aaa");  
set.add("bbb");  
set.add("ccc");  
for (String s:set) {  
 System.out.println(s);  
} 
```

