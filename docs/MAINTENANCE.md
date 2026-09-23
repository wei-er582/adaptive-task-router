---
status: active
owner: Samsung
last_verified: 2026-09-23
verified_commit: "adaptive-task-router-v0.2.0"
applies_to: [adaptive-task-router]
supersedes: []
---

# 结构、安装、测试与回滚

本插件只有技能、参考和合成测试材料，没有运行时服务。源目录维护权威，安装缓存可重建，ZIP 是交付快照。

## 工作流与文档责任

执行入口记录对话内持续启用；建议入口只在尚未启用且值得分工时询问。简单任务直接完成；复杂任务按需选型、分配共享名额、原生委派、按证据验收。领域技能保留专业流程。

| 内容 | 权威位置 |
| --- | --- |
| 持续启用、单次例外、关闭、快速路径与交付 | route-task/SKILL.md |
| 17 项启发式 | route-task/references/model-routing.md |
| 局部求解、全局整合与验证的高档配置 | route-task/references/high-effort-orchestration.md |
| 原生接口、配置覆盖、可观察调用证据 | route-task/references/native-dispatch.md |
| 全树名额、版本依赖、共享资源、停止恢复 | route-task/references/recursive-orchestration.md |
| 未启用时的建议与拒绝状态 | suggest-task-routing/SKILL.md |

表内路径相对插件 `skills/`。历史取舍见 [ADR 0002](architecture/adr/0002-session-routing.md)。治理信息放入技能标准 frontmatter 的 metadata，其余活动文档使用顶部元数据；版本 evidence ID 不冒充最终提交 SHA。

## 配置与数据边界

执行入口 implicit invocation 为 false，建议入口为 true。前者不妨碍沿用已明确启用的对话规则；需要在可信续接中保留状态，不能承诺任意上下文丢失后仍能恢复。

容量按宿主对当前任务的实际定义与用户限制核算，所有后代共用，没有固定三个上限。显式指定子代理模型/档位，使用 fork none。Root 分配名额属于技能指令，不能替代宿主硬限额或锁。

不新增全局模型、代理、Ultra、权限或 AGENTS 配置；不增加运行凭证、付费 API、Fast/Goal/自动化或跨任务记忆。必要检查点仅在允许写入时放入当前任务工作目录，不放用户全局配置。默认不保存完整任务资料和使用记录。

## 公开分发与个人开发安装

公开仓库为 `https://github.com/wei-er582/adaptive-task-router`；仓库市场通过 Git URL 指向根目录插件。面向使用者的安装入口以 [安装指南](INSTALLATION.md) 为准。下面保留个人本地源码的开发流程，不要求维护者重复启用两个同名来源。

本机源目录 `%USERPROFILE%\plugins\adaptive-task-router`，个人市场 `%USERPROFILE%\.agents\plugins\marketplace.json`，source.path 为 `./plugins/adaptive-task-router`，相对用户目录这个市场根解析。

新机器使用分发包中的完整插件目录，通过目标 Codex 的 plugin-creator 标准个人市场流程登记，保留已有市场条目。已有同名插件时先检查版本和本地改动，不直接覆盖。不要用分发包替换整份市场或全局配置，也不需要公开发布。标准个人市场隐式发现，不用额外 marketplace add。

在具有本机官方 helper 的环境中，从任意目录验证市场名称再安装：

```powershell
$routerMarket = python -X utf8 "$env:USERPROFILE\.codex\skills\.system\plugin-creator\scripts\read_marketplace_name.py"
if ($LASTEXITCODE -ne 0) { throw "个人市场标识验证失败" }
codex plugin add "adaptive-task-router@$routerMarket" --json
```

更新现有本地源后，先验证，再刷新缓存并重装：

```powershell
python -X utf8 "$env:USERPROFILE\.codex\skills\.system\plugin-creator\scripts\update_plugin_cachebuster.py" "$env:USERPROFILE\plugins\adaptive-task-router"
if ($LASTEXITCODE -ne 0) { throw "缓存版本更新失败" }
codex plugin add "adaptive-task-router@$routerMarket" --json
```

