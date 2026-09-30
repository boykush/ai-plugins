# wiki-remote-mcp

[boykush/wiki](https://github.com/boykush/wiki) を引くための remote MCP サーバーへの参照と、いつどう引くかを持つ skill の package。名前の `remote` は、ローカルで `scraps mcp serve` を立てる経路と区別するためのもの。そちらは [scraps 本体の `mcp-server` plugin](https://github.com/boykush/scraps/tree/main/plugins/mcp-server) が配る。

サーバー名は `scraps`。tool は `mcp__scraps__*` として見える（`search_scraps` / `get_scrap` / `lookup_scrap_links` など）。同梱の skill と [dotfiles](https://github.com/boykush/dotfiles) の `enabledMcpjsonServers` がこの名前を前提にしているので、変えない。

## 配る中身

- `https://wiki-mcp.boykush.com/mcp` への参照
- skill `wiki`。いつ wiki を引くかと、引いたものの扱い（[次の節](#skill-wiki)）

サーバーの実体は3つの repo に分かれている。

- **image** は wiki の CI が wiki の内容ごとビルドして `ghcr.io/boykush/wiki-mcp-server` へ push する。したがって引ける内容は **main に push 済みのもの**で、手元の未 push な編集は含まれない
- **動く場所と hostname** は [infrastructure-as-code](https://github.com/boykush/infrastructure-as-code) の `applications/remote-mcp-server/`。URL が変わるならここが先に変わる
- **参照** をこの package が配る

繋がらないときは公開サイト <https://boykush.github.io/wiki/> を見る。

### skill `wiki`

いつ引くかは SKILL.md の `description` が持つ。description は毎セッション載り、本文は呼ばれたときだけ読まれる。サーバーと同じ package に置くので、サーバーが届く所には条件も届く。

[adr-remote-mcp](../adr-remote-mcp) のように MCP の instructions でいつ使うかを言わないのは、`scraps` の instructions が [scraps](https://github.com/boykush/scraps) 本体の持ち物で、どの wiki にも同じ文が出るから。検索の広げ方・絞り方や関連の辿り方のような探し方の一般論はそちらが持ち、skill はこの wiki に固有のことだけを持つ。

## 使う

リポジトリの `apm.yml`:

```yaml
dependencies:
  apm:
    - boykush/ai-plugins/plugins/wiki-remote-mcp
```

```bash
apm install
```

Claude は `.mcp.json` と `.claude/skills/`、Codex は `.codex/config.toml` と `.agents/skills/` に入る。どれも project scope なので、リポジトリに commit すればそのリポジトリで開いた全セッションに効く。

Claude の `.mcp.json` は本来リポジトリごとに承認プロンプトが出るが、`~/.claude/settings.json` の `enabledMcpjsonServers` に `scraps` が入っているので出ない。入っていないマシンでは1回承認する。
