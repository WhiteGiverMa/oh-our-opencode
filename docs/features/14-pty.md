# 14：改用 PTY 与关闭旧交互工具

状态：完成（PTY 替代）。2026-10-05 用户明确要求关闭 `interactive_bash`，交互终端改用 PTY。本轮只改了当前 WSL 的 `~/.omo/omo.jsonc` `[opencode]`。

## 配置变更

- 新增 `disabled_tools: ["interactive_bash"]`；在既有 `disabled_hooks` 末尾追加 `interactive-bash-session`，保留所有原禁用项。
- 未修改 `tmux.enabled`、Team/子代理面板、PTY 插件入口、普通 `bash`、私密模板或其他工具。`tmux.enabled:false` 不是移除旧工具的开关，本次也不需要。
- OMO 对各层 `disabled_tools`/`disabled_hooks` 做集合并集，项目或 profile 不能通过另一组列表重新启用用户层禁用项。已用只读 `validatePluginConfig` 校验配置有效、用真实 `filterDisabledTools` 验证过滤；去掉本轮两项后的规范化配置 SHA-256 与修改前一致，其他有效设置未变化，Team 仍为 true。

## 验收

- 当前会话实际调用了 `pty_spawn/write/read/list/kill`：Node REPL 的 `6 * 7` 输出独立结果 `42`，帮助与故意错误输入正常；未知 PTY ID 明确拒绝。只清理了本轮创建的会话，列表恢复为空，进程退出。
- 配置在 OMO 插件加载时读取，不热重载工具/钩子；**本轮没有重启承载当前会话的服务**——已写入配置不等于运行中的旧工具已从列表消失。本会话立即遵守使用 PTY 的指令；即使在同一 serve 上新建会话，也不保证重新加载该插件。
- Windows 默认配置 `C:/Users/A1337/.config/opencode/opencode.jsonc` 未发现 PTY 入口，自动插件目录为空，预期 `G:/dev/opencode-pty` 目录不存在。本轮未扩大范围安装或修改 Windows 端，Windows 的 PTY 切换未完成。

## 证据与人工快验

证据：`.omo/evidence/20261005-pty-switch/README.md`（相对综合工作区 `/home/celestia/proj/o3p`）。JSONC 的 Biome LSP 未安装，用真实 OMO JSONC 解析、schema 校验和使用面实测替代。原 PTY v1/v2 验收是 2026-10-01 历史，本轮重验仅为当前 WSL 会话。

人工 v2 快验：打开 `oc2`，让 agent 用 PTY 创建 Node REPL、计算 `6 * 7`，再明确清理该会话；不需要恢复 `interactive_bash`。
