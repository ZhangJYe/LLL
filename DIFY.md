# DIFY

核心目标：

1. **实操**：把项目中的 RAG 模块从“手写 Redis”平滑迁移到“Dify API”。

2. **面试准备**：

	在每一步操作中，我会穿插讲解 **RAG 核心概念**（Embedding、Chunking、Vector DB、Recall/Precision）和 **Dify 架构**，让你不仅能改代码，还能在面试时讲出为什么要这么改。

### 第一步：理解现状与痛点 (RAG 原理篇)

**面试题 1：什么是 RAG？你现在的代码实现了 RAG 的哪一部分？**

**概念**：RAG (Retrieval-Augmented Generation) **检索增强生成**。简单说就是考试（User Question）允许带书（Knowledge Base）。

1. **R (Retrieval)**：先去书里找相关段落。
2. **A (Augmented)**：把找到的段落贴在试卷上。
3. **G (Generation)**：LLM 根据试卷和段落写答案。

代码对应：

- **Embedding (向量化)**：

	embeddingArk，把文字变成数字列表（向量），为了让计算机算出哪两句话意思相近。

- **Vector DB (向量库)**：

	redisIndexer，存向量的地方。

- **Retrieval (检索)**：

	 RetrieveDocuments，用余弦相似度算出 Top K 最像的段落。

为什么使用dify？

手写 RAG 最大的短板：**缺少 Chunking (切片)**。

- 如果你传了一本 500 页的书，Embedding API 可能会报错（Token 超限），或者检索回来的内容太多，LLM 记不住。
- **Dify 的价值**：它可以自动帮你把 PDF/Word 拆成 500 个小段落，还能做数据清洗（去空行、去乱码）。

### 第二步：准备 Dify 环境 (实操篇)

我们需要一个 Dify 实例。

1. **方案 A (推荐)**：使用 Dify 云端版 (dify.ai)。注册账号，创建一个应用。
2. **方案 B**：本地 Docker 部署 Dify (如果你有 Docker 环境)。

### 第三步：定义新的 Dify 结构体 (代码篇)

我们先不动旧代码，在 common/aihelper/ 下新建一个 dify_model.go。这叫“增量式重构”，风险最小。

**面试知识点：适配器模式 (Adapter Pattern)** 你现有的 AIModel接口（model.go 第 26 行）定义了 GenerateResponse 和 StreamResponse。 我们现在要写一个 DifyModel 来实现这个接口。

这体现了 Go 语言面向接口编程的优势：业务层（Controller）不需要知道底层是 OpenAI 还是 Dify，只要实现了接口就行。

将 **Dify**（作为一个不仅包含 LLM，还可能包含知识库 RAG、工作流编排的应用）封装成 Eino 框架可以理解的 `Model` 组件。

#### dify_model.go 核心功能总结

1. **适配器模式**：它实现了 Eino 中定义的类似 `ChatModel` 的接口（虽然代码中未显示接口定义，但从 `schema.Message` 和方法签名可以看出）。
2. **API 调用**：它负责将 Go 程序的请求转换成 HTTP 请求，发送给 Dify 的 `/v1/chat-messages` 接口。
3. **RAG 的黑盒化**：你提到的“使用 Dify 进行 RAG”，在这个代码层面体现为 **Query 透传**。
	- Go 代码只负责发送 `Query`（用户问题）。
	- Dify 收到问题后，在 Dify 平台内部进行 **知识库检索 (Retrieval) -> 上下文组装 -> LLM 生成**。
	- Go 代码接收最终生成的答案。
4. **双模式支持**：同时支持 **阻塞式（Blocking）** 等待完整回复和 **流式（Streaming）** 逐字输出。

**common/aihelper/dify_model.go：**

