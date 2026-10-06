# 20/21/22：代理定义与提示词策略（代理大块拆分评估）

初始调查 2026-10-05。首轮为只调查，随后用户明确授权仅配置 v2 的 Meidocho 和三个研究角色。05 和 13 由其他任务负责，本项不涉及它们的状态、设计记录、实现或关闭配置；候选中的输出恢复、历史检索、归档等附带功能也不构成合并理由。

## 基线

- OMO `lts/v4.19.4` / `a997a730430b823d49e5ace9e87284402305b280`；本地 OpenCode HEAD `3104c1428ec91f809e5ab86631300de41eb6952e`（不是 v2 发布源码）。
- 本节原生关键行为已直接核对冻结 `v2.0.21` / `8a8bd622a3d7dc29ccf30ec17f84e363ed95ed72`，公开插件类型为 `@opencode/plugin@2.0.21`。
- awesome-opencode 目录快照 `bcc5c03cec19789b45ef9af4a969e3fe3fd7431b`；被收录不代表成熟度或兼容性。

## 推荐切口

主推一个独立交付块：**20 的可选角色定义 + 21 的模型家族模板 + 22 的动态追加指令**。它决定「谁工作、按实际模型给什么指令」；同步、并行、后台启动、续接及完成交付交给 OpenCode v2 原生执行。既不是把 `BackgroundManager` 搬进另一个仓库，也不是把所有 agent 相关插件重组为一个「小 OMO」。

### 最新交付状态

当前所选行为已由配置层完成，暂不创建新的 agent runtime 模块：

- v2 `2.0.21` 已热加载单一主代理 Meidocho（`*:* allow`），原生 Build/Plan disabled；Explore 覆盖为内部搜索，Librarian 为外部搜索，Oracle 为推理/调试分析/审查。
- 复用并适配原稿：移除 Context7/Grep.app 必需调用，保留 Anysearch 优先与 websearch/webfetch 类替代；保留用户确认的 Explore 宽松 shell 写能力（不是只读沙箱）。
- 原 model/provider、LSP、PTY、v1 配置与服务不动；4098 PID405 未重启，真实服务四正文 SHA 与部署文件一致。
- 恢复配置、正文和具体 diff 见 [docs/config/opencode-v2-agents/README.md](../../../docs/config/opencode-v2-agents/README.md)。原始 Kimi 草稿和 OMO 源码均未改，完整原稿/原配置私人快照已保留；敏感 persona 删除的公开 diff 只记位置和 hash，redacted patch 不是完整复原材料。
- 隔离真实 v2/local mock 通过 10 项、23 请求：主代理写、三研究角色直接编辑拒绝、Explore shell 写、Anysearch skill/webfetch、三后台交付和同 child 续接。真实远端模型的缓存命中与指令遵循未测。证据 `.omo/evidence/20261005-v2-agent-profiles/`（相对综合工作区 `/home/celestia/proj/o3p`）。

剩余运行时增量只有重新选定的逐请求模型家族模板/条件 append 等；静态文件引用、普通 reload 和原生执行不重复实现。类别、并发配额、fallback、Team、续跑与强 sandbox 是另外的选择边界，不自动回塞本块。

### 已确认的设计决定

- **模块本体采用通用策略能力，不内置固定角色集合**；角色以独立、可选的原生定义提供。术语记入 [CONTEXT.md](../../../CONTEXT.md)。当前研究优先级：Explore、Librarian、Oracle，以及原生 general 与类别化 Junior 的区别——这还不是安装或采用原版提示词的授权。
- **职责目标：研究/咨询不主动实施项目改动，允许临时资料整理；权限另保留可信工具能力。** 用户进一步明确 Explore 原版的宽松 shell（包括写）是有意设计，本次原样保留并在隔离 fixture 实测，不添加 OS sandbox。注意：直接编辑工具 deny 不阻止 shell/MCP 写入，职责描述不是技术上的不可写保障。原生定义与自定义方式见 [v2 Agents 指南](../../../ref/draft/opencode-v2-agents.md)。
- 原版研究预览已写入 [ref/draft/README.md](../../../ref/draft/README.md)：三个重点角色、全部 8 个内置类别、Junior 的 9 个 source 分支 / 10 份模型预览，以及 general 归属说明。正文由本机 OMO LTS factory/builder 渲染，保留 SUL 与完整来源；这些参考稿不进入未来 MIT 分发 payload。05/13 及真实配置不动。

