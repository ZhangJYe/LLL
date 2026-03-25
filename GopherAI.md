#  GopherAI

### 快速入门

GopherAI 是一个**企业级 AI 对话系统**，类似于 ChatGPT，但具有以下特色功能：

1. **多种 AI 模式**
	- 标准对话模式（SimpleLLM）
	- ReAct Agent（能够使用工具的智能体）
	- Planner Agent（能够制定计划并执行的智能体）
2. **工具调用能力**
	- 内置工具：时间查询、计算器、天气查询
	- 支持 MCP 协议扩展外部工具
3. **知识库（RAG）**
	- 上传文档，AI 基于文档内容回答问题
	- 支持 PDF、TXT、Markdown 等格式
4. **企业级特性**
	- 用户注册/登录（JWT 认证）
	- 多会话管理
	- 流式输出（实时显示 AI 回复）
	- 消息历史记录

#### API接口

用户：

| 接口                    | 方法 | 说明           |
| :---------------------- | :--- | :------------- |
| `/api/v1/user/register` | POST | 注册账号       |
| `/api/v1/user/login`    | POST | 登录获取 Token |
| `/api/v1/user/captcha`  | POST | 获取邮箱验证码 |

对话相关（需要token）

| 接口                               | 方法 | 说明                 |
| :--------------------------------- | :--- | :------------------- |
| `/api/v1/AI/chat/sessions`         | GET  | 获取会话列表         |
| `/api/v1/AI/chat/send-new-session` | POST | 创建新会话并发送消息 |
| `/api/v1/AI/chat/send`             | POST | 在现有会话中发送消息 |
| `/api/v1/AI/chat/send-stream`      | POST | 流式发送消息         |
| `/api/v1/AI/chat/history`          | POST | 获取会话历史         |

文件相关（需要token）

| 接口                  | 方法 | 说明             |
| :-------------------- | :--- | :--------------- |
| `/api/v1/file/upload` | POST | 上传文件到知识库 |

#### 模型类型说明

在创建对话时，`modelType` 参数决定了 AI 的行为模式：

| 类型  | 名称          | 核心特点       | 适用场景       | 性能 | 成本 |
| :---- | :------------ | :------------- | :------------- | :--- | :--- |
| `"1"` | SimpleLLM     | 纯对话，无工具 | 闲聊、文本生成 | ⚡⚡⚡  | 💰    |
| `"2"` | AliRAG        | 自动检索知识库 | 文档问答       | ⚡⚡   | 💰💰   |
| `"3"` | MCP           | 手动工具调用   | 简单工具集成   | ⚡⚡   | 💰💰   |
| `"4"` | Ollama        | 本地模型       | 离线/隐私场景  | ⚡    | 免费 |
| `"5"` | Dify          | 企业知识库     | 企业级应用     | ⚡⚡   | 💰💰💰  |
| `"6"` | ReAct Agent   | 自主思考+工具  | 复杂任务       | ⚡    | 💰💰💰  |
| `"7"` | Planner Agent | 任务规划+执行  | 多步骤任务     | ⚡    | 💰💰💰💰 |
| `"8"` | Hybrid        | ReAct + Dify   | 最强组合       | ⚡    | 💰💰💰💰 |

设计思路：渐进式增强

第一层基础对话

```go
// SimpleLLM - 最简单的实现
type OpenAIModel struct {
    llm model.ToolCallingChatModel
}

func (o *OpenAIModel) GenerateResponse(ctx, messages) {
    return o.llm.Generate(ctx, messages)  // 直接调用
}
```

**为什么需要？**

- 用户只想聊天，不需要工具
- 响应速度最快（无额外逻辑）
- Token 消耗最少
- 适合：闲聊、创意写作、代码生成

### 系统架构

GopherAI 采用经典的 MVC 分层架构，并在此基础上增加了 AI 核心层：