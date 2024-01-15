### RabbitMQ的整体架构及核心概念

- publisher: 消息发送者
- consumer: 消息消费者
- queue: 队列,存储消息
- exchange: 交换机,负责路由消息

publisher -> exchange -> queue  -> consumer

### 消息可靠性问题 

- 发送者的可靠性
- MQ的可靠性
- 消费者的可靠性
- 延迟消息

### 发送者可靠性
通过配置我们可以开启链接失败后的重连机制
```
connection-timeout:  1s #设置MQ的连接超时时间
retry:
  enbale:true # 开启超时重试机制
  initial-interval: 1000ms # 失败后的初始等待时间
  multiplier: 1  # 等待时长倍数
  max- attemptes: 3 # 最大重试次数
  
```



MQ的可靠性
在默认情况下,RabbitMQ会将接收到的信息保存到内存中以降低消息收发的延迟,这样会导致两种问题:
- 一旦MQ宕机,内存中的消息会丢失
- 内存空间有限,当消费者故障或处理过慢时,导致消息堆积

数据持久化:
交换机持久化
队列持久化
消息持久化


## Lazy Queue
惰性队列的特征如下:
1. 接收到消息后直接存入磁盘而非内存
2. 消费者要消费哦信息时才会从磁盘中读取并加载到内存
3. 支持数百万条的消息存储


## 消费者可靠性

- 消费者确认机制
   - 当消费者处理消息结束后,应该向RabbitMQ发送一个回执,告知RabbitMQ自己消息处理状态
   - ack ： 成功处理消息 ,RabbitMQ从队列中删除该消息
   - nack: 消息处理失败, RabbitMQ需要再次投递消息
   - reject: 消息处理失败并拒绝该消息,RabbitMQ从队列中删除该消息
  
  
## 延迟消息
延迟消息:生产者发送消息时指定一个时间,消费者不会立刻收到消息,而是在指定时间内收到消息.
延迟任务:设置在一定时间之后才执行的任务

## Work Queue

模式说明: 一个生产者对应2个或多个消费者,但这两个消费者处于竞争的关系

应用场景: 任务过重或任务较多的情况使用工作队列可以提高任务处理速度


## 消息确认机制
- 我们不希望发生丢失工作数据的情况,假如消费者down掉了,可以有另外一个来接任它。
```
If a consumer dies (its channel is closed, connection is closed, or TCP connection is lost) without sending an ack, RabbitMQ will understand that a message wasn't processed fully and will re-queue it. If there are other consumers online at the same time, it will then quickly redeliver it to another consumer. That way you can be sure that no message is lost, even if the workers occasionally die.

```


- 手动确认消息(d.Ack(false))

## Message durability

- 如果服务器停止,我们的任务仍然会丢失 , 确保消息不丢失的情况:我们需要将消息队列和消息保持为持久的
```

	q, err := ch.QueueDeclare(
		"hello", // name
		true,    // durable
		false,   // delete when unused
		false,   // exclusive
		false,   // no-wait
		nil,     // arguments
	)
	
```

```

err = ch.PublishWithContext(ctx,
		"",     // exchange
		q.Name, // routing key
		false,  // mandatory
		false,
		amqp.Publishing{
			DeliveryMode: amqp.Persistent, //持久性
			ContentType:  "text/plain",
			Body:         []byte(body),
		})
		
```

## Note on message persistence
```
Marking messages as persistent doesn't fully guarantee that a message won't be lost. Although it tells RabbitMQ to save the message to disk, there is still a short time window when RabbitMQ has accepted a message and hasn't saved it yet. Also, RabbitMQ doesn't do fsync(2) for every message -- it may be just saved to cache and not really written to the disk. The persistence guarantees aren't strong, but it's more than enough for our simple task queue. If you need a stronger guarantee then you can use publisher confirms.
```


## 公平调度

```
You might have noticed that the dispatching still doesn't work exactly as we want. For example in a situation with two workers, when all odd messages are heavy and even messages are light, one worker will be constantly busy and the other one will do hardly any work. Well, RabbitMQ doesn't know anything about that and will still dispatch messages evenly.

```

- 使用消息确认和预取计数,您可以设置工作队列。即使Rabbitmq重新启动,持久型选项也能让任务继续存在

## Pub/Sub - 向多个消费者传递消息

