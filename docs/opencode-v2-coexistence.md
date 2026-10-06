# OpenCode v1/v2 双版本并存

官方说明与源码核对过的并存事实、安装与配置方案、迁移回退要点。调查基准：`@opencode/cli`、`@opencode/plugin`、`@opencode/client` `2.0.21`，源码 tag `v2.0.21` 对应 `8a8bd622a3d7dc29ccf30ec17f84e363ed95ed72`。

## 已确认事实

- 官方明确说明：v1 和 v2 的默认命令都叫 `opencode`，v2 curl 安装器会替换 v1；不能直接运行默认升级命令来实现并存。
- 应保留 v1 安装与现有 `opencode` 命令，为 v2 设置独立安装目录和命令名。当前 `@opencode/cli@2.0.21` 注册 `opencode` 与 `opencode2` 两个 bin；不能因此全局安装，因为它仍会接管 `opencode`。
- v2 从与 v1 相同的全局与项目位置读取配置，受支持的 v1 格式只在内存中规范化，不重写源文件。
- 长期同时使用 v1 时，共享基础配置应保留受支持的 v1 格式；native v2 格式放入 v2 专属层，不直接改写共享源。
- v1 插件实现不在 v2 中运行；必须分离两代插件清单，不能认为共享基础配置意味着旧插件自动迁移。
- v2 默认使用共享用户后台服务，支持 `--standalone` 私有服务模式；初期验证不连接或干扰现有 v1 服务。
- v2 的终端配置为全局 `cli.json`，首次启动可迁移支持的全局 `tui.json` 项且不改变 v1 源文件。
- 共用配置不等于共用数据库。默认不让两代进程并发读写同一会话数据库，数据迁移另外备份和验证。
- v2 默认打开与 v1 同名的 `opencode.db`，启动时自动迁移旧 `session` 数据。在第一次 v2 启动前就必须设置隔离，而不是试启动后再补。
- v2 凭据存入 SQLite，legacy `auth.json` 可在数据库迁移中导入；v1/v2 OAuth 更新不会自动双向同步。共用配置不能当作共用认证状态。

## 并存安装方案（尚未执行）

当前 WSL 采用本地包目录，不覆盖全局 bin；下面只是已核对参数和包元数据的操作方案，不是安装或启动的完成证据。

```bash
mkdir -p "$HOME/.local/opt/opencode-v2/2.0.21"
bun add --cwd "$HOME/.local/opt/opencode-v2/2.0.21" --trust --exact @opencode/cli@2.0.21
```

v2 二进制入口由本地包创建在 `$HOME/.local/opt/opencode-v2/2.0.21/node_modules/.bin/opencode2`。后续给它配置独立启动 wrapper，例如 `opencode-v2`；现有 `opencode` 保持 v1，不全局安装 v2，也不使用默认 curl 升级。
Windows 端需独立安装目录与对应环境变量，不能直接复制 WSL 路径；本轮不对其安装/运行作验证声明。

## 一份基础配置，两份版本层

推荐的逻辑布局（要在用户确认后从现有配置拆出，当前未改文件）：

```text
~/.config/opencode/common.jsonc        共享源：受支持的 v1 格式，不放两代插件清单
~/.config/opencode/opencode.jsonc      v1 专属项与 plugin 清单
~/.config/opencode-v2/opencode.jsonc   v2 专属项与 plugins 清单
~/.config/opencode-v2/cli.json         v2 终端配置
```

- v1 wrapper 通过 `OPENCODE_CONFIG` 读取 common，同时保留现有全局 v1 目录。
- v2 wrapper 通过 `OPENCODE_CONFIG_DIR` 替换全局目录，再通过 `OPENCODE_CONFIG` 读取同一个 common。
- 该最小布局中版本层只保存插件和不重叠的版本专属键；同名普通设置放在 common。若以后需要覆盖同一语义值，用最终 `OPENCODE_CONFIG_CONTENT` 层或重新明确文件顺序——较早加载的 v2 全局层不是最高优先级。
- common 不含 `plugin` / `plugins`，所以两代插件只在各自版本层配置；不要把含有 v1 插件清单的旧全局文件直接拿来当 common。
- OMO 在 config hook 里动态注册的代理/命令/工具不会因为共用 common 就出现在 v2；它们必须选做迁移或转成原生文件定义。
- 从现有文件抽出 common 时，必须同时处理 v1 服务的显式配置环境；不能只删旧全局字段却让现有服务失去设置。服务重启由用户操作。

v2 初次验证启动示例（必须先准备上述文件并确认 common 不含旧插件清单）：

```bash
OPENCODE_CONFIG_DIR="$HOME/.config/opencode-v2" \
OPENCODE_CONFIG="$HOME/.config/opencode/common.jsonc" \
OPENCODE_CONFIG_PROJECT_DISABLE=1 \
XDG_DATA_HOME="$HOME/.local/share/opencode-v2" \
XDG_STATE_HOME="$HOME/.local/state/opencode-v2" \
XDG_CACHE_HOME="$HOME/.cache/opencode-v2" \
"$HOME/.local/opt/opencode-v2/2.0.21/node_modules/.bin/opencode2" --standalone
```

XDG 指向的是父目录，v2 再追加 `opencode`；上述 DB 预计落在 `~/.local/share/opencode-v2/opencode/opencode.db`，用相同环境执行 `debug paths db` 核实。`--standalone` 避免误连共享服务；全量隔离 state/cache/data 比只换 DB 更稳妥。
首轮临时禁用项目配置意味着项目 `opencode.json[c]` 和 `.opencode` 定义不会被加载。完成逐项目插件审计后再放开；这不是长期丢弃项目设置的承诺。

## 配置覆盖的三个坑

1. v2 `plugins` 是操作列表，跨层累积，不是替换；`plugins:[]` 无法清除旧列表。`["-*"]` 会禁用可禁用插件，但不应靠它证明旧插件源文件从未被解析。需要启用新插件时，把其 entry 放在 remove 后。
2. v2 同时发现 `plugin/` 与 `plugins/`；只改文件夹名称不能隔离两代。最稳妥的是不同全局目录，首轮跳过项目配置；两项 policy/Console 内建插件不能被 remove。
3. v1 的 `OPENCODE_CONFIG_DIR` 是附加目录，v2 是替换目录；不能同设一个值却期待相同语义。`OPENCODE_HOME` 在调查的运行时代码中不是可用隔离入口。

v2 来源顺序：wellknown、全局目录、`OPENCODE_CONFIG`、项目根文件、项目 `.opencode` 文件、`OPENCODE_CONFIG_CONTENT`。

## 迁移与回退

- 长期并存时保留 common 的 v1 格式，v2 在内存中规范化。确认 v1 不再需要时，才把 common 转为 native v2。
- 会话迁移如有需要，用 SQLite 一致性 backup 在独立数据目录生成副本，再让 v2 迁移该副本；不链接数据库，不在 WAL 写入期间直接复制主 DB 单文件。
- 认证优先让两边使用同一组环境变量引用的 API key；OAuth 各自登录或通过受控 export/import 迁移，敏感导出不写入仓库或 QA 证据。
- 不要在并存期执行默认 `opencode uninstall`，其配置和数据目录跨版本共享。
