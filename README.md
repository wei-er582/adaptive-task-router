---
status: active
owner: wei-er582
last_verified: 2026-09-23
verified_commit: "adaptive-task-router-public-2026-09-23"
applies_to: [adaptive-task-router]
supersedes: []
---

# Adaptive Task Router · 智能任务调度

**一次启用，在当前对话持续按任务难度选型、分工与验收。**

Adaptive Task Router 是面向 Codex 的开源技能插件。它将“目标是什么、难点在哪里、谁来完成、怎样证明完成”组织成可执行流程，在有收益时使用不同模型、推理等级和递归子代理。简单任务直接完成。

原则是：**先保证交付质量，再减少不必要的推理、重复执行和返工。**

本项目为社区项目，不由 OpenAI 官方维护。插件提供行为指令；模型可用性、原生工具、权限、并发容量和实际执行由你的 Codex 宿主决定。

## 快速安装

需要能管理插件的 Codex 客户端。CLI 安装：

```sh
codex plugin marketplace add wei-er582/adaptive-task-router
codex plugin add adaptive-task-router@adaptive-task-router
```

安装后新建一个对话，以载入新版技能。首次使用：

```text
请使用 $adaptive-task-router:route-task 完成以下任务：……
```

之后在**同一对话**直接说“继续实现”“再分析这个新问题”即可，无需每条提示再次勾选。已选中插件时可以使用短名 `$route-task`。

图形界面安装、更新、已有个人安装的处理、离线包和排障见 [安装指南](docs/INSTALLATION.md)。这是 GitHub 仓库提供的插件市场，不表示已进入 OpenAI 官方公共目录。

## 适合什么任务

| 场景 | 可以带来的帮助 |
| --- | --- |
| 软件开发与调试 | 按模块划分修改责任，独立复核关键实现，再整合验证 |
| 研究与资料分析 | 区分检索、证据核对、综合判断和不确定结论 |
| 数学建模与复杂推理 | 分开局部求解、全局整合、反例搜索与证明检查 |
| 文档与方案评估 | 识别读者、约束、交付格式和验收要求，调用适用的领域技能 |
| 简单问答或小修改 | 直接完成，避免为了分工而制造额外代理 |

插件不会仅因任务“重要”就把所有节点设为最高档，也不会一律从最低档试起。它根据难度、错误影响、验证难度、上下文依赖和委派收益选择配置。

## 两个入口

| 技能 | 启用规则 | 功能 |
| --- | --- | --- |
| `route-task` | 明确调用或接受建议后，在本对话持续生效 | 分析目标、选型、原生委派、协调和验收 |
| `suggest-task-routing` | 可自动发现；尚未启用且分工收益明显时建议 | 只评估是否值得开启，不自行启动执行 |

执行入口保留 `allow_implicit_invocation: false`；后续使用沿用已经明确启用的对话规则。建议入口为 `true`，但**未答不是同意，拒绝后同一任务不重复询问**。

## 持续启用怎样控制

| 你说的话 | 行为 |
| --- | --- |
| “继续”“再处理这个新任务” | 沿用启用状态，重新判断本请求的目标与操作范围 |
| “这次不用插件” | 仅本请求暂时停用，结束后恢复默认 |
| “这一步不要委派” | 当前范围由主代理完成，保留其他路由规则 |
| “关闭插件”“后续不用了” | 持续关闭，直到你明确重新开启 |
| “先停止”“完成当前块后停止” | 暂停工作、保留启用偏好；收到继续指令后才恢复 |
| “只分析，不执行” | 只做规划；计划模式同样不能委派实施写入 |

仅讨论插件、引用其他对话中的启用语句，或调用建议入口，都不等于开启执行。没有有效继承状态的新对话不会自动启用。

## 调度方式

1. **明确任务**：确认交付物、验收、材料、约束和依赖；可自行查清的内容先检查。
2. **定位难点**：分别判断局部求解、全局整合与最终验证需要多强配置。
3. **按需分派**：复杂任务给出简明分工，显式请求模型/档位；简单任务直接做。
4. **共享资源与名额**：整棵后代树共用宿主容量。子代理在授权边界和分配名额内可继续委派。
5. **验证与调整**：依据原始证据、稳定版本和验收标准；连续两次无进展就重新诊断。
6. **交付**：说明完成内容、实际调用证据、检查结果和未解决事项。

参考覆盖 Astra、Sol、Luna 的 15 个基础组合及 Sol/Astra Ultra，共 17 个选项。它们是**可调整的起始策略，不是统一实测性能排行榜**。实际选项以当前工具公布的支持范围为准。

Ultra 的委派模式、主代理的推理投入、各子任务的模型/档位是不同决策。插件不会修改 Ultra 映射，也不会声称已经热切换主代理。详细依据见 [模型参考](skills/route-task/references/model-routing.md) 与 [高档配置策略](skills/route-task/references/high-effort-orchestration.md)。

## 一个使用示例

```text
请使用 $adaptive-task-router:route-task 检查这个项目：
目标是找出导致结算结果错误的根因，给出最小修复并验证。
可以按需要使用子代理；不同修改者不要同时写同一个文件。
不要部署，也不要增加本需求之外的功能。
```

该请求可能拆成实现定位、独立复现和边界审查；具体模型、档位和是否委派由实际问题决定。后续只需说：

```text
继续修复刚才确认的问题，再补一个针对舍入边界的验证。
```

更多开发、研究、建模、文档场景及启停示例见 [使用指南与 FAQ](docs/USAGE.md)。

## 能力边界与验证

- 持续启用依赖有效对话上下文或可信恢复记录，不保证上下文丢失后自动恢复，也不保证界面一直显示勾选。
- 递归名额、单写入者和停止传播是协调规则；宿主负责真正的硬限制和权限控制。
- 没有原生子代理工具时，可在已授权范围内用当前代理工作并说明限制，不另起 CLI、付费 API 或普通侧边栏任务冒充原生调度。
- 不承诺更便宜或更快，不推算订阅扣费；调用参数与实际服务端配置回显分开记录。
- 领域技能仍负责专业操作流程，本插件不替代它们。

0.2.0 的验证包括结构检查、9 轮真实续接、有界主→子→孙调用、18 项规则场景审查、合成任务独立复核和模拟摘要续接。实际宿主压缩未验证；历史临时 CLI 模型调用曾被凭据 401 阻断。两项限制完整保留在 [分类验证记录](docs/releases/0.2.0-validation.md)。

GitHub CI 检查文件结构和规则元数据，**不会调用模型**，不等于行为基准。

## 文档与贡献

- [安装、升级与排障](docs/INSTALLATION.md)
- [使用示例与 FAQ](docs/USAGE.md)
- [维护、测试与回滚](docs/MAINTENANCE.md)
- [持续启用与递归设计决策](docs/architecture/adr/0002-session-routing.md)
- [行为测试场景](tests/SCENARIOS.md)
- [变更记录](CHANGELOG.md)
- [贡献指南](CONTRIBUTING.md)
- [MIT 许可证](LICENSE)

## English overview

Adaptive Task Router is a community Codex plugin for persistent, task-aware model selection and native subagent coordination. Enable it once per conversation, then continue normally. It delegates only when useful, supports bounded recursion across a shared host capacity, and verifies outcomes against evidence. It does not hot-switch the root model, alter global defaults, guarantee runtime behavior, or provide an independent orchestration service. Documentation is primarily in Chinese.