```go
package aihelper

import (
	"bufio"
	"bytes"
	"context"
	"encoding/json"
	"fmt"
	"io"
	"net/http"
	"strings"

	"GopherAI/config"

    // 引入 Eino 的基础 Schema，主要是为了使用 Message 结构体
	"github.com/cloudwego/eino/schema"
)

// DifyModel 定义了 Dify 模型的客户端结构
// 它通常需要实现 Eino 的 ChatModel 接口
type DifyModel struct {
	apiKey  string       // Dify 应用的 API Key
	baseUrl string       // Dify 部署地址，SaaS 版为 https://api.dify.ai/v1
	client  *http.Client // 用于发送 HTTP 请求的标准客户端
}

// DifyChatRequest 是 Dify 官方 API 要求的 JSON 请求格式
// 参考文档: https://docs.dify.ai/api-reference/chat-messages
type DifyChatRequest struct {
    // Inputs 用于传递 Dify 编排中定义的变量（如 Prompt 中的变量）
	Inputs         map[string]interface{} `json:"inputs"`
    // Query 是用户的实际提问
	Query          string                 `json:"query"`
    // ResponseMode 决定是等待全部生成完再返回("blocking")，还是打字机效果("streaming")
	ResponseMode   string                 `json:"response_mode"` // "streaming" or "blocking"
    // ConversationID 用于保持多轮对话上下文，如果是新对话则留空
	ConversationID string                 `json:"conversation_id,omitempty"`
    // User 是终端用户的唯一标识，用于 Dify 后台的统计和限制
	User           string                 `json:"user"`
	// Files 字段用于支持多模态（如上传图片让 AI 分析）或文档上传
	Files []interface{} `json:"files,omitempty"`
}


// NewDifyModel 是构造函数，负责初始化配置
func NewDifyModel(ctx context.Context) (*DifyModel, error) {
	// 从 Config 中读取配置
	conf := config.GetConfig()
	apiKey := conf.DifyConfig.DifyApiKey
	baseUrl := conf.DifyConfig.DifyBaseUrl
    // 设置默认的 Dify SaaS 地址，如果是私有部署需要配置为自己的地址
	if baseUrl == "" {
		baseUrl = "https://api.dify.ai/v1"
	}

	return &DifyModel{
		apiKey:  apiKey,
		baseUrl: baseUrl,
		client:  &http.Client{},
	}, nil
}

// GenerateResponse 实现了非流式聊天（同步等待结果）
// 对应 Eino 中 ChatModel 的 Generate 方法
func (d *DifyModel) GenerateResponse(ctx context.Context, messages []*schema.Message) (*schema.Message, error) {
	if len(messages) == 0 {
		return nil, fmt.Errorf("no messages provided")
	}
	// 提取用户最新的问题
    // 注意：Dify 的 API 主要关注当前的 Query，历史上下文通常由 Dify 的 ConversationID 管理
    // 这里简单粗暴地取了最后一条消息作为 Query	
	lastMsg := messages[len(messages)-1].Content

	// 构建请求
	reqBody := DifyChatRequest{
		Inputs:       map[string]interface{}{}, // 如果你的 Dify 应用有设置变量（如 name, topic），需要在这里填充
		Query:        lastMsg,
		ResponseMode: "blocking",     // 阻塞模式：等待 AI 完全生成后一次性返回
		User:         "api-user-123", // TODO: 实际生产中，这里应该透传真实用户的 ID
	}

	jsonData, _ := json.Marshal(reqBody)
	// 拼接 URL：baseUrl +/chat-messages（dify的对话接口）
	req, _ := http.NewRequestWithContext(ctx, "POST", d.baseUrl+"/chat-messages", bytes.NewBuffer(jsonData))
    //设置必要的Header
	req.Header.Set("Authorization", "Bearer "+d.apiKey)
	req.Header.Set("Content-Type", "application/json")

	resp, err := d.client.Do(req)
	if err != nil {
		return nil, err
	}
	defer resp.Body.Close()

	// 读取响应错误
	bodyBytes, _ := io.ReadAll(resp.Body)
	if resp.StatusCode != 200 {
		return nil, fmt.Errorf("dify api error: %d, body: %s", resp.StatusCode, string(bodyBytes))
	}

	// 解析 Dify 返回的 JSON 结构
    // 成功响应通常包含: {"event": "message", "answer": "AI的回复...", "conversation_id": "..."}
	var result map[string]interface{}
	if err := json.Unmarshal(bodyBytes, &result); err != nil {
		return nil, fmt.Errorf("failed to unmarshal dify response: %w", err)
	}

    //提取对话内容
	answer, ok := result["answer"].(string)
	if !ok {
		return nil, fmt.Errorf("dify response missing 'answer' field")
	}

    //封装回 Eino 的 Message 结构
	return &schema.Message{Content: answer, Role: schema.Assistant}, nil
}

// StreamResponse 流式生成（对应 HTTP SSE 模式）
func (d *DifyModel) StreamResponse(ctx context.Context, messages []*schema.Message, cb StreamCallback) (string, error) {
	if len(messages) == 0 {
		return "", fmt.Errorf("no messages provided")
	}
	lastMsg := messages[len(messages)-1].Content

	reqBody := DifyChatRequest{
		Inputs:       map[string]interface{}{},
		Query:        lastMsg,
		ResponseMode: "streaming",
		User:         "api-user-123",
	}

	jsonData, _ := json.Marshal(reqBody)
	req, _ := http.NewRequestWithContext(ctx, "POST", d.baseUrl+"/chat-messages", bytes.NewBuffer(jsonData))
	req.Header.Set("Authorization", "Bearer "+d.apiKey)
	req.Header.Set("Content-Type", "application/json")

	resp, err := d.client.Do(req)
	if err != nil {
		return "", err
	}
	defer resp.Body.Close()

	reader := bufio.NewReader(resp.Body)
	var fullResp strings.Builder

	for {
		line, err := reader.ReadString('\n')
		if err != nil {
			if err == io.EOF {
				break
			}
			return "", err
		}

		// Dify 流式返回格式: data: {...}
		line = strings.TrimSpace(line)
		if strings.HasPrefix(line, "data:") {
			dataContent := strings.TrimPrefix(line, "data:")
			// 解析 JSON
			var chunk map[string]interface{}
			if err := json.Unmarshal([]byte(dataContent), &chunk); err != nil {
				continue
			}

			// 处理 message 事件，获取 answer 字段
			if event, ok := chunk["event"].(string); ok && event == "message" {
				if answer, ok := chunk["answer"].(string); ok {
					cb(answer)
					fullResp.WriteString(answer)
				}
			}
		}
	}

	return fullResp.String(), nil
}

func (d *DifyModel) GetModelType() string { return "5" }
```

既然你的目标是使用 Dify 做 RAG，这段代码是整个链路的 **"入口"**。

1. **RAG 发生在哪里？**
	- RAG 逻辑（文档切片、向量检索、重排序）**不在** 这段 Go 代码里。
	- RAG 逻辑是在 **Dify 的控制台** 里配置的。你需要在 Dify 创建一个“聊天助手”应用，并在“知识库（Context）”中关联你的文档。
2. **Go 代码需要改动什么？**
	- 目前的 `Inputs: map[string]interface{}{}` 是空的。如果你的 Dify RAG 应用配置了提示词变量（例如：用户要求以特定的风格回答，或者需要传入特定的业务 ID 来过滤知识库），你需要将这些参数填入 `Inputs`。
	- **User ID**: 代码中硬编码了 `"api-user-123"`。在 RAG 场景下，建议传入真实的用户 ID。这不仅是为了统计，更重要的是 Dify 的 RAG 能够记录该用户的对话历史，从而实现基于上下文的检索。
