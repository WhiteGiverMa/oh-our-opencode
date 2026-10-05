# Oh Our OpenCode

[中文](README.md)

Building an open, modular, fully transparent agent environment.

A modular collection of [OpenCode](https://github.com/anomalyco/opencode) plugins and agent tooling, focused on OpenCode v2, inspired by [OmO](https://github.com/code-yeongyu/oh-my-openagent).

Every component is detachable and composable: no need to replace a whole system because of one disagreeable tool, and no plugins coupling themselves to the OpenCode core like an octopus hug.

The principle: the environment is the harness, and the harness is everything around the agent.

Focused on OpenCode; all open-source agents are our allies. If you'd like to request or implement adapters for other agents (such as [pi](https://github.com/earendil-works/pi) or [DeepSeek Harness](https://github.com/deepseek-ai/deepseek-harness)), please contact the maintainer.

Plan for the worst. Assume every dictator will err and stray — including this repository's own maintainer. Build, and empower others to build, modular replacements for everything: zero lock-in, exit anytime.

Empowering everyone to build a highly personal agent.

## Components

| Repository | Description |
| --- | --- |
| [opencode-folio](https://github.com/WhiteGiverMa/opencode-folio) | Read-only CLI for OpenCode session history: `list` / `search` / `read` / `info` directly over SQLite, with a companion agent skill |
| [opencode-lsp](https://github.com/WhiteGiverMa/opencode-lsp) | Standalone stdio MCP server giving OpenCode v1/v2 LSP diagnostics, definitions, references, symbols, and rename |
| [opencode-webfetch-redirect-guard](https://github.com/WhiteGiverMa/opencode-webfetch-redirect-guard) | `webfetch` redirect guard: resolves redirect chains inside the permission boundary and hands the final URL to the native tool |
| [opencode-write-existing-file-guard](https://github.com/WhiteGiverMa/opencode-write-existing-file-guard) | `write` overwrite guard: existing files must be read before written; approvals are consumed once |
| [opencode-logo-pulse](https://github.com/WhiteGiverMa/opencode-logo-pulse) | TUI plugin reviving the charge-and-pulse easter egg from early OpenCode home logos |

## License

All components are open source under the [MIT](https://opensource.org/licenses/MIT) license. OpenCode is an independent project; this collection is not affiliated with or endorsed by it.
