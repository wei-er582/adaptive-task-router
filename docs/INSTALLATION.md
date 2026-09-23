---
status: active
owner: wei-er582
last_verified: 2026-09-23
verified_commit: "adaptive-task-router-public-2026-09-23"
applies_to: [adaptive-task-router]
supersedes: []
---

# 安装、更新与排障

## 支持范围与前提

需要支持插件的 Codex 客户端。已进行本地验证的 CLI 版本为 0.156.0；这不是最低版本承诺。先确认本机具备相应命令：

```sh
codex --version
codex plugin --help
codex plugin marketplace add --help
```

需要通过 Git 安装时还需可用的 Git 和 GitHub 网络连接。运行技能时使用你已有的 Codex 登录与模型权限；本插件没有自己的 API key、MCP 服务或额外付费 API。

本包保留经过验证的 `.codex-plugin/plugin.json` 兼容格式。不同客户端的插件格式与原生子代理能力可能不同，不能由安装成功推断所有模型均可调用。

## 方法一：从 GitHub 插件市场安装

```sh
codex plugin marketplace add wei-er582/adaptive-task-router
codex plugin add adaptive-task-router@adaptive-task-router
```

每一步成功后再运行下一步。第一项注册本仓库的市场；第二项安装插件。仓库市场名和插件名都为 `adaptive-task-router`，所以选择器两侧名称相同。

仓库内的市场文件位于 `.agents/plugins/marketplace.json`，通过 Git URL 指向本仓库根目录的插件，默认跟踪 `main`。它不覆盖个人市场 `personal`，也不改全局模型、推理档位或其他插件设置。

安装机制依据 [OpenAI 官方插件打包说明](https://developers.openai.com/plugins/build/plugins)，命令参数同时以本机 `--help` 为准。

安装后新建一个对话，输入：

```text
请使用 $adaptive-task-router:route-task 分析并完成以下任务：……
```

“插件已安装并启用”是宿主配置；“在当前对话开启执行规则”是一次明确调用。安装本身不会为所有对话开启路由。

## 方法二：插件界面添加仓库

在支持添加自定义市场的插件界面中，选择添加市场/从仓库添加，输入：

```text
https://github.com/wei-er582/adaptive-task-router
```

同步后选择 Adaptive Task Router 并安装。菜单文字因客户端而异；如果界面没有此入口，使用 CLI 方式或客户端自带的 plugin-creator。图形界面流程需要在你的客户端确认，不把本地 CLI 发现测试当作 UI 测试。

## 方法三：ZIP 或本地源码

从 GitHub Releases 获取插件 ZIP，解压时保留隐藏的 `.codex-plugin` 目录。下载源码也可以：

```sh
git clone https://github.com/wei-er582/adaptive-task-router.git
```

若要把此目录作为可修改的个人插件安装，使用 Codex 自带的 `plugin-creator`，让它把源码登记到本机个人市场：

```text
请使用 plugin-creator 将这个 adaptive-task-router 目录注册到个人市场并安装。
先检查同名插件及本地改动，保留已有市场条目，使用当前客户端支持的标准流程。
仅本地安装，不改全局模型或推理设置。
```

直接解压不等于安装。不要把别人整份 config.toml、市场文件或缓存目录复制到自己的配置中。本地开发更新流程见 [维护说明](MAINTENANCE.md)。

## 已经通过 personal 安装过

已有安装 `adaptive-task-router@personal` 可以继续使用，并从其登记的本地源码更新，无需再次添加公开市场。

如果你决定切换到 GitHub 来源，先确认本地改动已保存，再在插件界面停用旧来源、安装新来源。不要同时启用两份同名技能，以免分不清载入的是哪一份。切换来源与卸载源码是不同操作，不要求删除本地仓库。

## 更新

对于本指南的 GitHub 安装：

```sh
codex plugin marketplace upgrade adaptive-task-router
codex plugin add adaptive-task-router@adaptive-task-router
```

每步成功后继续，新建对话检查新版内容。具体版本和缓存路径以命令回显为准。跟踪 `main` 意味着更新时取其最新提交；需要冻结版本的使用者应选择保存的发布包并采用个人本地安装流程，而不是假定市场的 ref 会自动覆盖插件自身的 ref。

本地维护者应先保存改动，不直接覆盖源码。业务版本后的 `+codex.<timestamp>` 是安装缓存标识，不代表模型能力或一次新功能发布。

## 卸载与回滚

图形界面可停用或卸载对应来源。CLI 用户先核对：

```sh
codex plugin remove --help
codex plugin marketplace remove --help
```

卸载插件与移除市场是不同动作；具体清理范围由宿主决定。本地源码和任务成果不应顺带删除。

需要回滚时使用已保存的源码提交或发布包，通过本地市场验证、刷新缓存并重装。原始 0.1.0、0.2.0 标签用于历史追溯，不包含本次新增的 GitHub 安装入口；首次开源发布对应 `public-2026-09-23`。详细步骤见 [维护说明](MAINTENANCE.md)。

## 常见安装问题

| 表现 | 检查与处理 |
| --- | --- |
| 没有 plugin 命令 | 核对客户端版本及插件功能；不能假定所有 Codex 版本具备相同命令 |
| 注册市场失败 | 检查 Git、网络、仓库地址及报错，不重复覆盖全局配置 |
| 插件已安装但技能未出现 | 新建对话或按客户端要求重启，再检查安装来源和回显版本 |
| 安装新包后仍见旧内容 | 确认更新的是登记源码，按宿主缓存更新流程重装 |
| 模型不可用 | 以当前工具公布的模型/档位为准；安装插件不会给账号增加权限 |
| CLI 模型调用 401 | 属于客户端认证条件；插件结构或发现成功不能解决登录问题 |
| 无子代理工具 | 保留分析/验收能力，在可执行范围内由当前代理完成并报告限制 |

本仓库的公开安装检查、结构检查与实际模型行为验证分别记录，不互相替代。
