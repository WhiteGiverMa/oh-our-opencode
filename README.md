# Oh Our OpenCode

[English](README.en.md)

构建开源、模块化、全链路透明的综合智能体环境。

综合的、模块化的 [OpenCode](https://github.com/anomalyco/opencode) 插件和智能体工具，专注于 OpenCode v2，受 [OmO](https://github.com/code-yeongyu/oh-my-openagent) 启发。

各组件可拆卸、可组装：不必为了一个不顺心的工具替换整套系统，也不必忍受插件八爪鱼抱抱式地与 OpenCode 本体耦合。

宗旨：环境即 harness，harness 由智能体身边的一切组成。

专注于 OpenCode；所有的开源智能体都是我们的盟友。若有意向请求或实现其他智能体的适配（如 [pi](https://github.com/earendil-works/pi) 或 [DeepSeek Harness](https://github.com/deepseek-ai/deepseek-harness) 等），请联系维护者。

做最坏的打算。假设每个独裁者都会犯错、会走远，包括本仓库的维护者自己。构建以及赋能构建一切的模块化替代：0 绑定，随时可退出。

赋能每个人构建高度自定义的个人智能体。

## 组件

| 仓库 | 说明 |
| --- | --- |
| [opencode-folio](https://github.com/WhiteGiverMa/opencode-folio) | 只读查询 OpenCode 会话历史的 CLI：`list` / `search` / `read` / `info`，直读 SQLite，配套 agent skill |
| [opencode-lsp](https://github.com/WhiteGiverMa/opencode-lsp) | 独立 stdio MCP 服务器，为 OpenCode v1/v2 提供 LSP 诊断、定义、引用、符号与重命名 |
| [opencode-webfetch-redirect-guard](https://github.com/WhiteGiverMa/opencode-webfetch-redirect-guard) | `webfetch` 重定向守卫：在权限边界内解析跳转链，把最终 URL 交给原生工具 |
| [opencode-write-existing-file-guard](https://github.com/WhiteGiverMa/opencode-write-existing-file-guard) | `write` 覆盖守卫：已存在的文件须先读后写，批准一次性消费 |
| [opencode-logo-pulse](https://github.com/WhiteGiverMa/opencode-logo-pulse) | TUI 插件：复活 OpenCode 早期版本主页 logo 的蓄力脉冲彩蛋 |

## 许可

所有组件均以 [MIT](https://opensource.org/licenses/MIT) 协议开源。OpenCode 是独立项目，本系列与其无隶属或背书关系。
