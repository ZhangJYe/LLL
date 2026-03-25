# FreeExchanged

### 1. 项目概述 & 核心功能（匹配简历）

- **实时汇率**：每30秒从免费API拉取（exchangerate-api.com），推送到Redis缓存 + RabbitMQ广播。
- **金融资讯**：文章CRUD、实时推送。
- **多维度排行榜**：Redis ZSet（按涨跌幅、交易量、热度等）。
- **用户自选**：收藏货币对，实时提醒。
- **互动**：点赞/评论文章或汇率。
- **用户系统**：注册/登录 + Paseto token。
- **实时特性**：WebSocket推送汇率/资讯变化。
- **云原生**：服务拆分、Consul发现、限流熔断、Prometheus监控、K8s HPA自动扩缩容。

**技术栈**（完全对齐简历）：

- **框架**：go-zero (API Gateway + zRPC)
- **发现**：Consul
- **存储**：MySQL（业务数据） + Redis（缓存、ZSet排行）
- **消息**：RabbitMQ（事件驱动：汇率更新 → 更新排行 → 推送）
- **鉴权**：Paseto (比JWT更安全，无需secret共享)
- **监控**：Prometheus + Grafana + go-zero内置metrics
- **部署**：Docker + K8s（goctl自动生成）

### 2.整体架构图（你简历里可以画这个）

**客户端 → API Gateway (go-zero rest, Paseto + WebSocket) → Consul发现 → 6个 zRPC 服务** 

后端：MySQL / Redis / RabbitMQ 

监控：Prometheus scrape 所有服务 → Grafana + HPA

```go
freeexchanged/
├── desc/                               # 所有接口定义文件集中存放
│   ├── gateway.api                     # RESTful API 定义（gateway 专用）
│   ├── user.proto
│   ├── rate.proto
│   ├── ranking.proto
│   ├── article.proto
│   ├── interaction.proto
│   └── favorite.proto
│
├── app/                                # 所有业务模块（领域划分）
│   ├── gateway/                        # API 网关（对外暴露 HTTP/REST + WebSocket）
│   │   ├── etc/
│   │   │   └── gateway.yaml
│   │   ├── internal/
│   │   │   ├── config/
│   │   │   ├── handler/                # goctl 生成的 handler
│   │   │   ├── logic/                  # 主要做参数校验 + RPC 调用组装
│   │   │   ├── middleware/             # PasetoMiddleware、限流等
│   │   │   ├── svc/                    # 服务上下文（注入各个 rpc client）
│   │   │   └── types/                  # goctl 生成的 req/resp struct
│   │   └── gateway.go                  # 主程序入口
│   │
│   ├── user/                           # 用户域
│   │   ├── cmd/
│   │   │   └── rpc/
│   │   │       ├── internal/
│   │   │       │   ├── config/
│   │   │       │   ├── logic/
│   │   │       │   ├── server/
│   │   │       │   └── svc/
│   │   │       ├── user.proto          # 实际 proto 文件已移到 desc/
│   │   │       └── user.go             # rpc 服务启动入口
│   │   ├── model/                      # user 专属的 model（mysql、cache 等）
│   │   └── etc/
│   │       └── user-rpc.yaml
│   │
│   ├── rate/                           # 汇率域
│   │   ├── cmd/
│   │   │   ├── rpc/                    # 汇率查询、历史等 rpc 服务
│   │   │   │   └── ...                 # 结构同 user/cmd/rpc
│   │   │   └── job/                    # 定时拉取汇率的任务（独立进程）
│   │   │       ├── internal/
│   │   │       │   └── logic/
│   │   │       ├── ratejob.go
│   │   │       └── etc/
│   │   │           └── job.yaml
│   │   ├── model/                      # rate 专属 model
│   │   └── etc/
│   │       └── rate-rpc.yaml
│   │
│   ├── ranking/                        # 排行榜域
│   │   ├── cmd/
│   │   │   └── rpc/                    # 排行榜查询 rpc
│   │   ├── internal/
│   │   │   └── mq/                     # 消费者（监听事件更新 redis zset）
│   │   │       ├── logic/
│   │   │       └── consumer.go         # mq 消费主程序
│   │   ├── model/                      # redis zset 操作封装
│   │   └── etc/
│   │       ├── ranking-rpc.yaml
│   │       └── mq.yaml                 # rabbitmq 配置
│   │
│   ├── article/                        # 资讯/文章域
│   │   ├── cmd/
│   │   │   └── rpc/
│   │   ├── model/
│   │   └── etc/
│   │
│   ├── interaction/                    # 互动（点赞、评论）域
│   │   ├── cmd/
│   │   │   └── rpc/
│   │   ├── internal/
│   │   │   └── mq/                     # 监听点赞/评论事件
│   │   ├── model/
│   │   └── etc/
│   │
│   └── favorite/                       # 用户自选收藏域
│       ├── cmd/
│       │   └── rpc/
│       ├── model/
│       └── etc/
│
├── pkg/                                # 真正跨域共享的代码（非常克制）
│   ├── xerr/                           # 全局业务错误码
│   │   └── err.go
│   ├── interceptor/                    # rpc / http 通用拦截器
│   │   ├── rpc/
│   │   └── http/
│   ├── utils/                          # 通用工具函数（时间、加密、id 生成等）
│   └── event/                          # 所有 mq 事件的消息结构体定义
│       ├── rate.go
│       ├── article.go
│       └── interaction.go
│
├── deploy/
│   ├── script/
│   │   ├── init.sql                    # 数据库初始化脚本
│   │   └── migration/                  # 后续迁移脚本（可选）
│   └── k8s/                            # kubernetes 部署文件
│       ├── gateway-deployment.yaml
│       ├── rate-rpc-deployment.yaml
│       ├── rate-job-cronjob.yaml
│       ├── ranking-consumer-deployment.yaml
│       └── ...                         # 其他服务
│
├── docker-compose.yml                  # 本地开发全家桶（mysql redis rabbitmq consul prom grafana）
├── go.mod
├── go.sum
├── Makefile                            # 常用命令集合
├── README.md
└── .gitignore
```

这个结构是基于 **领域驱动设计 (DDD) 的 go-zero 微服务项目模板**，非常适合中大型后端系统（如你的简历项目 FreeExchanged）。

它强调**领域隔离、模块化、易扩展**，避免了“上帝类”或紧耦合问题。

结构中融入了云原生最佳实践（如独立 job/consumer、IDL 集中管理），符合 2026 年主流 Go 社区趋势（e.g., go-zero-looklook 或 Uber 的 DDD 示例）。

#### 1. 结构整体解析

