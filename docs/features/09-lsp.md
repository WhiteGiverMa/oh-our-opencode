# 09：LSP MCP

状态：完成（独立 MCP 替代）。旧 OMO 启动路由由用户同名 `mcp.lsp` 配置覆盖。

## 交付摘要

以下为较早的交接快照；2026-10-05 的 Python 依赖与启动失败修复见后文补充。

- 独立源码 `./opencode-lsp`（相对综合工作区 `/home/celestia/proj/o3p`），MIT 标准 stdio MCP，包名 `@whitegiver/opencode-lsp`，版本 `0.1.0`，Node.js 24+；main 为 `18c7e18`。七个工具：status、diagnostics、goto_definition、find_references、symbols、prepare_rename、rename，OpenCode 按 MCP 名暴露为 `lsp_*`；不含 install_decision/format。
- 已部署 WSL v1/v2 与 Windows v1；当时固定 WSL 安装在 `~/.local/opt/opencode-lsp/0.1.0-a3e4745ebda6f23f/package`，不是直接从源码工作树启动——源码搬迁不会自动热升级安装包或其他工作区进程（当前生产路径已不同，见 2026-10-05 补充）。
- 旧 OMO 启动路由由用户同名 MCP 配置覆盖。**不能**加入 `disabled_mcps:["lsp"]`：最终合并后会把新的同名 MCP 也删掉。旧配置构建函数仍执行，但旧程序不再从该路由启动。
- 交接时记录：80 测试通过、类型检查/构建通过；已修复 TS 空到空诊断超时、目标不依赖 cwd、共享后端构建域；在当时既有 v1 会话直接验收七工具、外部 `/tmp`、父目录及 `/mnt/g` 中文空格目录。这里引用的是历史证据，本轮未重跑。
- 未验收项：真实 Godot、全部语言、Windows 完整 LS 语义、v2/Windows 生产激活。交接中的拒绝记录来自旧配置迁移，不是用户最近的明确拒绝；不要据此清空或安装 LS。
- 最新交接：`~/dev/projects/oh-my-openagent/.agents/handoff/omo-lsp-feature09-live-20261004.md`；安装前历史：同目录 `omo-lsp-feature09-20261003.md`。源交付与原生验收证据见 OMO 仓库 `.omo/evidence/20261003-lsp-live/`。

## 2026-10-05：Python 依赖与启动失败修复

- 用户批准安装缺失的 Python LSP 后，在 o3p 根 `.venv` 安装 `basedpyright 1.40.2`（基于 Pyright 1.1.414），现有父目录虚拟环境启动器可用；当前会话两个 Python 文件的真实 MCP 诊断和类型检查均无错误/警告。未修改全局 LSP 配置，未给 ast-grep 分发包加依赖。
- 针对「启动器存在、对应运行时未安装却静默超时」修复了运行时的请求清理：连接关闭时及时拒绝未决请求；本地 EPIPE 按结构化错误码识别，保留服务器名、可用退出码和原始 stderr。未完成写入与关闭之间的竞态有回归保护；正常 JSON-RPC 错误中出现 EPIPE 不会误杀健康服务器，普通崩溃不会被一律标成未安装。
- 严格类型检查/构建通过，最终测试 **88/88**，修改文件 LSP 诊断为空。Oracle 首次复核指出的误判和清理两项均已修复，最终独立复核结论为无阻塞。最终指纹 `f081f0d88d872947`，版本仍为本地 `0.1.0`，未发布。
- 解包后的真实 stdio MCP 验收：缺运行时的真实 basedpyright 启动器五次 **114–119ms** 返回退出码 127 和安装原因；完全缺可执行文件 **107ms** 返回安装授权指导；普通崩溃 **124ms** 返回原始原因；健康服务器 **901ms** 返回零诊断。旧构建同场景为 **30110ms** 超时。临时 HOME/daemon 均隔离并清理，未创建编码会话或模型请求。
- 当前 WSL 生产 MCP 仍指向 `0.1.0-37995dd799ad2bf6/package`（不是旧摘要的 a3 路径）。**修复已交付到源码和本地 tgz，但运行中的生产 MCP 未切换**；未覆盖旧指纹安装目录、未重启 OpenCode、未改 Windows、未提交推送。归档和最终回执见 `.omo/evidence/20261005-lsp-startup-failure/`（相对综合工作区），安全并存激活方式见该目录 README。
