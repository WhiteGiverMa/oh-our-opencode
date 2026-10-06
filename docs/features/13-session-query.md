# 13：会话查询（四个会话工具与开源替代调查）

状态：完成。方向已收敛为「OpenCode 原样查询 CLI + 极简说明书 skill」，不做高效率记忆/跨 harness 检索。已确认决策与待定项以 [CLI 设计记录](../../../docs/todo/opencode-session-cli.md) 为准。

## Folio 交付（2026-10-05）

[Folio](https://github.com/WhiteGiverMa/opencode-folio) `0.1.0` / `c27ca4c` 已实现并公开：运行时零依赖，无服务或 SDK 前提。Linux Node 24.0/24.14 各 100/100；Windows 24.0/24.15 解包测试各 97/100（三项 file-symlink fixture 被权限拒绝，未删除/弱化）；四套 CLI 使用面及隔离原生 v2 skill 发现通过。临时环境已清理；未 npm 发布、未安装生产配置、未禁用旧工具；保留策略仍暂缓。完整矩阵及边界见产品 README 与 [13 交接](../../../.agents/handoff/o3p-feature13-folio-20261005.md)。

## 调查基线

调查日期 2026-10-04，OMO 行为基线为本地 `lts/v4.19.4` / `a997a7304`。本节是行为核对与选型记录：调查时未读取真实私密会话正文、未创建 QA 会话、未安装候选、未修改 OMO 配置或修复 OMO 会话生命周期。下文轮子调查与初始建议保留为历史，不再默认采用 deja-vu 或 recall。

## 四工具实际行为

| 工具 | 输入与默认值 | 实际结果与范围 |
| --- | --- | --- |
| `session_list` | 可选 `limit/from_date/to_date/project_path`。项目路径默认插件的 `ctx.directory`；无正数 limit 时工具端不截条数 | 只列无 parentID 的主会话，按元数据 updated 倒序；输出 ID、消息数、首末消息日期和 agent。日期筛选用最后一条消息时间，先筛日期后限条数。无消息的会话被跳过；无 cursor/offset |
| `session_read` | 必选 `session_id`；`include_todos/include_transcript` 默认 false；`limit` 默认全部；`from_end` 默认 false | 已知 ID 可读主/子会话。默认从头，正数 limit 加 from_end 才取末尾 N 条，返回仍是时间正序。正文、角色、agent、时间戳，加可选 Todo；不是无损导出 |
| `session_search` | 必选 `query`；可选 `session_id/case_sensitive/limit`；默认不区分大小写，20 条命中消息 | 只搜消息 `text` part 的字面子串，不搜工具输入/输出或 thinking，不是 regex/语义搜索。指定 ID 只搜该会话；未指定时不额外过滤项目/parentID，但只扫后端候选前 50 个，60 秒返回超时错误 |
| `session_info` | 必选 `session_id` | 聚合消息数、首末消息 ISO 时间、agents、Todo/附加 transcript 是否存在及条目数；时长至少一小时才显示。支持主/子 ID。空会话也可能显示 not found，且为聚合统计而非廉价元数据请求 |

兼容时必须分清的细节：

1. `include_transcript` 是暴露但未被 execute 使用的参数，打开也不会追加 transcript。`session_info` 的 Has Transcript 只统计 `$CLAUDE_CONFIG_DIR/transcripts/<id>.jsonl` 或 `~/.claude/transcripts/<id>.jsonl` 非空行数，与 OpenCode 自己存储的 messages 不是同一个概念。
2. `session_read` 文本本身不在 formatter 中截断，但 thinking 只保留前 200 字符、tool input 前 100、tool_result 前 200；SDK 映射也没有完整处理现代 `state.input/output` 等嵌套形状。工具日志/推理/媒体不能据此承诺原样还原，宿主还可能整体截断输出。
3. 搜索的 20 是命中消息数，不是关键词出现总数；每条计非重叠出现次数，但只展示第一个命中片段附近各 50 字符。空查询没有边界验证。无 ID 的前 50 候选来自 SDK ID 与文件 ID 合并，不保证按最近更新排序；未命中不等于整个库没有。
4. 「全局搜索」只是工具端不再过滤项目的意图，不是跨项目穷尽保证：SDK 调用是普通 `client.session.list()`，未用专用全局入口也未翻页，可能受宿主默认项目/目录/数量限制。核对的本地 OpenCode v1 源码会把插件 client 绑定 ctx.directory，列表也有默认数量上限。文件兜底可扫描存储目录里的主/子会话。后端范围与 50 会话上限都需要公开。
5. `project_path` 比较为目录字符串全等，未规范化符号链接或尾斜杠；`"/"` 被当成「不额外过滤项目」的特殊值，但不能突破 SDK 本身范围。日期上界为包含边界的精确时间；只写 `2026-10-04` 表示当天 00:00，不是当天结束。仓库搬迁不重写会话库中记录的旧 directory。
6. 存储通过 v1 SDK `session.list/messages/todo` 与旧 JSON 文件兜底，不直接 SQL 查询 SQLite。Todo 优先非空 SDK 结果，再尝试 Claude todos 文件；transcript 行数始终来自文件。仅网络/超时类错误允许 SDK 重试/文件兜底，语义 404 会抛出——不是「SDK 失败都会读旧文件」。
7. 空会话、真不存在、网络不可用的表现并不总能区分；结构化错误经 `String(error)` 可变成 `[object Object]`。最近 LSP 交接另定位到：同步 task 完成后约 10 分钟回收子会话，且不使用 background 的 1 小时配置。查询替换只能清楚报告已删除，不能使已删会话可续接；修保留策略属于另一任务。
8. 四工具返回字符串，不提供恢复、续接、删除/导入/导出写操作。无功能专属后台 manager；四个精确名称加入 OMO `disabled_tools` 可移除工具；`experimental.max_tools` 的裁剪会优先移除它们。无需顺带关闭 task/background。

典型读取用法：先 `session_list({limit:20})` 定位主会话，已知子会话 ID 直接读；`session_read({session_id:"ses_...",limit:10,from_end:true,include_todos:true})` 取结尾与任务；`session_search({query:"错误关键词",session_id:"ses_..."})` 避免未指定 ID 的候选上限。以上只是接口说明，不是实际调用过私密历史。

## 原生 OpenCode 能替代多少

- **list**：原生 v1/v2 CLI 都有 `session list --format json --max-count N`。v2 API 有 `GET /api/session`，支持范围与 cursor 分页；不要把 v1 `/session` 或旧表 SQL 套在 v2 上。
- **read**：v1 原生 `opencode export <id>`；冻结 v2 `2.0.21` 有 `session export`。v2 get/messages 可分段读取持久消息；`context` 是压缩后的活跃上下文，不等于完整历史。无需另造会话存储层。
- **info**：v2 `GET /api/session/<id>` 提供标题、位置、parentID、时间等元数据；若要 OMO 的精确消息数/agents/Todo 汇总仍需额外聚合。没有原生同名 `session_info` 模型工具。
- **search**：核对的 OpenCode core `search` 是会话标题 SQL LIKE，不是消息正文全文搜索。内容检索是本项真正的缺口，不能拿标题查询冒充。
- 版本隔离：本地 OpenCode 源码为 `3104c1428` / `v1.18.30`，内含两代模块；另核对最新远端 v2 `6721ff5328` 与已安装 `@opencode/cli@2.0.21`，三者不是同版本。parentID 查询、todo/history 端点与 SDK 命名存在差异；最终实现前必须按实际目标版本公开类型/OpenAPI 验证。本轮没有运行候选或调用生产会话 API，运行时适配未验收。
- 现有 `coding-agent-sessions` skill 可作为本机使用面参考，但 OMO 内分发无独立 MIT 许可且根为 SUL，不能复制后重标成 MIT。其 v1 DB path/旧表快路径也不等于 v2 支持。历史内容检索和 Hindsight/Supermemory 等提炼后记忆是不同层。

## 已有开源轮子

下表提交是调查时的源码快照；包自身 `2.x` 版本或 `@opencode-ai/sdk/v2` 路径不等于支持 OpenCode 2.0。

| 候选 | 许可 / 形式 | 能替代的用户行为 | v2 与采用边界 |
| --- | --- | --- | --- |
| [rmk40/opencode-session-recall](https://github.com/rmk40/opencode-session-recall)，`8a5ccd63a62e`，包 2.3.0 | MIT；OpenCode 插件，五工具 `recall_sessions/recall/recall_get/recall_context/recall_messages` | 发现会话、全文/regex/fuzzy 检索、精确消息获取、前后上下文与分页浏览；比 OMO 更能检索工具日志，且报告覆盖范围 | 原始历史经 SDK 读取，不直接查询源 DB；默认写本地派生 SQLite 索引。入口仍为 v1 `server` 与旧 hooks，依赖 SDK 内部 `_client.getConfig()`，不是原生 v2 插件；需要真实适配，不是改包名即完成。无独立 info/Todo 四工具同名对齐 |
| [vshulcz/deja-vu](https://github.com/vshulcz/deja-vu)，`c73c89f9c9dd` | MIT；Go CLI、stdio MCP、`opencode-deja` 插件与 skill，支持 OpenCode/pi 等多 harness | 跨 harness 检索、本地历史上下文、文件/错误溯源；CLI last/show/ctx 提供发现与查看，MCP context 也支持 ID | 已有明确 `opencode_v2.go`，解析 session_v2/session_message 及 v1 混合迁移库；源码记载测过 2.0.12。插件 `server.js` 提供 v1 server/v2 setup 双入口。是源码兼容证据，不是本机 2.0.21 QA；直接读 DB，需承担 schema 升级。索引脱敏并只投影部分消息/工具，不是完整无损 transcript/Todo/info 替换 |
| [AlaeddineMessadi/opencode-mcp](https://github.com/AlaeddineMessadi/opencode-mcp)，`6f1f62fd6c15` | MIT；多工具 MCP API 桥 | list/get/children/todo/status 等，但所谓 search 只匹配标题/ID | 源码走 v1 `/session`；包含 delete/share/revert/permission 等管理与写工具。不是本项小型只读替代，不建议整包引入 |
| [joeyism/opencode-history-search](https://github.com/joeyism/opencode-history-search)，`bf191ae22f2b` | MIT；插件/CLI | keyword/regex/fuzzy 与历史路径查询 | 读取 v1 SQLite/旧 JSON，无当前 v2 适配证据，列为搜索实现参考而非直接采用 |
| [kasbah/opencode-session-search](https://github.com/kasbah/opencode-session-search)，`4c190d27d774` | MIT；Rust TUI | 给人类跨项目浏览、模糊找会话 | 读取 v1 表，非模型四工具；resume/import 有写面，不优先 |

两条强候选的隐藏行为：

- **recall**：nudge 默认开；autoRecall、compactionRecall、semantic 与 LLM summaries 默认关。派生索引可删除重建；`mode:"ephemeral"` 跳过持久索引但搜索质量/覆盖会改变，不是免费的同等替代。普通检索默认下钻约 12 会话，不是任意 query 全库穷尽；deep 需要显式范围并受预算约束。启用 semantic 会下载模型；summaries 会花模型 token 并创建 worker 会话，不能为检索默认打开。
- **deja-vu**：当前 MCP 宣告一个 `deja` 工具，通过 recall/context/blame/fix/how/orient/remember mode 分派；旧工具名称仍作为兼容入口，不是七个独立宣告工具。remember 是写记忆，不是纯只读；CLI 另有安装改配置、promote/share/sync/secrets --scrub 等行为。初次试用应只用隔离 CLI/限定调用面，不运行 `install --auto`，不自动启用召回注入、远端 embedding 或跨机同步。索引脱敏不能替代保密边界，也不能当作原文无损。
- **许可排除**：OMO session-manager/内置 finder 是 SUL，仅参考行为，不复制为 MIT；[cass](https://github.com/Dicklesworthstone/coding_agent_session_search) 的 LICENSE 明写 MIT with OpenAI/Anthropic Rider，非纯 MIT，不作为本项目可移植轮子。Dashboard/记忆平台不能仅因能「找历史」就当四工具替换。

## 初始选型建议与未实施项（历史）

1. 先不写新 session 引擎。若接受「检索到历史与上下文」而非四工具名称/无损输出，**优先隔离评估 deja-vu**：已有 MIT、v2 数据解析、双代插件入口和跨 harness 能力，最有希望直接替代主要需求。是否接受 DB-schema 耦合、部分投影与附带写 mode，要在正式安装前明确。
2. 若只关注 OpenCode，并看重精确 message/context、原始数据仅通过 API 获取，**优先评估 recall 的 v1 功能与 v2 适配成本**；它更接近查询工具，但不是 v2 即装即用。不要先拷全插件连自动注入、索引/摘要后台一起移植。
3. 若最终必须保留四个工具、日期/Todo/完整历史和严格只读契约，才在实测候选缺口后决定用官方 v2 session/message API 做薄适配，复用许可合格的检索实现。先决定修正 include_transcript/no-op、搜索覆盖与错误区分，不机械复刻旧 bug。
4. 新模块的查询权限应显式区分当前会话、项目、全局，明确子代理是否纳入、缓存新鲜度、分页和部分覆盖；保留对真实已删除 ID 的清楚失败。不处理会话自动保留/恢复/续跑，除非另选对应功能。
5. 调查轮未安装/执行候选、未连接真实会话正文，候选 v1/v2/Windows QA 未通过；没有提交/推送/发布或同步 gist。

## 来源索引

- 本地 OMO（以下路径相对 `~/dev/projects/oh-my-openagent`）：`packages/omo-opencode/src/tools/session-manager/tools.ts:73`、`storage.ts:52`、`sdk-storage.ts:42`、`session-formatter.ts:44`、`session-formatter.ts:125`；注册与禁用：`src/plugin/tool-registry-core-tools.ts:131`、`src/plugin/tool-registry.ts:80`、`src/plugin/tool-registry-trimming.ts:5`，后三个 `src/` 同样位于 `packages/omo-opencode/` 下。
- 宿主 SDK 范围（相对 `~/dev/projects/opencode`）：`packages/opencode/src/plugin/index.ts:146`、`packages/opencode/src/server/routes/instance/httpapi/handlers/session.ts:64`、`packages/opencode/src/session/session.ts:955`；v2 标题搜索：`packages/core/src/session.ts:277`。
- 近期已删子会话诊断：OMO `.omo/evidence/20261003-lsp-live/session-error.md`，仅用已保存证据，未重新读真实会话。
- recall 原始 [README](https://github.com/rmk40/opencode-session-recall/blob/8a5ccd63a62e/README.md)、[package.json](https://github.com/rmk40/opencode-session-recall/blob/8a5ccd63a62e/package.json)、[入口](https://github.com/rmk40/opencode-session-recall/blob/8a5ccd63a62e/src/opencode-session-recall.ts)、[MIT](https://github.com/rmk40/opencode-session-recall/blob/8a5ccd63a62e/LICENSE)。
- deja-vu 原始 [README](https://github.com/vshulcz/deja-vu/blob/c73c89f9c9dd/README.md)、[v2 parser](https://github.com/vshulcz/deja-vu/blob/c73c89f9c9dd/internal/sources/opencode_v2.go)、[双入口](https://github.com/vshulcz/deja-vu/blob/c73c89f9c9dd/extensions/opencode/server.js)、[context ID 回归](https://github.com/vshulcz/deja-vu/blob/c73c89f9c9dd/cmd/deja/mcp_ctx_by_id_test.go)、[MIT](https://github.com/vshulcz/deja-vu/blob/c73c89f9c9dd/LICENSE)。
- opencode-mcp [session 工具](https://github.com/AlaeddineMessadi/opencode-mcp/blob/6f1f62fd6c15/src/tools/session.ts)、[MIT](https://github.com/AlaeddineMessadi/opencode-mcp/blob/6f1f62fd6c15/LICENSE)；cass [实际 LICENSE](https://github.com/Dicklesworthstone/coding_agent_session_search/blob/main/LICENSE)。
