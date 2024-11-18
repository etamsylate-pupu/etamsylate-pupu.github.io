---
title: 经典内部排序算法 | Go实现
tags: [sort]
date: 2024-07-11 11:19:52
categories: technique
urlname: 42
mathjax: true
---

内部排序的所有排序操作都在内存中完成，不需要额外的磁盘或其他存储设备的辅助。这适用于数据量小到足以完全加载到内存中的情况。常见的内部排序算法有：插入排序、希尔排序、选择排序、冒泡排序、归并排序、快速排序、堆排序等。外部排序通常涉及到数据的分区处理，部分数据被暂时存储在外部磁盘等存储设备上。常见的外部排序算法有计数排序、基数排序、桶排序。

### 冒泡排序

- 描述：对于要排序的数组，从第一位开始从前往后比较相邻两个数字，若前者大，则交换两数字位置，然后比较位向右移动一位。记录前一轮交换的最终位置，该位置之后的元素为已排序状态，下一轮的交换只需执行到该处，每一轮的比较将使得当前未排序数字中的最大者被排序，未排序数字总数减 1。第 $arr.length - 1$ 轮结束后排序完成。

- 稳定性：稳定
- 优化：当某一轮比较均未发生交换，说明排序已完成，可设置一个布尔值记录一轮排序是否有发生交换，若无则提前退出循环结束程序。
- 时间复杂度分析：有序数组为O($n$)，无序数组为O($n^2$)
- 空间复杂度：O(1)
- 代码：
```
func bubbleSort(arr []int) {
	n := len(arr)
	for i := 0; i < n-1; i++ {
		swapped := false
		for j := 0; j < n-i-1; j++ {
			if arr[j] > arr[j+1] {
				arr[j], arr[j+1] = arr[j+1], arr[j]
				swapped = true
			}
		}
		// 如果一轮遍历没有发生交换，说明数组已经有序，可以提前退出
		if !swapped {
			break
		}
	}
}
```

### 选择排序

- 描述：对于要排序的数组，设置一个 $minIdx$ 记录最小数字下标。先假设第 1 个数字最小，此时 minIdx = 0 ，将 $arr[minIdx]$ 与后续数字逐一比较，当遇到更小的数字时，使 $minIdx$ 等于该数字下标，第1轮比较将找出此时数组中最小的数字。找到后将 $minIdx$ 下标的数字与第 1 个数字交换，此时称一个数字已被排序。然后开始第2轮比较，令 minIdx = 1，重复上述过程。每一轮的比较将使得当前未排序数字中的最小者被排序，未排序数字总数减 1。第 $arr.length - 1$ 轮结束后排序完成。

- 稳定性：不稳定。存在跨越交换。找到本轮次最小值之后，将其与本轮起始数字交换，此时若中间有与起始元素同值的元素，将打破稳定性。例:　7 7 2 。第一轮交换第一个 7 和 2，则两个 7 位置关系改变。

- 时间复杂度分析：O($n^2$)（选择排序的交换次数是 $O(n)$，而冒泡排序的平均交换次数是O($n^2$)）
- 空间复杂度：O(1)

- 代码：

```
//单元选择

func selectSort(nums []int){
	minIndex := 0
	for i := 0; i < len(nums) - 1; i++{
		//查找最小值
		for j := i+1; j < len(nums); j++{
			if nums[minIndex] > nums[j]{
				minIndex = j
			}
		}
		//更新最小值
		if nums[i] != nums[minIndex]{
			nums[i], nums[minIndex] = nums[minIndex], nums[i]
		}
		
        minIndex = i + 1
	}
}
```

```
//双元选择 在遍历寻找最小值下标时，可以同时寻找最大值下标

func biSelectSort(nums []int) {
	minIndex := 0
	for i := 0; i < len(nums)- i- 1; i++ {
		maxIndex := len(nums) - i - 1
		if nums[maxIndex] < nums[minIndex] {
			nums[minIndex], nums[maxIndex] = nums[maxIndex], nums[minIndex]
		}
		//查找最大值，最小值
		for j := i + 1; j < len(nums)-i-1; j++ {
			if nums[minIndex] > nums[j] {
				minIndex = j
			}
			if nums[maxIndex] < nums[j] {
				maxIndex = j
			}
		}
		//更新最大值，最小值
		if nums[i] != nums[minIndex] {
			nums[i], nums[minIndex] = nums[minIndex], nums[i]
		}
		if nums[maxIndex] != nums[len(nums)-i-1] {
			nums[maxIndex], nums[len(nums)-i-1] = nums[len(nums)-i-1], nums[maxIndex]
		}
		minIndex = i + 1
	}
}
```

### 插入排序

