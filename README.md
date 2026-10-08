# Oh Our OpenCode

[English](README.en.md)

构建开源、模块化、全链路透明的综合智能体环境。

综合的、模块化的 [OpenCode](https://github.com/anomalyco/opencode) 插件和智能体工具，专注于 OpenCode v2。

本项目受 [OmO](https://github.com/code-yeongyu/oh-my-openagent) 启发，站在巨人的肩膀上——衷心感谢它的探索与开拓。

各组件可拆卸、可组装：喜欢哪个装哪个，想换哪个换哪个，不必接受整套的功能捆绑。

宗旨：环境即 harness，harness 由智能体身边的一切组成。

专注于 OpenCode；所有的开源智能体都是我们的盟友。若有意向请求或实现其他智能体的适配（如 [pi](https://github.com/earendil-works/pi) 或 [DeepSeek Harness](https://github.com/deepseek-ai/deepseek-harness) 等），请联系维护者。

做最坏的打算。假设每个独裁者都会犯错、会走远，包括本仓库的维护者自己。构建以及赋能构建一切的模块化替代：0 绑定，随时可退出。

赋能每个人构建高度自定义的个人智能体。

## 组件

| 仓库 | 说明 |
| --- | --- |
| [opencode-folio](https://github.com/WhiteGiverMa/opencode-folio) | 只读查询 OpenCode 会话历史的 CLI：`list` / `search` / `read` / `info`，直读 SQLite，配套 agent skill |
| [opencode-lsp](https://github.com/WhiteGiverMa/opencode-lsp) | 独立 stdio MCP 服务器，为 OpenCode v1/v2 提供 LSP 诊断、定义、引用、符号与重命名 |
| [opencode-prompts](https://github.com/WhiteGiverMa/opencode-prompts) | 角色提示词模板：给已有角色换上自己写的完整正文，按模型挑模板，槽位严格校验 |
| [opencode-webfetch-redirect-guard](https://github.com/WhiteGiverMa/opencode-webfetch-redirect-guard) | `webfetch` 重定向守卫：在权限边界内解析跳转链，把最终 URL 交给原生工具 |
| [opencode-write-existing-file-guard](https://github.com/WhiteGiverMa/opencode-write-existing-file-guard) | `write` 覆盖守卫：已存在的文件须先读后写，批准一次性消费 |
| [opencode-logo-pulse](https://github.com/WhiteGiverMa/opencode-logo-pulse) | TUI 插件：复活 OpenCode 早期版本主页 logo 的蓄力脉冲彩蛋 |

## 友情链接

维护者环境里在用的第三方组件——感谢它们的作者。它们与上述自制组件组成了完整的Agent Harness。

| 项目 | 许可 | 说明 |
| --- | --- | --- |
| [oh-my-openagent](https://github.com/code-yeongyu/oh-my-openagent) | SUL 1.0 | OMO，v1 侧的一体化智能体层，本系列的灵感来源与迁移原点 |
| [opencode-pty](https://github.com/shekohex/opencode-pty) | MIT | PTY 会话插件：持久交互终端，替代一次性 shell |
| [opencode-supermemory](https://github.com/supermemoryai/opencode-supermemory) | MIT | Supermemory 持久记忆插件 |
| [OpenViking](https://github.com/volcengine/OpenViking) | AGPL-3.0 | `@openviking/opencode-plugin`，资源与检索增强 |
| [opencode-quota](https://github.com/slkiser/opencode-quota) | MIT | `@slkiser/opencode-quota`，额度与价格状态提示 |
| [Hindsight](https://github.com/vectorize-io/hindsight) | MIT | 仿生记忆引擎，本地 MCP 服务 |
| [Anysearch](https://api.anysearch.com) | 服务 | 通用 + 垂直域检索 API，经 MCP 接入 |
| [ast-grep](https://github.com/ast-grep/ast-grep)（[ast-grep-skill](https://github.com/code-yeongyu/ast-grep-skill)） | MIT | 结构化代码搜索改写工具；本地 `opencode-ast-grep` 为其 skill 的简化迁移 |
| [codegraph](https://github.com/colbymchenry/codegraph) | MIT | 代码知识图谱 CLI；本地 `opencode-codegraph` 为其 skill 包装 |

## 许可

所有组件均以 [MIT](https://opensource.org/licenses/MIT) 协议开源。OpenCode 是独立项目，本系列与其无隶属或背书关系。
