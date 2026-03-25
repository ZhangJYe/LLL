## go-Redis



```go
import (
	"context"
	"fmt"
	"github.com/redis/go-redis/v9"
)

var ctx = context.Background()
var rdb = redis.NewClient(&redis.Options{
	Addr: "localhost:6379",
})

```

### 1. String：缓存与计数 (Cache & Counter)

这是最基础的类型。由于 Redis 是单线程处理命令的，所以它的自增操作天然是**原子性**的，非常适合做计数器（比如文章阅读量、点赞数、限流器）。

```go
// 场景A：缓存用户 Session 或 网页数据 (带过期时间)
err := rdb.Set(ctx, "user:session:1001", "token_xyz123", time.Hour).Err()

// 场景B：文章阅读量计数器 (原子自增)
// INCR article:views:100
views, err := rdb.Incr(ctx, "article:views:100").Result()
fmt.Printf("当前阅读量: %d\n", views)
```

### 2. Hash：存对象字段 (Object Fields)

如果用 String 存 JSON 字符串，每次修改一个字段都要重新序列化和反序列化整个大字符串。

Hash 允许你像操作 Map 一样，单独修改或获取对象里的某个字段，极其高效。

```go
// 场景：存储用户信息
userKey := "user:profile:1001"

// 写入对象的多个字段
rdb.HSet(ctx, userKey, map[string]interface{}{
    "name": "Tom",
    "age":  25,
    "city": "San Jose",
})

// 单独获取或修改某个字段
rdb.HIncrBy(ctx, userKey, "age", 1) // 过了个生日，年龄+1
name, _ := rdb.HGet(ctx, userKey, "name").Result()
fmt.Printf("用户名: %s\n", name)
```

### 3. List：队列 (Queue)

List 是一个双向链表。我们可以用它做简单的**消息队列**（先进先出），或者**最新内容展示**（比如朋友圈的时间线 Timeline）。

- 队列：`LPUSH` (左进) + `RPOP` (右出)
- 栈：`LPUSH` (左进) + `LPOP` (左出)

```go
queueKey := "queue:email_tasks"

// 生产者：把任务推入队列左侧
rdb.LPush(ctx, queueKey, "task_1", "task_2")

// 消费者：从队列右侧弹出任务 (阻塞版本 BRPop 常用于实际工作，没有数据时会等待)
task, err := rdb.BRPop(ctx, 0, queueKey).Result() // 0表示一直阻塞等待
if err == nil {
    fmt.Printf("处理任务: %s\n", task[1]) // task[0] 是 key 名字，task[1] 是值
}
```

### 4. Set：去重与集合运算 (Deduplication & Set Operations)

Set 是无序且不重复的。它的精髓在于**集合间的交集、并集、差集运算**。经典场景：共同好友、抽奖去重、用户画像标签。

```go
user1Tags := "user:1:tags"
user2Tags := "user:2:tags"

// 添加标签 (自动去重)
rdb.SAdd(ctx, user1Tags, "golang", "redis", "docker")
rdb.SAdd(ctx, user2Tags, "java", "redis", "kafka")

// 找共同爱好 (交集 SINTER)
common, _ := rdb.SInter(ctx, user1Tags, user2Tags).Result()
fmt.Printf("共同爱好: %v\n", common) // 输出: [redis]
```

### 5. ZSet：排行榜 (Leaderboard)

ZSet (Sorted Set) 在 Set 的基础上给每个元素加上了一个浮点数分数（Score），Redis 会根据这个分数自动排序。经典场景：游戏积分榜、热搜榜、延时队列（把时间戳当分数）。

