# Hello-Agent

# 智能体和语言模型基础

## 第一章初始智能体

### 什么是智能体？

​	在探索任何一个复杂概念时，我们最好从一个简洁的定义开始。

​	在人工智能领域，智能体被定义为任何能够通过**传感器（Sensors）**感知其所处**环境（Environment）**，并**自主**地通过**执行器（Actuators）**采取**行动（Action）**以达成特定目标的实体。

​	这个定义包含了智能体存在的四个基本要素。

​	环境是智能体所处的外部世界。

​	对于自动驾驶汽车，环境是动态变化的道路交通；对于一个交易算法，环境则是瞬息万变的金融市场。

​	智能体并非与环境隔离，它通过其传感器持续地感知环境状态。

​	摄像头、麦克风、雷达或各类**应用程序编程接口（Application Programming Interface, API）**返回的数据流，都是其感知能力的延伸。

​	获取信息后，智能体需要采取行动来对环境施加影响，它通过执行器来改变环境的状态。

​	执行器可以是物理设备（如机械臂、方向盘）或虚拟工具（如执行一段代码、调用一个服务）。

​	然而，真正赋予智能体"智能"的，是其**自主性（Autonomy）**。

​	智能体并非只是被动响应外部刺激或严格执行预设指令的程序，它能够基于其**感知和内部状态进行独立决策**，以达成其设计目标。这种从感知到行动的闭环，构成了所有智能体行为的基础。

#### 传统视角下的智能体

​	完全依赖于当前的感知输入，不具备记忆或预测能力。它像一种数字化的本能，可靠且高效，但也因此无法应对需要理解上下文的复杂任务。它的局限性引出了一个关键问题：**如果环境的当前状态不足以作为决策的全部依据，智能体该怎么办？**

![图片描述](https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/1-figures/1757242319667-1.png)

​	为了回答这个问题，研究者们引入了“状态”的概念，发展出**基于模型的反射智能体（Model-Based Reflex Agent）**。

​	这类智能体拥有一个内部的**世界模型（World Model）**，用于追踪和理解环境中那些无法被直接感知的方面。

​	它试图回答：“**世界现在是什么样子的**？”。例如，一辆在隧道中行驶的自动驾驶汽车，即便摄像头暂时无法感知到前方的车辆，它的内部模型依然会维持对那辆车存在、速度和预估位置的判断。

​	这个内部模型让智能体拥有了初级的“记忆”，使其决策不再仅仅依赖于瞬时感知，而是基于一个更连贯、更完整的世界状态理解。

​	然而，仅仅理解世界还不够，智能体需要有明确的目标。

​	这促进了**基于目标的智能体（Goal-Based Agent）**的发展。

​	与前两者不同，它的行为不再是被动地对环境做出反应，而是主动地、有预见性地选择能够导向某个特定未来状态的行动。

​	这类智能体需要回答的问题是：“**我应该做什么才能达成目标**？”。

​	经典的例子是 GPS 导航系统：你的目标是到达公司，智能体会基于地图数据（世界模型），通过搜索算法（如 A*算法）来规划（Planning）出一条最优路径。

​	这类智能体的核心能力体现在了对未来的考量与规划上。

#### 大语言模型驱动的新范式

​	以**GPT（Generative Pre-trained Transformer）**为代表的大语言模型的出现，正在显著改变智能体的构建方法与能力边界。

​	由大语言模型驱动的 LLM 智能体，其核心决策机制与传统智能体存在本质区别，从而赋予了其一系列全新的特性。

​	这种转变，可以从两者在核心引擎、知识来源、交互方式等多个维度的对比中清晰地看出，如表 1.1 所示。	简而言之，**传统智能体的能力源于工程师的显式编程与知识构建，其行为模式是确定且有边界的**；而 **LLM 智能体则通过在海量数据上的预训练**，获得了隐式的世界模型与强大的涌现能力，使其能够以更灵活、更通用的方式应对复杂任务。

![图片描述](https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/1-figures/1757242319667-2.png)

这种差异使得 LLM 智能体可以直接处理高层级、模糊且充满上下文信息的自然语言指令。

让我们以一个“智能旅行助手”为例来说明。



​	在 LLM 智能体出现之前，规划旅行通常意味着用户需要在多个专用应用（如天气、地图、预订网站）之间手动切换，并由用户自己扮演信息整合与决策的角色。

​	而一个 LLM 智能体则能将这个流程整合起来。

​	当接收到“规划一次厦门之旅”这样的模糊指令时，它的工作方式体现了以下几点：

- **规划与推理**：智能体首先会将这个高层级目标分解为一系列逻辑子任务，例如：`[确认出行偏好] -> [查询目的地信息] -> [制定行程草案] -> [预订票务住宿]`。这是一个内在的、由模型驱动的规划过程。
- **工具使用**：在执行规划时，智能体识别到信息缺口，会主动调用外部工具来补全。例如，它会调用天气查询接口获取实时天气，并基于“预报有雨”这一信息，在后续规划中倾向于推荐室内活动。
- **动态修正**：在交互过程中，智能体会将用户的反馈（如“这家酒店超出预算”）视为新的约束，并据此调整后续的行动，重新搜索并推荐符合新要求的选项。整个“**查天气 → 调行程 → 订酒店**”的流程，展现了其根据上下文动态修正自身行为的能力。

总而言之，我们正从**开发专用自动化工具**转向**构建能自主解决问题的系统**。核心不再是编写代码，而是引导一个通用的“大脑”去规划、行动和学习。

## 第二章智能体发展历史

## 第三章大语言模型基础

### [Transformer 架构解析](https://datawhalechina.github.io/hello-agents/#/./chapter3/第三章 大语言模型基础?id=_312-transformer-架构解析)

​	Transformer在2017 年由谷歌团队提出。它完全抛弃了循环结构，转而完全依赖一种名为**注意力 (Attention)** 的机制来捕捉序列内的依赖关系，从而实现了真正意义上的并行计算。

**（1）Encoder-Decoder 整体结构**

​	最初的 Transformer 模型是为端到端任务机器翻译而设计的。