## 职责边界

| 范围 | 本块承担什么 | 不承担什么 |
| --- | --- | --- |
| 20：选定代理 | 角色、description、mode、model、permissions、system；优先原生 Markdown/配置定义 | 不默认安装 OMO 全集；不抢占默认代理、不降级 build/plan；同名角色不宣称 OMO 行为等价 |
| 21：模板策略 | 引用用户模板文件、校验与渲染、按实际请求模型选择 GPT/Kimi/GLM 等家族、外部模板按需刷新 | 不将私密模板正文复制到仓库，本轮不读取或改动私密文件；不重复原生 agents 文件发现与普通文件热重载 |
| 22：动态 append | 按实际 agent/model 过滤追加内容，自己的标识与状态独立清理 | 不整体替换其他插件 system、不重写历史消息、不建立跨插件提示词框架 |
| 30：执行入口 | 使用原生按名 `subagent` 与模型/variant 选择 | 不迁移 category → junior 特殊语义、skills 预装策略或旧 `task` facade |
| 31：后台执行 | 使用原生 `background:true`、`sessionID` 续接、自动完成交付 | 不自建 `bg_*` 状态、轮询器、并发队列、后台 output/cancel 工具或重复 parent wake |
| 其他编号 | 只描述已存在的实际能力，不自动安装依赖 | 明确排除 05、13；23/24 整组、25 fallback、26/27/28 续跑、29 计划执行、32 Team、33 task 清单、35 UI、36 桥接也不随本块迁移 |

Build、Explore、General 已有原生角色；Oracle、Librarian、Meidocho 等非原生角色可以独立原创或采用来源合格的定义。用户已指定 Explore/Librarian/Oracle 为首批研究重点，最终采用的角色内容、权限和模型策略仍待逐项选择，不默认启用。OMO 的 SUL 提示词和动态能力段落只能参考行为，不能复制后重标 MIT。

```text
选定角色定义（原生 agents）
                 +
独立提示词策略（family 模板 / prompt_append）
                 |
                 v
原生 subagent(agent, model?, sessionID?, background?)
                 |
                 v
宿主 child session / Job / 完成交付 / parent wake
```

定义与动态策略可在同一交付中分别关闭；只安装静态代理文件也必须可用。v2 使用 `agent.transform` 和 session `context` 等公开接口，后者直接带实际 agent/model，无需像旧 identity hook 那样从最后一条 user message 猜当前代理。模板能力槽位须根据原生可用工具/代理原创生成；如果用户模板正文写死旧 OMO 工具名，列为迁移待确认项——不靠虚构别名或偷改私密文件掩盖差异。

## 为什么不先抽 30/31 manager

OMO 的「代理定义/提示词」与「委托执行/生命周期」是两个簇，并非所有具体代理必须跟随执行器搬迁：

- 注册侧从 `agent-config-handler` 经 builtin factories/assembly 写入 agents；模板与 append 在 `system-transform` 处理，不需要调用后台执行器。
- 委托侧查询宿主可用代理；但 category resolver 固定选择 `sisyphus-junior`，同步 `task` 与 `call_omo_agent` 也使用 manager 的并发/子代理额度，不只是异步 `task` 依赖它。
- `team_create` 绕过 `task` 工具直接 `launch`；Team 删除/状态、续跑/stop guard、fallback watchdog、TUI/tmux/OpenClaw 是其他消费者。`look_at` 另有直接创建媒体子会话的路径，不归该 manager 管。
- `BackgroundManager` 在工具过滤之前无条件构造；T 隐藏四个 task/background 工具不等于停止 manager。H `background-notification` 不仅停提醒，还会切断转发 idle/error/deleted 等事件，不能在仍保留消费者时顺手关闭。
- manager 的任务表、排队和 taskHistory 是内存状态，不能因历史在原生数据库中就宣称执行状态持久。其完成清理还会删除持久子会话；同步和后台有不同计时路径。此前后台保留诊断见[既有证据](../../../.omo/evidence/20261004-background-session-retention/README.md)，本轮不修该问题，也不将其并入 13。