3. **如何增强？**
	- **会话管理**: 目前代码没有利用 `conversation_id`。如果用户问“它有什么功能？”，Dify 需要知道“它”指的是上一轮检索到的内容。
	- **建议修改**: 在 `DifyModel` 结构体或方法参数中增加 `conversation_id` 的传递机制，并在收到 Dify 响应后保存它返回的 `conversation_id`，以便下一次请求带上。

##### fix：使用io.ReadAll

在生产环境中，**直接使用 `io.ReadAll` 确实存在风险**，虽然在目前的 Dify 文本对话场景下可能“勉强够用”，但绝对不是最佳实践。

###### 1. 会不会卡住？（Latency / Blocking）

**会，但主要原因是“等待”而不是“读取慢”。**

- **现象**：`io.ReadAll` 会一直阻塞，直到服务器（Dify）关闭连接或发送完**所有**数据。
- **场景**：如果 Dify 生成一个很长的回答（比如 2000 字），这可能需要 30 秒。在这 30 秒内，你的 Go 程序会停在 `io.ReadAll` 这一行，既不报错也不往下走。
- **用户体验**：前端用户会看到 loading 转圈转了 30 秒，然后突然弹出一大段话。这就是为什么 LLM 场景更推荐流式（Streaming）。

###### 2. 会不会内存溢出？（Memory / OOM）

**在纯文本对话场景下一般不会，但在通用场景下很危险。**

- **LLM 场景**：即便是 GPT-4 生成一篇长文，数据量通常也只有几 KB 到几十 KB。这点内存对于服务器来说是九牛一毛，所以这里用 `io.ReadAll` 不会导致内存崩溃。
- **潜在风险**：如果 Dify 的接口发生异常，或者你未来扩展功能（例如 Dify 返回了一张生成的图片、或者一个巨大的检索文档列表），`io.ReadAll` 会试图把这些内容**全部加载到内存中**。如果此时并发很高（例如 1000 个人同时请求），服务器内存瞬间就会被打爆。

###### 最佳实践改进方案

针对 HTTP JSON 响应，Go 语言的标准最佳实践是使用 `json.NewDecoder`，而不是 `io.ReadAll` + `json.Unmarshal`。

使用 `json.NewDecoder`（推荐）

这种方式是**边读边解析**，不需要把原始的 `[]byte` 全部存在内存里，效率更高，且代码更优雅。

**修改后的 `GenerateResponse` 代码片段：**

```go
// ... 发送请求代码 ...

resp, err := d.client.Do(req)
if err != nil {
    return nil, err
}
defer resp.Body.Close() // 记得关闭 Body

// 【改进点 1】：检查状态码
if resp.StatusCode != http.StatusOK {
    // 只有出错时才 ReadAll，为了打印错误日志（错误信息通常不长）
    bodyBytes, _ := io.ReadAll(resp.Body)
    return nil, fmt.Errorf("dify api error: %d, body: %s", resp.StatusCode, string(bodyBytes))
}

// 【改进点 2】：直接流式解析 JSON
var result map[string]interface{}
// 创建一个解码器，直接从网络流中读取
decoder := json.NewDecoder(resp.Body)
if err := decoder.Decode(&result); err != nil {
    return nil, fmt.Errorf("failed to decode dify response: %w", err)
}

// ... 后续提取 answer 的逻辑 ...
```

还有一个关键点：超时控制（防止真正的“卡死”）

你提到的“卡住”，最可怕的情况不是读得慢，而是**网络挂了或者 Dify 死锁了，永远不返回 EOF**。

你的代码中使用了 `ctx`，这是很好的，但 `http.Client` 默认是没有超时的。建议在初始化 `client` 时设置超时：

```go
// 在 NewDifyModel 中
return &DifyModel{
    apiKey:  apiKey,
    baseUrl: baseUrl,
    client:  &http.Client{
        Timeout: 60 * time.Second, // 设置一个合理的超时时间，比如 60 秒
    },
}, nil
```



### **基于 Eino 的本地 RAG 索引与 Dify 混合生成架构**

#### 1. 核心设计图

我们可以把整个流程想象成一个“三层汉堡包”：

- **数据层 (Data Layer) - 本地掌控**：原始文件存储在你的服务器磁盘。

- **检索层 (Retrieval Layer) - Eino 驱动**：利用 **CloudWeGo Eino** 的 Indexer 和 Retriever 组件，在本地 Redis 中建立向量索引。
	- **输入**：用户上传的文件。
	- **处理**：Eino Loader -> Eino Splitter (Markdown/Recursive) -> Embedding (Ark/OpenAI API) -> Redis Vector Store (Vector)。
	- **输出**：Top K 相关文档片段（Context）。

- **生成层 (Generation Layer) - Dify 托管**：通过 Dify API 调用云端 LLM。

	- **输入**：用户问题 (Query) + 本地检索到的 Context。

	- **Prompt**：在 Dify 平台配置好的 System Prompt（不用硬编码在 Go 里）。

	- **输出**：最终回答。

#### 2. 详细流程与组件选型

**组件选型理由 (面试重点)：**

- **Eino (Go)**: 高性能、流式处理、组件化。适合 R 过程。
- **Redis Stack**: 既做缓存又做向量库，省运维成本。比单独部署 Milvus 轻量。
- **Dify (SaaS)**: Prompt 管理、模型切换、日志监控。解放后端开发。
- **Ark (Embedding)**: 便宜、中文效果好。

**流程详解：**

**阶段一：知识库构建 (Offline Build)**

1. Loader: 使用eino/components/document/loader/file 读取本地.md/.txt文件。
2. Splitter: (关键改进) 使用markdown.HeaderSplitter 按章节切分，比之前的整文件强一万倍。

3. Embedder: 调用Ark API 生成 1024 维向量。

4. Indexer: 存入 Redis Hash，Key 格式 rag_docs:{filename}:{uuid}。

**阶段二：在线问答 (Online Serve)**

