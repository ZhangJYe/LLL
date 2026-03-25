### 单调栈

#### 239.滑动窗口最大值🔴

给你一个整数数组 `nums`，有一个大小为 `k` 的滑动窗口从数组的最左侧移动到数组的最右侧。你只可以看到在滑动窗口内的 `k` 个数字。滑动窗口每次只向右移动一位。

返回 *滑动窗口中的最大值* 。

```go
输入：nums = [1,3,-1,-3,5,3,6,7], k = 3
输出：[3,3,5,5,6,7]
```

```go
func maxSlidingWindow(nums []int,k int)[]int{
	if len(nums)==0 || k==0 {
	return []int{}
	}
	res:=make([]int,len(nums)-k+1)
	q:=[]int{}
    for i:=0;i<len(nums);i++{
        for len(q)>0 && nums[q[len(q)-1]]<nums[i] {
            q=q[:len(q)-1]
        }
        q=append(q,i)
        left:=i-k+1
        if q[0]<left{
            q=q[1:]
        }
        if left>=0 {
            res[left]=nums[q[0]]
        }
    }
    return res
}
```

#### （字节一面）402.移掉k位数🔴🔴

给你一个以字符串表示的非负整数 `num` 和一个整数 `k` ，移除这个数中的 `k` 位数字，使得剩下的数字最小。请你以字符串形式返回这个最小的数字。

```go
输入：num = "1432219", k = 3
输出："1219"
```

```go
func removeKdigits(num string, k int) string {
    q:=[]byte{}
    for i:=0;i<len(num);i++{
        digit:=num[i]
        for k>0 &&q len(q)>0 && q[len(q)-1]>digit {
            q=q[:len(q)-1]
            k--
        }
        q=append(q,digit)
    }
    // 特殊情况1：如果遍历完了，k 还没用完（比如 "12345", k=2）
    // 说明此时栈里已经是单调递增的了，那就删掉尾部最大的几个
    q=q[:len(q)-k]
    // 特殊情况2：去除前导零
    // 比如 10200, k=1 -> 删掉1变成 0200 -> 应该是 200
    res:=strings.TrimLeft(string(q),"0")
    // 特殊情况3：如果删光了或者结果是空字符串，返回 "0"
    if res=="" {
        return "0"
    }
    return res
}
```

通过观察发现如果是 1 4 3 2 2 19

递增1 2 3 4 5 6 k=3，就移动后三位。 123为最小值

递减6 5 4 3 2 1 k=3，就移动前三位。 321为最小值

##### strings.TrimLeft(string(q),"0")：去前导零、去左侧空格.

```
func TrimLeft(s string, cutset string) string
```

- **`s`**: 你要处理的原字符串。
- **`cutset`**: 你想移除的字符集合（注意：这里不是匹配一个单词，而是匹配这一堆字符里的**任意一个**）。

##### strings.TrimPrefix(s, "12")：去掉某个具体的单词或前缀

strings.TrimPrefix(s, "12")：只匹配开头是不是严格等于 "12"。

TrimPrefix("1212Hello", "12") 结果是 "12Hello"。它只去一次前缀。

### 滑动窗口

#### 3.无重复字符的最长字串

给定一个字符串 `s` ，请你找出其中不含有重复字符的 **最长 子串** 的长度。

外层 `for` 扩张，内层 `for` 遇不满足条件就收缩



整体思路就是 **“先强行拉人进来，一旦发现重复了，就把排在最前面的人按顺序踢出去，直到不重复为止”**。

- **`right` 指针**：负责在前面不断把新的人（新字符 `c`）拉进警戒线。
- **`left` 指针**：负责在后面把老的人踢出警戒线。
- **`ans`**：负责在每次“不重复”的时候，偷偷拿尺子量一下警戒线里现在有几个人，记录下历史最长长度。

**循环的奥秘：** 如果被踢走的 `s[left]` 刚好就是那个惹祸的“老 `c`”，那太好了，踢完之后 `mp[c]` 就变回 1 了，循环结束，警报解除。 如果被踢走的 `s[left]` 是无辜的路人（比如老 `c` 站在更靠右的位置），那警报依然没解除（`mp[c]` 还是 2），`for` 循环就会继续执行，继续踢下一个最左边的人，**直到把那个“老 `c`”给踢出去为止！**

```go
package main
import (
	"fmt"
)
func lengthOfLongestSubstring(s string) int {
    // 1. 打造“极速查重报警器”
    // 长度为 128 的数组，完美覆盖所有 ASCII 字符。里面存的是字符在当前窗口里的“出现次数”
    mp := [128]int{} 
    
    // 2. 初始化双指针和记录器
    left := 0 // 窗口的左边界
    ans := 0  // 全局最大长度
    
    // 3. right 指针主动向右扩张（遍历字符串）
    for right, c := range s { 
        // 步骤 A：新字符进入窗口，立刻登记造册，出现次数 +1
        mp[c]++ 
        
        // 步骤 B：查重报警系统触发！
        // 如果 mp[c] > 1，说明刚刚加进来的字符 c，在窗口里已经有一个一模一样的了！
        for mp[c] > 1 { 
            // 既然重复了，左边界 left 就必须被迫向右移动，把最左边的字符“踢出”窗口
            mp[s[left]]-- // 离开窗口的字符，出现次数 -1
            left++        // 左边界向右收缩
            
            // 注意：这个 for 循环会一直执行，直到那个导致重复的字符被彻底踢出窗口，
            // 此时 mp[c] 会重新变成 1，警报解除！
        }
        
        // 步骤 C：警报解除，此时窗口内绝对没有重复字符
        // 计算当前窗口的长度：right - left + 1，并挑战历史最高记录
        ans = max(ans, right-left+1) 
    }
    
    // 4. 返回历史最高记录
    return ans
}
func main(){
    var s string
    fmt.Scan(&s)   
    ans:=lengthOfLongestSubstring(s)
    fmt.Println("%d",ans)
}
```



#### 209.长度最小的数组

给定一个含有 `n` 个正整数的数组和一个正整数 `target` **。**

找出该数组中满足其总和大于等于 `target` 的长度最小的 **子数组** `[numsl, numsl+1, ..., numsr-1, numsr]` ，并返回其长度**。**

如果不存在符合条件的子数组，返回 `0` 。

```go
输入：target = 7, nums = [2,3,1,2,4,3]
输出：2
```

```go
func minSubArrayLen(target int, nums []int) int {
    n:=len(nums)
    res:=math.MaxInt
    sum:=0
    l:=0
    for r,num:=range nums{
        sum+=num
        for sum-nums[l]>=target{
            sum-=nums[l]
            l++
        }
        if sum>=target {
            res=min(res,r-l+1)
        }
    }
    if res>n {
        return 0
    }
    return res
}
```

#####  math.MaxInt:最大值

#### 567.字符串的排列

#### 76.最小覆盖子串🔴

给定两个字符串 `s` 和 `t`，长度分别是 `m` 和 `n`

返回 s 中的 **最短窗口 子串**，使得该子串包含 `t` 中的每一个字符（**包括重复字符**）。

如果没有这样的子串，返回空字符串 `""`。

测试用例保证答案唯一。

```go
输入：s = "ADOBECODEBANC", t = "ABC"
输出："BANC"
```

```go
func minWindow(s string, t string) string {
    // 1. 初始化“债务表”
    // cnt[c] > 0 表示还欠多少个该字符
    // cnt[c] <= 0 表示不欠该字符，或者窗口里该字符已经多出来了
    cnt := [128]int{}
    less := 0 // 记录还有多少种字符“还没凑够”
    for _, c := range t {
        if cnt[c] == 0 {
            less++ // 发现一种新零件需求，待凑齐种类+1
        }
        cnt[c]++ // 初始状态：欠债数量增加
    }

    // ansL 和 ansR 记录目前找到的最短覆盖子串的左右索引
    // 初始化 ansR 为 len(s) 是为了方便第一次比较最小长度
    ansL, ansR := -1, len(s)
    left := 0 // 窗口左边界

    // 2. 移动窗口右边界 right，开始“收货”
    for right, c := range s {
        cnt[c]-- // 收到一个零件，欠债减1 (减到负数说明多收了)
        
        if cnt[c] == 0 {
            // 如果减完后刚好等于 0，说明这种零件“刚好还清了”
            less-- 
        }

        // 3. 当 less == 0 时，说明清单上所有的零件种类都买够了
        // 此时尝试收缩左边界，来寻找以当前 right 为终点的“最短子串”
        for less == 0 {
            // 如果当前窗口 [left, right] 的长度比之前记录的更短，则更新
            if right-left < ansR-ansL {
                ansL, ansR = left, right
            }

            x := s[left] // 准备把左边界这个零件扔出去
            
            if cnt[x] == 0 {
                // 如果这个零件当前的欠债是 0，说明它是刚好凑齐的
                // 扔掉它之后，欠债就会变成 1，意味着这种零件“又不齐了”
                less++ 
            }
            
            cnt[x]++ // 零件扔出窗口，欠债数加1
            left++   // 左边界右移
        }
    }

    // 4. 检查是否找到过有效答案
    if ansL < 0 {
        return ""
    }
    // 注意：ansR 是闭区间索引，切片截取需要 ansR + 1
    return s[ansL : ansR+1]
}
```

**没有使用两个 Map 对比，而是只用了一个数组 `cnt` 来记录“债务（差值）”**。

为什么用切片不用map？

**减少 Map 开销**：`map` 的哈希计算和扩容是有开销的，而 `[128]int` 数组是连续内存，访问极快。

**`less` 的妙用**：

- 在普通解法中，你可能需要写一个 `check()` 函数遍历 Map 来判断是否满足条件（时间复杂度 $O(52 \times M)$）。
- 在灵神这个解法中，**只有在某种字符刚好凑齐（`cnt[c] == 0`）或刚好由于移出而不齐（`cnt[x] == 0`）时**，才去操作 `less`。
- 通过 `less` 的数值，我们能以 $O(1)$ 时间知道当前窗口是否“达标”。

##### 理解

1. 核心比喻：一份特殊的购物清单

假设你要去超市买东西，你的清单（字符串 `t`）是：**1个苹果，1个香蕉**。

- **`cnt` 数组**：就是你的**清单**。
	- 开始时：`苹果: 1, 香蕉: 1`。
	- **正数**表示你还**欠**超市多少个（你还没买到）。
	- **负数**表示你**多买了**多少个（你篮子里装得太多了）。
- **`less` 变量**：代表清单上**还有几种**水果没买够。
	- 开始时：`less = 2`（苹果、香蕉都没买够）。

2. 模拟运行：你在超市推车走 (右指针 `right`)

你推着车在货架（字符串 `s`）走，每看到一个水果，你就把它扔进篮子，并在清单上**减去**它。

情况 A：你捡到了一个清单上的水果（比如苹果）

1. 你把清单上的苹果减 1。
2. **关键动作**：如果苹果的欠债**刚好变成了 0**，说明苹果买够了！
3. **结果**：`less` 减 1（没买够的种类变少了）。

情况 B：你捡到了一个清单上没有的东西（比如西瓜）

1. 你照样在清单上减 1。西瓜本来是 0，现在变成了 **-1**。
2. **结果**：`less` 不变。因为 -1 意味着这东西你买多了，不影响你凑齐清单。

情况 C：你捡到了一个买过的水果（第二个苹果）

1. 你再减 1。苹果从 0 变成了 **-1**。
2. **结果**：`less` 也不变。因为你已经有苹果了，多拿一个不影响“买齐”这个状态。

3. 满足条件了！开始精简篮子 (左指针 `left`)

当 **`less == 0`** 时，说明你篮子里已经买齐了清单上所有的东西。这时候你开始嫌篮子太重了（窗口太长），想把左边的东西扔出去。

你从篮子里最旧的东西开始扔：

1. **扔掉一个西瓜**：
	- 西瓜在清单上从 -1 变回了 0。
	- **结果**：`less` 还是 0。因为你本来就没打算买西瓜，扔了也没事。
2. **扔掉一个“多出来的”苹果**：
	- 苹果在清单上从 -1 变回了 0。
	- **结果**：`less` 还是 0。因为你篮子里还有一个苹果，还是够的！
3. **扔掉那个“最后的”苹果**：
	- 苹果在清单上从 **0 变回了 1**。
	- **关键动作**：只要清单上的数字**变回了正数**，说明你欠债了！
	- **结果**：**`less` 加 1**。篮子又不达标了，你得赶紧停止扔东西，继续推车往右走去买新的

“为什么不管是不是清单里的字符，都要 `cnt[c]--`？”

**只要 `cnt[c] > 0`**，说明 `c` 是清单里的。

**只要 `cnt[c] <= 0`**，说明 `c` 是多出来的或者是无关的。

这样他就不需要写 `if (c 在 t 里面)` 这种判断了。通过 `cnt[c] == 0` 这个临界点，他能精准地捕捉到：**“刚好凑齐”** 和 **“刚好欠债”** 这两个瞬间。

**我们用最土的办法，看这 3 步：** 假设 `t = "AA"`, `s = "AAA"`

1. 初始：`cnt['A'] = 2`, `less = 1`
2. 右边进第一个 'A'：`cnt` 变 1，`less` 还是 1（没齐）。
3. 右边进第二个 'A'：`cnt` 变 0，**`less` 变 0**（齐了！）。
4. 右边进第三个 'A'：`cnt` 变 -1，`less` 还是 0（多了一个）。



### 双指针

#### 15.三数之和

给你一个整数数组 `nums` ，判断是否存在三元组 `[nums[i], nums[j], nums[k]]` 满足 `i != j`、`i != k` 且 `j != k` ，同时还满足 `nums[i] + nums[j] + nums[k] == 0` 。请你返回所有和为 `0` 且不重复的三元组。

**注意：**答案中不可以包含重复的三元组。

**1. 核心破局思路：降维打击**

怎么把找 3 个数变成找 2 个数？ **答案：先“固定”一个人，然后让另外两个人去配合他！**

1. **先排序**：这是极其关键的一步。排序后，数组从小到大排列，我们不仅能方便地使用双指针，还能**极其轻松地跳过重复元素**（因为重复的数字肯定挨在一起）。
2. **固定老大 (`nums[i]`)**：我们从左到右遍历数组，每次让 `nums[i]` 当“老大”。
3. **双指针找小弟 (`left` 和 `right`)**：老大固定了，现在的目标就变成了：在老大后面的数组里，找到两个数，使得 `nums[left] + nums[right] == -nums[i]`。
	- 如果三个数加起来 **小于 0**：说明总和太小了，`left` 指针往右移（变大）。
	- 如果三个数加起来 **大于 0**：说明总和太大了，`right` 指针往左移（变小）。
	- 如果 **等于 0**：恭喜，找到一组！记录下来。

```go
func threeSum(nums []int) [][]int {
    sort.Ints(nums)
    res:=[][]int{}
    n:=len(nums)
    
    for i:=0;i<n-2;i++ {
        if nums[i]>0 {
            break
        }
        if i>0 && nums[i]==nums[i-1]{
            continue
        }
        left:=i+1
        right:=n-1
        
        for left<right {
            sum:=nums[i]+nums[left]+nums[right]
            if sum==0 {
                res=append(res,[]int{nums[i],nums[left],nums[right]})
                for left<right && nums[left]==nums[left+1]{
                    left++
                }
                for left<right && nums[right]==nums[right-1]{
                    right--
                }
                left++
                right--
            }else if sum<0 {
                left++
            }else{
                right--
            }
              
        }
    }
    return res
}
```



### 数组

#### 53.最大子数组和

给你一个整数数组 `nums` ，请你找出一个具有**最大和的连续子数组**（子数组最少包含一个元素），返回其最大和。

```go
输入：nums = [-2,1,-3,4,-1,2,1,-5,4]
输出：6
解释：连续子数组 [4,-1,2,1] 的和最大，为 6 
```

暴力：

```
func maxSubArray(nums []int) int {
    maxcot,cot:=math.MinInt,0
    for i:=0;i<len(nums);i++{
        cot=0
        for j:=i;j<len(nums);j++{
            cot+=nums[j]
            maxcot=max(maxcot,cot)
        }
    }
    return maxcot
}
```

**贪心 / 动态规划（Kadane算法）—— 正解**

这道题的标准解法是 **Kadane 算法**，时间复杂度 $O(N)$。

它的核心思想不是“回溯”，而是**“止损”**。

想象你在滚雪球（累加和）：

1. 你前面的雪球如果是 **正的**（比如 10），哪怕你现在遇到一个负数（-2），你还是愿意带上前面的 10，因为 $10 + (-2) = 8$，还是比你光杆司令 -2 要强。
2. 你前面的雪球如果是 **负的**（比如 -5），无论你现在是正数还是负数，你都会觉得前面的雪球是**累赘**。你不如**扔掉前面的**，从自己开始重新滚一个新的雪球。

```
func maxSubArray(nums []int) int{
	// dp[i] 代表以第 i 个元素结尾的最大子数组和
    // 我们不需要开一个数组存 dp，只需要一个变量 pre 记录“前一个状态”即可
    pre:=0
    maxcot:=nums[0]
    for _,x:=range nums{
		pre=max(pre+x,x)
		maxcot=max(maxcot,pre)
	}
	return maxcot
}
```

#### 56.合并区间(微派笔试)

以数组 `intervals` 表示若干个区间的集合，其中单个区间为 `intervals[i] = [starti, endi]` 。

请你合并所有重叠的区间，并返回 *一个不重叠的区间数组，该数组需恰好覆盖输入中的所有区间* 。

```go
输入：intervals = [[1,3],[2,6],[8,10],[15,18]]
输出：[[1,6],[8,10],[15,18]]
```

排序 + 一次遍历:

```go
import (
    "sort"
)

func merge(intervals [][]int) [][]int {
    // 1. 特殊情况处理：如果是空的，直接返回
    if len(intervals) == 0 {
        return nil
    }

    // 2. 排序：按照区间的左端点 (start) 从小到大排序
    // Go 的 sort.Slice 非常方便
    sort.Slice(intervals, func(i, j int) bool {
        return intervals[i][0] < intervals[j][0]
    })
    //原数据：[2,6], [1,3], [15,18], [8,10]
	//排序后：[1,3], [2,6], [8,10], [15,18]

    // res 用于存放最终结果
    res := [][]int{}
    
    // 先把第一个区间放进去，作为“当前的待合并区间”
    res = append(res, intervals[0])
    //res (结果集): [[1, 3]]

    // 3. 从第二个区间开始遍历
    for i := 1; i < len(intervals); i++ {
        curr := intervals[i] // 当前取出的区间
        last := res[len(res)-1] // 结果集中最后一个区间（也就是我们正在维护的区间）

        // 核心判断：如果有重叠
        // 也就是：当前区间的左端点 <= 结果集中最后一个区间的右端点
        if curr[0] <= last[1] {
            // 合并操作：
            // 新的右端点 = max(原来的右端点, 当前区间的右端点)
            // 注意：不需要改左端点，因为已经排过序了，last[0] 一定是最小的
            if curr[1] > last[1] {
                res[len(res)-1][1] = curr[1] // 直接修改结果集里的那个区间
            }
        } else {
            // 如果没有重叠，直接把当前区间加入结果集
            res = append(res, curr)
        }
    }

    return res
}
```

##### sort.Slice

```
// Go 的 sort.Slice 非常方便
sort.Slice(intervals, func(i, j int) bool {
    return intervals[i][0] < intervals[j][0]
})
“嘿 Go，帮我排一下 intervals 这个数组。
至于谁排前谁排后，规则我写在这个匿名函数里了：如果你发现第 i 个元素的左端点，比第 j 个元素的左端点小，那第 i 个就应该排在前面。”
```

这是 Go 用来**给任意切片（Slice）进行自定义排序**的“万能钥匙”。

```go
sort.Slice(数据源, 比较规则函数)
```

它接收两个参数：

1. **`intervals` (数据源)**： 你要排序的那个切片（数组）。
2. **`func(i, j int) bool` (比较规则)**： 这是一个匿名函数（回调函数）。`sort` 包在内部排序时，会不断地调用这个函数来问你：“第 `i` 个元素和第 `j` 个元素，谁该排前面？”

###### 最简单的数字排队（升序 vs 降序）

```
nums := []int{9, 2, 5, 1, 7}

// 🟢 需求 1：从小到大（升序）
// 翻译：如果 nums[i] 小于 nums[j]，那 i 就在前。
sort.Slice(nums, func(i, j int) bool {
    return nums[i] < nums[j] 
})
// 结果：[1, 2, 5, 7, 9]

// 🔴 需求 2：从大到小（降序）
// 翻译：如果 nums[i] 大于 nums[j]，那 i 就在前。
sort.Slice(nums, func(i, j int) bool {
    return nums[i] > nums[j]
})
// 结果：[9, 7, 5, 2, 1]
```

###### Excel 表格排序（结构体排序）

```
type Student struct {
    Name  string
    Score int
}

students := []Student{
    {"张三", 80},
    {"李四", 95},
    {"王五", 60},
}

// 🟢 需求：按分数“从高到低”排名（也就是降序）
sort.Slice(students, func(i, j int) bool {
    // 问自己：什么时候 i 排前面？
    // 答：当 i 的分数 大于 j 的分数时
    return students[i].Score > students[j].Score
})

// 结果：李四(95) -> 张三(80) -> 王五(60)
```

###### 复杂的“多条件”排序（套娃）

```
students := []Student{
    {"Alice", 80},
    {"Bob",   90},
    {"Candy", 80}, // Candy 和 Alice 分数一样
}

sort.Slice(students, func(i, j int) bool {
    // 1. 先看分数
    if students[i].Score != students[j].Score {
        // 分数不同，谁分高谁在前
        return students[i].Score > students[j].Score
    }
    
    // 2. 代码走到这里，说明分数一样（平局）
    // 此时按名字从小到大排 (A 在 C 前面)
    return students[i].Name < students[j].Name
})

// 结果：
// 1. Bob (90) - 分最高
// 2. Alice (80) - 分数一样，A 在 C 前
// 3. Candy (80)
```

#### 189.轮转数组（字节）

给定一个整数数组 `nums`，将数组中的元素向右轮转 `k` 个位置，其中 `k` 是非负数。

```go
输入: nums = [1,2,3,4,5,6,7], k = 3
输出: [5,6,7,1,2,3,4]
```

朴素：

```
func rotate(nums []int, k int){
    k=k%len(nums)
    cnt:=[]int{}
    for i:=len(nums)-k;i<len(nums);i++{
        cnt=append(cnt,nums[i])
    }
    j:=0
    for i:=k;i<len(nums);i++{
        cnt=append(cnt,nums[j])
        j++
    }
    copy(nums,cnt)
}
```

**在不使用额外空间的情况下完成移动：**

