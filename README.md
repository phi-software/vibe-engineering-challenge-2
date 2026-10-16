# Vibe Engineering Challenge

## Challenge 001 — TaskForge: Multi-Tenant Async Job Processing Platform

## Introduction

你需要借助 AI Coding Agent，用 6 到 8 小时的开发时间实现 TaskForge：一个多租户的批量数据处理平台。用户上传 CSV 文件，后端在后台逐条处理每一行，用户在 Web 页面查看处理进度和结果。

这道 Take-Home Assignment 面向大四本科生和即将毕业的计算机相关专业硕士研究生，主要考察 AI Coding Agent 辅助下的全栈工程交付能力：能否借助 AI 独立交付一个有业务逻辑的 Full-Stack Engineering 项目。你可以用任意 AI 工具和模型生成全部代码，但要对最终交付的代码负责，规则见 [AI usage policy](#ai-usage-policy)。

> We evaluate engineering outcomes, not typing speed.

需求分为必做和加分项。必做项组成一个可运行的完整 MVP，评审首先看这部分；加分项包括并发限制、失败记录重试、幂等创建和重启恢复等。

## Challenge at a glance

| 项目 | 说明 |
|------|------|
| 挑战名称 | TaskForge |
| 项目类型 | Multi-Tenant Async Job Processing Platform |
| 目标岗位 | Full-Stack Software Engineer（校招） |
| 目标候选人 | 大四本科生、即将毕业的计算机相关专业硕士研究生 |
| 建议难度 | Entry level |
| 建议开发时间 | 6 到 8 小时（8 小时为建议上限） |
| 提交窗口 | 收到题目后 7 天 |
| 开发模式 | AI-Assisted Development |
| 技术栈 | 前端 React + TypeScript；后端 Node.js + TypeScript；数据库 PostgreSQL 或 SQLite |
| 需求分级 | 必做 + 加分项 |
| 部署要求 | 本地运行 |

## 候选人范围与公平性

- 不要求正式工作经验。
- 本科生和研究生使用同一套岗位能力评价标准。
- 可以从零开发，也可以基于 Vite、NestJS CLI 等脚手架或公开的基础项目模板开发，并在 README 中写明来源。可以使用开源库。
- 不强制使用 Docker、云部署、Redis 或 Kubernetes。
- 不需要提交完整的 AI 对话记录。

## Background

一家 SaaS 公司为多个企业客户（租户）提供批量数据处理服务。

企业用户在 Web 页面上传 CSV 文件后，系统把每一行转换为一条待处理记录，并创建一个后台异步任务（Job）。Background Worker 逐条处理这些记录。用户在 Web 页面查看 Job 的状态、执行进度、处理结果和失败原因。

任何用户都不能看到或操作其他租户的数据。

## Project objective

你需要交付一个可以实际运行、测试和验证的 MVP，包括：

- React + TypeScript Web Dashboard
- Node.js + TypeScript 后端 API
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
2. 用户退出登录。退出时清除客户端保存的登录凭证（token 或 Cookie），页面回到登录页；使用服务端 Session 时，服务端同时使该 Session 失效。退出后在同一浏览器中访问受保护页面会回到登录页，调用受保护 API 返回 `401`。
3. 维持登录状态：刷新页面后仍保持登录，直到用户退出或凭证过期。
4. 服务端验证每个受保护 API 请求的身份，未认证的请求返回 `401`。

认证方案可以是 JWT、Session 或其他合理方案。

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

Trim 指去除首尾空白，空白的范围与 JavaScript 的 `String.prototype.trim()` 相同。长度按 Trim 后的 Unicode 字符（code point）计数，不按字节或 UTF-16 编码单元计数：恰好 200 个字符的 `text` 通过规则 2，201 个字符为 FAILED。

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

1. 创建 Job 的 API 不得同步等待处理完成。完成校验和持久化后，必须立即返回 Job ID，此时 Job 为 `QUEUED`。
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
| Backend | Node.js + TypeScript |
| Framework | 不限，例如 Express、NestJS、Fastify |
| Database | PostgreSQL / SQLite |
| ORM | 不限 |
| Job Queue | 不限，基于数据库的 Worker 即可 |
| UI Library | 不限 |
| AI Tools | 不限 |
| Testing | 必须包含自动化测试 |
| Deployment | 本地运行即可 |

### API

除 `POST /api/jobs`（创建 Job）外，API 路径和格式由你设计。在你的 README 或 `docs/design.md` 中列出主要 API，包括登录、退出、Job 列表、Job 详情和记录结果。

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
| T01 | Tenant A 创建 Job | 以 `user-a@test.com` 登录，上传示例 CSV 创建 Job | 立即返回 Job ID，Job 进入 `QUEUED` 或 `RUNNING`；处理结束后状态为 `PARTIAL_FAILED`，成功 4 条、失败 1 条 |
| T02 | 跨租户读取 | 以 `user-b@test.com` 登录，通过 API 或 URL 访问 T01 中的 Job ID | 返回 `404` 或 `403`，不返回任何 Job 数据；Tenant B 的列表中不出现该 Job |
| T03 | 非法记录 | 上传一个只包含空白 `text`、超过 200 字符的 `text` 和包含 `[FAIL]` 的 `text` 的 CSV | 每条记录均为 FAILED，并带有可区分的失败原因；Job 终态为 `FAILED` |
| T04 | 失败隔离 | 处理包含失败记录的 Job | 失败记录之后的记录继续被处理，Job 最终进入终态 |
| T05 | 刷新页面 | Job 执行期间刷新详情页 | 页面重新加载后仍显示当前进度，并继续自动更新 |
| T06 | 重启 | 在有已完成 Job 时重启后端 | 已完成 Job 的状态、统计和记录结果完整保留 |
| T07 | 篡改参数 | 在请求体、Query 或 Header 中加入其他租户的 `tenant_id`，或修改 Job ID | 无法读取、创建或操作其他租户的数据 |
| T08 | 非法文件 | 上传空文件，或缺少 `text` 表头的 CSV | 返回 `400` 和明确的错误信息，不创建 Job |
| T09 | 退出登录 | 登录后退出，再在同一浏览器中打开 Job 列表页并调用受保护 API | 页面回到登录页，API 返回 `401` |

### 加分场景

| ID | 对应需求 | 操作 | 期望结果 |
|----|----------|------|----------|
| B01 | C3 并发限制 | 快速提交 5 个 Job | 任意时刻处于 `RUNNING` 的 Job 不超过 2 个，且有 2 个 Job 同时运行的时段；其余为 `QUEUED`，最终全部进入终态 |
| B02 | E1 重试 | 对 `PARTIAL_FAILED` 的 Job 发起重试 | 只重新处理失败记录；成功记录未被重新处理、结果不变；`[FAIL]` 记录仍为 FAILED；统计与明细一致 |
| B03 | E2 幂等创建 | 同一租户使用相同 `Idempotency-Key` 和相同请求连续或并发提交两次 | 只创建一个 Job，两次返回同一个 Job ID |
| B04 | E2 幂等冲突与租户范围 | 同一租户用相同 Key 提交不同的 CSV；另一个租户用相同 Key 提交 | 前者返回 `409`；后者正常创建自己的 Job |
| B05 | D2 恢复 | 在有 `RUNNING` 和 `QUEUED` 的 Job 时重启后端 | 这些 Job 继续处理并进入正确终态，成功结果不丢失，不重复计数 |

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
- 环境要求（Node.js 版本、数据库、其他依赖）
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

- 建议实际开发时间为 6 到 8 小时，8 小时为建议上限。
- 提交窗口为收到题目后 7 天，开发时间在窗口内自行安排。
- 先保证必做项能运行、能验证，再做加分项。一个完整可运行的必做版本，比功能更多但跑不起来的版本得分更高。
- 8 小时内没有完成所有功能也可以提交，但必须在 README 中说明完成情况和已知问题。完成清单是否如实计入 Documentation & Communication 维度。

## Evaluation criteria

总分 100 分：

| 维度 | 分值 | 评估内容 |
|------|------|----------|
| Core Functionality | 35 | 必做项是否完整实现，必做场景 T01 到 T09 是否通过 |
| Code Quality & Runnability | 20 | 能否按 README 从零启动，代码结构是否清楚，测试能否运行并覆盖核心逻辑 |
| Security & Edge Cases | 15 | 租户隔离、Authentication、输入校验、边界情况处理 |
| AI-Assisted Development & Debugging | 15 | 如何验证和控制 AI 生成的代码，遇到问题时如何排查，`docs/ai-development.md` 中的案例是否具体 |
| Documentation & Communication | 10 | README 和 `docs/design.md` 是否清楚，完成情况是否如实 |
| Bonus | 5 | 加分项按本 README 的定义实现并能验证，加分场景 B01 到 B05 是否通过 |

- 加分项最多 5 分。高质量的进阶实现（例如并发控制的正确性和对应测试）也会作为 Code Quality & Runnability、Security & Edge Cases 等相应维度的评分依据。
- 必做项扣掉的分数不能用加分项补回。
- 同一个问题只在最相关的一个维度扣分。
- 能跨租户读取或操作数据的漏洞属于重大安全问题，会显著影响整体评估结果，加分项无法弥补。
- 评分关注实际交付质量。代码量、Commit 数量和 AI Token 消耗不计分。
- 是否合格综合所有维度判断，不只看是否完成了全部功能。

## 后续面试

通过初筛的候选人参加两部分面试，都基于你自己提交的项目。

约 30 分钟的项目代码讲解，内容包括：

- Job 终态是怎样计算的，代码在哪里
- 租户隔离在后端是怎样实现的，为什么修改 Job ID 拿不到其他租户的数据
- `docs/ai-development.md` 中的一个案例：你如何验证和控制 AI 生成的代码

约 15 分钟的现场小型需求变更：在你现有代码的基础上完成一个小改动，不引入复杂的新技术栈。现场可以继续使用 AI Coding Agent。这部分重点观察代码理解、AI 使用、修改后的验证和 Debug 能力。

面试时间和方式由招聘方另行通知。

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

### 是否可以使用第三方开源库？

可以，但应遵守相关许可证。

### 需要事先熟悉消息队列吗？

不需要。必做项用一个按顺序处理数据库中 `QUEUED` Job 的 Worker 就够了。不熟悉的框架、数据库或队列，可以在挑战期间借助 AI 学习。

### 加分项没做，会影响通过吗？

加分项单独占 5 分，大部分分数来自必做项的质量。是否通过综合所有维度判断。

### 可以用模板或脚手架吗？

可以，例如 Vite 或 NestJS CLI 生成的项目骨架，或公开的基础项目模板。在 README 中写明来源；业务逻辑必须是你为这道题实现的。

### 需求中没有覆盖到的细节怎么处理？

做出合理假设，并写进你的 README 或 `docs/design.md`。