```go
boardKey := "leaderboard:game_x"

// 玩家获得积分
rdb.ZAdd(ctx, boardKey, redis.Z{Score: 9500, Member: "PlayerA"})
rdb.ZAdd(ctx, boardKey, redis.Z{Score: 12000, Member: "PlayerB"})
rdb.ZAdd(ctx, boardKey, redis.Z{Score: 8800, Member: "PlayerC"})

// 获取排名前 2 的玩家 (分数从高到低：ZRevRange)
topPlayers, _ := rdb.ZRevRangeWithScores(ctx, boardKey, 0, 1).Result()
for i, p := range topPlayers {
    fmt.Printf("第 %d 名: %v, 分数: %f\n", i+1, p.Member, p.Score)
}
```

### 6. Bitmap & HyperLogLog：海量统计 (Memory-efficient Stats)

这两个是高级玩法，主打一个**极致省内存**。

- **Bitmap (位图)**：用二进制的 0 和 1 记录状态。场景：几千万用户的日活状态、一年的签到记录（一年才365个位，不到50字节！）。
- **HyperLogLog**：做基数统计（去重后的数量）。场景：亿级网页的 UV（独立访客）统计。占用内存极小（固定12KB），但有极小的误差（约0.81%）。

```go
// --- Bitmap: 用户当月签到记录 ---
signKey := "sign:1001:202602"
rdb.SetBit(ctx, signKey, 21, 1) // 2月22日签到 (偏移量算21)
count, _ := rdb.BitCount(ctx, signKey, &redis.BitCount{Wait: true}).Result()
fmt.Printf("本月签到天数: %d\n", count)

// --- HyperLogLog: 统计今天网站的 UV ---
uvKey := "uv:homepage:20260222"
rdb.PFAdd(ctx, uvKey, "ip_192.168.1.1", "ip_10.0.0.1", "ip_192.168.1.1") // 重复IP自动去重
uv, _ := rdb.PFCount(ctx, uvKey).Result()
fmt.Printf("今日网站UV: %d\n", uv)
```

### 7. Geo：地理位置 (Location)

基于 ZSet 封装，专门用来处理经纬度，常用于“附近的人”、“附近的门店”、打车软件等。

```go
geoKey := "shops:coffee"

// 录入两家咖啡店的经纬度
rdb.GeoAdd(ctx, geoKey, 
    &redis.GeoLocation{Longitude: -121.88, Latitude: 37.33, Name: "San Jose Coffee"}, // 当前你的城市
    &redis.GeoLocation{Longitude: -122.41, Latitude: 37.77, Name: "SF Coffee"},
)

// 查找当前坐标 50 公里内的咖啡店
res, _ := rdb.GeoSearchLocation(ctx, geoKey, &redis.GeoSearchLocationQuery{
    GeoSearchQuery: redis.GeoSearchQuery{
        Longitude: -121.90, 
        Latitude: 37.35,
        Radius: 50, // 50
        RadiusUnit: "km",
    },
    WithDist: true, // 返回距离
}).Result()

for _, shop := range res {
    fmt.Printf("找到店面: %s, 距离: %f km\n", shop.Name, shop.Dist)
}
```

### 8. Stream：专业消息队列 (Message Queue)

相比于 List 做队列，Stream 是 Redis 5.0 专为消息队列设计的重磅功能。它支持**消费者组 (Consumer Groups)**、消息持久化、消息确认 (ACK) 机制。非常适合替代轻量级的 Kafka 或 RabbitMQ。

```go
streamKey := "stream:orders"

// 生产者：发布一条订单消息 (* 表示让 Redis 自动生成消息ID)
rdb.XAdd(ctx, &redis.XAddArgs{
    Stream: streamKey,
    Values: map[string]interface{}{"order_id": "8888", "amount": 100},
})

// 消费者：读取最新消息
// 实际工程中通常使用 XReadGroup 配合消费者组进行更安全的消费
streams, _ := rdb.XRead(ctx, &redis.XReadArgs{
    Streams: []string{streamKey, "0"}, // 0 表示从头开始读
    Count:   1,
}).Result()

if len(streams) > 0 && len(streams[0].Messages) > 0 {
    msg := streams[0].Messages[0]
    fmt.Printf("收到消息 ID: %s, 数据: %v\n", msg.ID, msg.Values)
}
```

