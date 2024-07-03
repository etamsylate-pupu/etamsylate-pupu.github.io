---
title: for range 介绍 | Go
tags: [go]
date: 2024-07-03 09:30:49
categories: technique
urlname: 38
---

go常用的遍历方式，有for和for range。

针对go中的for循环，如下

```
for i := 0; i < 10; i++{
    //dosomething
    fmt.Printf("%d ", i)
}
//output
//0 1 2 3 4 5 6 7 8 9
```

go中的for range如下，for-range 一般可以返回两个值，对于不同类型的data有不同的返回值，对于数组、切片和字符串，返回下标和值；对于map，返回key和value；对于chan，返回值。

数组
```
//for range over an array
arr := [3]int{1,2,3}
for index,value := range arr{
    //dosomething
    fmt.Printf("index: %d, value: %d \n", index, value)
}
//output
//index: 0, value: 1
//index: 1, value: 2
//index: 2, value: 3
```

切片
```
//for range over a slice.
nums := []int{1,2,3,4}
for index,value := range nums{
    //dosomething
    fmt.Printf("index: %d, value: %d \n", index, value)
}
//output
//index: 0, value: 1
//index: 1, value: 2
//index: 2, value: 3
//index: 3, value: 4
```


字符串
```
//for range over a string
str := "hello world"
for index,value := range str{
    //dosomething
    fmt.Printf("index: %d, value: %c \n", index, value)
}
//output
//index: 0, value: h
//index: 1, value: e
//index: 2, value: l
//index: 3, value: l
//index: 4, value: o
//index: 5, value:
//index: 6, value: w
//index: 7, value: o
//index: 8, value: r
//index: 9, value: l
//index: 10, value: d
```

映射
```
//for range over a map
userMap := map[string]int{
    "Alice": 18,
    "Bob": 20,
    "Lily": 16,
}
for key,value := range userMap{
    //dosomething
    fmt.Printf("key: %s, value: %d \n", key, value)
}
//output
//key: Lily, value: 16
//key: Alice, value: 18
//key: Bob, value: 20
```

通道
```
//for range over a channel
ch := make(chan int, 3)
go func() {
    defer close(ch)
    for i := 0; i < 3; i++ {
        ch <- i
    }	
}()

for value := range ch {
    //dosomething
    fmt.Printf("value: %d\n", value)
}
//value: 0
//value: 1
//value: 2
```

也可以忽略返回值，见如下写法：
```
slice := []int{1, 2, 3}
// 1
for range slice {
    fmt.Println()
}
// 2
for k := range slice {
    fmt.Println(k)
}
// 3
for k, v := range slice {
    fmt.Println(k, v)
}

```

- 1：因其range前没有接收变量，因此代表此次循环并不在意返回的索引以及数据，只关心循环次数。
- 2：因其range前仅有一个接收变量，因此代表此次循环仅关心返回的索引，不关心返回的数据，此代码等同于for k,_ := range slice
- 3：该方法是range的完全体使用形式，因此代表此次循环及关心返回的索引，也关心返回的数据。