```
The core idea in the messaging model in RabbitMQ is that the producer never sends any messages directly to a queue. Actually, quite often the producer doesn't even know if a message will be delivered to any queue at all.

```

生产者仅仅发送消息到交换机,然后根据交换机里的东东然后再去搞到队列里

![Alt text](https://www.rabbitmq.com/img/tutorials/exchanges.png)

```
err = ch.ExchangeDeclare(
  "logs",   // name
  "fanout", // type
  true,     // durable
  false,    // auto-deleted
  false,    // internal
  false,    // no-wait
  nil,      // arguments
)

```


- Code
```
func failOnError(err error, msg string) {
	if err != nil {
		log.Panicf("%s:%s", msg, err)
	}
}

func main() {
	conn, err := amqp.Dial("amqp://guest:guest@localhost:5672/")
	failOnError(err, "Failed to connect to RabbitMQ")
	defer conn.Close()

	ch, err := conn.Channel()
	failOnError(err, "Failed to open a channel")
	defer ch.Close()

	err = ch.ExchangeDeclare(
		"logs",
		"fanout",
		true,
		false,
		false,
		false,
		nil,
	)
	failOnError(err, "Failed to declare an exchange")

	ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
	defer cancel()
	body := bodyFrom(os.Args)
	ch.PublishWithContext(ctx, "logs", "", false, false, amqp.Publishing{
		ContentType: "text/plain",
		Body:        []byte(body),
	})

	failOnError(err, "Failed to publish a message")
	log.Printf("[x] Sent %s", body)

}

func bodyFrom(args []string) string {
	var s string
	if (len(args) < 2) || os.Args[1] == "" {
		s = "hello"
	} else {
		s = strings.Join(args[1:], " ")
	}
	return s
}

```


```
func failOnError(err error, msg string) {
	if err != nil {
		log.Panicf("%s: %s", msg, err)
	}
}

func main() {
	conn, err := amqp.Dial("amqp://guest:guest@localhost:5672/")
	failOnError(err, "Failed to connect to RabbitMQ")
	defer conn.Close()

	ch, err := conn.Channel()
	failOnError(err, "Failed to open a channel")
	defer ch.Close()

	err = ch.ExchangeDeclare(
		"logs",   // name
		"fanout", // type
		true,     // durable
		false,    // auto-deleted
		false,    // internal
		false,    // no-wait
		nil,      // arguments
	)
	failOnError(err, "Failed to declare an exchange")

	q, err := ch.QueueDeclare(
		"",    // name
		false, // durable
		false, // delete when unused
		true,  // exclusive
		false, // no-wait
		nil,   // arguments
	)
	failOnError(err, "Failed to declare a queue")
	err = ch.QueueBind(
		q.Name,
		"",
		"logs",
		false,
		nil,
	)
	failOnError(err, "Failed to bind a queue")

	msg, err := ch.Consume(
		q.Name,
		"",
		true,
		false,
		false,
		false,
		nil,
	)
	failOnError(err, "注册消费者失败")

	var forever chan struct{}

	go func() {
		for d := range msg {
			log.Printf("receive a message : %s", d.Body)
			dotCount := bytes.Count(d.Body, []byte("."))
			t := time.Duration(dotCount)
			time.Sleep(t * time.Second)
			log.Printf("Done")
			d.Ack(false)
		}
	}()
	log.Printf("[* ] Waiting for messages")
	<-forever

}
```


### Routing - 路由

```
err = ch.QueueBind(
  q.Name, // queue name
  "",     // routing key
  "logs", // exchange
  false,
  nil)

```

- A binding is a relationship between an exchange and a queue. This can be simply read as: the queue is interested in messages from this exchange.

### Direct exchange

```
We will use a direct exchange instead. The routing algorithm behind a direct exchange is simple - a message goes to the queues whose binding key exactly matches the routing key of the message.

```

### Topic exchange

Topic模式与Direct模式相比, 他们都可以根据Routing key把消息路由到对应的队列上,但是Topic模式相对于Direct来说,它可以基于多个标准进行路由。也就是在队列绑定Routing Key的时候使用通配符。

### 使用Topic模式的要点
```
routing key必须是由”.”进行分隔的单词列表,最大限制为255字节

```


### Topic 总结

通过使用通配符实现灵活性的应用有很多，例如nginx的请求转发，gateway为请求过滤等等都是使用了统配符的技术。通过这种联想来对知识进行结构化，找相同和不同，思考能力和学习力也会有很大的提高。