​	如图3.4所示，它在宏观上遵循了一个经典的**编码器-解码器 (Encoder-Decoder)** 架构。

![图片描述](https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/3-figures/1757249275674-3.png)

我们可以将这个结构理解为一个分工明确的团队：

1. **编码器 (Encoder)** ：任务是“**理解**”输入的整个句子。它会读取所有输入词元，最终为每个词元生成一个富含上下文信息的向量表示。
2. **解码器 (Decoder)** ：任务是“**生成**”目标句子。它会参考自己已经生成的前文，并“咨询”编码器的理解结果，来生成下一个词。

**（2）从自注意力到多头注意力**

​	想象一下我们阅读这个句子：“The agent learns because **it** is intelligent.”。

​	当我们读到加粗的 "**it**" 时，为了理解它的指代，我们的大脑会不自觉地将更多的注意力放在前面的 "agent" 这个词上。**自注意力 (Self-Attention)** 机制就是对这种现象的数学建模。它允许模型在处理序列中的每一个词时，都能兼顾句子中的所有其他词，并为这些词分配不同的“注意力权重”。权重越高的词，代表其与当前词的关联性越强，其信息也应该在当前词的表示中占据更大的比重。

​	为了实现上述过程，自注意力机制为每个输入的词元向量引入了三个可学习的角色：

- **查询 (Query, Q)**：代表**当前词元**，它正在主动地“查询”其他词元以获取信息。
- **键 (Key, K)**：代表**句子中可被查询的词元“标签”或“索引**”。
- **值 (Value, V)**：代表**词元本身所携带的“内容”或“信息”**。

三个向量都是由原始的词嵌入向量乘以三个不同的、可学习的权重矩阵得到。

## 第四章智能体经典范式构建

​	一个现代的智能体，其核心能力在于能将大语言模型的推理能力与外部世界联通。

​	它能够自主地理解用户意图、拆解复杂任务，并通过调用代码解释器、搜索引擎、API等一系列“工具”，来获取信息、执行操作，最终达成目标。 

​	然而，智能体并非万能，它同样面临着来自大模型本身的“幻觉”问题、在复杂任务中可能陷入推理循环、以及对工具的错误使用等挑战，这些也构成了智能体的能力边界。

为了更好地组织智能体的“思考”与“行动”过程，业界涌现出了多种经典的架构范式。

在本章中，我们将聚焦于其中最具代表性的三种，并一步步从零实现它们：

- **ReAct (Reasoning and Acting)：** 一种将“思考”和“行动”紧密结合的范式，让智能体边想边做，动态调整。
- **Plan-and-Solve：** 一种“三思而后行”的范式，智能体首先生成一个完整的行动计划，然后严格执行。
- **Reflection：** 一种赋予智能体“反思”能力的范式，通过自我批判和修正来优化结果。

了解了这些之后，你可能会问，市面上已有LangChain、LlamaIndex等众多优秀框架，为何还要“重复造轮子”？

答案在于，尽管成熟的框架在工程效率上优势显著，但直接使用高度抽象的工具，并不利于我们了解背后的设计机制是怎么运行的，或者是有何好处。其次，这个过程会暴露出项目的工程挑战。

框架为我们处理了许多问题，例如模型输出格式的解析、工具调用失败的重试、防止智能体陷入死循环等。亲手处理这些问题，是培养系统设计能力的最直接方式。

最后，也是最重要的一点，掌握了设计原理，你才能真正地从一个框架的“使用者”转变为一个智能体应用的“创造者”。当标准组件无法满足你的复杂需求时，你将拥有深度定制乃至从零构建一个全新智能体的能力。

### ReAct

​	在准备好LLM客户端后，我们将构建第一个，也是最经典的一个智能体范式**ReAct (Reason + Act)**。

​	ReAct由Shunyu Yao于2022年提出，其核心思想是模仿人类解决问题的方式，将**推理 (Reasoning)** 与**行动 (Acting)** 显式地结合起来，形成一个“思考-行动-观察”的循环。

​	ReAct的巧妙之处在于，它认识到**思考与行动是相辅相成的**。

​	思考指导行动，而行动的结果又反过来修正思考。为此，ReAct范式通过一种特殊的提示工程来引导模型，使其每一步的输出都遵循一个固定的轨迹：

- **Thought (思考)：** 这是智能体的“内心独白”。它会分析当前情况、分解任务、制定下一步计划，或者反思上一步的结果。
- **Action (行动)：** 这是智能体决定采取的具体动作，通常是调用一个外部工具，例如 `Search['华为最新款手机']`。
- **Observation (观察)：** 这是执行`Action`后从外部工具返回的结果，例如搜索结果的摘要或API的返回值。

智能体将不断重复这个 **Thought -> Action -> Observation** 的循环，将新的观察结果追加到历史记录中，形成一个不断增长的上下文，直到它在`Thought`中认为已经找到了最终答案，然后输出结果。

这个过程形成了一个强大的协同效应：**推理使得行动更具目的性，而行动则为推理提供了事实依据。**

![ReAct范式中的“思考-行动-观察”协同循环](https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/4-figures/4-1.png)

这种机制特别适用于以下场景：

- **需要外部知识的任务**：如查询实时信息（天气、新闻、股价）、搜索专业领域的知识等。
- **需要精确计算的任务**：将数学问题交给计算器工具，避免LLM的计算错误。
- **需要与API交互的任务**：如操作数据库、调用某个服务的API来完成特定功能。

因此我们将构建一个具备**使用外部工具**能力的ReAct智能体，来回答一个大语言模型仅凭自身知识库无法直接回答的问题。

例如：“华为最新的手机是哪一款？它的主要卖点是什么？” 这个问题需要智能体理解自己需要上网搜索，调用工具搜索结果并总结答案。

#### [ReAct 智能体的编码实现](https://datawhalechina.github.io/hello-agents/#/./chapter4/第四章 智能体经典范式构建?id=_423-react-智能体的编码实现)

现在，我们将所有独立的组件，LLM客户端和工具执行器组装起来，构建一个完整的 ReAct 智能体。