1. **User Request**: 用户发来 "如何配置数据库？"
2. **Embedding**: 把问题变成向量。
3. **Retriever**: 在 Redis 里搜 Top 5 最相似的片段。
4. Context Construction: 拼接片段为字符串[文档1内容]...[文档2内容]...
5. LLM Call: 调用 Dify API，入参query="如何配置数据库？"，inputs={"context": "拼接后的字符串"}。
6. **Response**: 返回 Dify 结果。

### **3. 面试题集锦 (必考题)**

**Q1: 为什么采用“本地检索 + 云端生成”的混合架构？**

> **答**：这是一个平衡了 **数据隐私** 和 **模型能力** 的最优解。
>
> - **数据安全**：原始文件（可能是保密合同）必须留在本地服务器，不能全传给第三方知识库。只把搜出来的几句话发给 LLM，泄密风险最小。
> - **可控性**：我想控制切片逻辑（比如按标题切分），SaaS 版一般只有固定那几种，不够灵活。
> - **成本**：全量向量存储在本地 Redis，不占云端知识库配额，省钱。

**Q2: 既然用了 Eino，为什么还用 Redis 而不是专业的向量库（如 Milvus/Weaviate）？**

> **答**：考虑到 **运维复杂度**。Redis Stack 本身支持 Vector Search，我的系统本身就依赖 Redis 做缓存，复用现有中间件可以减少系统熵增。对于目前的文档量级（百万级以下），Redis 的 HNSW 索引性能完全够用，而且支持 Metadata 过滤。

**Q3: 你是怎么做文本切片 (Chunking) 的？有什么优化策略？**

> **答**：起初我是简单的按字符数切分（Fixed Size），但发现经常把一句话切断。 后来我改用了 **Markdown Header Splitter** (Eino内置)，按章节层级切分，保证语义完整性。 此外，我在索引时加入了一些 Metadata（如文件名、章节名），检索时可以做过滤。

**Q4: 如果检索回来的内容不相关怎么办？(RAG 的通病)**

> **答**：这就是我在架构设计时预留的空间。因为检索逻辑在我手里，我以后可以加：
>
> 1. **Query Rewrite**: 把“它怎么用”改写为“Redis 怎么用”。
> 2. **Rerank**: 加一个重排序步骤。
> 3. **Hybrid Search**: 关键词 + 向量双路召回。

### **4. 执行计划**

1. 重构 go.mod: 确保引入了eino-ext的最新 indexer 和 retriever 组件

2. 重写 common/rag/rag.go:

	- 废弃旧的NewRAGIndexer。
	- 实现新的 EinoIndexer （带 Splitter 的）。
	- 实现新的 EinoRetriever。

3. 更新 dify_model.go:

	- 接入新的Retrieve 方法。
	- 把检索结果传给 Dify。

	3. 更新 `dify_model.go` (架构修正建议)

	- **你的计划**：“接入新的 Retrieve 方法”
	- **我的建议**：**千万不要修改 `DifyModel` 去做检索！**
		- **为什么？** `DifyModel` 应该是一个纯粹的“聊天组件”（Chat Model）。它的职责是：*给你一段话（Prompt），我吐出一段话（Response）*。它不应该知道“知识库”的存在。
		- **正确的做法**：在 `DifyModel` **外面** 再包一层（或者在 Service 层/Main 函数里），由这一层负责“先检索，再调用 Dify”。

### **第一步：RAG 核心引擎改造**

**引入 Eino 的 Document Splitter 组件**

**目标**：

重写 common/rag/rag.go 中的 IndexFile方法，使其支持**智能切片 (Markdown Chunking)**。

之前的代码是把整个文件当作一个 Document 塞进向量库。

- **后果**：比如一个 10000 字的文件，Embedding API 截断后只剩前 500 字，后面的内容完全没进索引，根本搜不到！
- **改进**：引入markdown.NewHeaderSplitter，按#这类标题把文章切成几百个小段，每段单独向量化，保证全部内容可检索。

 **common/rag/rag.go：**

