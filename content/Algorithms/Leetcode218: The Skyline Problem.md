```json
{
 “date”: “2024.01.09 11:06”,
 “tags”: “[the skyline problem][priority queue(heap)]”
}
```

### 天际线问题的题意
题目中,会给出多个建筑物的起点坐标,终点坐标以及高度坐标,让你求出这个每栋建筑物的[起始点,最高点], 但是如果有多个建筑物重叠后,有可能会出现某个建筑物的起始点对应是其他建筑物的高度,这个时候,我们需要找到上一个建筑物的结束点作为该建筑物的起始点。需要3个参数分别是记录我们当前的信息curPoint(当前位置),curHeight(当前高度),nextEnd(结束位)。

### 题解的思想
首先,根据要拿到各个建筑物所对应的高度,我首先想到的是维护一个MaxHeap,Maxheap的作用:
- 快速获取当前位置最高的建筑物, 
- 高度的动态变化,  将其高度加入最大堆。当发现当前建筑物发生变化时,要更新天际线。

这样,保持最大堆的性质,堆顶元素就始终是当前位置最高的建筑物的高度。
- 判断有无建筑物需要处理
- 提取每个建筑物的坐标,当curpoint等于开始坐标的时候,我们将对应的Push进堆里,在堆里该建筑物的结束点是heap[0][1]
- 对比这个建筑物的堆里最大高度高度是否比当前高度高,如果高把当前高度附值为此高度并更新天际线
- 无建筑物处理的话, 出堆,持续将最高的建筑物从队列中弹出
- 如果优先级队列非空,将当前的最高的高度添加到天际线
- 另外,如果为空,则添加当前位置的坐标的天际线信息为0并设置下一个处理点

```go
type PriorityQueue [][]int
func (pq PriorityQueue) Len() int {return len(pq)}
func (pq PriorityQueue) Less(i,j int) bool {
  if pq[i][2]==pq[j][2]{
    return pq[i][1] > pq[j][1]
  }
  return pq[i][2] > pq[j][2]
} 
func (pq PriorityQueue) Swap(i,j int) {
  pq[i],pq[j] = pq[j],pq[i]
}
func (pq PriorityQueue) Push(x interface{}){
  v := x.([]int)
  *pq = append(*pq,v)
}
func (pq PriorityQueue) Pop() interface{} {
  old := *pq
  n := len(old)
  item := old[n-1]
  old[n-1] = nil
  *pq = old[0:n-1]
  return item
}
```

- 具体的实现方法:

```go
func getSkyline(buildings [][]int)[][]int{
  pq := make(PriorityQueue,0)
  var skyline [][]int
  i := 0
  nextEnd := 1 << 32 + 1 
  curHeight := 0
  for i < len(buildings) || len(pq) > 0 {
    if i < len(buildings) && buildings[i][0] <= nextEnd {
      curPoint := buildings[i][0] // Record our start x
      for i < len(buildings) && buildings[i][0]  <= nextEnd{
        // 当前的建筑物Push进去
        head.Push(&pq,buildings[i]) 
        i++ 
        
      }
      nextEnd = pq[0][1] //当前最高建筑物的结束点为下一个建筑物做准备
      if curHeight < pq[0][2]{
         curHeight = pq[0][2]
         
         skyline = append(skyline,[]int{curPoint,curHeight})
        
      }
    }else {
      // 没有建筑物需要处理
      for len(pq) > 0 && pq[0][1] <= nextEnd {
        heap.Pop(&pq)
      }
      if len(pq) > 0 {
        skyline = append(skyline,[]int{nextEnd,pq[0][2]})
        curHeight = pq[0][2]
        nextEnd = pq[0][1]
        
      }else {
        
      skyline = append(skyline,[]int{nextEnd,0})
        curHeight = pq[0][2]
        nextEnd =1 << 32 + 1
      
    }
     
   
  }
}
```


- 最高的建筑物每次肯定都是pq[0][2]  因为我们维护的是最大堆
- 每次处理结束点时,需要从最大堆中移除相应的建筑物高度。确实堆顶元素仍然是当前位置最高建筑物的高度。
- 正确处理天际线信息: 在每个位置，根据当前位置最高建筑物的高度，更新天际线信息。注意处理相邻位置高度变化的情况，避免重复添加相同高度的点。
- 结束为止的处理,在处理建筑物时，要考虑所有建筑物都处理完后的情况。在算法的最后，确保将所有剩余的天际线信息添加到结果中。


### 感想
草泥马的司马题目,南斯YourDad了已经, 吗的晚上又得复习堆了,我就是个垃圾卧槽了好难受啊. 这什么答辩的东西,日了呀。 傻逼的人 傻逼的题 傻逼的心从来都不复习 