我们将通过一个 `ReActAgent` 类来封装其核心逻辑。

为了便于理解，我们将这个类的实现过程拆分为以下几个关键部分进行讲解。

##### **（1）系统提示词设计**

提示词是整个 ReAct 机制的基石，它为大语言模型提供了行动的操作指令。

我们需要精心设计一个模板，它将动态地插入可用工具、用户问题以及中间步骤的交互历史。

```go
# ReAct 提示词模板
REACT_PROMPT_TEMPLATE = """
请注意，你是一个有能力调用外部工具的智能助手。

可用工具如下:
{tools}

请严格按照以下格式进行回应:

Thought: 你的思考过程，用于分析问题、拆解任务和规划下一步行动。
Action: 你决定采取的行动，必须是以下格式之一:
- `{{tool_name}}[{{tool_input}}]`:调用一个可用工具。
- `Finish[最终答案]`:当你认为已经获得最终答案时。
- 当你收集到足够的信息，能够回答用户的最终问题时，你必须在Action:字段后使用 Finish[最终答案] 来输出最终答案。

现在，请开始解决以下问题:
Question: {question}
History: {history}
"""
```

这个模板定义了智能体与LLM之间交互的规范：

- **角色定义**： “你是一个有能力调用外部工具的智能助手”，设定了LLM的角色。
- **工具清单 (`{tools}`)**： 告知LLM它有哪些可用的“**手脚**”。
- **格式规约 (`Thought`/`Action`)**： 这是最重要的部分，它**强制LLM的输出具有结构性**，使我们能通过代码精确解析其意图。
- **动态上下文 (`{question}`/`{history}`)**： 将用户的原始问题和不断累积的交互历史注入，让LLM基于完整的上下文进行决策。

##### **（2）核心循环的实现**

`ReActAgent` 的核心是一个循环，它不断地“**格式化提示词 -> 调用LLM -> 执行动作 -> 整合结果**”，直到任务完成或达到最大步数限制。

```go
class ReActAgent:
    def __init__(self, llm_client: HelloAgentsLLM, tool_executor: ToolExecutor, max_steps: int = 5):
        self.llm_client = llm_client
        self.tool_executor = tool_executor
        self.max_steps = max_steps
        self.history = []

    def run(self, question: str):
        """
        运行ReAct智能体来回答一个问题。
        """
        self.history = [] # 每次运行时重置历史记录
        current_step = 0

        while current_step < self.max_steps:
            current_step += 1
            print(f"--- 第 {current_step} 步 ---")

            # 1. 格式化提示词
            tools_desc = self.tool_executor.getAvailableTools()
            history_str = "\n".join(self.history)
            prompt = REACT_PROMPT_TEMPLATE.format(
                tools=tools_desc,
                question=question,
                history=history_str
            )

            # 2. 调用LLM进行思考
            messages = [{"role": "user", "content": prompt}]
            response_text = self.llm_client.think(messages=messages)
            
            if not response_text:
                print("错误:LLM未能返回有效响应。")
                break

            # ... (后续的解析、执行、整合步骤)
```

`run` 方法是智能体的入口。

它的 `while` 循环构成了 ReAct 范式的主体，`max_steps` 参数则是一个重要的安全阀，防止智能体陷入无限循环而耗尽资源。



**golang**：

**ReAct 实际上是用文本（Prompt）诱导模型控制流程**

定义核心抽象类 

```go
package main

import (
	"context"
	"fmt"
	"strings"
)

// LLMClient 定义大模型的能力接口
type LLMClient interface {
	Think(ctx context.Context, prompt string) (string, error)
}

// ToolExecutor 定义工具执行器的能力接口
type ToolExecutor interface {
	GetAvailableTools() string
	Execute(toolName string, args string) (string, error)
}
```

核心结构体

```go
// ReActAgent 结构体
type ReActAgent struct {
	llm      LLMClient
	executor ToolExecutor
	maxSteps int
	history  []string // 用于存储 Thought -> Action -> Observation 的链路
}

// NewReActAgent 构造函数 (对应 __init__)
func NewReActAgent(llm LLMClient, executor ToolExecutor, maxSteps int) *ReActAgent {
	return &ReActAgent{
		llm:      llm,
		executor: executor,
		maxSteps: maxSteps,
		history:  make([]string, 0),
	}
}
```

核心逻辑

这是最关键的部分。请注意如何处理 Go 的 `Context`、错误处理以及循环控制。

```go
const ReactPromptTemplate = `
你是一个 ReAct 智能体。请按照以下格式进行思考：
1. Thought: 思考当前需要做什么
2. Action: 工具名称 (参数)  <-- 如果需要调用工具
3. Observation: 工具返回的结果 <-- 这一步由系统补充

当你知道最终答案时，请返回：
Final Answer: [你的答案]

可用工具:
%s

