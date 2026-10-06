# OpenCode v2 插件开发约束与实施闭环

适用于 o3p 各独立插件的开发规范：v2 API 约束、每个选中功能的实施闭环，以及 OMO 关闭路径的历史源码索引。调查基准：`@opencode/plugin@2.0.21`，冻结源码 `v2.0.21` / `8a8bd622a3d7dc29ccf30ec17f84e363ed95ed72`。

## v2 兼容性的实施约束

- 使用 `@opencode/plugin`，不是用旧 `@opencode-ai/plugin` 类型套 v2 名称。
- 服务端使用稳定插件 id 与 `Plugin.define({ id, setup(ctx) })`；资源在 cleanup 释放。
- 工具通过 `ctx.tool.transform` 注册；before/after 通过 `ctx.tool.hook`。
- 系统提示词、模型可见消息、语义请求选项通过 `ctx.session.hook("context")`；原始 provider HTTP 修改用适当的 request hook。
- 注册前准备数据；transform callback 必须同步、便宜、可重放，不能在 callback 中做一次性安装或进程启动。
- v2 的 `prompt` hook 在 durable admission 前运行；续跑必须使用 v2 持久 inbox / session API，不能复制旧 `promptAsync` 时序假设。
- 模型请求、compaction、generate、title 是不同使用面；不能假定一个 context hook 覆盖所有模型请求种类。
- TUI 组件另走 `@opencode/plugin/tui` 和 `cli.json`，可与服务端入口在同仓库分开。
- v2 已有 `subagent(background:true)`、`shell(background:true)`、原生 MCP/skills、AGENTS 目录注入。只补用户选中的 OMO 差异。
- v2 不提供原生 LSP；`instructions` 数组虽被接受，当前文档明确说尚未解析成模型指令；`CLAUDE.md` 也不是 fallback。规则能力需以真实请求验证。
- beta 文档有版本偏差：介绍页的下载示例仍为 `2.0.6`，发布包为 `2.0.21`；构建文档部分旧依赖示例与当前类型不一致。以冻结的包类型和实测为准。
- `https://opencode.ai/config.json` 在调查时仍呈 v1 schema，不能拿它单独证明 native v2 配置有效。
- 每个插件 README 写清精确实测版本；只有覆盖实际版本矩阵后才扩大兼容声明。

## 每个选中功能的实施闭环

### 1. 确定行为边界

记录功能目的、明确排除项、当前 LTS 路径和关闭开关、v2 原生重叠能力、输入输出与状态所有权。
如果用户选择了高度耦合的能力，先说明依赖，再决定同仓库作为一个插件还是拆成可独立安装的插件。

### 2. 先保证 LTS 可关闭

- 复用现有配置控制，分别追踪工具、hook、命令、初始化和后台进程；只隐藏工具不等于完整关闭。
- 缺少完整关闭路径时，写出逐文件计划，在任务专属 worktree 给 LTS 增加默认保留现状的最小开关。
- 通过真实 v1 隔离运行验证默认行为仍在、关闭后对应行为消失，无残余注入或功能专属初始化。
- 关闭配置属于 OMO 配置视图；应注明实际 `omo.json[c]` / `[opencode]` / profile 层，不能放进 OpenCode 严格 schema 的任意顶层。
- 当前 LTS 的 `disabled_*` 列表在层间执行集合合并，空数组不会撤销父层禁用；回退需恢复实际配置差异。

### 3. 单仓库实现并验证 v2

仅实现所选行为，记录支持的 OpenCode v2 精确版本和 API；不直接沿用 v1 hook/SDK。
验证配置、类型、相关行为测试、构建与真实使用面，并记录启用、关闭、出错和清理行为。
修改系统提示词或消息时捕获实际请求；涉及续跑、后台与团队时验证活动会话保护和重复触发去重。
涉及 UI 时真实驱动并保存视觉证据；涉及工具时通过真实会话调用工具；涉及 MCP 时验证真实连接和关闭。

### 4. 切换与回退

按版本分别维护插件清单，提供替代关系、LTS 关闭配置、新插件启用配置和回退步骤。
若实现也支持 v1，额外验证 v1 中 LTS 关闭与替代插件启用的组合，确保没有重复工具或重复注入。
不擅自启用真实配置、不重启当前服务、不公开发布；由用户决定实际切换。

### 5. 保存证据并更新进度

代码和 QA 证据位于各任务所属仓库；LTS 补丁使用其工作区与 QA 规则。
证据必须写明测试动作、观察结果、隔离证明、覆盖范围、残余风险与被省略的敏感内容。
计划文档只保留功能状态、仓库路径、版本、LTS 替代开关与证据索引，不复制私密提示词或凭据。

## 附：OMO 历史关键源码索引

以下路径均相对 OMO 仓库 `~/dev/projects/oh-my-openagent`：

- 全局开关 schema：`packages/omo-opencode/src/config/schema/oh-my-opencode-config.ts:43`；应用：`packages/omo-opencode/src/testing/create-plugin-module.ts:279`。
- hook 注册：`packages/omo-opencode/src/plugin/hooks/create-tool-guard-hooks.ts:65`、`create-session-hooks.ts:83`、`create-transform-hooks.ts:39`、`create-continuation-hooks.ts:44`。
- 工具 exposure filter：`packages/omo-opencode/src/plugin/tool-registry.ts:80`；它不是 manager/init 总开关。
- MCP 与 bootstrap：`packages/omo-opencode/src/mcp/index.ts:54`、`packages/omo-opencode/src/hooks/codegraph-bootstrap/hook.ts:231`。
- skill 与兼容发现：`packages/omo-opencode/src/plugin/skill-context.ts:121`、`packages/omo-opencode/src/plugin-handlers/command-config-handler.ts:93`。
- runtime 模板与 append：`packages/omo-opencode/src/plugin/system-transform.ts:37`、`packages/omo-opencode/src/agents/builtin-agents/meidocho-agent.ts:50`、`packages/omo-opencode/src/testing/create-plugin-module.ts:299`。
- 未 gated 参数层：`packages/omo-opencode/src/plugin-interface.ts:43`；消息修复：`packages/omo-opencode/src/plugin/messages-transform.ts:252`。
- 两个 compaction 总处理器：`packages/omo-opencode/src/testing/create-plugin-module.ts:349`。
- background manager：`packages/omo-opencode/src/create-managers.ts:134`；通知控制：同文件 `:196`。
- Monitor / Goal / Team 的工具 gate：`packages/omo-opencode/src/plugin/tool-registry-gated-tools.ts:42`、`tool-registry-core-tools.ts:145`、`tool-registry-team-tools.ts:18`。
- 当前 OpenCode 的 `ulw-loop` CLI 只是 Codex passthrough：`packages/omo-opencode/src/cli/runtime-commands.ts:49`。不应从旧目录存在推断它是现行 OpenCode runtime feature；当前目标续跑为 Goal。
