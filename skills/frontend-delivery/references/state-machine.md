# 状态字段与状态机

本文档是 `workflow-state.json` 所有字段和状态取值的唯一权威定义。SKILL.md 和其他参考文件不重复定义状态语义，出现分歧时以本文档为准。

状态文件的作用是让工作流可以在任意时刻中断，并在下次调用时准确恢复到正确位置。它不是日志，不记录过程细节；待确认问题写 `context.md`，验证命令和结果写 `delivery.md`。

---

## 目录

- [完整字段示例](#完整字段示例)
- [字段逐项说明](#字段逐项说明)
- [`status`：全局状态四取值](#status全局状态四取值)
- [`stageStatus`：阶段状态六取值](#stagestatus阶段状态六取值)
- [`blockingItems`：阻塞项](#blockingitems阻塞项)
- [`invalidatedItems`：失效项](#invalidateditems失效项)
- [`blocked` 与 `invalidated` 的区别](#blocked-与-invalidated-的区别)
- [版本三字段的关系](#版本三字段的关系)
- [`executionMode` 与 `targetStage` 的约束](#executionmode-与-targetstage-的约束)
- [多项目文件归属规则](#多项目文件归属规则)
- [凭证禁止写入](#凭证禁止写入)
- [常见误判纠正](#常见误判纠正)
- [总结](#总结)

三级标题见正文。

---

## 完整字段示例

```json
{
  "requirementId": "feature-001",
  "workspaceRoot": "/workspace/my-project",
  "outputDir": "/workspace/my-project/docs/ai-delivery/feature-001",
  "primaryProjectPath": "/workspace/my-project/client-app",
  "projectPaths": [
    "/workspace/my-project/client-app",
    "/workspace/my-project/admin-app"
  ],
  "requirementDocPath": "/workspace/my-project/docs/requirements/feature-001.md",
  "uiSourceType": "local",
  "uiDir": "/workspace/my-project/docs/requirements/ui",
  "assetsDir": "/workspace/my-project/docs/requirements/assets",
  "apiDocPath": "/workspace/my-project/docs/api/feature-001.md",
  "executionMode": "full_delivery",
  "targetStage": "delivery",
  "requirementVersion": "2026-08-13T09:00:00+08:00",
  "planBasedOnRequirementVersion": "2026-08-13T09:00:00+08:00",
  "planVersion": "2026-08-13T10:00:00+08:00",
  "currentStage": "planning",
  "status": "awaiting_confirmation",
  "stageStatus": {
    "context": "complete",
    "planning": "awaiting_confirmation",
    "implementation": "pending",
    "verification": "pending",
    "delivery": "pending"
  },
  "blockingItems": [],
  "invalidatedItems": [],
  "approvedPlanVersion": null,
  "approvedAt": null,
  "approvedByRole": null,
  "lastUpdate": "2026-08-13T10:30:00+08:00"
}
```

---

## 字段逐项说明

### 标识与路径

| 字段 | 类型 | 取值范围 | 更新时机 |
| --- | --- | --- | --- |
| `requirementId` | string | 开发指定的唯一标识，如 `feature-001`、`202608131512` | 首次创建时写入，之后不变 |
| `workspaceRoot` | string | 规范化绝对路径 | 首次创建时写入；开发显式变更工作区时更新 |
| `outputDir` | string | 规范化绝对路径 | 首次创建时写入；开发显式迁移产物目录时更新 |
| `primaryProjectPath` | string | 规范化绝对路径 | 首次创建时写入；项目集合变化时复核 |
| `projectPaths` | string[] | 一个或多个规范化绝对路径 | 开发新增或移除关联项目时更新 |
| `requirementDocPath` | string \| null | 绝对路径或 `null` | 开发补充需求文档后更新 |
| `uiSourceType` | string | 首版只有 `local` | 固定值，首版不变 |
| `uiDir` | string \| null | 绝对路径或 `null` | 开发补充 UI 目录后更新 |
| `assetsDir` | string \| null | 绝对路径或 `null` | 开发补充切图目录后更新 |
| `apiDocPath` | string \| null | 绝对路径或 `null` | 开发补充接口文档后更新 |

补充规则：

- `workspaceRoot` 是所有相对输入路径的唯一解析基准。开发给出 `./docs/ui` 这类相对路径时，一律相对 `workspaceRoot` 解析，然后以规范化绝对路径持久化。
- `primaryProjectPath` 只有两个用途：单项目时决定默认 `outputDir`；多项目时决定项目内文件建议的默认落点。它**不**参与文件归属判断（见"多项目文件归属"）。
- 路径字段允许为空或指向尚不存在的位置。这不是错误，工作流在对应阶段补充。缺 `uiDir` 只在"本次需求依赖 UI"时才构成阻塞。
- `outputDir` 默认值：单项目 `<primaryProjectPath>/docs/ai-delivery/<requirementId>/`；多项目 `<workspaceRoot>/docs/ai-delivery/<requirementId>/`。

### 执行控制

| 字段 | 类型 | 取值范围 | 更新时机 |
| --- | --- | --- | --- |
| `executionMode` | string | `review_only` \| `planning_only` \| `full_delivery` | 首次创建时写入；开发显式升级或降级时更新 |
| `targetStage` | string | `context` \| `planning` \| `implementation` \| `verification` \| `delivery` | 随 `executionMode` 同步更新 |
| `currentStage` | string | 同上五个阶段名 | 每次阶段切换时更新 |

`executionMode` 与 `targetStage` 的对应关系：

| `executionMode` | `targetStage` | 含义 |
| --- | --- | --- |
| `review_only` | `context` | 只做项目分析和需求评审，产出 `context.md` 后停 |
| `planning_only` | `planning` | 做到技术计划产出并等待批准，不改业务代码 |
| `full_delivery` | `delivery` | 走完五个阶段 |

### 版本

| 字段 | 类型 | 取值范围 | 更新时机 |
| --- | --- | --- | --- |
| `requirementVersion` | string \| null | 带时区 ISO 8601，或需求系统原始版本号 | 需求基线确立时写入；需求变更确认后更新 |
| `planBasedOnRequirementVersion` | string \| null | 同上格式 | 生成或更新 `plan.md` 时，设为当时的 `requirementVersion` |
| `planVersion` | string \| null | 带时区 ISO 8601 | `plan.md` 的**可执行内容**变化时更新 |

时间格式规则：使用带时区的 ISO 8601，如 `2026-08-13T09:00:00+08:00`。如果需求系统本身提供版本号（如 `v3`、`rev-17`），优先记录原始版本号，不自行改写成时间戳。

### 批准

| 字段 | 类型 | 取值范围 | 更新时机 |
| --- | --- | --- | --- |
| `approvedPlanVersion` | string \| null | 必须等于某个 `planVersion` 值，或 `null` | 开发批准计划时写入；变更影响已批准计划时清空 |
| `approvedAt` | string \| null | 带时区 ISO 8601 | 与 `approvedPlanVersion` 同时写入或同时清空 |
| `approvedByRole` | string \| null | 角色名，如 `developer`、`tech_lead` | 与 `approvedPlanVersion` 同时写入或同时清空 |

三个批准字段必须**同时**写入、**同时**清空。出现 `approvedPlanVersion` 有值但 `approvedAt` 为 `null` 的组合，视为状态文件损坏，需要重新走门禁二。

`approvedByRole` 只记录角色，不记录姓名。确认人未提供姓名时不虚构。

### 状态

| 字段 | 类型 | 取值范围 | 更新时机 |
| --- | --- | --- | --- |
| `status` | string | `active` \| `awaiting_confirmation` \| `blocked` \| `complete` | 每次写状态文件时按派生规则重算 |
| `stageStatus` | object | 五个阶段各一个状态值 | 阶段推进、阻塞、失效、批准时更新 |
| `blockingItems` | object[] | 阻塞项数组 | 新增阻塞时追加；阻塞解除时移除 |
| `invalidatedItems` | object[] | 失效项数组 | 需求变更导致局部失效时追加；范围重做完成后移除 |
| `lastUpdate` | string | 带时区 ISO 8601 | 每次写状态文件时更新。模板里是 `null`，创建状态文件时必须立即替换为当前时间——落盘后的状态文件中该字段不允许为 `null` |

---

## `status`：全局状态四取值

| 取值 | 含义 |
| --- | --- |
| `active` | 存在可以立即执行的任务，工作流正在推进，不需要等人 |
| `awaiting_confirmation` | 不存在 `blockingItems`，剩余工作在等门禁确认（计划批准、人工验证结论） |
| `blocked` | 剩余工作全部因 `blockingItems` 或 `invalidatedItems` 无法推进，含 `p0_unresolved`、缺少输入、接口冲突 |
| `complete` | 已到达 `targetStage` 且该阶段状态为 `complete` |

**`status` 不独立维护，也不靠猜测。** 它是每次写状态文件时从 `stageStatus`、`blockingItems` 和"是否仍有可执行任务"派生出来的结果。

### 派生规则

按顺序判断，第一个匹配的即为结果：

1. `targetStage` 对应的 `stageStatus` 为 `complete` → `status = "complete"`
2. 存在至少一个当前可执行任务（未被任何 `blockingItems` 的 scope 覆盖、未被 `invalidatedItems` 阻断、不需要等人工确认）→ `status = "active"`
3. 没有可执行任务，且 `blockingItems` 或 `invalidatedItems` 非空 → `status = "blocked"`
4. 没有可执行任务，且两个数组都为空（剩余工作在等门禁确认）→ `status = "awaiting_confirmation"`

两个关键点：

**规则 2 优先于规则 3 和 4。** 只要还有一件事能做，全局就是 `active`，无论 `blockingItems` 里有多少条。这是范围级阻塞的核心语义——阻塞停的是范围，不是整个工作流。

**规则 3 优先于规则 4，判别依据是 `blockingItems` 是否为空，不是"在等谁"。** 有阻塞项就是 `blocked`，即使等待对象是人（产品回复 P0）。这样区分是因为两者的性质不同：`blocked` 表示缺了必需的输入或存在冲突，需要外部补齐；`awaiting_confirmation` 表示产出已完备，只等门禁放行。P0 未确认属于前者——缺的是决策输入，`context.md` 里的问题还是问号。而计划待批准属于后者——`plan.md` 是完整的，只是没盖章。

按此规则，三种常见等待的结果是：

| 情形 | `blockingItems` | 全局 `status` |
| --- | --- | --- |
| P0 未确认 | 有 `p0_unresolved` | `blocked` |
| 计划待批准 | 空 | `awaiting_confirmation` |
| 人工验证待确认 | 空 | `awaiting_confirmation` |

### 组合示例

**示例 A：多项目，一个项目不可访问，另一个可以继续**

```json
{
  "currentStage": "context",
  "status": "active",
  "stageStatus": { "context": "active", "planning": "pending", "implementation": "pending", "verification": "pending", "delivery": "pending" },
  "blockingItems": [
    { "code": "inaccessible_project", "scope": "/workspace/my-project/admin-app", "createdAt": "2026-08-13T09:10:00+08:00", "details": "路径不存在或无读取权限" }
  ]
}
```

有 `blockingItems` 但 `status` 是 `active`：`client-app` 的分析仍可进行，命中派生规则 2。

**示例 B：等待计划批准**

```json
{
  "currentStage": "planning",
  "status": "awaiting_confirmation",
  "stageStatus": { "context": "complete", "planning": "awaiting_confirmation", "implementation": "pending", "verification": "pending", "delivery": "pending" },
  "blockingItems": [],
  "approvedPlanVersion": null
}
```

没有可执行任务，两个数组都为空，唯一的下一步是"开发批准计划"，命中规则 4。计划未批准**不是**阻塞，不进 `blockingItems`。

**示例 C：完全阻塞**

```json
{
  "currentStage": "context",
  "status": "blocked",
  "stageStatus": { "context": "blocked", "planning": "pending", "implementation": "pending", "verification": "pending", "delivery": "pending" },
  "blockingItems": [
    { "code": "missing_requirement", "scope": "context", "createdAt": "2026-08-13T08:00:00+08:00", "details": "未提供需求文档也未提供需求描述" }
  ]
}
```

没有需求就什么都做不了，`blockingItems` 非空，命中规则 3。

**示例 D：范围阻塞 + 范围等确认，但仍有活可干**

```json
{
  "currentStage": "implementation",
  "status": "active",
  "stageStatus": { "context": "complete", "planning": "complete", "implementation": "active", "verification": "pending", "delivery": "pending" },
  "blockingItems": [
    { "code": "missing_api_contract", "scope": "client-app/api/user-profile", "createdAt": "2026-08-13T13:00:00+08:00", "details": "资料保存接口缺少后端确认的字段定义" },
    { "code": "api_conflict", "scope": "admin-app/api/user", "createdAt": "2026-08-13T14:00:00+08:00", "details": "实际响应字段与已确认契约不一致" }
  ]
}
```

两个接口任务停了，但页面骨架、样式还原、埋点等任务不依赖它们，命中规则 2。

**示例 E：只剩人工验证**

```json
{
  "currentStage": "verification",
  "status": "awaiting_confirmation",
  "stageStatus": { "context": "complete", "planning": "complete", "implementation": "complete", "verification": "awaiting_confirmation", "delivery": "pending" },
  "blockingItems": []
}
```

自动验证已通过，真机验证需要开发执行，两个数组都为空，命中规则 4。此时 `delivery` 不得标 `complete`。

**示例 F：review_only 模式已完成**

```json
{
  "executionMode": "review_only",
  "targetStage": "context",
  "currentStage": "context",
  "status": "complete",
  "stageStatus": { "context": "complete", "planning": "pending", "implementation": "pending", "verification": "pending", "delivery": "pending" }
}
```

`targetStage` 是 `context` 且已 `complete`，命中规则 1。后面四个阶段保持 `pending` 是正确的，不是未完成。

---

## `stageStatus`：阶段状态六取值

| 取值 | 含义 | 典型来源 |
| --- | --- | --- |
| `pending` | 尚未开始，前置条件未满足或轮次未到 | 初始值 |
| `active` | 正在执行，有可推进的任务 | 前一阶段完成后进入 |
| `awaiting_confirmation` | 本阶段工作已做到位，等待人工确认才能标完成 | 计划待批准、人工验证待执行 |
| `blocked` | 本阶段全部剩余工作都无法推进 | 缺需求、P0 未解决、全部范围被阻塞 |
| `invalidated` | 本阶段**全部**已完成或已批准内容因需求变更失效 | 需求变更影响面覆盖整个阶段 |
| `complete` | 本阶段全部必要工作完成，含必要的人工确认 | 门禁通过或验证全部完成 |

五个阶段固定为：`context`、`planning`、`implementation`、`verification`、`delivery`。

### 门禁状态归属（易错点）

等待计划批准时，状态是：

```json
{
  "stageStatus": {
    "context": "complete",
    "planning": "awaiting_confirmation",
    "implementation": "pending"
  },
  "approvedPlanVersion": null
}
```

批准之后才变成：

```json
{
  "stageStatus": {
    "context": "complete",
    "planning": "complete",
    "implementation": "active"
  },
  "approvedPlanVersion": "2026-08-13T10:00:00+08:00",
  "approvedAt": "2026-08-13T10:35:00+08:00",
  "approvedByRole": "developer"
}
```

❌ **错误：** `planning: "complete"` + `implementation: "awaiting_confirmation"`

✅ **正确：** `planning: "awaiting_confirmation"` + `implementation: "pending"`

原因：等待批准的对象是**计划**，责任在 planning 阶段。计划没批准，实现阶段一步都没走，不该有 `pending` 之外的状态。把等待挂到 `implementation` 上会让恢复逻辑误判"实现已经开始"，进而可能跳过门禁二。

门禁一同理：P0 未解决时是 `context` 为 `blocked`、`planning` 保持 `pending`。等待的对象是需求确认，责任在 context 阶段——`context.md` 已生成不等于该阶段完成，门禁没过就还没完成。不要让 `planning` 提前变成 `blocked` 或 `awaiting_confirmation`，那会让恢复逻辑误判"已进入计划阶段"。

### 验证阶段三态（易错点）

`verification` 的三个可能落点，区分标准是"必要验证是否全部完成"和"失败是否由本次修改导致"：

| 情形 | `stageStatus.verification` | `blockingItems` | 说明 |
| --- | --- | --- | --- |
| 自动验证通过，人工验证项（真机、视觉、权限账号）未执行 | `awaiting_confirmation` | 无 | 在 `delivery.md` 列出待人工执行清单；`delivery` 保持 `pending` |
| 本次修改导致 lint / 测试 / 构建失败，且无法立即修复 | `blocked`（或范围级阻塞时仍为 `active`） | 追加 `verification_failed` | `verification` 和 `delivery` 都不得标 `complete` |
| 既有问题导致失败，与本次修改无关 | 可继续（不因此变 `blocked`） | 不追加 | 必须记录可复现证据、与本次修改的关系、未覆盖风险；开发明确接受风险并写入 `decisions.md` 后才继续交付，但**不得**写成"全部验证通过" |
| 必要验证全部完成，人工验证结论已由开发确认 | `complete` | 无 | 可进入 `delivery` |

---

## `blockingItems`：阻塞项

### 格式

```json
{
  "code": "api_conflict",
  "scope": "admin-app/api/user",
  "createdAt": "2026-08-13T14:00:00+08:00",
  "details": "实际响应字段与已确认契约不一致"
}
```

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `code` | string | 必须是下表 7 个允许值之一，不得自造 |
| `scope` | string | 受影响的项目、模块、文件或接口标识 |
| `createdAt` | string | 带时区 ISO 8601，用于判断阻塞已挂多久 |
| `details` | string | 一句话说明为什么阻塞，不含凭证 |

### 允许的 7 个 code

| `code` | 触发条件 | 典型 `scope` 粒度 |
| --- | --- | --- |
| `missing_requirement` | 既无需求文档也无需求描述 | 项目级或 `context` |
| `missing_ui` | 本次任务依赖 UI，但没有 UI 资料，或 UI 缺关键状态 | 页面级或任务级 |
| `missing_api_contract` | 需要对接真实接口，但没有后端确认的契约 | 接口级 |
| `p0_unresolved` | 存在未解决的 P0 问题 | 问题 ID 级，如 `PRD-01` |
| `api_conflict` | 真实接口响应结构或语义与已确认契约不一致 | 接口级 |
| `inaccessible_project` | `projectPaths` 中某项不存在或无读取权限 | 项目级 |
| `verification_failed` | 本次修改导致必要验证失败 | 模块级或文件级 |

不在这 7 个之内的情况，不要发明新 code。改为暂停并把问题写入 `context.md` 的开放问题，等开发回答。

### `scope` 粒度约定

`scope` 决定停多少。粒度从粗到细：

| 粒度 | 写法示例 | 停的范围 |
| --- | --- | --- |
| 项目级 | `/workspace/my-project/admin-app` 或 `admin-app` | 该项目全部任务 + 依赖该项目产出的跨项目任务 |
| 模块级 | `client-app/user-profile` | 该模块的页面、组件、样式、测试 + 下游依赖 |
| 文件级 | `client-app/src/router/index.js` | 该文件相关改动 + 依赖该改动的任务 |
| 接口级 | `admin-app/api/user` | 该接口的对接、联调、相关测试 + 依赖该接口数据的页面逻辑 |

**"只暂停该范围及其下游依赖"的含义：**

假设 `plan.md` 的技术任务依赖是：

```text
头像上传组件 → 用户资料页面 → 路由配置
接口 A（资料读取）→ 用户资料页面
埋点接入（独立）
```

`missing_api_contract` 的 `scope` 是 `client-app/api/profile-read`（接口 A）时：

- 停：接口 A 的对接、联调、相关测试
- 停：用户资料页面中**依赖接口 A 数据**的部分（真实数据渲染、失败态联调）
- 不停：头像上传组件（不依赖接口 A）
- 不停：用户资料页面的骨架、样式还原、静态状态（可用已批准的 Mock）
- 不停：埋点接入（完全独立）
- 全局 `status`：`active`

判断下游的依据只有一个：`plan.md` 的"技术任务及依赖"章节。不靠直觉推断依赖关系。如果计划里没写清依赖，先补计划，不要凭感觉决定停哪些任务。

选择粒度的原则：**能细就细。** 一个接口有问题就写接口级，不要图省事写项目级把整个项目停掉。

### 阻塞项的生命周期

- **新增**：发现阻塞条件的那一刻立即追加，同时更新 `stageStatus` 和 `status`。
- **不合并**：两个不同接口都缺契约时，写两条 `missing_api_contract`，各自带自己的 `scope`。合并成一条项目级会误停无关任务。
- **移除**：阻塞条件解除后移除对应条目，然后重算 `status`。移除是精确移除，不清空整个数组。
- **不覆盖**：同一 `scope` 上出现新问题时，如果是不同 `code`，两条并存。

---

## `invalidatedItems`：失效项

### 格式

```json
{
  "stage": "implementation",
  "scope": "admin-app/user-profile",
  "reason": "requirement_change",
  "changeId": "CR-001",
  "createdAt": "2026-08-13T15:00:00+08:00"
}
```

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `stage` | string | 失效发生在哪个阶段：`context` / `planning` / `implementation` / `verification` / `delivery` |
| `scope` | string | 失效范围，粒度约定与 `blockingItems.scope` 相同 |
| `reason` | string | 失效原因，常见 `requirement_change`、`ui_change`、`api_contract_change` |
| `changeId` | string | 关联的变更记录编号，如 `CR-001`、`UCR-002`、`TECH-001` |
| `createdAt` | string | 带时区 ISO 8601 |

`changeId` 必须能在 `decisions.md` 中查到对应记录。没有变更记录就不该有失效项——先记变更，再标失效。

### 何时用 `invalidatedItems`，何时用 `stageStatus: "invalidated"`

| 情形 | 处理 |
| --- | --- |
| 阶段内**部分**已完成内容失效 | 只追加 `invalidatedItems`；`stageStatus` 保持原值；未受影响任务继续 |
| 阶段内**全部**已完成或已批准内容失效 | 追加 `invalidatedItems` 之外，将该 `stageStatus` 设为 `invalidated` |

`stageStatus: "invalidated"` 是粗粒度信号，只在整个阶段的结论都不能用了的时候使用。绝大多数需求变更是局部的，只需要 `invalidatedItems`。

**示例：变更只影响管理端的资料模块**

```json
{
  "currentStage": "implementation",
  "status": "active",
  "stageStatus": {
    "context": "complete",
    "planning": "invalidated",
    "implementation": "active",
    "verification": "pending",
    "delivery": "pending"
  },
  "invalidatedItems": [
    { "stage": "implementation", "scope": "admin-app/user-profile", "reason": "requirement_change", "changeId": "CR-001", "createdAt": "2026-08-13T15:00:00+08:00" }
  ],
  "approvedPlanVersion": null,
  "approvedAt": null,
  "approvedByRole": null
}
```

`planning` 是 `invalidated`：批准被清空，计划需要重出。`implementation` 仍是 `active`：客户端的实现没受影响，可以继续；但 `admin-app/user-profile` 那部分停在失效状态等新计划。

---

## `blocked` 与 `invalidated` 的区别

这两个概念经常被混用，但含义完全不同。

| 维度 | `blocked` | `invalidated` |
| --- | --- | --- |
| 本质 | **当前无法继续** | **已有结论需要重做** |
| 时间指向 | 指向未来：缺东西，做不下去 | 指向过去：做过的东西不能用了 |
| 已有产出 | 没有产出，或产出仍然有效 | 产出存在但已过期 |
| 触发者 | 缺输入、有冲突、验证失败 | 需求变更、UI 变更、契约变更 |
| 记录位置 | `blockingItems` | `invalidatedItems` |
| 解除方式 | 补齐缺失输入或解决冲突 | 更新对应产物并重新批准 |
| 对全局 `status` 的影响 | 全部范围都被阻塞时 → `blocked` | 不直接产生 `blocked`；重做期间通常是 `active` 或 `awaiting_confirmation` |

**辨析示例：**

| 事实 | 分类 | 理由 |
| --- | --- | --- |
| 资料页面需要 UI 图，但没有 UI 目录 | `blocked` / `missing_ui` | 缺输入，一行都写不了 |
| 资料页面已按旧 UI 实现完，UI 改版了 | `invalidated` | 代码存在但要按新 UI 重做 |
| 接口契约没确认，不能对接真实接口 | `blocked` / `missing_api_contract` | 缺输入 |
| 接口已对接完，产品改了字段长度上限 | `invalidated` | 已有实现需要按新规则调整 |
| 真实响应字段和已确认契约不一致 | `blocked` / `api_conflict` | 存在冲突，需要人来裁决用哪个 |
| 昵称长度从 20 改成 40，表单校验已写好 | `invalidated` | 校验逻辑要改 |
| 本次改动导致单测失败，原因不明 | `blocked` / `verification_failed` | 当前推进不了交付 |

**同一件事可能同时产生两者。** 需求变更导致已有实现失效（`invalidated`），而新需求又缺 UI（`blocked`）。这时两个数组各记一条，不要试图用一条记录表达两件事。

### `invalidated` 的恢复路径

失效项不由开发手动清理。恢复流程：

```text
更新对应产物（context.md 或 plan.md）
→ 设置新的 planVersion，使 planBasedOnRequirementVersion 与 requirementVersion 一致
→ 开发重新批准
→ Skill 自动移除已恢复的 invalidatedItems 条目
→ Skill 自动把因失效而改动的 stageStatus 重置回合理值
→ 重算全局 status
```

**开发不需要、也不应该手动修改 `workflow-state.json`。** 开发要做的只有两件事：回答问题、批准计划。状态文件的一致性由 Skill 负责。

范围重做完成后，移除对应 `invalidatedItems` 条目，并在 `decisions.md`（变更结论）或 `delivery.md`（重做后的验证证据）保留追溯证据。移除记录不等于抹掉历史——历史在 `decisions.md` 里，那是追加式的。

---

## 版本三字段的关系

三个版本字段共同回答一个问题：**当前计划是否基于当前需求，并且得到了有效批准？**

```text
requirementVersion              当前生效的需求基线
        ↓ 计划基于哪个需求版本
planBasedOnRequirementVersion   计划的需求依据
        ↓ 计划本身的版本
planVersion                     当前计划版本
        ↓ 批准的是哪个计划版本
approvedPlanVersion             有效批准
```

### 两个必须成立的等式

| 等式 | 不成立的含义 | 后果 |
| --- | --- | --- |
| `planBasedOnRequirementVersion == requirementVersion` | 需求变了但计划还没跟上 | 不得实现受影响部分 |
| `approvedPlanVersion == planVersion` | 计划改过但没重新批准 | 批准无效，不得改业务代码 |

`approvedPlanVersion` 为 `null` 也属于第二个等式不成立。

### 组合判断表

| `planBasedOn...` vs `requirementVersion` | `approvedPlanVersion` vs `planVersion` | 可以改业务代码？ | 该做什么 |
| --- | --- | --- | --- |
| 相等 | 相等 | 是 | 按计划实现 |
| 相等 | 不等或为 `null` | 否 | 请求批准（门禁二） |
| 不等 | 相等 | 否 | 需求变过：更新 `plan.md`、更新 `planVersion`、清空批准、重新批准 |
| 不等 | 不等或为 `null` | 否 | 先更新计划再走门禁二 |

### `planVersion` 什么时候更新

| `plan.md` 的变化 | 更新 `planVersion`？ |
| --- | --- |
| 新增或删除文件清单条目 | 是 |
| 修改技术任务或任务依赖顺序 | 是 |
| 修改 API 对接清单 | 是 |
| 修改验证方案的命令或通过标准 | 是 |
| 修改"不允许修改的范围" | 是 |
| 修正错字、调整排版、补充说明性文字 | 否 |
| `context.md` 的估时附录变化 | 否（估时不属于可执行计划） |

判断标准：**这次改动会让实现或验证的实际动作不同吗？** 会 → 更新版本并重新批准。不会 → 不更新。

---

## `executionMode` 与 `targetStage` 的约束

### 恢复时不得越过 `targetStage`

每次恢复调用，在决定继续执行哪个阶段之前必须检查：

```text
候选下一阶段在五阶段序列中的位置 > targetStage 的位置？
    是 → 停止，向开发报告已到执行终点，询问是否升级
    否 → 继续执行
```

**这个检查在阶段推进前执行，不是在阶段完成后。** 不允许"先做了再说"。

示例：`executionMode: "planning_only"`、`targetStage: "planning"`，计划已生成并被批准。此时正确行为是停下并报告：

```text
计划已批准（版本 2026-08-13T10:00:00+08:00）。
当前执行模式为 planning_only，执行终点为 planning 阶段，不继续修改代码。

如需继续实现，请调用：
  继续 frontend-delivery，升级为完整交付。
  需求标识：feature-001
```

错误行为是批准后自动进入实现阶段。开发选了 `planning_only` 就是明确不要代码改动，批准计划不等于授权越过执行终点。

### 升级与降级

- **升级**（如 `review_only` → `full_delivery`）：开发显式要求时更新 `executionMode` 和 `targetStage`，然后从当前 `currentStage` 之后继续。已完成阶段不重做。
- **降级**：一般不需要。开发只想停下时直接结束会话即可，已有状态保留。如果开发显式要求降级，更新两个字段，但**不回滚**已完成的工作，也不删除已有产物。

### 到达终点后的 `status`

`targetStage` 对应阶段变为 `complete` 时，全局 `status` 为 `complete`，其后阶段保持 `pending`。这是正常终态，不是"未完成"。

---

## 多项目文件归属规则

多项目场景下，一个文件属于哪个项目，只由**绝对路径的层级深度**决定。

### 规则

```text
1. 取文件的规范化绝对路径
2. 找出所有「是该文件路径前缀」的 projectPaths 条目
3. 在这些候选中，选择路径层级最深的那一个
4. 候选为空 → 暂停询问开发
5. 层级最深的候选有多个（路径完全相同）→ 暂停询问开发
```

**不按 `projectPaths` 的声明顺序决定。** 声明顺序没有任何语义。

**`primaryProjectPath` 不参与归属判断。** 它只影响默认 `outputDir` 和项目内文件建议的落点。

### 示例

```json
{
  "primaryProjectPath": "/workspace/my-project/client-app",
  "projectPaths": [
    "/workspace/my-project/client-app",
    "/workspace/my-project/client-app/packages/shared-ui",
    "/workspace/my-project/admin-app"
  ]
}
```

| 文件 | 归属 | 理由 |
| --- | --- | --- |
| `/workspace/my-project/client-app/src/pages/profile.vue` | `client-app` | 唯一匹配 |
| `/workspace/my-project/client-app/packages/shared-ui/src/button.vue` | `client-app/packages/shared-ui` | 两个都是前缀，选层级更深的 |
| `/workspace/my-project/admin-app/src/api/user.js` | `admin-app` | 唯一匹配 |
| `/workspace/my-project/tools/build.js` | 无 | 不在任何项目下 → 暂停询问开发 |

第二行是关键：即使 `client-app` 在数组里排在前面，`shared-ui` 层级更深，文件归 `shared-ui`。

### 必须暂停询问开发的情形

- 文件不属于任何 `projectPaths` 条目
- 项目根目录重叠后，按层级深度仍无法唯一确定
- 项目之间存在循环依赖

### 相关约定

- 每个项目独立分析技术栈、规范、构建命令和验证条件。不假设多个项目用同一套工具链。
- `context.md` 和 `plan.md` 中的文件、任务、接口、验证命令必须标注所属项目。
- 公共代码位于独立目录或独立仓库时，将其作为单独项目加入 `projectPaths`。这样它的规范和构建命令才会被独立分析，归属判断也才准确。
- 跨项目跳转、参数传递、接口契约、发布顺序、联调依赖写入 `plan.md` 的"技术任务及依赖"。

---

## 凭证禁止写入

`workflow-state.json` **禁止**保存以下内容：

- 账号、用户名、密码
- Token、API Key、Secret、证书
- 验证码、短信码、一次性口令
- 数据库连接串、含凭证的 URL
- Cookie、Session ID

这条限制同样适用于 `context.md`、`plan.md`、`delivery.md`、`decisions.md`。

需要测试账号或环境凭证时，在 `plan.md` 的验证方案里写**需要什么类型的账号和权限**，由开发在运行时自行提供。

✅ 正确：`需要一个已完成实名认证的普通用户账号，和一个具备用户管理权限的管理端账号（由开发提供）`

❌ 错误：把具体账号密码写进任何产物文件

---

## 常见误判纠正

### 误判 1：有 `blockingItems` 就把全局 `status` 设为 `blocked`

❌ **错误：** 一个接口缺契约，就把 `status` 设成 `blocked`。

✅ **正确：** 只要还有其他可执行任务，`status` 就是 `active`。`blocked` 的条件是**全部**剩余工作都推进不了。范围级阻塞的意义就在于不误停无关工作。

### 误判 2：把"计划待批准"记成阻塞项

❌ **错误：** `blockingItems: [{ "code": "plan_not_approved", ... }]`

✅ **正确：** 计划未批准是 `awaiting_confirmation`，不是阻塞，不进 `blockingItems`。而且 `plan_not_approved` 不在允许的 7 个 code 里。

### 误判 3：等批准时让 `implementation` 变成 `awaiting_confirmation`

❌ **错误：** `planning: "complete"` + `implementation: "awaiting_confirmation"`

✅ **正确：** `planning: "awaiting_confirmation"` + `implementation: "pending"`。等待的对象是计划，责任在 planning。

### 误判 4：需求变更时把所有阶段标成 `invalidated`

❌ **错误：** 一个 `CR-001` 就把 `planning`、`implementation`、`verification` 全设为 `invalidated`。

✅ **正确：** 先做影响分析。只有某阶段的全部已有结论都失效才设 `invalidated`；局部失效只追加 `invalidatedItems`，`stageStatus` 保持原值。变更几乎总是局部的。

### 误判 5：自动验证通过就把 `verification` 标 `complete`

❌ **错误：** lint、单测、构建都过了，`verification: "complete"`，进入交付。

✅ **正确：** 只要计划里的人工验证项（真机、视觉对比、权限账号、跨端跳转）还没执行并被开发确认，`verification` 就是 `awaiting_confirmation`，`delivery` 保持 `pending`。

### 误判 6：`approvedPlanVersion` 有值就认为批准有效

❌ **错误：** 看到 `approvedPlanVersion` 不是 `null` 就开始改代码。

✅ **正确：** 必须核对 `approvedPlanVersion == planVersion`。计划改过而没重批时，旧的批准值还留在字段里，但已经无效。同时还要核对 `planBasedOnRequirementVersion == requirementVersion`。

### 误判 7：按 `projectPaths` 顺序决定文件归属

❌ **错误：** 数组第一项是 `client-app`，就把 `client-app/packages/shared-ui/` 下的文件归给 `client-app`。

✅ **正确：** 选路径层级最深的匹配项。声明顺序无语义。

### 误判 8：为新情况自造 `blockingItems` 的 code

❌ **错误：** 遇到"开发本地有未提交修改与本次改动冲突"，写 `code: "uncommitted_conflict"`。

✅ **正确：** 只有 7 个允许的 code。不在其中的情况，暂停并把事实、影响、建议选项写进 `context.md` 的开放问题，询问开发。

### 误判 9：开发手动改状态文件来解除失效

❌ **错误：** 提示开发"把 `invalidatedItems` 里那条删掉就能继续了"。

✅ **正确：** 开发只回答问题和批准计划。更新产物 → 重新批准 → Skill 自动清理失效项并重置状态。

---

## 总结

**三层状态：**

1. `status`（全局）— 派生的，不手写
2. `stageStatus`（阶段）— 五个阶段各自的进展
3. `blockingItems` / `invalidatedItems`（范围）— 精确到项目、模块、文件、接口

**判断技巧：**

- 要写 `status` 时，按顺序走完这四问，第一个成立的就是答案：
  1. 终点已达？→ `complete`
  2. 还有一件事能立刻做？→ `active`
  3. 没有可执行任务，且 `blockingItems` 或 `invalidatedItems` 非空？→ `blocked`
  4. 没有可执行任务，且两个数组都为空，只等门禁确认？→ `awaiting_confirmation`

  别用"在等人"当判据——等产品解决 P0 也是等人，但那时 `blockingItems` 非空，应该是 `blocked`。判据只有数组是否为空。
- 要分 `blocked` 还是 `invalidated` 时，先问：**"是做不下去，还是做过的不能用了？"** 做不下去 → `blocked`。做过的不能用 → `invalidated`。
- 要定 `scope` 粒度时，先问：**"最小需要停哪些任务？"** 依据只有 `plan.md` 的任务依赖，不靠直觉。
- 要判断能不能改代码时，先问：**"两个版本等式都成立吗？"** `planBasedOnRequirementVersion == requirementVersion` 且 `approvedPlanVersion == planVersion`，两个都成立才能动业务代码。