helper 保留业务版本前缀并生成单个 `+codex.<timestamp>` 后缀。0.2.0 是本次真实行为更新；不为普通缓存刷新反复递增业务版本。不要编辑安装缓存或手改市场注册。重装后用新对话首次调用，新内容可能不会自动进入已有任务的旧上下文。

## 验证

```powershell
python -X utf8 "$env:USERPROFILE\.codex\skills\.system\plugin-creator\scripts\validate_plugin.py" "$env:USERPROFILE\plugins\adaptive-task-router"
python -X utf8 "$env:USERPROFILE\.codex\skills\.system\skill-creator\scripts\quick_validate.py" "$env:USERPROFILE\plugins\adaptive-task-router\skills\route-task"
python -X utf8 "$env:USERPROFILE\.codex\skills\.system\skill-creator\scripts\quick_validate.py" "$env:USERPROFILE\plugins\adaptive-task-router\skills\suggest-task-routing"
python -X utf8 "$env:USERPROFILE\plugins\adaptive-task-router\tests\check_structure.py"
git -C "$env:USERPROFILE\plugins\adaptive-task-router" diff --check
```

同时检查元数据、本地 Markdown 引用、两个策略标志、17 项参考、占位内容、合成材料完整性和安装缓存哈希。结构通过不是行为证明，详见[场景](../tests/SCENARIOS.md)。

安装发现可通过临时 `codex app-server --stdio` 的 `skills/list` 和 `plugin/read` 只读核对；它只证明本地注册及入口可见，不能证明模型实际触发。再用 `codex exec --ephemeral --json` 的只读小任务检查端到端调用，保留必要事件证据，不持久创建普通任务。凭据或服务阻断时单列未完成项，不混称通过；这些 CLI 仅作维护验收，不能作为运行时缺少原生工具的绕行方案。多轮状态采用原生 follow-up；实际压缩与模拟摘要必须分开记录。真实任务验证使用合成材料及有限原生代理，不跑全部 17 档。

## 回滚与卸载

本机本地 Git 的 `v0.1.0` 保存升级前与安装缓存一致的 18 文件基线。回滚前检查 `git status --short`，妥善提交或保存未提交成果；在干净树中建立专用回退分支：

```powershell
git -C "$env:USERPROFILE\plugins\adaptive-task-router" switch -c rollback-v0.1.0 v0.1.0
```

分支已存在时检查其内容并选择新的回退分支名，不强制覆盖。随后按上面的 helper 更新缓存后缀、重装并验证版本/启用状态，提交回退记录。需要恢复新版时返回保存的新版提交，重复验证及重装。ZIP 不含 .git；新机器没有旧版本历史时，回退需要自行保留的旧版源码或分发包。

卸载通过插件界面，或先检查 `codex plugin remove --help` 再按该版本支持的命令操作。卸载插件、保留源码和保留市场条目是不同事项，不宣称卸载会删除全部文件。不盲目 reset 或清理未知改动。

本项目通过 GitHub 公开分发，不需要服务器部署。打包排除 .git、临时脚本、原始会话、凭据和测试中间产物；核对解压清单与每文件哈希。个人市场及其他插件设置保留原值。旧验证报告中的“仅本地”描述其当时范围，不代表现在没有公开仓库。

## 公开发布

本地修改通过结构检查、两个内置技能验证器及插件验证器后提交，再推送公开仓库。仓库 CI 只运行结构检查，不访问模型。公开市场安装应在独立测试配置中检查，避免覆盖维护者的个人安装来源。

发布包从已提交文件生成，核对 ZIP 条目和 SHA-256；把版本、目标提交、已运行检查与未覆盖项写入发布说明。不要上传包含本机路径的私人验收 JSON、完整会话或临时日志。业务版本不因文档扩充而变化，安装缓存后缀按官方 helper 更新。历史标签保持不动，首次开源包装使用 `public-2026-09-23` 标签。