这个结构是 **Monorepo 风格**（单一仓库多模块），以 **app/** 为核心（业务域）、**pkg/** 为共享库、**desc/** 为接口契约、**deploy/** 为运维。总共 ~7 个顶级目录，逻辑清晰：

- **desc/**：接口定义层（IDL，如 .api / .proto）。集中管理所有服务接口，便于生成代码 & 文档。
- **app/**：业务层。按领域 (Domain) 分模块，每个 domain 自成一体（包含 rpc/job/mq/model），实现 DDD 的“Bounded Context”。
- **pkg/**：公共层。只放真正跨域的代码（如错误码、工具），防止“公共包膨胀”。
- **deploy/**：运维层。脚本 & K8s 配置，体现“基础设施即代码 (IaC)”。
- 其他：go.mod（依赖）、Makefile（构建命令）、docker-compose.yml（本地环境）。

**关键特点**：

- **DDD 导向**：每个 domain（如 rate、ranking）有私有 model（不暴露），通过 rpc/event 通信。
- **进程隔离**：rpc 是核心服务；job 是定时任务；mq 是异步消费者。可独立部署/扩容。
- **go-zero 集成**：用 goctl 生成代码，内置限流/熔断/监控。
- **规模**：适合 1-10 人团队，项目从 MVP 到生产（用户量 ~10k 日活）。

#### 3. 为什么这样设计？（教学部分）

这个结构不是随意堆砌，而是基于 **DDD + 云原生原则** 设计，解决 Go 项目常见痛点（如代码混乱、扩展难）。

我按层/模块教你理由、好处 & 潜在问题（让你避坑）。

##### 3.1 整体设计哲学

- **DDD (Domain Driven Design) 驱动**：

	核心是“领域模型”，每个 domain（如 rate 是“汇率域”）是一个“Bounded Context”（边界上下文）。

	为什么？因为金融资讯系统（如 FreeExchanged）有多个子域（用户、汇率、排行、互动），如果全塞一个包，会导致“意大利面代码”。隔离后，每个域独立演化（e.g., rate 加新 API 不影响 ranking）。

- **分层架构**：

	外部 (Client) → Gateway (暴露) → Domains (业务) → Storage (持久化) → Tools (支撑)。

	为什么？遵循“依赖倒置原则”（高层不依赖底层），易测试/替换（e.g., 换 MySQL 为 Postgres 只改 model）。

- **进程/组件隔离**：

	rpc 是同步服务；job 是定时（e.g., 汇率拉取）；mq 是异步（e.g., 事件驱动更新排行）。

	为什么？高并发场景下，job/mq 可独立 scale（K8s HPA），避免单进程瓶颈。

- **IDL First**：

	desc/ 集中 proto/api。

	为什么？接口即契约，先定义再生成代码（goctl），减少手写 bug；便于 API 文档（swagger） & 版本控制。

##### 3.2 逐模块设计理由

- desc/
	- 为什么集中？IDL 是“源头真理”，分散易版本冲突。集中后，CI/CD 可自动校验/生成（e.g., proto breaking change 检测）。
	- 好处：团队协作时，FE/BE 只看 desc/ 就知道接口；支持 gRPC/REST 双暴露。
	- 坑：别放业务逻辑，只放纯定义。
- app/：
	- 为什么按 domain 分？避免“service 神包”（所有逻辑塞一个 svc）
	- 每个 domain 自含 rpc/model/etc，便于微服务拆分（未来 rate 独立 repo）
	- 子设计：
		- **cmd/**：入口文件（e.g., rpc.go 是 zRPC server；job.go 是 cron）。为什么？进程级隔离，部署时 kubectl run rate-job 独立。
		- **internal/**：私有实现（logic/server/svc）。为什么？Go 的 internal/ 包可见性，防外部误用。
		- **model/**：私有数据操作（e.g., rate model 只操作汇率表/缓存）。为什么？封装存储细节，换 DB 不改上层；防止跨域污染（ranking 不能直接读 user model）。
		- **mq/**（ranking/interaction）：异步消费者。为什么？事件驱动架构 (EDA)，解耦（如点赞事件 → 更新排行 ZSet），提升系统弹性（RabbitMQ 缓冲高峰）。
	- 好处：模块化测试（e.g., unit test rate logic 独立）；易扩展（加 payment domain 复制结构）。
	- 坑：domain 太多（>10）时，考虑 Polyrepo（多仓库）。
- pkg/：
	- 为什么克制（只 4 个子包）？公共包易成“垃圾桶”，导致循环依赖。xerr/event 是纯定义；utils/interceptor 是无状态工具。
	- 好处：复用而不滥用（e.g., event/rate.go 定义 RateUpdatedEvent struct，所有 mq 用）。
	- 坑：如果 pkg 膨胀，拆成独立模块（Go workspace）。
- deploy/：
	- 为什么 IaC？script/init.sql 自动化 DB；k8s/ yaml 声明式部署。为什么？生产环境一键 rollout，避免手工操作。
	- 好处：本地 docker-compose → 云上 K8s 平滑迁移；HPA 自动扩容（基于 Prometheus metrics）。
	- 坑：初学者配置 yaml 复杂，先用 Makefile 简化。
- 其他文件：
	- **Makefile**：为什么？一键命令（gen-rpc 等），加速开发循环。
	- **docker-compose.yml**：本地全栈（MySQL/Redis/RabbitMQ/Consul/Prom/Grafana）。为什么？开发环境一致性，防“本地跑通线上炸”。
	- **go.mod**：统一依赖，防版本冲突。

##### 3.3 优缺点 & 适用场景

- 优点：
	- **可维护**：代码边界清晰，新人 1 天上手。
	- **可扩展**：加功能只改一个 domain；支持微服务（zRPC + Consul 发现）。
	- **性能**：go-zero 内置限流/熔断；Redis ZSet 高效排行；RabbitMQ 异步。
	- **云原生**：K8s ready，监控完备（Prometheus 刮取 /metrics）。
- 缺点：
	- **学习曲线**：DDD + goctl 初学需 1 周；目录深（app/rate/cmd/job/internal/logic）。
	- Overhead：小项目（<5k 行）过重，用单文件（如 PocketBase）更简单。
- **适用**：你的 FreeExchanged（实时汇率 + 微服务）完美匹配。中型 SaaS、FinTech 项目；不适合纯脚本工具。

##### 3.4 如何应用到你的项目？

1. **起步**：用 Makefile gen-all 生成骨架。
2. **开发流**：改 desc/proto → make gen-rpc → 写 logic/model → 测试 rpc → 集成 gateway。
3. **优化**：加 OpenTelemetry 追踪（interceptor）；用 ArgoCD 管理 K8s。
4. **简历建议**：在项目描述加“采用 DDD 架构，按领域隔离模块，实现 rpc/job/mq 进程解耦，支持事件驱动 & 云原生部署”。这会让 HR 眼前一亮，突出你的深度。

### 3.数据库设计（MySQL + Redis）

**MySQL 主要表**（用 go-zero model 生成）：

```sql
-- users
CREATE TABLE users (
  id BIGINT PRIMARY KEY AUTO_INCREMENT,
  email VARCHAR(128) UNIQUE,
  password VARCHAR(255),
  name VARCHAR(64),
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP
);

-- exchange_rates (历史)
CREATE TABLE exchange_rates (
  id BIGINT PRIMARY KEY AUTO_INCREMENT,
  base_currency CHAR(3),
  target_currency CHAR(3),
  rate DECIMAL(18,8),
  change_percent DECIMAL(10,4),
  updated_at DATETIME
);

-- articles (资讯)
CREATE TABLE articles (
  id BIGINT PRIMARY KEY AUTO_INCREMENT,
  title VARCHAR(255),
  content TEXT,
  author_id BIGINT,
  view_count INT DEFAULT 0,
  created_at DATETIME
);

-- favorites (自选)
CREATE TABLE favorites (
  id BIGINT PRIMARY KEY AUTO_INCREMENT,
  user_id BIGINT,
  currency_pair VARCHAR(10),  -- e.g. USD_CNY
  created_at DATETIME
);

-- interactions (点赞/评论)
CREATE TABLE interactions (
  id BIGINT PRIMARY KEY AUTO_INCREMENT,
  user_id BIGINT,
  target_type ENUM('article','rate'),
  target_id BIGINT,
  action ENUM('like','comment'),
  content TEXT,
  created_at DATETIME
);
```

**Redis**：

- rate:USD:CNY → 当前汇率（hash）
- ranking:change_percent、ranking:volume 等 ZSet（member: "USD_CNY", score: 涨跌幅）
- ws:clients → WebSocket在线用户

### Pkg

#### ①paseto鉴权

1) `pkg/token/payload.go`

```go
package token

import (
	"errors"
	"time"
)

var (
	ErrExpiredToken   = errors.New("token expired")
	ErrNotValidYet    = errors.New("token not valid yet")
	ErrInvalidToken   = errors.New("token invalid")
	ErrInvalidKeySize = errors.New("paseto v2.local key must be 32 bytes (after base64 decode)")
)

// Payload 是你 token 里承载的声明（claims）
// PASETO v2.local 会把它作为 JSON 加密进 token 中（机密性 + 完整性）
type Payload struct {
	ID        string    `json:"jti"`              // token id，用于撤销/踢人
	UserID    int64     `json:"uid"`              // 用户 id
	Issuer    string    `json:"iss,omitempty"`    // 签发方（可选）
	Audience  string    `json:"aud,omitempty"`    // 受众（可选）
	IssuedAt  time.Time `json:"iat"`              // 签发时间
	NotBefore time.Time `json:"nbf,omitempty"`    // 生效时间（可选）
	ExpiredAt time.Time `json:"exp"`              // 过期时间
}

// Valid 做基础合法性校验（过期/未生效）
func (p *Payload) Valid(now time.Time) error {
	if now.Before(p.NotBefore) && !p.NotBefore.IsZero() {
		return ErrNotValidYet
	}
	if now.After(p.ExpiredAt) {
		return ErrExpiredToken
	}
	return nil
}

```

2) `pkg/token/paseto.go`

```go
package token  // (1) 包名 token，建议放 pkg/token，便于跨 domain 复用（如 gateway/user）。

import (
	"crypto/rand"  // (2) 随机数生成，安全 key 用。为什么？os.Read 可能不安全，rand.Read 用 CSPRNG。
	"encoding/base64"  // (3) base64 编码 key，便于 yaml 配置（字符串存储）。
	"errors"  // (4) 标准错误处理，建议用 pkg/xerr 统一业务码。
	"time"  // (5) 时间处理，exp/iat 用 UTC 避免时区问题。
	"github.com/google/uuid"  // (6) UUID 生成 ID，防重放（jti 类似）。
	"github.com/o1egl/paseto"  // (7) PASETO 库，v2.local 是对称加密，简单高效。
)

// Maker 抽象接口：方便你以后换 JWT/PASETO v4 或者做单测  
// (8) 接口设计优秀，依赖倒置（DIP）。为什么？测试时 mock Maker；未来换实现不改代码。
type Maker interface {
	CreateToken(userID int64, duration time.Duration) (string, *Payload, error)
	VerifyToken(token string) (*Payload, error)
}

// PasetoMaker 使用 PASETO v2.local（对称加密）  // (9) 结构体实现接口，v2 是成熟版（v4 更新但兼容差）。
// key 必须是 32 bytes（强制要求）  // (10) key 32 bytes 是 AES-256 要求，短了不安全。
type PasetoMaker struct {
	paseto   *paseto.V2  // (11) PASETO v2 实例，复用避免重复 new。
	key      []byte  // (12) 对称密钥，内存存储，生产用 env 或 config 加载。
	issuer   string  // (13) iss 发行者（e.g., "freeexchanged-api"），可选防跨服务 token。
	audience string  // (14) aud 受众（e.g., "frontend"），可选防滥用。
}

// NewPasetoMakerFromBase64Key 推荐：配置里存 base64 字符串（更容易管理）  // (15) 工厂函数，base64 加载 key，便于 yaml/env。为什么？二进制 key 难配置。
// base64 解码后必须是 32 bytes  // (16) 强制 32 bytes，防配置错误。
func NewPasetoMakerFromBase64Key(keyBase64 string, issuer string, audience string) (*PasetoMaker, error) {
	key, err := base64.StdEncoding.DecodeString(keyBase64)  // (17) 标准 base64 解码。
	if err != nil {
		// 兼容 URL-safe base64（有些人喜欢用这个）  // (18) 兼容 RawURL，贴心设计（e.g., Kubernetes secret 用 URL-safe）。
		key, err = base64.RawURLEncoding.DecodeString(keyBase64)
		if err != nil {
			return nil, errors.New("invalid base64 key")  // (19) 自定义错误，建议用 fmt.Errorf("base64 decode: %w", err) 包装。
		}
	}

	if len(key) != 32 {  // (20) 长度校验，防安全漏洞。
		return nil, ErrInvalidKeySize  // (21) 假设 ErrInvalidKeySize 是自定义 error，常量定义在包里。
	}

	return &PasetoMaker{
		paseto:   paseto.NewV2(),  // (22) NewV2() 是单例安全，复用。
		key:      key,
		issuer:   issuer,
		audience: audience,
	}, nil
}

func (m *PasetoMaker) CreateToken(userID int64, duration time.Duration) (string, *Payload, error) {
	now := time.Now().UTC()  // (23) UTC 时间，防时区问题。

	payload := &Payload{  // (24) Payload 结构体（假设定义在同包），自定义字段。
		ID:        uuid.NewString(),  // (25) UUID ID，jti 防重放。
		UserID:    userID,  // (26) 核心 payload，userID 用于业务。
		Issuer:    m.issuer,  // (27) iss，校验发行者。
		Audience:  m.audience,  // (28) aud，校验受众。
		IssuedAt:  now,  // (29) iat 发行时间。
		NotBefore: now,  // (30) nbf 立即生效，防未来 token。
		ExpiredAt: now.Add(duration),  // (31) exp 过期时间，duration 从 config 取。
	}

	token, err := m.paseto.Encrypt(m.key, payload, nil)  // (32) Encrypt 加密 payload，nil 是 footer（可选）。
	if err != nil {
		return "", nil, err
	}
	return token, payload, nil  // (33) 返回 token + payload，便于调试/日志。
}

func (m *PasetoMaker) VerifyToken(token string) (*Payload, error) {
	var payload Payload  // (34) 解密到 struct。
	err := m.paseto.Decrypt(token, m.key, &payload, nil)  // (35) Decrypt 解密。
	if err != nil {
		return nil, ErrInvalidToken  // (36) 自定义错误。
	}

	if err := payload.Valid(time.Now().UTC());  // (37) 假设 Payload 有 Valid 方法，校验 exp/nbf/iat。
	if err != nil {
		return nil, err
	}

	// 可选：做 iss/aud 校验（你如果填了 issuer/audience）  // (38) 额外校验，防 token 滥用。
	if m.issuer != "" && payload.Issuer != m.issuer {
		return nil, ErrInvalidToken
	}
	if m.audience != "" && payload.Audience != m.audience {
		return nil, ErrInvalidToken
	}

	return &payload, nil
}

// GenerateSymmetricKeyBase64 生成一把安全的 32 bytes key（base64 形式）  // (39) 辅助函数，开发时生成 key 贴到 config。
// 你可以把输出贴到 yaml 的 AccessSecret 里  // (40) 实用，生产用 KMS 生成。
func GenerateSymmetricKeyBase64() (string, error) {
	key := make([]byte, 32)  // (41) 32 bytes AES-256。
	if _, err := rand.Read(key); err != nil {  // (42) 安全随机，防预测。
		return "", err
	}
	return base64.StdEncoding.EncodeToString(key), nil  // (43) base64 编码，易复制。
}

```

架构闭环（推荐）

1. **user.rpc 登录成功签发 PASETO token**
2. **gateway 的 PasetoMiddleware 校验 token**
3. 校验通过：把 `uid` 写入 request context（或 header），再转发到下游服务
4. （可选增强）Redis 做 token 撤销/踢人/单点登录

面试题

下面这些老弟背下来就够用了（我给你“答题要点”）：

1. **JWT vs PASETO 的核心区别？**
	 要点：JWT 易踩算法混淆/兼容坑；PASETO 设计上避免 “alg=none/选择算法” 这类问题；PASETO 强制安全默认值。
2. **PASETO v2.local 和 v2.public 区别？**
	 要点：local=对称加密（机密性+完整性）；public=非对称签名（完整性，无机密性）；local 适合服务端签发/服务端验证，public 适合多方验证。
3. **为什么 token 里要有 jti？**
	 要点：支持撤销、踢人、单点登录、风控追踪、黑名单/白名单。
4. **如何实现“注销/踢人/改密后旧 token 失效”？**
	 要点：Redis 存 jti 白名单；注销删 key；改密可以升级 `tokenVersion`（用户维度版本号）或清空该用户所有 jti。
5. **为什么不要区分“用户不存在”和“密码错误”？**
	 要点：防止账号枚举攻击；统一返回“账号或密码错误”。
6. **bcrypt 为什么适合存密码？**
	 要点：自带盐、抗 GPU 暴力破解，成本可调；不要存明文/MD5。
7. **AccessSecret 如何管理？**
	 要点：不写死仓库；用环境变量/密钥管理；支持轮换；PASETO local key 必须满足长度要求。
8. **网关鉴权 vs 各服务鉴权怎么选？**
	 要点：网关统一鉴权更省；内部服务可二次校验（零信任）；看安全要求与性能。
9. **如何做 Refresh Token？**
	 要点：短 access token + 长 refresh token；refresh 存库/redis；旋转机制防重放。
10. **PASETO token 放哪里更安全？Cookie 还是 LocalStorage？**
	 要点：Web 场景：HttpOnly Cookie 防 XSS；LocalStorage 易被 XSS；Cookie 要配合 CSRF 防护。
11. **如何在 gRPC 里传 token？**
	 要点：用 metadata：`authorization: Bearer ...`；服务端 unary interceptor 校验并放进 context。
12. **Consul 服务发现挂了怎么办？**
	 要点：客户端缓存、重试、降级；注册续约/健康检查；启动顺序与容错。





### user.service

数据模型、服务发现配置和鉴权模块集成。

**开发一个 domain 的标准流程（记住这 6 步，以后所有 domain 都这样开发）**：

1. **定义 IDL**（desc/xxx.proto）—— 接口契约先行
2. **goctl 生成代码** —— 自动生成骨架
3. **实现 ServiceContext** —— 注入依赖（Redis、Paseto、其他 RpcClient）
4. **实现 Logic** —— 核心业务逻辑（这里写登录）
5. **注册 Server** —— 把 logic 挂到 gRPC 方法上
6. **启动 & 测试** —— 本地验证

**为什么这样开发？**

- **IDL First**：先定义接口，后端、前端、文档三方统一，避免后期改动大返工。
- **Logic 层集中业务**：所有判断、调用 model、发事件都在这里，handler 只负责参数校验（go-zero 已自动）。
- **ServiceContext 统一注入**：以后加 Redis、RabbitMQ、其他服务 client，只改这里。
- **model 私有**：User 域的数据库操作不被其他域直接访问，符合 DDD 边界。

**避坑**：

- 不要在 logic 里直接 new redis/gorm，每次都从 svcCtx 拿。
- proto 改了必须重新 make gen-rpc。
- 错误统一用 pkg/xerr（后面我们再补）。

#### 1.user.proto

```go
syntax = "proto3";

package user;
option go_package = "./user";

// 1. 登录
message LoginReq {
  string mobile = 1;
  string password = 2;
}

message LoginResp {
  int64 id = 1;
  string nickname = 2;
  string token = 3; // 既然你希望在 RPC 里做登录，我们直接返回 Token
}

// 2. 注册 (为了能登录，我们顺便写个注册)
message RegisterReq {
  string mobile = 1;
  string password = 2;
  string nickname = 3;
}
message RegisterResp {
  int64 id = 1;
}

service User {
  rpc Login(LoginReq) returns(LoginResp);
  rpc Register(RegisterReq) returns(RegisterResp);
}
```

#### 2.goctl 生成代码 (自动生成骨架)

```
# 生成 RPC 骨架
goctl rpc protoc desc/user.proto --go_out=app/user/cmd/rpc --go-grpc_out=app/user/cmd/rpc --zrpc_out=app/user/cmd/rpc --style gozero
```

数据库初始化（deploy/script/init.sql）并接入真实 MySQL

```
# 1. 先确保 MySQL 容器正在运行（重要！）
docker compose up -d mysql

# 2. 用 Get-Content 把 SQL 文件内容 pipe 给 docker exec（这就是 Windows 版的 < ）
Get-Content deploy\script\init.sql | docker exec -i freeexchanged-mysql-1 mysql -uroot -proot
```

```
# 生成 Model 代码 (连你的 3307 端口)
# 如果还没执行过，先确保 deploy/script/init.sql 已导入数据库
goctl model mysql datasource -url="root:root@tcp(127.0.0.1:3307)/freeexchanged" -table="user" -dir="app/user/model" -style gozero
```

安装 Consul 扩展依赖

```
go get github.com/zeromicro/zero-contrib/zrpc/registry/consul
go mod tidy
```

#### 3.实现 ServiceContext (注入依赖)

这一步我们要把 **MySQL、Redis、Consul 配置** 全部组装起来。

**修改配置文件结构 (`config.go`)** 打开 `app/user/cmd/rpc/internal/config/config.go`。 我们需要把 `Consul` 的配置加进去。

```go
package config

import (
    "github.com/zeromicro/go-zero/zrpc"
    "github.com/zeromicro/zero-contrib/zrpc/registry/consul" // 引入 consul 定义
)

type Config struct {
    zrpc.RpcServerConf
    DataSource string      // MySQL 配置
    Consul     consul.Conf // Consul 配置 (关键！)
    Auth       struct {    // Paseto/JWT 密钥
        AccessSecret string
        AccessExpire int64
    }
}
```

**修改 YAML 配置文件 (`user.yaml`)** 

打开 `app/user/cmd/rpc/etc/user.yaml`，**完全替换**为以下内容（适配你的 docker-compose）：

告诉 RPC 服务“叫什么、监听在哪、怎么连数据库、要不要注册到 Consul、鉴权参数怎么配”。

User 服务不仅需要连接数据库和缓存，还需要配置 Paseto 鉴权密钥。

关键配置项

| 配置项       | 说明                         | 示例值                                           |
| :----------- | :--------------------------- | :----------------------------------------------- |
| `Name`       | 服务名称（用于 Consul 注册） | `user.rpc`                                       |
| `ListenOn`   | 监听地址                     | `0.0.0.0:8080`                                   |
| `Consul`     | 注册中心配置                 | `Host: 127.0.0.1:8500`, `Key: user.rpc`          |
| `DataSource` | MySQL 连接串                 | `root:root@tcp(127.0.0.1:3307)/freeexchanged...` |
| `Auth`       | Paseto 鉴权配置              | `AccessExpire: 86400` (秒)                       |

```go
Name: user.rpc
ListenOn: 0.0.0.0:8080

# --- Consul 配置 ---
Consul:
  Host: 127.0.0.1:8500 # 对应 docker-compose 的 consul 端口
  Key: user.rpc        # 服务注册的 Key
  Tag: [rpc]           # 标签

# --- MySQL 配置 (3307) ---
DataSource: root:root@tcp(127.0.0.1:3307)/freeexchanged?charset=utf8mb4&parseTime=true&loc=Asia%2FShanghai

# --- Auth 配置 ---
Auth:
  AccessSecret: "uSeR_sErViCe_sEcReT_kEy_123456" # 随便写个长的
  AccessExpire: 86400                            # 86400 = 24 小时
```

**注入依赖 (`servicecontext.go`)** 

`ServiceContext` 是 go-zero 中资源初始化的核心位置。

在这里我们将初始化：

1. **MySQL 连接** (`sqlx.SqlConn`)
2. **Redis 连接** (`redis.Redis`)
3. **Paseto Maker** (用于 Token 生成)

**整个服务的“依赖注入容器（DI Container）”**，也就是把你这个 RPC 服务运行时需要用到的东西（配置、数据库连接、model、redis、下游 RPC client、消息队列等）**统一创建一次，然后传给所有 logic 用**。

你现在这段代码就是典型用法：把 **Config、UserModel、RedisClient** 放进一个结构体里。

有了 `ServiceContext`：

- **初始化集中在一个地方**（`NewServiceContext`）
- 只初始化一次，整个服务复用
- logic 里只关心业务，不关心“怎么创建依赖”

打开 `app/user/cmd/rpc/internal/svc/servicecontext.go`。

启动 RPC 服务时通常是这样：

1. 读取 `yaml` → 得到 `config.Config`
2. 用 config 创建 `svc := svc.NewServiceContext(c)`
3. 创建 server 时把 `svc` 传进去
4. 每个 handler/logic 需要什么，就从 `svcCtx` 里拿

```go
package svc

import (
    "freeexchanged/app/user/cmd/rpc/internal/config"
    "freeexchanged/app/user/model"
    "github.com/zeromicro/go-zero/core/stores/redis" // 引入 go-zero 自带 redis
    "github.com/zeromicro/go-zero/core/stores/sqlx"
)

type ServiceContext struct {
    Config    config.Config
    UserModel model.UserModel
    RedisClient *redis.Redis  // 注入 Redis
}

func NewServiceContext(c config.Config) *ServiceContext {
    conn := sqlx.NewMysql(c.DataSource)
    
    // 初始化 Redis (连接你的 6380)
    // 注意：通常 Redis 配置也放在 yaml 里，这里为了演示直接硬编码或复用
    redisConf := redis.RedisConf{
        Host: "127.0.0.1:6380",
        Type: "node",
    }

    return &ServiceContext{
        Config:    c,
        UserModel: model.NewUserModel(conn),
        RedisClient: redis.MustNewRedis(redisConf),
    }
}
```

#### 4.实现 Logic (核心业务逻辑)

这里我们实现 **登录 (Login)**。我们需要验证密码，并生成 Token。

```go
go get golang.org/x/crypto/bcrypt
```

实现logic逻辑

打开 `app/user/cmd/rpc/internal/logic/loginlogic.go`。

```go
package logic

import (
    "context"
    "errors"
    "time"

    "freeexchanged/app/user/cmd/rpc/internal/svc"
    "freeexchanged/app/user/cmd/rpc/user"
    "freeexchanged/app/user/model" // 引入 model

    "github.com/golang-jwt/jwt/v4" // 使用 JWT 生成 Token (Go-Zero 标配)
    "github.com/zeromicro/go-zero/core/logx"
    "golang.org/x/crypto/bcrypt"
)

type LoginLogic struct {
    ctx    context.Context
    svcCtx *svc.ServiceContext
    logx.Logger
}

func NewLoginLogic(ctx context.Context, svcCtx *svc.ServiceContext) *LoginLogic {
    return &LoginLogic{
        ctx:    ctx,
        svcCtx: svcCtx,
        Logger: logx.WithContext(ctx),
    }
}

func (l *LoginLogic) Login(in *user.LoginReq) (*user.LoginResp, error) {
    // 1. 根据手机号查询用户
    userInfo, err := l.svcCtx.UserModel.FindOneByMobile(l.ctx, in.Mobile)
    if err != nil {
        if err == model.ErrNotFound {
            return nil, errors.New("用户不存在")
        }
        return nil, err
    }

    // 2. 验证密码
    err = bcrypt.CompareHashAndPassword([]byte(userInfo.Password), []byte(in.Password))
    if err != nil {
        return nil, errors.New("密码错误")
    }

    // 3. 生成 Token (这里演示 JWT，Paseto 逻辑类似，Go-Zero 内置 JWT 更好用)
    now := time.Now().Unix()
    claims := make(jwt.MapClaims)
    claims["exp"] = now + l.svcCtx.Config.Auth.AccessExpire
    claims["iat"] = now
    claims["userId"] = userInfo.Id // 把 userId 塞进 token
    
    token := jwt.New(jwt.SigningMethodHS256)
    token.Claims = claims
    tokenString, err := token.SignedString([]byte(l.svcCtx.Config.Auth.AccessSecret))
    if err != nil {
        return nil, err
    }

    // 4. (可选) 将 Token 存入 Redis 做白名单或踢人逻辑
    // l.svcCtx.RedisClient.Set(...)

    return &user.LoginResp{
        Id:       userInfo.Id,
        Nickname: userInfo.Nickname,
        Token:    tokenString,
    }, nil
}
```

运行:

```
go run app/user/cmd/rpc/user.go -f app/user/cmd/rpc/etc/user.yaml
```



### Paseto 鉴权闭环

配置 Gateway：Gateway 的 etc/gateway.yaml 也需要加上同样的 Identity 配置（密钥必须和 User 服务一模一样！）。

实现 Middleware：在 app/gateway/internal/middleware 下编写 PasetoMiddleware。 

挂载 Middleware：在 app/gateway/internal/svc/servicecontext.go 里注册这个中间件。

验证闭环：

用 Postman 模拟：先 Login 拿到 Token，再带 Token 调一个受保护的接口（比如 Ping）

##### postman测试

前提：确保两个服务都在运行

- **User RPC**:go run app/user/cmd/rpc/user.go -f app/user/cmd/rpc/etc/user.yaml
- **Gateway**:go run app/gateway/gateway.go -f app/gateway/etc/gateway.yaml

Step 1：注册用户

| 项目            | 内容                                     |
| :-------------- | :--------------------------------------- |
| Method          | `POST`                                   |
| URL             | `http://localhost:8888/v1/user/register` |
| Headers         | `Content-Type: application/json`         |
| Body (raw JSON) | 见下方                                   |

```
{

  "mobile": "13800138000",

  "password": "123456",

  "nickname": "老弟"

}
```

期望返回:

```
{

  "id": 1,

  "token": "v2.local.xxxxxx...",

  "expire_at": 1708300800

}
```

Step 2：登录获取 Token

| 项目            | 内容                                  |
| :-------------- | :------------------------------------ |
| Method          | `POST`                                |
| URL             | `http://localhost:8888/v1/user/login` |
| Headers         | `Content-Type: application/json`      |
| Body (raw JSON) | 见下方                                |

```
{
  "mobile": "13800138000",
  "password": "123456"
}
```

**把返回的** token**字段复制下来**，下一步要用。

Step 3：访问受保护接口（验证鉴权闭环）

| 项目    | 内容                                                         |
| :------ | :----------------------------------------------------------- |
| Method  | `GET`                                                        |
| URL     | `http://localhost:8888/v1/user/info`                         |
| Headers | `Authorization: Bearer <v2.local.oxnwLq0e8oj86uSgMjAZRBg8esBiXAKYwasuk4a3B9u8R14VufnppltTBIOst9c8gQMfdB1y2owVH2TptgcfV6uII_AhVuSI67ceJ0Dac1JoVUEPqUi2HUabADSkOpgtSjvLewNN6woMmLP9DXbjoHFqbr0tTELDpYTWXxKIbKZbJW3H6GgSzBQD01Fo2tAbR_0h6cFIPXhqFkfcejkU7BEwkFnafgaUPSoEUia5nhzEp1V1SK4tvmpBisOsbElzXjP8UiNiwbSAk3nB2YyaBbvIKMql7Vi2XR5j3TWhUt8mqe0QbM6dp6ITSOBINUPF.bnVsbA>` |

期望返回:

```
{
  "id": 1,
  "mobile": "13800138000",
  "nickname": "老弟"
}
```

**如果不带 Token 或 Token 错误**，会返回 401 Unauthorized。

黑名单:

```
POST /v1/user/logout
```

```
URL: http://localhost:8888/v1/user/logout
Method: POST
Headers: Authorization: Bearer <v2.local...> 
Body: 不需要（空的 JSON {} 即可）。
```

目前阶段：✅ 登录签发 Paseto → ✅ 网关校验 → ✅ 黑名单让 token 失效

这已经是鉴权系统的核心了。接下来你要把它变成**真正像一个产品/微服务项目**：有“用户信息接口”、有“缓存”、有“可演示的文档”。

### Interaction-ranking.service

构建一个基于 **RabbitMQ 消息驱动** 和 **Redis ZSet 实时计算** 的高并发互动与排行榜系统。

这一阶段将涵盖点赞、取消点赞、以及热度排行榜的实时更新与查询。

我们将引入两个新的微服务：**Interaction RPC** (互动服务) 和 **Ranking RPC** (排行榜服务)，并集成 **RabbitMQ** 作为削峰填谷的中间件。

#### 结构图

![image-20260219012407328](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20260219012407328.png)

**写路径异步化 + 读路径走缓存排行**”架构：点赞是写（高频、要抗压），排行榜是读（要快、要稳定）。

**这张图整体在解决什么问题？**

- **点赞/取消点赞**：高并发写入，不能每次都去算排行榜（太慢）
- **排行榜 TopN**：要秒出结果，不能每次临时统计（太慢）

所以做法是：

- 写路径：只负责“**记录状态 + 发事件**”
- 读路径：只负责“**从 Redis ZSet 取 TopN**”

参与角色:

1) User

- 发两个请求：
	- `POST /v1/article/like`（点赞/取消）
	- `GET /v1/ranking/top`（看榜）

2) API Gateway

- 统一入口：**鉴权、限流、路由转发**
- 把 **HTTP 请求转换成 RPC 调用**：
	- 调 `Interaction RPC: Like`
	- 调 `Ranking RPC: GetTop`

##### 写路径:点赞

1）Check Duplicate（去重）

“一个用户对同一篇文章只能点赞一次”，否则会重复加分。

常见实现：

- Redis Set / Bitmap（图里写 Redis Bitmap/Set）
	- Set：`SISMEMBER like:article:<aid> <uid>`
	- Bitmap：对大 uid 空间更省内存，但实现更复杂
- 或数据库唯一索引（`(user_id, article_id)` unique）作为最终兜底

> 目的：**幂等**。点赞接口多次点，结果一致，不会刷分。

2）Persist State（落库）

把点赞状态写到 **Interaction DB**：

- 记录：用户 uid、文章 aid、action(1/0)、时间
- 这样你就能支持：
	- 取消点赞
	- 查询用户点赞历史
	- 审计/风控

> Redis 去重快，但 DB 才是最终可信。

3）Publish Event（发事件到 MQ）

**点赞成功后发一条消息到 RabbitMQ**：

- exchange：比如 `article.events`
- routing key：`article.like`
- payload：`{articleId, delta(+1/-1), ts, uid, eventId}`

> 关键点：这里不直接改排行榜，而是 **异步**，这样点赞接口可以很快返回。

**MQ：RabbitMQ Exchange**

它的作用是**解耦**：

- Interaction 只管发消息，不关心谁消费
- Ranking 只管消费，不关心谁生产

未来你还可以加其他消费者：

- 风控服务（异常点赞检测）
- 通知服务（点赞提醒作者）
- 推荐服务（热度特征）

##### 读路径:排行榜

**A) Ranking Consumer（异步消费）**

图里写 “Consume: article.like”。

收到消息后做：

4）ZINCRBY

把文章热度更新到 Redis ZSet：

- key：`hot_articles`
- member：`articleId`
- score：热度分（点赞 +1，取消 -1）

命令：

- `ZINCRBY hot_articles +1 123`
- `ZINCRBY hot_articles -1 123`

> ZSet 是做排行榜的神器：天然按 score 排序。

**B) Ranking RPC（同步读）**

用户请求 `GET /v1/ranking/top` 时：

5）ZREVRANGE

从 ZSet 取 TopN：

- `ZREVRANGE hot_articles 0 N-1 WITHSCORES`

返回文章 id + 热度分，再（可选）批量查文章标题/封面：

- 可以从 Redis Hash 拿（更快）
- 或从 article.rpc 查详情（更准确）

##### 面试回答

1. **写路径异步化**：点赞只落库+发事件，避免同步更新排行榜带来的延迟和耦合。
2. **读路径缓存化**：排行榜直接从 Redis ZSet 读，复杂度低、速度快。
3. **幂等保证正确性**：Redis/DB 去重防止重复点赞导致热度异常。
4. **可扩展**：MQ 一条消息可以被多个服务消费，天然支持新增功能。

##### 细节

1) 消息重复怎么办？

RabbitMQ “至少一次投递”很常见 → consumer 可能重复消费。
 处理方法：

- 最简单：依赖 Interaction 的幂等，确保 delta 只发一次
- 更稳：消息带 `eventId`，Ranking 用 Redis set `seen:eventId` 去重（TTL）

2) 点赞落库成功但发消息失败怎么办？

这叫一致性问题：

- 简化：失败重试 + 记录日志
- 进阶：Outbox 模式（把事件先写 DB，再异步投递）

3) 取消点赞怎么处理？

- Like：delta=+1
- Unlike：delta=-1
- 前提：状态机正确（必须先点赞才能取消）

4) 热度衰减/时间窗口怎么做？

可以做：

- 每日榜：key `hot_articles:20260218`
- 周榜/月榜类似
- 或用定时任务对 score 做衰减

#### 关键技术选型

- RabbitMQ: 用于解耦**点赞动作与排行榜计算**，确保高并发下的写入稳定性。
	- **Exchange**: `interaction.topic` (Topic 模式)
	- **RoutingKey**: `article.like`, `article.unlike`
- **Redis ZSet**: 利用 `Sorted Set` 数据结构实现实时排行榜，Score 为文章热度值。
- **Redis Bitmap/Set**: 用于去重校验（判断用户是否已点赞），避免刷榜。
- **MySQL**: 存储点赞记录的持久化数据（`user_likes` 表），用于数据兜底和归档。

#### 接口设计

##### Interaction RPC (互动服务)

- **Proto 文件**: `desc/interaction.proto`

- Methods :

	- `Like(LikeReq) returns (LikeResp)`: 点赞/取消点赞。

- Message:

	```protobuf
	message LikeReq {
	  int64 userId = 1;
	  int64 articleId = 2;
	  bool cancel = 3; // true=取消点赞, false=点赞
	}
	```

##### Ranking RPC (排行榜服务)

- **Proto 文件**: `desc/ranking.proto`

- Methods:

	- `GetTop(GetTopReq) returns (GetTopResp)`: 获取前 N 名热度文章。

- Message:

	```protobuf
	message GetTopReq {
	  int32 n = 1; // 获取前几名，例如 10
	}
	message ArticleScore {
	  int64 articleId = 1;
	  double score = 2;
	}
	```

##### Gateway API

- **API 文件**: `desc/gateway.api` (新增 Interaction/Ranking 组)
- Endpoint:
	- `POST /v1/article/like`: 调用 Interaction RPC。
	- `GET /v1/ranking/top`: 调用 Ranking RPC。

#### 实施步骤

**Step 1: 基础设施检查**

- 确保 `docker-compose.yml` 中 RabbitMQ 服务正常运行 (端口 5673/15672)。
- 确保 Redis 服务正常运行。

**Step 2: Interaction Service 开发 (生产者)**

1. **定义 Proto**: 创建 `desc/interaction.proto`。
2. **生成代码**: `goctl rpc ...`。
3. RabbitMQ 集成:
	- 在 `ServiceContext` 中初始化 RabbitMQ Channel。
	- 实现消息发送逻辑：`Publish("interaction.topic", "article.like", payload)`。
4. 业务逻辑:
	- 实现 `LikeLogic`。
	- 校验用户是否点赞 -> 写库 -> 发消息。

**Step 3: Ranking Service 开发 (消费者 & 查询)**

1. **定义 Proto**: 创建 `desc/ranking.proto`。
2. **生成代码**: `goctl rpc ...`。
3. Consumer 实现:
	- 创建一个后台 Goroutine 或独立 Process。
	- 监听 `interaction.topic`。
	- 收到消息 -> 解析 ArticleId -> `Redis.ZIncrBy("hot_articles", 1, articleId)`。
4. 查询逻辑:
	- 实现 `GetTopLogic` -> `Redis.ZRevRangeWithScores`。

Step 4: Gateway 集成

1. 修改 `gateway.api` 注册新路由。
2. 在 `gateway.yaml` 中配置新的 RPC 客户端 (InteractionRPC, RankingRPC)。
3. 实现 Gateway Logic 调用后端 RPC。

#### 数据流转

**场景：用户 A (ID: 101) 给文章 B (ID: 202) 点赞**

1. **Gateway**: 收到 `POST /like {article_id: 202}`。解析 Token 得到 `user_id: 101`。
2. Interaction RPC:
	- 查重：Key `user:101:likes` (Set) 是否包含 `202`？ -> 无。
	- 持久化：`INSERT INTO user_likes ...`
	- **MQ Publish**: Exchange=`interaction.topic`, Key=`article.like`, Body=`{uid:101, aid:202, action:1}`。
	- Redis Set: `SADD user:101:likes 202`。
	- 返回成功。
3. Ranking Consumer:
	- 监听到消息 `{aid:202, action:1}`。
	- Redis ZSet: `ZINCRBY hot_articles 1 202`。
4. **Redis 状态**: `hot_articles`: `[{member: "202", score: 1}]`。
5. 用户 B 查询榜单:
	- `GET /top Redis` -> `ZREVRANGE` -> 返回文章 202 排第一。

#### 核心代码实现

##### Interaction Service (生产者)

核心逻辑：接收请求 -> 发送 MQ 消息 -> 返回成功。 

**代码位置**: `app/interaction/cmd/rpc/internal/logic/likelogic.go`

```go
func (l *LikeLogic) Like(in *pb.LikeReq) (*pb.LikeResp, error) {
    // 构造消息体
    msg := LikeMsg{ UserId: in.UserId, ArticleId: in.ArticleId, ... }
    body, _ := json.Marshal(msg)

    // 发送消息到 RabbitMQ (Topic Exchange)
    err := l.svcCtx.MqChannel.Publish(
        "interaction.topic", // Exchange
        "article.like",      // Routing Key
        // ...
        amqp.Publishing{Body: body},
    )
    return &pb.LikeResp{}, nil
}
```

**面试点**: 这里没有直接写数据库，而是发消息，是为了**降低响应延迟** (RT)。

##### Ranking Service (消费者)

核心逻辑：**后台 Goroutine 监听队列 -> 解析消息 -> 更新 Redis ZSet。** 

**代码位置**: `app/ranking/cmd/rpc/internal/mq/consumer.go`

```go
func Start(c config.Config, rds *redis.Redis) {
    // ... 连接 RabbitMQ ...
    msgs, _ := ch.Consume("ranking.queue", ...)

    go func() {
        for d := range msgs {
            // 解析消息
            var msg LikeMsg
            json.Unmarshal(d.Body, &msg)

            // 原子更新 Redis ZSet (ArticleId 分数 +1)
            rds.Zincrby("hot_articles", 1, fmt.Sprint(msg.ArticleId))
        }
    }()
}
```

**面试点**: 使用 `ZINCRBY` 是原子操作，不用担心并发计数错误。

#####  Ranking Service (查询接口)

核心逻辑：查询 Redis ZRevRange。 

**代码位置**: `app/ranking/cmd/rpc/internal/logic/gettoplogic.go`

```go
func (l *GetTopLogic) GetTop(in *pb.GetTopReq) (*pb.GetTopResp, error) {
    // 获取前 N 名 (带分数)
    pairs, _ := l.svcCtx.Redis.ZrevrangeWithScores("hot_articles", 0, int64(in.N-1))
    
    // 组装返回
    var items []*pb.RankItem
    for _, pair := range pairs {
        // ...
    }
    return &pb.GetTopResp{Items: items}, nil
}
```

#### 面试

**面试官**: 你这个点赞功能，如果同一用户疯狂重复点赞怎么办？ **你**:

1. **Gateway 层限流**: go-zero 自带限流中间件。
2. 业务层去重:
	- 方案 A (简单): Interaction Service 在发 MQ 前，先查 Redis Bitmap/Set 判断是否点过。
	- 方案 B (最终一致性): 允许重复发 MQ，但 Consumer 在更新 ZSet 前，先查数据库或 Redis 判断是否已记录。
	- 当前实现：为了演示高性能架构，目前暂时做了全量推送，后续计划加入 Redis Bitmap 去重。

**面试官**: 如果 RabbitMQ 挂了怎么办？ **你**:

1. **降级**: Catch 到 Publish 错误，降级为直接写 DB，或者写入本地磁盘/Log，由 Filebeat 采集恢复。
2. **高可用**: 生产环境会部署 RabbitMQ Cluster (镜像队列)。

**面试官**: 排行榜数据量很大怎么办？ **你**:

1. Redis ZSet 单键支持数百万元素。如果过大，可以按时间分片 (hot_articles_202310)，或者只保留 Top 1000 (定时任务裁剪)。

#### 测试

### prometheus.service

核心知识点（面试必考）：

1. **Pull vs Push**: Prometheus 是**拉取**模式（主动去你的服务端口拉数据），这对服务端压力更小，更适合云原生环境。
2. Metrics 类型:
	- **Counter**: 只赠不减（如：API 请求数，累计错误数）。
	- **Gauge**: 可增可减（如：当前 goroutine 数，内存占用）。
	- **Histogram**: 直方图（如：API 耗时分布，P99 耗时）。
3. **Go-Zero 的黑科技**: 内置了 Prometheus 中间件，只要你会配置 YAML，甚至不用写一行代码。

#### 监控架构和原理

![image-20260219042503581](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20260219042503581.png)

核心组件

- **Exporter**: 你的微服务 (Go-Zero)。哪怕你不写代码，它也能吐出 `/metrics`。
- **Prometheus**: 定期抓取 Exporter 的数据并存储。
- **Grafana**: 展示数据的 Dashboard。

数据流向:

```go
Service (:909x)` <--- Pull --- `Prometheus (:9090)` --- Query ---> `Grafana (:3000)
```

为什么需要监控?

想象一下，你的系统上线了，用户开始使用。突然有人反馈"系统很慢"。

- **没有监控的你**: 打开日志，一行一行翻，不知道从哪里开始。
- **有监控的你**: 打开 Grafana，5秒内定位到 Ranking RPC 的 P99 延迟在 3 分钟前突然从 10ms 飙升到 500ms，同时 Redis 连接池耗尽。

这就是监控的价值：**把不可见的问题变成可见的图表**

监控体系的三大支柱

业界公认的可观测性 (Observability) 由三部分组成：

| 支柱                   | 工具                 | 回答的问题                                 |
| :--------------------- | :------------------- | :----------------------------------------- |
| **Metrics (指标)**     | Prometheus + Grafana | 系统现在**健不健康**？QPS 多少？延迟多少？ |
| **Logging (日志)**     | ELK / Loki           | 某个请求**发生了什么**？报错信息是什么？   |
| **Tracing (链路追踪)** | Jaeger / Zipkin      | 一个请求**经过了哪些服务**？慢在哪里？     |

我们 Phase 3 专注于 **Metrics**，这是最基础也是最重要的一环。





#### 服务端配置

我们需要告知每个服务开启 Prometheus 端口。

Gateway (`app/gateway/etc/gateway.yaml`)

```yaml
Prometheus:
  Host: 0.0.0.0
  Port: 9091
  Path: /metrics
```

User Service (`app/user/cmd/rpc/etc/user.yaml`)

```yaml
Prometheus:
  Host: 0.0.0.0
  Port: 9092
  Path: /metrics
```

Interaction Service (`app/interaction/cmd/rpc/etc/interaction.yaml`)

```yaml
Prometheus:
  Host: 0.0.0.0
  Port: 9093
  Path: /metrics
```

Ranking Service (`app/ranking/cmd/rpc/etc/ranking.yaml`)

```yaml
Prometheus:
  Host: 0.0.0.0
  Port: 9094
  Path: /metrics
```

**注意**: 修改配置后，必须**重启所有 Go 服务**才能生效！

#### p8s配置

创建或修改此文件，告诉 Prometheus 去哪里抓数据。 由于 Prometheus 运行在 Docker 容器中，我们要用 `host.docker.internal` 访问宿主机上的服务。

```
global:
  scrape_interval: 10s

scrape_configs:
  - job_name: 'gateway'
    static_configs:
      - targets: ['host.docker.internal:9091']

  - job_name: 'user-rpc'
    static_configs:
      - targets: ['host.docker.internal:9092']

  - job_name: 'interaction-rpc'
    static_configs:
      - targets: ['host.docker.internal:9093']

  - job_name: 'ranking-rpc'
    static_configs:
      - targets: ['host.docker.internal:9094']
```

#### 指标

**Prometheus 里已有的关键指标**：

- ```
	http_server_requests_duration_ms_count
	```

	→ Gateway 的 HTTP 请求数

- ```
	rpc_server_requests_duration_ms_count
	```

	→ RPC 服务的请求数

- ```
	rpc_client_requests_duration_ms_count
	```

	→ RPC 客户端调用数

- ```
	redis_client_requests_duration_ms_count
	```

	→ Redis 操作数

QPS:吞吐
![image-20260219043251248](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20260219043251248.png)

P99:延迟

Error Rate:错误率





### go-stress-test 压测

- 压测：使用 go-stress-test 对/v1/ranking/top进行压测，看看 Prometheus 里的 QPS 能飙到多少？
- **限流降级**：在 Gateway 配置限流（Sentinel 或 go-zero自带），并在 Grafana 上观察被拒请求数。
- **链路追踪 (Tracing)**：接入 Jaeger，看看一个请求到底在哪个环节慢了。

#### 工具介绍

- **go-stress-testing** (`link1st/go-stress-testing`) 是一个 Go 语言实现的压测工具。
- 特性:
	- **原生并发**: 充分利用 Goroutine，单机可模拟数万并发。
	- **无依赖**: 编译后就是一个二进制文件，不需要安装 Java (JMeter) 或 Python (Locust)。
	- **可视化报告**: 自动生成 HTML/Markdown 报告，包含详细的耗时分布图。
	- **HTTP/2 支持**: 能够测试现代 Web 服务。

#### 基础用法

##### 简单压测

测试排行榜接口:

```go
./stress.exe -c 50 -n 1000 -u "http://localhost:8888/v1/ranking/top?n=10"
```

- `-c 50`: 并发数 (Concurrency)，即同时有 50 个用户在请求。
- `-n 1000`: 请求总数 (Total Requests)，**注意：这里是单并发的请求数**。总请求数 = 50 * 1000 = 50,000。
- `-u`: 目标 URL。

##### 复杂压测 (POST + Body + Header)

测试点赞接口 (需要 Token)。

```go
./stress.exe -c 20 -n 100 -u "http://localhost:8888/v1/article/like" -data '{"article_id": 9999, "action": 1}' -H "Content-Type: application/json" -H "Authorization: Bearer <YOUR_TOKEN>"
```

#### 实战演练：压测 Ranking 服务 (读性能)

Ranking 服务是 **IO 密集型 (Redis)**，主要瓶颈在于 Redis 带宽和连接池。

### Jaeger

### exchanged.service

1. 重写 desc/rate.proto
2. 用 goctl重新生成代码
3. 修改 YAML（换成 Consul，加 Redis）
4. 实现 Job（定时拉取汇率）
5. 实现 RPC Logic（从 Redis 读取）
6. 接入 Gateway

### article.service

**Step 1: 基础设施搭建**

1. 数据库建表 (articles)

2. goctl 生成 Model 代码

3. 更新 Proto (

	article.proto

	) + goctl 生成 RPC 代码

4. 更新 Article RPC 配置 (YAML)

**Step 2: Article 核心实现**

1. Config/ServiceContext 更新（引入 MySQL + MGProducer）
2. 封装mq包(NewProducer+PublishArticleEven)
3. 实现PublishLogic（存库 + 发消息）和其他 Logic

**Step 3: Ranking 消费端实现**

1. 实现mq.ArticleConsumer（监听 Queue + 更新 Redis）
2. 在 Rankingmain.go启动 Consumer

**Step 4: Gateway 集成**

1. 更新gateway.api(新增/v1/article/publish) + goctl 生成
2. 实现 Gateway Logic 并注入 ArticleRpc client

## 整体架构

### 服务端接口规划

| 服务名称 (Service)  | 类型     | 端口 (Port) | Metrics 端口 | 描述                                    |
| :------------------ | :------- | :---------- | :----------- | :-------------------------------------- |
| **Gateway**         | HTTP API | **8888**    | 9091         | 统一网关，鉴权，路由分发，WebSocket     |
| **User RPC**        | gRPC     | **8080**    | 9090         | 用户注册、登录 (PASETO)、信息管理       |
| **Interaction RPC** | gRPC     | **8081**    | 9092         | 点赞、收藏、阅读数 (Redis计数)          |
| **Ranking RPC**     | gRPC     | **8082**    | 9093         | 排行榜服务 (Redis ZSet)                 |
| **Rate RPC**        | gRPC     | **8083**    | 9094         | 汇率查询服务 (Cache-Aside)              |
| **Article RPC**     | gRPC     | **8084**    | 9096         | 文章发布、列表 (RabbitMQ 异步解耦)      |
| **Watchlist RPC**   | gRPC     | **8085**    | 9097         | 自选服务 (Write-Through, Orchestration) |
| **Rate Job**        | Process  | -           | -            | 定时任务：每 5min 拉取汇率并写入 DB     |

###  数据库和中间件依赖

| 组件              | 端口        | 用途                          | 依赖该组件的服务                                    |
| :---------------- | :---------- | :---------------------------- | :-------------------------------------------------- |
| **MySQL**         | 3307        | 核心业务数据存储              | User, Interaction, Article, Rate, Watchlist         |
| **Redis**         | 6380        | 缓存、计数器、排行榜、Pub/Sub | All Services (特别是 Ranking, Interaction, Gateway) |
| **RabbitMQ**      | 5672        | 异步消息队列 (削峰填谷)       | Article (生产), Article (消费)                      |
| **Etcd** / Consul | 2379 / 8500 | 服务注册与发现                | All RPC Services + Gateway                          |
| **Prometheus**    | 9090        | 监控指标采集                  | All Services (暴露 /metrics)                        |
| **Jaeger**        | 16686       | 分布式链路追踪                | All Services (OpenTelemetry)                        |

### 服务调用拓扑

![image-20260219191922550](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20260219191922550.png)

我做的是 go-zero 微服务项目 FreeExchanged。客户端通过 API Gateway 统一入口，HTTP/WS 接入，网关负责 Paseto 鉴权、限流、观测。内部服务用 gRPC 拆分为用户、文章、自选、互动、排行、汇率等。
 高并发写（点赞）走 Interaction 服务，采用 Redis 去重 + MySQL 落库 + RabbitMQ 事件驱动，Ranking 异步消费更新 Redis ZSet，实现读写分离。排行查询全走 Redis，压测下网关 QPS ~200、P99 ~5ms，并用 Prometheus/Grafana + Jaeger 做可观测与定位。

**讲一次完整链路**

比如点赞：网关从 token 解析 uid，调用 Interaction RPC；Interaction 先在 Redis 做幂等校验，然后落库，再发布 MQ 事件。Ranking 消费事件后用 ZINCRBY 更新 ZSet。用户查排行榜时，Ranking RPC 直接 ZREVRANGE 返回 TopN。这样写链路削峰、读链路极快，系统稳定性更好。

**你怎么排障**

> 指标层用 Grafana 看 QPS/P99/错误率；
>
> 如果 P99 上升，用 Jaeger trace 拆分到网关、RPC、Redis、MQ 具体 span，快速定位慢点。

#### 入口层：Client/Web → API Gateway:8888（HTTP/WS）

- **统一入口**：HTTP 请求 + WebSocket 都走网关
- 网关负责：**鉴权（Paseto）、路由、限流、聚合、观测（Prometheus/Tracing）**
- 网关通过 **gRPC** 调后端微服务（User/Article/Watchlist/Interaction/Ranking/Rate）

> 这很好：网关“薄”，业务逻辑尽量在服务里，网关做横切能力。

#### 核心服务层（gRPC 微服务）

- **User RPC :8080**：用户登录/注册、token 签发、黑名单/白名单（你已经做得很硬）
- **Article RPC :8084**：文章/资讯相关（**读多写少**）
- **Watchlist RPC :8085**：自选/收藏（用户维度的数据）
- **Interaction RPC :8081**：互动（点赞/取消/阅读等，**高并发写**）
- **Ranking RPC :8082**：排行榜（高并发读）
- **Rate RPC :8083**：汇率（通常**外部拉取 + 缓存**）

> 这样的拆分是合理的：互动/排行独立出来，能支撑高并发与异步化。

#### 基础设施层：RabbitMQ / MySQL / Redis

**RabbitMQ（Async MQ）**：用来“削峰+解耦”，典型是 Interaction 发事件（article.like），Ranking 消费后更新 ZSet

**MySQL（Persist）**：存最终一致数据（互动记录、自选、文章等）

**Redis**：

- Ranking：**ZSet**（ZADD/ZINCRBY/ZREVRANGE）做热榜 TopN
- Interaction：**Set/Bitmap** 去重（一个用户对同一文章只能点赞一次）
- 你项目里还可用于：token 黑名单、热点缓存等

#### 链路

**A. 读链路（快）：/v1/ranking/top**

Client → Gateway → Ranking RPC → Redis ZSet（ZREVRANGE）→ 返回 TopN
 特点：

- **全程不打 MySQL**，P99 很低（你 Grafana 上已经看到 ~5ms）
- 适合高频读

**B. 写链路（抗压）：/v1/article/like**

Client → Gateway（鉴权拿 uid）→ Interaction RPC
 Interaction 做：

1. Redis 去重（Set/Bitmap）
2. MySQL 落库（最终一致）
3. MQ publish 事件（削峰、异步）
	 Ranking Consumer：
4. 消费消息 → Redis ZSet `ZINCRBY hot_articles 1 articleId`
	 特点：

- 写路径**不阻塞**排行计算
- MQ 消峰 + 异步更新，系统更稳

### 核心功能与技术亮点

| 模块            | 核心功能     | 技术亮点 (面试关键词)                                        |
| :-------------- | :----------- | :----------------------------------------------------------- |
| **User**        | 登录认证     | **PASETO** (比 JWT 更安全), PBKDF2 密码加密                  |
| **Interaction** | 点赞/收藏    | **Redis原子计数**, **异步落库** (防止 DB 压力过大)           |
| **Ranking**     | 热门文章排行 | **Redis ZSet**, **Pipeline** 批量写入, 定时刷新策略          |
| **Article**     | 文章发布     | **RabbitMQ** 消息队列解耦, 流量削峰                          |
| **Rate**        | 汇率查询     | **Cache-Aside** 模式, **Cron Job** 定时同步, 熔断降级        |
| **Watchlist**   | 自选行情     | **Write-Through** 缓存一致性, **Service Orchestration** (RPC Fan-out), **Gateway 聚合** |
| **Gateway**     | 统一接入     | **WebSocket** 实时推送, **JWT/PASETO 统一鉴权**, 全局限流 (TokenLimit) |

###  项目启动顺序

为了避免依赖报错，建议按以下顺序启动：

1. 基础设施 (Infrastructure):
	- MySQL, Redis, RabbitMQ, Etcd/Consul, Jaeger, Prometheus
2. 被依赖的基础服务 (Base Services):
	- User RPC
	- Rate RPC (被 Watchlist 依赖)
3. 核心业务服务 (Core Services):
	- Interaction RPC
	- Article RPC
	- Ranking RPC
	- Watchlist RPC
4. 接口网关 (Gateway):
	- Gateway (最后启动，等待 RPC 就绪)
5. 后台任务 (Background Jobs):
	- Rate Job (独立运行)

## 学习路径

面试不是考你“记住所有代码”，而是考你能不能把项目讲成**一套清晰的系统**：

**目标是什么、架构怎么跑、关键链路怎么保证稳定、出了问题怎么定位。**

### 项目六件套

面试时你必须能讲清楚这 6 件事（也是你学习的目录）：

1. **业务闭环**：系统做什么、核心用例是什么
2. **架构拓扑**：Gateway + 6 微服务 + MySQL/Redis/MQ + Consul
3. **两条关键链路**：读（ranking/top）和写（article/like）
4. **一致性与幂等**：为什么用 MQ、怎么防重复、失败怎么补偿
5. **可观测性**：Prometheus/Grafana 指标 + Jaeger trace 怎么定位慢点
6. **性能与稳定性**：QPS/P99/错误率、限流、降级、熔断策略

### 第一阶段：系统能跑，能复述整个系统

目标你能在 2 分钟内口述“系统结构”和“请求怎么走”。

首先这个项目是我从原来的gin单体项目使用gozero进行重构的微服务项目FreeExchanged。

客户端通过 API Gateway 统一入口，网关负责 **Paseto 鉴权**、**限流**、**路由和观测**。

内部服务使用 gRPC 拆分为**用户、文章、自选、互动、排行、汇率**。

点赞写链路通过 Interaction 服务完成幂等校验、落库并发布 RabbitMQ 事件，Ranking 异步消费更新 Redis ZSet 热榜；

排行榜查询直接从 Redis ZSet 读取，延迟低。

项目接入 Prometheus/Grafana 监控 QPS/P99/错误率，并用 Jaeger 做链路追踪定位性能瓶颈。

#### **链路一：Ranking/Top (Redis ZSet 高性能读)**

**核心逻辑**：前端请求热榜 -> Gateway -> Ranking RPC -> Redis (ZREVRANGE) -> Article RPC (获取详情) -> 返回。

**面试考点**：Redis ZSet 数据结构、Cache Miss 击穿、Pipeline 优化。

##### **Step 1: 入口 (Gateway)**

- **看点**：**Gateway 是如何调用 Ranking RPC 的？是否只拿到了 ID？需不需要再调 Article RPC 补全信息？**

- 观察：

	注意 l.svcCtx.RankingRpc.GetTop 调用的参数和返回值。

	这里体现了BFF (Backend for Frontend)的聚合思想。

gettoplogic：

```go
// Code scaffolded by goctl. Safe to edit.
// goctl 1.9.2

package ranking

import (
	"context"

	"freeexchanged/app/gateway/internal/svc"
	"freeexchanged/app/gateway/internal/types"
	rankingclient "freeexchanged/app/ranking/cmd/rpc/rankingclient"

	"github.com/zeromicro/go-zero/core/logx"
)

type GetTopLogic struct {
	logx.Logger
	ctx    context.Context
	svcCtx *svc.ServiceContext
}

// 获取热榜
func NewGetTopLogic(ctx context.Context, svcCtx *svc.ServiceContext) *GetTopLogic {
	return &GetTopLogic{
		Logger: logx.WithContext(ctx),
		ctx:    ctx,
		svcCtx: svcCtx,
	}
}

func (l *GetTopLogic) GetTop(req *types.GetTopReq) (resp *types.GetTopResp, err error) {
	// 直接使用 svcCtx.RankingRpc (已经是 Ranking 接口，无需重新 NewRanking)
	rpcResp, err := l.svcCtx.RankingRpc.GetTop(l.ctx, &rankingclient.GetTopReq{
		N: req.N,
	})
	if err != nil {
		l.Logger.Errorf("Call Ranking RPC failed: %v", err)
		return nil, err
	}

	var items []types.RankItem
	if rpcResp.Items != nil {
		items = make([]types.RankItem, len(rpcResp.Items))
		for i, v := range rpcResp.Items {
			items[i] = types.RankItem{
				ArticleId: v.ArticleId,
				Score:     v.Score,
				Title:     v.Title,
			}
		}
	}

	return &types.GetTopResp{
		Items: items,
	}, nil
}
```

1.**Gateway 是如何调用 Ranking RPC 的？**

在 Go-Zero 框架中，网关层不应该关心底层的网络寻址、连接池或重试机制，一切都通过**依赖注入 (Dependency Injection)** 来完成。

```go
rpcResp, err := l.svcCtx.RankingRpc.GetTop(l.ctx, &rankingclient.GetTopReq{
    N: req.N,
})
```

**上下文透传 (`l.ctx`)**：它把 HTTP 请求的 Context 传递给了 RPC。

这非常关键，如果你的系统里配置了链路追踪 (Tracing) 或者超时控制，这个 `ctx` 就是把整个调用链串起来的“风筝线”。

**服务上下文 (`l.svcCtx`)**：`svcCtx` 是所有底层资源的超级大管家。

Gateway 并不是在这里临时 `Dial` 建立 gRPC 连接，而是**直接从上下文中取出了预先初始化好的 `RankingRpc` 客户端实例，并调用了它的 `GetTop` 方法。**

**2.是否只拿到了 ID？**

要回答这个问题，我们需要观察 Gateway 在拿到 `rpcResp` 后做了什么样的数据映射（Mapping）。

```go
for i, v := range rpcResp.Items {
    items[i] = types.RankItem{
        ArticleId: v.ArticleId,
        Score:     v.Score,
        Title:     v.Title, // 👈 破案的关键在这里
    }
}
```

很显然，Gateway 从 Ranking RPC 拿到的**不仅仅是 ID**。

除了排序依据的 `Score` 和文章的唯一标识 `ArticleId` 之外，它直接拿到了用于页面展示的业务数据——**`Title` (文章标题)**。

**3.需不需要再调 Article RPC 补全信息？**

**就这段 Gateway 的代码而言：不需要。** 

因为前端展示热榜通常只需要“排名、标题、热度分数”，而这个 `rpcResp` 已经把 `Title` 送到你手里了，Gateway 的组装任务已经闭环，直接将其封装进 `GetTopResp` 返回给前端即可。

**四步剥洋葱读代码法**

第一层：看“门牌和说明书”（元信息与依赖）

拿到文件，先不看具体逻辑，先扫一眼头部：

```go
// Code scaffolded by goctl. Safe to edit.
// goctl 1.9.2
package ranking

import (
    "freeexchanged/app/gateway/internal/types"
    rankingclient "freeexchanged/app/ranking/cmd/rpc/rankingclient"
    // ...
)
```

**注释阅读法**：`Safe to edit` 告诉你，虽然这是生成的代码，但它是留给你写业务逻辑的地方，下次生成不会覆盖它。

**依赖判位法**：看到 `types`（HTTP 接口的结构体）和 `rankingclient`（RPC 的客户端），你立刻就能在脑海中画出坐标：

**当前位置是网关层**。它的左手牵着前端（HTTP），右手牵着后端的微服务（Ranking RPC）。

第二层：看“操作台和工具箱”（结构体定义）

不要直接看函数体，先看这个类（Go里的结构体）拥有什么“资产”：

```go
type GetTopLogic struct {
    logx.Logger
    ctx    context.Context
    svcCtx *svc.ServiceContext
}
```

这是 Go-Zero Logic 层的**“黄金三叉戟”**，你读懂了它，就读懂了上下文：

1. **`logx.Logger`（内嵌）**：Go 的语法糖。内嵌后，你可以直接用 `l.Errorf()` 打印日志，而且它自带了 Trace ID（链路追踪ID）。
2. **`ctx`**：贯穿整个 HTTP 请求的生命周期。**超时控制、主动取消、传递用户 Token 解析出来的 ID，全靠它**。
3. **`svcCtx`（超级大管家）**：它装着你所有的外部依赖（数据库连接池、Redis 实例、各种 RPC 客户端）。你要调谁，都在这里面找。

第三层：看“流水线作业”（核心逻辑拆解）

现在进入真正的 `GetTop` 函数。工业级的代码，通常严格遵循 **Input -> Process -> Transform -> Output** 的流水线模式。

我们按这四步切分：

**1. Input 组装（将 HTTP 请求包装成 RPC 请求）**

```go
// 准备发给后端的快递单
&rankingclient.GetTopReq{
    N: req.N,
}
```

