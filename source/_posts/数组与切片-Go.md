---
title: 数组与切片 | Go
tags: [go]
date: 2024-07-03 18:49:32
categories: technique
urlname: 40
---

### 数组

数组是具有**相同唯一类型**的一组**长度固定**的数据项序列。类型可以是任意的原始类型例如整型、字符串或者自定义类型。数组元素可以通过 索引（位置）来读取（或者修改），索引从 0 开始。数组长度必须是一个常量表达式（且为非负整数），数组长度也是数组类型的一部分，数组的类型为[len]Type，所以 [5] int 和 [10] int 是属于不同类型的。

数组声明的格式如下，声明数组时所有的元素都会被自动初始化为类型的零值：
```
var identifier [len]type
//var arr [5]int
```

如果数组值已经提前知道了，那么可以通过 数组常量 的方法来初始化数组：

```
var arr1 = [5]int{18, 20, 15, 22, 16}

//arr2时有 10 个元素的数组，除了前三个元素外其他元素都为 0。
var arr2 = [10]int {1, 2, 3} 


var arr3 = [...]int{5, 6, 7, 8, 22}

//只有索引 3 和 4 被赋予实际的值，其他元素都被设置为空的字符串
var arr4 = [5]string{3: "Chris", 4: "Ron"}
```

### 切片

切片（slice）是对数组一个连续片段的引用（数组实际上是切片的构建块），其长度是可变的。slice由三部分组成：长度、容量、指针。指针指向底层数组，长度为当前容纳的数据长度，容量为能容纳数据的最大长度。slice的类型为[]Type。

> A slice is a data structure describing a contiguous section of an array stored separately from the slice variable itself. A slice is not an array. A slice describes a piece of an array

```
// runtime/slice.go
type slice struct {
	array unsafe.Pointer // 元素指针
	len   int // 长度 
	cap   int // 容量
}
```

切片的声明格式如下，不说明长度。一个切片在未初始化之前默认为 nil，长度为 0。
```
var identifier []type
//var a []int
//fmt.Println(a)
//fmt.Println(a==nil)
//[]
//true
```
切片也可以用类似数组的方式初始化：
```
var x = []int{2, 3, 5, 7, 11}
```

创建slice的方式有以下几种：

- 对数组/切片进行截取（左闭右开）
```
var data [10]int
slice := data[2:8]  //仅指定索引和长度
slice = slice[1:3]
slice1 := data[1:2:3] //指定长度为2-1 = 1，容量为3-1 = 2
```

查看如下代码，一开始slice1是数组arr1从2到4索引对应的元素，此时数组长度为6。对于slice1来说，容量为6-2 = 4。因此`slice1 = slice1[0:4]`可以正常运行，因此仍然在容量内：
```
package main
import "fmt"

func main() {
    var arr1 [6]int
    var slice1 []int = arr1[2:5] // item at index 5 not included!

    // load the array with integers: 0,1,2,3,4,5
    for i := 0; i < len(arr1); i++ {
        arr1[i] = i
    }

    // print the slice
    for i := 0; i < len(slice1); i++ {
        fmt.Printf("Slice at %d is %d\n", i, slice1[i])
    }

    fmt.Printf("The length of arr1 is %d\n", len(arr1))
    fmt.Printf("The length of slice1 is %d\n", len(slice1))
    fmt.Printf("The capacity of slice1 is %d\n", cap(slice1))

    // grow the slice
    slice1 = slice1[0:4]
    for i := 0; i < len(slice1); i++ {
        fmt.Printf("Slice at %d is %d\n", i, slice1[i])
    }
    fmt.Printf("The length of slice1 is %d\n", len(slice1))
    fmt.Printf("The capacity of slice1 is %d\n", cap(slice1))

    // grow the slice beyond capacity
    //slice1 = slice1[0:7] // panic: runtime error: slice bound out of range
}
```

输出为：
```
Slice at 0 is 2  
Slice at 1 is 3  
Slice at 2 is 4  
The length of arr1 is 6  
The length of slice1 is 3  
The capacity of slice1 is 4  
Slice at 0 is 2  
Slice at 1 is 3  
Slice at 2 is 4  
Slice at 3 is 5  
The length of slice1 is 4  
The capacity of slice1 is 4  
```

