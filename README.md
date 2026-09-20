# ai-plugins

`boykush` owner のリポジトリに横断して使う AI 向けの設定を、[apm](https://github.com/microsoft/apm) の package として配る置き場。

## 目的

MCP サーバーや skill の定義を、リポジトリごとに書き写さない。

置き場所は今まで2つしか無かった。global（`~/.mcp.json` や `~/.codex/config.toml`）に置けば全セッションに効くが、リポジトリが「これに依存している」と宣言できない。リポジトリの `.mcp.json` に置けば依存は明示できるが、同じ JSON が増えていく——wiki の MCP サーバーは実際に dotfiles と `adr` と `livt` の3箇所に同じ内容で置かれていた。

ここに package として1つ置き、各リポジトリは依存の宣言だけを持つ。クライアントごとの設定ファイルは呼び出し元で `apm install` が生成する。**Claude だけでなく Codex もリポジトリ単位になる**のがこの方式の効きどころで、Codex の plugin marketplace はマシン単位（`~/.codex/config.toml`）にしか効かないため、そちらでは同じことができない。

この repo が持つのは package の定義だけ。クライアント向けの設定は置かない（展開は呼び出し元の仕事）。

## 使う側

リポジトリに `apm.yml` を置く。

```yaml
name: adr
targets:
  - claude
  - codex
dependencies:
  apm:
    - boykush/ai-plugins/plugins/wiki-remote-mcp#v0.1.0
```

```bash
apm install
```

生成されるのは Claude 向けの `.mcp.json` と Codex 向けの `.codex/config.toml`（どちらも project scope）、それに `apm.lock.yaml`。**生成物も commit する**。そうしておけば apm を持たないセッションでもそのまま効き、apm が要るのは依存を更新するときだけになる。`apm_modules/` だけ gitignore する。

版は tag で固定する。`#v0.1.0` のほか `#^0.1.0` のような semver range も書ける。

呼び出し元の CI で生成物を守るなら、**消してから作り直して比べる**。

```bash
rm -f .mcp.json .codex/config.toml apm.lock.yaml && apm install && git diff --exit-code
```

`apm install --frozen` + diff では足りない。apm は同名の MCP サーバーを見つけると `already configured` として中身を直さないので（`--force` でも直らない）、手で書き換えられた URL を見逃す。`--frozen` は `apm.yml` と lockfile のズレを見る用途に留まる。

## 置いてある package

| package | 中身 |
| --- | --- |
| [wiki-remote-mcp](plugins/wiki-remote-mcp) | [boykush/wiki](https://github.com/boykush/wiki) を引く remote MCP サーバー |
| [adr-remote-mcp](plugins/adr-remote-mcp) | [boykush/adr](https://github.com/boykush/adr) の決定を引く remote MCP サーバー |

## 置く / 置かない

- **置く**: 2つ以上のリポジトリで使う MCP サーバー・skill。対象は `boykush` owner のうち fork と archive を除いたもの（[github-management](https://github.com/boykush/github-management) の fan-out と同じ範囲）
- **置かない**: マシンに紐づく設定。hooks の shell script、statusline、helix 連携のような手元の環境ありきの物は [dotfiles](https://github.com/boykush/dotfiles) に残す。いつ wiki を引くかのような方針も dotfiles の `agents/AGENTS.md`（`~/.claude/CLAUDE.md`）が持つ
- **置かない**: 特定のプロダクトの利用者に配る物。[scraps](https://github.com/boykush/scraps) の `llm-wiki` / `mcp-server` のように、そのプロダクトの repo が自前で配る

## package を足す

1. `plugins/<name>/apm.yml` を書く。MCP サーバーなら `dependencies.mcp` に `{ name, registry: false, transport, url }`
2. `plugins/<name>/README.md` に何を配るかと使い方を書く
3. `mise run check` を通す。使い捨ての consumer を temp に作って `apm install` し、宣言した URL が `.mcp.json` と `.codex/config.toml` の両方に落ちることを見る（CI もこれを回す）

## Agent Plugins bundle

`mise run pack plugins/<name>` で [Agent Plugins 1.0.0](https://agent-plugins.org/specification) 形式（`plugin.json` + `mcp.json`、transport は `streamable-http`）を `build/` に出せる。今のところ Claude Code も Codex もこの形式から MCP サーバーを読まず、apm もこの形式の package は copilot ターゲットにしか展開しないので、配布の主経路にはしていない。可搬な形で渡したいときだけ使う。

## 引っかかり所

- `~/.apm` が実体の無い symlink だと apm は `Refusing symlinked lifecycle lock path` で起動を拒否する
- この repo が private の間、呼び出し元の `apm install` は git 認証を要求する。visibility は [github-management](https://github.com/boykush/github-management) の `repositories/ai-plugins.tf` が持つ
