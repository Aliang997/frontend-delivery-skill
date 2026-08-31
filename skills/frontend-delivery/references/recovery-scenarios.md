# 失败与恢复场景

本文档定义九个典型失败场景的触发条件、期望状态和恢复步骤，以及每次调用共用的恢复入口逻辑。

状态字段的语义和取值范围见 `state-machine.md`，本文档只描述具体场景下字段应该变成什么。两者冲突时以 `state-machine.md` 为准。

---

## 目录

- [通用恢复入口逻辑](#通用恢复入口逻辑)
- [场景一：需求文档或需求描述缺失](#场景一需求文档或需求描述缺失)
- [场景二：产品问题未确认（P0）](#场景二产品问题未确认p0)
- [场景三：计划未批准时会话中断](#场景三计划未批准时会话中断)
- [场景四：实现时接口与文档不符](#场景四实现时接口与文档不符)
- [场景五：多项目中的一个项目不可访问](#场景五多项目中的一个项目不可访问)
- [场景六：验证失败](#场景六验证失败)
- [场景七：依赖 UI 但没有 UI 资料](#场景七依赖-ui-但没有-ui-资料)
- [场景八：缺少接口契约](#场景八缺少接口契约)
- [场景九：实现过程中收到需求变更](#场景九实现过程中收到需求变更)
- [场景对照速查](#场景对照速查)
- [常见误判纠正](#常见误判纠正)
- [总结](#总结)

三级标题见正文。

---

## 通用恢复入口逻辑

每次调用 frontend-delivery（无论首次还是恢复）都走同一套入口判断。

### 1. 定位状态文件

```text
开发给了需求标识
→ 在候选 outputDir 下查找 <requirementId>/workflow-state.json
→ 找到：读取，进入步骤 2
→ 没找到：按首次启动处理，从阶段一开始
```

候选 `outputDir` 的查找顺序：开发本次显式指定的产物目录 → 单项目默认位置 `<primaryProjectPath>/docs/ai-delivery/<requirementId>/` → 多项目默认位置 `<workspaceRoot>/docs/ai-delivery/<requirementId>/`。

开发未给需求标识但明确说"继续"时，在默认位置下列出已有需求标识让开发选择，不猜。

### 2. 校验状态文件一致性

读到状态文件后，先做四项校验：

| 校验 | 不通过时的处理 |
| --- | --- |
| 五个 `stageStatus` 键齐全，取值都在允许范围内 | 报告字段异常，询问开发是否按当前产物重建状态 |
| 三个批准字段（`approvedPlanVersion` / `approvedAt` / `approvedByRole`）同时有值或同时为 `null` | 视为损坏，清空三者并重新走门禁二 |
| 每个 `invalidatedItems.changeId` 能在 `decisions.md` 中找到对应记录 | 报告失效项缺少变更记录，询问开发 |
| `blockingItems` 的 `code` 都在允许的 7 个之内 | 报告非法 code，询问开发该阻塞的真实原因 |

校验通过后重算全局 `status`（按 `state-machine.md` 的派生规则），不直接信任文件里存的旧值。

### 3. 检查 `targetStage` 边界

```text
候选下一阶段在五阶段序列中的位置 > targetStage 的位置？
    是 → 停止推进，报告已到执行终点，给出升级调用方式
    否 → 进入步骤 4
```

这个检查在**推进阶段之前**执行。已到终点时不允许"先做一点再说"。

五阶段序列：`context` → `planning` → `implementation` → `verification` → `delivery`。

### 4. 校验版本一致性

进入实现或验证阶段前必须核对两个等式：

| 等式 | 不成立时 |
| --- | --- |
| `planBasedOnRequirementVersion == requirementVersion` | 需求变过而计划未跟上：不实现受影响范围，先更新 `plan.md` |
| `approvedPlanVersion == planVersion` | 批准无效：不改业务代码，请求重新批准 |

### 5. 决定从哪里继续

按顺序检查，第一个匹配的即为入口：

| 检查项 | 入口动作 |
| --- | --- |
| `targetStage` 对应阶段为 `complete` | 报告已完成，展示产物路径，不做任何改动 |
| 存在 `stageStatus == "invalidated"` 的阶段 | 从该阶段重做：更新对应产物，走对应门禁 |
| 存在 `invalidatedItems` 未清理 | 先处理失效范围的重做，同时可推进不受影响任务 |
| 存在 `stageStatus == "blocked"` 的阶段 | 报告 `blockingItems` 内容，请求缺失输入；同时检查是否有未被 scope 覆盖的可执行任务 |
| 存在 `stageStatus == "awaiting_confirmation"` 的阶段 | 报告等待什么确认，展示待确认产物，不推进 |
| 存在 `stageStatus == "active"` 的阶段 | 从该阶段继续执行 |
| 全部前置阶段 `complete`，下一阶段 `pending` | 推进到下一阶段（先过步骤 3 的边界检查） |

### 6. 报告后再动手

无论从哪里继续，先向开发报告三件事：当前处在哪个阶段、上次停下的原因、这次准备做什么。然后才开始执行。

不要静默继续。开发可能已经忘了上次停在哪，也可能中间补充了资料而没告知。

---

## 场景一：需求文档或需求描述缺失

### 触发条件

启动时既没有需求文档路径，也没有任何需求描述。开发只给了项目路径。

注意区分：开发给了简短但可理解的口头描述（如"资料页加个头像上传"）**不属于**本场景，那是需求信息不完整，走正常的需求评审补齐问题。本场景是完全没有需求内容。

### 期望行为

创建 `workflow-state.json`，并在 `context.md` 列出缺失清单。

```json
{
  "currentStage": "context",
  "status": "blocked",
  "stageStatus": {
    "context": "blocked",
    "planning": "pending",
    "implementation": "pending",
    "verification": "pending",
    "delivery": "pending"
  },
  "blockingItems": [
    {
      "code": "missing_requirement",
      "scope": "context",
      "createdAt": "2026-08-13T08:00:00+08:00",
      "details": "未提供需求文档路径，也未提供需求描述"
    }
  ],
  "invalidatedItems": [],
  "requirementVersion": null,
  "planVersion": null,
  "approvedPlanVersion": null
}
```

全局 `status` 为 `blocked`。理由：没有需求就无法判断哪些代码相关、要分析什么、要复用什么。项目分析在这个阶段无法有针对性地进行——核心原则要求"只在与需求有关时检查，不机械扫描全部业务"，所以不存在可执行任务，命中派生规则 4。

`context.md` 至少写清：

```markdown
## 开放问题

| ID | 优先级 | 问题 | 确认原因 | 状态 |
| --- | --- | --- | --- | --- |
| REQ-01 | P0 | 本次需求的业务目标是什么？ | 缺少需求内容无法开始分析 | 待确认 |

## 缺失输入

- 需求文档路径或需求描述（必需）
- UI 目录（依赖 UI 时必需，可后补）
- 接口文档（对接真实接口前必需，可后补）
```

### 恢复步骤

1. 开发补充需求文档路径或直接描述需求。
2. 写入 `requirementDocPath`（如果是文档），设置 `requirementVersion`。
3. 移除 `missing_requirement` 阻塞项。
4. `stageStatus.context` 设为 `active`，重算 `status` 为 `active`。
5. 从阶段一开头继续：分析项目、理解需求、复用分析、生成分级评审清单。

---

## 场景二：产品问题未确认（P0）

### 触发条件

`context.md` 已生成，存在一个或多个未解决的 P0 问题。

### 期望行为

上下文分析本身已完成，但门禁一不通过。

```json
{
  "currentStage": "context",
  "status": "blocked",
  "stageStatus": {
    "context": "blocked",
    "planning": "pending",
    "implementation": "pending",
    "verification": "pending",
    "delivery": "pending"
  },
  "blockingItems": [
    {
      "code": "p0_unresolved",
      "scope": "PRD-01",
      "createdAt": "2026-08-13T09:30:00+08:00",
      "details": "未登录用户能否查看资料页，决定权限边界与数据暴露范围"
    },
    {
      "code": "p0_unresolved",
      "scope": "PRD-03",
      "createdAt": "2026-08-13T09:30:00+08:00",
      "details": "删除操作是否可撤销，决定数据删除策略"
    }
  ]
}
```

`scope` 是**具体问题 ID**，不是模块名也不是项目名。每个未解决 P0 一条记录，不合并。这样产品逐个回复时可以逐条移除，恢复逻辑能准确知道还剩哪些没答。

全局 `status` 为 `blocked`。理由：P0 的定义是"不确认就无法安全实现"，未解决 P0 时不能制定最终计划，而阶段一的其余工作（项目分析、复用分析）已经完成。没有可执行任务，命中派生规则 4。

`context.md` 的评审结论必须标明不可进入最终计划：

```markdown
## 需求评审结论
- 评审状态：待确认
- P0 未解决：2
- P1 未解决：1
- 是否允许进入最终技术计划：否
```

同时生成可直接转发给产品的确认清单，含问题编号、优先级、确认原因、影响范围、建议选项。

### 恢复步骤

1. 开发转述产品回复。
2. 更新 `context.md` 对应问题的状态列，从"待确认"改为已确认结论。
3. 追加 `DEC-*` 记录到 `decisions.md`，含关联问题 ID、决策、确认来源、影响、替代决策。
4. 移除该问题 ID 对应的 `p0_unresolved` 条目（精确移除，不清空数组）。
5. 重新判断门禁一：
   - 仍有 P0 未解决 → `stageStatus.context` 保持 `blocked`，报告剩余问题
   - 全部 P0 已解决 → `context` 设为 `complete`，`planning` 设为 `active`，进入阶段二
6. 产品回复与原需求冲突时，按需求变更处理（见 `change-control.md`），不是简单更新问题状态。

开发要求"P0 都按默认方案继续"时不接受。说明每个 P0 的具体风险，保留阻塞项。如果开发本人对该问题有决策权限，请其以决策者身份明确回答，按步骤 2-5 记录并注明 `approvedByRole` 或确认角色为开发。

---

## 场景三：计划未批准时会话中断

### 触发条件

`plan.md` 已生成，P1 已全部解决，但开发尚未批准。会话在此中断。

### 期望行为

```json
{
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
  "planVersion": "2026-08-13T10:00:00+08:00",
  "planBasedOnRequirementVersion": "2026-08-13T09:00:00+08:00",
  "approvedPlanVersion": null,
  "approvedAt": null,
  "approvedByRole": null
}
```

四个关键点：

1. `currentStage` 保持 `planning`，不前移。
2. `stageStatus.planning` 是 `awaiting_confirmation`，`stageStatus.implementation` 保持 `pending`。**不是** `planning: complete` + `implementation: awaiting_confirmation`。等待的对象是计划，责任在 planning 阶段；实现阶段一步都没走，不该有 `pending` 之外的状态。把等待挂到 `implementation` 会让恢复逻辑误判"实现已开始"，进而可能跳过门禁二。
3. `blockingItems` 为空。计划未批准属于 `awaiting_confirmation`，不是错误、不是阻塞、不进 `blockingItems`，也不存在对应的合法 code。
4. 三个批准字段全为 `null`。

全局 `status` 为 `awaiting_confirmation`。理由：没有阻塞项，也没有可以不经批准就执行的任务（计划未批准不得改业务代码），唯一的下一步是等开发批准，命中派生规则 3。

不得修改任何业务代码。

### 恢复步骤

1. 读取状态文件，检测到 `stageStatus.planning == "awaiting_confirmation"`。
2. 核对 `planBasedOnRequirementVersion == requirementVersion`。不等说明中断期间需求变过，转场景九处理。
3. 向开发展示计划摘要（创建文件数、修改文件数、对接接口数）和 `plan.md` 路径，请求批准。
4. 开发批准 → 设置 `approvedPlanVersion` = 当前 `planVersion`、`approvedAt` = 当前时间、`approvedByRole`；追加 `DEV-*` 记录；`planning` 转 `complete`、`implementation` 转 `active`。
5. 开发要求修改计划 → 改 `plan.md`、更新 `planVersion`、追加 `DEV-*` 记录（"替代决策"引用原计划内容）、重新请求批准。此时状态仍是 `planning: awaiting_confirmation`。
6. 批准后仍要过 `targetStage` 边界检查。`executionMode` 为 `planning_only` 时，批准即到终点，不进入实现。

---

## 场景四：实现时接口与文档不符

### 触发条件

实现阶段对接真实接口时，发现实际响应结构或语义与已确认的接口契约不一致。例如文档写 `data.list` 实际返回 `data.records`，或状态码含义与文档描述不同。

### 期望行为

停止受影响的接口任务，其余任务继续。

```json
{
  "currentStage": "implementation",
  "status": "active",
  "stageStatus": {
    "context": "complete",
    "planning": "complete",
    "implementation": "active",
    "verification": "pending",
    "delivery": "pending"
  },
  "blockingItems": [
    {
      "code": "api_conflict",
      "scope": "admin-app/api/user",
      "createdAt": "2026-08-13T14:00:00+08:00",
      "details": "列表接口实际响应字段为 data.records，已确认契约为 data.list"
    }
  ]
}
```

`scope` 是接口级：项目名 + 接口标识。不写成整个项目，也不写成整个模块。

全局 `status` 保持 `active`，`stageStatus.implementation` 保持 `active`。理由：只有依赖该接口的任务停了，其他任务（不相关的页面、组件、埋点、其他接口）仍可执行，命中派生规则 2。

停的范围按 `plan.md` 的"技术任务及依赖"确定：该接口的对接和联调、依赖该接口数据的页面渲染逻辑和失败态处理、相关测试。不依赖该接口的任务不停。

将未确认的冲突写入 `context.md` 的开放问题，含三部分：

```markdown
## 开放问题

| ID | 优先级 | 问题 | 事实 | 影响范围 | 建议选项 | 状态 |
| --- | --- | --- | --- | --- | --- | --- |
| API-01 | P0 | 用户列表接口响应字段以文档还是实际为准？ | 文档 `data.list`，实际返回 `data.records` | 管理端列表页数据渲染、分页、相关测试 | A 后端改为 data.list / B 更新文档并按 data.records 实现 | 待确认 |

## 缺失输入

- API-01 的裁决结论（后端或开发确认）
```

**不允许自行选择一种写法继续实现。** 猜错会导致联调返工，也可能掩盖后端的真实 Bug。

### 恢复步骤

1. 开发或后端给出裁决。
2. 追加 `DEC-*` 记录到 `decisions.md`，写明最终契约。
3. 更新 `plan.md` 的 API 对接清单。
4. 判断计划的可执行内容是否变化：
   - 变化（字段映射改、新增转换层、错误码处理变化）→ 更新 `planVersion`、清空三个批准字段、重新请求批准
   - 未变化（仅确认文档写错，实现方式与原计划一致）→ 不更新 `planVersion`，不重批
5. 移除 `api_conflict` 阻塞项。
6. 继续该接口任务及其下游任务。
7. 联调完成后，重跑受影响接口的相关测试。

---

## 场景五：多项目中的一个项目不可访问

### 触发条件

`projectPaths` 中某个路径不存在、没有读取权限，或指向的目录明显不是一个前端项目（无任何依赖清单和源码）。

### 期望行为

```json
{
  "currentStage": "context",
  "status": "active",
  "stageStatus": {
    "context": "active",
    "planning": "pending",
    "implementation": "pending",
    "verification": "pending",
    "delivery": "pending"
  },
  "blockingItems": [
    {
      "code": "inaccessible_project",
      "scope": "/workspace/my-project/admin-app",
      "createdAt": "2026-08-13T09:05:00+08:00",
      "details": "路径不存在或无读取权限，未做任何实现假设"
    }
  ]
}
```

`scope` 是项目级，写规范化绝对路径。

全局 `status` 保持 `active`。理由：其他项目的分析和后续实现仍可进行，**其他无依赖任务可以继续**，命中派生规则 2。只有当全部 `projectPaths` 都不可访问时，才变成 `blocked`。

关键约束：**不假设该项目的实现。** 不猜它用什么框架、什么目录结构、有什么可复用组件。`context.md` 中该项目的画像章节标注"待补齐"，`plan.md` 中涉及该项目的文件清单和任务留空并标注原因。

跨项目依赖的处理：如果 `plan.md` 的技术任务里存在"客户端跳转依赖管理端的路由参数约定"这类跨项目依赖，那么**依赖该项目的下游任务也一并暂停**。判断依据是 `plan.md` 的"技术任务及依赖"章节，不靠直觉。

### 恢复步骤

1. 开发补齐路径或授予读取权限。
2. 校验路径可访问。
3. **只重新分析受影响项目**：技术栈、规范、依赖版本、包管理器、构建命令、验证条件、与需求相关的页面组件接口。已完成的其他项目分析不重做。
4. 补齐 `context.md` 中该项目的画像章节和跨项目复用分析。
5. 移除 `inaccessible_project` 阻塞项。
6. 该项目的分析结果如果影响已批准的计划（新增文件、改变任务依赖、发现可复用实现），更新 `plan.md` 和 `planVersion`，重新批准。
7. 恢复该项目及其下游任务。

如果开发明确表示该项目本次不参与，从 `projectPaths` 移除该条目，在 `decisions.md` 追加 `DEV-*` 记录，然后移除阻塞项。这属于范围调整，需要复核 `plan.md` 中是否有依赖该项目的任务。

---

## 场景六：验证失败

### 触发条件

验证阶段执行 lint、类型检查、单测、组件测试、E2E 或构建时出现失败。

### 期望行为

先区分失败来源，两种来源处理方式完全不同。

**情形 A：本次修改导致的失败**

```json
{
  "currentStage": "verification",
  "status": "active",
  "stageStatus": {
    "context": "complete",
    "planning": "complete",
    "implementation": "complete",
    "verification": "active",
    "delivery": "pending"
  },
  "blockingItems": [
    {
      "code": "verification_failed",
      "scope": "client-app/user-profile",
      "createdAt": "2026-08-13T16:00:00+08:00",
      "details": "资料页组件测试 3 项失败，头像上传大小校验分支未覆盖预期错误提示"
    }
  ]
}
```

`scope` 是模块级或文件级。全局 `status` 视情况：还能修的时候是 `active`（修复本身就是可执行任务）；确认修不动、必须等开发决策时，`stageStatus.verification` 转 `blocked`，全局也转 `blocked`。

必要验证未通过前，`verification` 和 `delivery` 都**不得**标 `complete`。

**情形 B：既有问题导致的失败**

不追加 `verification_failed` 阻塞项，也不因此改变阶段状态。但必须在 `delivery.md` 记录：

```markdown
### 既有问题（非本次引入）

**命令：** <项目实际的测试命令>
**失败数：** 2
**证据：** <失败摘要>
**复现方式：** 在本次改动前的提交上执行同一命令，同样失败
**与本次修改的关系：** 失败用例位于订单模块，与本次资料页改动无文件交集
**未覆盖风险：** 该模块的回归能力缺失，本次无法通过该套测试验证订单相关影响
**开发是否接受该风险：** 待确认
```

开发明确接受该风险并写入 `decisions.md` 后才能继续交付。但**不得**在任何地方写成"全部验证通过"。

`delivery.md` 必须记录：执行的准确命令、结果、失败数量、未执行项及原因、人工验证项状态。

### 恢复步骤

**情形 A：**

1. 定位失败根因，修复本次引入的问题。
2. 修复方案改变了已批准计划时（需要新增文件、改变接口对接方式、扩大修改范围），先更新 `plan.md` 和 `planVersion`，重新批准，再改代码。
3. **重新执行对应的完整验证**，不是只跑失败的那几个用例。局部重跑可能掩盖修复引入的新问题。
4. 全部通过后移除 `verification_failed` 阻塞项。
5. 检查人工验证项是否已执行：
   - 未执行 → `verification` 设为 `awaiting_confirmation`，在 `delivery.md` 列出待人工执行清单
   - 已执行并由开发确认 → `verification` 设为 `complete`
6. 更新 `delivery.md`，记录修复后的完整验证证据（含本次运行的命令和结果，不复用上一轮的输出）。

**情形 B：**

1. 记录证据和风险评估。
2. 请开发决定是否接受风险。
3. 接受 → 追加 `DEC-*` 或 `DEV-*` 记录，继续交付，`delivery.md` 保留未覆盖风险章节。
4. 不接受 → 询问是否将修复既有问题纳入本次范围。纳入属于范围扩大，需要更新 `plan.md`、更新 `planVersion`、重新批准。

---

## 场景七：依赖 UI 但没有 UI 资料

### 触发条件

进入阶段二时，本次需求包含页面新增、页面改版或视觉还原类任务（依赖 UI），但没有提供本地 UI 目录，或提供了但缺少影响实现的关键信息（缺加载态、空数据态、失败态、交互说明）。

判断"是否依赖 UI"：页面新增、页面改版、视觉还原依赖 UI；接口调整、埋点、纯逻辑修改、Bug 修复不依赖 UI。不依赖 UI 时把 `plan.md` 的"UI与资源映射"标为"不适用"后继续，不构成本场景。

### 期望行为

```json
{
  "currentStage": "planning",
  "status": "active",
  "stageStatus": {
    "context": "complete",
    "planning": "active",
    "implementation": "pending",
    "verification": "pending",
    "delivery": "pending"
  },
  "blockingItems": [
    {
      "code": "missing_ui",
      "scope": "client-app/user-profile-page",
      "createdAt": "2026-08-13T10:00:00+08:00",
      "details": "资料页依赖 UI 还原，未提供 UI 目录"
    }
  ]
}
```

`scope` 是页面级或任务级，不是项目级。

全局 `status` 保持 `active`。理由：**其他无依赖任务可以继续**——不依赖 UI 的任务（接口对接、埋点接入、纯逻辑改造、其他页面）不受影响；即使全部任务都涉及这个页面，计划阶段的其他内容（文件清单骨架、API 清单、验证方案）仍可推进。命中派生规则 2。只有当所有剩余任务都依赖缺失的 UI 时，才变成 `blocked`。

`context.md` 必须列出**具体缺什么**，不能只写"缺 UI"：

```markdown
## 缺失输入

### UI 资料（scope: client-app/user-profile-page）

需要的页面图：
- 用户资料页 — 默认态（已加载完成，有头像有昵称）
- 用户资料页 — 加载中
- 用户资料页 — 加载失败
- 用户资料页 — 昵称为空的展示方式

需要的交互图或说明：
- 头像上传的选择、裁剪、上传进度、失败重试流程
- 表单校验失败的提示位置和样式

需要的适配说明：
- 目标屏幕尺寸范围
- 安全区、吸底按钮行为
```

关键约束：**不得为缺失的关键 UI 状态自行创造设计。** 可以按项目现有同类页面的既有样式实现骨架，但加载态、空态、失败态的具体呈现方式必须由 UI 提供，不自行发明。

UI 资料存在但缺关键信息时，同样记录 `missing_ui` 并列出缺失项，询问开发或 UI，不自行补全。

### 恢复步骤

1. 开发补充 UI 目录（写入 `uiDir`）或补充缺失的页面图。
2. **只重新执行 UI 分析**：图片名称、尺寸、更新时间、对应页面或状态、切图映射、布局、颜色、字体、间距、圆角、阴影、响应式、安全区、滚动、吸顶吸底、键盘行为。
3. 检查 UI 图与切图之间的缺失、重复、冲突，有冲突时记录并询问。
4. 补齐 `plan.md` 的"UI与资源映射"章节和受影响的文件清单、技术任务。
5. 移除 `missing_ui` 阻塞项。
6. `plan.md` 可执行内容变化时更新 `planVersion`，清空批准字段，重新批准。计划已批准过的情况下几乎必然需要重批。
7. 只重做受影响的计划内容和实现，其他部分不动。

---

## 场景八：缺少接口契约

### 触发条件

UI 任务或已批准的 Mock 任务已完成，需要开始真实接口编码或联调，但对应接口没有后端确认的契约（无接口文档，或文档缺该接口，或字段、错误码、分页方式未定）。

### 期望行为

```json
{
  "currentStage": "implementation",
  "status": "active",
  "stageStatus": {
    "context": "complete",
    "planning": "complete",
    "implementation": "active",
    "verification": "pending",
    "delivery": "pending"
  },
  "blockingItems": [
    {
      "code": "missing_api_contract",
      "scope": "client-app/api/profile-save",
      "createdAt": "2026-08-13T13:00:00+08:00",
      "details": "资料保存接口缺少后端确认的请求字段与错误码定义"
    }
  ]
}
```

`scope` 是接口级，对应到 `plan.md` API 清单里的具体条目。多个接口缺契约时写多条，不合并成项目级。

全局 `status` 保持 `active`。理由：**其他无依赖任务可以继续**——页面骨架、样式还原、静态状态、埋点、已有契约的其他接口都不受影响，命中派生规则 2。

关键约束：

- **不编写猜测字段的真实接口代码。** 可以识别需要哪些业务接口、列出待后端确认的问题，但不编造 URL、方法、字段名、错误码。
- 可以完成已批准的 Mock 和独立逻辑。Mock 必须在 `plan.md` 里被明确批准过，并在 `delivery.md` 标注哪些部分是 Mock。
- Mock 的数据结构不构成契约。后端确认的契约与 Mock 不一致时按契约改，不要求后端配合 Mock。

`context.md` 列出待后端确认的具体问题：

```markdown
## 开放问题

| ID | 优先级 | 问题 | 确认原因 | 影响范围 | 状态 |
| --- | --- | --- | --- | --- | --- |
| API-02 | P0 | 资料保存接口的请求字段与必填约束？ | 决定表单提交结构和前端校验 | 资料页保存流程、错误提示、相关测试 | 待确认 |
| API-03 | P1 | 保存失败的错误码与前端展示映射？ | 决定错误处理分支 | 失败态交互 | 待确认 |
```

### 恢复步骤

1. 开发提供后端确认的契约（写入或更新 `apiDocPath`）。
2. 追加 `DEC-*` 记录到 `decisions.md`，写明契约来源和确认角色。
3. 更新 `plan.md` 的 API 对接清单：请求方法、路径、请求字段、响应结构、错误码、分页方式。
4. 判断可执行内容是否变化：
   - 变化（与原计划假设的结构不同、需要新增转换层、错误分支增加）→ 更新 `planVersion`、清空三个批准字段、重新批准
   - 未变化（契约与计划中已列出的完全一致，只是从"待确认"变成"已确认"）→ 不更新 `planVersion`
5. 移除 `missing_api_contract` 阻塞项。
6. 实现真实接口对接，替换 Mock。
7. 移除本次修改产生的无用 Mock 代码和临时数据。
8. 重跑受影响接口的相关测试，更新 `delivery.md`。

---

## 场景九：实现过程中收到需求变更

### 触发条件

实现阶段进行中，收到产品变更，且影响接口和页面状态。

先确认这是需求变更而不是需求澄清、UI 变更、技术调整或缺陷修复。分类标准见 `change-control.md`。

### 期望行为

先做影响分析，再决定失效范围。

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
  "blockingItems": [],
  "invalidatedItems": [
    {
      "stage": "implementation",
      "scope": "admin-app/user-profile",
      "reason": "requirement_change",
      "changeId": "CR-001",
      "createdAt": "2026-08-13T15:00:00+08:00"
    }
  ],
  "requirementVersion": "2026-08-13T15:00:00+08:00",
  "planBasedOnRequirementVersion": "2026-08-13T09:00:00+08:00",
  "planVersion": "2026-08-13T10:00:00+08:00",
  "approvedPlanVersion": null,
  "approvedAt": null,
  "approvedByRole": null
}
```

字段变化逐项：

| 字段 | 变化 | 原因 |
| --- | --- | --- |
| `requirementVersion` | 更新为变更确认时间 | 需求基线变了 |
| `planBasedOnRequirementVersion` | **不动**，保持旧值 | 计划还没更新；与 `requirementVersion` 不等正是"计划已过期"的信号 |
| `approvedPlanVersion` / `approvedAt` / `approvedByRole` | 全部清空为 `null` | 变更影响已批准计划，批准失效 |
| `stageStatus.planning` | 设为 `invalidated` | 已批准的计划整体不再有效，必须重出 |
| `invalidatedItems` | 追加受影响的实现范围 | 局部失效 |
| `stageStatus.implementation` | 保持 `active` | 未受影响的实现任务可以继续 |

全局 `status` 为 `active`。理由：未受影响且无依赖的实现任务仍可继续，同时"更新 `context.md` 和 `plan.md`"本身就是可执行任务，命中派生规则 2。需求变更本身不产生阻塞——变更是"已有结论要重做"，不是"当前做不下去"。

`stageStatus.implementation` 保持 `active` 而不是 `invalidated`：只有实现阶段**全部**已完成内容都失效才设 `invalidated`。局部失效只记 `invalidatedItems`。

同时追加 `CR-*` 记录到 `decisions.md`，含原内容、新内容、变更来源、影响项目、受影响范围、不受影响范围、处理结论、需求版本。格式见 `change-control.md`。

暂停受影响的实现。未受影响且无依赖的任务继续。

### 恢复步骤

1. 更新 `context.md`：需求理解、已确认需求、验收标准。变更引入新的未确认问题时，加入开放问题并按 P0/P1/P2 分级，存在新 P0 时门禁一重新生效。
2. 更新 `plan.md`：文件清单、技术任务及依赖、API 对接清单、验证方案。只改受影响部分。
3. 设置新的 `planVersion`，并使 `planBasedOnRequirementVersion` 等于当前 `requirementVersion`。
4. 请求重新批准。批准后设置 `approvedPlanVersion` = 新 `planVersion`，记录 `approvedAt` 和 `approvedByRole`。
5. `stageStatus.planning` 从 `invalidated` 恢复为 `complete`。
6. 实现受影响范围。完成后移除对应 `invalidatedItems` 条目（Skill 自动清理，开发不手动改状态文件）。
7. **只重跑受影响测试和必要回归**，不从头重跑全部验证。必要回归的判断依据：变更是否触及公共能力、路由、请求层、权限或埋点。
8. `delivery.md` 记录本次变更的改动范围和验证证据，保留追溯。

已经写好但被废弃的代码：删除本次修改产生的无效代码和资源，在 `delivery.md` 说明哪些工作被废弃。不保留"以后可能用得上"的死代码。

---

## 场景对照速查

| 场景 | `code` / 机制 | `scope` 粒度 | 全局 `status` | 其他任务能否继续 |
| --- | --- | --- | --- | --- |
| 一 需求缺失 | `missing_requirement` | `context` | `blocked` | 否 |
| 二 P0 未确认 | `p0_unresolved` | 问题 ID | `blocked` | 否 |
| 三 计划未批准 | 无阻塞项 | — | `awaiting_confirmation` | 否（不得改代码） |
| 四 接口与文档不符 | `api_conflict` | 接口级 | `active` | 是 |
| 五 项目不可访问 | `inaccessible_project` | 项目级 | `active` | 是 |
| 六 验证失败（本次引入） | `verification_failed` | 模块/文件级 | `active` 或 `blocked` | 修复即可执行任务 |
| 六 验证失败（既有问题） | 不记阻塞项 | — | 不因此改变 | 是 |
| 七 缺 UI | `missing_ui` | 页面/任务级 | `active` | 是 |
| 八 缺接口契约 | `missing_api_contract` | 接口级 | `active` | 是 |
| 九 需求变更 | `invalidatedItems` | 阶段 + 范围 | `active` | 是 |

---

## 常见误判纠正

### 误判 1：场景三让 `implementation` 进入 `awaiting_confirmation`

❌ **错误：** `planning: "complete"` + `implementation: "awaiting_confirmation"`

✅ **正确：** `planning: "awaiting_confirmation"` + `implementation: "pending"`。等待的对象是计划。错误写法会让下次恢复误判实现已开始，跳过门禁二。

### 误判 2：范围级阻塞把全局 `status` 设成 `blocked`

❌ **错误：** 场景五、七、八各自把 `status` 设成 `blocked`。

✅ **正确：** 这三个场景全局都是 `active`。范围级阻塞只停该范围及其下游依赖，其他无依赖任务继续。判断依据是 `plan.md` 的任务依赖，不是直觉。

### 误判 3：场景二的 `scope` 写成模块名

❌ **错误：** `{ "code": "p0_unresolved", "scope": "user-profile" }`

✅ **正确：** `{ "code": "p0_unresolved", "scope": "PRD-01" }`。P0 阻塞的 `scope` 是具体问题 ID，每个未解决问题一条。写成模块名会导致产品逐条回复时无法精确移除。

### 误判 4：需求变更时记成阻塞项

❌ **错误：** 场景九追加 `{ "code": "requirement_changed", ... }`。

✅ **正确：** 需求变更用 `invalidatedItems`，不用 `blockingItems`。而且 `requirement_changed` 不在允许的 7 个 code 里。变更是"已有结论要重做"，不是"当前做不下去"。

### 误判 5：场景九把 `planBasedOnRequirementVersion` 一起改了

❌ **错误：** 变更确认时同步把 `planBasedOnRequirementVersion` 更新成新的 `requirementVersion`。

✅ **正确：** 保持旧值，直到 `plan.md` 真的更新完。两者不等正是"计划已过期，不得实现受影响部分"的判断依据。提前改掉会让这个保护失效。

### 误判 6：场景六局部重跑就算通过

❌ **错误：** 修复 3 个失败用例后只跑这 3 个，通过就标 `complete`。

✅ **正确：** 重新执行对应的完整验证。局部重跑可能掩盖修复引入的新问题。

### 误判 7：自动验证通过就进交付

❌ **错误：** lint、单测、构建都过，`verification: "complete"`，进入阶段五。

✅ **正确：** 人工验证项（真机、视觉对比、权限账号、跨端跳转）未执行并确认时，`verification` 是 `awaiting_confirmation`，`delivery` 保持 `pending`。

### 误判 8：批准计划后自动越过 `targetStage`

❌ **错误：** `executionMode: "planning_only"`，开发批准计划后直接开始改代码。

✅ **正确：** 批准计划不等于授权越过执行终点。停下报告，给出升级调用方式。

### 误判 9：场景五对不可访问项目做假设

❌ **错误：** 管理端读不到，就按客户端的技术栈假设它也一样，先把任务写进 `plan.md`。

✅ **正确：** 不假设任何实现。该项目的画像标"待补齐"，涉及它的文件清单和任务留空并标注原因。每个项目独立分析技术栈。

### 误判 10：恢复时直接信任文件里的 `status`

❌ **错误：** 读到 `status: "active"` 就直接继续。

✅ **正确：** `status` 是派生值。读取后按 `stageStatus`、`blockingItems`、`invalidatedItems` 和是否有可执行任务重算。中断期间开发可能补了资料或改了文件。

---

## 总结

**九个场景的两条主线：**

1. **缺输入或有冲突** → `blockingItems`，按 `scope` 精确停范围（场景一、二、四、五、六、七、八）
2. **已有结论要重做** → `invalidatedItems`，更新产物并重批后自动恢复（场景九）

场景三是第三类：既不缺输入也不需重做，只是在等人确认。

**恢复判断技巧：**

- 先问：**"上次为什么停？"** 缺东西 → 补齐后移除对应阻塞项。等人 → 展示待确认产物。做过的不能用了 → 更新产物并重批。
- 再问：**"这次能做什么？"** 逐条对照 `blockingItems.scope` 和 `plan.md` 的任务依赖，找出未被覆盖的任务。有就做，同时保留阻塞项。
- 最后问：**"能做到哪儿？"** 核对 `targetStage`，到终点就停下报告，不越过。
- 动手前先问：**"两个版本等式成立吗？"** `planBasedOnRequirementVersion == requirementVersion` 且 `approvedPlanVersion == planVersion`，都成立才能改业务代码。
