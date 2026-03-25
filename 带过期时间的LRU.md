带过期时间的LRU

```go
package main

import (
	"fmt"
	"time"
)
type Node struct{
	key,value int
	expireAt  int64
	prev,next *Node
}
type LRUCache struct{
	capacity   int
	dummy	   *Node
	keytonode  map[int]*Node
}
func Construtor(capacity int)LRUCahche{
	dummy:=&Node{}
	dummy.prev=dummy
	dummy.next=dummy
	return LRUCache{
		capacity:capacity,
		dummy:dummy,
		ketonode:map[int]*Node{},
	}
}
func(c *LRUCache) remove(x *Node){
	x.prev.next=x.next
	x.next.prev=x.prev
}
func(c *LRUCache)pushFront(x *Node){
	x.prev=c.dummy
	x.next=c.dummy.next
	x.prev.next=x
	x.netx.prev=x
}
func(c *LRUCache)getNode(key int)*Node{
	node:=keytonode[key]
	if node==nil {
		return nil
	}
	if time.Now().Unix()>node.expireAt{
		c.remove(node)
		delete(c.keytonode,key)
		return nil
	}
	c.remove(node)
	c.pushFront(node)
	return node
}
func(c *LRUCache)Get(key int)int{
	node:=c.keytonode[key]
	if node==nil{
		return -1
	}
	return node.value
}
func(c *LRUCache)Put(key,value int,ttl int64){
    expiretime:=time.Now().Unix()+ttl
    node:=c.getNode(key)
    if node!=nil{
        node.value=value
        node.expireAt=expiretime
        return
    }
    node=&Node{
        key:key,
        value:value,
        expireAt:expiretime,
    }
    c.keytonode[key]=node
    c.pushFront(node)
    if len(c.keytonode)>c.capacity{
        backNode:=dummy.prev
        delete(c.keytonode,backnode.key)
        c.remove(backnode)
    }
}
func main() {
	fmt.Println("=== LRU 缓存大军演习开始 ===")
	// 建立一个容量为 2 的缓存大营
	cache := Constructor(2)

	// 存入物资 1，保质期 2 秒
	cache.Put(1, 100, 2)
	fmt.Println("存入物资 1，保质期 2 秒。")

	// 存入物资 2，保质期 5 秒
	cache.Put(2, 200, 5)
	fmt.Println("存入物资 2，保质期 5 秒。")

	// 瞬间获取物资 1
	fmt.Printf("立刻获取物资 1: %d\n", cache.Get(1)) // 预期: 100

	// 让大军原地休息 3 秒...
	fmt.Println("大军休息 3 秒...")
	time.Sleep(3 * time.Second)

	// 再次尝试获取物资 1 (应该已经腐坏被踢出了)
	fmt.Printf("3 秒后获取物资 1: %d\n", cache.Get(1)) // 预期: -1 (因为只活 2 秒)

	// 获取物资 2 (还活得好好的)
	fmt.Printf("3 秒后获取物资 2: %d\n", cache.Get(2)) // 预期: 200 (因为能活 5 秒)

	// 存入物资 3，测试容量挤压
	cache.Put(3, 300, 10)
	fmt.Println("存入物资 3。此时大营应该只剩 [2, 3]。")

	// 物资 2 刚才被 Get 过，所以它是最新鲜的；物资 3 刚放进去也是新鲜的。
	// 大营容量为 2，运行正常！
}
```

