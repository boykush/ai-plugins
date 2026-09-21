# adr-remote-mcp

[boykush/adr](https://github.com/boykush/adr) の決定を引くための remote MCP サーバーへの参照と、決定が持つルールを読むための skill を持つ package。名前の `remote` は、手元で adi を立てる経路と区別するため。

サーバー名は `adr`。tool は `mcp__adr__*` として見える。wiki 側が `scraps`（サーバーを提供するツールの名前）なのに対してこちらを `adr` にしたのは、package 名と hostname と揃えたかったから——ツール名で揃えるなら `adi` になる。

## 配る中身

- `https://adr-mcp.boykush.com/mcp` への参照
- skill `ade-rule-dsl`。ルールは [ADE](https://github.com/phi42/ad-enforcement-tool) の DSL で書かれていて、`list_rules` は書かれたままを返す。それを読むための DSL の reference を、サーバーと同じ package で届ける

サーバーの実体は2つの repo に分かれている。

- **image** は adr の CI が `ghcr.io/boykush/adr-mcp-server` へ push する。adr の adi に、決定と `.rule` を同梱したもの。したがって引ける内容は **main に push 済みのもの**
- **動く場所と hostname** は [infrastructure-as-code](https://github.com/boykush/infrastructure-as-code) の `applications/remote-mcp-server/adr/`。URL が変わるならここが先に変わる
- **参照** をこの package が配る

読み取り専用で、前段に認証は無い（決定は元から公開）。落ちているときは repo の `decisions/` を直接読む。

### skill の reference

`.apm/skills/ade-rule-dsl/references/` の `dsl-reference.md` と `LICENSE` は、ADE の `dsl/dsl-reference.md` と LICENSE（Apache-2.0）を写したもの。手では直さず、adr の `tools/ruledsl` が書く。版は adr の go.mod が要求する ADE に揃え、adr がルールを検査する parser と文法を合わせる。ADE を上げたら adr で書き直す。

```bash
go run ./tools/ruledsl vendor <ai-plugins>/plugins/adr-remote-mcp/.apm/skills/ade-rule-dsl/references
```

adr もこの package に依存していて、adr の `go test` が、配られてきた reference がその版のものかを確かめる。

## 使う

リポジトリの `apm.yml`:

```yaml
dependencies:
  apm:
    - boykush/ai-plugins/plugins/adr-remote-mcp
```

```bash
apm install
```

Claude は `.mcp.json` と `.claude/skills/`、Codex は `.codex/config.toml` と `.agents/skills/` に入る。どれも project scope なので、リポジトリに commit すればそのリポジトリで開いた全セッションに効く。

`~/.claude/settings.json` の `enabledMcpjsonServers` には `scraps` しか入っていないので、**Claude では初回に `adr` の承認プロンプトが出る**。出したくなければ [dotfiles](https://github.com/boykush/dotfiles) 側に `adr` を足す。