```go
package rag

import (
	"GopherAI/common/redis"
	redisPkg "GopherAI/common/redis"
	"GopherAI/config"
	"context"
	"fmt"
	"os"
	"path/filepath"
	"strings"
	"sync"

	// Eino 扩展组件
	"github.com/cloudwego/eino-ext/components/document/transformer/splitter/semantic"
	embeddingArk "github.com/cloudwego/eino-ext/components/embedding/ark"
	redisIndexer "github.com/cloudwego/eino-ext/components/indexer/redis"
	redisRetriever "github.com/cloudwego/eino-ext/components/retriever/redis"

	// Eino 核心接口
	"github.com/cloudwego/eino/components/embedding"
	"github.com/cloudwego/eino/components/retriever"
	"github.com/cloudwego/eino/schema"
	redisCli "github.com/redis/go-redis/v9"
)

/*
   === DevDoc: RAG 模块优化说明 ===

   1. Singleton (单例模式):
      Embedding 模型 (globalEmbedder) 是无状态的，我们在包级别使用 sync.Once 初始化它。
      避免了每次请求都重新建立 HTTP Client 的开销。

   2. Caching (缓存池):
      Retriever 是有状态的（绑定了特定的 Redis Index 即特定用户的文件）。
      我们使用 sync.Map (retrieverCache) 来缓存已创建的 retriever。
      Key: "username:indexName"
      Value: *RAGQuery 实例

   这意味着：当同一个用户连续提问时，系统不需要重新连接 Redis、重新配置检索器，而是直接从内存复用，性能极高。
*/

var (
	// 全局唯一的 Embedding 实例
	globalEmbedder embedding.Embedder
	embedderOnce   sync.Once

	// 全局 Retriever 缓存池
	// Key: indexName (string), Value: *RAGQuery
	retrieverCache sync.Map
)

// getGlobalEmbedder 获取或初始化全局 Embedding 模型
func getGlobalEmbedder(ctx context.Context) (embedding.Embedder, error) {
	var err error
	embedderOnce.Do(func() {
		apiKey := os.Getenv("OPENAI_API_KEY")
		embedConfig := &embeddingArk.EmbeddingConfig{
			BaseURL: config.GetConfig().RagModelConfig.RagBaseUrl,
			APIKey:  apiKey,
			Model:   config.GetConfig().RagModelConfig.RagEmbeddingModel,
		}
		globalEmbedder, err = embeddingArk.NewEmbedder(ctx, embedConfig)
	})
	if err != nil {
		return nil, err
	}
	return globalEmbedder, nil
}

type RAGIndexer struct {
	embedding embedding.Embedder
	indexer   *redisIndexer.Indexer
}

type RAGQuery struct {
	embedding embedding.Embedder
	retriever retriever.Retriever
}

// NewRAGIndexer 初始化索引器 (Write Path)
func NewRAGIndexer(filename, embeddingModel string) (*RAGIndexer, error) {
	ctx := context.Background()
	dimension := config.GetConfig().RagModelConfig.RagDimension

	// 复用全局 Embedder
	embedder, err := getGlobalEmbedder(ctx)
	if err != nil {
		return nil, fmt.Errorf("failed to get embedder: %w", err)
	}

	if err := redisPkg.InitRedisIndex(ctx, filename, dimension); err != nil {
		return nil, fmt.Errorf("failed to init redis index: %w", err)
	}

	rdb := redisPkg.Rdb
	indexerConfig := &redisIndexer.IndexerConfig{
		Client:    rdb,
		KeyPrefix: redis.GenerateIndexNamePrefix(filename),
		BatchSize: 10,
		DocumentToHashes: func(ctx context.Context, doc *schema.Document) (*redisIndexer.Hashes, error) {
			source := ""
			if s, ok := doc.MetaData["source"].(string); ok {
				source = s
			}
			return &redisIndexer.Hashes{
				Key: fmt.Sprintf("%s:%s", filename, doc.ID),
				Field2Value: map[string]redisIndexer.FieldValue{
					"content":  {Value: doc.Content, EmbedKey: "vector"},
					"metadata": {Value: source},
				},
			}, nil
		},
	}
	indexerConfig.Embedding = embedder

	idx, err := redisIndexer.NewIndexer(ctx, indexerConfig)
	if err != nil {
		return nil, fmt.Errorf("failed to create indexer: %w", err)
	}

	return &RAGIndexer{
		embedding: embedder,
		indexer:   idx,
	}, nil
}

// IndexFile 索引文件 (Write Path)
func (r *RAGIndexer) IndexFile(ctx context.Context, filePath string) error {
	content, err := os.ReadFile(filePath)
	if err != nil {
		return fmt.Errorf("failed to read file: %w", err)
	}

	doc := &schema.Document{
		ID:      filePath,
		Content: string(content),
		MetaData: map[string]any{
			"source": filePath,
			"type":   "text",
		},
	}

	// === 语义切分 (Semantic Splitter) ===
	// 使用 Eino 的 semantic.NewSplitter：
	// 1. 先按分隔符（换行、句号等）将文档拆成小段
	// 2. 用 Embedding 模型计算相邻段落的向量相似度
	// 3. 在语义断裂处（相似度低于阈值）切分
	// 优势：不会把语义相关的内容切开，比简单按字数切分质量高得多
	embedder, embedErr := getGlobalEmbedder(ctx)
	if embedErr != nil {
		return fmt.Errorf("failed to get embedder for splitter: %w", embedErr)
	}

	splitter, err := semantic.NewSplitter(ctx, &semantic.Config{
		Embedding:    embedder,                                   // 复用全局 Embedder
		BufferSize:   2,                                          // 上下文缓冲区：前后各看 2 段
		MinChunkSize: 100,                                        // 最小片段 100 字符，避免碎片
		Separators:   []string{"\n\n", "\n", "。", ".", "?", "!"}, // 中英文友好分隔符
		Percentile:   0.9,                                        // 90% 分位数作为切分阈值
	})
	if err != nil {
		return fmt.Errorf("failed to create semantic splitter: %w", err)
	}

	chunks, err := splitter.Transform(ctx, []*schema.Document{doc})
	if err != nil {
		return fmt.Errorf("failed to split document: %w", err)
	}

	// 为每个 chunk 生成唯一 ID
	baseName := filepath.Base(filePath)
	for i, chunk := range chunks {
		chunk.ID = fmt.Sprintf("%s_chunk_%d", baseName, i)
	}

	_, err = r.indexer.Store(ctx, chunks)
	if err != nil {
		return fmt.Errorf("failed to store document chunks: %w", err)
	}

	// 索引更新后，清除该文件的检索器缓存，确保下次读取是最新的
	// filename is actually baseName or uuid, need to match logic in InitRedisIndex
	// Here we just invalidate cache to be safe.
	// Since IndexFile is rare operation, this is fine.
	retrieverCache.Range(func(key, value interface{}) bool {
		// 简单暴力清除缓存，或者更精细地只清除该 filename 对应的
		// 这里暂未传入 filename 参数 (只传了 filePath)，
		// 实际项目中可以优化为更精确的 Invalidate
		retrieverCache.Delete(key)
		return true
	})

	fmt.Printf("Successfully indexed file: %s, total chunks: %d\n", filePath, len(chunks))
	return nil
}

func DeleteIndex(ctx context.Context, filename string) error {
	if err := redisPkg.DeleteRedisIndex(ctx, filename); err != nil {
		return fmt.Errorf("failed to delete redis index: %w", err)
	}
	// 清除缓存
	indexName := redis.GenerateIndexName(filename)
	retrieverCache.Delete(indexName)
	return nil
}

// NewRAGQuery 获取 RAG 查询器 (Read Path) - 支持缓存
func NewRAGQuery(ctx context.Context, username string) (*RAGQuery, error) {
	// 1. 获取文件名 (Identify User's Knowledge Base)
	userDir := fmt.Sprintf("uploads/%s", username)
	files, err := os.ReadDir(userDir)
	if err != nil || len(files) == 0 {
		return nil, fmt.Errorf("no uploaded file found for user %s", username)
	}
	var filename string
	for _, f := range files {
		if !f.IsDir() {
			filename = f.Name()
			break
		}
	}
	if filename == "" {
		return nil, fmt.Errorf("no valid file found for user %s", username)
	}

	// 2. 计算 Cache Key
	indexName := redis.GenerateIndexName(filename)

	// 3. 查缓存
	if val, ok := retrieverCache.Load(indexName); ok {
		// Cache Hit!
		if q, ok := val.(*RAGQuery); ok {
			return q, nil
		}
	}

	// 4. Cache Miss - 初始化 (Thread Safe roughly, assuming slight race is ok or use singleflight)
	// 复用全局 Embedder
	embedder, err := getGlobalEmbedder(ctx)
	if err != nil {
		return nil, fmt.Errorf("failed to get embedder: %w", err)
	}

	rdb := redisPkg.Rdb
	retrieverConfig := &redisRetriever.RetrieverConfig{
		Client:       rdb,
		Index:        indexName,
		Dialect:      2,
		ReturnFields: []string{"content", "metadata", "distance"},
		TopK:         5,
		VectorField:  "vector",
		DocumentConverter: func(ctx context.Context, doc redisCli.Document) (*schema.Document, error) {
			resp := &schema.Document{
				ID:       doc.ID,
				Content:  "",
				MetaData: map[string]any{},
			}
			for field, val := range doc.Fields {
				if field == "content" {
					resp.Content = val
				} else {
					resp.MetaData[field] = val
				}
			}
			return resp, nil
		},
	}
	retrieverConfig.Embedding = embedder

	rtr, err := redisRetriever.NewRetriever(ctx, retrieverConfig)
	if err != nil {
		return nil, fmt.Errorf("failed to create retriever: %w", err)
	}

	queryInstance := &RAGQuery{
		embedding: embedder,
		retriever: rtr,
	}

	// 5. 存入缓存
	retrieverCache.Store(indexName, queryInstance)

	return queryInstance, nil
}

func (r *RAGQuery) RetrieveDocuments(ctx context.Context, query string) ([]*schema.Document, error) {
	docs, err := r.retriever.Retrieve(ctx, query)
	if err != nil {
		return nil, fmt.Errorf("failed to retrieve documents: %w", err)
	}
	return docs, nil
}

func RetrieveContext(ctx context.Context, username, query string) (string, error) {
	ragQuery, err := NewRAGQuery(ctx, username)
	if err != nil {
		return "", nil // 返回空表示无知识库，不报错
	}
	docs, err := ragQuery.RetrieveDocuments(ctx, query)
	if err != nil {
		return "", fmt.Errorf("failed to retrieve documents: %w", err)
	}
	if len(docs) == 0 {
		return "", nil
	}
	var sb strings.Builder
	for i, doc := range docs {
		sb.WriteString(fmt.Sprintf("[文档片段 %d]: %s\n\n", i+1, doc.Content))
	}
	return sb.String(), nil
}
```

