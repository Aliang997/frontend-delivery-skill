# 技术计划

**需求标识：** {requirementId}  
**计划版本：** {planVersion}  
**基于需求版本：** {planBasedOnRequirementVersion}  
**批准状态：** {未批准 | 已批准（approvedPlanVersion={approvedPlanVersion}，批准角色={approvedByRole}，批准时间={approvedAt}）}  
**计划性质：** {最终计划 | 草案（存在未解决 P1，见 context.md）}

> 多项目场景下，本文档所有文件、任务、接口和验证命令都必须标注所属项目。
>
> 生成规则：填充真实内容后删除所有未使用的占位行和示例行。不适用的章节写"不适用"并注明原因（例如需求不依赖 UI 时的"UI与资源映射"），不要留占位符。

**涉及项目：**

| 项目 | 路径 | 本次职责 |
| --- | --- | --- |
| {projectName1} | {projectPath1} | {responsibility1} |
| {projectName2} | {projectPath2} | {responsibility2} |

---

## 目标与非目标

### 目标

- {goal1}
- {goal2}

### 非目标

- {nonGoal1}
- {nonGoal2}

### 已确认前提

| 前提 | 来源 | 决策记录 |
| --- | --- | --- |
| {premise1} | {source1} | {DEC-001} |

### 未解决假设

> 仅计划草案阶段存在；批准前必须清空。

| 假设 | 关联问题ID | 若假设不成立的影响 |
| --- | --- | --- |
| {assumption1} | {PRD-0X} | {impactIfWrong1} |

---

## UI与资源映射

**UI 依赖判定：** {依赖 UI | 不适用（说明：{reasonNotApplicable}）}

**UI 目录：** {uiDir}　**切图目录：** {assetsDir}

### 页面与 UI 图对应

| 项目 | 页面/状态 | UI 图文件 | 尺寸 | 备注 |
| --- | --- | --- | --- | --- |
| {projectName} | {pageOrState} | {uiFileName} | {dimensions} | {note} |

### 切图使用

| 项目 | 切图文件 | 格式/尺寸 | 用途 | 落地路径 |
| --- | --- | --- | --- | --- |
| {projectName} | {assetFileName} | {formatAndSize} | {usage} | {targetPath} |

### 视觉规范

| 项 | 取值 | 来源 |
| --- | --- | --- |
| 颜色 | {colors} | {source} |
| 字体与字号 | {typography} | {source} |
| 间距 | {spacing} | {source} |
| 圆角与阴影 | {radiusAndShadow} | {source} |

### 适配与交互行为

| 项 | 处理方式 |
| --- | --- |
| 响应式断点 | {breakpoints} |
| 安全区 | {safeArea} |
| 滚动与吸顶吸底 | {scrollAndSticky} |
| 键盘行为 | {keyboardBehavior} |

### UI 缺失、重复与冲突

| 类型 | 内容 | 影响范围 | 处理 |
| --- | --- | --- | --- |
| {缺失/重复/冲突} | {description} | {scope} | {待UI确认 / 已确认见 UCR-00X} |

> 不得为缺失的关键 UI 状态自行创造设计。

---

## 需要创建的文件

| 项目 | 文件路径 | 类型 | 说明 |
| --- | --- | --- | --- |
| {projectName} | {newFilePath1} | 业务 | {purpose1} |
| {projectName} | {newFilePath2} | 测试 | {purpose2} |
| {projectName} | {newFilePath3} | 资源 | {purpose3} |

---

## 需要修改的文件

| 项目 | 文件路径 | 改动内容 | 影响面 |
| --- | --- | --- | --- |
| {projectName} | {modifiedFilePath1} | {change1} | {impact1} |
| {projectName} | {modifiedFilePath2} | {change2} | {impact2} |

### 复用而不修改的实现

| 项目 | 路径 | 复用方式 |
| --- | --- | --- |
| {projectName} | {reusedPath1} | 直接复用 |

---

## 数据流和页面状态

### 数据流

```text
{dataFlowDiagram}
```

### 状态清单

| 项目 | 页面/组件 | 状态 | 触发条件 | 展示与行为 |
| --- | --- | --- | --- | --- |
| {projectName} | {pageOrComponent} | 初始 | {trigger} | {behavior} |
| {projectName} | {pageOrComponent} | 加载中 | {trigger} | {behavior} |
| {projectName} | {pageOrComponent} | 空数据 | {trigger} | {behavior} |
| {projectName} | {pageOrComponent} | 失败 | {trigger} | {behavior} |
| {projectName} | {pageOrComponent} | 无权限 | {trigger} | {behavior} |

### 状态存放位置

| 数据 | 存放位置 | 生命周期 | 是否敏感 |
| --- | --- | --- | --- |
| {dataName1} | {location1} | {lifecycle1} | {isSensitive1} |

---

## API对接清单

**契约状态说明：** 已确认 = 后端书面确认；待确认 = 不得编写猜测字段的真实接口代码。

| 序号 | 项目 | 用途 | 方法 | 路径 | 契约状态 | 契约来源 | 调用页面/时机 |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 1 | {projectName} | {usage1} | {method1} | {path1} | 已确认 | {contractSource1} | {caller1} |
| 2 | {projectName} | {usage2} | {method2} | {path2} | 待确认 | — | {caller2} |

### 请求与响应处理

| 序号 | 关键请求参数 | 关键响应字段 | 前端加工 |
| --- | --- | --- | --- |
| 1 | {requestParams1} | {responseFields1} | {transform1} |

### 错误与重试