- 描述：对于待排序数组，从第 2 个元素开始 (称作插入对象元素) ，比较它与之前的元素 (称作比较对象元素) ，当插入对象元素小于比较对象元素时，继续往前比较，直到不小于 (≥) 比较对象，此时将插入对象元素插入到该次比较对象元素之后。重复这个插入过程直到最后一个元素作为插入对象元素完成插入操作。

- 稳定性：简单插入和二分插入是稳定的

- 时间复杂度：O($n^2$)

- 空间复杂度：O(1)

- 代码：
```
简单插入
func insertSort(nums []int) {
	for i := 1; i < len(nums); i++ {
		insertIndex := i
		for j := i - 1; j >= 0; j-- {
			//如果元素更小
			if nums[insertIndex] < nums[j] {
				nums[insertIndex], nums[j] = nums[j], nums[insertIndex]
				insertIndex = j
			}else{
                break // 如果元素不小于前一个元素，不再需要插入
            }
		}
	}
}
```

```
// 二分插入
//每一轮向前插入都使得该元素在完成插入后，从第一个元素到该元素是排序状态（指这部分的相对排序状态，在它们中间后续可能还会插入其他数字），利用这一点，对一个新的插入对象向前执行折半插入，能够显著减少比较的次数。

func binInsertSort(nums[]int){
	for i := 1 ; i < len(nums); i++{
		insertIndex := i
		left := 0
		right := i - 1
		for left <= right{
			mid := left + (right - left)/2 
			if nums[insertIndex] < nums[mid]{
                right = mid - 1
            }else if nums[insertIndex] > nums[mid]{
                left = mid + 1
            }else{
				left = mid
			}
		}
		temp := nums[insertIndex]
		for j := insertIndex ; j > left; j--{
			 nums[j] = nums[j -1] 
		}
		nums[left] = temp
		
	}
}

```

### 希尔排序

- 描述：是简单插入排序经过改进之后的一个更高效的版本，也称为递减增量排序算法。先将整个待排序的记录序列分割成为若干子序列分别进行直接插入排序，待整个序列中的记录 “基本有序” 时，再对全体记录进行依次直接插入排序。对原待排序列中相隔 $gap$ 的数字执行简单插入排序，然后缩小 $gap$，对新的 $gap$ 间隔的数字再次执行简单插入排序。希尔提出的增量序列生成式为 $n / 2^k$，k = 1, 2, 3, ... ，例如 n = 11，则增量序列为 $\{1,2,5\}$ 。在讨论希尔排序时，可将其称为 Shell增量，另有更优的 Hibbard增量、Knuth增量、Sedgewick增量 等。

- 稳定性：不稳定

- 时间复杂度：最差O($n^2$) 最好O($nlogn$)

- 空间复杂度：O(1)

- 代码：
```
//shell增量
func shellSort(arr []int)[]int{
    n := len(arr)
    gap := n / 2
    for gap > 0{
        for i := gap; i < n; i++{
            current := arr[i]
            preIndex := i - gap
            //插入排序
            for(preIndex >= 0 && arr[preIndex] > current){
                arr[preIndex + gap] = arr[preIndex]
                preIndex -= gap
            }
            arr[preIndex + gap] = current
        }
        gap /= 2
    }
    return arr
}

```

### 归并排序

- 描述：归并排序是 分治思想 的应用，即将原待排数组 递归或迭代 地分为左右两半，直到数组长度为 1，然后合并 (merge) 左右数组，在合并中完成排序。

- 稳定性：稳定

- 时间复杂度：O($nlogn$)

- 空间复杂度：O(n)
- 代码：可以通过 自顶向下 (top-down) 或 自底向上 (bottom-up) 的方式实现归并排序。

```
//自顶向下 (top-down)：从输入数组出发，不断二分该数组，直到数组长度为1，再执行合并。适合用 递归 实现。
//归
func mergeSort(arr []int) []int {
	if len(arr) < 2 {
		return arr
	}
	mid := len(arr) / 2
	left := mergeSort(arr[:mid])
	right := mergeSort(arr[mid:])
	result := merge(left, right)
	return result
}

//并
func merge(left, right []int) []int {
	temp := make([]int, 0)
	i, j := 0, 0
	for i < len(left) && j < len(right) {
		if left[i] <= right[j] {
			temp = append(temp, left[i])
			i++
		} else {
			temp = append(temp, right[j])
			j++
		}
	}
	if i < len(left) {
		temp = append(temp, left[i:]...)
	}
	if j < len(right) {
		temp = append(temp, right[j:]...)
	}
	return temp

}
```
参考资料：

[十大排序从入门到入赘](https://iyukiyama.github.io/sorting/)