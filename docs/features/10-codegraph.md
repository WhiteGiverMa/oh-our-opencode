# 10：CodeGraph CLI + 极简 skill

状态：已交付（2026-10-06）。用户选择原版 `@colbymchenry/codegraph` CLI + 简单使用说明，description 强调「能用时必须使用」；随后授权更新或补装 Windows/WSL CLI。独立本地仓库 `./opencode-codegraph`（相对综合工作区 `/home/celestia/proj/o3p`），未提交、发布或配置生产 skill 源。

## 交付内容

- **Skill**：`skills/codegraph/SKILL.md` 共 49 行，采用 GitHub `thkt/dotclaude` 的 MIT `use-cli-codegraph`（固定 `97372afe55b29115c65d0795c44b62279b8904dd`）精简适配；仓库与可独立复制的 skill 目录均保留完整 MIT 及 `Copyright (c) 2026 thkt`。只保留原生命令，不加脚本/插件/MCP；描述明确 CLI 与目标索引可用时 `MUST USE`，无索引不擅自初始化，纯 CLI 的索引新鲜度用显式 sync 处理。
- **现成技能调查**：skills.sh 的 `onsager-ai/dev-skills` 是同引擎 MIT 但偏 MCP；`stark-ai-de/agent-skills` 是 Apache-2.0 生命周期指南，均未采用。ClawHub `wei840222/codegraph` 是同 CLI 的较大技能，发布者声明 MIT-0，但下载包未带 LICENSE，未复制。最终采用最贴近的 GitHub 短指南；固定来源、搜索覆盖及限制见[产品 README](../../../opencode-codegraph/README.md)。
- **WSL CLI**：旧 PATH 包为 `1.0.1` 且正被 MCP 使用，保留其文件；官方 `1.6.2` 并存安装至 `~/.local/opt/codegraph/1.6.2`，新增 `~/.local/bin/codegraph` 链接，只把当前 NVM 的 CLI 链接切向新入口。原 MCP PID 与旧包路径不变，未重启承载服务。
- **Windows CLI**：原 PATH 无 CodeGraph，官方 npm 包 `1.6.2` 已安装，入口 `C:/Users/A1337/AppData/Roaming/npm/codegraph.cmd`；原 `~/.omo/codegraph` 的 `1.5.0` 留存。两端 npm 安装均固定版本、使用官方 registry、禁用生命周期脚本；未运行 `codegraph install`，未改 MCP/AGENTS/PowerShell 执行策略。

## 验收

两端真实 CLI 的帮助、缺索引错误、中文空格路径显式初始化、query/explore/node、callers/callees/impact 与 sync 全部通过。隔离真实 OpenCode v2.0.21 原生技能发现返回精确 ID/description/路径/完整正文，随后按说明执行 CLI 查询成功。无模型请求或编码会话；临时宿主/索引/项目已清理；QA JS 语法与 LSP 通过，Markdown 用原生解析替代 LSP。Windows v2 skill 发现与远端模型指令遵循未验收。

## 切换边界

本轮只交付 CLI 与 skill 源码：旧 OMO/用户 MCP、自动准备、初始化指导、进程清扫均未关闭，也未安装全局 skill。移除复制的 skill 目录即可停止技能发现；CLI 不需要 MCP 配置。旧 LTS 的初始化指导及启动清扫仍有关闭缺口——CLI 已安装不等于旧项已退出。

## 证据与人工快验

证据：`.omo/evidence/20261006-codegraph-cli-skill/README.md`（相对综合工作区）、同目录 `usage.json` 与 `qa.mjs`。

人工 v2 快验：打开 `oc2`，让 agent 读取交付的 `SKILL.md`，执行 `codegraph --version` 与带显式 `-p` 的 `explore`；原生发现安装方法见产品 README。