| 序号 | 错误场景 | 前端处理 | 重试策略 | 幂等归属 |
| --- | --- | --- | --- | --- |
| 1 | {errorCase1} | {handling1} | {retryPolicy1} | {idempotencyOwner1} |

### Mock 策略

| 序号 | 是否需要 Mock | Mock 方式 | 切换真实接口的条件 |
| --- | --- | --- | --- |
| 2 | 是 | {mockApproach} | 后端确认契约后 |

### 待后端确认的问题

| ID | 项目 | 问题 | 影响任务 |
| --- | --- | --- | --- |
| API-01 | {projectName} | {apiQuestion1} | {affectedTask1} |

---

## 技术任务及依赖

| 任务ID | 项目 | 任务 | 产出文件 | 前置任务 | 阻塞条件 |
| --- | --- | --- | --- | --- | --- |
| T1 | {projectName} | {task1} | {output1} | — | {blockingCondition1} |
| T2 | {projectName} | {task2} | {output2} | T1 | {blockingCondition2} |
| T3 | {projectName} | {task3} | {output3} | T2 | {blockingCondition3} |

### 执行顺序

```text
{T1} → {T2} → {T3}
```

### 可并行任务

- {parallelGroup1}

### 跨项目依赖

| 依赖类型 | 上游项目 | 下游项目 | 内容 | 约束 |
| --- | --- | --- | --- | --- |
| 页面跳转 | {upstreamProject} | {downstreamProject} | {jumpAndParams} | {constraint} |
| 接口契约 | {upstreamProject} | {downstreamProject} | {sharedContract} | {constraint} |
| 联调顺序 | {upstreamProject} | {downstreamProject} | {integrationOrder} | {constraint} |
| 发布顺序 | {upstreamProject} | {downstreamProject} | {releaseOrder} | {constraint} |

---

## 验证方案

### 1. 需要运行的命令与通过标准

| 序号 | 项目 | 类型 | 命令 | 通过标准 |
| --- | --- | --- | --- | --- |
| V1 | {projectName} | Lint | {lintCommand} | 退出码 0，error 数 0，warning ≤ {warningThreshold} |
| V2 | {projectName} | 类型检查 | {typecheckCommand} | 退出码 0，无新增类型错误 |
| V3 | {projectName} | 单元测试 | {unitTestCommand} | 退出码 0，失败 0，{coverageTarget} |
| V4 | {projectName} | 组件测试 | {componentTestCommand} | 退出码 0，失败 0 |
| V5 | {projectName} | E2E | {e2eCommand} | 退出码 0，用例全部通过 |
| V6 | {projectName} | 构建 | {buildCommand} | 退出码 0，产出 {buildArtifact} |

### 2. 手动验证项

| 序号 | 项目 | 验证内容 | 环境/设备 | 通过标准 | 执行人 |
| --- | --- | --- | --- | --- | --- |
| M1 | {projectName} | 登录与权限流程 | {environment} | {criteria} | 开发 |
| M2 | {projectName} | 视觉还原对比 | {environment} | {criteria} | 开发 |
| M3 | {projectName} | 真机/目标屏幕 | {deviceOrScreen} | {criteria} | 开发 |
| M4 | {projectName} | 跨端跳转链路 | {environment} | {criteria} | 开发 |
| M5 | {projectName} | 旧功能回归 | {environment} | {criteria} | 开发 |

### 3. 需要的测试账号、权限、环境与数据

> 只列出需要什么，不在任何产物文件中保存账号、密码、Token、验证码或其他凭证。凭证由开发提供并自行保管。

| 类型 | 需要的内容 | 用途 | 提供方 |
| --- | --- | --- | --- |
| 账号 | {accountRoleDescription} | {purpose} | 由开发提供 |
| 权限 | {permissionScope} | {purpose} | 由开发提供 |
| 环境 | {environmentName} | {purpose} | 由开发提供 |
| 测试数据 | {testDataDescription} | {purpose} | 由开发提供 |

### 4. 验证顺序、前置条件与跨项目依赖

| 步骤 | 项目 | 执行内容 | 前置条件 | 跨项目依赖 |
| --- | --- | --- | --- | --- |
| 1 | {projectName} | {V1}、{V2} | 依赖已安装 | 无 |
| 2 | {projectName} | {V3}、{V4} | 步骤 1 通过 | 无 |
| 3 | {projectName} | {V5} | {environmentReady} | {upstreamProject} 已联调 |
| 4 | {projectName} | {V6} | 步骤 1-3 通过 | 无 |
| 5 | {projectName} | {M1}-{M5} | 步骤 4 通过 | {crossProjectDependency} |

### 5. 缺少测试能力时的替代方式与未覆盖风险

| 缺失能力 | 项目 | 替代验证方式 | 未覆盖风险 | 风险等级 |
| --- | --- | --- | --- | --- |
| {missingCapability1} | {projectName} | {alternativeApproach1} | {uncoveredRisk1} | {高/中/低} |

### 6. 本次验证不覆盖的范围

- {outOfVerificationScope1}
- {outOfVerificationScope2}

---

## 不允许修改的范围

| 项目 | 路径或能力 | 原因 |
| --- | --- | --- |
| {projectName} | {protectedPath1} | {reason1} |
| {projectName} | {protectedCapability2} | {reason2} |

### 需要单独授权的动作

- 新增或升级依赖：{dependencyChangeNeeded | 无}
- 修改环境配置：{envConfigChangeNeeded | 无}
- 提交、推送、创建 PR、部署、发布：需开发单独授权

---

**说明：** 本文档由 frontend-delivery skill 自动生成。计划批准后修改可执行内容需更新 planVersion 并重新批准；任务拆分与估时属于 context.md 的按需附录，不在本文档中。
