### SORTED_SET 
- 数据排序有利于数据的有效展示,需要提供一种可以根据自身特征进行排序的方式
- 在SET的存储结构基础上添加可排序字段(score)(它不是数据 不用来存储数据只用来排序)

### SORTED_SET基本操作
- 添加数据
```
zadd key score1 member1 [score2 member2]
```
- 获取全部数据
```
zrange key start stop 
zrevrange key start stop 
```
- 删除数据
```
zrem key member [member ...]
```

- 按条件获取数据
```
zrangebyscore key min max [withscores][limit]
zrevrangebyscore key max min [withscores]
```

- 条件删除数据
```
zremrangebyrank key start stop
zremrangebyscore key min max
```

注意:
- min与max用于限定搜索查询的条件 
- start与stop用于限定查询范围,作用于索引,表示开始和结束索引
- offset与count用于限定查询范围,作用于查询结果,表示开始位置和数据总量

- 获取集合数据总量
```
zcard key
zcount key min max
```
- 集合交、并操作
```
zinterstore dest numkeys key [key ...]
zunionstore dest numkeys key [key ...]
```

### 数据的扩展操作
业务场景: TOP K , 活跃度统计,游戏好友亲密度

- 获取数据对应的索引(排名)
```
zrank key member 
zrevrank key member
```
- score值获取与修改
```
zscore key member

zincrby key increment member 
```

```go
type Article struct {
	ID      int
	Title   string
	Views   int64
	Content string
}

var ZSetKey = "article"

func DoCreateArticle() {
	insertList := []*redis.Z{
		&redis.Z{Score: 1, Member: 11},
		&redis.Z{Score: 1, Member: 12},
		&redis.Z{Score: 1, Member: 13},
		&redis.Z{Score: 1, Member: 14},
		&redis.Z{Score: 3, Member: 15},
		&redis.Z{Score: 1, Member: 16},
		&redis.Z{Score: 1, Member: 17},
		&redis.Z{Score: 1, Member: 18},
	}

	num, err := Rdb.ZAdd(ZSetKey, insertList...).Result()
	if err != nil {
		fmt.Println(err)
		return
	}
	fmt.Println("添加成功%d\n", num)
	for i := 11; i <= 18; i++ {
		article := Article{
			ID:      i,
			Title:   "文章" + strconv.Itoa(i) + "的标题",
			Views:   1,
			Content: "卧槽牛皮 ",
		}
		articleMap.Store(strconv.Itoa(i), article)
	}
	ret, err := Rdb.ZRevRangeWithScores(ZSetKey, 0, -1).Result()
	if err != nil {
		fmt.Println("获取所有失败,err:%v\n", err)
		return
	}

	for k, z := range ret {
		fmt.Println(k, z.Score, z.Member, articleMap[z.Member.(string)].Title)
	}

	// 获取前5
	top5Ret, err := Rdb.ZRevRangeWithScores(ZSetKey, 0, 4).Result()
	if err != nil {
		return
	}
	for k, z := range top5Ret {
		fmt.Println(k, z.Score, z.Member, articleMap[z.Member.(string)].Title)
	}
}
```

### 注意事项
- score保持的数据存储空间是64位,如果是整数范围是-9～9xxxx
- score保存的数据也可以是一个双精度的double值,基于双精度浮点数的特征,可能会丢失精度,谨慎使用
- sorted_set底层存储还是基于SET结构的,因此数据不能重复,重复了只会保持下一次修改的值

### sorted_set应用场景 - 阶级
- VIP临时体验VIP

- 权重的任务队列和消息队列模型构建


### 业务场景
使用微信过程中,当微信接收消息后,会默认将最近的消息置顶,当多个好友及关注的订阅号同时发送消息时,该排序会不停的进行交替。同时还可以将重要的会话设置为置顶。 一旦用户离线,再次打开微信会按照什么方式展示?

- 依赖LIST的数据具有顺序的特征对消息进行管理,将list结构作为栈使用
- 对置顶与普通会话分别创建独立的list分别管理
- 当某个list中接收到用户消息后,将消息发送方的id从list的一侧加入list
- 多个相同id发出消息反复入栈会出现问题,在入栈之前先删除之前对应的记录再插入
- 推送消息时先推送置顶会话list,再推送普通会话list,推送完成的list清楚所有数据
- 消息的数量,采用计数器的思想进行记录,伴随list操作进行更新



