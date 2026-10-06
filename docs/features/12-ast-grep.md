# 12：AST-grep 简化迁移

状态：完成交付，未部署。2026-10-05 用户确认「简化迁移」：不原样 vendor、不另造常驻插件。独立本地仓库 `./opencode-ast-grep`（相对综合工作区 `/home/celestia/proj/o3p`），尚无提交或远端；本轮未安装生产 skill、未改 OMO 配置、未关闭旧项、未发布包、未同步 gist。

## 来源与许可

采用独立 [ast-grep-skill](https://github.com/code-yeongyu/ast-grep-skill) 固定 `3148c69c370a51afb661b9f37879c0bd7cf0cc3b` 的 MIT 内容。仓库和可单独分发的 skill 目录均逐字保留 `Copyright (c) 2026 Yeongyu Kim` 及完整 MIT 文本；来源与改动映射见 [SOURCE.md](../../../opencode-ast-grep/SOURCE.md)，skill 自带 NOTICE。未复制 OMO-only 补丁、SUL hook/提示词或二进制。

## 保留与删除

- **保留**：精简 skill 的工具选择、metavariable/引用/语言规则、常见配方、YAML 关系/context/selector 和排错流程；可选 `scripts/rewrite.py` 保留完整预览、匹配/文件计数、1-based 位置、默认不写与显式 `--apply`。
- **删除**：自动安装/下载、OMO/Codex 缓存探测、误判合法代码的正则式校验、残缺 JSON 抢救、固定语言清单和只是转发原生 CLI 的多命令包装。真实解析交给 `sg`；外部 JSON 必须完整且字段合法才允许进入写入阶段。

## 原生接入与关闭

整个 `skills/ast-grep` 目录可复制到项目 `.opencode/skills/ast-grep`，无需 plugin 配置；移除该目录/链接即不再发现。注意：本机 OMO 的 S `ast-grep` 按名字过滤所有发现来源，会连新同名 skill 一起隐藏，不能用它做「只关旧来源」的切换；H `ast-grep-sg-provision` 单独控制旧自动准备。当前仅交付源码，OMO 运行时未切换。

## 验收

真实 ast-grep 0.43.0 与 0.45.0 各 16/16 回归通过；Python 编译通过；basedpyright 类型检查 0 errors / 0 warnings；两个 Python 文件的真实 MCP LSP 诊断为空。在 tmux 中驱动隔离 OpenCode v2.0.21：原生 `GET /api/skill` 的 ID/路径/完整正文、发现后脚本 `--help`、预览不写、显式应用、非法语言不写及原生 AST 复查全部通过。未创建编码会话或模型请求；临时宿主已停、数据目录已清理。Windows 及其他 v2 版本未做完整运行验收。

## 证据与备注

证据：`.omo/evidence/20261005-ast-grep-simplified/usage.json`（相对综合工作区）、同目录 `qa.mjs` 和 README；安装/人工 v2 快验路径见[产品 README](../../../opencode-ast-grep/README.md)。

本轮另获授权在 o3p 根 `.venv` 安装 `basedpyright 1.40.2`，使原有按父目录查找虚拟环境的启动器可用——这不是 skill 的依赖，未修改全局 LSP 配置或重启 OpenCode 服务。LSP 启动失败的错误传播改进作为 09 的后续处理（见 [09 记录](./09-lsp.md)），不并入本 skill。
