---
status: active
owner: Samsung
last_verified: 2026-09-23
verified_commit: "adaptive-task-router-v0.1.0"
applies_to: [adaptive-task-router]
supersedes: []
---

# 结构、配置与维护

这是一个仅包含技能和参考材料的本地插件。本页合并小型插件的结构、配置、开发、测试、安装和回退说明；不建立没有实际内容的服务运维目录。

## 结构与数据流

插件清单发现两个技能。`suggest-task-routing` 只做轻量适用性判断；得到当前任务的明确同意后转入 `route-task`。用户直接调用执行技能时跳过建议阶段。

执行技能先定义交付与验收，按需读取选型参考和原生调用参考，再使用当前会话的调度工具。子代理交回证据、变更和检查结果，主代理集成并完成最终验收。插件本身没有 HTTP API、数据库、MCP、后台进程或凭证配置。

| 权威内容 | 位置 |
| --- | --- |
| 执行授权、任务拆分、升级与交付规则 | `skills/route-task/SKILL.md` |
| 17 个选项的启发式参考 | `skills/route-task/references/model-routing.md` |
| 原生调用兼容边界 | `skills/route-task/references/native-dispatch.md` |
| 建议触发与同意/拒绝处理 | `skills/suggest-task-routing/SKILL.md` |
| 技能显示名、默认提示、自动发现策略 | 各技能的 `agents/openai.yaml` |
| 插件身份及用户界面入口 | `.codex-plugin/plugin.json` |

技能 frontmatter 使用标准 schema；治理信息放入受支持的 `metadata` 中。其他活动 Markdown 使用顶部治理元数据。`adaptive-task-router-v0.1.0` 是本地版本证据标识，不冒充 Git SHA；提交记录由 Git 提供。

## 配置边界

执行入口 `allow_implicit_invocation` 为 false；建议入口为 true。改变这两个值会改变用户的触发约定，不能作为无关安装修复顺手修改。

默认三个并发子代理是提示策略，宿主容量是硬限制；有效数量还要考虑已经占用的槽位。子任务逐次指定 `model` 和 `reasoning_effort`，当前 `collaboration.spawn_agent` 使用 `fork_turns: none`。没有全局模型或推理设置修改，也不持久保存每次任务的输入、资料或用量。

## 本机安装与其他机器安装

本机使用 `%USERPROFILE%\plugins\adaptive-task-router` 作为源目录，个人市场在 `%USERPROFILE%\.agents\plugins\marketplace.json`，源条目为 `./plugins/adaptive-task-router`，市场根为用户目录。

新机器需要 Codex 插件功能与 Python（仅结构验证时需要 PyYAML）。如需从分发包安装，在目标 Codex 中使用 `plugin-creator` 的标准个人市场流程：先检查目录和现有条目，用 scaffold 创建/登记此名称的源插件，再把分发包的插件文件放到该源目录，保留已有市场条目和顺序。不要用压缩包中的内容覆盖整份个人市场配置。

注册后运行：

```powershell
codex plugin add adaptive-task-router@personal --json
codex plugin list --marketplace personal --json
```

标准个人市场隐式发现，不需要额外 `marketplace add`。仅安装本地插件，不发布到公共市场或同步到 GitHub/服务器。安装后用新任务验证技能发现。

## 结构验证

在包含对应官方技能脚本的机器上运行；PowerShell 中 Python 的 UTF-8 模式可避免中文内容受系统代码页影响：

```powershell
python -X utf8 "$env:USERPROFILE\.codex\skills\.system\plugin-creator\scripts\validate_plugin.py" "$env:USERPROFILE\plugins\adaptive-task-router"
python -X utf8 "$env:USERPROFILE\.codex\skills\.system\skill-creator\scripts\quick_validate.py" "$env:USERPROFILE\plugins\adaptive-task-router\skills\route-task"
python -X utf8 "$env:USERPROFILE\.codex\skills\.system\skill-creator\scripts\quick_validate.py" "$env:USERPROFILE\plugins\adaptive-task-router\skills\suggest-task-routing"
```

同时检查本地 Markdown 链接、治理元数据、两个 invocation 标志和 17 项参考。官方验证器不证明模型会正确执行全部规则，行为验证见[测试场景](../tests/SCENARIOS.md)。

## 更新、回退与卸载

修改前检查本地 Git 状态，辨别未提交内容。对变化执行相称的结构或行为验证，更新相关说明与开发记录。

对已安装版本按 `plugin-creator` 的更新流程：使用 `read_marketplace_name.py` 验证个人市场标识，使用 `update_plugin_cachebuster.py` 更新清单缓存后缀，再执行 `codex plugin add adaptive-task-router@personal --json`。不要手改全局模型设置或直接编辑安装缓存。源码更新后重新安装才会刷新缓存，已有任务仍应在新任务中重新试用。

需要回退时先检查工作树，保存有价值的未提交内容，然后在干净的本地版本上按同样的缓存后缀与重装流程恢复；不盲目执行 destructive reset。卸载使用 Codex 插件界面或当前 CLI `codex plugin remove` 帮助确认的参数形式，只卸载该插件。市场源文件与本地源码是否保留是独立选择，不宣称卸载已删除所有源码。

源目录是维护权威；安装缓存是可重建产物，`outputs` 中的 ZIP 是不可变分发快照。打包不包含 `.git`、临时材料或用户会话记录。每次包生成后核对文件清单和内容哈希。