```go
// 主函数：将数组 nums 向右轮转 k 个位置
func rotate(nums []int, k int) {
    // 1. 预处理 k
    // 如果 k 大于数组长度（比如 len=5, k=7），实际上只轮转了 7%5=2 次
    // 这一步必不可少，否则下面切片 nums[:k] 可能会越界
    k %= len(nums)
    
    // 核心思路：通过三次翻转来实现“搬运”
    // 假设 nums = [1, 2, 3, 4, 5, 6, 7], k = 3

    // 第一步：翻转整个数组
    // 目的：把原本在后面的元素（5,6,7）弄到最前面去，虽然顺序是反的
    // 结果：[7, 6, 5, 4, 3, 2, 1]
    reverse(nums)

    // 第二步：翻转前 k 个元素 (0 到 k-1)
    // 目的：把刚才弄到最前面的那段乱序的“尾巴”，纠正回正序
    // 范围：nums[:3] -> [7, 6, 5]
    // 结果：[5, 6, 7, 4, 3, 2, 1]
    reverse(nums[:k])

    // 第三步：翻转剩余的元素 (k 到最后)
    // 目的：把剩下的那段乱序的“头”，纠正回正序
    // 范围：nums[3:] -> [4, 3, 2, 1]
    // 结果：[5, 6, 7, 1, 2, 3, 4] -> 成功！
    reverse(nums[k:])
}

// 辅助函数：翻转切片（原地修改）
// 参数 a 是一个切片，在 Go 中切片是引用类型，
// 所以这里对 a 的修改会直接影响到底层数组
func reverse(a []int) {
    // i, n 是初始化语句； i < n/2 是循环条件； i++ 是步进
    // n/2 意味着只需要遍历一半即可（比如长度5，交换2次；长度4，也是交换2次）
    for i, n := 0, len(a); i < n/2; i++ {
        // Go 语言特有的“多重赋值”，可以一行代码完成交换，不需要临时变量 temp
        // a[i] 是头，a[n-1-i] 是对应的尾
        a[i], a[n-1-i] = a[n-1-i], a[i]
    }
}
```

#### 238.除了自身之外数组的乘积

给你一个整数数组 `nums`，返回 数组 `answer` ，其中 `answer[i]` 等于 `nums` 中除了 `nums[i]` 之外其余各元素的乘积 。

题目数据 **保证** 数组 `nums`之中任意元素的全部前缀元素和后缀的乘积都在 **32 位** 整数范围内。

请 **不要使用除法，**且在 `O(n)` 时间复杂度内完成此题。

```go
输入: nums = [1,2,3,4]
输出: [24,12,8,6]
```

暴力：

```go
func productExceptSelf(nums []int) []int {
	res:=[]int{}
	for i:=0;i<len(nums);i++ {
		sum:=1
		for j:=0;j<len(nums);j++{
			if j!=i{
            sum*=nums[j]
            }
		}
		res=append(res,sum)
	}
    return res
}
```

左右乘积法：

如果不让你把大家乘起来再除以你自己，那你怎么算出除去你自己之外的乘积？

答案是：**把你左边的所有数乘起来，再把你右边的所有数乘起来，最后把这两坨数乘在一起。**

```go
func productExceptSelf(nums []int) []int {
    n := len(nums)
    res := make([]int, n)

    // 1. 先算“左边所有数的乘积”
    // res[i] 此时表示 nums[i] 左侧所有元素的乘积
    res[0] = 1 // 第一个数左边没有元素，初始化为 1
    for i := 1; i < n; i++ {
        res[i] = res[i-1] * nums[i-1]
    }

    // 2. 再算“右边所有数的乘积”，并直接乘到 res 中
    // R 用来动态维护“右边所有数的乘积”
    R := 1
    for i := n - 1; i >= 0; i-- {
        res[i] = res[i] * R // 左边积 * 右边积
        R *= nums[i]        // R 累乘当前的数，为下一个位置做准备
    }

    return res
}
```

#### 41.缺失的第一个正数(字节)

给你一个未排序的整数数组 `nums` ，请你找出其中没有出现的最小的正整数。

请你实现时间复杂度为 `O(n)` 并且只使用常数级别额外空间的解决方案。

```go
输入：nums = [1,2,0]
输出：3
解释：范围 [1,2] 中的数字都在数组中。
```

暴力：使用map

```go
func firstMissingPositive(nums []int) int {
    // 1. 初始化 map
    // 建议使用 map[int]bool，因为我们只关心“在该位置有没有人”，不关心值是多少
    mp := map[int]bool{} 

    // 2. 存入数据
    for i := 0; i < len(nums); i++ {
        mp[nums[i]] = true // 标记这个数字存在
    }

    // 3. 查找缺失
    // 这里的逻辑是对的：从 1 开始找，找到 len+1
    for i := 1; i <= len(nums)+1; i++ {
        // 修正点：这里必须判断 bool 值
        if _, ok := mp[i]; !ok { // !ok 等价于 ok == false
            return i
        }
    }
    
    return 1
}
```

**Comma-ok 断言**

```
if _, ok := mp[i]; !ok { 
    return i 
}
```



核心思路：每个萝卜一个坑

想象一下，如果数组长度是 $N$，那么“缺失的第一个正数”一定在 `[1, N+1]` 这个范围内。

**策略：** 我们要把数组当作一个哈希表。

**把数值 `x` 放到下标 `x-1` 的位置上。** 

比如：数值 `1` 应该放在下标 `0`，数值 `3` 应该放在下标 `2`。

**算法流程**

1. **原地交换（归位）**： 遍历数组，如果当前数字 `nums[i]` 满足以下 **3 个条件**，就把它交换到它该去的地方（即 `nums[i]-1` 处）：
	- `nums[i]` 是个正数（`> 0`）。
	- `nums[i]` 没有超标（`<= N`）。
	- 它该去的地方（`nums[nums[i]-1]`）还没坐对人（避免死循环或重复交换）。
2. **查找缺失**： 再遍历一次数组。 如果下标 `i` 的位置存的不是 `i+1`，说明 `i+1` 这个正数缺失了。直接返回 `i+1`。
3. **兜底**： 如果所有位置都对得上（比如 `[1, 2, 3]`），说明缺的是 `N+1`。

```go
func firstMissingPositive(nums []int) int {
    n := len(nums)

    for i := 0; i < n; i++ {
        // 核心循环：只要当前数字符合条件，且没在正确的位置上，就一直交换
        // 条件解析：
        // 1. nums[i] > 0: 只关心正数
        // 2. nums[i] <= n: 只关心 1 到 n 范围内的数
        // 3. nums[nums[i]-1] != nums[i]: 目标位置上还没放对数字（防止死循环，比如 nums[i]=1, 但 nums[0] 已经是 1 了）
        for nums[i] > 0 && nums[i] <= n && nums[nums[i]-1] != nums[i] {
            // 交换到正确的位置
            // 例如：nums[i] 是 3，把它换到下标 2 的位置去
            targetIndex := nums[i] - 1
            nums[i], nums[targetIndex] = nums[targetIndex], nums[i]
        }
    }

    // 第二次遍历：找茬
    // 看哪个位置没有坐对人
    for i := 0; i < n; i++ {
        if nums[i] != i+1 {
            // 发现下标 i 处不是 i+1，说明 i+1 缺失了
            return i + 1
        }
    }

    // 如果 1 到 n 都在，那缺的就是 n+1
    return n + 1
}
```

**如果面试官问，你这写了两个for循环，时间复杂度不是n方吗？**

虽然代码里有两层循环，但这个算法的时间复杂度实际上是严格的 **$O(N)$**，而不是 $O(N^2)$

这里的核心逻辑是 **‘交换归位’**。

我们可以关注一下**‘交换’ (Swap)** 这个操作：

1. 每次执行 `swap`，至少会把**一个**数字放到了它**最终正确**的位置上（即 `nums[i]` 变成了 `i+1`）。
2. 一旦一个数字被放到了正确的位置，它在后续的遍历中就**再也不会被移动了**。
3. 数组里一共只有 $N$ 个位置，所以整个算法运行过程中，**有效的交换次数最多只有 $N$ 次**。

我们可以把整个过程的操作拆分成两部分来看：

1. **外层循环 (`i` 的移动)**：指针 `i` 从头走到尾，这部分是 **$N$ 次**操作。
2. **内层循环 (`while` 里的交换)**：虽然有时候内层会循环多次，但有时候一次都不执行。因为每个数字最多被交换一次就‘归位’了，所以**所有循环累加起来的交换总次数，绝对不会超过 $N$ 次**。

所以，总的时间复杂度 = $O(N)$（遍历）+ $O(N)$（总交换次数）= **$O(N)$**

### 矩阵

#### 输入输出

```go
输入：
var m, n int
if _,err:=fmt.Scan(&m,&n);err!=nil{
    return
}
matrix := make([][]int, m)
for i := 0; i < m; i++ {
    matrix[i] = make([]int, n)
    for j := 0; j < n; j++ {
        fmt.Scan(&matrix[i][j]) // 读每一个格子
    }
}

输出：
    for i:=0;i<m;i++{
        for j:=0;j<n;j++{
            if j==n-1 {
                fmt.Printf("%d", matrix[i][j])
            }else{
                fmt.Printf("%d ", matrix[i][j])
            }
        }
        fmt.Println()
    }
}
```

*注意：* `fmt.Scan` 适合处理一般规模的数据。如果数据量非常大（比如 $10^5$ 以上的输入），建议使用 `bufio.Scanner`，但面试中通常 `fmt.Scan` 足够。



#### 73.矩阵清零

给定一个 `*m* x *n*` 的矩阵，如果一个元素为 **0** ，则将其所在行和列的所有元素都设为 **0** 。

