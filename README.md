# ai-plugins

`boykush` owner のリポジトリに横断して使う AI 向けの設定を、[apm](https://github.com/microsoft/apm) の package として配る置き場。

## 目的

MCP サーバーや skill の定義を、リポジトリごとに書き写さない。

置き場所は今まで2つしか無かった。global（`~/.mcp.json` や `~/.codex/config.toml`）に置けば全セッションに効くが、リポジトリが「これに依存している」と宣言できない。リポジトリの `.mcp.json` に置けば依存は明示できるが、同じ JSON が増えていく——wiki の MCP サーバーは実際に dotfiles と `adr` と `livt` の3箇所に同じ内容で置かれていた。

ここに package として1つ置き、各リポジトリは依存の宣言だけを持つ。クライアントごとの設定ファイルは呼び出し元で `apm install` が生成する。**Claude だけでなく Codex もリポジトリ単位になる**のがこの方式の効きどころで、Codex の plugin marketplace はマシン単位（`~/.codex/config.toml`）にしか効かないため、そちらでは同じことができない。

この repo が持つのは package の定義だけで、配る物のクライアント向け設定は置かない（展開は呼び出し元の仕事）。例外は root の `apm.yml` で、これはここで開くセッション自身が catalog を引くためのもの——何がどの repo に届くかは catalog の `consumesApis` に書いてある。

## 使う側

リポジトリに `apm.yml` を置く。

```yaml
name: adr
targets:
  - claude
  - codex
dependencies:
  apm:
    - boykush/ai-plugins/plugins/wiki-remote-mcp
```

```bash
apm install
```

生成されるのは Claude 向けの `.mcp.json` と Codex 向けの `.codex/config.toml`、skill を持つ package なら Claude 向けの `.claude/skills/` と Codex 向けの `.agents/skills/`（どれも project scope）、それに `apm.lock.yaml`。**生成物も commit する**。そうしておけば apm を持たないセッションでもそのまま効き、apm が要るのは依存を更新するときだけになる。`apm_modules/` だけ gitignore する。

ref を書かなければ default branch を解決し、**版は lockfile が commit で固定する**。上げるのは `apm update`。tag を打つ運用にするなら `#v0.1.0` で固定、`#^0.1.0` のような semver range も書ける——range は tag に対して解決されるので、tag が無ければ使えない。

呼び出し元の CI で生成物を守るなら、**消してから作り直して比べる**。

```bash
rm -f .mcp.json .codex/config.toml && apm install && git diff --exit-code
```

`apm install --frozen` + diff では足りない。apm は同名の MCP サーバーを見つけると `already configured` として中身を直さないので（`--force` でも直らない）、手で書き換えられた URL を見逃す。`--frozen` は `apm.yml` と lockfile のズレを見る用途に留まる。

lockfile は消さない。消すと apm は手で直された skill を「apm の管理外のファイル」として飛ばし、書き戻さない。残しておけば skill は lockfile の内容に書き戻され、MCP の設定は package から作り直されるので、どちらのズレも diff に出る。

## 置いてある package

配っている package は [`plugins/`](plugins) にある。何を配るかはそれぞれの `apm.yml` の `description` が、使い方はそれぞれの README が持つ。[上流の plugin から作った package](#上流の-plugin-から作る) は README を持たず、先頭の `upstream` 行が指す plugin の README が使い方を持つ。

## 置く / 置かない

- **置く**: 2つ以上のリポジトリで使う MCP サーバー・skill。対象は `boykush` owner のうち fork と archive を除いたもの（[github-management](https://github.com/boykush/github-management) の fan-out と同じ範囲）
- **実体は置かない**: 中身と一緒に変わる repo がある物。その repo が [Agent Plugins](https://agent-plugins.org/specification) の形で持ち、ここには apm の package にした生成物だけを置く（[上流の plugin から作る](#上流の-plugin-から作る)）
- **置かない**: マシンに紐づく設定。hooks の shell script、statusline、helix 連携のような手元の環境ありきの物は [dotfiles](https://github.com/boykush/dotfiles) に残す。いつ wiki を引くかのような方針も dotfiles の `agents/AGENTS.md`（`~/.claude/CLAUDE.md`）が持つ
- **置かない**: 特定のプロダクトの利用者に配る物。[scraps](https://github.com/boykush/scraps) の `llm-wiki` / `mcp-server` のように、そのプロダクトの repo が自前で配る

## package を足す

中身と一緒に変わる repo があるなら、ここには書かずに[上流の plugin から作る](#上流の-plugin-から作る)。ここで書くのは、`doc-refactor` のように持ち主の repo が無いものだけ。

1. `plugins/<name>/apm.yml` を書く。MCP サーバーなら `dependencies.mcp` に `{ name, registry: false, transport, url }`。skill なら `plugins/<name>/.apm/skills/<skill>/SKILL.md` に置く
2. `plugins/<name>/README.md` に何を配るかと使い方を書く
3. `mise run check` を通す。使い捨ての consumer を temp に作って `apm install` し、宣言した URL が `.mcp.json` と `.codex/config.toml` の両方に、skill が `.claude/skills/` と `.agents/skills/` の両方に落ちることを見る（CI もこれを回す）

## 上流の plugin から作る

中身と一緒に変わる repo が Agent Plugins の形で置いた plugin を、apm の package に書き直す。apm は Agent Plugins の package を copilot にしか展開しない（Claude と Codex を target にすると install が止まる）ので、その変換をここが持つ。上流の repo は apm を知らなくてよい。

```bash
mise run generate boykush/<repo>/plugins/<name>   # main の HEAD で取り込む
mise run generate                                # 取り込み済みの全部を、固定した commit で書き直す
```

書かれるのは `plugins/<name>/apm.yml` だけで、変換は次の2つ。

- `mcp.json` の各サーバーを `dependencies.mcp` に。transport は `streamable-http` を `http` に読み替える。`type` と `url` 以外を持つ宣言は扱えず、止まる
- `skills/` の下の各 skill を `dependencies.apm` に、上流の repo のその skill の場所として1つずつ。plugin の root を指すと Agent Plugins の package として扱われ、何も展開されない

上流の commit は、先頭の `upstream` 行と各 skill の `#<sha>` が固定する。Renovate は他の apm の依存と同じくその SHA を上げ、CI が同じ commit から書き直して差分を見る。上流で skill や接続先が変わっていればそこで止まるので、`mise run generate` を回して commit する。

## Agent Plugins bundle

`mise run pack plugins/<name>` で [Agent Plugins 1.0.0](https://agent-plugins.org/specification) 形式（`plugin.json` + `mcp.json`、skill があれば `skills/`。transport は `streamable-http`）を `build/` に出せる。今のところ Claude Code も Codex もこの形式から MCP サーバーを読まず、apm もこの形式の package は copilot ターゲットにしか展開しないので、配布の主経路にはしていない。可搬な形で渡したいときだけ使う。

## 引っかかり所

- `~/.apm` が実体の無い symlink だと apm は `Refusing symlinked lifecycle lock path` で起動を拒否する
