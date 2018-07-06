* 1. slice 有点像动态数组， 数组的容量capacity，在超过容量以后，会扩容；length，表示了包含数据的长度；append操作会在length位置增加新的元素；
* 2. 例子：
```go

func main() {
	nums := make([]int, 10)
	for i := 1; i <= 10; i++ {
		nums[i-1] = i
	}

	fmt.Printf("step 1: nums = %v, len = %d, cap = %d\n", nums, len(nums), cap(nums))

	nums2 := nums[:5]

	fmt.Printf("steps 2: nums2 = %v, len = %d, cap = %d\n", nums2, len(nums2), cap(nums2))

	nums3 := nums[3:8]

	fmt.Printf("steps 3: nums3 = %v, len = %d, cap = %d\n", nums3, len(nums3), cap(nums3))
	nums3[0] = 20

	for i := 0; i < 5; i++ {
		nums3 = append(nums3, i+13)
	}

	fmt.Printf("steps 4, extends nums3\n")
	fmt.Printf("			nums = %v, len = %d, cap = %d\n", nums, len(nums), cap(nums))
	fmt.Printf("			nums2 = %v, len = %d, cap = %d\n", nums2, len(nums2), cap(nums2))
	fmt.Printf("			nums3 = %v, len = %d, cap = %d\n", nums3, len(nums3), cap(nums3))
}

```
输出
```
step 1: nums = [1 2 3 4 5 6 7 8 9 10], len = 10, cap = 10
steps 2: nums2 = [1 2 3 4 5], len = 5, cap = 10
steps 3: nums3 = [4 5 6 7 8], len = 5, cap = 7
steps 4, extends nums3
                        nums = [1 2 3 20 5 6 7 8 13 14], len = 10, cap = 10
                        nums2 = [1 2 3 20 5], len = 5, cap = 10
                        nums3 = [20 5 6 7 8 13 14 15 16 17], len = 10, cap = 14
```