问题: %s
历史记录:
%s
`

// Run 运行智能体 (对应 def run)
func (a *ReActAgent) Run(ctx context.Context, question string) (string, error) {
	// 每次运行重置历史 (对应 self.history = [])
	a.history = []string{}
	currentStep := 0

	for currentStep < a.maxSteps {
		currentStep++
		fmt.Printf("--- 第 %d 步 ---\n", currentStep)

		// 1. 格式化提示词 (Construct Prompt)
		toolsDesc := a.executor.GetAvailableTools()
		historyStr := strings.Join(a.history, "\n")
		prompt := fmt.Sprintf(ReactPromptTemplate, toolsDesc, question, historyStr)

		// 2. 调用 LLM 进行思考 (Think)
		response, err := a.llm.Think(ctx, prompt)
		if err != nil {
			return "", fmt.Errorf("LLM error: %w", err)
		}
		
		fmt.Printf("[LLM Response]: %s\n", response)

		// 3. 解析 LLM 的响应
		// 情况 A: LLM 决定给出最终答案
		if strings.Contains(response, "Final Answer:") {
			finalAnswer := extractFinalAnswer(response)
			return finalAnswer, nil
		}

		// 情况 B: LLM 想要调用工具 (Action)
		// 假设 LLM 返回格式为 "Action: Search(天气)"
		action, args, hasAction := parseAction(response)
		if !hasAction {
			// 如果没解析出动作，也没最终答案，可能是 LLM 还在碎碎念，强制让它继续或报错
			fmt.Println("未检测到有效动作，继续思考...")
			a.history = append(a.history, fmt.Sprintf("Thought: %s", response))
			continue
		}

		// 记录 LLM 的思考和动作决策到历史
		a.history = append(a.history, response) 

		// 4. 执行工具 (Act)
		fmt.Printf("[Executing Tool]: %s Args: %s\n", action, args)
		result, err := a.executor.Execute(action, args)
		if err != nil {
			result = fmt.Sprintf("Error executing tool: %v", err)
		}

		// 5. 观察结果 (Observe) 并追加到历史
		observation := fmt.Sprintf("Observation: %s", result)
		fmt.Println(observation)
		a.history = append(a.history, observation)
	}

	return "", fmt.Errorf("也就是超过了最大步数 %d 仍未找到答案", a.maxSteps)
}

// ---------------- 辅助函数 (模拟 Python 里的解析逻辑) ----------------

func extractFinalAnswer(text string) string {
	parts := strings.Split(text, "Final Answer:")
	if len(parts) > 1 {
		return strings.TrimSpace(parts[1])
	}
	return text
}

func parseAction(text string) (string, string, bool) {
	// 这里做一个极简的解析器，实际项目中可能用 Regex 或更强的 String 处理
	// 假设格式是: Action: ToolName(Args)
	if !strings.Contains(text, "Action:") {
		return "", "", false
	}
    
    // 简单的字符串切割逻辑 (仅作演示)
	// 实际开发中，建议让 LLM 返回 JSON 格式，解析会更稳定
	lines := strings.Split(text, "\n")
	for _, line := range lines {
		if strings.HasPrefix(line, "Action:") {
			content := strings.TrimPrefix(line, "Action:")
			content = strings.TrimSpace(content)
            // 简单拆分 Tool 和 Args (假设用括号包裹)
            // 实际这里需要复杂的正则匹配
            parts := strings.SplitN(content, "(", 2)
            if len(parts) == 2 {
                toolName := strings.TrimSpace(parts[0])
                args := strings.TrimSuffix(parts[1], ")")
                return toolName, args, true
            }
		}
	}
	return "", "", false
}
```

Go vs Python 实现的关键区别 

**接口隔离 (`Interface`)**：

- Python 里直接传对象。
- Go 代码中，`ReActAgent` 不关心 `llm` 具体是 OpenAI 还是 DeepSeek，也不关心 `executor` 是本地函数还是 HTTP 代理。
	- 这符合 Go 的 **Dependency Injection** 最佳实践。

**上下文控制 (`context.Context`)**：

- 在 `Run` 和 `Think` 方法中加入了 `ctx`。这是 Go 后端的标配。
- 如果用户取消了请求，或者超时了，这个 Context 会级联取消 LLM 的请求，防止 Goroutine 泄漏。

**错误处理 (`error`)**：

- Go 显式处理 `error`。
	- 比如工具执行失败，是直接 `return error` 还是把错误信息作为 `Observation` 喂回给 LLM？
- **Agent 开发的一个 Trick**：通常工具报错时，**不要 `return err` 崩掉程序**，而是把错误字符串作为 `Observation` 告诉 LLM（代码中第 65 行）。这样 LLM 看到错误后，可能会尝试自我修正参数。

**结构化历史 (`history`)**：

- Python 里的 `self.history = []` 很随意。
- 在 Go 中，我们用 `[]string` 只是为了演示。在实际 Eino 开发中，这里通常是一个 `[]*schema.Message` 类型的切片，包含 `Role` (User/Assistant/System/Tool) 字段。

##### **（3）输出解析器的实现**

LLM 返回的是纯文本，我们需要从中精确地提取出`Thought`和`Action`。

这是通过几个辅助解析函数完成的，它们通常使用正则表达式来实现。

```go
# (这些方法是 ReActAgent 类的一部分)
    def _parse_output(self, text: str):
        """解析LLM的输出，提取Thought和Action。
        """
        # Thought: 匹配到 Action: 或文本末尾
        thought_match = re.search(r"Thought:\s*(.*?)(?=\nAction:|$)", text, re.DOTALL)
        # Action: 匹配到文本末尾
        action_match = re.search(r"Action:\s*(.*?)$", text, re.DOTALL)
        thought = thought_match.group(1).strip() if thought_match else None
        action = action_match.group(1).strip() if action_match else None
        return thought, action

    def _parse_action(self, action_text: str):
        """解析Action字符串，提取工具名称和输入。
        """
        match = re.match(r"(\w+)\[(.*)\]", action_text, re.DOTALL)
        if match:
            return match.group(1), match.group(2)
        return None, None
```

- `_parse_output`： 负责从LLM的完整响应中分离出`Thought`和`Action`两个主要部分。
- `_parse_action`： 负责进一步解析`Action`字符串，例如从 `Search[华为最新手机]` 中提取出工具名 `Search` 和工具输入 `华为最新手机`。

##### (4) 工具调用与执行

```go

```

#### [ReAct 的特点、局限性与调试技巧](https://datawhalechina.github.io/hello-agents/#/./chapter4/第四章 智能体经典范式构建?id=_424-react-的特点、局限性与调试技巧)

通过亲手实现一个 ReAct 智能体，我们不仅掌握了其工作流程，也应该对其内在机制有了更深刻的认识。任何技术范式都有其闪光点和待改进之处，本节将对 ReAct 进行总结。

（1）ReAct 的主要特点

1. **高可解释性**：ReAct 最大的优点之一就是透明。通过 `Thought` 链，我们可以清晰地看到智能体每一步的“心路历程”——它为什么会选择这个工具，下一步又打算做什么。这对于理解、信任和调试智能体的行为至关重要。
2. **动态规划与纠错能力**：与一次性生成完整计划的范式不同，ReAct 是“走一步，看一步”。它根据每一步从外部世界获得的 `Observation` 来动态调整后续的 `Thought` 和 `Action`。如果上一步的搜索结果不理想，它可以在下一步中修正搜索词，重新尝试。
3. **工具协同能力**：ReAct 范式天然地将大语言模型的推理能力与外部工具的执行能力结合起来。LLM 负责运筹帷幄（规划和推理），工具负责解决具体问题（搜索、计算），二者协同工作，突破了单一 LLM 在知识时效性、计算准确性等方面的固有局限。