- 通过make创建，可以指定len、cap
```
slice := make([]int,5,10)
```
使用make创建切片，make 的使用方式是：`func make([]T, len, cap)`，其中 cap 是可选参数。

`var slice1 []type = make([]type, len)`，也可以简写为`slice1 := make([]type, len)`。这里 len 是底层数组的长度并且也是 slice 的初始长度，容量大小也为len。【也要注意使用make时设置的长度和容量】

makeslice分配的内存大小为类型et的size * cap，创建时会判断是否超过允许的分配的最大内存。
```
// runtime/malloc.go
func makeslice(et *_type, len, cap int) unsafe.Pointer {
    mem, overflow := math.MulUintptr(et.size, uintptr(cap))
    if overflow || mem > maxAlloc || len < 0 || len > cap {
        // NOTE: Produce a 'len out of range' error instead of a
        // 'cap out of range' error when someone does make([]T, bignumber).
        // 'cap out of range' is true too, but since the cap is only being
        // supplied implicitly, saying len is clearer.
        // See golang.org/issue/4085.
        mem, overflow := math.MulUintptr(et.size, uintptr(len))
        if overflow || mem > maxAlloc || len < 0 {
            panicmakeslicelen()
        }
        panicmakeslicecap()
    }

    return mallocgc(mem, et, true)
}
```

newarray分配的内存大小为size * len，与makeslice比，主要少了cap相关的检查：
```
// newarray allocates an array of n elements of type typ.
func newarray(typ *_type, n int) unsafe.Pointer {
    if n == 1 {
        return mallocgc(typ.size, typ, true)
    }
    mem, overflow := math.MulUintptr(typ.size, uintptr(n))
    if overflow || mem > maxAlloc || n < 0 {
        panic(plainError("runtime: allocation size out of range"))
    }
    return mallocgc(mem, typ, true)
}
```

再看如下代码，sliceHeader是slice在运行时的表示，sliceHeader在构造时，先通过unsafe_NewArray创建Data。而unsafe_NewArray就是调用的newarray，因此MakeSlice就是创建一个持有cap大小的数组的sliceHeader。
```
// reflect/value.go
// SliceHeader is the runtime representation of a slice.
// It cannot be used safely or portably and its representation may
// change in a later release.
// Moreover, the Data field is not sufficient to guarantee the data
// it references will not be garbage collected, so programs must keep
// a separate, correctly typed pointer to the underlying data.
type SliceHeader struct {
	Data uintptr
	Len  int
	Cap  int
}


// sliceHeader is a safe version of SliceHeader used within this package.
type sliceHeader struct {
    Data unsafe.Pointer
    Len  int
    Cap  int
}
// MakeSlice creates a new zero-initialized slice value
// for the specified slice type, length, and capacity.
func MakeSlice(typ Type, len, cap int) Value {
	if typ.Kind() != Slice {
		panic("reflect.MakeSlice of non-slice type")
	}
	if len < 0 {
		panic("reflect.MakeSlice: negative len")
	}
	if cap < 0 {
		panic("reflect.MakeSlice: negative cap")
	}
	if len > cap {
		panic("reflect.MakeSlice: len > cap")
	}

	s := unsafeheader.Slice{Data: unsafe_NewArray(typ.Elem().(*rtype), cap), Len: len, Cap: cap}
	return Value{typ.(*rtype), unsafe.Pointer(&s), flagIndir | flag(Slice)}
}
```



- 对slice进行append生成

```
slice := make([]int,5,10)
slice = append(slice,6)
```

append func说明如下，append会追加一个或多个数据至slice中，这些数据会存储在slice的持有的数组中，最后返回一个新的slice，因此必须保存append的结果：