因此 30/31 是可识别的执行大块，但机械抽取会带上大量与 v2 重叠的状态机、取消竞态、通知与回收责任。**只有类别语义、provider/model 并发额度等差异被确认是不可放弃的硬需求时，才重新选择这块做实现；当前不作为首刀。**

## 冻结 v2 原生能替代多少

| 行为 | `2.0.21` 已核对的源码行为 | 不应扩大的声明 |
| --- | --- | --- |
| 专用代理 | 原生 agents 的模式、模型/variant、权限、发现及文件更新 reload | 普通文件热重载不等于按运行时模型切换外部家族模板 |
| 同步、多个/并行、后台 | `subagent` 创建 child session；前台阻塞，后台立即返回；内部 Job ID 就是 child `sessionID` | 并行不等于 OMO provider/model 并发上限、队列、后代 breadth cap；原生深度默认 1 是另一约束 |
| 续接 | 输入 `sessionID` 继续既有 child；检查它属于当前 parent，可切 agent/model | 不保留 OMO `bg_*` / `task_id` 双 ID 契约，不承诺任意跨 parent 续接 |
| 完成与父会话 | 原生观察者等待 Job，通过 `session.synthetic` 交付终态文本；notification ID 用于持久 admission，交付后确认 marker | 不需要新插件再注入一次提醒；实际忙碌父会话安全边界与一次交付仍须真实使用面验收 |
| 恢复与持久性 | Job 活动 registry 在内存，但 `job.background/` KV 保存 recovery/notification marker、终态结果；不是后台全不持久 | marker 不等于通用持久并发调度器或所有 OMO 状态可重建，不能以较旧 v2 分支的纯内存行为替代此冻结版本 |
| 取消与清理 | core 有 Job cancel、session interrupt；任务 bookkeeping 与持久 child 是不同所有权 | 公开插件 Context 没有 Job start/wait/cancel，不能把 session interrupt 当作完整 `background_cancel` 替换；本块关闭不取消原生后台任务或删除会话 |

这次只是原生源码核对，不是本机 v2 后台运行验收。最小交付是原生定义与独立策略：**不注册新的 executor、结果查询、输出恢复或归档工具**。原生完成响应也不是 13 的整段会话查询/导出。

## awesome-opencode：后台轮子的实际边界

以下按固定提交核对许可证、入口与关键生命周期源码；测试只检查存在与覆盖内容，没有执行候选，候选自身的实测声明也不当成本机验收。