核心概念速查 (给初学者的比喻)

- **Embedding (向量化)**: 翻译官。把文字（如“苹果”）翻译成计算机能算的数字坐标（如 `[0.1, 0.9, -0.5]`）。
- **Indexer (索引器)**: 搬运工。负责把翻译好的数字坐标存进仓库（Redis）。
- **Retriever (检索器)**: 搜查员。当用户问问题时，去仓库里找最相似的坐标。
- **Singleton (单例)**: 全局唯一的工具人。比如翻译官很贵，我们只雇佣一个，大家共用。
- **Cache (缓存)**: 记忆。搜查员查过一次某人的仓库后，就记住了仓库的位置，下次不用再重新办手续。

#### 第一部分：全局优化与基础设施 (性能的地基)

这部分代码负责“省钱”和“提速”。

```go
package rag

import (
    // ... 引入标准库和 Eino 组件 ...
    "sync" // 用于实现单例和并发安全的缓存
    // ...
)

/*
   === 设计思路解析 ===
   这里使用了两个重要的设计模式：
   1. 单例模式 (Singleton): 针对 Embedding 模型。
      原因：创建 HTTP Client 很耗资源，没必要每次请求都创建。全局共用一个即可。
   2. 缓存模式 (Caching): 针对 Retriever 检索器。
      原因：Retriever 初始化需要连接 Redis 并配置参数。如果用户连续提问，直接复用之前的 Retriever 对象最快。
*/

var (
    // globalEmbedder: 存放全局唯一的 Embedding 实例
    globalEmbedder embedding.Embedder
    // embedderOnce: sync.Once 确保初始化代码只执行一次，即使高并发也没问题
    embedderOnce   sync.Once

    // retrieverCache: 一个并发安全的 Map (类似 Java 的 ConcurrentHashMap)
    // Key: 索引名 (string), Value: *RAGQuery 对象
    retrieverCache sync.Map
)

// getGlobalEmbedder: 获取全局翻译官
// 只有第一次调用时会真正创建，后面调用直接返回现成的
func getGlobalEmbedder(ctx context.Context) (embedding.Embedder, error) {
    var err error
    // Do 方法保证里面的函数只执行一次
    embedderOnce.Do(func() {
        // 从环境变量拿 API Key
        apiKey := os.Getenv("OPENAI_API_KEY")
        // 配置火山引擎 (Ark) 的参数
        embedConfig := &embeddingArk.EmbeddingConfig{
            BaseURL: config.GetConfig().RagModelConfig.RagBaseUrl,
            APIKey:  apiKey,
            Model:   config.GetConfig().RagModelConfig.RagEmbeddingModel,
        }
        // 创建实例
        globalEmbedder, err = embeddingArk.NewEmbedder(ctx, embedConfig)
    })
    if err != nil {
        return nil, err
    }
    return globalEmbedder, nil
}
```