```
// builtin/builtin.go
// The append built-in function appends elements to the end of a slice. If
// it has sufficient capacity, the destination is resliced to accommodate the
// new elements. If it does not, a new underlying array will be allocated.
// Append returns the updated slice. It is therefore necessary to store the
// result of append, often in the variable holding the slice itself:
//
//	slice = append(slice, elem1, elem2)
//	slice = append(slice, anotherSlice...)
//
// As a special case, it is legal to append a string to a byte slice, like this:
//
//	slice = append([]byte("hello "), "world"...)
func append(slice []Type, elems ...Type) []Type
```


再看Append源码，首先判断当前slice长度i0与追加数据的总长度i1是否溢出，溢出则报错；接着，若i1小于/等于slice的cap，直接返回原slice的起始及结束数据部分，否则，当前底层数组已无法存储所有的追加数据（如果索引 len-1 所指向的元素已经是底层数组的最后一个元素），会迁移到新的内存位置进行扩容处理；使用扩容后的容量构建新的slice，将原slice拷贝至新slice中。

```
// refelct/value.go
// grow grows the slice s so that it can hold extra more values, allocating
// more capacity if needed. It also returns the old and new slice lengths.
func grow(s Value, extra int) (Value, int, int) {
	i0 := s.Len()
	i1 := i0 + extra
	if i1 < i0 {
		panic("reflect.Append: slice overflow")
	}
	m := s.Cap()
	if i1 <= m {
		return s.Slice(0, i1), i0, i1
	}
	if m == 0 {
		m = extra
	} else {
		const threshold = 256
		for m < i1 {
			if i0 < threshold {
				m += m
			} else {
				m += (m + 3*threshold) / 4
			}
		}
	}
	t := MakeSlice(s.Type(), i1, m)
	Copy(t, s)
	return t, i0, i1
}

// Append appends the values x to a slice s and returns the resulting slice.
// As in Go, each x's value must be assignable to the slice's element type.
func Append(s Value, x ...Value) Value {
	s.mustBe(Slice)
	s, i0, i1 := grow(s, len(x))
	for i, j := i0, 0; i < i1; i, j = i+1, j+1 {
		s.Index(i).Set(x[j])
	}
	return s
}
```

注意，`s.Index(i).Set(x[j])`，此时追加元素，向索引为当前切片长度值的位置设置值，因此，要注意make和append一起的使用。

例如下面的代码，info长度和容量都为10，前10个元素都为0，追加元素时，从尾部添加。因此，需要使用`info := make([]int,0,10)`或者`info := make([]int,0)`，设置长度为0。
```
info := make([]int,10)
info = append(info,1)
fmt.Print(info)
//output
//[0 0 0 0 0 0 0 0 0 0 1]
```

> go 1.18版本之前：当原 slice 容量小于 1024 的时候，新 slice 容量变成原来的 2 倍；原 slice 容量超过 1024，新 slice 容量变成原来的1.25倍。go 1.18版本之后：当原slice容量(oldcap)小于256的时候，新slice(newcap)容量为原来的2倍；原slice容量超过256，新slice容量newcap = oldcap+(oldcap+3*256)/4。但实际上，newcap 作了一个内存对齐`capmem = roundupsize(uintptr(newcap) * ptrSize)`（内存地址是所存储数据大小（按字节为单位）的整数倍，以便CPU可以一次将该数据从内存中读取出来），这和内存分配策略相关。进行内存对齐之后，新 slice 的容量是要 大于等于 按照前半部分生成的newcap。

```
// runtime/slice.go
func growslice(et *_type, old slice, cap int) slice {
    // ……
    newcap := old.cap
	doublecap := newcap + newcap
	if cap > doublecap {
		newcap = cap
	} else {
		const threshold = 256
		if old.cap < threshold {
			newcap = doublecap
		} else {
			for 0 < newcap && newcap < cap {
                // Transition from growing 2x for small slices
				// to growing 1.25x for large slices. This formula
				// gives a smooth-ish transition between the two.
				newcap += (newcap + 3*threshold) / 4
			}
			if newcap <= 0 {
				newcap = cap
			}
		}
	}
	// ……
    
	capmem = roundupsize(uintptr(newcap) * ptrSize)
	newcap = int(capmem / ptrSize)
}
```


使用代码验证go1.18版本之后的扩容机制