（2）ReAct 的固有局限性

1. **对LLM自身能力的强依赖**：ReAct 流程的成功与否，高度依赖于底层 LLM 的综合能力。如果 LLM 的逻辑推理能力、指令遵循能力或格式化输出能力不足，就很容易在 `Thought` 环节产生错误的规划，或者在 `Action` 环节生成不符合格式的指令，导致整个流程中断。
2. **执行效率问题**：由于其循序渐进的特性，完成一个任务通常需要多次调用 LLM。每一次调用都伴随着网络延迟和计算成本。对于需要很多步骤的复杂任务，这种串行的“思考-行动”循环可能会导致较高的总耗时和费用。
3. **提示词的脆弱性**：整个机制的稳定运行建立在一个精心设计的提示词模板之上。模板中的任何微小变动，甚至是用词的差异，都可能影响 LLM 的行为。此外，并非所有模型都能持续稳定地遵循预设的格式，这增加了在实际应用中的不确定性。
4. **可能陷入局部最优**：步进式的决策模式意味着智能体缺乏一个全局的、长远的规划。它可能会因为眼前的 `Observation` 而选择一个看似正确但长远来看并非最优的路径，甚至在某些情况下陷入“原地打转”的循环中。

（3）调试技巧

当你构建的 ReAct 智能体行为不符合预期时，可以从以下几个方面入手进行调试：

- **检查完整的提示词**：在每次调用 LLM 之前，将最终格式化好的、包含所有历史记录的完整提示词打印出来。这是追溯 LLM 决策源头的最直接方式。
- **分析原始输出**：当输出解析失败时（例如，正则表达式没有匹配到 `Action`），务必将 LLM 返回的原始、未经处理的文本打印出来。这能帮助你判断是 LLM 没有遵循格式，还是你的解析逻辑有误。
- **验证工具的输入与输出**：检查智能体生成的 `tool_input` 是否是工具函数所期望的格式，同时也要确保工具返回的 `observation` 格式是智能体可以理解和处理的。
- **调整提示词中的示例 (Few-shot Prompting)**：如果模型频繁出错，可以在提示词中加入一两个完整的“Thought-Action-Observation”成功案例，通过示例来引导模型更好地遵循你的指令。
- **尝试不同的模型或参数**：更换一个能力更强的模型，或者调整 `temperature` 参数（通常设为0以保证输出的确定性），有时能直接解决问题。

### [Plan-and-Solve](https://datawhalechina.github.io/hello-agents/#/./chapter4/第四章 智能体经典范式构建?id=_43-plan-and-solve)

在我们掌握了 ReAct 这种反应式的、步进决策的智能体范式后，接下来将探讨一种风格迥异但同样强大的方法，**Plan-and-Solve**。

顾名思义，这种范式将任务处理明确地分为两个阶段：**先规划 (Plan)，后执行 (Solve)**。

如果说 ReAct 像一个经验丰富的侦探，根据现场的蛛丝马迹（Observation）一步步推理，随时调整自己的调查方向；

那么 Plan-and-Solve 则更像一位建筑师，在动工之前必须先绘制出完整的蓝图（Plan），然后严格按照蓝图来施工（Solve）。

**事实上我们现在用的很多大模型工具的Agent模式都融入了这种设计模式。**

#### [Plan-and-Solve 的工作原理](https://datawhalechina.github.io/hello-agents/#/./chapter4/第四章 智能体经典范式构建?id=_431-plan-and-solve-的工作原理)

Plan-and-Solve Prompting 由 Lei Wang 在2023年提出[2]。其核心动机是为了解决思维链在处理多步骤、复杂问题时容易“偏离轨道”的问题。

与 ReAct 将思考和行动融合在每一步不同，Plan-and-Solve 将整个流程解耦为两个核心阶段，如图4.2所示：

1. **规划阶段 (Planning Phase)**： 首先，智能体会接收用户的完整问题。它的第一个任务不是直接去解决问题或调用工具，而是**将问题分解，并制定出一个清晰、分步骤的行动计划**。这个计划本身就是一次大语言模型的调用产物。
2. **执行阶段 (Solving Phase)**： 在获得完整的计划后，智能体进入执行阶段。它会**严格按照计划中的步骤，逐一执行**。每一步的执行都可能是一次独立的 LLM 调用，或者是对上一步结果的加工处理，直到计划中的所有步骤都完成，最终得出答案。

这种“先谋后动”的策略，使得智能体在处理需要长远规划的复杂任务时，能够保持更高的目标一致性，避免在中间步骤中迷失方向。

![Plan-and-Solve范式的两阶段工作流](https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/4-figures/4-2.png)

Plan-and-Solve 尤其适用于那些结构性强、可以被清晰分解的复杂任务，例如：

- **多步数学应用题**：需要先列出计算步骤，再逐一求解。
- **需要整合多个信息源的报告撰写**：需要先规划好报告结构（引言、数据来源A、数据来源B、总结），再逐一填充内容。
- **代码生成任务**：需要先构思好函数、类和模块的结构，再逐一实现。

#### [规划阶段](https://datawhalechina.github.io/hello-agents/#/./chapter4/第四章 智能体经典范式构建?id=_432-规划阶段)

为了凸显 Plan-and-Solve 范式在结构化推理任务上的优势，我们将不使用工具的方式，而是通过提示词的设计，完成一个推理任务。

这类任务的特点是，答案无法通过单次查询或计算得出，必须先将问题分解为一系列逻辑连贯的子步骤，然后按顺序求解。

这恰好能发挥 Plan-and-Solve “先规划，后执行”的核心能力。