for range其实是一种语法糖，内部调用还是 for 循环，初始化会拷贝需要遍历的对象，每次对象的值/地址赋值给同一个元素。（查看[go编译源码](https://github.com/golang/gofrontend/blob/e387439bfd24d5e142874b8e68e7039f74c744d7/go/statements.cc#L5484)，伪代码如下。以array为例，`range_temp := range `在这里对原对象进行了拷贝，`value_temp = range_temp[index_temp]`每次是对同一个变量`value_temp`赋值。）

> 语法糖（Syntactic Sugar）是指编程语言中的一种语法结构，这种结构并不会改变语言的功能或能力，而只是为了让代码的书写更加简洁、更易于理解。通常来说使用语法糖能够增加程序的可读性，从而减少程序代码出错的机会。举个例子：在 C 语言里用 a[i] 表示 *(a+i)，用 a[i][j] 表示 *(*(a+i)+j)

数组
```
//for range over an array
//The loop we generate:
len_temp := len(range)
range_temp := range
for index_temp = 0; index_temp < len_temp; index_temp++ {
    value_temp = range_temp[index_temp]
    index = index_temp
    value = value_temp
    original body
}
```

切片
```
//for range over a slice
//The loop we generate:
for_temp := range
len_temp := len(for_temp)
for index_temp = 0; index_temp < len_temp; index_temp++ {
    value_temp = for_temp[index_temp]
    index = index_temp
    value = value_temp
    original body
}
```

字符串：不同的字符集，占用的长度不同，所以会对是否是utf8字符集字符进行判断，并进行不同的处理
```
//for range over a string
//The loop we generate:
len_temp := len(range)
var next_index_temp int
for index_temp = 0; index_temp < len_temp; index_temp = next_index_temp {
    value_temp = rune(range[index_temp])
    if value_temp < utf8.RuneSelf {
        next_index_temp = index_temp + 1
    } else {
        value_temp, next_index_temp = decoderune(range, index_temp)
    }
    index = index_temp
    value = value_temp
    // original body
}
```

映射：map的遍历是无序的，同时不会指定次数
```
//for range over a map
//The loop we generate:
var hiter map_iteration_struct
for mapiterinit(type, range, &hiter); hiter.key != nil; mapiternext(&hiter) {
    index_temp = *hiter.key
    value_temp = *hiter.val
    index = index_temp
    value = value_temp
    original body
}
```

通道：使用操作符<-读取数据的，会对第二个值进行判断，所以可以保证for-range返回的值都是成功读取的值，当没有数据读取的时候，会因为操作符<-阻塞。

```
//for range over a channel
//The loop we generate:
for {
    index_temp, ok_temp = <-range
    if !ok_temp {
        break
    }
    index = index_temp
    original body
}
```

考虑如下代码，遍历一个切片并将每次遍历的索引追加至切片，遍历会停止吗？  
```
v := []int{1, 2, 3}
for i := range v {
    v = append(v, i)
}
```

【会停止】，遍历3次结束，结束后切片v为`[1 2 3 0 1 2]`。如上所述，遍历前对v做了拷贝，所以期间对原来v的修改不会反映到遍历中。

考虑如下代码，遍历一个数组并将变量的地址追加到切片，输出解引用后的切片元素，输出是什么？ 
```
arr := [2]int{1, 2}
res := []*int{}
for _, v := range arr {
    res = append(res, &v)
}
fmt.Println(*res[0],*res[1])
//expect: 1 2
//but output: 2 2
fmt.Println(res[0],res[1])
//0xc000014158 0xc000014158
```
【输出为2 2】，可以发现，切片的每一个元素解引用后的值都是数组最后一个元素的值，也就是说，存储的地址是同一个。如上所述，go在for range遍历数组时，每次是对同一个变量`value_temp`赋值。因为`v`变量的地址在遍历过程中是不变的，切片元素存储同一个变量的地址，最后一次遍历，`v`变量值为2，因此最后的输出为`2 2`。

那么如何让输出变为`1 2`。

- 使用局部变量拷贝v

```
arr := [2]int{1, 2}
res := []*int{}
for _, v := range arr {
    m := v
    res = append(res, &m)
}
fmt.Println(*res[0],*res[1])
fmt.Println(res[0],res[1])
//output
//1 2
//0xc000014158 0xc000014160
```

- 直接通过索引获取原数组元素

```
arr := [2]int{1, 2}
res := []*int{}
for k := range arr {
    res = append(res, &arr[k])
}
fmt.Println(*res[0],*res[1])
fmt.Println(res[0],res[1])
//output
//1 2
//0xc0000a6130 0xc0000a6138
```


遍历时，拷贝是深拷贝还是浅拷贝呢？首先，回忆一下什么是深拷贝和浅拷贝

> 浅拷贝：是对对象的表面层次的复制。它创建一个新的对象，并复制原始对象的所有非引用类型字段的值。然而，对于引用类型的字段（如切片、映射、通道、接口和指向结构体或数组的指针），浅拷贝仅仅复制了引用的地址，而非引用的实际内容。这意味着新对象和原始对象共享相同的引用类型字段的数据。

> 深拷贝：是对对象的完全复制，包括对象引用的其他对象。它递归地遍历原始对象的所有字段，并创建新的内存空间来存储这些字段的值，包括引用类型字段所指向的实际数据。这样，深拷贝后的对象与原始对象在内存中是完全独立的，对其中一个对象的修改不会影响另一个对象。

值类型和引用类型：

> 值类型：变量直接存储值，内存通常在栈上分配，栈在函数调用完会被释放。比如：int、float、bool、string、array、sturct 等。

> 引用类型：变量存储的是一个地址（变量的值存储的是对最终值的引用），这个地址存储最终的值。内存通常在堆上分配，通过GC回收。

也就是说，深拷贝和浅拷贝的主要区别在于它们处理**引用类型**字段的方式，浅拷贝仅仅复制了引用的地址，因此新对象和原始对象共享相同的数据。相反，深拷贝则创建了新的内存空间来存储引用类型字段的数据，确保新对象与原始对象完全独立。

以slice（切片）为例，slice实际上是一个结构体，包含长度、容量、指向底层数组的指针。底层数组是可以被多个 slice 同时指向的，因此对一个 slice 的元素进行操作是有可能影响到其他 slice 的。
```
// runtime/slice.go
type slice struct {
	array unsafe.Pointer // 元素指针
	len   int // 长度 
	cap   int // 容量
}
```

如下代码所示，slice1拷贝至slice2，改变slice2第一个元素的值时，slice1也跟着变。因为Go的赋值为浅拷贝，仅拷贝了指针并没有拷贝指针指向的数组。Go函数的参数传递也是同样的，为值传递[When are function parameters passed by value?](https://go.dev/doc/faq#pass_by_value)。

```
slice1 := []int{1, 2}
slice2 := slice1

slice2[0] = 2

fmt.Println(slice1)
fmt.Println(slice2)

//output
//[2 2]
//[2 2]
```

对大数组这样遍历有问题吗？
```
//假设值都为1，这里只赋值3个
var arr = [102400]int{1, 1, 1}
for i, n := range arr {
    //just ignore i and n for simplify the example
    _ = i
    _ = n
}
```
【有问题】，遍历前的拷贝对内存是极大浪费。

如何优化：
- 对数组取地址遍历for i, n := range &arr
- 对数组做切片引用for i, n := range arr[:]


类似地：对大量元素的 slice 和 map 遍历会有内存浪费问题吗？ 不会，浅拷贝仅拷贝指针。遍历slice时，实际上遍历的是一个指向底层数组的指针，而不会创建原始数组的拷贝。遍历map时，也是类似的情况，遍历的是指向底层哈希表的指针。


【输出2 2 2】

对 map 遍历时删除元素能遍历到么？
```
	var m = map[int]int{1: 1, 2: 2, 3: 3}
	//only del key once, and not del the current iteration key
	var o sync.Once
	for i := range m {
		o.Do(func() {
			for _, key := range []int{1, 2, 3} {
				if key != i {
					fmt.Printf("when iteration key %d, del key %d\n", i, key)
					delete(m, key)
					break
				}
			}
		})
		fmt.Printf("key: %d value: %d ", i, m[i])
	}
```

输出（其中一种情况）
```
when iteration key 1, del key 2
key: 1 value: 1 key: 3 value: 3
```

不会。map 内部实现是一个链式 hash 表，为保证每次无序，初始化时会随机一个遍历开始的位置, once.Do函数内保证第一次执行时删除未遍历的一个元素，因此删除的元素之后无法遍历到。


对 map 遍历时新增元素能遍历到么？

```
var m = map[int]int{1:1, 2:2, 3:3}
for i, _ := range m {
    m[4] = 4
    fmt.Printf("key: %d value: %d ", i, m[i])
}
```
【可能会，输出中可能会有4 4】，注意是可能会。map迭代顺序是不确定的，有时候可能会包含新添加的键值对。

其中一部分输出：
```
key: 1 value: 1 key: 2 value: 2 key: 3 value: 3
key: 2 value: 2 key: 3 value: 3 key: 4 value: 4 key: 1 value: 1
key: 1 value: 1 key: 2 value: 2 key: 3 value: 3 key: 4 value: 4
key: 2 value: 2 key: 3 value: 3 key: 4 value: 4 key: 1 value: 1
key: 1 value: 1 key: 2 value: 2 key: 3 value: 3 key: 4 value: 4
key: 1 value: 1 key: 2 value: 2 key: 3 value: 3 key: 4 value: 4
```



参考资料:

[深挖 Go 之 for-range 排坑指南](https://mp.weixin.qq.com/s/rfbZ79TmZ61lx_JBnwDJMQ#)
