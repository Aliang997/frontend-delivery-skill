# 需求上下文分析

**需求标识：** {requirementId}  
**分析时间：** {timestamp}  
**需求版本：** {requirementVersion}

> 生成规则：填充真实内容后删除所有未使用的占位行、示例行和不适用的章节。表格只保留有实际内容的行——空表格比留着 `{placeholder}` 行更清楚。不适用的章节写"不适用"并注明原因，不要留占位符。

---

## 项目画像

### 项目基本信息

| 项目 | 路径 | 类型 | 框架 |
| --- | --- | --- | --- |
| {projectName} | {projectPath} | {projectType} | {framework} |

### 技术栈

**核心框架：** {framework} {version}

**路由：** {router}

**状态管理：** {stateManagement}

**UI 组件库：** {uiLibrary}

**样式方案：** {styleSolution}

**HTTP 库：** {httpLibrary}

**包管理器：** {packageManager}

**构建工具：** {buildTool}

**TypeScript：** {isTypescript}

### 项目能力

**登录和权限：**
- {loginMechanism}
- {authGuard}

**埋点和统计：**
- {analytics}

**跨端能力：**
- {crossPlatform}

**Mock 数据：**
- {mockSupport}

### 开发规范

**代码规范：**
- ESLint: {eslintConfig}
- Prettier: {prettierConfig}
- StyleLint: {stylelintConfig}

**提交规范：**
- {commitLint}

**项目规范文档：**
- {projectDocs}

### 构建和验证命令

```bash
# 开发
{devCommand}

# 构建
{buildCommand}

# Lint
{lintCommand}

# 类型检查
{typecheckCommand}

# 测试
{testCommand}
```

### Git 状态

**当前分支：** {currentBranch}

**未提交修改：** {uncommittedChanges}

---

## AI 对需求的理解

### 业务目标

{businessGoal}

**成功标准：**
- {successCriteria1}
- {successCriteria2}

### 功能范围

**页面清单：**
1. {page1} - {description1}
2. {page2} - {description2}

**主要交互流程：**
```
{userFlow}
```

**状态管理需求：**
- {stateRequirement}

### 用户与场景

**目标用户：** {targetUser}

**入口和前置条件：**
- 入口：{entry}
- 前置条件：{precondition}

**权限要求：**
- {authRequirement}

### 异常与边界

**状态处理：**
- 加载中：{loadingState}
- 空数据：{emptyState}
- 失败：{errorState}
- 重复操作：{duplicateAction}

**幂等性归属：**
- {idempotency}

### 跨端与平台

**目标平台：** {targetPlatform}

**平台差异：**
- {platformDifference}

### 数据与安全

**敏感数据处理：**
- {sensitiveData}

**埋点需求：**
- {trackingRequirement}

**数据统计口径：**
- {analyticsSpec}

### 明确不做的内容

- {outOfScope1}
- {outOfScope2}

### 验收标准

**功能标准：**
- {functionalCriteria}

**性能要求：**
- {performanceRequirement}

**兼容性要求：**
- {compatibilityRequirement}

---

## 已确认需求

{confirmedRequirements}

---

## 产品待确认问题

| ID | 优先级 | 问题 | 确认原因 | 影响范围 | 建议选项 | 状态 |
| --- | --- | --- | --- | --- | --- | --- |
| PRD-01 | P0 | {question1} | {reason1} | {impact1} | A ... / B ... | 待确认 |
| PRD-02 | P1 | {question2} | {reason2} | {impact2} | A ... / B ... | 待确认 |
| PRD-03 | P2 | {question3} | {reason3} | {impact3} | 默认：{default} | 待确认 |

**P0 问题（{p0Count}个）：** 不解决无法制定最终计划

**P1 问题（{p1Count}个）：** 不解决无法批准计划或开始编码

**P2 问题（{p2Count}个）：** 已给出默认方案，可继续；产品否决时按需求变更处理

---

## 需求明确不做

- {explicitOutOfScope1}
- {explicitOutOfScope2}

---

## 复用分析

### 可复用的页面模式

| 相似页面 | 路径 | 可复用点 |
| --- | --- | --- |
| {similarPage1} | {path1} | {reusablePoint1} |

### 可复用的组件

| 组件名称 | 路径 | 用途 | 适用场景 |
| --- | --- | --- | --- |
| {component1} | {componentPath1} | {usage1} | {applicableScenario1} |

### 可复用的 API 封装

| 封装名称 | 路径 | 功能 |
| --- | --- | --- |
| {apiWrapper1} | {apiPath1} | {functionality1} |

### 可复用的工具函数

| 函数名称 | 路径 | 功能 |
| --- | --- | --- |
| {utility1} | {utilityPath1} | {utilityFunction1} |

### 可复用的样式和主题

| 样式资源 | 路径 | 用途 |
| --- | --- | --- |
| {styleResource1} | {stylePath1} | {styleUsage1} |

---

## 可选：任务拆分与估时

> ⚠️ 本章节仅在开发明确要求时添加，不影响主流程。

### 任务清单

| 任务ID | 任务名称 | 工作内容 | 前置任务 | 估时（小时）|
| --- | --- | --- | --- | --- |
| T1 | {taskName1} | {taskContent1} | {dependency1} | {hours1} |
| T2 | {taskName2} | {taskContent2} | {dependency2} | {hours2} |

**总估时：** {totalHours} 小时

**关键路径：** {criticalPath}

**并行可能性：** {parallelPossibility}

---

**说明：** 本文档由 frontend-delivery skill 自动生成，记录项目分析和需求理解结果。