#### 第二部分：写入流程 (把书放进图书馆)

这部分负责：读取文件 -> 智能切分 -> 变成向量 -> 存入 Redis。

```go
// RAGIndexer: 负责写入的结构体
type RAGIndexer struct {
    embedding embedding.Embedder   // 向量化工具
    indexer   *redisIndexer.Indexer // 存储工具
}

// NewRAGIndexer: 组装一个写入器
func NewRAGIndexer(filename, embeddingModel string) (*RAGIndexer, error) {
    ctx := context.Background()
    // 获取向量维度（例如 1536 维），Redis 建表时需要知道
    dimension := config.GetConfig().RagModelConfig.RagDimension

    // 1. 获取全局唯一的 Embedding 实例 (复用)
    embedder, err := getGlobalEmbedder(ctx)
    if err != nil {
        return nil, fmt.Errorf("failed to get embedder: %w", err)
    }

    // 2. 初始化 Redis 索引 (相当于在数据库里建表)
    if err := redisPkg.InitRedisIndex(ctx, filename, dimension); err != nil {
        return nil, fmt.Errorf("failed to init redis index: %w", err)
    }

    // 3. 配置 Indexer
    rdb := redisPkg.Rdb
    indexerConfig := &redisIndexer.IndexerConfig{
        Client:    rdb,
        KeyPrefix: redis.GenerateIndexNamePrefix(filename), // 给 Key 加前缀
        BatchSize: 10, // 批量写入，一次写 10 条，提高速度

        // [关键] 定义数据映射：怎么把 Go 的文档对象转成 Redis 的 Hash 结构
        DocumentToHashes: func(ctx context.Context, doc *schema.Document) (*redisIndexer.Hashes, error) {
            source := ""
            if s, ok := doc.MetaData["source"].(string); ok {
                source = s
            }
            return &redisIndexer.Hashes{
                // Redis Key: 文件名:文档ID
                Key: fmt.Sprintf("%s:%s", filename, doc.ID),
                // 字段映射
                Field2Value: map[string]redisIndexer.FieldValue{
                    // EmbedKey: "vector" 意思是：
                    // 请把 "content" 里的文字拿去算向量，结果存到 "vector" 字段里
                    "content":  {Value: doc.Content, EmbedKey: "vector"},
                    "metadata": {Value: source},
                },
            }, nil
        },
    }
    indexerConfig.Embedding = embedder // 绑定 Embedding

    idx, err := redisIndexer.NewIndexer(ctx, indexerConfig)
    // ... 错误处理 ...

    return &RAGIndexer{
        embedding: embedder,
        indexer:   idx,
    }, nil
}

// IndexFile: 核心业务 - 处理文件并入库
func (r *RAGIndexer) IndexFile(ctx context.Context, filePath string) error {
    // 1. 读取文件内容
    content, err := os.ReadFile(filePath)
    // ...

    // 2. 包装成 Eino 文档对象
    doc := &schema.Document{
        ID:      filePath,
        Content: string(content),
        MetaData: map[string]any{
            "source": filePath,
            "type":   "text",
        },
    }

    // 3. [进阶] 语义切分 (Semantic Splitter)
    // 这里使用了高级切分技术，而不是简单的按字数切。
    // 它会计算段落间的相似度，尽量不把意思连贯的话切断。
    embedder, _ := getGlobalEmbedder(ctx)
    splitter, err := semantic.NewSplitter(ctx, &semantic.Config{
        Embedding:    embedder,        // 需要 Embedding 来计算相似度
        BufferSize:   2,               // 往前看 2 段，往后看 2 段
        MinChunkSize: 100,             // 最小片段 100 字
        Separators:   []string{"\n\n", "。", "."}, // 分隔符优先级
        Percentile:   0.9,             // 相似度阈值
    })
    
    // 执行切分
    chunks, err := splitter.Transform(ctx, []*schema.Document{doc})
    
    // 给切片生成唯一 ID
    baseName := filepath.Base(filePath)
    for i, chunk := range chunks {
        chunk.ID = fmt.Sprintf("%s_chunk_%d", baseName, i)
    }

    // 4. 存入 Redis
    // 这里会自动执行：文本 -> 向量 -> Redis存储
    _, err = r.indexer.Store(ctx, chunks)
    
    // 5. [关键优化] 清除缓存
    // 因为文件更新了，之前的检索器可能用的是旧索引数据。
    // 为了保险，我们清空缓存，下次查询时会重新初始化检索器。
    retrieverCache.Range(func(key, value interface{}) bool {
        retrieverCache.Delete(key)
        return true
    })

    return nil
}
```

#### 第三部分：读取流程 (去图书馆借书)

这部分负责：识别用户 -> 找缓存 -> 没缓存就初始化 -> 执行检索。