```
package main

import "fmt"

func main() {
	s := make([]int, 0)

	oldCap := cap(s)

	for i := 0; i < 2048; i++ {
		s = append(s, i)

		newCap := cap(s)

		if newCap != oldCap {   //容量不足时
			fmt.Printf("[%d -> %4d] cap = %-4d  |  after append %-4d  cap = %-4d\n", 0, i-1, oldCap, i, newCap)
			oldCap = newCap
		}
	}
}
```

输出：
```
[0 ->   -1] cap = 0     |  after append 0     cap = 1
[0 ->    0] cap = 1     |  after append 1     cap = 2
[0 ->    1] cap = 2     |  after append 2     cap = 4
[0 ->    3] cap = 4     |  after append 4     cap = 8
[0 ->    7] cap = 8     |  after append 8     cap = 16
[0 ->   15] cap = 16    |  after append 16    cap = 32
[0 ->   31] cap = 32    |  after append 32    cap = 64
[0 ->   63] cap = 64    |  after append 64    cap = 128
[0 ->  127] cap = 128   |  after append 128   cap = 256
[0 ->  255] cap = 256   |  after append 256   cap = 512
[0 ->  511] cap = 512   |  after append 512   cap = 848
[0 ->  847] cap = 848   |  after append 848   cap = 1280
[0 -> 1279] cap = 1280  |  after append 1280  cap = 1792
[0 -> 1791] cap = 1792  |  after append 1792  cap = 2560
```


再看下面这段代码。
```
func main() {
	s := []int{1,2}
	s = append(s,4,5,6)
	fmt.Printf("len=%d, cap=%d",len(s),cap(s))
}
```

再看这段代码，这段代码和上面代码的输出一致吗？如果不一致，分别是什么？

```
func main() {
	s := []int{1, 2}
	s = append(s, 4)
	s = append(s, 5)
	s = append(s, 6)
	fmt.Printf("len=%d, cap=%d", len(s), cap(s))
}
```

第一段代码输出为：

```
len=5, cap=6
```

第二段代码输出为：

```
len=5, cap=8
```

为什么两段代码输出不一致？

两段代码中的s原来都只有2个元素，len和cap都为2。

- 第一段代码一次append三个元素后，len变为5，cap最小要变成5，调用`growslice`函数时，传入`cap=5`，旧slice`old = []int{1, 2}`，因此doublecap为4，满足第一个if条件，`newcap`更新为5，接着调用`roundupsize(uintptr(newcap) * ptrSize)`，ptrSize是指一个指针的大小，在64位机上是8，查看roundupsize的源码，传入size = 8*5 = 40 < smallSizeMax-8=1016，因此计算`class_to_size[size_to_class8[(size+smallSizeDiv-1)/smallSizeDiv]]`，class_to_size通过 spanClass 获取 span划分的 object大小。而 size_to_class8 表示通过 size 获取它的 spanClass。`(size+smallSizeDiv-1)/smallSizeDiv = 5`，`size_to_class8[5] = 5`,`class_to_size[5] = 48`，因此`newcap = int(capmem / ptrSize) = 48/8 = 6`，新的slice容量为6。

- 第二段代码每次append一个元素，满足第二个if条件`cap <= doublecap`，且旧slice容量小于256，因此直接分为doublecap，再进行内存对齐，最后为8。

```
// runtime/msize.go
func roundupsize(size uintptr) uintptr {
	if size < _MaxSmallSize {
		if size <= smallSizeMax-8 {
			return uintptr(class_to_size[size_to_class8[(size+smallSizeDiv-1)/smallSizeDiv]])
		} else {
			//……
		}
	}
    //……
}

const _MaxSmallSize = 32768
const smallSizeMax = 1024
const smallSizeDiv = 8

```