## 设计

作为初学者，当你接到“设计抽奖系统”的需求时，首先要做的不是写代码，而是**分析核心痛点**。一个真实的抽奖系统通常面临三大挑战：

1. **高并发读写**：瞬间涌入大量用户点击“抽奖”，传统关系型数据库（如 MySQL）扛不住这么高的 QPS。
2. **防超卖（Overselling）**：比如只有 10 台 iPhone，绝不能因为并发发出去 11 台。
3. **防重复（Deduplication）**：规则通常限制“每人只能中奖一次”，需要精准拦截用户的重复请求。

Redis 因为其**基于内存的极高性能**和**单线程执行命令的原子性**，天生就是解决这三个问题的完美利器。

### 第一步：兵器挑选（Redis 数据结构选型）

基于上一节课我们学的知识，你想想该用什么？

- **奖品库存**：用 **`List` (队列)**。

	我们将真实的奖品一个个塞进 List 里。用户抽奖时，用 `LPOP` 弹出一个。如果弹不出（返回 nil），说明奖品发完了。`LPOP` 是绝对原子的，天然防超卖！

- **中奖名单**：用 **`Set` (集合)**。

	利用 `SADD` 和 `SISMEMBER` 来记录和检查用户是否已经中过奖，保证每人最多中一次。

### 第二步：初始化奖品库存 (Admin 端)

在活动开始前，我们需要把奖品推入 Redis 队列中。

```go
// 初始化奖品池 (假设活动发 5 台 iPhone)
func InitPrizes() {
	prizeKey := "lottery:prizes:iphone"
	// 先清空历史数据（防止重复初始化）
	rdb.Del(ctx, prizeKey)
	
	// 把 5 个奖品推入 List
	// 实际业务中，这里推入的可能是具体的奖品实体 JSON 或唯一编码，这里为了简单用字符串
	prizes := []interface{}{"iphone_1", "iphone_2", "iphone_3", "iphone_4", "iphone_5"}
	err := rdb.LPush(ctx, prizeKey, prizes...).Err()
	if err != nil {
		fmt.Println("初始化奖品失败:", err)
		return
	}
	fmt.Println("奖品初始化成功，共 5 台 iPhone！")
}
```

### 第三步：抽奖核心逻辑（新手易错版 vs 教授黄金版）

#### ❌ 新手容易犯的错（Race Condition 竞态条件）

新手通常会这么写 Go 代码：

1. `rdb.SIsMember()` 检查用户是否在 Set 里。在？返回“已中奖”。
2. 不在的话，执行 `rdb.LPop()` 扣库存。
3. 扣除成功后，执行 `rdb.SAdd()` 把用户加入 Set。

**破绽在哪里？** 假设用户写了个并发脚本，同一时间发了 10 个请求过来： 这 10 个请求同时执行第一步，发现都没中奖，然后同时去执行 `LPop`，结果这个用户一个人抽走了好几台 iPhone！这就是经典的**并发竞态问题**。

#### ✅ 教授的工业级解法：使用 Lua 脚本

为了保证“**检查资格 -> 扣减库存 -> 记录中奖**”这三个步骤是**绝对原子化**的（在 Redis 执行这三步时，其他请求必须排队），我们会使用 **Lua 脚本**。Redis 完美内置了 Lua 引擎。

把这段核心代码保存在你的 Go 项目中：

```go
// 定义 Lua 脚本
// KEYS[1]: 用户的唯一标识 (如 userID)
// KEYS[2]: 奖品库存的 List Key
// KEYS[3]: 中奖名单的 Set Key
const drawScript = `
local user_id = KEYS[1]
local prize_key = KEYS[2]
local winner_set = KEYS[3]

-- 1. 检查是否已经中过奖 (防重复)
if redis.call("SISMEMBER", winner_set, user_id) == 1 then
    return -1 -- 返回 -1 表示已中奖
end

