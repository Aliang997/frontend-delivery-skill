# Frontend Delivery Skill

[简体中文](./README.md) | [English](./README_EN.md)

`frontend-delivery` 是面向 Codex 的通用前端需求交付 Skill。它根据任务影响和不确定性选择小改动、常规需求或完整流程，避免简单修改承担整套流程成本，同时为复杂需求保留评审、批准、恢复、验证和交付能力。

当前稳定版本：**v2.3.1**

## 主要能力

- 小改动、常规需求、完整流程三级分流。
- 评审、计划、完整交付三种执行终点。
- 区分本次代码交接完成与整体验收完成。
- 按任务 ID 和版本管理批准、阻塞与局部失效。
- 图片、截图和 Figma MCP UI 还原。
- 动效还原与 Figma 来源变更检测。
- 按场景触发前端风险检查，不要求每次加载全部规则。
- 兼容 Windows、macOS 和 Linux 的文本、换行与路径约定。
- 衔接项目已有的代码评审、CI 和发布要求。

## 版本下载

| 版本 | 主要变化 | 浏览 | 下载 |
| --- | --- | --- | --- |
| 2.3.1 | 顺序分流、跨项目与公共修改下限、小改动精简回复 | [查看](https://github.com/new-forever/frontend-delivery-skill/tree/v2.3.1/skills/frontend-delivery) | [ZIP](https://github.com/new-forever/frontend-delivery-skill/archive/refs/tags/v2.3.1.zip) |
| 2.3.0 | 实现交接边界、任务重批收紧、Figma 基线与 R8 相关性 | [查看](https://github.com/new-forever/frontend-delivery-skill/tree/v2.3.0/skills/frontend-delivery) | [ZIP](https://github.com/new-forever/frontend-delivery-skill/archive/refs/tags/v2.3.0.zip) |
| 2.2.0 | 动效还原、Figma 变更检测、R8 条件性风险 | [查看](https://github.com/new-forever/frontend-delivery-skill/tree/v2.2.0/skills/frontend-delivery) | [ZIP](https://github.com/new-forever/frontend-delivery-skill/archive/refs/tags/v2.2.0.zip) |
| 2.1.0 | 图片、截图与 Figma MCP UI 还原 | [查看](https://github.com/new-forever/frontend-delivery-skill/tree/v2.1.0/skills/frontend-delivery) | [ZIP](https://github.com/new-forever/frontend-delivery-skill/archive/refs/tags/v2.1.0.zip) |
| 2.0.0 | 三级流程、任务级批准、恢复与风险检查 | [查看](https://github.com/new-forever/frontend-delivery-skill/tree/v2.0.0/skills/frontend-delivery) | [ZIP](https://github.com/new-forever/frontend-delivery-skill/archive/refs/tags/v2.0.0.zip) |
| 1.0.0 | 原始五阶段、两道门禁流程 | [查看](https://github.com/new-forever/frontend-delivery-skill/tree/v1.0.0/skills/frontend-delivery) | [ZIP](https://github.com/new-forever/frontend-delivery-skill/archive/refs/tags/v1.0.0.zip) |

完整变化见 [CHANGELOG.md](./skills/frontend-delivery/CHANGELOG.md)。

## 使用 Codex 安装

把下面这句话发送给 Codex，即可安装当前稳定版本：

```text
请安装 https://github.com/new-forever/frontend-delivery-skill/tree/v2.3.1/skills/frontend-delivery
```

安装指定旧版本时，将地址中的版本号替换为对应 Tag，例如：

```text
请安装 https://github.com/new-forever/frontend-delivery-skill/tree/v2.1.0/skills/frontend-delivery
```

安装完成后，Skill 会在下一轮对话中可用。

## 手动安装

1. 下载目标版本的 ZIP 并解压。
2. 找到 `skills/frontend-delivery` 目录。
3. 将该目录复制到 Codex Skill 目录：

Windows：

```text
%USERPROFILE%\.codex\skills\frontend-delivery
```

macOS / Linux：

```text
~/.codex/skills/frontend-delivery
```

同一环境不要同时安装多个同名版本。切换版本时，先备份或移除已有的 `frontend-delivery` 目录，再复制目标版本。

## 使用示例

评审需求：

```text
使用 $frontend-delivery 评审这个前端需求，只输出问题和风险。
```

制定计划：

```text
使用 $frontend-delivery 为这个需求制定前端实施计划，暂不修改代码。
```

完成小改动：

```text
使用 $frontend-delivery 修改这个按钮文案，并做必要的静态检查。
```

按 Figma 还原页面：

```text
使用 $frontend-delivery 按这个 Figma 节点还原页面：<Figma 节点链接>
```

## 目录结构

```text
skills/frontend-delivery/
├─ SKILL.md
├─ CHANGELOG.md
├─ references/
├─ assets/templates/
├─ workflow-diagram.md
└─ workflow-diagram.html
```

- `SKILL.md`：入口、流程分级和按需读取规则。
- `references/`：需求、计划、实现、UI、风险、恢复和交付细则。
- `assets/templates/`：常规与完整流程需要时使用的记录模板。
- [workflow-diagram.md](./skills/frontend-delivery/workflow-diagram.md)：当前版本流程图。
- `workflow-diagram.html`：可下载后在浏览器中查看的流程图页面。

流程图用于理解结构，具体执行条件、权限和完成标准以同版本 `SKILL.md` 与参考文件为准。

## 版本规则

本项目遵循语义化版本：

- 主版本：存在不兼容的工作流或状态变化。
- 次版本：增加向后兼容的能力。
- 修订版本：修正规则或文档，不改变兼容性。

已经公开的 Tag 保持不变。历史版本需要修正时发布新的修订版本，例如 `v2.2.1`。