**我们的目标问题是：**“一个水果店周一卖出了15个苹果。周二卖出的苹果数量是周一的两倍。周三卖出的数量比周二少了5个。请问这三天总共卖出了多少个苹果？”

这个问题对于大语言模型来说并不算特别困难，但它包含了一个清晰的逻辑链条可供参考。在某些实际的逻辑难题上，如果大模型不能高质量的推理出准确的答案，可以参考这个设计模式来设计自己的Agent完成任务。

智能体需要：

1. **规划阶段**：首先，将问题分解为三个独立的计算步骤（计算周二销量、计算周三销量、计算总销量）。
2. **执行阶段**：然后，严格按照计划，一步步执行计算，并将每一步的结果作为下一步的输入，最终得出总和。

规划阶段的目标是让大语言模型接收原始问题，并输出一个清晰、分步骤的行动计划。这个计划必须是结构化的，以便我们的代码可以轻松解析并逐一执行。因此，我们设计的提示词需要明确地告诉模型它的角色和任务，并给出一个输出格式的范例。

~~~go
PLANNER_PROMPT_TEMPLATE = """
你是一个顶级的AI规划专家。你的任务是将用户提出的复杂问题分解成一个由多个简单步骤组成的行动计划。
请确保计划中的每个步骤都是一个独立的、可执行的子任务，并且严格按照逻辑顺序排列。
你的输出必须是一个Python列表，其中每个元素都是一个描述子任务的字符串。

问题: {question}

请严格按照以下格式输出你的计划,```python与```作为前后缀是必要的:
```python
["步骤1", "步骤2", "步骤3", ...]
```
"""
~~~

这个提示词通过以下几点确保了输出的质量和稳定性：

- **角色设定**： “顶级的AI规划专家”，激发模型的专业能力。
- **任务描述**： 清晰地定义了“分解问题”的目标。
- **格式约束**： 强制要求输出为一个 Python 列表格式的字符串，这极大地简化了后续代码的解析工作，使其比解析自然语言更稳定、更可靠。

接下来，我们将这个提示词逻辑封装成一个 `Planner` 类，这个类也是我们的规划器。

```
# 假定 llm_client.py 中的 HelloAgentsLLM 类已经定义好
# from llm_client import HelloAgentsLLM

class Planner:
    def __init__(self, llm_client):
        self.llm_client = llm_client

    def plan(self, question: str) -> list[str]:
        """
        根据用户问题生成一个行动计划。
        """
        prompt = PLANNER_PROMPT_TEMPLATE.format(question=question)
        
        # 为了生成计划，我们构建一个简单的消息列表
        messages = [{"role": "user", "content": prompt}]
        
        print("--- 正在生成计划 ---")
        # 使用流式输出来获取完整的计划
        response_text = self.llm_client.think(messages=messages) or ""
        
        print(f"✅ 计划已生成:\n{response_text}")
        
        # 解析LLM输出的列表字符串
        try:
            # 找到```python和```之间的内容
            plan_str = response_text.split("```python")[1].split("```")[0].strip()
            # 使用ast.literal_eval来安全地执行字符串，将其转换为Python列表
            plan = ast.literal_eval(plan_str)
            return plan if isinstance(plan, list) else []
        except (ValueError, SyntaxError, IndexError) as e:
            print(f"❌ 解析计划时出错: {e}")
            print(f"原始响应: {response_text}")
            return []
        except Exception as e:
            print(f"❌ 解析计划时发生未知错误: {e}")
            return []
```

#### [执行器与状态管理](https://datawhalechina.github.io/hello-agents/#/./chapter4/第四章 智能体经典范式构建?id=_433-执行器与状态管理)

在规划器 (`Planner`) 生成了清晰的行动蓝图后，我们就需要一个执行器 (`Executor`) 来逐一完成计划中的任务。

执行器不仅负责调用大语言模型来解决每个子问题，还承担着一个至关重要的角色：**状态管理**。

它必须记录每一步的执行结果，并将其作为上下文提供给后续步骤，确保信息在整个任务链条中顺畅流动

执行器的提示词与规划器不同。

它的目标不是分解问题，而是**在已有上下文的基础上，专注解决当前这一个步骤**。

因此，提示词需要包含以下关键信息：

- **原始问题**： 确保模型始终了解最终目标。
- **完整计划**： 让模型了解当前步骤在整个任务中的位置。
- **历史步骤与结果**： 提供至今为止已经完成的工作，作为当前步骤的直接输入。
- **当前步骤**： 明确指示模型现在需要解决哪一个具体任务。

```
EXECUTOR_PROMPT_TEMPLATE = """
你是一位顶级的AI执行专家。你的任务是严格按照给定的计划，一步步地解决问题。
你将收到原始问题、完整的计划、以及到目前为止已经完成的步骤和结果。
请你专注于解决“当前步骤”，并仅输出该步骤的最终答案，不要输出任何额外的解释或对话。

# 原始问题:
{question}

# 完整计划:
{plan}

# 历史步骤与结果:
{history}

# 当前步骤:
{current_step}

请仅输出针对“当前步骤”的回答:
"""
```

我们将执行逻辑封装到 `Executor` 类中。这个类将循环遍历计划，调用 LLM，并维护一个历史记录（状态）。

```go
class Executor:
    def __init__(self, llm_client):
        self.llm_client = llm_client

    def execute(self, question: str, plan: list[str]) -> str:
        """
        根据计划，逐步执行并解决问题。
        """
        history = "" # 用于存储历史步骤和结果的字符串
        
        print("\n--- 正在执行计划 ---")
        
        for i, step in enumerate(plan):
            print(f"\n-> 正在执行步骤 {i+1}/{len(plan)}: {step}")
            
            prompt = EXECUTOR_PROMPT_TEMPLATE.format(
                question=question,
                plan=plan,
                history=history if history else "无", # 如果是第一步，则历史为空
                current_step=step
            )
            
            messages = [{"role": "user", "content": prompt}]
            
            response_text = self.llm_client.think(messages=messages) or ""
            
            # 更新历史记录，为下一步做准备
            history += f"步骤 {i+1}: {step}\n结果: {response_text}\n\n"
            
            print(f"✅ 步骤 {i+1} 已完成，结果: {response_text}")

        # 循环结束后，最后一步的响应就是最终答案
        final_answer = response_text
        return final_answer
```