-- 2. 尝试获取奖品 (防超卖)
local prize = redis.call("LPOP", prize_key)
if not prize then
    return 0 -- 返回 0 表示没库存了 (谢谢参与)
end

-- 3. 记录中奖用户
redis.call("SADD", winner_set, user_id)
return 1 -- 返回 1 表示中奖成功
`

func PlayLottery(userID string) {
	prizeKey := "lottery:prizes:iphone"
	winnerSet := "lottery:winners:iphone"

	// 执行 Lua 脚本
	// Eval(ctx, 脚本内容, Keys数组, Args数组)
	result, err := rdb.Eval(ctx, drawScript, []string{userID, prizeKey, winnerSet}).Int()
	if err != nil {
		fmt.Println("系统繁忙，请稍后再试:", err)
		return
	}

	// 处理 Lua 脚本的返回值
	switch result {
	case -1:
		fmt.Printf("用户 %s: 你已经中过奖啦，把机会留给别人吧！\n", userID)
	case 0:
		fmt.Printf("用户 %s: 哎呀，奖品已经被抢光了，谢谢参与！\n", userID)
	case 1:
		fmt.Printf("用户 %s: 恭喜你！！抽中了一台 iPhone！\n", userID)
		// 实际业务中：这里可以通过 Kafka/Stream 发送一个消息给发货系统，进行异步发货
	}
}
```

### 第四步：模拟高并发测试

我们用 Goroutine 模拟 10 个人，外加一个恶意用户（并发发起了多次请求），来看看系统的表现：

```go
func main() {
	InitPrizes() // 放入 5 台 iPhone

	// 模拟 10 个正常用户并发抽奖
	for i := 1; i <= 10; i++ {
		go func(id int) {
			PlayLottery(fmt.Sprintf("user_%d", id))
		}(i)
	}

	// 模拟 1 个恶意用户 (user_100) 瞬间发起 3 次并发请求
	for i := 0; i < 3; i++ {
		go func() {
			PlayLottery("user_100")
		}()
	}

	// 阻塞等待一会，让协程跑完
	time.Sleep(2 * time.Second)
}
```

在这段代码中，我们一共使用了 6 个核心的 Redis 操作，它们各司其职：

1. **`DEL key [key ...]`**
	- **用法**: 删除一个或多个 Key。
	- **场景**: 在这里我们用于清理上一次测试残留的库存队列和中奖名单，确保每次运行环境干净。
2. **`LPUSH key value [value ...]`**
	- **用法**: 将一个或多个值插入到列表（List）的头部（左侧）。
	- **场景**: 构建奖品库存池。相当于把实物奖品一个个放进一个深桶里。
3. **`EVAL script numkeys key [key ...] arg [arg ...]`**
	- **用法**: 在 Redis 服务端执行一段 Lua 脚本。
	- **场景**: 这是高并发防超卖的核心！它保证了后续的 `SISMEMBER`、`LPOP`、`SADD` 作为一个整体被**原子化**执行。在脚本执行期间，其他客户端发来的命令只能排队等待，完美解决了并发竞态问题。
4. **`SISMEMBER key member`**
	- **用法**: 判断 `member` 元素是否是集合（Set）`key` 的成员。返回 1 表示存在，0 表示不存在。
	- **场景**: 防重复（防薅羊毛）。在发奖前，先查一下这个用户 ID 是不是已经在中奖名单里了。
5. **`LPOP key`**
	- **用法**: 移除并返回列表（List）的第一个元素（头部）。如果列表没有元素会返回 `nil`。
	- **场景**: 扣减库存。如果返回了具体的值（比如 "iphone_1"），说明库存扣减成功；如果返回空，说明奖品发完了。
6. **`SADD key member [member ...]`**
	- **用法**: 将一个或多个成员元素加入到集合（Set）中，已经存在于集合的成员元素将被忽略。
	- **场景**: 记录中奖者。将成功拿到奖品的用户 ID 存起来，供下次查验。

