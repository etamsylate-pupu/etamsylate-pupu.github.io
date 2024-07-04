---
title: copy函数 | Go
tags: [go]
date: 2024-07-03 22:16:42
categories: technique
urlname: 41
---


copy函数格式如下，仅接受切片类型的参数。其中 srcSlice 为数据来源切片，destSlice 为复制的目标（也就是将 srcSlice 复制到 destSlice），**来源和目标的类型必须一致**，copy() 函数的返回值表示实际发生复制的元素个数。

```
copy( destSlice, srcSlice []T) int
```

copy使用,copy()会先计算 dst 的长度 l=len(dst)，再计算**复制长度 n=min(len(src), l)**，因此，这里将 s1 切片中的 3 个元素复制到 s2 中,n 返回复制的元素个数 3。s4 的长度只有 2,所以只复制了 s3 的前 2 个元素,n 返回 2。slice1的长度为0，因此复制长度为0，输出[]。
```
s1 := []int{1, 2, 3}
s2 := make([]int, 10) 
n := copy(s2, s1)
fmt.Println(s1, s2, n) // [1 2 3] [1 2 3 0 0 0 0 0 0 0] 3
s3 := []int{1, 2, 3}
s4 := make([]int, 2)
n = copy(s4, s3) 
fmt.Println(s3, s4, n) // [1 2 3] [1 2] 2
slice1 := make([]int,0,3)
slice2 := []int{6, 7, 8, 9}
copy(slice1, slice2)
fmt.Println(slice1)  //[]
```

那么数组能拷贝到切片吗？ 由于copy来源和目标的类型必须一致，因此需要使用`[:]`操作将数组转为切片。
```
array4 := [4]int{4, -4, 4, -4}
s6 := []int{1, -1, 1, -1, -5, 5}

copy(s6, array4[0:])
fmt.Println("s6:", s6) //s6: [4 -4 4 -4 -5 5]
```


查看copy的源码，为深拷贝，将src切片指向底层数组中的值拷贝到dst切片指向的数组中，针对dst切片进行的操作不会对src产生任何的影响。

```
// runtime/slice.go
// slicecopy is used to copy from a string or slice of pointerless elements into a slice.
func slicecopy(toPtr unsafe.Pointer, toLen int, fromPtr unsafe.Pointer, fromLen int, width uintptr) int {
	if fromLen == 0 || toLen == 0 {
		return 0
	}

	n := fromLen
	if toLen < n {
		n = toLen
	}

	if width == 0 {
		return n
	}

	size := uintptr(n) * width
	if raceenabled {
		callerpc := getcallerpc()
		pc := abi.FuncPCABIInternal(slicecopy)
		racereadrangepc(fromPtr, size, callerpc, pc)
		racewriterangepc(toPtr, size, callerpc, pc)
	}
	if msanenabled {
		msanread(fromPtr, size)
		msanwrite(toPtr, size)
	}
	if asanenabled {
		asanread(fromPtr, size)
		asanwrite(toPtr, size)
	}

	if size == 1 { // common case worth about 2x to do here
		// TODO: is this still worth it with new memmove impl?
		*(*byte)(toPtr) = *(*byte)(fromPtr) // known to be a byte pointer
	} else {
		memmove(toPtr, fromPtr, size)
	}
	return n
}

```





查看如下代码：
```
package main

import "fmt"

func main() {
    src := []int{1, 2, 3, 4, 5}
    dst := make([]int, 5)
    numberOfElementsCopied := copy(dst, src)

    fmt.Printf("Number Of Elements Copied: %d\n", numberOfElementsCopied)
    fmt.Printf("dst: %v\n", dst)
    fmt.Printf("src: %v\n", src)

    //After changing dst
    dst[0] = 10
    fmt.Println("\nAfter changing dst")
    fmt.Printf("dst: %v\n", dst)
    fmt.Printf("src: %v\n", src)

    //Length of destination is less than length of source
    dst = make([]int, 4)
    numberOfElementsCopied = copy(dst, src)
    fmt.Println("\nLength of dst less than src")
    fmt.Printf("Number Of Elements Copied: %d\n", numberOfElementsCopied)
    fmt.Printf("dst: %v\n", dst)
    fmt.Printf("src: %v\n", src)

    //Length of destination is greater than length of source
    dst = make([]int, 6)
    numberOfElementsCopied = copy(dst, src)
    fmt.Println("\nLength of dst less than src")
    fmt.Printf("Number Of Elements Copied: %d\n", numberOfElementsCopied)
    fmt.Printf("dst: %v\n", dst)
    fmt.Printf("src: %v\n", src)
}
```

输出为：
```
Number Of Elements Copied: 5
dst: [1 2 3 4 5]
src: [1 2 3 4 5]

After changing dst
dst: [10 2 3 4 5]
src: [1 2 3 4 5]

Length of dst less than src
Number Of Elements Copied: 4
dst: [1 2 3 4]
src: [1 2 3 4 5]

Length of dst less than src
Number Of Elements Copied: 5
dst: [1 2 3 4 5 0]
src: [1 2 3 4 5]

```





参考资料：

[Copy function in Go (Golang)](https://golangbyexample.com/copy-function-in-golang/)
[golang Slice的创建、添加、删除等操作和源码分析](https://liangtian.me/post/go-slice/)