现在已经分别构建了负责“规划”的 `Planner` 和负责“执行”的 `Executor`。最后一步是将这两个组件整合到一个统一的智能体 `PlanAndSolveAgent` 中，并赋予它解决问题的完整能力。我们将创建一个主类 `PlanAndSolveAgent`，它的职责非常清晰：接收一个 LLM 客户端，初始化内部的规划器和执行器，并提供一个简单的 `run` 方法来启动整个流程。

# 第三部分：高级知识扩展

## 第八章 记忆和探索

如果智能体无法记住之前的交互内容，也无法从历史经验中学习，那么在连续对话或复杂任务中，其表现将受到极大限制。

本章将在第七章构建的框架基础上，为HelloAgents增加两个核心能力：

**记忆系统（Memory System）**和**检索增强生成（Retrieval-Augmented Generation, RAG）**。

我们将采用"框架扩展 + 知识科普"的方式，在构建过程中深入理解Memory和RAG的理论基础，最终实现一个具有完整记忆和知识检索能力的智能体系统。

### [从认知科学到智能体记忆](https://datawhalechina.github.io/hello-agents/#/./chapter8/第八章 记忆与检索?id=_81-从认知科学到智能体记忆)

#### [为何智能体需要记忆与RAG](https://datawhalechina.github.io/hello-agents/#/./chapter8/第八章 记忆与检索?id=_812-为何智能体需要记忆与rag)

​	人类智能的一个重要特征就是能够记住过去的经历，从中学习，并将这些经验应用到新的情况中。

​	同样，一个真正智能的智能体也需要具备记忆能力。

​	对于基于LLM的智能体而言，通常面临两个根本性局限：**对话状态的遗忘**和**内置知识的局限**。

（1）局限一：无状态导致的对话遗忘

​	当前的大语言模型虽然强大，但设计上是**无状态的**。

​	这意味着，每一次用户请求（或API调用）都是一次独立的、无关联的计算。模型本身不会自动“记住”上一次对话的内容。

这带来了几个问题：

1. **上下文丢失**：在长对话中，早期的重要信息可能会因为上下文窗口限制而丢失
2. **个性化缺失**：Agent无法记住用户的偏好、习惯或特定需求
3. **学习能力受限**：无法从过往的成功或失败经验中学习改进
4. **一致性问题**：在多轮对话中可能出现前后矛盾的回答

要解决这个问题，我们的框架需要引入记忆系统。

（2）局限二：模型内置知识的局限性

除了遗忘对话历史，LLM 的另一个核心局限在于其知识是**静态的、有限的**。这些知识完全来自于它的训练数据，并因此带来一系列问题：

1. **知识时效性**：大模型的训练数据有时间截止点，无法获取最新信息
2. **专业领域知识**：通用模型在特定领域的深度知识可能不足
3. **事实准确性**：通过检索验证，减少模型的幻觉问题
4. **可解释性**：提供信息来源，增强回答的可信度

为了克服这一局限，**RAG技术应运而生**。

它的核心思想是**在模型生成回答之前，先从一个外部知识库（如文档、数据库、API）中检索出最相关的信息，并将这些信息作为上下文一同提供给模型。**

#### [记忆与RAG系统架构设计](https://datawhalechina.github.io/hello-agents/#/./chapter8/第八章 记忆与检索?id=_813-记忆与rag系统架构设计)

![HelloAgents记忆与RAG系统架构图](https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/8-figures/8-2.png)

在实现上，我们将记忆和RAG设计为两个独立的工具：

`memory_tool`负责存储和维护对话过程中的交互信息，`rag_tool`则负责从用户提供的知识库中检索相关信息作为上下文，并可将重要的检索结果自动存储到记忆系统中。

 记忆系统采用了四层架构设计：

```go
HelloAgents记忆系统
├── 基础设施层 (Infrastructure Layer)
│   ├── MemoryManager - 记忆管理器（统一调度和协调）
│   ├── MemoryItem - 记忆数据结构（标准化记忆项）
│   ├── MemoryConfig - 配置管理（系统参数设置）
│   └── BaseMemory - 记忆基类（通用接口定义）
├── 记忆类型层 (Memory Types Layer)
│   ├── WorkingMemory - 工作记忆（临时信息，TTL管理）
│   ├── EpisodicMemory - 情景记忆（具体事件，时间序列）
│   ├── SemanticMemory - 语义记忆（抽象知识，图谱关系）
│   └── PerceptualMemory - 感知记忆（多模态数据）
├── 存储后端层 (Storage Backend Layer)
│   ├── QdrantVectorStore - 向量存储（高性能语义检索）
│   ├── Neo4jGraphStore - 图存储（知识图谱管理）
│   └── SQLiteDocumentStore - 文档存储（结构化持久化）
└── 嵌入服务层 (Embedding Service Layer)
    ├── DashScopeEmbedding - 通义千问嵌入（云端API）
    ├── LocalTransformerEmbedding - 本地嵌入（离线部署）
    └── TFIDFEmbedding - TFIDF嵌入（轻量级兜底）
```

**RAG系统专注于外部知识的获取和利用：**

```go
HelloAgents RAG系统
├── 文档处理层 (Document Processing Layer)
│   ├── DocumentProcessor - 文档处理器（多格式解析）
│   ├── Document - 文档对象（元数据管理）
│   └── Pipeline - RAG管道（端到端处理）
├── 嵌入表示层 (Embedding Layer)
│   └── 统一嵌入接口 - 复用记忆系统的嵌入服务
├── 向量存储层 (Vector Storage Layer)
│   └── QdrantVectorStore - 向量数据库（命名空间隔离）
└── 智能问答层 (Intelligent Q&A Layer)
    ├── 多策略检索 - 向量检索 + MQE + HyDE
    ├── 上下文构建 - 智能片段合并与截断
    └── LLM增强生成 - 基于上下文的准确问答
```