```
var size_to_class8 = [smallSizeMax/smallSizeDiv + 1]uint8{0, 1, 2, 3, 4, 5, 5, 6, 6, 7, 7, 8, 8, 9, 9, 10, 10, 11, 11, 12, 12, 13, 13, 14, 14, 15, 15, 16, 16, 17, 17, 18, 18, 19, 19, 19, 19, 20, 20, 20, 20, 21, 21, 21, 21, 22, 22, 22, 22, 23, 23, 23, 23, 24, 24, 24, 24, 25, 25, 25, 25, 26, 26, 26, 26, 27, 27, 27, 27, 27, 27, 27, 27, 28, 28, 28, 28, 28, 28, 28, 28, 29, 29, 29, 29, 29, 29, 29, 29, 30, 30, 30, 30, 30, 30, 30, 30, 31, 31, 31, 31, 31, 31, 31, 31, 31, 31, 31, 31, 31, 31, 31, 31, 32, 32, 32, 32, 32, 32, 32, 32, 32, 32, 32, 32, 32, 32, 32, 32}

var class_to_size = [_NumSizeClasses]uint16{0, 8, 16, 24, 32, 48, 64, 80, 96, 112, 128, 144, 160, 176, 192, 208, 224, 240, 256, 288, 320, 352, 384, 416, 448, 480, 512, 576, 640, 704, 768, 896, 1024, 1152, 1280, 1408, 1536, 1792, 2048, 2304, 2688, 3072, 3200, 3456, 4096, 4864, 5376, 6144, 6528, 6784, 6912, 8192, 9472, 9728, 10240, 10880, 12288, 13568, 14336, 16384, 18432, 19072, 20480, 21760, 24576, 27264, 28672, 32768}
```




下面的代码输出什么？[数组与切片有什么异同](https://golang.design/go-questions/slice/vs-array/)
```
package main

import "fmt"

func main() {
	slice := []int{0, 1, 2, 3, 4, 5, 6, 7, 8, 9}
	s1 := slice[2:5]
	s2 := s1[2:6:7]

	s2 = append(s2, 100)
	s2 = append(s2, 200)

	s1[2] = 20

	fmt.Println(s1)
	fmt.Println(s2)
	fmt.Println(slice)
}
```

输出：
```
[2 3 20]
[4 5 6 7 100 200]
[0 1 2 3 20 5 6 7 100 9]
```

Go 语言的函数参数传递，只有值传递，没有引用传递。如果传递给函数的是一个指针，指针的值（一个地址）会被复制，但指针的值所指向的地址上的值不会被复制，对指针类型而言，变量的值是指针，即传递的值也是指针，传递指针也是值传递。切片作为函数参数时，不管传的是 slice 还是 slice 指针，slice 结构体自身不会被改变，也就是说底层数据地址不会被改变。 但是通过指向底层数据的指针，可以改变切片的底层数据。通过 slice 的 array 字段就可以拿到数组的地址。在代码里，是直接通过类似 s[i]=10 这种操作改变 slice 底层数组元素值。

```
func main() {
	s := []int{1, 1, 1}
	f(s)
	fmt.Println(s)
}

func f(s []int) {
	// i只是一个副本，不能改变s中元素的值
	/*for _, i := range s {
		i++
	}
	*/

	for i := range s {
		s[i] += 1
	}
}
```

输出为：
```
[2 2 2]
```

要改变外层 slice，只有将返回的新的 slice 赋值到原始 slice，或者向函数传递一个指向 slice 的指针
```
func myAppend(s []int) []int {
	// s为外部切片的拷贝，使用append返回新的切片，不影响外层函数的s
	s = append(s, 100)
	return s
}

func myAppendPtr(s *[]int) {
	// 会改变外层 s 本身
	*s = append(*s, 100)
	return
}

func main() {
	s := []int{1, 1, 1}
	newS := myAppend(s)

	fmt.Println(s)
	fmt.Println(newS)

	s = newS

	myAppendPtr(&s)
	fmt.Println(s)
}
```

输出：

```
[1 1 1]
[1 1 1 100]
[1 1 1 100 100]
```



参考资料：

[深入了解Go Slice（一）—— make的详细处理过程](https://blog.csdn.net/xz_studying/article/details/106311831)
[深入了解Go slice（二）—— append的处理过程](https://blog.csdn.net/xz_studying/article/details/106483759)
[切片](https://learnku.com/docs/the-way-to-go/72-slice/3613)
[数组与切片](https://golang.design/go-questions/slice/vs-array/)