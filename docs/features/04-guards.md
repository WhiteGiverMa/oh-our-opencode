# 04：写文件守卫与 WebFetch 重定向守卫

状态：部分完成。写已存在文件守卫与 WebFetch 重定向守卫已实现、推送并双端部署；fsync 跳过警告、工作笔记保护、shell 读取提醒三项暂缓。验收日期 2026-10-01。

## 背景

开源搜索没有找到可直接采用的轮子：MIT 的 `opencode-workflow-guard` 含读后写策略，但捆绑了其他不能逐项关闭的策略；现有 WebFetch 相关轮子主要替换工具本身或依赖外部服务，不符合「增强原生工具」的边界。因此两个子项都是原创 MIT 实现，不依赖 OMO 运行时。GitHub 仓库默认私有，未发布 npm。

## 子项状态

| 子项 | 仓库与提交 | 状态与边界 |
| --- | --- | --- |
| 写已存在文件守卫 | `./opencode-write-existing-file-guard`；https://github.com/WhiteGiverMa/opencode-write-existing-file-guard ；`2dad58d` | 已实现、推送并部署配置。原生 `write` 覆盖已有文件前须同会话成功 `read`，一次授权一次写；新文件可写，跨会话不共享。不保护 `edit`、`apply_patch`、shell 或 MCP 写入。 |
| WebFetch 重定向守卫 | `./opencode-webfetch-redirect-guard`；https://github.com/WhiteGiverMa/opencode-webfetch-redirect-guard ；`4e8276d` | 已实现、推送并部署配置。仅在权限快照可证明自动允许时预解析，每一跳都检查权限；拒绝循环（含只改变 fragment 的循环）和超限，正常内容仍由原生工具返回。ask、deny 或不可证明的授权不预请求。 |
| fsync 跳过警告 | 未建仓库 | 暂缓（用户确认）。只有 OMO 内部生产者会触发，v2 没有公开 fsync 结果事件；独立提示 hook 观察不到真实跳过，旧钩子保留。 |
| 工作笔记保护 | 未建仓库 | 暂缓，`notepad-write-guard` 不变。 |
| shell 读取提醒 | 未建仓库 | 暂缓，`bash-file-read-guard` 不变。 |

表中仓库路径相对综合工作区 `/home/celestia/proj/o3p`。

## 部署与关闭

- WSL v1 与 Windows v1 均已追加两个插件入口，并在各自 `~/.omo/omo.jsonc` 的 `[opencode].disabled_hooks` 中只追加了 `write-existing-file-guard`、`webfetch-redirect-guard`。其他配置字段、私密模板、PTY 与 tmux 不变。
- Windows 独立 Node bundle 位于 `G:/dev/opencode-write-existing-file-guard/dist/index.js` 和 `G:/dev/opencode-webfetch-redirect-guard/dist/index.js`，与 WSL 验证产物逐字节一致。
- WSL v2 使用两个独立 `v2-entry` 薄入口；4098 实时实例已确认注册两个守卫与原 PTY。未重启、未创建实时测试会话。Windows v2 未部署。
- 未重启 WSL v1 服务或 Windows Desktop：v1 的启动时加载与旧钩子关闭要用户重启后才生效，写入配置不等于当前会话已切换。
- 五份私有配置备份为对应文件的 `.bak-20261001-modular-guards`。回退方式：移除新入口并恢复两个旧钩子。如果备份之后配置还有其他改动，不要整份覆盖恢复。

## 验收

- 写守卫 59 条、重定向守卫 74 条测试通过；严格类型检查与构建退出 0；源码 LSP 无错误。
- 本地 mock 模型驱动真实 OpenCode 原生工具：Linux v1 `1.18.34`、Linux v2 `2.0.21`、Windows 原生 v1 `1.18.34`，各跑启用/关闭两组、每组 11 场景，共 66 场景通过；比较了实际文件内容、工具结果与 HTTP 请求。
- HOME/USERPROFILE、XDG 与数据库隔离；真实会话库计数（WSL 370、Windows 571）未变化。
- Markdown 无 LSP；JSON/JSONC 的 Biome 未安装（此前已拒绝安装），用解析与真实启动验证替代。

## 证据

`.omo/evidence/20261001-modular-guards/README.md`、`summary.json`、六份 `*-release*.json`（相对综合工作区）；两个独立仓库各保留本地 `.omo/evidence/20261001-modular-guards/` 副本。证据不含真实 provider 配置、凭据、私密提示词或环境变量转储。
