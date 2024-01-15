```

type Queue struct {
    queue []int
}

func NewMyQueue() *Queue {
    return &Queue{
        queue:make([]int,0),
    }
}
func (m *Queue)IsEmpty() bool {
    return len(m.queue) == 0
}
func (m *Queue)Front() int {
  return m.queue[0]
}
func (m *Queue)Back() int {
  return m.queue[len(m.queue)-1]
}
func (m *Queue) Push(val int){
    for !m.IsEmpty() && val > m.Back(){
        m.queue = m.queue[:len(m.queue)-1]
    }
    m.queue = append(m.queue,val)
}

func (m *Queue) Pop(val int) {
     if !m.IsEmpty() && val == m.Front(){
         m.queue = m.queue[1:]
     } 
}

func maxSlidingWindow(nums []int, k int) []int {
    queue := NewMyQueue()
    length := len(nums)
     res := make([]int,0)
    // 入
    for i := 0 ; i < k ; i++{
      queue.Push(nums[i])
    }
    //记录前k个的最大值
    res = append(res,queue.Front())
    
     // 滑动串口移动元素
     for i:= k ; i < length;i++{
         queue.Pop(nums[i-k])
         queue.Push(nums[i])
         res = append(res,queue.Front())
     }

     return res

}

```


## Heap - Priority Queue

- Input: nums = [1,3,-1,-3,5,3,6,7], k = 3
- Output: [3,3,5,5,6,7]

```
处理想法:
   1. 维护一个大顶堆 
   2. 用i,j 分别代表 窗口的起始位置 和 结束位置 
   3. 判断当前大顶堆的最大值的索引要跟起始位置,结束位置的索引做比较 
   4. 合适的话给Result的当前index赋值最大值的值 
   5. 窗口移动 
```

- Achieve Code 
```
 type KV struct{
    index,value int // get heap’s index and value 
}
type MaxHeap []KV

func (h MaxHeap) Len() int           { return len(h) }
func (h MaxHeap) Less(i, j int) bool { return h[i].value > h[j].value }
func (h MaxHeap) Swap(i, j int)      { h[i], h[j] = h[j], h[i] }

func (h *MaxHeap) Push(x interface{}) {
	*h = append(*h, x.(KV))
}

func (h *MaxHeap) Pop() interface{} {
	old := *h
	n := len(old)
	item := old[n-1]
	*h = old[0 : n-1]
	return item
}
```


```
func maxSlidingWindow(nums []int, k int) []int {
	if k <= 0 || len(nums) == 0 {
		return nil
	}

	maxHeap := &MaxHeap{}
	heap.Init(maxHeap)

    res := make([]int,len(nums)-k+1)

    # i 窗口的起始位置, j 窗口的结束位置 , index 结果切片的索引 
    i,j,index := 0,0,0
    
    for j < len(nums){
        // 组成堆 
        kv := KV{j,nums[j]}
        heap.Push(maxHeap,kv)
        // 假如窗口的尺寸小于k
        if j-i+1 < k{
            j++
        }// 窗口的尺寸等于k的时候 
        else if j-i+1 == k{
            // 当前的堆顶索引小于 i 移除
            for (*maxHeap)[0].index < i {
                heap.Pop(maxHeap)
            }
            // 正常赋值
            first := (*maxHeap)[0]
            res[index] = first.value
            
            // 如果上个最大值的索引没在当前窗口里 将移除
            if first.index < i+1 {
                heap.Pop(maxHeap)
            }
            i++
            j++
            index++
        }
    }

	
    return res
} 

```