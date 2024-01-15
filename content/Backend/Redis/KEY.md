### KEY通用操作 - key是一个字符串,通过key获取redis中保存的数据

- Key基本操作
删除指定key:
```
del key
```

获取key是否存在
```
exists key
```

获取key的类型
```
type key

```


### 扩展操作(时校性控制)
- 为指定key设置有效期
```
expire key seconds
pexpire key milliseconds
expireat key timestamp
pexpireat key milliseconds-timestamp
```

- 获取key的有效时间
```
ttl key
pttl key

```
- 切换key从时效性转换为永久性
```
persist key

```

- 查询key
```
keys pattern

```

- key改名
```
rename key newkey
renamenx key newkey
```

- 所有key排序
```
sort
```

- 其他key通用操作
```
help @generic
```


### 数据库通用操作
key的重复问题
- 切换数据库
```
select index
```
- 数据移动
```
move key db
```

- 数据清楚
```
dbsize
flushdb
flushall

```