# o3p 运行时插件命名

维护者约定，只覆盖自有运行时插件。第三方插件、独立 CLI、MCP 服务器和 skill 不改名。

## 规则

- 插件 ID：`o3p.<domain>.<feature>`，全小写，点分层，功能名内多词用连字符。
- `domain` 按功能领域划分，不按 server/TUI 或 v1/v2 实现划分；同一插件的各入口共用 ID。
- `prompt` 为提示词模板，`tool` 为原生工具增强，`terminal` 为终端交互与显示。
- npm 包、仓库和目录仍采用 `opencode-<feature>`，不随运行时 ID 改名。
- 自有 TUI 命令使用 `<plugin-id>.<action>`。命令 ID 是独立标识，宿主不会自动添加插件前缀。
- `opencode.*` 留给宿主；不为第三方项目冒用 `o3p.*`。

## 当前 ID

| 仓库 | 原 ID | 当前 ID | 入口 |
| --- | --- | --- | --- |
| `opencode-prompts` | `opencode-prompts` | `o3p.prompt.templates` | v2 server |
| `opencode-write-existing-file-guard` | `opencode-write-existing-file-guard` | `o3p.tool.write-existing-file-guard` | v1/v2 server |
| `opencode-webfetch-redirect-guard` | `webfetch-redirect-guard` | `o3p.tool.webfetch-redirect-guard` | v1/v2 server |
| `opencode-logo-pulse` | `local.logo-pulse` | `o3p.terminal.logo-pulse` | v1 TUI |
| `opencode-suspend-guard` | 已采用新规范 | `o3p.terminal.suspend-guard` | v2 TUI |

Suspend Guard 的确认命令为 `o3p.terminal.suspend-guard.confirm`。

## 加载与开关

包名或目录决定加载入口，ID 决定身份和开关。配置按顺序处理；选择器不会安装包，且仅影响对应 server/TUI 配置中的插件。

```json
{
  "plugins": [
    "/absolute/path/opencode-webfetch-redirect-guard/v2-entry",
    "-o3p.tool.*",
    "o3p.tool.webfetch-redirect-guard"
  ]
}
```

`-o3p.*` 禁用此前加载的所有自有插件；`-o3p.tool.*` 按领域禁用；最后的精确 ID 可重新启用已经加载的插件。

## 身份迁移边界

ID 改名需要同步插件管理器中的精确开关与通配规则。包路径不变时，无需为改 ID 重写包路径。

宿主插件存储按 ID 分区；有持久化状态的插件必须明确迁移数据，不能把新 ID 下的空数据当作迁移完成。这五个插件没有使用宿主 ID 分区存储，不迁移或直接编辑会话数据库。

提示词的 `<<<opencode-prompts|...>>>` 标记、日志名、错误前缀、User-Agent、原生工具名称及 OMO `disabled_hooks` 都不是运行时插件 ID，本轮保持不变。保留标记确保已管理的角色正文仍可恢复和重新渲染。

本地切换先备份配置并在隔离副本构建验证，再部署完整产物；不在当前会话中重启承载它的服务。既有插件保留包路径与选项，不增加旧 ID 别名。

提示词种子只在请求准入时生成，内存状态不跨插件世代保留。运行中不能在模型或工具循环内热卸载该插件：等待受影响会话通过 `session.wait` 到达空闲，并确认没有活跃会话，再切换完整产物。之后由新的正常用户请求准入重新生成种子；不能把跳过准入的 synthetic/resume 通知当作安全恢复路径。
