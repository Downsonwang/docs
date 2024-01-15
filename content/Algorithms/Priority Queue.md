```json
{
 “date”: “2024.01.09 20:44”,
 “tags”:[“Priority Queue”,”优先级队列“]
}
```

### 优先级队列简介
优先级队列:一种特殊的队列。当访问队列元素时,具有最高优先级的元素最先删除。

优先队列与普通队列最大的不同点在于出队顺序。
- 普通队列的出队顺序跟入队顺序相关,符合First In,First Out
- 优先队列的出队顺序跟入队顺序无关, 优先队列是按照元素的优先级来决定出队顺序的
- 队尾出队,优先级最高的元素出队


### 二叉堆结构实现优先级队列 O(log2n):

二叉堆的定义: 符合以下两个条件之一的完全二叉树:
大顶堆: 根节点值>=子节点值 小顶堆: 根节点值 <= 子节点值

- 定义任务和优先级队列

```go
type Item struct {
  value string
  priority int 
  index int 
}

type PriorityQueue []*Item
```

- 实现sort.Interface接口以便在堆操作中使用
```go
func (pq PriorityQueue) Len() int {
  return len(pq)
}

func (pq PriorityQueue) Less(i,j int) bool {
  return pq[i].priority > pq[j].priority
}

func (pq PriorityQueue) Swap(i,j int) {
  pq[i],pq[j] = pq[j],pq[i]
  pq[i].index = i
  pq[j].index = j
}

func (pq *PriorityQueue) Push(x interface{}){
  n :=len(*pq)
  item := x.(*Item)
  item.index = n
  *pq = append(*pq,item)
}

func (pq *PriorityQueue) Pop() interface{} {
  old := *pq
  n := len(old)
  item := old[n-1]
  old[n-1] = nil
  item.index = -1
  *pq = old[0 : n-1]
  return item
}
```


### Heap
对于同一组数据,我们可以构建多种不同形态的堆

1.往堆中插入一个元素