| 候选 / 快照 | 许可与真实 API | 完善度证据、缺口与本轮建议 |
| --- | --- | --- |
| [Background Agents](https://github.com/kdcokenny/opencode-background-agents)，`46cfb3dec517` | MIT 文件；v1 `@opencode-ai/plugin`。README 明确 Retired、V1-only、无 v2 port，并推荐原生 v2 | 有终态防回退、先保存结果再通知、压缩恢复提示；但限只读委托，15 分钟超时，无暴露 cancel/并发上限，未见测试，结果文件持久不等于运行任务态持久。不能作为完整 v2 替换；其归档/压缩功能不纳入本块 |
| [Arise（bluelovers fork）](https://github.com/bluelovers/opencode-arise)，`f667a891fd9a` | MIT 文件；v1 plugin/sdk `1.3.9`，借用 SDK `/v2` 类型不等于 OpenCode v2 插件 | 有 launch/output/retry/cancel，但完成仅 toast，不唤醒父 agent；abort 失败仍报告取消成功，manual retry 重发 description 而非原 prompt；任务仅内存，无后台并发上限。manager 测试只测高负载正则，未见生命周期覆盖。不采用为执行底座 |
| [OMO Slim](https://github.com/alvinunreal/oh-my-opencode-slim)，`16190dccf964` | MIT 文件；确有 v2 setup，类型镜像核对 `@opencode/plugin@2.0.18`，文档自述运行基线 `2.0.7` | 有 job board、concurrency/terminal-gate、cancel/revive 及对应测试，不应误判为纯 v1；但 v2 setup 复用整套 v1 factory，带入代理、策略、技能、观察与自有状态。不是独立后台轮子，不整包采用，未验证本机 `2.0.21` |
| [Pocket Universe](https://github.com/spoons-and-mirrors/pocket-universe)，`c321d8cb5145` | package 声明 MIT，未见 LICENSE/COPYING 文件；v1 API | README 标 WIP，依赖 `session.before.idle` 的 PR #9272 仍 open；#7725 已 closed 但未 merged。未见测试。「closed loop/resilient」不是验收证据，不作为可直接采用的完整轮子 |

Background Agents 与 Slim 都自述 OMO 来源，拟搬用文件仍需核实取自哪个许可时期；旧 MIT 快照并不因上游后来改 SUL 自动失效，仓库创建时间也不能证明复制了 SUL 代码。**本轮只记录来源链未决，不断言其 MIT 无效或侵权。** Arise 的错误恢复/重试能力属于本轮排除的附带范围，表格仅用它判断执行生命周期可靠性，不提出迁移这些能力。

## 其他相关插件：按职责放回边界之外

| 候选 / 快照 | 实际职责与代际 | 对本块的价值 |
| --- | --- | --- |
| [Agentic](https://github.com/Cluster444/agentic)，`3a3915310d3d` | MIT 文件；CLI 分发 Markdown agents/commands，不是后台执行器 | 可审查具体静态代理素材，不整包安装旧工作流；定义带旧 model/tools 配置，需要原生 v2 格式与角色质量核对。不能默认把 web-researcher 等同于 Librarian |
| [Agent Identity](https://github.com/gotgenes/opencode-agent-identity)，`6ed87ad92ae5` | MIT 文件；v1 system/messages hooks + attribution 查询工具，有小型测试 | 身份提示是窄扩展参考，不具备 family/template 路由；v2 context 已带 agent，无需迁移其状态推断。attribution/历史读取工具排除，不吸收进 13 |
| [CrewBee](https://github.com/CrewBeeLab/CrewBee)，`3efe39b9e28e` | MIT 文件；v1 `@opencode-ai/plugin@1.15.10`，team manifest 投影 agents + leader/member 委托，有测试 | 代理资产/投影参考，不是 OMO peer messaging/共享 board 完整替换。启动还有配置修补与 release refresh，不为代理定义一起安装这些行为 |
| [Ensemble](https://github.com/hueyexe/opencode-ensemble)，`269207ebc612` | MIT 文件；真 v1/v2 双入口，`@opencode/plugin@2.0.3`，peer 消息、SQLite board、worktrees 与 v2 测试 | 若以后选择 32，可单独隔离评估；它是团队执行 runtime，不是仅后台研究委托，不随当前块引入 dashboard/RPC/TUI 或其任务状态库；未验收本机 `2.0.21` |
| [Agent Tmux](https://github.com/AnganSamadder/opencode-agent-tmux)，`d950e44ca7de` | package/README 声明 MIT，未见 LICENSE 文件；v1-shaped event hook，用 `opencode attach` 打开已存在子会话的 pane，有测试 | 是观察适配器，不是子代理启动器或并发调度器；归 14/35 的独立界面候选，不能算后台执行已替代 |

目录里的 FlowDeck、GoopSpec、Mission Control 等整套工作流在本轮仅做目录级筛选，未逐源码评估。Swarm Plugin 链接现重定向到 `joelhooks/swarm-tools`；核对快照 `b6852530c3dc` 的入口同时注册任务、mail、memory、skills、review、queue/compaction，仍用 v1 API，只有包/README 的 MIT 声明而未找到适用主 LICENSE，亦不整包引入。shell 进程 Background/PTY、会话召回/记忆插件不是 agent runtime 替代。

## 实施前的关闭与验收门槛

1. **先选角色与模板/append 契约，再决定原创薄插件或合格定义素材。** 当前没有找到能原样覆盖 20/21/22 并满足 v2/MIT/私密模板边界的整包；不因此先建设通用 framework、installer、durable scheduler 或 pi 执行 adapter。
2. **v1 的定义/策略与执行分开关闭。** 所选代理注册、模板校验/热加载、family reconcile 与 append 都要有完整 gate；仅 `disabled_agents:["meidocho"]` 或撤去 append 配置不作为已验收总开关。保留 Team/续跑等消费者时允许旧执行器保留：不宣称 30/31 在 v1 完整退役，不全关共享 background-notification/stop guard。
3. **若未来完整退役委托执行块，必须重做依赖收敛。** 同步/异步 task、call_omo_agent、Team 的启动与状态消费者都处理后，才使 manager 不构造、不订阅、不轮询、不安排回收定时器；dispose 不能再访问它。另核对 look_at 的独立创建路径。这不是仅加一个 enabled 开关的小补丁。
4. **隔离真实 `2.0.21` 验收独立策略与原生执行的组合。** 验证代理发现/权限、实际模型切换、模板刷新/失败阻止请求、跨会话/项目隔离和其他插件内容保留；驱动一次同步、多 child 并行后台、父会话忙碌/空闲时完成与同 child 续接，确认没有重复通知。若承诺恢复语义，增加隔离重启测试，不操作现有服务。
5. **关闭只清理自身。** agent 定义与动态策略可分别停用；移除插件后没有残余指令、watcher 或外部模板状态，但已经启动的原生任务和持久会话不被本插件取消/删除。取消、类别、并发额度未覆盖的差异如实公开；05/13 始终不进入此验收范围。

## 来源与证据边界

- OMO 本地源码，路径相对 `~/dev/projects/oh-my-openagent`：注册 `packages/omo-opencode/src/plugin-handlers/agent-config-handler.ts:25`、`agent-config-assembly.ts:139`；策略 `packages/omo-opencode/src/plugin/system-transform.ts:37`、`agents/meidocho/runtime-template.ts:64`、`agents/runtime-prompt-append-reconciler.ts:169`，后两项同属该 adapter 的 `src/`。
- OMO 执行/关闭：`packages/omo-opencode/src/create-managers.ts:134`、`plugin/tool-registry.ts:80`、`tools/delegate-task/category-resolver.ts:278`、`tools/delegate-task/sync-spawn-reservation.ts:18`、`features/team-mode/team-runtime/create.ts:195`、`hooks/background-notification/hook.ts:20`；除首项外均同属该 adapter 的 `src/`。独立媒体路径：`tools/look-at/look-at-session-runner.ts:34`。
- 冻结 v2：[subagent](https://github.com/anomalyco/opencode/blob/8a8bd622a3d7dc29ccf30ec17f84e363ed95ed72/packages/core/src/tool/plugin/subagent.ts)、[Job 与 KV marker](https://github.com/anomalyco/opencode/blob/8a8bd622a3d7dc29ccf30ec17f84e363ed95ed72/packages/core/src/job.ts)、[Job/child 对接](https://github.com/anomalyco/opencode/blob/8a8bd622a3d7dc29ccf30ec17f84e363ed95ed72/packages/core/src/session/subagent-job.ts)、[完成交付](https://github.com/anomalyco/opencode/blob/8a8bd622a3d7dc29ccf30ec17f84e363ed95ed72/packages/core/src/session/subagent-completion.ts)、[agents 文件与 reload](https://github.com/anomalyco/opencode/blob/8a8bd622a3d7dc29ccf30ec17f84e363ed95ed72/packages/core/src/config/plugin/agent.ts)。公开插件 [session 类型](https://github.com/anomalyco/opencode/blob/8a8bd622a3d7dc29ccf30ec17f84e363ed95ed72/packages/plugin/src/promise/session.ts)、[agent 类型](https://github.com/anomalyco/opencode/blob/8a8bd622a3d7dc29ccf30ec17f84e363ed95ed72/packages/plugin/src/promise/agent.ts) 与本地缓存 `@opencode/plugin@2.0.21` 对照。
- 目录完整快照：[awesome-opencode README](https://github.com/awesome-opencode/awesome-opencode/blob/bcc5c03cec19789b45ef9af4a969e3fe3fd7431b/README.md)。首轮网页提取截断，最终用该提交完整源文件筛选，截断内容未当完整候选名单。
- Background Agents：[退休声明](https://github.com/kdcokenny/opencode-background-agents/blob/46cfb3dec517b8686dfb085dbff8a7a67e689480/README.md)、[入口与生命周期](https://github.com/kdcokenny/opencode-background-agents/blob/46cfb3dec517b8686dfb085dbff8a7a67e689480/src/plugin/background-agents.ts)、[MIT](https://github.com/kdcokenny/opencode-background-agents/blob/46cfb3dec517b8686dfb085dbff8a7a67e689480/LICENSE)。
- Arise：[notify/cancel/manual retry](https://github.com/bluelovers/opencode-arise/blob/f667a891fd9a65255b2a0408434de8124e51455e/src/tools/lib/background-manager.ts#L1443)、[manager 测试实际内容](https://github.com/bluelovers/opencode-arise/blob/f667a891fd9a65255b2a0408434de8124e51455e/src/tools/lib/background-manager.test.ts)、[MIT](https://github.com/bluelovers/opencode-arise/blob/f667a891fd9a65255b2a0408434de8124e51455e/LICENSE)。
- Slim：[真实 v2 setup](https://github.com/alvinunreal/oh-my-opencode-slim/blob/16190dccf964a94dabd3311bbd1456cc8f78c773/src/v2/setup.ts)、[版本与自述验收边界](https://github.com/alvinunreal/oh-my-opencode-slim/blob/16190dccf964a94dabd3311bbd1456cc8f78c773/docs/opencode-v2-compatibility.md)、[MIT 文件](https://github.com/alvinunreal/oh-my-opencode-slim/blob/16190dccf964a94dabd3311bbd1456cc8f78c773/LICENSE)。
- Pocket：[WIP/PR 依赖声明](https://github.com/spoons-and-mirrors/pocket-universe/blob/c321d8cb514579958d4bcd680c9807e9fa9f0409/README.md)、[包与 API](https://github.com/spoons-and-mirrors/pocket-universe/blob/c321d8cb514579958d4bcd680c9807e9fa9f0409/package.json)。[PR #9272](https://github.com/anomalyco/opencode/pull/9272) 与 [#7725](https://github.com/anomalyco/opencode/pull/7725) 状态为本轮 GitHub API 读取，README 的 pending 不是唯一证据。
- Agentic：[定义样例](https://github.com/Cluster444/agentic/blob/3a3915310d3d03d4a45114b7b0c0a17c34bf0e8b/agent/codebase-analyzer.md)、[MIT](https://github.com/Cluster444/agentic/blob/3a3915310d3d03d4a45114b7b0c0a17c34bf0e8b/LICENSE)；Identity：[身份提示实现](https://github.com/gotgenes/opencode-agent-identity/blob/6ed87ad92ae58bcd6c3e9a8d08f1843c6b3af5ea/src/agent-self-identity.ts)、[MIT](https://github.com/gotgenes/opencode-agent-identity/blob/6ed87ad92ae58bcd6c3e9a8d08f1843c6b3af5ea/LICENSE)。
- CrewBee：[入口与附带启动行为](https://github.com/CrewBeeLab/CrewBee/blob/3efe39b9e28ebb4da22f368366c0bc55d328dc82/src/adapters/opencode/plugin.ts)、[MIT](https://github.com/CrewBeeLab/CrewBee/blob/3efe39b9e28ebb4da22f368366c0bc55d328dc82/LICENSE)；Ensemble：[双入口](https://github.com/hueyexe/opencode-ensemble/blob/269207ebc612a6ccb37ad2188fb52c35e7787e5c/src/index.ts#L684)、[v2 setup](https://github.com/hueyexe/opencode-ensemble/blob/269207ebc612a6ccb37ad2188fb52c35e7787e5c/src/v2-setup.ts)、[MIT](https://github.com/hueyexe/opencode-ensemble/blob/269207ebc612a6ccb37ad2188fb52c35e7787e5c/LICENSE)。
- Tmux：[只观察 session.created](https://github.com/AnganSamadder/opencode-agent-tmux/blob/d950e44ca7de1b5bbd6f4804ca6f4aa56ec35eeb/src/tmux-session-manager.ts#L87)、[attach pane](https://github.com/AnganSamadder/opencode-agent-tmux/blob/d950e44ca7de1b5bbd6f4804ca6f4aa56ec35eeb/src/utils/tmux.ts#L368)、[包许可声明](https://github.com/AnganSamadder/opencode-agent-tmux/blob/d950e44ca7de1b5bbd6f4804ca6f4aa56ec35eeb/package.json)；Swarm：[大范围注册入口](https://github.com/joelhooks/swarm-tools/blob/b6852530c3dc998392ea04d5ab29e1b1342186a7/packages/opencode-swarm-plugin/src/index.ts)。

本轮没有软件实现、候选安装/测试运行、真实 provider 调用、主动创建 QA 会话、读取生产会话正文、修改私密模板/配置、重启服务、提交推送或同步 gist。研究子代理只用于只读源码调查，结果已收齐。文档通过回读、结构和来源核对验收。所有新设计只记录在文档，不写入记忆系统。