请使用 **[原地](http://baike.baidu.com/item/原地算法)** 算法**。**

```go
输入：matrix = [[1,1,1],[1,0,1],[1,1,1]]
输出：[[1,0,1],[0,0,0],[1,0,1]]
```

核心思路 ($O(1)$ 空间)

1. **记录首行首列状态**：

	因为我们要征用第一行和第一列来做标记，所以首先要判断**第一行和第一列本身是否包含 0**。我们用两个变量 `row0Flag` 和 `col0Flag` 来记录。

2. **利用首行首列做标记**：

	遍历矩阵的其余部分（从 `(1,1)` 开始）。

	如果发现 `matrix[i][j] == 0`，我们就把这一行的头 `matrix[i][0]` 和这一列的头 `matrix[0][j]` 设为 0。

	- 这就像是在表头和左侧索引打个勾，表示“这行/这列废了”。

3. **根据标记置零**：

	再次遍历矩阵（从 `(1,1)` 开始）。如果发现某格子的**行头**或**列头**是 0，就把该格子设为 0。

4. **处理首行首列**：

	最后，根据第 1 步记录的 `row0Flag` 和 `col0Flag`，决定是否把第一行和第一列全部填 0。

```go
package main
import(
	"fmt"
)
func setZeroes(matrix [][]int){
	if len(matrix)==0{
		return
	}
	m,n:=len(matrix),len(matrix[0])
	// 1. 检查第一行和第一列是否本来就有 0
	rowzero,colzero:=false,false
	
	//检查第一列
	for i:=0;i<m;i++ {
		if matrix[i][0]==0{
			colzero=true
			break
		}
	}
	//检查第一行
	for j:=0;j<n;j++ {
		if matrix[0][j]==0{
			rowzero=true
			break
		}
	}
	//使用第一行和第一列作为标记数组
	for i:=1;i<n;i++ {
		for j:=1;j<m;j++ {
			if matrix[i][j]==0 {
				matrix[i][0]=0
				matrix[0][j]=0
			}
        }
	}
	//根据标记，将内部元素置零
	for i:=1;i<n;i++ {
		for j:=1;j<m;j++ {
			if if matrix[i][0] == 0 || matrix[0][j] == 0 {
				matrix[i][j]=0
			}
        }
	}
	//最后处理第一列和第一行
	if colzero {
	   for i := 0; i < m; i++ {
			matrix[i][0] = 0
		}
	}
	if rowzero {
		for j := 0; j < n; j++ {
			matrix[0][j] = 0
		}
	}
}
func main(){
	var m,n int
	if _,err:=fmt.Scan(&m,&n);err!=nil{
		return
	}
	matrix:=make([][]int,m)
	for i := 0; i < m; i++ {
		matrix[i] = make([]int, n)
		for j := 0; j < n; j++ {
			fmt.Scan(&matrix[i][j])
		}
	}
    setZeroes(matrix)
    for i:=0;i<m;i++{
        for j:=0;j<n;j++{
            if j==n-1 {
                fmt.Printf("%d", matrix[i][j])
            }else{
                fmt.Printf("%d ", matrix[i][j])
            }
        }
        fmt.Println()
    }
}
```

##### 2

```go
func setZeroes(matrix [][]int)  {
    m,n:=len(matrix),len(matrix[0])
    firstRowZero:=slices.Contains(matrix[0],0)
}
```

**slices.Contains**：

这行代码 `firstRowZero := slices.Contains(matrix[0], 0)`，翻译成大白话就是：

 **“去 `matrix[0]`（第一行）这个切片里给我搜，看看里面到底有没有包含数字 `0`？”**

- **参数 1 (`matrix[0]`)**：你要搜查的地盘。这里传入的是矩阵的第一行，也就是一个 `[]int` 类型的切片。

- **参数 2 (`0`)**：你要通缉的目标。

- **返回值**：如果找到了，直接返回 `true`；

	​	        如果找遍了都没有，返回 `false`。

	​	        它会自动把结果存到 `firstRowZero` 这个布尔变量里。

```go
func setZeroes(matrix [][]int)  {
    m,n:=len(matrix),len(matrix[0])
    firstRowZero:=slices.Contains(matrix[0],0)
    for i:=1;i<m;i++ {
        for j,x:=range matrix[i] {
            if x==0 {
                matrix[i][0]=0
                matrix[0][j]=0
            }
        }
    }
    for i:=1;i<m;i++ {
        for j:=n-1;j>=0;j-- {
            if matrix[i][0]==0||matrix[0][j]==0{
                matrix[i][j]=0
            }
        }
    }
    if firstRowZero{
        clear(matrix[0])
    }
}
```

```go
    for i:=1;i<m;i++ {
        for j:=n-1;j>=0;j--{
            if matrix[i][0]==0 || matrix[0][j]==0{
                matrix[i][j]=0
            }
        }
    }
```

只要你把状态记在数组的**最左边（开头）**，那你就必须从**最右边（末尾）开始干活。这样才能保证“记录状态的那个格子”是全行最后一个**被修改的，从而保护情报在处理中途不被污染。

**`clear` (物理抹除器)**

- **作用：** 将切片或 map 中的所有元素，一键重置为该类型的**零值**。对于 `[]int` 来说，就是全部变成 0。
- **原理：** 这是 Go 1.21 新增的 **builtin (内置) 函数**（和 `len`、`make` 同级别，不需要 import！）。它在底层做了极高的内存优化，通常会调用底层的 `memclr`（内存清零）指令，速度远比你手写 `for i := range arr { arr[i] = 0 }` 要快得多！
- **为什么用它：** 一行 `clear(matrix[0])`，霸气侧漏地完成了整行置零的任务，代码洁癖患者的福音。

#### 54.螺旋矩阵（b站）

给你一个 `m` 行 `n` 列的矩阵 `matrix` ，请按照 **顺时针螺旋顺序** ，返回矩阵中的所有元素。

```go
输入：matrix = [[1,2,3],[4,5,6],[7,8,9]]
输出：[1,2,3,6,9,8,7,4,5]
```

##### 灵神

```go
//右 ➡️ 下 ➡️ 左 ➡️ 上。
var dirs =[4][2]int{{0,1},{1,0},{0,-1},{-1,0}}

func spiralOrder(matrix [][]int)[]int{
	m,n:=len(matrix),len(matrix[0])
    ans:=[]int{}
    i,j:=0,-1
    t:=m*n
    for d:=0;len(ans)<t;d=(d+1)%4 {
        for range n{
            i+=dirs[d][0]
            j+=dirs[d][1]
            ans=append(ans,matrix[i][j])
        }
        n,m=m-1,n
    }
    return ans
}
```



##### 核心思路：四个边界（墙）

想象你在控制一条贪吃蛇，在一个 $M \times N$ 的格子里游走。

你只能顺时针走：**右 $\rightarrow$ 下 $\rightarrow$ 左 $\rightarrow$ 上**。

为了不撞墙，也不走回头路，我们需要定义四个 **“墙壁” (Boundaries)**：

1. **`top` (上墙)**：当前还没遍历的最上面一行。初始为 `0`。
2. **`bottom` (下墙)**：当前还没遍历的最下面一行。初始为 `m-1`。
3. **`left` (左墙)**：当前还没遍历的最左边一列。初始为 `0`。
4. **`right` (右墙)**：当前还没遍历的最右边一列。初始为 `n-1`。

 **算法流程：不断“缩圈”**

这是一个循环过程，每走完一条边，对应的墙就要 **往里缩一步**。

1. **向右走**：从 `left` 走到 `right`。
	- 走完这行，**上墙下移** (`top++`)。
	- **检查**：如果 `top > bottom`，说明上下墙撞上了，没路了，结束！
2. **向下走**：从 `top` 走到 `bottom`。
	- 走完这列，**右墙左移** (`right--`)。
	- **检查**：如果 `left > right`，左右墙撞上了，结束！
3. **向左走**：从 `right` 走到 `left`。
	- 走完这行，**下墙上移** (`bottom--`)。
	- **检查**：如果 `top > bottom`，结束！
4. **向上走**：从 `bottom` 走到 `top`。
	- 走完这列，**左墙右移** (`left++`)。
	- **检查**：如果 `left > right`，结束！

```go
package main
import(
	"fmt"
)
func spiralOrder(matrix [][]int) []int {
	if len(matrix)==0 {
	return []int{}
}
    m,n:=len(matrix),len(matrix[0])
    top,bottom:=0,m-1
    left,right:=0,n-1
    res:=[]int{}
    for {
        for i:=left;i<=right;i++{
            res=append(res,matrix[top][i])
        }
        top++
        if top >bottom {break}
        for i:=top;i<=bottom;i++{
            res=append(res,matrix[i][right])
        }
        right--
        if left>right {break}
        for i:=right;i>=left;i-- {
            res=append(res,matrix[bottom][i])
        }
        bottom--
        if top >bottom {break}
        for i:=bottom;i>=top;i-- {
            res=append(res,matrix[i][left])
        }
        left++
        if left > right { break }
    }
    return res
}
func main() {
	// --- ACM 输入模版 ---
	var m, n int
	// 读取行数和列数
	if _, err := fmt.Scan(&m, &n); err != nil {
		return
	}

	// 初始化矩阵
	matrix := make([][]int, m)
	for i := 0; i < m; i++ {
		matrix[i] = make([]int, n)
		for j := 0; j < n; j++ {
			fmt.Scan(&matrix[i][j])
		}
	}

	// --- 调用算法 ---
	result := spiralOrder(matrix)

	// --- ACM 输出模版 ---
	for i, num := range result {
		if i == len(result)-1 {
			fmt.Printf("%d", num) // 最后一个数后面不加空格
		} else {
			fmt.Printf("%d ", num)
		}
	}
	fmt.Println() // 结尾换行
}
```

#### 48.旋转图像( 字节)

给定一个 *n* × *n* 的二维矩阵 `matrix` 表示一个图像。请你将图像顺时针旋转 90 度。

你必须在**[ 原地](https://baike.baidu.com/item/原地算法)** 旋转图像，这意味着你需要直接修改输入的二维矩阵。

**请不要** 使用另一个矩阵来旋转图像。

```go
输入：matrix = [[1,2,3],[4,5,6],[7,8,9]]
输出：[[7,4,1],[8,5,2],[9,6,3]]
```

**顺时针旋转 90 度 = 先转置 (Transpose) + 再水平翻转 (Reflect/Reverse Rows)**

转置：沿着主对角线交换元素

```go
package main
import(
	"fmt"
)
func rotate(matrix [][]int)  {
    m:=len(matrix)
    for i:=0;i<m;i++ {
        for j:=i+1;j<m;j++ {
            matrix[i][j],matrix[j][i]=matrix[j][i],matrix[i][j]
        }
    }
    for i:=0;i<m;i++ {
        left,right:=0,m-1
        for left<right{
            matrix[i][right],matrix[i][left]=matrix[i][left],matrix[i][right]
            right--
            left++
        }
    }
}
func main() {
	// --- 1. ACM 输入处理 ---
	var n int
	// 读取矩阵大小 n
	if _, err := fmt.Scan(&n); err != nil {
		return
	}

	// 初始化 n x n 矩阵
	matrix := make([][]int, n)
	for i := 0; i < n; i++ {
		matrix[i] = make([]int, n)
		for j := 0; j < n; j++ {
			fmt.Scan(&matrix[i][j])
		}
	}

	// --- 2. 调用算法 ---
	rotate(matrix)

	// --- 3. ACM 输出处理 ---
	for i := 0; i < n; i++ {
		for j := 0; j < n; j++ {
			if j == n-1 {
				// 行末无空格
				fmt.Printf("%d", matrix[i][j])
			} else {
				// 中间有空格
				fmt.Printf("%d ", matrix[i][j])
			}
		}
		// 每一行输出完要换行
		fmt.Println()
	}
}
```

#### 240.搜索矩阵Ⅱ（字节）

编写一个高效的算法来搜索 `m x n` 矩阵 `matrix` 中的一个目标值 `target` 。

该矩阵具有以下特性：

- 每行的元素从左到右升序排列。
- 每列的元素从上到下升序排列。

##### 核心思路：从右上角（或左下角）开始找

最经典的解法是把这个矩阵想象成一个 **二叉搜索树 (BST)**。

我们选择矩阵的 **右上角 `(0, n-1)`** 作为起点：

- **向左看**：元素变小（因为行是升序的）。
- **向下看**：元素变大（因为列是升序的）。

**算法流程：**

1. 初始化指针 `row = 0`, `col = n - 1`（即右上角）。
2. 当指针没有越界时，比较 `matrix[row][col]` 和 `target`：
	- 如果 **相等**：找到了，返回 `true`。
	- 如果 **当前值 > target**：说明当前列下面的数肯定更大，更不可能是 target，所以我们要 **向左移** (`col--`) 去找小一点的数。
	- 如果 **当前值 < target**：说明当前行左边的数肯定更小，更不可能是 target，所以我们要 **向下移** (`row++`) 去找大一点的数。
3. 如果走出边界还没找到，返回 `false`。

```go
package main
import(
	"fmt"
)

func searchMatrix(matrix [][]int, target int) bool {
    m,n:=len(matrix),len(matrix[0])
    if matrix[0][0]>target || matrix[m-1][n-1]<target {
        return false
    }
    row,col:=0,n-1
    for row<m &&col>=0 {
        current:=matrix[row][col]
        
        if current==target {
            return true
        }else if current>target{
            col--
        }else if current<target{
            row++
        }
    }
    return false
}


func main() {
	// --- ACM 输入处理 ---
	var m, n, target int

	// 1. 读取行数 m, 列数 n
	if _, err := fmt.Scan(&m, &n); err != nil {
		return
	}
    
    // 2. 读取目标值 target 
    // (注意：有些题目是先给矩阵再给target，这里假设先给target，具体视题目而定)
    fmt.Scan(&target)

	// 3. 读取矩阵内容
	matrix := make([][]int, m)
	for i := 0; i < m; i++ {
		matrix[i] = make([]int, n)
		for j := 0; j < n; j++ {
			fmt.Scan(&matrix[i][j])
		}
	}

	// --- 调用算法 ---
	result := searchMatrix(matrix, target)

	// --- ACM 输出处理 ---
	if result {
		fmt.Println("true")
	} else {
		fmt.Println("false")
	}
}
```

### 链表

#### 基础

基础结构体定义

```go
type ListNode struct {
    Val  int
    Next *ListNode
}
```

两个核心辅助函数

你需要两个 helper 函数：

1. **数组 -> 链表**：把输入的 `[]int` 变成链表，方便你测试算法。
2. **链表 -> 打印**：把处理完的链表打印出来，验证结果。

(1) 构建链表 (Build List)

```go
func createLinkedList(nums []int) *ListNode {
    if len(nums) == 0 {
        return nil
    }
    // 使用 dummy 节点简化代码
    dummy := &ListNode{}
    curr := dummy
    for _, num := range nums {
        curr.Next = &ListNode{Val: num}
        curr = curr.Next
    }
    return dummy.Next
}
```

(2) 打印链表 (Print List)

```go
func printLinkedList(head *ListNode) {
    curr := head
    for curr != nil {
        // ACM 模式通常要求数字间有空格，最后一个数字后无空格
        // 这里为了简便，直接打印带空格的，最后会多一个空格
        // 严格模式下可以用 slice 收集后再 strings.Join
        fmt.Printf("%d ", curr.Val)
        curr = curr.Next
    }
    fmt.Println() // 换行
}
```

常用技巧

A. 哨兵节点 (Dummy Head) —— **最重要！**

**场景**：当头节点 `head` 可能会被删除、修改，或者链表可能为空时。

```
dummy := &ListNode{Next: head}
// ... 操作 ...
return dummy.Next
```

**作用**：避免处理 `if head == nil` 这种恶心的边界情况。

B. 快慢指针 (Fast & Slow Pointers)

**场景**：找中点、判断环、找倒数第 K 个节点。

```
slow, fast := head, head
for fast != nil && fast.Next != nil {
    slow = slow.Next      // 走一步
    fast = fast.Next.Next // 走两步
}
```

C. 反转链表 (Reverse)

**场景**：很多复杂题目的子步骤（如 K 个一组翻转）。

```
var pre *ListNode
cur := head
for cur != nil {
    next := cur.Next
    cur.Next = pre
    pre = cur
    cur = next
}
// pre 变成了新的头
```

#### 160.相交链表(tx)

给你两个单链表的头节点 `headA` 和 `headB` ，请你找出并返回两个单链表相交的起始节点。

如果两个链表不存在相交节点，返回 `null` 。

题目数据 **保证** 整个链式结构中不存在环。

**注意**，函数返回结果后，链表必须 **保持其原始结构** 。

**自定义评测：**

**评测系统** 的输入如下（你设计的程序 **不适用** 此输入）：

- `intersectVal` - 相交的起始节点的值。如果不存在相交节点，这一值为 `0`
- `listA` - 第一个链表
- `listB` - 第二个链表
- `skipA` - 在 `listA` 中（从头节点开始）跳到交叉节点的节点数
- `skipB` - 在 `listB` 中（从头节点开始）跳到交叉节点的节点数

评测系统将根据这些输入创建链式数据结构，并将两个头节点 `headA` 和 `headB` 传递给你的程序。

如果程序能够正确返回相交节点，那么你的解决方案将被 **视作正确答案** 。

具体算法如下：

1. 初始化两个指针 p=headA, q=headB。

2. 不断循环，直到 p=q。

3. 每次循环，p 和 q 各向后走一步。

	具体来说，如果 p 不是空节点，那么更新 p 为 p.next，否则更新 p 为 headB；

	如果 q 不是空节点，那么更新 q 为 q.next，否则更新 q 为 headA。

4. 循环结束时，如果两条链表相交，那么此时 p 和 q 都在相交的起始节点处，返回 p；

	如果两条链表不相交，那么 p 和 q 都走到空节点，所以也可以返回 p，即空节点。

迭代法和递归法：

```go
package main

import (
	"fmt"
)

// Definition for singly-linked list.
type ListNode struct {
	Val  int
	Next *ListNode
}

// ---------------------------------------------------------	
// 解法一：迭代法 (Iterative) - 推荐，空间 O(1)
// ---------------------------------------------------------
// 思路：pA 走完 A 走 B，pB 走完 B 走 A，最终在交点或 nil 处相遇
func getIntersectionNodeIterative(headA, headB *ListNode) *ListNode {
	if headA == nil || headB == nil {
		return nil
	}
	pA, pB := headA, headB
	
	// 只要不相等就继续走
	for pA != pB {
		// pA 逻辑
		if pA == nil {
			pA = headB
		} else {
			pA = pA.Next
		}
		
		// pB 逻辑
		if pB == nil {
			pB = headA
		} else {
			pB = pB.Next
		}
	}
	return pA
}

// ---------------------------------------------------------
// 解法二：递归法 (Recursive)
// ---------------------------------------------------------
// 思路：完全模拟迭代法的状态转移。
// 状态：当前 pA 指向哪里，当前 pB 指向哪里。
// 注意：Go 的递归栈深度有限，链表太长会 Stack Overflow，仅用于算法逻辑训练。
func getIntersectionNodeRecursive(headA, headB *ListNode) *ListNode {
	if headA == nil || headB == nil {
		return nil
	}
	return recursiveHelper(headA, headB, headA, headB)
}

func recursiveHelper(pA, pB, originalHeadA, originalHeadB *ListNode) *ListNode {
	// 1. Base Case: 相遇了（要么是交点，要么都是 nil）
	if pA == pB {
		return pA
	}

	// 2. 计算 pA 的下一步
	var nextA *ListNode
	if pA == nil {
		nextA = originalHeadB // A 走完了，切到 B 头
	} else {
		nextA = pA.Next
	}

	// 3. 计算 pB 的下一步
	var nextB *ListNode
	if pB == nil {
		nextB = originalHeadA // B 走完了，切到 A 头
	} else {
		nextB = pB.Next
	}

	// 4. 递归调用
	return recursiveHelper(nextA, nextB, originalHeadA, originalHeadB)
}

// ---------------------------------------------------------
// 辅助函数：构建 ACM 输入需要的 Y 型链表
// ---------------------------------------------------------
func buildYLinkedList(intersectVal int, listA, listB []int, skipA, skipB int) (*ListNode, *ListNode) {
	// 如果没有数据
	if len(listA) == 0 || len(listB) == 0 {
		return nil, nil
	}

	// 1. 先把 listA 完整构建出来，并存下所有节点的指针
	// 因为我们要让 B 指向 A 中的某个节点，必须有 A 节点的内存地址
	nodesA := make([]*ListNode, len(listA))
	dummyA := &ListNode{}
	currA := dummyA
	
	for i, val := range listA {
		node := &ListNode{Val: val}
		currA.Next = node
		currA = node
		nodesA[i] = node // 存指针
	}
	headA := dummyA.Next

	// 2. 构建 listB
	dummyB := &ListNode{}
	currB := dummyB

	// 2.1 构建 B 独有的部分 (即 skipB 之前的节点)
	// 注意：题目保证 listB 的后半段数值和 listA 后半段一致，
	// 但我们需要在 skipB 处物理连接到 A 的节点上。
	for i := 0; i < skipB; i++ {
		node := &ListNode{Val: listB[i]}
		currB.Next = node
		currB = node
	}

	// 2.2 处理相交
	if intersectVal != 0 {
		// 这里的核心是：B 的 Next 直接指向 A 中已经创建好的节点
		if skipA < len(nodesA) {
			currB.Next = nodesA[skipA] 
		}
	} else {
		// 如果不相交，就把 B 剩下的部分独立构建完
		for i := skipB; i < len(listB); i++ {
			node := &ListNode{Val: listB[i]}
			currB.Next = node
			currB = node
		}
	}
	headB := dummyB.Next

	return headA, headB
}

// ---------------------------------------------------------
// Main 函数
// ---------------------------------------------------------
func main() {
	var intersectVal, n, m, skipA, skipB int

	// 1. 读取相交值
	fmt.Scan(&intersectVal)

	// 2. 读取 ListA
	fmt.Scan(&n)
	listA := make([]int, n)
	for i := 0; i < n; i++ {
		fmt.Scan(&listA[i])
	}

	// 3. 读取 ListB
	fmt.Scan(&m)
	listB := make([]int, m)
	for i := 0; i < m; i++ {
		fmt.Scan(&listB[i])
	}

	// 4. 读取跳过的步数
	fmt.Scan(&skipA, &skipB)

	// 5. 构建链表
	headA, headB := buildYLinkedList(intersectVal, listA, listB, skipA, skipB)

	// 6. 执行算法 (这里演示两次调用，实际比赛选一种)
	fmt.Println("--- Iterative Result ---")
	resIter := getIntersectionNodeIterative(headA, headB)
	if resIter != nil {
		fmt.Printf("Intersected at '%d'\n", resIter.Val)
	} else {
		fmt.Println("null")
	}

	fmt.Println("--- Recursive Result ---")
	resRecur := getIntersectionNodeRecursive(headA, headB)
	if resRecur != nil {
		fmt.Printf("Intersected at '%d'\n", resRecur.Val)
	} else {
		fmt.Println("null")
	}
}
```

#### 206.反转链表(字节、xhs)

给你单链表的头节点 `head` ，请你反转链表，并返回反转后的链表。

```go
输入：head = [1,2,3,4,5]
输出：[5,4,3,2,1]
```

解法一：迭代法（双指针 / 三指针）—— 最推荐、最常用

```go
package main
import (
	"fmt"
)
type ListNode struct{
    val int
    next *ListNode
}

func reverseListIterative(head *ListNode) *ListNode {
    var prev *ListNode // prev 	初始为 nil
    curr := head       // curr 从头节点开始
    
    for curr != nil {
        next := curr.Next // 1. 暂存原链表的下一个节点
        curr.Next = prev  // 2. 反转指针，当前节点指向前一个节点
        prev = curr       // 3. prev 指针前移
        curr = next       // 4. curr 指针前移
    }
    
    // 遍历结束时，curr 指向 nil，prev 正好指向原链表的最后一个节点（新链表的头节点）
    return prev 
}

func main(){
    var n int
    if _,err:=fmt.Scan(&n);err!=nil||n!==0{
        return
    }
    dummy:=&ListNode{}
    cur:=dummy
    for i:=0;i<n;i++ {
        var val int
        fmt.Scan(&val)
        cur.next=&ListNode(val:val)
        cur=cur.next
    }
    head:=dummy.next
    newHead := reverseList(head)
    for newHead!=nil{
        fmt.Printf("%d",newHead.val)
        newHead = newHead.Next
    }
    fmt.Println()
}
```

解法二：递归法 —— 考验逻辑思维的利器

递归的核心思想是：**“假设后面的链表已经反转好了，我当前节点该怎么办？”**

假设链表是 `1 -> 2 -> 3 -> 4 -> 5`，我们递归到节点 `1` 时：

1. 先调用递归，把 `2 -> 3 -> 4 -> 5` 反转变成 `5 -> 4 -> 3 -> 2`。
2. 此时，节点 `2` 的 `Next` 是 `nil`，而节点 `1` 的 `Next` 依然指向节点 `2`。
3. 我们只需要让节点 `2` 指向节点 `1`：`head.Next.Next = head`。
4. 然后让节点 `1` 指向 `nil`（防止成环）：`head.Next = nil`。

```go
func reverseListRecursive(head *ListNode) *ListNode {
	// 1. 递归终止条件：
    // 如果链表为空，或者只有一个节点，直接返回它自己
    if head == nil || head.Next == nil {
        return head
    }
    // 2. 递：把大问题拆成小问题，去反转后面的链表
    // newHead 会一直传递原本的最后一个节点（比如 5），它将是新链表的头
    newHead := reverseListRecursive(head.Next)
    // 3. 归：处理当前节点和下一个节点的关系
    // 此时 head 是 1，head.Next 是 2。我们要让 2 指向 1
    head.Next.Next = head
    // 断开 1 指向 2 的指针，防止产生环
    head.Next = nil
    // 返回新的头节点
    return newHead
}
```

#### 234.回文链表(美团)

给你一个单链表的头节点 `head` ，请你判断该链表是否为回文链表。如果是，返回 `true` ；否则，返回 `false` 。

```go
输入：head = [1,2,2,1]
输出：true
```

**核心思路：把后半段掰过来对比**

想象把一根棍子从中间折断，然后把后半段反转一下，跟前半段比对，看看是不是一模一样。

1. **找中点**：用快慢指针（`fast` 走两步，`slow` 走一步）。`fast` 走到头时，`slow` 刚好在中间。
2. **反转后半部分**：从 `slow` 开始，把后面的链表就地反转。（这就是你上一题刚练过的代码！）
3. **对比**：一个指针从原始头节点 `head` 出发，另一个指针从反转后的尾部出发，挨个对比值。

```go
package main

import (
	"fmt"
)

// 1. 定义结构体
type ListNode struct {
	Val  int
	Next *ListNode
}

// 2. 核心算法：判断是否为回文链表
func isPalindrome(head *ListNode) bool {
	if head == nil || head.Next == nil {
		return true
	}

	// 第一步：快慢指针找中点
	slow, fast := head, head
	for fast != nil && fast.Next != nil {
		slow = slow.Next
		fast = fast.Next.Next
	}

	// 第二步：反转后半部分链表 (此时 slow 是中点)
	var prev *ListNode
	curr := slow
	for curr != nil {
		next := curr.Next // 暂存下一个
		curr.Next = prev  // 回头指
		prev = curr       // 往前挪
		curr = next       // 往前挪
	}

	// 第三步：双指针对比验证
	// prev 现在是反转后的后半段的“头节点”
	left, right := head, prev
	for right != nil { // 注意：后半段可能比前半段短一个节点（链表长度为奇数时），所以以 right != nil 为准
		if left.Val != right.Val {
			return false // 只要有一个不一样，就不是回文
		}
		left = left.Next
		right = right.Next
	}

	return true
}

func main() {
	var n int
	// 读取节点个数
	if _, err := fmt.Scan(&n); err != nil || n == 0 {
		return
	}

	// 极简构建链表
	dummy := &ListNode{}
	curr := dummy
	for i := 0; i < n; i++ {
		var val int
		fmt.Scan(&val)
		curr.Next = &ListNode{Val: val}
		curr = curr.Next
	}

	// 执行算法并输出
	result := isPalindrome(dummy.Next)
	if result {
		fmt.Println("true")
	} else {
		fmt.Println("false")
	}
}
```

#### 142.环形链表Ⅱ(字节)

给定一个链表的头节点  `head` ，返回链表开始入环的第一个节点。 *如果链表无环，则返回 `null`。*

如果链表中有某个节点，可以通过连续跟踪 `next` 指针再次到达，则链表中存在环。 为了表示给定链表中的环，评测系统内部使用整数 `pos` 来表示链表尾连接到链表中的位置（**索引从 0 开始**）。如果 `pos` 是 `-1`，则在该链表中没有环。**注意：`pos` 不作为参数进行传递**，仅仅是为了标识链表的实际情况。

**不允许修改** 链表。

```go
输入：head = [3,2,0,-4], pos = 1
输出：返回索引为 1 的链表节点
解释：链表中有一个环，其尾部连接到第二个节点。
```

1. 核心数学推导（一定要理解背诵）

我们依然让 `slow` 一次走 1 步，`fast` 一次走 2 步。假设它们在环内的某一点相遇了。

我们定义三段距离：

- **$a$**：从链表头节点到**环入口**的距离。

- **$b$**：从环入口到**相遇点**的距离。

- **$c$**：从相遇点继续走到**环入口**的剩余距离。

	*(注意：环的总长度就是 $b + c$)*

**开始推导：**

1. 相遇时，慢指针走过的距离是：$a + b$

2. 快指针走过的距离是慢指针的两倍：$2(a + b)$

3. 同时，快指针比慢指针多走了 $n$ 圈的环（假设 $n \ge 1$），所以快指针走过的距离也是：$a + n(b + c) + b$

4. 我们把这两个式子等价起来：

	​									$$2(a + b) = a + n(b + c) + b$$

5. 稍微变形一下，提取出一个环的长度 $(b+c)$：

	​									$$a = (n - 1)(b + c) + c$$

	从头节点走到入口的距离（$a$），刚好等于从相遇点绕环几圈后，再走到入口的距离（$c$）。

根据上面的数学推导，我们的算法分为两个阶段：

- **阶段一（判断是否有环）**：

	 `fast` 和 `slow` 从头开始跑。如果没相遇就遇到了 `nil`，说明无环，直接返回 `nil`。如果相遇了，进入阶段二。

- **阶段二（寻找入口处）**： 

	把其中一个指针（比如 `fast`）**重新放到头节点 `head`**，另一个指针（`slow`）留在**相遇点**。 

	然后，两个指针**每次都只走 1 步**。 当它们**再次相遇**时，相遇的那个节点，就是**环的入口节点**！

```go
package main

import (
	"fmt"
)

type ListNode struct {
	Val  int
	Next *ListNode
}

// 核心算法：返回环的入口节点
func detectCycle(head *ListNode) *ListNode {
	if head == nil || head.Next == nil {
		return nil
	}

	slow, fast := head, head

	// 阶段一：判断是否有环，并找到第一次相遇点
	hasCycle := false
	for fast != nil && fast.Next != nil {
		slow = slow.Next
		fast = fast.Next.Next
		if slow == fast {
			hasCycle = true
			break // 找到相遇点，跳出循环
		}
	}

	// 如果没有环，直接返回 nil
	if !hasCycle {
		return nil
	}

	// 阶段二：找环的入口
	// 将其中一个指针重新指向头节点
	fast = head
	// 此时两个指针速度一样，每次各走一步
	for fast != slow {
		fast = fast.Next
		slow = slow.Next
	}

	// 再次相遇点即为环的入口
	return fast
}

func main() {
	var n int
	// 读取节点个数
	if _, err := fmt.Scan(&n); err != nil || n == 0 {
		return
	}

	// 1. 构建所有节点 (用切片存下来方便等下连环)
	nodes := make([]*ListNode, n)
	for i := 0; i < n; i++ {
		var val int
		fmt.Scan(&val)
		nodes[i] = &ListNode{Val: val}
	}

	// 2. 连成普通链表
	for i := 0; i < n-1; i++ {
		nodes[i].Next = nodes[i+1]
	}

	// 3. 读取 pos 并制造环
	var pos int
	fmt.Scan(&pos)
	if pos >= 0 && pos < n {
		nodes[n-1].Next = nodes[pos] // 尾部指向 pos 索引所在的节点
	}

	// 4. 调用核心算法
	var head *ListNode
	if n > 0 {
		head = nodes[0]
	}
	
	entranceNode := detectCycle(head)

	// 5. 打印结果
	if entranceNode != nil {
		fmt.Printf("tail connects to node with value %d\n", entranceNode.Val)
	} else {
		fmt.Println("null")
	}
}
```

#### 21.合并两个有序链表(字节)

将两个升序链表合并为一个新的 **升序** 链表并返回。新链表是通过拼接给定的两个链表的所有节点组成的。 

```go
输入：l1 = [1,2,4], l2 = [1,3,4]
输出：[1,1,2,3,4,4]
```



```go
func mergeTwoLists(list1 *ListNode, list2 *ListNode) *ListNode {
    dummy := &ListNode{}
	curr := dummy 
    p,q:=list1,list2
    for p != nil && q != nil {
		if p.Val <= q.Val {
			curr.Next = p // 把 p 挂上去
			p = p.Next    // p 往前走
		} else {
			curr.Next = q // 把 q 挂上去
			q = q.Next    // q 往前走
		}
		curr = curr.Next // 游标往前走，准备挂下一个
	}
    if p != nil {
		curr.Next = p
	}
	if q != nil {
		curr.Next = q
	}
    return dummy.Next
}
```

#### 19.删除链表的倒数第N个节点(腾讯)

给你一个链表，删除链表的倒数第 `n` 个结点，并且返回链表的头结点。

#### 25.k个一组翻转链表

给你链表的头节点 `head` ，每 `k` 个节点一组进行翻转，请你返回修改后的链表。

`k` 是一个正整数，它的值小于或等于链表的长度。如果节点总数不是 `k` 的整数倍，那么请将最后剩余的节点保持原有顺序。

你不能只是单纯的改变节点内部的值，而是需要实际进行节点交换。

#### 146.LRU✨✨✨✨✨

设计并实现一个满足 [LRU (最近最少使用) 缓存](https://baike.baidu.com/item/LRU) 约束的数据结构。

实现 `LRUCache` 类：

- `LRUCache(int capacity)` 以 **正整数** 作为容量 `capacity` 初始化 LRU 缓存
- `int get(int key)` 如果关键字 `key` 存在于缓存中，则返回关键字的值，否则返回 `-1` 。
- `void put(int key, int value)` 如果关键字 `key` 已经存在，则变更其数据值 `value` ；如果不存在，则向缓存中插入该组 `key-value` 。如果插入操作导致关键字数量超过 `capacity` ，则应该 **逐出** 最久未使用的关键字。

函数 `get` 和 `put` 必须以 `O(1)` 的平均时间复杂度运行。



```go
type node struct{
    value,key int
    pre,next  *node
}
type LRUCache struct{
    capacity int
    dummy    *node
    keytonode map[int]*node
}
func Constructor(capacity int)LRUCache{
    dummy:=&Node{}
    dummy.pre=dummy
    dummy.next=dummy
    return LRUCache{
        capacity:capacity,
        dummy:dummy,
        keytonode:map[int]*node{},
    }
}
func(c *LRUCache) remove(x *node){
    x.pre.next=x.next
    x.next.pre=x.pre
}
func(c *LRUCache)pushFront(x *node){
    x.pre=c.dummy
    x.next=c.dummy.next
 
    x.pre.next=x
    x.next.pre=x
}
func(c *LRUCache)getNode(key int)*node{
    node:=c.keytonode[key]
    if node==nil{
		return nil
    }
    c.remove(node)
    c.pushFront(node)
    return node
}
func(c *LRUCache)Get(key int)int{
    node:=c.getNode(key)
    if node==nil{
        return -1
    }
    return node.value
}
func(c *LRUCache)Put(key int,value int){
    node:=c.getNode(key)
    if node!=nil{
        node.value=value
        return
    }
    node=&node{key:key,value:value}
    c.keytonode[key]=node
    c.pushFront(node)
    if len(c.keytonode)>c.capacity{
        backnode:=c.dummy.pre
        delete(c.ketonode,backnode.key)
        c.remove(backnode)
    }
    
}
```

------

#### 带过期时间的 LRU 缓存 (LRU Cache with TTL)

**题目描述：**

请你设计并实现一个支持过期时间的 LRU 缓存。

实现 `LRUCacheWithTTL` 类：

- `Constructor(int capacity)`：以正整数作为容量 `capacity` 初始化 LRU 缓存。
- `int get(int key)`：如果关键字 `key` 存在于缓存中**且未过期**，则返回关键字的值，否则返回 `-1`。
- `void put(int key, int value, int ttl)`：向缓存中插入或更新 `key-value`，并设置其存活时间 `ttl`（毫秒）。
	- 如果插入导致超出 `capacity`，则应该逐出最久未使用的关键字。
	- **要求**：`get` 和 `put` 操作的时间复杂度必须保持 $O(1)$。

核心解题思路：惰性删除 (Lazy Deletion)

面试官考这道题，最想听到的核心词汇就是**“惰性删除”**。

如果你搞一个后台定时任务，每秒钟去扫描一遍所有的节点看有没有过期，这就变成了 $O(N)$ 的时间复杂度，不仅消耗 CPU，还会引发锁竞争。

**真正的 $O(1)$ 做法是“惰性删除”**：

1. **节点增加时间戳**：在 `Node` 结构体里加一个 `expireAt` 字段，存下它“该死”的绝对时间点（当前时间 + TTL）。
2. **Get 时再检查**：每次调用 `Get(key)` 时，先看一下当前时间是不是已经超过了 `expireAt`。如果超了，**当场把它删掉**，假装它本来就不存在，返回 `-1`。
3. **Put 的正常淘汰**：如果容量满了，依然淘汰双向链表尾部（最久未使用）的节点。

```go
// 1. 结构体改造：增加 expireAt
type Node struct {
	key, value int
	expireAt   int64 // 记录过期的绝对时间戳 (毫秒)
	prev, next *Node
}

type LRUCacheWithTTL struct {
	capacity  int
	dummy     *Node
	keyToNode map[int]*Node
}

func Constructor(capacity int) LRUCacheWithTTL {
	dummy := &Node{}
	dummy.prev = dummy
	dummy.next = dummy
	return LRUCacheWithTTL{
		capacity:  capacity,
		dummy:     dummy,
		keyToNode: make(map[int]*Node),
	}
}

// 基础操作：从链表抽出节点
func (c *LRUCacheWithTTL) remove(x *Node) {
	x.prev.next = x.next
	x.next.prev = x.prev
}

// 基础操作：放到最前面
func (c *LRUCacheWithTTL) pushFront(x *Node) {
	x.prev = c.dummy
	x.next = c.dummy.next
	x.prev.next = x
	x.next.prev = x
}

// 2. Get 的改造：增加过期判断
func (c *LRUCacheWithTTL) Get(key int) int {
	node := c.keyToNode[key]
	if node == nil {
		return -1
	}

	// 【核心逻辑：惰性删除】
	// 获取当前时间的毫秒时间戳
	now := time.Now().UnixMilli()
	if now > node.expireAt {
		// 已经过期了，当场销毁
		c.remove(node)
		delete(c.keyToNode, key)
		return -1 // 告诉用户没找到
	}

	// 没过期，按正常 LRU 逻辑提到前面
	c.remove(node)
	c.pushFront(node)
	return node.value
}

// 3. Put 的改造：写入过期时间
func (c *LRUCacheWithTTL) Put(key, value, ttl int) {
	// 计算绝对过期时间
	now := time.Now().UnixMilli()
	expireTime := now + int64(ttl)

	node := c.keyToNode[key]

	if node != nil {
		// 存在，更新值和过期时间，并提到最前面
		node.value = value
		node.expireAt = expireTime
		c.remove(node)
		c.pushFront(node)
		return
	}

	// 不存在，创建新节点
	newNode := &Node{
		key:      key,
		value:    value,
		expireAt: expireTime,
	}
	c.keyToNode[key] = newNode
	c.pushFront(newNode)

	// 如果超出容量，淘汰最久未使用的尾部节点
	if len(c.keyToNode) > c.capacity {
		backNode := c.dummy.prev
		delete(c.keyToNode, backNode.key)
		c.remove(backNode)
	}
}
```

### 栈

#### 155.最小栈

设计一个支持 `push` ，`pop` ，`top` 操作，并能在常数时间内检索到最小元素的栈。

实现 `MinStack` 类:

- `MinStack()` 初始化堆栈对象。
- `void push(int val)` 将元素val推入堆栈。
- `void pop()` 删除堆栈顶部的元素。
- `int top()` 获取堆栈顶部的元素。
- `int getMin()` 获取堆栈中的最小元素。

把 nums 视作栈，本题相当于在 nums 的末尾动态地添加/删除元素。

栈中除了保存添加的元素，还**保存前缀最小值**。（栈中保存的是 pair）
添加元素：

​	设当前栈的大小是 n。

​	添加元素 val 后，额外维护 preMin[n]=min(preMin[n−1],val)，其中 preMin[n−1] 是添加 val 之前，栈顶保存的前缀最小值。
删除元素：

​	弹出栈顶即可。



面试官您好。这道题要求我们在 `O(1)` 的时间复杂度内获取栈内的最小值。

如果每次都去遍历栈，时间复杂度是 `O(N)`，显然不行。

所以核心思想是**‘空间换时间’**：我们需要把每一个元素入栈时，**那一瞬间的‘历史最小值’给缓存下来**。

我打算用一个单一的栈来实现。

栈里存的不是单纯的数字，而是一个自定义的结构体（Pair）。

这个 Pair 既保存当前压入的值 `val`，也保存从栈底到当前位置的‘**前缀最小值**’ `preMin`。

```go
type pair sturct{
	val,preMin int
}
type MinStack []pair
```

首先，我定义一个 `pair` 结构体。

​	`val` 存真实数据，`preMin` 存到当前元素为止的最小值。 

然后，我的 `MinStack` 底层直接使用 Go 的切片（Slice）来实现。”



```go
import "math"

func Constructor() MinStack {
    // 放入一个哨兵节点
    return MinStack{{0, math.MaxInt}}
}
```

在初始化这里，我做了一个小优化。

通常我们在 Push 第一个元素，或者取 Min 的时候，都需要判断栈是否为空，这会让代码里充满 `if len(st) == 0`。

为了消灭这些边界判断，我在栈底垫了一个**‘哨兵节点’ (Dummy Node)**。

它的值不重要（我随便写了 0），但它的 `preMin` 我设为了 `math.MaxInt`（正无穷）。

这样一来，任何真实数据压栈时，跟正无穷去取 `min`，都不会出问题，代码会极其清爽！



```go
func (st *MinStack) Push(val int) {
    // 比较当前要入栈的值，和前一个状态的最小值，取更小的那个
    *st = append(*st, pair{val, min(st.GetMin(), val)})
}
```

对于 Push 操作，由于涉及切片的扩容，我使用了指针接收者 `*st`。

每次入栈时，我只需要看一下当前栈顶的最小值（通过 `st.GetMin()` 获取），然后跟新来的 `val` 打个擂台，谁更小，谁就是新的 `preMin`，然后把它们打包成 pair 压入切片。



```go
func (st *MinStack) Pop() {
    *st = (*st)[:len(*st)-1]
}

func (st MinStack) Top() int {
    return st[len(st)-1].val
}

func (st MinStack) GetMin() int {
    return st[len(st)-1].preMin
}
```

对于 Pop 操作，直接将切片长度减一即可，Go 的垃圾回收会在合适的时机处理底层的数组。

 `Top` 和 `GetMin` 都是只读操作，我使用值接收者 `st MinStack` 即可。

因为有了栈底的‘哨兵节点’，题目保证了弹栈和获取操作都是在非空状态下进行，所以我们直接取切片的最后一个元素 `len(st)-1` 就能安全返回结果



在 Go 语言里，方法的接收者（就是函数名紧挨着前面的那个括号）带不带 `*`，完全取决于你**想不想修改原来的数据**：

- **带星号 `(st \*MinStack)`：你需要修改原件！** 这叫“指针接收者”。相当于你拿到了仓库的**真钥匙**，你进去把东西搬空了，别人再来看，仓库就真的空了。

- **不带星号 `(st MinStack)`：你只是看看，不改数据！** 这叫“值接收者”。相当于 Go 语言偷偷给你**复印了一份**栈的数据。你就算把这份复印件撕碎了，原来的真仓库也完好无损。

	

假设你有一个普通栈，你依次压入了四个数字：**`[5, 2, 7, 1]`**。

- 现在谁是最小值？显然是顶部的 **`1`**。获取最小值，美滋滋。
- **🚨 灾难来了：此时执行一次 `Pop()`，把顶部的 `1` 弹走。**
- 现在的栈变成了 **`[5, 2, 7]`**。
- 这时候如果面试官问你：“现在的最小值是多少？



因为那个最小的 `1` 已经被扔掉了，为了找到新的最小值，你只能把剩下的 `[5, 2, 7]` **全部遍历一遍**，才能发现新的最小值是 `2`。 这就意味着，获取最小值的时间复杂度从 $O(1)$ 暴跌成了 **$O(N)$**！这在 LeetCode 上是绝对要超时的

`preMin`（前缀最小值）的本质是：**让每一个入栈的元素，都随身携带一张“属于它那个时代的冠军合影”。**

我们来看看加入了 `preMin` 之后，刚才的操作会变成什么神仙画风：

我们压入数字，但每次压入的不是单身汉，而是一个组合 `{自己真实的数字, 截止到目前的最小值}`：

1. **压入 5**：

	​	前面没人，目前的最小值就是 5。

	​	入栈：**`{val: 5, preMin: 5}`**

2. **压入 2**：

	​	2 跟它脚底下的前缀最小值 (5) 打擂台，2 赢了。 

	​	入栈：**`{val: 2, preMin: 2}`**

3. **压入 7（极其关键的一步！）**：

	​	7 发现它脚底下的前缀最小值是 2。

	​	7 打不过 2，所以这个时代的冠军依然是 2。

	​	入栈：**`{val: 7, preMin: 2}`** *(翻译：我虽然是 7，但我清楚地记得，只要我还踩在最上面，这堆人里的最小值其实是 2！)*

4. **压入 1**：

	​	1 跟脚底下的 2 打擂台，1 赢了。 

	​	入栈：**`{val: 1, preMin: 1}`**

现在的栈长这样（从栈底到栈顶）： `[{5, 5}, {2, 2}, {7, 2}, {1, 1}]`







### 树

#### 236.二叉树的最近公共祖先

给定一个二叉树, 找到该树中两个指定节点的最近公共祖先。

[百度百科](https://baike.baidu.com/item/最近公共祖先/8918834?fr=aladdin)中最近公共祖先的定义为：对于有根树 T 的两个节点 p、q，最近公共祖先表示为一个节点 x，满足 x 是 p、q 的祖先且 x 的深度尽可能大（**一个节点也可以是它自己的祖先**）。

1. **核心思路：向上汇报机制**

我们写一个递归函数，这个函数的作用是：**在以 `root` 为根的树中，找 `p` 和 `q`。**

它向上一层汇报的规则非常简单：

1. **Base Case (终止条件)**：
	- 如果当前节点 `root` 是 `nil`，说明到底了啥也没找到，向上汇报 `nil`。
	- 如果当前节点 `root` 刚好就是 `p` 或者 `q`，说明找到了其中一个！不用再往下找了，直接把当前节点向上汇报。
2. **去左右子树打探消息 (递归)**：
	- 叫左小弟去左子树找：`left := lowestCommonAncestor(root.Left, p, q)`
	- 叫右小弟去右子树找：`right := lowestCommonAncestor(root.Right, p, q)`
3. **汇总小弟的汇报内容 (核心逻辑)**：
	- **情况 A：左边找到一个，右边找到一个** (`left != nil` 且 `right != nil`)。 这说明 `p` 和 `q` 分别在当前节点的左右两边。那么**当前节点 `root` 绝对就是它俩的最近公共祖先！** 直接把 `root` 往上汇报。
	- **情况 B：左边找到了，右边是空的** (`left != nil` 且 `right == nil`)。 这说明 `p` 和 `q` 都在左子树里。那刚才左小弟汇报上来的节点，就是我们要的答案（它要么是提前相遇的公共祖先，要么是先被找到的 `p` 或 `q`），直接把 `left` 往上汇报。
	- **情况 C：右边找到了，左边是空的** (`right != nil` 且 `left == nil`)。 同理，`p` 和 `q` 都在右子树，把 `right` 往上汇报。
	- **情况 D：两边都没找到** (`left == nil` 且 `right == nil`)。 这棵树里根本没有 `p` 和 `q`，往上汇报 `nil`。

```go
type TreeNode struct{
    Val    int
    Left  *TreeNode
    Right *TreeNode
}
func lowestCommonAncestor(root, p, q *TreeNode) *TreeNode {
    if root=nil || root==p||root==q{
        return root
    }
    left:=lowestCommonAncestor(root.Left,p,q)
    right:=lowestCommonAncestor(root.Right,p,q)
    
    if Left!=nil &&right!=nil {
		return root
    }
    if left == nil {
        return right
    }
    return left
}
```

整个过程就像是你在电脑里搜文件：

1. **往下找（从根节点开始）**： 从根节点（C盘）开始，代码一层一层地调用自己：`left := lowestCommonAncestor(root.Left, ...)`。 这就像你双击打开一个个文件夹，一直往下走，直到碰到死胡同（遇到 `nil`），或者找到了你要的文件（遇到了 `p` 或 `q`）。在这个阶段，代码**还没有得到任何结果**，只是在不断地深入。
2. **往上报（从叶子节点/目标节点开始）**： 当你碰壁了（`nil`），或者找到目标了（`p` 或 `q`），这个深不见底的“打开文件夹”的过程终于停止了。 接着，程序开始**“带着线索往回退”**。这才是 `return root`、`return left` 这些代码真正发力的时候。线索从最底层开始，一层一层地向上汇总、合并，最后交回给最顶层的根节点。

#### 114.二叉树展开为链表

给你二叉树的根结点 `root` ，请你将它展开为一个单链表：

- 展开后的单链表应该同样使用 `TreeNode` ，其中 `right` 子指针指向链表中下一个结点，而左子指针始终为 `null` 。
- 展开后的单链表应该与二叉树 [**先序遍历**](https://baike.baidu.com/item/先序遍历/6442839?fr=aladdin) 顺序相同。

**1. 核心思路：寻找“嫁接点”**

先序遍历的顺序是：

**根节点 -> 左子树所有节点 -> 右子树所有节点**。 

```
	1
   / \
  2   5
 / \   \
3   4   6

	1
   / 
  2   
 / \   
3   4
     \
      5
       \
        6
        
 1
  \
   2
  / \
 3   4
      \
       5
        \
         6
         
         
 1
  \
   2
    \
     3
      \
       4
        \
         5
          \
           6
```

这意味着，原二叉树的**右子树**，在展开成链表后，必须要紧紧地跟在**左子树的最后一个节点**的后面。





那么，左子树的“最后一个节点”是谁呢？ 根据先序遍历的规则，它就是左子树里**最右下角的那个节点**！





**“嫁接”四步曲：** 假设当前我们站在某个节点 `curr` 上：

1. 如果它有左子树，我们就一路往右找，找到左子树的**最右节点**。
2. 把 `curr` 原本的**右子树**，整个剪下来，嫁接到那个**最右节点**的右边。
3. 把 `curr` 原本的**左子树**，整个移到 `curr` 的**右边**。
4. 把 `curr` 的左指针清空（设为 `nil`）。

处理完当前节点后，顺着右指针往下走一步（`curr = curr.Right`），重复上面的步骤，直到走完整个链表。

```go
type TreeNode struct{
	Val		int
    Left	*TreeNode
    Right	*TreeNode
}

func flatten(root *TreeNode){
    curr:=root
    for curr!=nil {
        if curr.Left!=nil {
            pre:=curr.Left
            for pre.Right!=nil {
                pre=pre.Right
            }
            //嫁接
            pre.Right=curr.Right
            //平移
            curr.Right=cur.Left
        	curr.Left=nil
        }
       curr=curr.Right
    }
}
```

**嫁接**：**把右子树，挂到左子树的最右下角。**（此时，根节点的右边其实已经空了，所有节点全连在左边了）。

**平移**：把你刚刚缝合好的这一大坨左子树，**整体拔下来，插到右边**。原来在上面的还是在上面，原来在下面的还是在下面，没有任何“颠倒”或“逆置”的动作。

**清空**：最后把左边设为 `null`。

#### 124.二叉树中的最大路径和

二叉树中的 **路径** 被定义为一条节点序列，序列中每对相邻节点之间都存在一条边。同一个节点在一条路径序列中 **至多出现一次** 。该路径 **至少包含一个** 节点，且不一定经过根节点。

**路径和** 是路径中各节点值的总和。

给你一个二叉树的根节点 `root` ，返回其 **最大路径和** 。

![img](https://assets.leetcode.com/uploads/2020/10/13/exx2.jpg)

```go
输入：root = [-10,9,20,null,null,15,7]
输出：42
解释：最优路径是 15 -> 20 -> 7 ，路径和为 15 + 20 + 7 = 42
```

1. 核心破局点：你是“拐点”还是“过客”？

对于树中的任意一个节点，它在一条路径中只能扮演**两种角色**之一：

**角色 A：作为最高“拐点” (Peak)** 

**路径从左子树上来，经过当前节点，然后拐弯走向右子树。** 

此时，这条路径的真正和是：`当前节点值 + 左边能提供的最大和 + 右边能提供的最大和`。

 **注意：** 这种“倒 V 型”的路径，**无法再继续往上汇报给父节点了**，因为它已经把当前节点当作最高顶点了，一条线不能分叉成三根！

**角色 B：作为单向“过客” (Contribution)** 

当前节点只是整条大路径中的一环，它要把自己和自己下面（左边或右边**其中一条**最优的路线）连起来，继续向上汇报给父节点。 

此时，它能向父节点提供的最大贡献是：`当前节点值 + max(左边提供的最大和, 右边提供的最大和)`。

如果一个小弟（左子树或右子树）汇报上来的最大路径和是**负数**怎么办？ **果断断绝关系！** 既然加上你会让总和变得更小，那我宁愿不带你玩，把你当成 `0` 来看待。

```go

type TreeNode struct{
	Val		int
    Left	*TreeNode
    Right	*TreeNode
}

func maxPathSum(root *TreeNode)int{
    maxSum:=math.MinInt
    var dfs func(t *TreeNode)int
    dfs=func(t *TreeNode)int{
        if t==nil {
			return 0
        }
        left:=max(0,dfs(t.Left))
        right:=max(0,dfs(t.Right))
        if t.Val+left+right>maxSum {
            maxSum=t.Val+left+right
        }
        return t.Val+max(left,right)
    }
    dfs(root)
    return maxSum
}
```

#### 105.从前序与中序遍历构建二叉树

给定两个整数数组 `preorder` 和 `inorder` ，其中 `preorder` 是二叉树的**先序遍历**， `inorder` 是同一棵树的**中序遍历**，请构造二叉树并返回其根节点。

**先序遍历 (Preorder)**：`[根节点, 左子树所有节点, 右子树所有节点]`

**中序遍历 (Inorder)**：`[左子树所有节点, 根节点, 右子树所有节点]`

**先序遍历是“指挥官”**：它的**第一个元素永远是当前的根节点**！我们直接抓它出来当老大。

**中序遍历是“地图”**：我们拿着刚刚抓到的老大，去中序遍历里找他的位置。找到之后，老大的**左边就是整个左子树**，老大的**右边就是整个右子树**！

```go
type TreeNode struct {
    Val   int
    Left  *TreeNode
    Right *TreeNode
}

func buildTree(preorder []int, inorder []int) *TreeNode {
    // 1. 终止条件：如果数组都空了，说明没有节点要构建了
    if len(preorder) == 0 {
        return nil
    }
    
    // 2. 抓出老大：先序遍历的第一个元素绝对是当前的根节点
    rootVal := preorder[0]
    root := &TreeNode{Val: rootVal}
    
    // 3. 去中序遍历里找老大，确定地盘的分界线
    i := 0
    for ; i < len(inorder); i++ {
        if inorder[i] == rootVal {
            break // 找到了！此时 i 的值刚好也就是左子树的长度
        }
    }
    
    // 4. 递归大军出动！利用 Go 的切片语法完美切割数组
    // 左子树：
    // preorder 跳过根节点（从 1 开始），往后取 i 个元素：preorder[1 : i+1]
    // inorder 取老大左边的所有元素：inorder[:i]
    root.Left = buildTree(preorder[1:i+1], inorder[:i])
    
    // 右子树：
    // preorder 剩下的全是右子树的：preorder[i+1:]
    // inorder 老大右边的全是右子树的：inorder[i+1:]
    root.Right = buildTree(preorder[i+1:], inorder[i+1:])
    
    // 5. 汇报建好的树
    return root
}
```

#### 889.根据前序和后序建树（抖音）

#### 102.二叉树的层序遍历

给你二叉树的根节点 `root` ，返回其节点值的 **层序遍历** 。 （即逐层地，从左到右访问所有节点）。

##### 方法一：标准解法 —— BFS (广度优先搜索 / 队列)

这是最直观、最符合物理直觉的解法。我们利用**队列 (Queue)** “先进先出”的特性，一层一层地扫描。

**核心技巧：“分层打包”** 

队列里平时会混杂着上下两层的节点。

为了能把同一层的节点装进同一个小数组里，我们需要在每一层开始处理前，**先记录下当前队列的长度 (`levelSize`)**。这个长度就是当前这一层一共有多少个节点。

接下来，我们就固定从队列里取 `levelSize` 次，取完刚好这层结束！

```go
type TreeNode struct{
	Val		int
    Left	*TreeNode
    Right	*TreeNode
}

func levelOrder(root *TreeNode) [][]int {
    var res [][]int
    if root == nil {
        return res
    }
    
    // 1. 初始化队列，把大老板（根节点）请进去
    queue := []*TreeNode{root}
    
    // 2. 只要队列里还有人，就继续按层处理
    for len(queue) > 0 {
        // 🚨 核心逻辑：记录当前层的人数
        levelSize := len(queue)
        
        // currentLevel 用来装当前这一层所有节点的值
        var currentLevel []int
        
        // 3. 严格按照当前层的人数进行批量处理
        for i := 0; i < levelSize; i++ {
            // 从队头请出一个人
            node := queue[0]
            queue = queue[1:] // 弹出队头
            
            // 把他的值记录进当前层的数组
            currentLevel = append(currentLevel, node.Val)
            
            // 如果他有左小弟，让左小弟去队尾排队（留给下一层处理）
            if node.Left != nil {
                queue = append(queue, node.Left)
            }
            // 如果他有右小弟，让右小弟去队尾排队
            if node.Right != nil {
                queue = append(queue, node.Right)
            }
        }
        
        // 4. 当前层处理完毕，把这一层的数组封存，放入最终结果集
        res = append(res, currentLevel)
    }
    
    return res
}
```

##### 方法二：降维打击 —— DFS (深度优先搜索 / 递归)

用 DFS 做层序遍历，听起来有点反直觉：“DFS 不是一条路走到黑吗？怎么能按层输出呢？”

**核心技巧：“带上你的楼层号”** 

DFS 确实是一条路走到黑，但只要我们在往下走的时候，**在参数里带上当前节点所在的“楼层号” (`level`)**，我们就能随时把它塞进对应的数组里！

```go
func levelOrder(root *TreeNode)[][]int{
	res:=[][]int{}
    var dfs func(node *TreeNode,level int)
    dfs=func(node *TreeNode,level int){
        if node==nil {
            return
        }
        if len(res)==level{
            res=append(res,[]int{})
        }
        res[level]=append(res[level],node.Val)
        dfs(node.Left,level+1)
        dfs(node.Right,level+1)
    }
    dfs(root,0)
    return res
}
```

#### 437.路径总和

给定一个二叉树的根节点 `root` ，和一个整数 `targetSum` ，求该二叉树里节点值之和等于 `targetSum` 的 **路径** 的数目。

**路径** 不需要从根节点开始，也不需要在叶子节点结束，但是路径方向必须是向下的（只能从父节点到子节点）。

##### **前缀和 + 回溯 (Hash Map)**。

1. 核心破局思想：把树变成数组看待

你可能做过一道经典的数组题：**“和为 K 的子数组”**。 

在数组中，如果 `[0 到 i 的和] - [0 到 j 的和] = target`，那就说明 `[j+1 到 i]` 这一截的和就是 `target`。

在二叉树里，**完全是一模一样的道理！** 我们可以把从**根节点**一直走到**当前节点**的这条垂直路径，看作是一个数组。

- **当前前缀和 (`currentSum`)**：

	从根节点一路加到当前节点的值。

- **我们想找什么？** 

	我们回头往上看，历史路径中有没有哪个祖先节点的“前缀和”，刚好等于 `currentSum - targetSum`？

- 如果有，说明那个祖先节点到当前节点之间的这一截，和刚好等于 `targetSum`！

```go
type TreeNode struct {
    Val   int
    Left  *TreeNode
    Right *TreeNode
}

func pathSum(root *TreeNode, targetSum int) int {
    // prefixMap 记录【前缀和】出现的【次数】
    prefixMap := make(map[int]int)
    
    // 🚨 极其重要：初始化。代表“前缀和为 0 的情况出现了 1 次”
    // 为什么？如果 currentSum 刚好等于 targetSum，
    // currentSum - targetSum = 0，我们需要从 map 里查到 1，代表从根节点直接到当前节点的路径。
    prefixMap[0] = 1
    
    var dfs func(node *TreeNode, currentSum int) int
    dfs = func(node *TreeNode, currentSum int) int {
        if node == nil {
            return 0
        }
        
        // 1. 累加当前节点，算出当前的前缀和
        currentSum += node.Val
        
        // 2. 查账：历史账本里，有没有刚好等于 currentSum - targetSum 的前缀和？
        // 如果有，就说明存在从某个祖先走到当前的路径，和为 targetSum。
        // （直接累加找到的次数，因为可能会有多个祖先的前缀和相同）
        count := prefixMap[currentSum - targetSum]
        
        // 3. 登记：把当前的前缀和记入账本，给接下来的小弟（左右子树）去查
        prefixMap[currentSum]++
        
        // 4. 往下走：去左右子树里继续找
        count += dfs(node.Left, currentSum)
        count += dfs(node.Right, currentSum)
        
        // 5. 🚨 核心回溯：过河拆桥
        // 当这个节点及其所有子树都遍历完了，准备退回父节点时，
        // 必须把它的前缀和从账本里减掉！
        // 因为二叉树有分叉，它不能被它的“兄弟节点”看到。
        prefixMap[currentSum]--
        
        return count
    }
    
    return dfs(root, 0)
}
```

##### 双重回溯（DFS）

```go
func pathSum(root *TreeNode, targetSum int) int {
    // 1. 终止条件：如果没有树了，路径数为 0
    if root == nil {
        return 0
    }
    
    // 2. 以当前节点为起点，往下死磕，看能找出几条路径
    count := rootSum(root, targetSum)
    
    // 3. 递归：让左孩子及其子孙后代也去当起点试试
    count += pathSum(root.Left, targetSum)
    
    // 4. 递归：让右孩子及其子孙后代也去当起点试试
    count += pathSum(root.Right, targetSum)
    
    return count
}

// 辅助函数：专门负责“以 node 为起点”，往下寻找和为 target 的路径
func rootSum(node *TreeNode, target int) int {
    if node == nil {
        return 0
    }
    
    count := 0
    // 如果当前节点的值刚好等于目标值，恭喜，找到一条路径！
    if node.Val == target {
        count++
    }
    
    // 继续往下找：目标值减去当前节点的值，交接给左右小弟
    count += rootSum(node.Left, target - node.Val)
    count += rootSum(node.Right, target - node.Val)
    
    return count
}
```

##### 数组回溯（物理路径记录法）

这个版本不再“各自为战”了，而是**只从根节点往下走一次（单次 DFS）**。

我们在往下走的过程中，用一个**数组（切片）把路过的节点值全都记录下来。**

每走到一个节点，我们就从后往前遍历这个数组，看看有没有哪一截的后缀加起来等于 `targetSum`。

```go
func pathSum(root *TreeNode,target int)int{
    path:=[]int{}
    var dfs func(node *TreeNode)int
    dfs=func(node *TreeNode)int{
        if node==nil {
            return 0
        }
        path=append(path,node.Val)
        count:=0
        sum:=0
        for i:=len(path)-1;i>=0;i-- {
            sum+=path[i]
            if sum==target {
                count++
            }
        }
        count+=pathSum(node.Left)
        count+=pathSum(node.Right)
        path=path[:len(path)-1]
        return count
    }
    return dfs(root)
}
```

### 图

#### 200.岛屿数量

1. 核心破局思路：“沉岛行动” (DFS 深度优先搜索)

想象你驾驶着一架直升机在海面上巡逻（用两个 `for` 循环遍历整个二维矩阵）：

1. **发现新大陆**：只要你看到脚下有一个 `1`（陆地），恭喜你，发现了一座新岛屿！**岛屿总数立刻 +1**。
2. **沉岛行动（核心步骤）**：为了防止你下次巡逻时重复计算这座岛，你必须立刻降落，派出地面部队（DFS 递归函数），沿着上下左右四个方向，把这座岛上**所有相连的 `1` 全部炸平（变成 `0`）**。
3. **继续巡逻**：炸平之后，你继续开直升机往前飞，因为刚才的岛已经被你抹掉了，你接下来遇到的 `1`，绝对是属于另一座全新岛屿的！

```go
package main

import (
	"fmt"
)

// 核心算法：求岛屿数量
func numIslands(grid [][]byte) int {
	if len(grid) == 0 {
		return 0
	}
	
	rows := len(grid)
	cols := len(grid[0])
	count := 0

	// 定义 DFS “沉岛”函数
	var dfs func(r, c int)
	dfs = func(r, c int) {
		// 1. 终止条件：越界了，或者当前遇到的是水 ('0')，直接返回
		if r < 0 || c < 0 || r >= rows || c >= cols || grid[r][c] == '0' {
			return
		}
		
		// 2. 核心动作：把陆地变成水！（沉岛）
		grid[r][c] = '0'
		
		// 3. 派出四支小分队，向上下左右继续沉岛
		dfs(r-1, c) // 上
		dfs(r+1, c) // 下
		dfs(r, c-1) // 左
		dfs(r, c+1) // 右
	}

	// 开启全图巡逻
	for i := 0; i < rows; i++ {
		for j := 0; j < cols; j++ {
			// 只要发现一块陆地，必定属于一座新岛屿
			if grid[i][j] == '1' {
				count++      // 岛屿数量 +1
				dfs(i, j)    // 立刻执行沉岛行动，把这片连着的陆地全变 0
			}
		}
	}

	return count
}

func main() {
	var m, n int
	
	// 1. 无脑吸入行数(m)和列数(n)
	if _, err := fmt.Scan(&m, &n); err != nil {
		return
	}

	// 2. 提前建好二维数组的“骨架”
	grid := make([][]byte, m)

	// 3. 按行读取地图
	for i := 0; i < m; i++ {
		var rowStr string
		fmt.Scan(&rowStr)         // 每次吸入一整串，比如 "11110"
		grid[i] = []byte(rowStr)  // 把字符串直接转成 byte 数组塞进去
	}

	// 4. 调用核心算法，秒出结果！
	ans := numIslands(grid)
	fmt.Println(ans)
}
```

#### 207.课程表

你这个学期必须选修 `numCourses` 门课程，记为 `0` 到 `numCourses - 1` 。

在选修某些课程之前需要一些先修课程。 先修课程按数组 `prerequisites` 给出，其中 `prerequisites[i] = [ai, bi]` ，表示如果要学习课程 `ai` 则 **必须** 先学习课程 `bi` 。

- 例如，先修课程对 `[0, 1]` 表示：想要学习课程 `0` ，你需要先完成课程 `1` 。

请你判断是否可能完成所有课程的学习？如果可以，返回 `true` ；否则，返回 `false` 。

##### DFS三色标记法

```go
func canFinish(numCourses int, prerequisites [][]int) bool {
    // 1. 建图：这和 BFS 一样，把依赖关系存进二维数组
    // g[x] 存的是“学完课程 x 后，能解锁哪些后续课程 y”
    g := make([][]int, numCourses)
    for _, p := range prerequisites {
        g[p[1]] = append(g[p[1]], p[0]) 
    }

    // 2. 状态记录本（代替了 boolean 的 visited 数组）
    // 默认全为 0 (未访问)
    colors := make([]int, numCourses)
    
    // 3. 核心探测雷达：DFS 函数
    // 作用：从课程 x 出发，顺藤摸瓜，看能不能找到环
    var dfs func(int) bool
    dfs = func(x int) bool {
        // 【关键动作】刚进门，立刻留下“正在访问”的脚印
        colors[x] = 1 
        
        // 依次推开 x 房间里的每一扇门（遍历所有后续课程 y）
        for _, y := range g[x] {
            // 🚨 这一行是整段代码的灵魂！用了 Go 的短路逻辑（|| 和 &&）
            
            // 危险情况一：colors[y] == 1 
            // 门背后的房间竟然有我们刚刚留下的脚印！实锤有环，立刻返回 true！
            
            // 危险情况二：colors[y] == 0 && dfs(y)
            // 门背后的房间是全新的 (0)，那我们就递归进去查 dfs(y)。
            // 如果底层排查小队发回电报说“里面有环 (true)”，我们也立刻向上级报告 true！
            
            // (隐藏情况三)：如果 colors[y] == 2，上面两个条件都不满足，直接跳过！不浪费时间。
            if colors[y] == 1 || (colors[y] == 0 && dfs(y)) {
                return true 
            }
        }
        
        // 【功成身退】x 房间里的所有门都安全排查完毕了！
        // 把它标记为“绝对安全”，以后别人再从其他地方走到 x，就不用再查了。
        colors[x] = 2 
        return false // 没有发现环
    }

    // 4. 总指挥部：确保每个房间都被查过
    for i, c := range colors {
        // 如果房间 i 还没去过 (c == 0)，就派 DFS 小队进去查
        // 只要有一支小队报告说发现了环 (true)，直接判定毕不了业 (false)
        if c == 0 && dfs(i) {
            return false 
        }
    }
    
    // 全军排查完毕，没有任何死循环，恭喜毕业！
    return true 
}
```



##### BFS

1. **核心破局思路：把课程依赖变成“有向图”**

这道题的本质，其实就是**“判断一个有向图中是否存在环”**。

- **节点 (Node)**：每一门课就是一个节点。
- **有向边 (Edge)**：先修课程 `[0, 1]` 意味着你必须先学 1 再学 0。我们在图里画一条线：**`1 ➔ 0`**。
- **死锁（存在环）**：如果你发现 `A ➔ B ➔ C ➔ A`，这就像是职场上的“踢皮球”，你永远毕不了业！只要图里没环，你就一定能排出一个上课顺序。

2. **破局关键数据结构：入度 (In-degree)**

- **什么是入度？** 就是“有几根箭头指向你”。
- **物理意义**：这门课**还有几门先修课没上**。
	- 如果一门课的入度是 `0`，说明它不需要任何先修课，**你现在立刻马上就能去上它！**

3. **BFS (广度优先搜索) 剥洋葱大法**

我们利用一个**队列**，像剥洋葱一样，一层一层把课上完：

1. **统计入度与建图**：遍历 `prerequisites`，把每门课的入度算出来，并建立一个“谁指向谁”的邻接表（也就是你学完这门课，能解锁哪些课）。
2. **零入度入队**：把所有入度为 `0` 的课（可以直接上的课）全部塞进队列。
3. **疯狂上课（出队）**：
	- 从队列拿出一门课，学掉它（已学课程数 +1）。
	- 学完之后，把这门课指向的所有后续课程的入度 **减 1**（相当于帮它们解除了一层依赖）。
	- 如果某个后续课程的入度减到了 `0`，说明它的前置条件全满足了，立刻把它也塞进队列！
4. **最终清算**：等队列空了，看看你“已学课程数”是不是等于 `numCourses`。如果等于，恭喜毕业！如果小于，说明剩下的课互相依赖（死锁/有环），返回 `false`。

```go
func canFinish(numCourses int, prerequisites [][]int) bool {
    // 1. 准备工作：建图和入度数组
    // graph 记录：学完某门课，可以【解锁】哪些后续课 (key: 先修课, value: 后续课列表)
    graph := make([][]int, numCourses)
    // inDegree 记录：某门课还有几个【前置条件】
    inDegree := make([]int, numCourses)
    
    // 遍历条件，填充图和入度数组
    for _, pre := range prerequisites {
        curr := pre[0] // 想要学的课
        prev := pre[1] // 必须先学的课
        
        graph[prev] = append(graph[prev], curr) // prev 指向 curr
        inDegree[curr]++                        // curr 的先修条件 +1
    }
    
    // 2. 寻找突破口：把所有不需要先修课的节点放入队列
    queue := []int{}
    for i := 0; i < numCourses; i++ {
        if inDegree[i] == 0 {
            queue = append(queue, i)
        }
    }
    
    // 3. 开始 BFS 剥洋葱
    count := 0 // 记录成功上完的课程数量
    
    for len(queue) > 0 {
        // 从队列拿出一门可以上的课
        node := queue[0]
        queue = queue[1:]
        count++ // 成功学完一门！
        
        // 看看这门课能解锁哪些后续课程
        for _, nextCourse := range graph[node] {
            inDegree[nextCourse]-- // 后续课程的前置条件 -1
            
            // 如果后续课程的所有前置条件都搞定了，就可以入队准备上了
            if inDegree[nextCourse] == 0 {
                queue = append(queue, nextCourse)
            }
        }
    }
    
    // 4. 结算：如果上完的课等于总课数，说明没有死循环
    return count == numCourses
}
```

#### 1260.二维网络迁移

给你一个 `m` 行 `n` 列的二维网格 `grid` 和一个整数 `k`。你需要将 `grid` 迁移 `k` 次。

每次「迁移」操作将会引发下述活动：

- 位于 `grid[i][j]`（`j < n - 1`）的元素将会移动到 `grid[i][j + 1]`。
- 位于 `grid[i][n - 1]` 的元素将会移动到 `grid[i + 1][0]`。
- 位于 `grid[m - 1][n - 1]` 的元素将会移动到 `grid[0][0]`。

请你返回 `k` 次迁移操作后最终得到的 **二维网格**。



**解题思路**

想象扁平化后的一维数组的平移，再将一维数组映射回二维。
具体来说:
原来的(i,j)对应的一维坐标为i∗n+j,
向右平移k后为i∗n+j+k,
总共只有m∗n个位置所以最终坐标为(i∗n+j+k) % (m∗n),
转换回二维坐标即可。





### 回溯

回溯法，一般可以解决如下几种问题：

- 组合问题：N个数里面按一定规则找出k个数的集合
- 切割问题：一个字符串按一定规则有几种切割方式
- 子集问题：一个N个数的集合里有多少符合条件的子集
- 排列问题：N个数按一定规则全排列，有几种排列方式
- 棋盘问题：N皇后，解数独等等

#### 回溯的三步曲与两个变量

所有的回溯问题，都在回答三个问题：

1. **路径 (tempList/path)**：我已经做出了哪些选择？
2. **选择列表 (nums)**：我现在还有哪些选择可以选？
3. **结束条件**：我什么时候把路径装进结果集里？

**核心变量 1：`start` (控制向后看，不回头)**

- **什么时候用？** 

	​	做**组合 (Combination)** 和 **子集 (Subset)** 的时候！

- **为什么？**

	​	 因为组合里 `[1, 2]` 和 `[2, 1]` 是一样的。

	​	为了防止选了 `2` 之后再回头选 `1`，我们需要一个 `start` 变量。

	​	每次进入下一层，只能从 `start` 开始往后选，绝对不允许回头。

**核心变量 2：`used` 数组 (控制全局，谁被用了)**

- **什么时候用？** 

	​	做**排列 (Permutation)** 的时候！

- **为什么？** 

	​	排列里 `[1, 2]` 和 `[2, 1]` 是两个不同的结果。

	​	每次都要从头（索引 `0`）开始选，所以不能用 `start`。

	​	为了防止同一个位置的数字被重复挑中，必须用一个 `used` 数组记录“哪些坑位已经被占了”。

##### 1. 子集系列 (Subsets) —— 只要组合，不要排列

**特点**：每个节点都是答案，不限制长度。

```go
// 1. 普通子集 (无重复元素)
func subsets(nums []int) [][]int {
    var res [][]int
    var path []int
    
    var backtrack func(start int)
    backtrack = func(start int) {
        // 无条件加入结果集 (深拷贝)
        res = append(res, append([]int(nil), path...))
        
        for i := start; i < len(nums); i++ {
            path = append(path, nums[i])
            backtrack(i + 1)      // 往后走，不能重复用自己
            path = path[:len(path)-1] // 回溯
        }
    }
    backtrack(0)
    return res
}

// 2. 子集 II (有重复元素，必须去重)
import "sort"
func subsetsWithDup(nums []int) [][]int {
    var res [][]int
    var path []int
    sort.Ints(nums) // 🚨 有重复必须先排序！
    
    var backtrack func(start int)
    backtrack = func(start int) {
        res = append(res, append([]int(nil), path...))
        
        for i := start; i < len(nums); i++ {
            // 🚨 核心去重逻辑：同一树层上，遇到相同的直接跳过
            if i > start && nums[i] == nums[i-1] {
                continue 
            }
            path = append(path, nums[i])
            backtrack(i + 1)
            path = path[:len(path)-1]
        }
    }
    backtrack(0)
    return res
}
```

##### 2. 组合总和系列 (Combination Sum) —— 加上了约束条件

**特点**：和子集一样用 `start`，但只有达到 `target` 才算成功。

```go
// 1. 组合总和 (无重复元素，但元素可以无限次重复使用)
func combinationSum(nums []int, target int) [][]int {
    var res [][]int
    var path []int
    
    var backtrack func(start, remain int)
    backtrack = func(start, remain int) {
        if remain < 0 { return } // 剪枝
        if remain == 0 {
            res = append(res, append([]int(nil), path...))
            return
        }
        for i := start; i < len(nums); i++ {
            path = append(path, nums[i])
            // 🚨 因为可以无限次使用自己，所以传 i，而不是 i + 1！
            backtrack(i, remain - nums[i]) 
            path = path[:len(path)-1]
        }
    }
    backtrack(0, target)
    return res
}

// 2. 组合总和 II (有重复元素，每个元素只能用一次)
func combinationSum2(nums []int, target int) [][]int {
    var res [][]int
    var path []int
    sort.Ints(nums) // 🚨 排序为了去重
    
    var backtrack func(start, remain int)
    backtrack = func(start, remain int) {
        if remain < 0 { return }
        if remain == 0 {
            res = append(res, append([]int(nil), path...))
            return
        }
        for i := start; i < len(nums); i++ {
            // 🚨 同层去重！
            if i > start && nums[i] == nums[i-1] { continue }
            path = append(path, nums[i])
            // 因为只能用一次，所以是 i + 1
            backtrack(i + 1, remain - nums[i]) 
            path = path[:len(path)-1]
        }
    }
    backtrack(0, target)
    return res
}
```

##### 3. 排列系列 (Permutations) —— 放弃 start，拥抱 used 数组

**特点**：顺序有关，`[1,2]` 和 `[2,1]` 都要，每次 `for` 循环从 `0` 开始。

```go
// 1. 全排列 (无重复元素)
func permute(nums []int) [][]int {
    var res [][]int
    var path []int
    used := make([]bool, len(nums)) // 记录坑位
    
    var backtrack func()
    backtrack = func() {
        if len(path) == len(nums) {
            res = append(res, append([]int(nil), path...))
            return
        }
        for i := 0; i < len(nums); i++ { // 每次都从 0 开始挑
            if used[i] { continue }      // 这个数字被别人占了，跳过
            
            used[i] = true
            path = append(path, nums[i])
            
            backtrack()
            
            path = path[:len(path)-1]
            used[i] = false              // 🚨 撤销占位！
        }
    }
    backtrack()
    return res
}

// 2. 全排列 II (有重复元素，终极 BOSS)
func permuteUnique(nums []int) [][]int {
    var res [][]int
    var path []int
    used := make([]bool, len(nums))
    sort.Ints(nums) // 🚨 排序！
    
    var backtrack func()
    backtrack = func() {
        if len(path) == len(nums) {
            res = append(res, append([]int(nil), path...))
            return
        }
        for i := 0; i < len(nums); i++ {
            if used[i] { continue }
            
            // 🚨 史上最难理解的去重条件！
            // nums[i] == nums[i-1] 意思是遇到重复数字了
            // !used[i-1] 意思是前一个相同的数字刚刚被撤销（说明在同一个树层上）
            if i > 0 && nums[i] == nums[i-1] && !used[i-1] {
                continue
            }
            
            used[i] = true
            path = append(path, nums[i])
            backtrack()
            path = path[:len(path)-1]
            used[i] = false
        }
    }
    backtrack()
    return res
}
```

你看，这七道题的骨架是不是**完全一模一样**？

- `for` 循环横向遍历（决定这一层选谁）。
- 递归纵向遍历（决定下一层干嘛）。
- 剩下的无非就是修修补补：能不能回头（`start`）、能不能重复用（`i` 还是 `i+1`）、要不要去重（`nums[i] == nums[i-1]`）。

##### 浅拷贝切片陷阱

在整个回溯的过程中，`path` 这个切片就像是一块**只有一块的白板**。

- 我们在白板上写字（`append` 加元素），然后擦掉字（`[:len(path)-1]` 回溯）。
- 如果你写 `res = append(res, path)`，你以为你是把白板上的字装进了结果集，**但实际上，你只是把“这块白板的地址（指针）”装进了结果集！**
- 等到整个算法运行结束时，结果集里装了 10 个指针，但这 10 个指针指向的都是**同一块白板**。而此时这块白板刚好被回溯操作“擦”成了空板，或者留着最后一次修改的残影。所以你的结果全错了。

**破局之道：深拷贝（Deep Copy）**

```go
res = append(res, append([]int(nil), path...))
```

既然不能交地址，那我们该怎么办？ 

答案是：**给这块白板拍一张“拍立得照片”（申请全新的内存），然后把照片存进结果集。**

这样后续哪怕你把白板砸了，照片上的内容也永远不会变。

这就叫**深拷贝**。这行奇怪的代码，就是 Go 语言里极其优雅的“拍立得”一键拍照法：

```go
append([]int(nil), path...)
```

**`[]int(nil)`**：凭空创造一个完全干净的、长度为 0 的空切片（新相纸）。

**`path...`**：这是 Go 语言的“打散（Unpack）”语法。它把 `path` 里面的元素 `[1, 2, 3]` 拆解成了独立的数字 `1, 2, 3`。

**`append(空切片, 1, 2, 3)`**：把这些独立的数字，塞进那个全新的空切片里。因为是全新的切片，Go 底层会自动为它**开辟一块全新的内存空间**！





#### 模板

- 回溯函数模板返回值以及参数

在回溯算法中，我的习惯是函数起名字为backtracking，这个起名大家随意。

回溯算法中函数返回值一般为void。

再来看一下参数，因为回溯算法需要的参数可不像二叉树递归的时候那么容易一次性确定下来，所以一般是先写逻辑，然后需要什么参数，就填什么参数。

但后面的回溯题目的讲解中，为了方便大家理解，我在一开始就帮大家把参数确定下来。

回溯函数伪代码如下：

```text
void backtracking(参数)
```

- 回溯函数终止条件

既然是树形结构，那么我们在讲解[二叉树的递归 (opens new window)](https://programmercarl.com/二叉树的递归遍历.html)的时候，就知道遍历树形结构一定要有终止条件。

所以回溯也有要终止条件。

什么时候达到了终止条件，树中就可以看出，一般来说搜到叶子节点了，也就找到了满足条件的一条答案，把这个答案存放起来，并结束本层递归。

所以回溯函数终止条件伪代码如下：

```text
if (终止条件) {
    存放结果;
    return;
}
```

- 回溯搜索的遍历过程

在上面我们提到了，回溯法一般是在集合中递归搜索，集合的大小构成了树的宽度，递归的深度构成的树的深度。

![回溯算法理论基础](https://file1.kamacoder.com/i/algo/20210130173631174.png)

回溯函数遍历过程伪代码如下：

```text
for (选择：本层集合中元素（树中节点孩子的数量就是集合的大小）) {
    处理节点;
    backtracking(路径，选择列表); // 递归
    回溯，撤销处理结果
}
```

for循环就是遍历集合区间，可以理解一个节点有多少个孩子，这个for循环就执行多少次。

backtracking这里自己调用自己，实现递归。

大家可以从图中看出**for循环可以理解是横向遍历，backtracking（递归）就是纵向遍历**，这样就把这棵树全遍历完了，一般来说，搜索叶子节点就是找的其中一个结果了。



```go
void backtracking(参数) {
    if (终止条件) {
        存放结果;
        return;
    }

    for (选择：本层集合中元素（树中节点孩子的数量就是集合的大小）) {
        处理节点;
        backtracking(路径，选择列表); // 递归
        回溯，撤销处理结果
    }
}
```

#### 79.单词搜索

给定一个 `m x n` 二维字符网格 `board` 和一个字符串单词 `word` 。

如果 `word` 存在于网格中，返回 `true` ；否则，返回 `false` 。

单词必须按照字母顺序，通过相邻的单元格内的字母构成，其中“相邻”单元格是那些水平相邻或垂直相邻的单元格。同一个单元格内的字母不允许被重复使用。

回溯解法：

```go
package main

import (
	"fmt"
)

// ================== 1. 核心算法区 (代码随想录模板) ==================
func exist(board [][]byte, word string) bool {
	rows, cols := len(board), len(board[0])

	// 工具 1：visited 数组，记录走过的路，防止回头
	visited := make([][]bool, rows)
	for i := range visited {
		visited[i] = make([]bool, cols)
	}

	// 工具 2：方向数组 (上、下、左、右)
	dirs := [][]int{{-1, 0}, {1, 0}, {0, -1}, {0, 1}}

	// 定义回溯函数
	var backtracking func(r, c, index int) bool
	backtracking = func(r, c, index int) bool {
		// 1. 终止条件 (剪枝与成功判断)
		if board[r][c] != word[index] {
			return false // 字符不匹配，直接走不通，剪枝！
		}
		if index == len(word)-1 {
			return true // 匹配到了单词的最后一个字符，大功告成！
		}

		// 2. 处理节点
		visited[r][c] = true // 标记当前格子已经踩过了

		// 3. 本层集合的遍历 (for 循环遍历四个方向)
		for _, d := range dirs {
			nextR, nextC := r + d[0], c + d[1]

			// 判断下一个位置是否合法：不越界，且没有被踩过
			if nextR >= 0 && nextR < rows && nextC >= 0 && nextC < cols && !visited[nextR][nextC] {

				// 4. 递归
				// 如果某一个方向最终找到了答案，直接一路 return true
				if backtracking(nextR, nextC, index+1) {
					return true
				}
			}
		}

		// 5. 回溯，撤销处理结果
		visited[r][c] = false

		return false
	}

	// 主逻辑：寻找起始点
	for i := 0; i < rows; i++ {
		for j := 0; j < cols; j++ {
			// 如果找到了一个和首字母一样的格子，就开始启动回溯
			// 只要有一条路走通了，就返回 true
			if backtracking(i, j, 0) {
				return true
			}
		}
	}
	return false
}

// ================== 2. ACM 输入输出区 ==================
func main() {
	var m, n int

	// 1. 读取行数 m 和列数 n (如果读不到，说明输入结束了)
	if _, err := fmt.Scan(&m, &n); err != nil {
		return
	}

	// 2. 初始化二维网格
	board := make([][]byte, m)
	for i := 0; i < m; i++ {
		board[i] = make([]byte, n)
		for j := 0; j < n; j++ {
			// 读取每一个字符。fmt.Scan 会自动忽略所有的空格和换行！
			var s string
			fmt.Scan(&s)
			board[i][j] = s[0] // 提取字符串的第一个字节存入网格
		}
	}

	// 3. 读取最后要寻找的目标单词
	var word string
	fmt.Scan(&word)

	// 4. 调用核心算法并输出结果
	ans := exist(board, word)
	if ans {
		fmt.Println("true")
	} else {
		fmt.Println("false")
	}
}
```

### 动规

**对于动态规划问题，我将拆解为如下五步曲，这五步都搞清楚了，才能说把动态规划真的掌握了！**

1. 确定dp数组（dp table）以及下标的含义
2. 确定递推公式
3. dp数组如何初始化
4. 确定遍历顺序
5. 举例推导dp数组

#### 闫式DP

https://www.cnblogs.com/IzayoiMiku/p/13635809.html

**所有的DP问题，本质上都是有限集中的最值问题**

动态规划有两个要点：状态与状态转移

那么阶段自然也应该有两个：**状态表示**和**状态计算**

##### 状态表示：化零为整

把几个具有相同点的元素合在一起考虑，成为一个状态

对于一个状态 F(i) ，考虑两个角度：

- **集合** ：F(i) 表示什么集合

由于 F(i) 表示的是一堆东西(这也是DP优于枚举的核心)，我们要考虑这一堆东西的共同特征，如：所有满足某个条件的元素集合

这一点请仔细考虑，到底是大于等于，大于，小于，小于等于，等于......这些的不同会导致状态计算方式的不同

- **属性**：F(i) 存的数与集合的关系：如 max,min,count,sum 等

很明显，F(i) 大多数时候是一个数，代表这个集合的某一个属性，多是最大值、最小值、数量、总和等。题目问什么，属性一般就是什么

##### 状态计算：化整为零

先看 F(i) 表示的集合

![img](https://pic.downk.cc/item/5f578578160a154a67b0048d.png)

将其划分为若干个子集，要求**不重**(针对涉及加和类型的属性)和**不漏**

![img](https://pic.downk.cc/item/5f57862f160a154a67b03566.png)

划分的依据：找最后一个不同点(这个待会会讲)

划分过后，求 F(i) 就可根据子集来求

如：

当属性为 max 时，F(i)=max(子集的max)
当属性为 count 时，F(i)=∑(子集的count)

##### 具体例子

###### 0-1背包问题

有 N 件物品以及一个容量为 V 的背包，每个物品只能使用一次。

放入第 i 件物体的代价是 Ci ，得到的价值是 Wi。

求在不超过容量的情况下获得的最大价值。







#### 279.完全平方数

给你一个整数 n ，返回 和为 n 的完全平方数的最少数量 。

完全平方数 是一个整数，其值等于另一个整数的平方；换句话说，其值等于一个整数自乘的积。例如，1、4、9 和 16 都是完全平方数，而 3 和 11 不是



很多人拿到这道题，第一反应是：“既然要数量最少，那我就每次都**挑最大的完全平方数**去减呗！”

我们来推演一下这个“贪心”策略：

假设 $n = 12$。

- 比 12 小的最大平方数是 9。我们选了 **9**，还剩下 12 - 9 = 3。
- 3 里面最大的平方数只能是 1。我们选了 **1**，还剩下 2。
- 再选 **1**，剩 1。
- 再选 **1**，凑齐了。
- 贪心策略得出的答案是：12 = 9 + 1 + 1 + 1（共需要 **4** 个数）。

但是，稍等一下！作为一个精打细算的资深 Gopher，你肯定发现了更优解：

**12 = 4 + 4 + 4**（只需要 **3** 个数！）

**军师点拨：** 贪心算法在这道题里彻底破产了。因为“局部最优”（每次选最大的）并不等于“全局最优”。既然不能贪心，我们就必须老老实实地去评估**所有可能的组合**，这正是**动态规划**拿手的好戏！

**我们定义 $dp[i]$ 表示：**凑够 $i$ 元最少需要的硬币数量**。**

现在，假设我们要凑够 $i$ 元，我们手里最后一枚要给出去的硬币面值是 $j^2$（即 $j \times j$，且这个面值不能超过 $i$）。

那么，在给这枚硬币**之前**，我们手里还需要凑够多少钱？

答案是：$i - j^2$ 元。

既然我们已经知道了凑够 $i - j^2$ 元最少需要 $dp[i - j^2]$ 枚硬币，那么我们只要再加上手里这最后 1 枚硬币，就凑够了 $i$ 元！

​										$$dp[i] = \min(dp[i], dp[i - j^2] + 1)$$	



#### 322.零钱总换

给你一个整数数组 `coins` ，表示不同面额的硬币；以及一个整数 `amount` ，表示总金额。

计算并返回可以凑成总金额所需的 **最少的硬币个数** 。如果没有任何一种硬币组合能组成总金额，返回 `-1` 。

你可以认为每种硬币的数量是无限的。

定义 $dp[i]$ 为：**凑齐总金额 $i$ 所需要的最少硬币个数**。



假设我们现在要算 $dp[i]$，并且我们开始遍历口袋里的硬币数组 `coins`。对于当前手里拿到的一枚面值为 $c$ 的硬币（前提是 $i \ge c$，不然钱就爆了）： 如果我决定把这枚硬币扔出去，那我还需要凑的钱就是 $i - c$。 那么，凑齐 $i$ 元的硬币数 = 凑齐 $i - c$ 元的最少硬币数 + 1（刚刚扔出去的这 1 枚）。

$$dp[i] = \min(dp[i], dp[i - c] + 1)$$



#### 139.单词拆分

给你一个字符串 `s` 和一个字符串列表 `wordDi	ct` 作为字典。

如果可以利用字典中出现的一个或多个单词拼接出 `s` 则返回 `true`。

**注意：**不要求字典中出现的单词全部都使用，并且字典中的单词可以重复使用。

🏃‍♂️ 核心思想：一场接力赛

想象一下，你要在一座断桥上铺路，目标是铺到第 `i` 个桥墩（也就是字符串的第 `i` 个字符）。

你手里有很多不同长度的木板（字典里的单词）。

你要怎么确定自己能不能铺到第 `i` 个桥墩？ 很简单，你只需要回头看看：

**在前面的桥墩里，有没有哪一个桥墩（假设是第 `j` 个）是我已经成功铺到的？并且，从第 `j` 个桥墩到第 `i` 个桥墩之间的这个缺口，我手里刚好有一块木板（字典里的单词）能严丝合缝地填上？**

只要满足这两个条件，这就完成了一次完美的“接力”，你就能成功到达第 `i` 个桥墩！

定义 `dp[i]` 为：**字符串 `s` 的前 `i` 个字符（即 `s[0:i]`），能不能被字典里的单词成功拼出来？**（它的值是一个布尔类型：`true` 或 `false`）。

假设我们现在要计算 `dp[i]`，我们要怎么做？

我们派出一个“探子” `j`，从 `0` 一直往后走到 `i-1`，去寻找那个能完成接力的“前置桥墩”。

只要在这个过程中，碰到**任意一个** `j` 满足以下两个条件，`dp[i]` 就直接宣布通关（设为 `true`）：

1. **接力棒传到了：** `dp[j]` 是 `true`（说明前 `j` 个字符已经成功拼出来了）。
2. **缺口刚好补上：** 字符串截取的后半截 `s[j:i]`，刚好存在于字典 `wordDict` 中。

这就得出了我们极其优雅的状态转移方程：

​								$$dp[i] = dp[j] \land (s[j:i] \in wordDict)$$

```go
func wordBreak(s string, wordDict []string) bool {
    wordmap:=make(map[string]struct{})
    for _,word:=range wordDict{
        wordmap[word]=struct{}{}
    }
    n:=len(s)
    dp:=make([]bool,n+1)
    dp[0]=true
    for i:=1;i<=n;i++ {
        for j:=0;j<i;j++ {
            _,ok:=wordmap[s[j:i]]
            if dp[j] && ok {
                dp[i]=true
                break
            }
        }
    }
    return dp[n]
}
```

**空结构体 `struct{}` 的“0 字节魔法”**

在 Go 语言里，如果你只想要一个集合（Set）来判断“元素存不存在”，而完全不关心它的 Value 是什么，最硬核、最老道的写法是使用**空结构体 `struct{}`**！

在 Go 中，`struct{}` 是一个极其神奇的存在，如果你用 `unsafe.Sizeof(struct{}{})` 去测一下，你会发现它的占用空间是惊人的 **0 字节**！

当我们定义 `map[string]struct{}` 时，Go 编译器在底层会做极大的优化，这个 map 将完全退化成一个纯粹的键集合（Key Set），Value 部分不占用任何额外的内存。

使用 `map[string]struct{}` 配合 `_, ok := map[key]` 的语法，是 Go 语言标准库和各种顶级开源项目（比如 Kubernetes, Docker）里实现 Set 的**唯一标准姿势**。

#### 300.最长递增子序列

给你一个整数数组 `nums` ，找到其中最长严格递增子序列的长度。

**子序列** 是由数组派生而来的序列，删除（或不删除）数组中的元素而不改变其余元素的顺序。例如，`[3,6,2,7]` 是数组 `[0,3,1,6,2,2,7]` 的子序列。

**🎯 极其反直觉的状态定义 (新手第一坑)**

如果按以前的套路，你可能会想把 $dp[i]$ 定义为：“前 $i$ 个元素里的最长递增子序列长度”。

**千万别这么干！死路一条！**

因为如果你这么定义，当你推导到第 $i+1$ 个元素时，你根本不知道前一个最长序列的**结尾那个数到底是多少**。不知道结尾是谁，你就没法判断当前这个数能不能接在后面！

所以，标准定义是：

定义 $dp[i]$ 为：**强制以 `nums[i]` 这个数作为结尾的**最长严格递增子序列的长度。

*(划重点：这个子序列的最后一个元素，**必须**是 `nums[i]` 本身！)*



既然强制以 `nums[i]` 结尾，那在遇到 `nums[i]` 之前，这个序列长什么样？

很简单，我们再次派出“探子” $j$。让 $j$ 在 $0$ 到 $i-1$ 的范围内去挨个视察前面的前辈们。



当你（`nums[i]`）站在前辈（`nums[j]`）面前时，只有两种情况：

- **情况 A（你比前辈矮或一样高）：** $nums[i] \le nums[j]$。严格递增被打破了，你绝对不能接在人家后面。这条路不通，直接忽略。
- **情况 B（你比前辈高）：** $nums[i] > nums[j]$。恭喜你！你可以光明正大地站在前辈后面，蹭一蹭前辈积累的长度。此时你获得的序列长度就是前辈的长度加上你自己：$dp[j] + 1$。

因为我们要找**最长**的，所以你要一路打听所有比你矮的前辈，看看谁背后的队伍最长，你就果断加入谁！



我们的作案公式闪亮登场：

​												$$	dp[i] = \max(dp[i], dp[j] + 1)$$

*(前提条件是 $nums[i] > nums[j]$)*

```go
func lengthOfLIS(nums []int) int {
    //dp[i]=max(dp[i],dp[j]+1)
    dp:=make([]int,len(nums))
    maxn:=1
    for i:=0;i<len(nums);i++ {
        dp[i]=1
        for j:=0;j<i;j++ {
            if nums[i]>nums[j] {
                dp[i]=max(dp[i],dp[j]+1)
            }
        }
        maxn=max(maxn,dp[i])
    }
    return maxn
}
```

接下来，我们要抛弃传统的 DP 思维，引入一种被称为**“纸牌游戏”（Patience Sorting）的贪心+二分查找策略**。

它能把寻找合适前辈的时间，从遍历的 $O(n)$ 瞬间压缩到 $O(\log n)$。

**🃏 核心思想：发牌桌上的“贪心”**

想象你现在是一个荷官，手里拿着 `nums` 数组，准备往桌子上发牌。我们发牌的最终目的，是让桌子上的牌堆尽可能多（牌堆的数量，就是最长递增子序列的长度）。

**贪心法则：** 要想让牌堆变得最多，我们每次放在牌堆顶上的牌就要**尽可能地小**。因为只有结尾的数字越小，后面才越容易接上更大的数字！

**发牌规则：**

1. 只能把牌压在比它**大（或等于）**的牌堆顶上。
2. 如果找不到能压的牌堆（当前牌比所有牌堆顶的牌都大），那就**在最右边新建一个牌堆**。
3. 如果有多个牌堆都能压，**必须压在最左边**的那个合法牌堆上。



🧠 实战推演：以 `[10, 9, 2, 5, 3, 7, 101, 18]` 为例

我们用一个数组 `tails` 来记录每个牌堆**最顶上**的那张牌。

来了一张 **10**：桌上没牌，建新堆。`tails = [10]`

来了一张 **9**：9 比 10 小，压在 10 上面。`tails = [9]` （贪心体现：结尾变成 9，比 10 更有利于后续接牌）

来了一张 **2**：2 比 9 小，压在 9 上面。`tails = [2]`

来了一张 **5**：5 比 2 大，没地方压，在右边建新堆！`tails = [2, 5]`

来了一张 **3**：3 比 5 小，压在 5 上面。`tails = [2, 3]`

来了一张 **7**：7 比 3 大，建新堆！`tails = [2, 3, 7]`

来了一张 **101**：建新堆！`tails = [2, 3, 7, 101]`

来了一张 **18**：18 比 101 小，压在 101 上面。`tails = [2, 3, 7, 18]`

牌发完了！`tails` 数组的长度是 4，所以最长递增子序列的长度就是 **4**！

```go
func lengthOfLIS(nums []int) int {
    // tails 数组存储每个“牌堆”顶部的最小元素
    tails := make([]int, 0)

    for _, num := range nums {
        // 在 tails 数组中进行二分查找，寻找第一个大于等于 num 的位置
        left, right := 0, len(tails)
        
        for left < right {
            mid := left + (right - left) / 2
            if tails[mid] < num {
                // 当前牌堆顶比 num 小，说明 num 不能压在这里，去右边找
                left = mid + 1
            } else {
                // 当前牌堆顶大于等于 num，num 可以压，但为了贪心，我们看看左边还有没有更合适的
                right = mid
            }
        }

        // left 就是 num 应该放置的牌堆位置
        if left == len(tails) {
            // 如果 left 跑到了 tails 的末尾，说明 num 比所有牌堆顶都大，只能新建一个牌堆
            tails = append(tails, num)
        } else {
            // 否则，用 num 覆盖掉原来牌堆顶的牌（让牌堆顶变得更小，更有利于后续接牌）
            tails[left] = num
        }
    }

    // tails 数组的长度，就是牌堆的数量，也就是 LIS 的长度
    return len(tails)
}
```

在 Go 语言中，你可以手写二分，也可以直接调用标准库 `sort.SearchInts`

```go
import "sort"

func lengthOfLIS(nums []int) int {
    tails := make([]int, 0)

    for _, num := range nums {
        // 1. 直接调用标准库，一行代码搞定二分查找！
        // idx 就是当前这张牌 num 应该压在哪个牌堆上
        idx := sort.SearchInts(tails, num)

        // 2. 根据返回的索引决定是“新建牌堆”还是“覆盖旧牌”
        if idx == len(tails) {
            // 所有牌堆顶都比 num 小，只能在最右边追加一个新牌堆
            tails = append(tails, num)
        } else {
            // 找到了可以压的牌堆，为了贪心策略，用较小的 num 覆盖掉原来的堆顶
            tails[idx] = num
        }
    }

    return len(tails)
}
```





#### 152.乘积最大子数组

给你一个整数数组 `nums` ，请你找出数组中**乘积最大的非空连续子数组**（该子数组中至少包含一个数字），并返回该子数组所对应的乘积。

测试用例的答案是一个 **32-位** 整数。

**请注意**，一个只包含一个元素的数组的乘积是这个元素的值。

一：

```go
fMax[i] = max(fMax[i-1]*x, fMin[i-1]*x, x)
fMin[i] = min(fMax[i-1]*x, fMin[i-1]*x, x)
```

这句代码的意思是，把下面这三个候选人拉出来打一架，选出**最大的当 `fMax[i]`，最小的当 `fMin[i]`**：

1. **`x` (单干)**：

	​	前面的累积太垃圾了（比如前面乘积是 0 或者负数，而我是正数）

	​	我不想带前面玩了，我自己自立门户，作为一个新子数组的开头！

2. **`fMax[i-1] * x` (锦上添花)**：

	​	我 `x` 是正数，前面的最大乘积也是正数，强强联合，正正得正！

3. **`fMin[i-1] * x` (绝地翻盘)**：

	​	前面的最小乘积是一个极其可怕的负数（比如 -100），但我 `x` 刚好也是个负数（比如 -2）！

	​	**负负得正**，瞬间暴涨成 200，完成惊天逆转！

```go
func maxProduct(nums []int) int {
    n := len(nums)
    fMax := make([]int, n)
    fMin := make([]int, n)
    fMax[0], fMin[0] = nums[0], nums[0]
    for i := 1; i < n; i++ {
        x := nums[i]
        // 把 x 加到右端点为 i-1 的（乘积最大/最小）子数组后面，
        // 或者单独组成一个子数组，只有 x 一个元素
        fMax[i] = max(fMax[i-1]*x, fMin[i-1]*x, x)
        fMin[i] = min(fMax[i-1]*x, fMin[i-1]*x, x)
    }
    return slices.Max(fMax)
}
```

二：

```go
func maxProduct(nums []int) int {
    res:=math.MinInt
    dpMax,dpMin:=1,1
    for _,x:=range nums{
        dpMax, dpMin = max(dpMax*x, dpMin*x, x), min(dpMin*x, dpMax*x, x)
        res=max(res,dpMax)
    }
    return res
}
```

#### 416.分割等和子集

给你一个 **只包含正整数** 的 **非空** 数组 `nums` 。

请你判断是否可以将这个数组分割成两个子集，使得两个子集的元素和相等。

**把题目“翻译”成背包问题**

题目要求把数组分成和相等的两部分。 这意味着什么？意味着**这两部分的和，必定都等于整个数组总和的一半！**

在最原始的背包问题里，物品有**重量 (Weight)** 和 **价值 (Value)** 两个属性，我们的目标是在不超过背包容量的情况下，装入**最大价值**的物品。

在这道“分割等和子集”的题里，数字 `nums[i]` 既是它的“重量”，也是它的“价值”！

- **背包总容量**：`target`（也就是 `sum / 2`）。
- **物品的重量**：`nums[i]`。
- **物品的价值**：也是 `nums[i]`。
- **定义状态**：`dp[j]` 表示容量为 `j` 的背包，**最多**能装下的物品总价值（总和）。

如果按照你的公式去不断装东西，等到所有的数字都考察完，我们去翻看 `dp[target]`（容量为 `target` 的背包最多装了多少价值）： 如果 **`dp[target] == target`**，这意味着什么？意味着背包装得严严实实，一丁点缝隙都没留！这不就刚好说明我们挑出了和为 `target` 的子集了吗！

#### 32.最长有效括号

给你一个只包含 `'('` 和 `')'` 的字符串，找出最长有效（**格式正确且连续**）括号 子串 的长度。

左右括号匹配，即每个左括号都有对应的右括号将其闭合的字符串是格式正确的，比如 `"(()())"`。

##### 栈模拟

```go
func longestValidParentheses(s string) int {
    maxLen := 0
    
    // 你的切片模拟栈，但这次我们存的是下标 (int)
    // 灵魂操作：在最底下垫一个 -1！
    // 它的作用相当于给第一个合法的子串画一条“起跑线”
    stack := []int{-1}

    for i := 0; i < len(s); i++ {  
            // 遇到左括号，下标老老实实入栈
            stack = append(stack, i)
        } else {
            // 遇到右括号，不管三七二十一，先弹出一个栈顶元素
            // (代表它跟当前右括号匹配掉了，或者把那条垫底的边界弹出了)
            stack = stack[:len(stack)-1]

            if len(stack) == 0 {
                // 如果栈被弹空了，说明刚才弹出的其实是那条“垫底边界”
                // 这意味着当前这个右括号是个多余的“捣蛋鬼”，它把连续性切断了！
                // 没关系，我们把捣蛋鬼的下标放进去，作为新一段子串的“起跑线”
                stack = append(stack, i)
            } else {
                // 如果栈没空，说明真真切切地匹配成了一对！
                // 当前有效长度 = 当前下标 i - 匹配后栈顶的下标 (起跑线)
                currentLen := i - stack[len(stack)-1]
                maxLen = max(maxLen, currentLen)
            }
        }
    }
    
    return maxLen
}

// 辅助函数
func max(a, b int) int {
    if a > b { return a }
    return b
}
```

##### DP

定义一个和字符串等长的数组 `dp`。 

`dp[i]` 表示：**严格以索引 `i` 处的字符结尾的、最长有效括号子串的长度。**



这里有一个非常直白的推论：

 只要 `s[i]` 是左括号 `(`，那它绝对不可能是一个有效闭合子串的结尾。

 所以，**所有左括号对应的 `dp[i]` 全部为 0**。我们只需要把精力放在右括号 `)` 上！

当 `s[i] == ')'` 时，它要怎么和前面的字符组成合法序列？

我们需要看它**紧挨着的前一个字符 `s[i-1]`** 是什么。

这就分成了两种情况：

**情况 A：前一个是左括号，长这样 `...()`**

这就是最简单的“情投意合”。他们俩直接配对，长度为 2。

那配对完之后呢？我们要加上他们俩**前面**那个合法子串的长度（也就是 `dp[i-2]`）。

所以公式是：

$$dp[i] = dp[i-2] + 2$$

*(当然，前提是 $i - 2 \ge 0$)*

**情况 B：前一个也是右括号，长这样 `...))`**

这个情况最烧脑！当前字符是 `)`，前一个也是 `)`。

既然前一个字符是 `)`，如果它属于一个有效子串，那么这个有效子串的长度就是 `dp[i-1]`。

为了让当前的 `)` 也能匹配上，我们需要**跨过**前面那个已经配对好的子串，去它的**前面**找一个落单的 `(`！

这个落单的 `(` 的位置在哪里？

就在：`i - dp[i-1] - 1`。

如果这个位置刚好是一个 `(`，谢天谢地，跨界配对成功！

这次配对成功新增了 2 的长度，加上被跨过的那个子串长度 `dp[i-1]`，同时**千万别忘了**，那个落单的 `(` 前面可能还有合法的子串（位置在 `i - dp[i-1] - 2`），也要一并加起来！

所以终极的转移公式是：

$$dp[i] = dp[i-1] + 2 + dp[i - dp[i-1] - 2]$$

```go
func longestValidParentheses(s string) int {
	// s_byte := []byte(s)
	// l := len(s)
	// stack := make([]int, 1)
	// stack[0] = -1

	// maxLen := 0
	// for i := 0; i < l; i++ {
	// 	if s_byte[i] == '(' {
	// 		stack = append(stack, i)
	// 	} else {
	// 		stack = stack[:len(stack)-1]
	// 		if len(stack) == 0 {
	// 			stack = append(stack, i)
	// 		} else {
	// 			maxLen = max(maxLen, i-stack[len(stack)-1])
	// 		}
	// 	}
	// }
	// return maxLen 

	s_byte := []byte(s)
	l := len(s)
	dp := make([]int, l+1)
	maxLen := 0
	for i := 1; i < l; i++ {
		if s_byte[i] == '(' {
			dp[i] = 0
		} else {
			if s_byte[i-1] == '(' {
				if i-2 >= 0 {
					dp[i] = dp[i-2] + 2
				} else {
					dp[i] = 2
				}

			} else if i-1-dp[i-1] >= 0 && s_byte[i-1-dp[i-1]] == '(' {
				if i-1-dp[i-1]-1 >= 0 {
					dp[i] = dp[i-1] + 2 + dp[i-1-dp[i-1]-1]
				} else {
					dp[i] = dp[i-1] + 2
				}
			}
			maxLen = max(maxLen, dp[i])
		}
	}
	return maxLen    
}
```

##### 双向扫描（双指针）

核心法则：天平的平衡

有效条形码的沟通特征是什么？是**平衡**。左条形码`(`的数量，必须等于右条形码`)`的数量。

所以，我们只带上两个整数：`left`和`right`。需要从左到右遍历字符串：

- 遇到`(`，`left++`
- 遇到`)`，`right++`

就在这期间，天平会发生明显的情况：

1. **`left == right`**：完美平衡！说明我们刚刚走过了一段极大的有效横跨子串。今天记录长度：`maxLen = max(maxLen, 2 * right)`。
2. **`left > right`**：左括号暂时先行。没关系，也许后面还有右括号能补上，我们按兵不动，继续往下走。
3. **`right > left`**：致命失衡！右边的事实居然比左事实上还多！这就像大坝决堤，前面的左事实全被耗光了，还多出一个`)`。这个多出来的`)`直接宣判了可能序列的消耗。**处理方法：马上把`left`和`right`全部清零，从下一个字符重新统计。**

🕳️致命的盲区：为什么只扫一遍不行？

如果只从左到右扫一遍，遇到`s = "(()"`会发生什么？遍历结束时，`left = 2, right = 1`。天平永远没有达到`left == right`完美的平衡，导致`maxLen`一直是0！但显然里面有一个合法的`()`啊！

降维打击：时光倒流，逆向再扫一遍！

如此从左往右扫，处理不了“左宽度过多”的情况；那我们就调转车头，**从右往左**再扫一遍！

```go
func longestValidParentheses(s string) int {
    left, right := 0, 0
    maxLen := 0

    // 第一波攻势：从左到右正向扫描
    // 专治“右括号过多”切断连续性的情况
    for i := 0; i < len(s); i++ {
        if s[i] == '(' {
            left++
        } else {
            right++
        }
        
        if left == right {
            maxLen = max(maxLen, 2*right)
        } else if right > left {
            // 右括号太多了，天平彻底崩塌，直接清零重来
            left, right = 0, 0
        }
    }

    // 重置天平，准备第二波攻势
    left, right = 0, 0

    // 第二波攻势：从右到左反向扫描
    // 专治“左括号过多”导致无法触发 left == right 的情况
    for i := len(s) - 1; i >= 0; i-- {
        if s[i] == '(' {
            left++
        } else {
            right++
        }
        
        if left == right {
            maxLen = max(maxLen, 2*left)
        } else if left > right {
            // 这次反过来了，左括号太多了，清零重来
            left, right = 0, 0
        }
    }

    return maxLen
}

func max(a, b int) int {
    if a > b { return a }
    return b
}
```

#### 62.不同路径

```go
func uniquePaths(m int, n int) int {
    dp:=make([][]int,m+1)
    for i:=range dp{
        dp[i]=make([]int,n+1)
    }
    dp[0][1]=1
    for i:=range m{
        for j:=range n{
            dp[i+1][j+1]=dp[i][j+1]+dp[i+1][j]
        }
    }
    return dp[m][n]
}
```



核心思想：把走格子变成“排座位”

我们要从左上角$(0, 0)$走到右下角$(m-1, n-1)$你仔细想一想，不管机器人怎么绕路，它**总共需要走多少步？**

- 为了到达最右边，它**必须且只能**向右走 $n-1$ 步。
- 为了到达最下边，它**必须且只能**向下走 $m-1$ 步。

所以，无论它选择什么路线，**总步数永远是固定的：$(m-1) + (n-1) = m+n-2$ 步。**

既然总步数是固定的，那所谓的“不同路径”，其实就是一个纯粹的数学组合问题：

**在这总共 $m+n-2$ 个步子里，我只需要挑出 $m-1$ 个步子用来“向下走”**（剩下的步子自然就全是“向右走”了）。

这就变成了一个高中数学里最经典的问题：从 $N$ 个空位中，挑出 $K$ 个位置，有几种挑法？

公式就是组合数：$C_{m+n-2}^{m-1}$。

这就是 `uniquePaths` 函数里直接 `return comb(m+n-2, m-1)` 的根本原因！



**在面试场上，千万不要一上来就写这个纯数学解法！**

```go
func comb(n, k int) int {
    k = min(k, n-k)
    res := 1
    for i := 1; i <= k; i++ {
        res = res * (n + 1 - i) / i
    }
    return res
}

func uniquePaths(m, n int) int {
    return comb(m+n-2, m-1)
}
```

#### 5.最长回文子串

专门对付回文串的绝杀技——**“中心扩展法” (Expand Around Center)**。

它不仅直观、好写，而且**不需要任何额外的数组，空间复杂度是完美的 $O(1)$**！

1. **🪞 核心思想：寻找“对称轴”**

什么是回文串？就是正着读和反着读都一样，这意味着它必然是**绝对对称**的。

 既然是对称的，那它一定有一个“中心对称轴”。我们只要遍历字符串，把每一个位置都当成一次“对称轴”，然后同时向左和向右扩展，看看能扩展多远不就行了？

2. **🕳️ 最大的坑：奇数与偶数的幽灵**

很多新手在写中心扩展时，会死在这个坑里：

- **奇数长度的回文串（如 `"aba"`）：** 它的中心是一个**字符**（`'b'`）。
- **偶数长度的回文串（如 `"abba"`）：** 它的中心是两个字符中间的**缝隙**！

##### 双指针+中心扩展

```go

func longestPalindrome(s string) string {
    res := ""
    for i , n := 0 , len(s) ; i < n ; i++{
        l , r := i , i
        for l >= 0 && r < n && s[l] == s[r]{
            l--
            r++
        }
        if r-l-1 > len(res){
            res = s[l+1 : r]
        }
        l , r = i , i+1
        for l >= 0 && r < n && s[l] == s[r]{
            l--
            r++
        }
        if r-l-1 > len(res){
            res = s[l+1 : r]
        }
    }
    return res
}
```

为了不漏掉任何一种情况，我们在遍历字符串 `s` 的每一个字符 `s[i]` 时，必须做**两次扩展尝试**：

1. 把 `s[i]` 当作单一中心，向两边扩展（去抓奇数长度的回文）。
2. 把 `s[i]` 和 `s[i+1]` 当作双中心，向两边扩展（去抓偶数长度的回文）。

```go
func longestPalindrome(s string) string {
    if len(s) < 2 {
        return s
    }

    start, end := 0, 0

    // 遍历每一个可能的中心点
    for i := 0; i < len(s); i++ {
        // 尝试以 s[i] 为中心（奇数长度）
        len1 := expandAroundCenter(s, i, i)
        // 尝试以 s[i] 和 s[i+1] 为中心（偶数长度）
        len2 := expandAroundCenter(s, i, i+1)

        // 取这两次尝试中，较长的那一个
        maxLen := max(len1, len2)

        // 如果找到了更长的回文串，更新全局记录的起始和结束索引
        if maxLen > end - start {
            // 这里是数学小技巧：通过中心点索引 i 和最大长度，反推回文串的边界
            start = i - (maxLen - 1) / 2
            end = i + maxLen / 2
        }
    }

    // 返回截取到的最长回文子串
    return s[start : end+1]
}

// 辅助函数：从给定的 left 和 right 中心向两边扩展，返回能扩展出的回文串长度
func expandAroundCenter(s string, left int, right int) int {
    // 只要没有越界，且左右字符相等，就继续向两边扩张
    for left >= 0 && right < len(s) && s[left] == s[right] {
        left--
        right++
    }
    // 注意：退出循环时，left 和 right 已经指向了不满足条件的字符
    // 所以实际的回文串长度是 (right - 1) - (left + 1) + 1 = right - left - 1
    return right - left - 1
}

func max(a, b int) int {
    if a > b { return a }
    return b
}
```

##### **马拉车算法 (Manacher's Algorithm)**

**1.  魔法一：奇偶统一（字符串预处理）**

我们前面在“中心扩展法”里吃过最大的亏，就是回文串有奇数和偶数长度之分，导致我们要扩展两次。

马拉车算法的第一步，就是用极其暴力的物理手段，**把所有回文串全部变成奇数长度！**

**怎么做？** 在字符串的首尾、以及每个字符之间，强行插入一个原本不存在的特殊字符（比如 `#`）。

为了防止后续扩展时越界，再在最前面和最后面加上两个不一样的边界字符（比如 `^` 和 `$`）。

举个栗子：

- `s = "aba"` (奇数，长度3) $\rightarrow$ 变成 `t = "^#a#b#a#$"` (长度9，中心是 `b`)
- `s = "abba"` (偶数，长度4) $\rightarrow$ 变成 `t = "^#a#b#b#a#$"` (长度11，中心是中间的 `#`)

**奇迹发生了：** 处理后，任何回文串的长度必然是奇数，并且都有一个明确的中心字符（要么是原字符，要么是 `#`）！

**2. 📏 魔法二：神奇的半径数组 `p`**

我们开辟一个和新字符串 `t` 一样长的数组 `p`。 

定义 `p[i]` 为：**以 `t[i]` 为中心的回文串的“半径”（不包括中心自己）。**

这里隐藏着一个惊天大秘密：**`p[i]` 的值，竟然极其精准地等于原字符串 `s` 中该回文串的总长度！**

- 比如 `^#a#b#a#$` 中，`b` 的索引是 4。
- 向两边扩展，最长回文是 `#a#b#a#`，半径是 3。
- 原回文 `"aba"` 的长度正好也是 3！

所以，求出了 `p` 数组的最大值，就等于求出了最终答案！

3. **魔法三：核心灵魂——“借力打力” (镜像映射)**

这是马拉车算法把时间复杂度降到 O(n) 的绝对核心。

假设我们正在从左往右计算 `p[i]`。

在这个过程中，我们用两个变量记录我们**目前触达过的最右边的回文边界**：

- `R`：目前所有已知回文串中，向右延伸到的最远边界的索引。
- `C`：那个触达 `R` 的回文串的中心点索引。

现在，我们走到了索引 `i`（且 `i < R`，说明 `i` 被包裹在那个大回文串里）。

因为以 `C` 为中心的大回文串是**绝对对称**的，那么 `i` 在左边一定有一个镜像兄弟 `j`！

通过坐标推导，镜像位置为：

$$j = 2C - i$$

既然是对称的，那 `i` 的回文半径 `p[i]`，是不是可以直接**抄袭**它镜像兄弟 `p[j]` 的答案？

是的！但有一个极其致命的前提：**抄答案不能越界！**

如果兄弟 `p[j]` 的范围太大了，导致 `i` 抄过来的半径超过了我们已知的最右边界 `R`，那 `R` 外面的世界是未知的，不能瞎抄。所以，我们只能抄 `R - i` 和 `p[j]` 里较小的那个。

这就是马拉车算法最著名的状态转移方程：

$$p[i] = \min(R - i, p[2C - i])$$

利用这个公式，我们给 `p[i]` 赋予了一个极其强悍的“保底初始值”，然后再继续用中心扩展法去向外试探。大部分情况下直接 O(1) 得出结果，这就是它快如闪电的秘密！

```go
func longestPalindrome(s string) string {
    if len(s) < 2 {
        return s
    }

    // 1. 预处理：插入特殊字符
    // 假设 s = "aba"，处理后 t = "^#a#b#a#$"
    t := "^"
    for _, char := range s {
        t += "#" + string(char)
    }
    t += "#$"

    n := len(t)
    p := make([]int, n) // 记录每个中心的回文半径
    C, R := 0, 0        // C: 最右回文的中心, R: 最右回文的右边界
    
    maxLen, centerIndex := 0, 0

    // 从 1 到 n-2 遍历，避开两端的 ^ 和 $
    for i := 1; i < n-1; i++ {
        // 2. 核心魔法：如果 i 在已知最右边界 R 内，利用镜像 j 借力打力
        if i < R {
            p[i] = min(R-i, p[2*C-i])
        }

        // 3. 踩在巨人的肩膀上，继续向外使用“中心扩展法”探路
        for t[i + p[i] + 1] == t[i - p[i] - 1] {
            p[i]++
        }

        // 4. 如果新计算的回文串突破了历史最右边界 R，更新 C 和 R
        if i + p[i] > R {
            C = i
            R = i + p[i]
        }

        // 5. 实时记录全局最长回文串的长度和中心点
        if p[i] > maxLen {
            maxLen = p[i]
            centerIndex = i
        }
    }

    // 6. 剥离伪装，推导原字符串中的起始位置
    // 因为加入了 #，原字符串的起始索引公式为 (centerIndex - maxLen) / 2
    start := (centerIndex - maxLen) / 2
    return s[start : start+maxLen]
}

func min(a, b int) int {
    if a < b { return a }
    return b
}
```

在 Go 语言里，在 `for` 循环中频繁使用 `+=` 拼接字符串，绝对是工程实践中的大忌。

**🚨 扒一扒底层：为什么 `+=` 这么坑？**

根本原因在于：**Go 语言中的字符串是不可变的 (Immutable)。**

每次你执行 `t += "#" + string(char)` 时，底层并不会在原来的 `t` 后面直接追加内容。相反，Go 运行时会：

1. **重新申请**一块更大的内存空间。
2. 把旧的 `t` 完整地**拷贝**过去。
3. 把新的字符**追加**到末尾。
4. 把旧的内存扔给垃圾回收器 (GC)。

如果原字符串 `s` 的长度是 $N$，在这个循环里，内存申请和拷贝的次数会累加，最终导致这一步的时间复杂度退化成了极其恐怖的 $O(N^2)$！

**🛠️ 资深 Gopher 的终极武器：`strings.Builder`**

在 Go 1.10 之后，官方引入了专为高效拼接字符串而生的神器：`strings.Builder`。它在底层维护了一个字节切片 (`[]byte`)，追加内容时只是在原内存上原地操作，几乎没有多余的内存分配。

```go
import "strings"

// ... 算法前半部分 ...

// 1. 极致优化：使用 strings.Builder 替代 +=
var builder strings.Builder

// 2. 巅峰细节：提前计算并分配好需要的精确内存容量
// 原长度 n，加上 n+1 个 '#'，加上开头 '^' 和结尾 '$'，总共 2*n + 3
builder.Grow(len(s)*2 + 3)

// 3. 开始高效组装
builder.WriteByte('^')
for i := 0; i < len(s); i++ {
    builder.WriteByte('#')
    builder.WriteByte(s[i]) // string 索引取出来本身就是 byte，连 string(char) 的强转都省了！
}
builder.WriteString("#$")

// 获取最终的极速拼接结果
t := builder.String()

// ... 继续马拉车核心逻辑 ...
```

**`builder.Grow()`**：直接向操作系统要足内存，杜绝了底层切片在追加过程中的任何扩容拷贝。

**`WriteByte`**：避免了 `string(char)` 带来的类型转换开销，直接在字节层面操作，快到飞起。



```go
import "strings"

func longestPalindrome(s string) string {
    // 边界条件：少于2个字符必定是回文
    if len(s) < 2 {
        return s
    }

    // 1. 预处理：使用 strings.Builder 高效拼接，舍弃繁琐的容量计算，保持代码清爽
    // 目标：将 "aba" 变成 "^#a#b#a#$"，强行统一奇偶长度
    var builder strings.Builder
    builder.WriteByte('^')
    for i := 0; i < len(s); i++ {
        builder.WriteByte('#')
        builder.WriteByte(s[i]) // string 索引取出来本身就是 byte，极其高效
    }
    builder.WriteString("#$")
    t := builder.String()

    // 2. 初始化核心变量
    n := len(t)
    p := make([]int, n) // p[i] 记录以 t[i] 为中心的最大回文半径
    C, R := 0, 0        // C: 触达最右边界的回文中心, R: 当前已知的最右边界
    
    maxLen := 0         // 记录全局最长回文串的长度 (等同于最大的 p[i])
    centerIndex := 0    // 记录全局最长回文串的中心点索引

    // 3. 开始马拉车核心逻辑 (从 1 遍历到 n-2，完美避开开头的 ^ 和结尾的 $)
    for i := 1; i < n-1; i++ {
        // 魔法一：借力打力
        // 如果当前考察的 i 被包裹在已知最右边界 R 内
        if i < R {
            // 找到 i 关于中心 C 的镜像对称点 j (j = 2*C - i)
            // p[i] 至少可以白嫖 p[j] 的长度，但绝不能越过右边界 R
            p[i] = min(R-i, p[2*C-i])
        }

        // 魔法二：中心扩展
        // 踩在保底的 p[i] 基础之上，继续向两边试探
        for t[i + p[i] + 1] == t[i - p[i] - 1] {
            p[i]++
        }

        // 魔法三：更新版图
        // 如果新扩展出的回文串突破了历史最右边界 R，立刻“迁都”，更新 C 和 R
        if i + p[i] > R {
            C = i
            R = i + p[i]
        }

        // 实时记录最高纪录
        if p[i] > maxLen {
            maxLen = p[i]
            centerIndex = i
        }
    }

    // 4. 剥离伪装，还原回原字符串
    // 预处理后的中心索引和最大长度，可以通过简单的数学关系还原出原字符串的起始索引
    start := (centerIndex - maxLen) / 2
    return s[start : start+maxLen]
}

// 辅助函数：求最小值
func min(a, b int) int {
    if a < b {
        return a
    }
    return b
}
```

#### 1143.最长公共子序列

给定两个字符串 `text1` 和 `text2`，返回这两个字符串的最长 **公共子序列** 的长度。如果不存在 **公共子序列** ，返回 `0` 。

一个字符串的 **子序列** 是指这样一个新的字符串：它是由原字符串在不改变字符的相对顺序的情况下删除某些字符（也可以不删除任何字符）后组成的新字符串。

- 例如，`"ace"` 是 `"abcde"` 的子序列，但 `"aec"` 不是 `"abcde"` 的子序列。

两个字符串的 **公共子序列** 是这两个字符串所共同拥有的子序列。

处理两个字符串的匹配问题，最标准的姿势就是建一个二维矩阵。 设 `text1` 的长度为 $m$，`text2` 的长度为 $n$。

我们定义 $dp[i][j]$ 为：**`text1` 的前 $i$ 个字符，和 `text2` 的前 $j$ 个字符，它们的最长公共子序列的长度。**

(注意细节：为了处理空字符串的边界情况，我们通常把 `dp` 数组的大小设为 $(m+1) \times (n+1)$。其中 $dp[0][j]$ 和 $dp[i][0]$ 都代表其中一个是空串，公共长度自然全是 0。)

🕵️‍♂️ 灵魂拷问：状态转移方程

假设我们现在正盯着 `text1` 的第 $i$ 个字符（索引为 `i-1`）和 `text2` 的第 $j$ 个字符（索引为 `j-1`），想要计算 $dp[i][j]$。

我们只面临两种非黑即白的情况：

**情况 A：两个字符竟然一样！(`text1[i-1] == text2[j-1]`)**

简直是天作之合！既然这俩字符长得一样，那它们绝对可以加入到“公共子序列”的队伍里。

那它们加入之前的队伍有多长呢？就是这两者都各退一步的历史记录：$dp[i-1][j-1]$。

所以作案公式：

$$dp[i][j] = dp[i-1][j-1] + 1$$



**情况 B：两个字符不一样！**

没对上眼，说明这两个字符**绝对不可能同时**出现在当前的最长公共子序列的末尾。

那怎么办？我们只能“甩锅”，兵分两路去历史记录里找最高分：

1. 抛弃 `text1` 的当前字符，看看 `text1` 的前 $i-1$ 个字符和 `text2` 的前 $j$ 个字符能凑出多长（即 $dp[i-1][j]$）。

2. 抛弃 `text2` 的当前字符，看看 `text1` 的前 $i$ 个字符和 `text2` 的前 $j-1$ 个字符能凑出多长（即 $dp[i][j-1]$）。

	我们作为精打细算的军师，当然是取这两条路里的最大值：

	$$dp[i][j] = \max(dp[i-1][j], dp[i][j-1])$$

```go
func longestCommonSubsequence(text1 string, text2 string) int {
    m, n := len(text1), len(text2)
    
    // 1. 初始化 (m+1) x (n+1) 的二维 DP 表格
    // Go 中 make 出来的二维切片默认全为 0，完美契合空串边界条件
    dp := make([][]int, m+1)
    for i := range dp {
        dp[i] = make([]int, n+1)
    }
    
    // 2. 双层循环，从左到右、从上到下填表
    for i := 1; i <= m; i++ {
        for j := 1; j <= n; j++ {
            if text1[i-1] == text2[j-1] {
                // 字符相同，继承左上角的历史记录并 +1
                dp[i][j] = dp[i-1][j-1] + 1
            } else {
                // 字符不同，取正上方和正左方的最大值
                dp[i][j] = max(dp[i-1][j], dp[i][j-1])
            }
        }
    }
    
    // 3. 表格右下角的最终值，就是两支大军完整比对后的最长长度
    return dp[m][n]
}

func max(a, b int) int {
    if a > b { return a }
    return b
}
```

##### 优化 字节大模型算法 二面（要求时间小于o(mn)）

```go
func longestCommonSubsequence(text1 string, text2 string) int {
    n := len(text2)
    // 一维滚动数组：f[j+1] 代表匹配到 text2 的第 j 个字符时的最长公共子序列长度
    // 容量设为 n+1 是为了处理空字符串的边界情况（默认全为 0）
    f := make([]int, n + 1)

    // 外层循环：逐个引入 text1 的字符 s (相当于二维表格里的“行 i”)
    for _, s := range text1 {
        // 极其核心的变量 pre！
        // 每开始新的一行，左上角的初始值必定是 0（因为和空串匹配长度为 0）
        // pre 永远用来保存二维表格里的“左上方”数据 (即 dp[i-1][j-1])
        pre := 0 
        
        // 内层循环：逐个引入 text2 的字符 t (相当于二维表格里的“列 j”)
        for j, t := range text2 {
            // 抢救现场：
            // 此时的 f[j+1] 还没有被当前行更新，所以它里面存的还是“上一行”的数据
            // 也就是二维里的“正上方” (dp[i-1][j])。
            // 我们必须把它存进 tmp 里，因为等会儿 f[j+1] 就要被新数据覆盖了！
            tmp := f[j+1] 
            
            if s == t {
                // 情况 A：字符相同。
                // 长度 = 左上方的历史记录 + 1。
                // 此时 pre 里存的刚好就是“左上方”的数据！
                f[j+1] = pre + 1
            } else {
                // 情况 B：字符不同。
                // 长度 = max(正上方, 正左方)
                // f[j+1] 还没被覆盖，所以它就是“正上方”
                // f[j] 在前一轮循环已经被更新过了，所以它是当前行的“正左方”
                f[j+1] = max(f[j+1], f[j]) 
            }
            
            // 完美闭环：
            // 当前格子计算完了，准备进入下一轮循环（向右走一格）。
            // 刚才抢救下来的 tmp（当前格子的正上方），对于下一个格子来说，不就变成了“左上方”吗！
            // 所以把它赋值给 pre，完美传递给下一次循环使用。
            pre = tmp 
        }
    }

    // 整个表格(虽然只有一维)填完后，最后一个格子的值就是最终答案
    return f[n]
}

// 辅助函数
func max(a, b int) int {
    if a > b { return a }
    return b
}
```

**场景重现：军师带你“铺地砖”**

想象你面前只有**一排**旧地砖（这就是我们的一维数组 `f`）。

这排旧地砖上，写着**上一行**计算出来的答案。

现在，你要从左到右，一块一块地把这排旧地砖，**重新刷上当前行的新油漆（新答案）**。

假设你现在刚好走到第 `j` 块地砖面前，准备动手刷漆。你手里拿着刷子，面临一个极其尴尬的局面：

根据我们的 LCS 规则，你要算出第 `j` 块地砖的新颜色，你必须同时低头看**三个旧数据**：

1. **正左方的地砖：** 这个好办，你刚刚刷完前一块地砖，前一块的新颜色就是“正左方”。
2. **正上方的地砖：** 这个也好办，你脚下这块地砖**还没被你刷上新漆**，所以它上面写的旧数字，就是“正上方”。
3. **💥 左上方的地砖：** **灾难来了！** 什么是“左上方”？就是你刚刚刷完的那块“正左方”地砖**在被你刷漆之前的旧样子**！但是，它刚刚已经被你无情地刷上新漆了！旧数据被永久抹除了！

发现问题所在了吗？**如果你直接一路刷过去，你的“左上方”数据永远会丢失！**

pre` 和 `tmp 的魔法：左手倒右手

为了防止“左上方”数据丢失，我们必须引入两个临时工：`pre` 和 `tmp`。

你可以把 `pre` 想象成你**左手一直攥着的一张小纸条**。 把 `tmp` 想象成你**右手用来抄写的一支笔**。

现在，我们把你在第 `j` 块地砖面前的动作放慢一万倍：

1. **抢救旧数据（`tmp := f[j+1]`）**： 在你的刷子落下去之前，你用右手（`tmp`），把脚下这块地砖的**旧数字**赶紧抄下来。 *(此时，`tmp` 里存着这块地砖的旧样子。)*
2. **大胆刷漆（`f[j+1] = ...`）**： 现在旧数字已经备份了，你可以放心大胆地用公式去计算新数字，并且一刷子盖在脚下的地砖上。 *(在计算公式里，如果你遇到字符相等，需要用到“左上方”数据，你直接低头看你左手攥着的纸条 `pre`，里面绝对是你想要的！)*
3. **魔法交接（`pre = tmp`）**： 脚下的地砖刷完了，准备往右跨一步去刷下一块了。 在跨步之前，**你把右手抄着旧数字的纸条，塞进左手里！** *(奇迹发生了：当你向右跨出一步后，刚才脚下的这块旧地砖，是不是刚好变成了你现在的“左上方”？而它被刷漆前的旧模样，刚好就攥在你的左手 `pre` 里！)*

🎬 极简复盘

- **`f[j+1]`**：永远是被刷漆的那块砖（刷之前是“正上方”，刷之后是“正左方”）。
- **`tmp`**：永远在刷漆前，赶紧把“正上方”的遗照拍下来。
- **`pre`**：永远是手里攥着的前一块砖的遗照。当你走到下一块砖时，这张遗照就成了救命的“左上方”。

怎么样？把这个过程想象成**“拍照留念 -> 泼油漆 -> 把照片递给下一块砖”**，那个看起来极其反人类的 `tmp` 和 `pre` 是不是瞬间就活过来了？

#### 72. 编辑距离

给你两个单词 `word1` 和 `word2`， *请返回将 `word1` 转换成 `word2` 所使用的最少操作数* 。

你可以对一个单词进行如下三种操作：

- 插入一个字符
- 删除一个字符
- 替换一个字符

老规矩，对比两个字符串，直接建一个二维表格。 我们定义 $dp[i][j]$ 为：**将 `word1` 的前 $i$ 个字符，转换成 `word2` 的前 $j$ 个字符，所需要使用的【最少操作数】。**

我们的最终目标，就是算出表格最右下角的值：$dp[m][n]$。

**夯实地基：处理边界（第 0 行和第 0 列）**

在这道题里，边界初始化极其重要，也非常符合现实直觉：

- **如果 `word2` 是空串（第 0 列）：** 把 `word1` 的前 $i$ 个字符变成空串，你需要几步？当然是把它们**全删掉**，需要 $i$ 步！所以 $dp[i][0] = i$。
- **如果 `word1` 是空串（第 0 行）：** 把空串变成 `word2` 的前 $j$ 个字符，你需要几步？当然是老老实实**全插入**进去，需要 $j$ 步！所以 $dp[0][j] = j$。

这就好比我们在表格的最上面和最左边，画好了两排标尺。

**兵法推演：状态转移方程（核心灵魂）**

现在我们站在格子里，盯着 `word1` 的第 $i$ 个字符和 `word2` 的第 $j$ 个字符。我们只面临两种命运：

**情况 A：两个字符一模一样！**

这简直是天上掉馅饼！既然这俩字符已经匹配了，我们**根本不需要做任何操作**。

所以，当前的最少操作数，直接等于它们俩还没加入战场时的操作数（也就是左上方的数据）：

$$dp[i][j] = dp[i-1][j-1]$$

**情况 B：两个字符不一样！**

麻烦来了，必须要动手了。题目给了我们三种武器，我们来看看这三种武器在二维表格里分别意味着什么：

1. **替换 (Replace)：** 如果我们把 `word1` 的第 $i$ 个字符**强行替换**成 `word2` 的第 $j$ 个字符，那它俩就匹配了。操作数 = 替换这一步操作 (1) + 替换前它俩前面的状态（左上方）：$dp[i-1][j-1] + 1$。
2. **删除 (Delete)：** 如果我们觉得 `word1` 的这个字符太碍事，直接把它**删了**。那我们就得指望 `word1` 剩下的前 $i-1$ 个字符能匹配上 `word2` 的前 $j$ 个字符。操作数 = 删除这一步操作 (1) + 正上方的状态：$dp[i-1][j] + 1$。
3. **插入 (Insert)：** 如果我们在 `word1` 后面**凭空插入**一个和 `word2` 第 $j$ 个字符一样的字符，那最后一个字符就搞定了。接下来的任务是让 `word1` 的前 $i$ 个字符去匹配 `word2` 剩下的前 $j-1$ 个字符。操作数 = 插入这一步操作 (1) + 正左方的状态：$dp[i][j-1] + 1$。

作为精打细算的军师，我们面临不匹配时，当然是把这三种武器都试一遍，然后**选代价最小（操作数最少）的那个**！

作案公式华丽诞生：

$$dp[i][j] = \min(dp[i-1][j-1], dp[i-1][j], dp[i][j-1]) + 1$$

```go
func minDistance(word1 string, word2 string) int {
    m, n := len(word1), len(word2)
    
    // 1. 初始化二维 DP 表格
    dp := make([][]int, m+1)
    for i := range dp {
        dp[i] = make([]int, n+1)
    }
    
    // 2. 夯实地基：处理第一列（删掉 word1 的所有字符变成空串）
    for i := 0; i <= m; i++ {
        dp[i][0] = i
    }
    // 夯实地基：处理第一行（在空串里不断插入变成 word2）
    for j := 0; j <= n; j++ {
        dp[0][j] = j
    }
    
    // 3. 开始填表大业
    for i := 1; i <= m; i++ {
        for j := 1; j <= n; j++ {
            if word1[i-1] == word2[j-1] {
                // 字符相同，躺平，直接继承左上方代价
                dp[i][j] = dp[i-1][j-1]
            } else {
                // 字符不同，在替换(左上)、删除(正上)、插入(正左)中找个最便宜的，加上 1 步操作费
                dp[i][j] = min(dp[i-1][j-1], min(dp[i-1][j], dp[i][j-1])) + 1
            }
        }
    }
    
    return dp[m][n]
}

// Go 语言里只能传两个参数的 min 辅助函数
func min(a, b int) int {
    if a < b { return a }
    return b
}
```



### 技巧

#### 75.颜色分类

给定一个包含红色、白色和蓝色、共 `n` 个元素的数组 `nums` ，**[原地](https://baike.baidu.com/item/原地算法)** 对它们进行排序，使得**相同颜色的元素相邻**，并按照**红色、白色、蓝色**顺序排列。

我们使用整数 `0`、 `1` 和 `2` 分别表示红色、白色和蓝色。

必须在不使用库内置的 sort 函数的情况下解决这个问题。

**1. 核心破局思路：三足鼎立**

想象数组是一个长长的走廊，我们要把 0 赶到最左边，把 2 赶到最右边，剩下的 1 自然就被挤在中间了。

我们需要三个指针：

- **`p0` (左边界)**：用来安放 `0`。它指向的位置，表示“下一个 `0` 应该被换到这里”。
- **`p2` (右边界)**：用来安放 `2`。它指向的位置，表示“下一个 `2` 应该被换到这里”。
- **`curr` (游标)**：这是我们的探路先锋，从左到右遍历整个数组。

**2. 游标探路的三条铁律**

当游标 `curr` 遇到不同的数字时，我们执行极其严格的纪律：

1. **如果遇到 `0`（该去左边）**：

	​	把 `curr` 和 `p0` 指向的数字**交换**。 

	​	因为 0 安置妥当了，`p0` 向右走一步准备接客（`p0++`）。 

	​	游标 `curr` 也向右走一步继续探路（`curr++`）。

2. **如果遇到 `1`（本来就该在中间）**：

	​	 这是最省事的，不管它，游标 `curr` 直接向右走一步（`curr++`）。

3. **🚨 如果遇到 `2`（该去右边，这里有致命陷阱！）**： 

	​	把 `curr` 和 `p2` 指向的数字**交换**。 

	​	因为 2 安置妥当了，`p2` 向左收缩一步（`p2--`）。 

	​	**注意！此时游标 `curr` 绝对不能动！** 为什么？

	​	因为你刚从 `p2` 那里换过来的数字，是个盲盒（它可能是 0，也可能是 1，也可能是 2），你还没对它进行检查呢！

	​	必须留在原地，下一轮循环再审问它一次。

```go
func sortColors(nums []int) {
    p0 := 0              // p0 初始化在最左头
    p2 := len(nums) - 1  // p2 初始化在最右头
    curr := 0            // 游标从 0 开始探路
    
    // 游标只要越过了右边界 p2，说明剩下的全是被安置好的 2，战斗结束
    for curr <= p2 {
        if nums[curr] == 0 {
            // 遇到 0：和 p0 交换，两人双双往前走
            nums[curr], nums[p0] = nums[p0], nums[curr]
            p0++
            curr++
        } else if nums[curr] == 2 {
            // 遇到 2：和 p2 交换，p2 往左退一步
            // 🚨 极其关键：curr 停在原地不动，继续审问换过来的新数字！
            nums[curr], nums[p2] = nums[p2], nums[curr]
            p2--
        } else {
            // 遇到 1：直接无视，游标继续往前看
            curr++
        }
    }
}
```

```
[2, 0, 2, 1, 1, 0]


【初始状态】
[ 2,  0,  2,  1,  1,  0 ]
↑                       ↑
p0,curr                p2

【第一轮结束】（2和0交换）
[ 0,  0,  2,  1,  1,  2 ]
↑                 ↑
p0,curr           p2
(解说：
最右边的 2 已经安置妥当，p2 退到了索引 4。
curr 留在原地准备审问新换过来的 0。)
```

#### 31.下一个排列

整数数组的一个 **排列** 就是将其所有成员以序列或线性顺序排列。

- 例如，`arr = [1,2,3]` ，以下这些都可以视作 `arr` 的排列：`[1,2,3]`、`[1,3,2]`、`[3,1,2]`、`[2,3,1]` 。

整数数组的 **下一个排列** 是指其整数的下一个字典序更大的排列。更正式地，如果数组的所有排列根据其字典顺序从小到大排列在一个容器中，那么数组的 **下一个排列** 就是在这个有序容器中排在它后面的那个排列。如果不存在下一个更大的排列，那么这个数组必须重排为字典序最小的排列（即，其元素按升序排列）。

- 例如，`arr = [1,2,3]` 的下一个排列是 `[1,3,2]` 。
- 类似地，`arr = [2,3,1]` 的下一个排列是 `[3,1,2]` 。
- 而 `arr = [3,2,1]` 的下一个排列是 `[1,2,3]` ，因为 `[3,2,1]` 不存在一个字典序更大的排列。

给你一个整数数组 `nums` ，找出 `nums` 的下一个排列。

必须**[ 原地 ](https://baike.baidu.com/item/原地算法)**修改，只允许使用额外常数空间。



想象你在拨动一个密码锁。当前密码是 `123654`。 你想找到比它**大**，但又是**最接近它**的下一个密码（这就是“下一个字典序”）。

- **原则一：尽量别动前面的高位。** 如果你把第一位的 `1` 换成 `2`，那数字就变太大了！我们应该**从后往前**找，尽量修改低位。
- **原则二：把后面的“大数字”换到前面来。** 但是，如果你看最后三位 `654`，它们已经是**降序**排列了。降序意味着这三个数字组合出的已经是最大的可能，没法更大了！
- **原则三：找到“拐点”。** 我们必须越过 `654` 这个降序区间，往前看。哎！发现前面的 `3` 比 `6` 小（`3 < 6`）。这个 `3` 就是我们要替换的**“拐点”**！



🗺️ 军师的“四步走”战术

我们继续拿 `[1, 2, 3, 6, 5, 4]` 这个数组来推演：

**第一步：从后往前找“拐点” `i`。

 我们从右往左看，寻找第一个出现“左边小于右边”的位置。 

`4 -> 5 -> 6` 都是一路走高的（降序）。 

直到遇到 `3`，因为 `3 < 6`，打破了降序规律。 

**锁定拐点：`nums[i] = 3`（索引为 2）。**

**第二步：从后往前找“替补” `j`。

 我们要把 `3` 换掉，换成谁呢？

既然要变大，必须找一个比 `3` 大的数。

但为了“大得尽可能少”，我们要从后往前找**第一个**比 `3` 大的数。 

从最后看：`4` 比 `3` 大！ **锁定替补：`nums[j] = 4`（索引为 5）。**

**第三步：两人互换位置 (Swap)。** 把 `3` 和 `4` 交换。 数组变成了：`[1, 2, 4, 6, 5, 3]`。

此时，前三位已经变成了 `124`，确实比原来的 `123` 大了。

**第四步：将尾部“降序”翻转为“升序” (Reverse)。** 你看交换后的尾部 `[6, 5, 3]`，它依然是降序的，这是当前组合下**最大**的排列。 

但我们要找的是**刚好处在下一步**的最小排列。怎么办？ 直接把这个尾部**反转**，变成升序 `[3, 5, 6]`！ 最终结果：`[1, 2, 4, 3, 5, 6]`。完美收官！

*(注：如果第一步走到头都没找到拐点，说明整个数组完全降序，比如 `[3, 2, 1]`，那它已经是最大的了，直接跳到第四步把整个数组反转成 `[1, 2, 3]` 即可。)*



```go
func nextPermutation(nums []int) {
    n := len(nums)
    if n <= 1 {
        return
    }

    // 第一步：从后往前找“拐点” i
    i := n - 2
    for i >= 0 && nums[i] >= nums[i+1] {
        i--
    }

    // 如果 i >= 0，说明找到了拐点，不是全降序的最末排列
    if i >= 0 {
        // 第二步：从后往前找“替补” j
        j := n - 1
        for j >= 0 && nums[j] <= nums[i] {
            j--
        }
        // 第三步：交换拐点和替补
        nums[i], nums[j] = nums[j], nums[i]
    }

    // 第四步：将拐点之后的区间反转（如果是全降序，这里会反转整个数组）
    reverse(nums, i+1)
}

// 辅助函数：双指针反转切片指定区间
func reverse(nums []int, start int) {
    left, right := start, len(nums)-1
    for left < right {
        nums[left], nums[right] = nums[right], nums[left]
        left++
        right--
    }
}
```

库函数：

```go
import "slices"
func nextPermutation(nums []int)  {
    n:=len(nums)
    i:=n-2
    for i>=0 &&nums[i]>=nums[i+1] {
        i--
    }
    if i>=0 {
        j:=n-1
        for nums[j]<=nums[i]{
            j--
        }
        nums[i],nums[j]=nums[j],nums[i]
    }
    slices.Reverse(nums[i+1:])
}
```

#### 寻找重复数

给定一个包含 `n + 1` 个整数的数组 `nums` ，其数字都在 `[1, n]` 范围内（包括 `1` 和 `n`），可知至少存在一个重复的整数。

假设 `nums` 只有 **一个重复的整数** ，返回 **这个重复的数** 。

你设计的解决方案必须 **不修改** 数组 `nums` 且只用常量级 `O(1)` 的额外空间。



这道题的标准答案，竟然是借用**链表**里的顶级神技——**“快慢指针（龟兔赛跑算法 / Floyd 判圈算法）”**

1. **视觉欺骗：把数组变成“链表”**

你可能会问：这明明是个数组，哪来的链表？

这道题有一个极其特殊的条件：**数组有 $n+1$ 个数字，且都在 $[1, n]$ 范围内。**

这意味着，如果我们把**数组的索引**当成链表的节点，把**数组里的值**当成指向下一个节点的指针（即 `next` 指针），这个网络绝对不会越界！

更致命的是，**因为有重复的数字，意味着必定有两个不同的索引，指向了同一个值！** 多条路汇聚到了同一个节点，这说明什么？**这说明链表里一定有环 (Cycle)！而那个环的入口，必定就是我们要找的重复数字！**



2. 🐢🐇 军师战术：龟兔赛跑两步走

既然变成了“链表找环入口”的问题（和 LeetCode 142 题一模一样），我们就直接祭出快慢指针：

**第一阶段：在环中相遇 (判断有环)**

- 让乌龟（慢指针 `slow`）每次走一步：`slow = nums[slow]`
- 让兔子（快指针 `fast`）每次走两步：`fast = nums[nums[fast]]`
- 因为有环，跑得快的兔子必定会在环里的某个位置，从后面追上并狠狠撞上乌龟（`slow == fast`）。

**第二阶段：寻找环的入口 (精准定位重复数)**

- 当它们相遇时，我们施展一个极其优美的数学魔法：把乌龟瞬间传送回起点（`slow = 0`），兔子停在相遇点原地待命。
- 然后，**让乌龟和兔子同时以相同的速度（每次只走一步）往前跑**。
- 它们下一次相遇的地方，**绝对、必定、正好是环的入口！**（这背后有着严谨的相对运动数学推导，在面试时只要能说出这个结论，面试官就懂你是在行家了）。、

```go
func findDuplicate(nums []int) int {
    // 1. 第一阶段：龟兔赛跑，找到环中的相遇点
    slow := nums[0]
    fast := nums[nums[0]]
    
    // 只要没相遇，就一直跑
    for slow != fast {
        slow = nums[slow]         // 乌龟走一步
        fast = nums[nums[fast]]   // 兔子走两步
    }
    
    // 2. 第二阶段：寻找环的入口（即重复的数字）
    // 把乌龟揪回起点，兔子原地不动
    slow = 0
    
    // 此时乌龟和兔子同速前进，每次走一步
    for slow != fast {
        slow = nums[slow]
        fast = nums[fast]
    }
    
    // 再次相遇点，就是我们要找的罪魁祸首！
    return slow
}
```