### [RAG系统：知识检索增强](https://datawhalechina.github.io/hello-agents/#/./chapter8/第八章 记忆与检索?id=_83-rag系统：知识检索增强)

#### [RAG的基础知识](https://datawhalechina.github.io/hello-agents/#/./chapter8/第八章 记忆与检索?id=_831-rag的基础知识)

在深入HelloAgents的RAG系统实现之前，让我们先了解RAG技术的基础概念、发展历程和核心原理。由于本文内容不是以RAG为基础进行创作，为此这里只帮读者快速梳理相关概念，以便更好地理解系统设计的技术选择和创新点。

（1）什么是RAG？

**检索增强生成**（Retrieval-Augmented Generation，RAG）是一种**结合了信息检索和文本生成的技术**。

它的核心思想是：

**在生成回答之前，先从外部知识库中检索相关信息，然后将检索到的信息作为上下文提供给大语言模型，从而生成更准确、更可靠的回答。**

因此，检索增强生成可以拆分为三个词汇。

**检索**是指从知识库中查询相关内容；

**增强**是将检索结果融入提示词，辅助模型生成；

**生成**则输出兼具准确性与透明度的答案。

（2）基本工作流程

一个完整的RAG应用流程主要分为两大核心环节。

在**数据准备阶段**，系统通过**数据提取**、**文本分割**和**向量化**，将外部知识构建成一个可检索的数据库。

随后在**应用阶段**，系统会响应用户的**提问**，从数据库中**检索**相关信息，将其**注入Prompt**，并最终驱动大语言模型**生成答案**。

#### [RAG系统工作原理](https://datawhalechina.github.io/hello-agents/#/./chapter8/第八章 记忆与检索?id=_832-rag系统工作原理)

在深入实现细节之前，可以通过流程图来梳理Helloagents的RAG系统完整工作流程：

![RAG系统核心原理](https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/8-figures/8-5.png)

如图8.5所示，展示了RAG系统的两个主要工作模式：

1. **数据处理流程**：处理和存储知识文档，在这里我们采取工具`Markitdown`，设计思路是将传入的一切外部知识源统一转化为Markdown格式进行处理。
2. **查询与生成流程**：根据查询检索相关信息并生成回答。

#### [ RAG系统架构设计](https://datawhalechina.github.io/hello-agents/#/./chapter8/第八章 记忆与检索?id=_834-rag系统架构设计)

在这一节中，我们采取与记忆系统不同的方式讲解。因为`Memory_tool`是系统性的实现，而RAG在我们的设计中被定义为一种工具，可以梳理为一条pipeline。

我们的RAG系统的核心架构可以概括为"五层七步"的设计模式：

```go
用户层：RAGTool统一接口
  ↓
应用层：智能问答、搜索、管理
  ↓  
处理层：文档解析、分块、向量化
  ↓
存储层：向量数据库、文档存储
  ↓
基础层：嵌入模型、LLM、数据库
```

这种分层设计的优势在于每一层都可以独立优化和替换，同时保持整体系统的稳定性。例如，可以轻松地将嵌入模型从sentence-transformers切换到百炼API，而不影响上层的业务逻辑。同样的，这些处理的流程代码是完全可复用的，也可以选取自己需要的部分放进自己的项目中。RAGTool作为RAG系统的统一入口，提供了简洁的API接口。

```go

```

#### [ 高级检索策略](https://datawhalechina.github.io/hello-agents/#/./chapter8/第八章 记忆与检索?id=_835-高级检索策略)

RAG系统的检索能力是其核心竞争力。

在实际应用中，用户的查询表述与文档中的实际内容可能存在用词差异，导致相关文档无法被检索到。

为了解决这个问题，HelloAgents实现了三种互补的高级检索策略：

**多查询扩展（MQE）、假设文档嵌入（HyDE）和统一的扩展检索框架。**

1）多查询扩展（MQE）

多查询扩展（Multi-Query Expansion）是一种通过生成语义等价的多样化查询来提高检索召回率的技术。

这种方法的核心洞察是：

同一个问题可以有多种不同的表述方式，而不同的表述可能匹配到不同的相关文档。

例如，"如何学习Python"可以扩展为"Python入门教程"、"Python学习方法"、"Python编程指南"等多个查询。

通过并行执行这些扩展查询并合并结果，系统能够覆盖更广泛的相关文档，避免因用词差异而遗漏重要信息。

MQE的优势在于它能够自动理解用户查询的多种可能含义，特别是对于模糊查询或专业术语查询效果显著。系统使用LLM生成扩展查询，确保扩展的多样性和语义相关性：

```
def _prompt_mqe(query: str, n: int) -> List[str]:
    """使用LLM生成多样化的查询扩展"""
    try:
        from ...core.llm import HelloAgentsLLM
        llm = HelloAgentsLLM()
        prompt = [
            {"role": "system", "content": "你是检索查询扩展助手。生成语义等价或互补的多样化查询。使用中文，简短，避免标点。"},
            {"role": "user", "content": f"原始查询：{query}\n请给出{n}个不同表述的查询，每行一个。"}
        ]
        text = llm.invoke(prompt)
        lines = [ln.strip("- \t") for ln in (text or "").splitlines()]
        outs = [ln for ln in lines if ln]
        return outs[:n] or [query]
    except Exception:
        return [query]

```

2）假设文档嵌入（HyDE）

假设文档嵌入（Hypothetical Document Embeddings，HyDE）是一种创新的检索技术，它的核心思想是"用答案找答案"。传统的检索方法是用问题去匹配文档，但问题和答案在语义空间中的分布往往存在差异——问题通常是疑问句，而文档内容是陈述句。HyDE通过让LLM先生成一个假设性的答案段落，然后用这个答案段落去检索真实文档，从而缩小了查询和文档之间的语义鸿沟。

这种方法的优势在于，假设答案与真实答案在语义空间中更加接近，因此能够更准确地匹配到相关文档。即使假设答案的内容不完全正确，它所包含的关键术语、概念和表述风格也能有效引导检索系统找到正确的文档。特别是对于专业领域的查询，HyDE能够生成包含领域术语的假设文档，显著提升检索精度：