```go
// RAGQuery: 负责读取的结构体
type RAGQuery struct {
    embedding embedding.Embedder
    retriever retriever.Retriever
}

// NewRAGQuery: 获取查询器 (带缓存逻辑)
func NewRAGQuery(ctx context.Context, username string) (*RAGQuery, error) {
    // 1. 确定要查哪个文件
    // 这里简化逻辑：去用户目录看文件名
    userDir := fmt.Sprintf("uploads/%s", username)
    // ... 省略查找文件名的代码 ...
    
    // 2. 计算缓存 Key (索引名)
    indexName := redis.GenerateIndexName(filename)

    // 3. [查缓存] 看看之前是不是已经创建过这个用户的检索器
    if val, ok := retrieverCache.Load(indexName); ok {
        // 命中缓存！直接转换类型并返回，速度极快
        if q, ok := val.(*RAGQuery); ok {
            return q, nil
        }
    }

    // 4. [缓存未命中] 需要重新初始化
    
    // 获取全局 Embedder
    embedder, err := getGlobalEmbedder(ctx)
    
    // 配置 Redis 检索器
    rdb := redisPkg.Rdb
    retrieverConfig := &redisRetriever.RetrieverConfig{
        Client:       rdb,
        Index:        indexName,
        ReturnFields: []string{"content", "metadata", "distance"}, // 需要返回哪些字段
        TopK:         5,        // 找最相似的前 5 个
        VectorField:  "vector", // 拿着向量去比对 "vector" 字段
        // ... DocumentConverter 省略 ...
    }
    retrieverConfig.Embedding = embedder

    rtr, _ := redisRetriever.NewRetriever(ctx, retrieverConfig)

    // 创建实例
    queryInstance := &RAGQuery{
        embedding: embedder,
        retriever: rtr,
    }

    // 5. [写入缓存] 下次就能直接用了
    retrieverCache.Store(indexName, queryInstance)

    return queryInstance, nil
}

// RetrieveContext: 对外暴露的最终接口
func RetrieveContext(ctx context.Context, username, query string) (string, error) {
    // 获取查询器 (可能会走缓存)
    ragQuery, err := NewRAGQuery(ctx, username)
    if err != nil { return "", nil } // 没找到知识库，返回空

    // 执行检索
    docs, err := ragQuery.RetrieveDocuments(ctx, query)
    
    // 拼接结果字符串
    var sb strings.Builder
    for i, doc := range docs {
        sb.WriteString(fmt.Sprintf("[文档片段 %d]: %s\n\n", i+1, doc.Content))
    }
    return sb.String(), nil
}
```

不要试图一次背下来。按照这个 **“破坏-观察”** 流程来学习：

**观察单例的效果**：

- 在 `getGlobalEmbedder` 里加一句 `fmt.Println("正在初始化 Embedding...")`。
- 然后运行程序，连续问 5 次问题。
- **预期**：你应该只看到**一次**打印输出。这证明单例生效了。

**观察缓存的效果**：

- 在 `NewRAGQuery` 的 `if val, ok := retrieverCache.Load` 代码块里，加一句 `fmt.Println("命中缓存！")`。
- 在 `Cache Miss` 的部分加一句 `fmt.Println("未命中缓存，正在初始化...")`。
- **预期**：第一次提问显示“未命中”，第二次及以后显示“命中缓存”。

**理解切分**：

- 在 `IndexFile` 里，把 `MinChunkSize` 改成 `10`，然后重新上传文件。
- **预期**：你会发现检索回来的片段变得特别碎，甚至只有半句话。这就让你理解了切分参数的重要性。

#### **总结**

打造一个**完全自主可控的知识库引擎**。

把“知识处理（切片）”和“知识检索（Recall）”的命脉掌握在自己手里，而不是依赖 Dify 的黑盒。

 这意味着：

1. **数据隐私**：原始文件只在本地服务器流转，不上传云端。
2. **检索精度**：通过手动控制切片大小（Chunk Size 1000, Overlap 200），解决了之前“整篇索引导致后半截搜不到”的致命问题。
3. **架构解耦**：Go 服务不再是一个简单的传声筒，而是承担了重计算（Embedding）和重存储（Redis Vector）的核心逻辑。

**设计方案回顾**：

- 输入：用户上传的文件（如.md 和.txt）
- **处理**：
	- 切片：手动实现滑动窗口法
	- 向量化：调用Ark Embedding 模型。
	- **存储**：存入Redis stack，key 为 doc:{uuid}:chunk:{i}。
- **输出**：提供 RetrieveContext 接口，给定 Query，返回 Top 5相关片段。

### **第二步 (Step 2)：Dify 智能大脑接入**

**DifyModel 应该只是一个纯粹的 API Connector，不包含任何业务逻辑（如 RAG 检索）。**

- **Service 层 (业务大脑)**：负责 Retrieve (检索文档) ->Inject（注入上下文） ->Invoke (调用模型)。这是真正的 RAG 控制流。
- **AIModel 层 (执行手脚)**：负责 Extract（从上下文获取数据）-> Request(请求dify api)->Response(标准化返回)

**架构图 (Micro-Workflow)**：

```
graph TD
    UserRequest[用户请求] --> ServiceLayer[Service层: ChatService]
    
    subgraph Service Logic
        ServiceLayer -- "1. Extract Query" --> Query[问题]
        ServiceLayer -- "2. Retrieve (R)" --> RAG[rag.RetrieveContext]
        RAG -- "3. Return Docs" --> Docs[文档片段]
        Docs -- "4. Inject into Context" --> Context[ctx With Value: dify_inputs]
    end
    
    ServiceLayer -- "5. Call Generate(ctx, msg)" --> AIModel[DifyModel]
    
    subgraph AIModel Logic
        AIModel -- "6. Read Context" --> Inputs[Dify Inputs]
        Inputs -- "7. Build Request" --> API[Dify API]
    end
```

 **2. 方案详解**

#### **编码执行计划**

**第一步：定义 Context Key** 在 common/aihelper 包中（甚至最好单独一个 constants 包，但为了方便暂时放这里）定义 Key。

**第二步：净化** DifyModel

1. 删除 rag 相关的 import 和调用。
2. 添加 ctx.Value 读取逻辑。

**第三步：改造 Service 层**

1. 找到调用入口（不仅是 main.go ，可能是 service/chat/chat.go）。
2. 插入 rag.RetrieveContext 和 Context 注入代码。

#### Step 1: 净化 DifyModel (Model Layer)

把 DifyModel 变回一个**纯粹**的 API 调用器

**设计思路**：

