```json
{
 "date": "2024.04.08 22:35",
  "tags":["UNIONFIND"],
  "title": "Prims-UnionFind"
}
#### Disjoint Sets

并查集,又称为不交集数据集合,是一种数据结构。
用于处理一系列没有重复的元素的合并及查询问题。

- 查询

find(x): 

1. 并查集通常由一个数组表示，每个元素对应一个节点，并且每个节点有一个指向父节点的指针。find(x) 操作通过迭代查找元素的父节点，直到找到根节点（也就是代表元素），并返回该根节点的索引。

2. 这个操作的目的是确定元素 x 属于哪个集合。

- 合并


merge(x, y) 是并查集中的一个操作，它将两个集合（或者说是两个元素所属的集合）合并为一个新的集合。

1. 在这个操作中，假设 x 属于集合 Si，y 属于集合 Sj。执行 merge(x, y) 操作将导致集合 S 中 Si 和 Sj 被移除，然后将它们合并为一个新的集合 Si ∪ Sj，即 Si 和 Sj 的并集。

2. 根据您提供的示例，merge({a}, {d}) 的结果是将集合 {a} 和 {d} 合并为 {a, d}，并更新集合 S 为 { {a, d}, {b}, {c}, {e} }。

3. 在并查集中，合并操作通常伴随着路径压缩等优化技巧的使用，以提高整个并查集的性能


#### 并查集的应用

- 找到图的连通分量(将图中的顶点按照连接性质分成的最大子集)

步骤如下:
    1. 初始化: 创建一个并查集,其他每个顶点初始时属于自己的集合。
    2. 合并操作: 对于图中的每一条边,如果边的两个端点不在同一结合中,则将它们的集合合并为一个。
    3. 找到连通分量: 合并完成后，每个集合的根节点代表一个连通分量，因此可以通过对每个顶点执行 find 操作来确定其所属的连通分量。


```go

type UnionFind struct {
	parent []int
	// 秩的作用就是通过记录树的高度（或者近似的高度），在合并操作时优化树的高度，从而尽可能保持树的平衡
	// ，避免树的高度过高。通过将较短的树链接到较高的树上，可以减少整个树的高度，
	// 从而提高查找操作（find）和合并操作（union）的效率。
	rank []int
}

func NewUnionFind(n int) *UnionFind {
	uf := &UnionFind{
		parent: make([]int, n),
		rank:   make([]int, n),
	}

	for i := 0; i < n; i++ {
		uf.parent[i] = i
		uf.rank[i] = 0
	}

	return uf
}

// 保持并查集平衡 防止树过高
func (uf *UnionFind) union(x, y int) {
	rx, ry := uf.find(x), uf.find(y)
	if rx != ry {
		if uf.rank[rx] < uf.rank[ry] {
			uf.parent[rx] = ry
		} else if uf.rank[rx] > uf.rank[ry] {
			uf.parent[ry] = rx
		} else {
			uf.parent[ry] = rx
			uf.rank[rx]++
		}
	}
}

// 路径压缩  +  减少树高 拽出
func (uf *UnionFind) find(x int) int {
	if uf.parent[x] != x {

		uf.parent[x] = uf.find(uf.parent[x])
	}
	return uf.parent[x]
}

func FindConnectedComponents(graph map[int][]int) [][]int {
	numVer := len(graph)
	uf := NewUnionFind(numVer)
	for u := range graph {
		for _, v := range graph[u] {
			if uf.find(u) != uf.find(v) {
				uf.union(u, v)
			}
		}
	}

	components := make(map[int][]int)

	for i := 0; i < numVer; i++ {
		root := uf.find(i)
		components[root] = append(components[root], i)
	}

	var result [][]int
	for _, vertices := range components {
		result = append(result, vertices)
	}
	return result

}


```

并查集是一种重要的数据结构，它主要用于解决元素的动态连通性问题。以下是一些并查集的应用场景：

1. **网络连接问题：** 在计算机网络中，可以使用并查集来维护各个计算机之间的连接关系。例如，用于判断网络中的两台计算机是否能够通信。

2. **图论算法：** 并查集在图论中有着广泛的应用，例如在 Kruskal 算法中用于寻找最小生成树，以及在计算连通分量等方面。

3. **区域分割问题：** 在图像处理和计算机视觉领域，可以使用并查集来对图像中的区域进行分割和合并。

4. **动态连通性问题：** 并查集可以用于解决动态连通性问题，例如在社交网络中判断两个人是否具有间接的社交关系。

5. **数据库系统：** 并查集可以用于数据库系统中对数据的连接和分组操作，例如在关系型数据库中对表进行连接操作。

空间复杂度：并查集的空间复杂度主要取决于存储父节点和秩的数组。通常情况下，父节点数组和秩数组的空间复杂度均为 O(n)，其中 n 是元素的数量。

时间复杂度：并查集的时间复杂度取决于 find 和 union 操作的实现。在路径压缩和按秩合并的优化下，find 操作的平均时间复杂度近似为 O(α(n))，其中 α 是阿克曼函数的反函数，α(n) 的增长极其缓慢，可以认为是一个很小的常数。而 union 操作的时间复杂度也近似为 O(α(n))。因此，对于包含 n 个元素的并查集，整体的时间复杂度近似为 O(mα(n))，其中 m 是操作的次数。由于 α(n) 很小，因此并查集的操作通常具有很高的效率。