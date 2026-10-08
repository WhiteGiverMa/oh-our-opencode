# Oh Our OpenCode

[中文](README.md)

Building an open, modular, fully transparent agent environment.

A modular collection of [OpenCode](https://github.com/anomalyco/opencode) plugins and agent tooling, focused on OpenCode v2.

This project is inspired by [OmO](https://github.com/code-yeongyu/oh-my-openagent) and stands on the shoulders of that giant — heartfelt thanks for its exploration and pioneering work.

Every component is detachable and composable: install the ones you like, swap out the ones you don't, without taking on a whole bundle.

The principle: the environment is the harness, and the harness is everything around the agent.

Focused on OpenCode; all open-source agents are our allies. If you'd like to request or implement adapters for other agents (such as [pi](https://github.com/earendil-works/pi) or [DeepSeek Harness](https://github.com/deepseek-ai/deepseek-harness)), please contact the maintainer.

Plan for the worst. Assume every dictator will err and stray — including this repository's own maintainer. Build, and empower others to build, modular replacements for everything: zero lock-in, exit anytime.

Empowering everyone to build a highly personal agent.

## Components

| Repository | Description |
| --- | --- |
| [opencode-folio](https://github.com/WhiteGiverMa/opencode-folio) | Read-only CLI for OpenCode session history: `list` / `search` / `read` / `info` directly over SQLite, with a companion agent skill |
| [opencode-lsp](https://github.com/WhiteGiverMa/opencode-lsp) | Standalone stdio MCP server giving OpenCode v1/v2 LSP diagnostics, definitions, references, symbols, and rename |
| [opencode-prompts](https://github.com/WhiteGiverMa/opencode-prompts) | Per-agent prompt templates: replace an existing agent's system prompt with your own text, pick templates per model, strict slot validation |
| [opencode-webfetch-redirect-guard](https://github.com/WhiteGiverMa/opencode-webfetch-redirect-guard) | `webfetch` redirect guard: resolves redirect chains inside the permission boundary and hands the final URL to the native tool |
| [opencode-write-existing-file-guard](https://github.com/WhiteGiverMa/opencode-write-existing-file-guard) | `write` overwrite guard: existing files must be read before written; approvals are consumed once |
| [opencode-logo-pulse](https://github.com/WhiteGiverMa/opencode-logo-pulse) | TUI plugin reviving the charge-and-pulse easter egg from early OpenCode home logos |

## Friend links

Third-party components in use in our environment — thanks to their authors.

| Project | License | Notes |
| --- | --- | --- |
| [oh-my-openagent](https://github.com/code-yeongyu/oh-my-openagent) | SUL 1.0 | OMO, the integrated agent layer on the v1 side; the inspiration and migration origin of this collection |
| [opencode-pty](https://github.com/shekohex/opencode-pty) | MIT | PTY session plugin: persistent interactive terminals instead of one-shot shells |
| [opencode-supermemory](https://github.com/supermemoryai/opencode-supermemory) | MIT | Supermemory persistent-memory plugin |
| [OpenViking](https://github.com/volcengine/OpenViking) | Apache-2.0 | `@openviking/opencode-plugin`, resource and retrieval augmentation |
| [opencode-quota](https://github.com/slkiser/opencode-quota) | MIT | `@slkiser/opencode-quota`, quota and pricing status toasts |
| [Hindsight](https://github.com/vectorize-io/hindsight) | MIT | Biomimetic memory engine, served as a local MCP |
| [Anysearch](https://api.anysearch.com) | service | General and vertical search API, connected over MCP |
| [ast-grep](https://github.com/ast-grep/ast-grep) ([ast-grep-skill](https://github.com/code-yeongyu/ast-grep-skill)) | MIT | Structural code search and rewrite; the local `opencode-ast-grep` is a simplified migration of its skill |
| [codegraph](https://github.com/colbymchenry/codegraph) | MIT | Code knowledge-graph CLI; the local `opencode-codegraph` wraps it as a skill |

## License

All components are open source under the [MIT](https://opensource.org/licenses/MIT) license. OpenCode is an independent project; this collection is not affiliated with or endorsed by it.
