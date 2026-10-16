# Vibe Engineering Challenge

## Challenge 001 — TaskForge: Multi-Tenant Async Job Processing Platform

## Introduction

本次挑战要求你借助 AI Coding Agent，开发 TaskForge：一个多租户批量数据处理平台。用户上传 CSV 文件后，系统创建后台任务、逐条处理记录，并在 Web 页面展示执行进度与结果。建议实际开发时间为 6～8 小时。

本题面向大四本科生和即将毕业的计算机相关专业硕士研究生，考察 AI 辅助下的全栈工程交付能力。你可以使用任意 AI 工具和模型，甚至让 AI 生成全部代码，但必须理解、验证并能够解释最终交付的项目。详见 [AI 使用规则](#ai-usage-policy)。

> We evaluate engineering outcomes, not typing speed.

题目分为**必做项**和**加分项**。先完成可运行的最小可用产品（MVP），再根据时间选择进阶功能。评审优先考察业务正确性、运行稳定性和你对 AI 生成代码的验证能力。

**阅读导航：** [功能要求](#functional-requirements) · [技术要求](#technical-requirements) · [验收标准](#acceptance-criteria) · [交付内容](#deliverables) · [评分规则](#evaluation-criteria) · [提交方式](#submission)

## Challenge at a glance

| 项目 | 说明 |
|------|------|
| 挑战名称 | TaskForge |
| 项目类型 | Multi-Tenant Async Job Processing Platform |
| 目标岗位 | Full-Stack Software Engineer（校招） |
| 建议开发时间 | 6～8 小时（8 小时为建议上限） |
| 提交窗口 | 收到题目后 7 天 |
| 开发模式 | AI-Assisted Development |
| 技术栈 | 前端 React + TypeScript；后端语言、框架不限；数据库 PostgreSQL 或 SQLite |
| 需求分级 | 必做 + 加分项 |
| 部署要求 | 本地运行 |

## 候选人范围与公平性

本题面向具备基本 Web 开发知识、希望应聘全栈工程师岗位的应届毕业生，不要求正式工作经验。本科生与研究生按照**相同的岗位能力标准**评估。

- **前端固定使用 React + TypeScript；后端语言和框架不作限制。** 请在项目 README 中说明实际技术栈。
- 可以从零开发，也可以使用 Vite、NestJS CLI 等脚手架或公开的基础项目模板；使用模板时需注明来源。允许使用符合许可证要求的开源库。
- 不强制使用 Docker、云部署、Redis 或 Kubernetes，也不要求购买任何付费服务。
- 不要求提交完整的 AI 对话记录、Token 消耗或屏幕录像。
- 可以提交未完成版本；请如实说明完成情况和已知问题。

## Background

一家 SaaS 公司为多个企业客户（租户）提供批量数据处理服务。

企业用户在 Web 页面上传 CSV 文件后，系统把每一行转换为一条待处理记录，并创建一个后台异步任务（Job）。Background Worker 逐条处理这些记录。用户在 Web 页面查看 Job 的状态、执行进度、处理结果和失败原因。

任何用户都不能看到或操作其他租户的数据。

## Project objective

你需要交付一个可以实际运行、测试和验证的 MVP，包括：

- React + TypeScript Web Dashboard
- 后端 API 代码
- 后台 Worker，在创建请求返回后处理 Job
- 持久化存储（PostgreSQL 或 SQLite）
- 覆盖核心逻辑的自动化测试
- 说明设计和 AI 使用过程的文档

评审重点是后端业务处理是否正确：记录处理规则、Job 状态计算和租户隔离。

## Functional requirements

每个小节标题后标有级别：

| 标记 | 含义 |
|------|------|
| （必做） | 必须实现。缺少任何一项都会在 Core Functionality 维度扣分 |
| （加分项） | 可选。实现时必须符合本节的定义，符合定义才计分 |

### Module A: Authentication & Multi-Tenancy

#### A1 测试租户与账号（必做）

系统初始化后（例如运行 seed 脚本）必须至少存在两个租户，每个租户至少一个用户：

| 租户 | 用户 Email |
|------|------------|
| Tenant A | `user-a@test.com` |
| Tenant B | `user-b@test.com` |

- 密码由你设定，并在你的项目 README 中写明密码和登录方式。
- 测试账号只用于本地开发环境，不得使用任何真实账号或真实密码。
- 数据库中不得以明文存储密码。

#### A2 Authentication（必做）

必须支持：

1. 用户使用 Email 和密码登录。
2. 用户退出登录。退出后，当前浏览器不再保留可供应用自动使用的登录状态，受保护页面返回登录页。使用 Token 的实现需要清除客户端保存的 Token；使用 Cookie 的实现需要使浏览器中的认证 Cookie 失效；使用服务端 Session 的实现还需要在服务端使 Session 失效。
3. 维持登录状态：刷新页面后仍保持登录，直到用户退出或凭证过期。
4. 服务端验证每个受保护 API 请求的身份；未携带有效凭证的请求返回 `401`。

认证方案可以是 JWT、Session 或其他合理方案。**不要求对已签发的无状态 JWT 实现服务端撤销或黑名单。** 退出登录的最低验收标准是：从同一浏览器退出后，应用不再附带旧凭证访问受保护资源；不携带有效凭证调用受保护 API 时返回 `401`。如手动保留并重放尚未过期的 JWT，系统可以在其有效期内接受它。

不要求实现 OAuth、SSO、用户注册、邮件验证或角色权限（RBAC）。

#### A3 Authorization 与租户隔离（必做）

后端根据登录用户确定租户，并在每次读取或修改 Job 时按这个租户过滤数据。

- 用户只能读取、查看和操作自己租户的 Job 和记录结果。
- 租户身份必须来自服务端验证过的登录信息，例如 Session 或 JWT 中的用户。不得从请求体、Query 参数、自定义 Header 或前端状态中读取租户身份。
- 修改 URL 中的 Job ID，或在请求中加入 `tenant_id`、`tenantId` 等参数，都不能访问或操作其他租户的数据。
- 访问其他租户的资源时，返回 `404`（推荐，不暴露资源是否存在）或 `403`，响应中不得包含该资源的任何数据。
- 列表类接口只返回当前用户所属租户的数据。
- 绕过前端直接调用 API 时，后端也必须拒绝跨租户访问。

### Module B: Job Creation

#### B1 创建流程（必做）

用户在 Web 页面完成以下操作：

1. 输入 Job 名称。名称必填，去除首尾空白后不能为空，长度上限由你设定并写入文档。
2. 上传一个 CSV 文件。
3. 提交创建请求。
4. 获得新 Job 的 ID。
5. 进入该 Job 的状态页面。

#### B2 CSV 格式（必做）

```csv
id,text
1,Hello World
2,  Vibe Coding is amazing
3,This is a test
4,[FAIL] Simulated error
5,AI Engineering
```

- 第一行必须是表头，且必须包含 `id` 和 `text` 两列。是否允许额外的列由你决定，并在 README 中说明。
- 按标准 CSV 规则解析，支持双引号包裹的字段，字段内可以包含逗号。解析时不得自动去除 `text` 的首尾空白，空白只按 B3 的规则处理。
- 每个数据行是一条独立的处理记录。
- `id` 是业务输入字段，原样保存并在结果中展示，不能直接作为数据库的全局主键。不同 Job 或不同租户中出现相同的 `id` 是正常情况。不需要校验 `id` 的唯一性或格式。

#### B3 记录处理规则（必做）

每条记录按以下顺序判断，命中第一条规则即确定结果：

| 顺序 | Condition | Expected Behavior |
|------|-----------|-------------------|
| 1 | `text` 为空或仅包含空白 | FAILED |
| 2 | `text` 去除首尾空白后长度超过 200 个字符 | FAILED |
| 3 | `text` 包含 `[FAIL]`（区分大小写） | FAILED |
| 4 | 其他内容 | 成功，输出 Trim + Uppercase 后的文本 |

Trim 指去除首尾空白，空白范围以 JavaScript 的 `String.prototype.trim()` 为准。长度按 Trim 后的 Unicode **码点（code point）**计算，不按字节或 UTF-16 编码单元计算：恰好 200 个码点的文本不因长度失败；201 个码点的文本因长度超限而失败。不同后端语言需实现与这一语义等价的行为；例如普通空格、制表符、换行符及不换行空格（U+00A0）均应按该规则处理。

后端语言不限，但处理行为应满足上述规则。成功记录需要输出大写文本。不同语言或运行时的非 ASCII 字母大写映射可能存在差异，因此**跨语言自动验收主要使用 ASCII 英文字母检查大小写转换**；Unicode 测试侧重验证码点长度、空白处理和字符保留，不以语言特有的大写映射作为失败依据。

示例：

| Input | Output |
|-------|--------|
| `"  hello world  "` | `"HELLO WORLD"` |
| `"[FAIL] Simulated error"` | FAILED，失败原因说明命中 `[FAIL]` 规则 |

- 每条失败记录必须保存可读的失败原因，不同规则的失败原因必须能区分。
- 每条记录必须有可观察的模拟处理耗时，建议 200 到 500 ms，固定或随机均可。
- 不要调用真实的 AI API。
- 不得在创建时直接写入最终结果，也不得用一个 `setTimeout` 把整个 Job 标记为完成。后台必须逐条处理记录：处理过程中，Job 的进度和记录状态必须逐步变化，并且能通过 API 观察到。

#### B4 输入校验（必做）

以下情况必须在创建阶段拒绝，返回 `4xx` 和明确的错误信息，并且不创建 Job：

| 情况 | 期望 |
|------|------|
| 未上传文件或文件为空 | 返回 `400`，说明缺少文件或文件为空 |
| CSV 无法解析 | 返回 `400`，说明格式错误 |
| 缺少 `id` 或 `text` 表头 | 返回 `400`，说明缺少哪一列 |
| 只有表头、没有数据行 | 返回 `400`，说明没有可处理的记录 |
| Job 名称缺失，去除首尾空白后为空，或超过你设定的长度上限 | 返回 `400` |
| 文件超过你设定的大小或行数上限 | 返回 `400` 或 `413`，上限需在 README 中说明 |

单条记录内容不合法（例如 `text` 为空）不算创建失败。这条记录照常进入 Job，在处理阶段被标记为 FAILED。

错误响应格式由你设计，但必须统一，前端必须把错误展示给用户。

### Module C: Async Job Processing

#### C1 Job 状态（必做）

```text
QUEUED
RUNNING
COMPLETED
PARTIAL_FAILED
FAILED
```

| 状态 | 含义 |
|------|------|
| `QUEUED` | Job 已创建（或已提交重试，见 E1），等待后台处理 |
| `RUNNING` | Worker 正在处理该 Job |
| `COMPLETED` | 所有记录均处理成功 |
| `PARTIAL_FAILED` | 至少一条记录成功，且至少一条记录失败 |
| `FAILED` | 没有任何成功记录，所有记录均处理失败 |

`COMPLETED`、`PARTIAL_FAILED` 和 `FAILED` 是终态，表示本轮执行结束。只有本轮的所有记录都处理完毕后，系统才根据全部记录的结果计算终态。实现了 E1 时，`PARTIAL_FAILED` 或 `FAILED` 的 Job 可以因重试重新进入 `QUEUED`，开始新一轮执行。

#### C2 后台处理（必做）

1. 创建 Job 时，先将其持久化为 `QUEUED`，再交由后台 Worker 异步执行。API 不得等待整个任务处理完成，应尽快返回 Job ID。客户端第一次查询时，Job 可能仍为 `QUEUED`，也可能已进入 `RUNNING`；如果任务很短，也可能已经进入终态。
2. 后台 Worker 按创建顺序取出 `QUEUED` 的 Job，把它置为 `RUNNING`，然后逐条处理记录。
3. 每处理完一条记录，立即持久化该记录的结果，并更新 Job 的进度（已处理数 / 总数）。
4. 单条记录失败后，其他记录继续处理。
5. 所有记录处理完毕后，按 C1 计算并保存 Job 终态。

同一时刻只运行 1 个 Job、其余 Job 保持 `QUEUED` 依次等待，即满足必做要求。单个 Job 内的记录可以串行处理。

后台可以用基于数据库轮询的 Worker、进程内队列、BullMQ + Redis 或其他方案实现。

#### C3 并发限制（加分项）

- 有 2 个或更多 Job 等待时，同时执行 2 个；任意时刻处于 `RUNNING` 的 Job 不超过 2 个。
- 该限制作用于单个应用实例内的全部 Job，跨所有租户计算。实现了 E1 时，重试触发的执行同样计入。
- 超出限制的 Job 保持 `QUEUED`，有空位时按创建或提交重试的顺序依次执行。
- 只要求单实例正确。

#### C4 Job 数据（必做）

每个 Job 至少记录以下字段，你可以增加其他字段：

| 字段 | 说明 |
|------|------|
| Job ID | 系统生成的唯一标识 |
| Tenant ID | 所属租户 |
| Job Name | 用户输入的名称 |
| Job Status | C1 中的状态之一 |
| Total Records | 数据行总数 |
| Successful Records | 成功记录数 |
| Failed Records | 失败记录数 |
| Created At | 创建时间 |
| Started At | 开始执行时间 |
| Finished At | 最近一轮执行进入终态的时间 |

任意时刻必须满足 `Successful Records + Failed Records ≤ Total Records`。Job 处于终态时，两者之和等于 `Total Records`。

### Module D: Persistence & Recovery

#### D1 持久化（必做）

以下数据必须持久化到数据库：

- 租户与用户
- Job 及其状态和统计
- 每条记录的输入、处理结果与失败原因

服务正常重启后，已进入终态的 Job 的状态、统计和记录结果必须完整保留。

如果你没有实现 D2，必须在 README 中写明：重启时处于 `QUEUED` 或 `RUNNING` 的 Job 在重启后会怎样。

#### D2 未完成 Job 的恢复（加分项）

重启时处于 `QUEUED` 或 `RUNNING` 的 Job，在服务启动后继续处理并最终进入终态。恢复策略由你选择，例如重新置为 `QUEUED`，或从未处理的记录继续执行。无论哪种策略，都必须满足：

- 已持久化的成功结果不丢失。
- 同一条记录的结果不被重复计数。
- 没有 Job 永久停留在 `RUNNING`，统计与记录明细一致。

在 `docs/design.md` 中说明你的恢复策略。

#### D3 执行日志（加分项）

为每个 Job 持久化执行日志，至少包括 Job 创建、开始执行、记录失败和进入终态。实现了 E1 或 D2 时，还要记录提交重试和重启后的恢复操作。执行日志同样受租户隔离约束。

### Module E: Retry & Idempotency

#### E1 Failed record retry（加分项）

用户可以对处于 `PARTIAL_FAILED` 或 `FAILED` 状态的 Job 发起“重试失败记录”。

- 只重新处理该 Job 中当前为失败状态的记录。
- 已成功的记录不得被重新处理，其结果保持不变。
- 重试完成后，Job 的统计数据与记录明细一致。
- 重试同样受租户隔离约束，不能对其他租户的 Job 发起重试。
- 对处于 `QUEUED`、`RUNNING` 或 `COMPLETED` 状态的 Job 发起重试，必须拒绝并返回明确错误（例如 `409`）。同一 Job 的并发重试请求只能生效一次。

确定性失败的记录（例如 `[FAIL] Simulated error`）重试后仍然失败，这是预期行为。不得为了让重试成功而修改输入数据或放宽处理规则。

重试期间的状态转换：

```text
PARTIAL_FAILED / FAILED
        │  提交重试：被重试的记录重置为待处理，Failed Records 相应减少
        ▼
     QUEUED
        │  Worker 开始处理
        ▼
     RUNNING
        │  被重试的记录全部处理完毕
        ▼
COMPLETED / PARTIAL_FAILED / FAILED   （按全部记录的最终结果重新计算）
```

#### E2 Idempotent job creation（加分项）

创建 Job 的接口 `POST /api/jobs` 支持 `Idempotency-Key` 请求头：

```http
POST /api/jobs
Idempotency-Key: abc123
```

| 场景 | 期望行为 |
|------|----------|
| 同一租户 + 相同 Key + 相同请求（Job 名称和 CSV 内容均相同） | 不创建新 Job，返回首次创建的同一个 Job ID |
| 同一租户 + 相同 Key + 不同请求（Job 名称或 CSV 内容不同） | 拒绝请求，返回 `409` 和明确的冲突错误 |
| 不同租户使用相同 Key | 互不影响，各自正常创建 Job |
| 请求未携带 `Idempotency-Key` | 正常创建，不做幂等判断 |

- 幂等必须在后端实现。
- 相同 Key 的重复请求同时到达时，也必须只创建一个 Job。
- Key 的有效期由你决定并写入文档，不要求实现过期 Key 的清理。

### Module F: Web Dashboard

前端必须使用 React + TypeScript，至少包含登录页、Job 列表页和 Job 详情页。

#### F1 Login page（必做）

- Email 输入框
- Password 输入框
- **Login** 按钮
- 登录失败时显示的错误信息
- 退出登录的入口（登录后可见）

#### F2 Job list（必做）

每个 Job 显示：

- Job Name
- Job Status
- Created Time
- Total Records
- Success Count
- Failed Count
- Progress

必须支持：

- 进入 Job 详情页
- 创建新 Job 的入口（对应 Module B）

#### F3 Job detail（必做）

必须显示：

- Job 基本信息与当前状态
- Total、Success 和 Failed 统计
- 执行进度
- 每条记录的输入、处理结果与失败原因

Job 处于 `QUEUED` 或 `RUNNING` 时，页面必须自动更新，用 Polling 实现即可。

#### F4 页面加分项（加分项）

- 在 Job 列表中按 Job Status 筛选。
- 用 SSE 或 WebSocket 推送进度，代替 Polling。
- 实现了 D3 时，在详情页显示执行日志。
- 实现了 E1 时，在详情页提供 **Retry Failed Records** 按钮，只在 Job 可重试时可用；后端仍需独立校验。

#### F5 UI 要求（必做）

- 页面布局合理，信息层级清晰，操作可用。
- 必须处理 Loading、Empty 和 Error 状态。
- 状态展示要让用户能区分 `PARTIAL_FAILED` 和 `FAILED`。
- 不要求复杂动画或像素级还原。

## Technical requirements

### 技术栈

| 类别 | 要求 |
|------|------|
| Frontend | React + TypeScript |
| Backend | 语言不限，例如 TypeScript、Java、Python、Go、C#、Rust |
| Framework | 不限，例如 Express、NestJS、Fastify、Spring Boot、FastAPI、Gin、ASP.NET Core |
| Database | PostgreSQL / SQLite |
| ORM | 不限 |
| Job Queue | 不限，基于数据库的 Worker 即可 |
| UI Library | 不限 |
| AI Tools | 不限 |
| Testing | 必须包含自动化测试 |
| Deployment | 本地运行即可 |

### API

创建 Job 的接口固定为 `POST /api/jobs`。其他 API 路径与请求、响应格式由你设计。在项目 README 或 `docs/design.md` 中列出主要 API（登录、退出、创建 Job、Job 列表、Job 详情、记录结果），并给出必要的请求示例或字段说明，方便面试官验证。

### 自动化测试（必做）

测试至少覆盖以下核心逻辑：

- B3 的记录处理规则，每条规则至少一个用例
- C1 的 Job 终态计算（`COMPLETED`、`PARTIAL_FAILED`、`FAILED`）
- 租户隔离：一个租户的用户无法读取另一个租户的 Job

实现了加分项时，应为它补充测试。测试必须能按你 README 中的说明运行并通过，不要求特定覆盖率。

### 配置与依赖

- 必须提供 `.env.example`，列出所有需要的环境变量及示例值。
- 如果依赖 Redis、PostgreSQL 等外部服务，必须写明本地启动这些服务的步骤。
- Docker Compose（加分项）：提供 `docker-compose.yml`，用一条 `docker compose up` 命令启动项目依赖的服务或整个项目。
- 所有依赖必须免费且能在本地运行。

### 不允许

- 在仓库中提交真实的 API Key、数据库密码或其他 Secret
- 用静态 Mock 数据代替后端
- 只实现前端
- 把前端逻辑当作服务端 Authorization

## Acceptance criteria

评审还可能使用其他输入和边界情况，判断依据只有本 README 中的规则。

### 必做场景

| ID | 场景 | 操作 | 期望结果 |
|----|------|------|----------|
| T01 | Tenant A 创建 Job | 以 `user-a@test.com` 登录，上传示例 CSV 创建 Job | 创建请求不等待处理完成，返回 Job ID；后台完成后状态为 `PARTIAL_FAILED`，成功 4 条、失败 1 条 |
| T02 | 跨租户读取 | 以 `user-b@test.com` 登录，通过 API 或 URL 访问 T01 中的 Job ID | 返回 `404` 或 `403`，不返回任何 Job 数据；Tenant B 的列表中不出现该 Job |
| T03 | 非法记录 | 上传一个只包含空白 `text`、超过 200 字符的 `text` 和包含 `[FAIL]` 的 `text` 的 CSV | 每条记录均为 FAILED，并带有可区分的失败原因；Job 终态为 `FAILED` |
| T04 | 失败隔离 | 处理包含失败记录的 Job | 失败记录之后的记录继续被处理，Job 最终进入终态 |
| T05 | 刷新页面 | Job 执行期间刷新详情页 | 页面重新加载后仍显示当前进度，并继续自动更新 |
| T06 | 重启 | 在有已完成 Job 时重启后端 | 已完成 Job 的状态、统计和记录结果完整保留 |
| T07 | 篡改参数 | 在请求体、Query 或 Header 中加入其他租户的 `tenant_id`，或修改 Job ID | 无法读取、创建或操作其他租户的数据 |
| T08 | 非法文件 | 上传空文件，或缺少 `text` 表头的 CSV | 返回 `400` 和明确的错误信息，不创建 Job |
| T09 | 退出登录 | 登录后退出，在同一浏览器重新打开 Job 列表；随后不携带有效凭证调用受保护 API | 页面回到登录页，API 返回 `401`；无状态 JWT 不要求服务端撤销 |

### 加分场景

| ID | 对应需求 | 操作 | 期望结果 |
|----|----------|------|----------|
| B01 | C3 并发限制 | 快速提交 5 个 Job | 任意时刻处于 `RUNNING` 的 Job 不超过 2 个，且有 2 个 Job 同时运行的时段；其余为 `QUEUED`，最终全部进入终态 |
| B02 | E1 重试 | 对 `PARTIAL_FAILED` 的 Job 发起重试 | 只重新处理失败记录；成功记录未被重新处理、结果不变；`[FAIL]` 记录仍为 FAILED；统计与明细一致 |
| B03 | E2 幂等创建 | 同一租户使用相同 `Idempotency-Key` 和相同请求连续或并发提交两次 | 只创建一个 Job，两次返回同一个 Job ID |
| B04 | E2 幂等冲突与租户范围 | 同一租户用相同 Key 提交不同的 CSV；另一个租户用相同 Key 提交 | 前者返回 `409`；后者正常创建自己的 Job |
| B05 | D2 恢复 | 在有 `RUNNING` 和 `QUEUED` 的 Job 时重启后端 | 这些 Job 继续处理并进入正确终态，成功结果不丢失，不重复计数 |
| B06 | 其他扩展 | 检查声明实现的执行日志、Job 筛选、SSE/WebSocket、失败重试按钮或 Docker Compose | 对应功能可运行并符合文档要求；按下文的加分细则评分 |

## Deliverables

在你自己的 GitHub 仓库中提交项目。

### 建议目录结构

可以使用 Monorepo。`docker-compose.yml` 只在你做了 Docker Compose 加分项时需要。

```text
taskforge/
├── frontend/
├── backend/
├── README.md
├── .env.example
├── docker-compose.yml
└── docs/
    ├── design.md
    └── ai-development.md
```

### 必须提交的内容

#### 1. Source code

完整的前端和后端源代码，以及数据库迁移或初始化脚本、seed 脚本。

#### 2. README.md

至少包含：

- 项目简介和技术栈
- 环境要求（前端构建环境、后端运行时和框架版本、数据库及其他依赖）
- 安装、数据库初始化和启动步骤
- 测试账号与登录方式
- 运行测试的方式
- 完成情况：逐项列出必做项和加分项，标明已完成、部分完成或未做
- 已知问题，以及重启时未完成的 Job 会怎样（未实现 D2 时）
- 使用的脚手架或项目模板及其来源（如果有）

#### 3. docs/design.md

一到两页即可，说明：

- 数据模型：有哪些表，Job、记录和租户之间的关系
- Job 的处理流程：创建请求返回后，Worker 如何取到 Job、逐条处理并计算终态
- 租户隔离：租户身份从哪里来，后端在哪一层按租户过滤
- 你做了哪些加分项，每项的实现思路（一两段即可）

#### 4. docs/ai-development.md

说明以下内容：

- 使用了哪些 AI 工具
- 你如何拆分工作，哪些部分交给 AI，哪些部分自己写或自己决定
- 至少 2 个具体的 AI 辅助工程实践案例，例如纠正 AI 生成的代码、做架构决策、改进 Prompt、用测试验证 AI 的输出、处理边界问题
- 如果再给你 2 小时，你接下来会做什么

每个案例写清楚涉及哪段代码或哪个决策、你做了什么、怎么验证 AI 生成的结果。重点是你如何验证和控制 AI 生成的代码，案例中 AI 出错的次数不影响评分。不需要提交完整对话记录；不得提交包含个人隐私或 Secret 的日志。

#### 5. Automated tests

满足 [Technical requirements](#technical-requirements) 中自动化测试的要求，并能按你 README 中的说明运行。

## Time expectations

- 建议实际开发时间为 6～8 小时，8 小时为建议上限。
- 提交窗口为收到题目后 7 天，开发时间在窗口内自行安排。
- 先保证必做项能运行、能验证，再做加分项。一个完整可运行的必做版本，比功能更多但跑不起来的版本得分更高。
- 8 小时内没有完成所有功能也可以提交，但必须在 README 中说明完成情况和已知问题。完成清单是否如实计入 Documentation & Communication 维度。

## Evaluation criteria

总分 100 分：

| 维度 | 分值 | 评估内容 |
|------|------|----------|
| Core Functionality | 35 | 核心业务流程是否完整可用，重点参考 T01、T03～T06 等功能验收场景 |
| Code Quality & Runnability | 20 | 能否按 README 从零启动，代码结构是否清楚，测试能否运行并覆盖核心逻辑 |
| Security & Edge Cases | 15 | 认证与租户隔离、输入校验和异常边界，重点参考 T02、T07～T09 |
| AI-Assisted Development & Debugging | 15 | 如何验证和控制 AI 生成的代码，遇到问题时如何排查，`docs/ai-development.md` 中的案例是否具体 |
| Documentation & Communication | 10 | README 和 `docs/design.md` 是否清楚，完成情况是否如实 |
| Bonus | 5 | 按 B01～B06 对已实现的进阶功能评分，最多 5 分 |

### Bonus 计分细则

| 进阶功能 | 对应验收 | 分值 |
|----------|----------|------|
| 双任务并发控制 | B01 | 1 |
| 失败记录重试 | B02 | 1 |
| 幂等创建（包含重复、冲突及租户范围） | B03、B04 均符合要求 | 1 |
| 未完成任务重启恢复 | B05 | 1 |
| 其他增强：执行日志 | B06，D3 | 0.2 |
| 其他增强：Job 状态筛选 | B06，F4 | 0.2 |
| 其他增强：SSE 或 WebSocket 实时进度 | B06，F4 | 0.2 |
| 其他增强：失败重试按钮 | B06，F4（需实现 E1） | 0.2 |
| 其他增强：Docker Compose 一键启动 | B06，配置与依赖 | 0.2 |
| **合计** | | **5** |

没有实现加分项不影响对应必做项的分数。加分功能只有在实际可用、可验证时才计分；加分项目实现质量（如代码结构和测试）也可以作为相应工程能力维度的证据，但同一缺陷不重复扣分。

### 评审原则

- 必做项扣掉的分数不能用加分项直接抵扣。
- 同一个缺陷只在最相关的评分维度扣分一次。功能缺失、权限漏洞、运行失败等应记录具体可复现的证据。
- 能跨租户读取或操作数据的漏洞属于重大安全问题，会显著影响整体评估结果，加分项无法弥补。
- 评分关注实际交付质量。代码行数、Commit 数量和 AI Token 消耗不计分。
- 是否合格综合所有维度判断，不只看是否完成全部功能。
- AI-Assisted Development & Debugging 的 15 分中，建议将 10 分用于项目和 `docs/ai-development.md`，5 分用于后续面试的现场变更。**初筛阶段这 5 分暂不评分，不计为扣分**；最终成绩在面试后确定。

## 后续面试

通过项目初筛的候选人，将参加约 **45 分钟**的技术交流，内容全部围绕自己提交的项目：

1. **代码讲解（约 30 分钟）**：说明 Job 状态如何计算、租户隔离在哪里执行、如何验证某个 AI 生成的实现，以及哪些需求尚未完成。
2. **现场小型变更（约 15 分钟）**：基于现有代码完成一项范围明确的修改，允许继续使用 AI Coding Agent。重点考察需求理解、代码定位、修改验证与 Debug，而不是手写速度。

现场变更不引入新的大型技术栈，具体要求由面试官现场给出。面试安排由招聘方另行通知。

## Submission

1. 创建你自己的 GitHub 仓库，公开或按招聘方要求设置访问权限。
2. 完成开发，把完整源代码推送到你的仓库。
3. 按你的 README 步骤从零启动项目并运行测试，确认步骤可用。
4. 通过与招聘方约定的渠道提交仓库地址。

不要向本挑战仓库的 `main` 分支提交代码或 Pull Request。如果招聘方要求，也可以 Fork 本仓库后在 Fork 中开发。

提交时提供：

- 仓库 URL
- 本地启动方式
- 测试方式
- 完成情况（必做项和加分项）
- 已知限制

## AI usage policy

- 可以使用任何 AI 工具和模型，例如 Cursor、OpenAI Codex、Claude Code、GitHub Copilot、OpenCode、Gemini CLI，也可以组合使用。
- 允许 100% 由 AI 生成代码，不限制 AI 模型、Token 数量或工具组合。
- 你对最终交付的代码完全负责，必须能够理解、解释、验证和维护它。
- 不得在仓库中提交 Secret、API Key，或包含隐私数据的 Agent 日志与配置。
- 可以使用第三方开源库，但必须遵守其许可证。

## FAQ

### 是否允许使用 AI 编程工具？

允许，不限制 AI 使用比例。

### 是否必须使用指定 AI 模型？

不需要。

### 可以使用 AI 生成整个项目吗？

可以，但你必须能够理解、验证和维护最终代码。

### 是否需要部署到公网？

不需要，本地运行即可。

### 是否需要使用真实 AI API？

不需要。记录处理规则是确定性的文本转换，使用模拟耗时即可。

### 是否必须使用 Docker？

不需要，但需要提供可复现的启动方式。Docker Compose 是加分项。

### 是否必须实现所有功能才能提交？

不需要，但必须在 README 中说明未完成的部分。

### 后端必须使用 Node.js 吗？

不需要。后端语言和框架不限，可以使用 Java、Python、Go、TypeScript、C# 等。**前端仍须使用 React + TypeScript，数据库仍须使用 PostgreSQL 或 SQLite。** 项目需要能够按 README 在本地运行。

### 是否可以使用第三方开源库？

可以，但应遵守相关许可证。

### 可以使用现成的脚手架或项目模板吗？

可以使用公开的基础模板或脚手架，但需在项目 README 中写明来源。任务处理、权限隔离等本题业务逻辑必须按要求完成；不能提交与本题无关的现成成品作为自己的实现。

### 需要事先熟悉消息队列吗？

不需要。必做项用一个按顺序处理数据库中 `QUEUED` Job 的 Worker 就够了。不熟悉的框架、数据库或队列，可以在挑战期间借助 AI 学习。

### 加分项没做，会影响通过吗？

加分项单独占 5 分，大部分分数来自必做项的质量。是否通过综合所有维度判断。

### 需求中没有覆盖到的细节怎么处理？

做出合理假设，并写进你的 README 或 `docs/design.md`。
