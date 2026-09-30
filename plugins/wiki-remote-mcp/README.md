# wiki-remote-mcp

[boykush/wiki](https://github.com/boykush/wiki) を引くための remote MCP サーバーへの参照を1つ持つだけの package。名前の `remote` は、ローカルで `scraps mcp serve` を立てる経路と区別するためのもの。そちらは [scraps 本体の `mcp-server` plugin](https://github.com/boykush/scraps/tree/main/plugins/mcp-server) が配る。

サーバー名は `scraps`。tool は `mcp__scraps__*` として見える（`search_scraps` / `get_scrap` / `lookup_scrap_links` など）。`~/.claude/CLAUDE.md`（[dotfiles](https://github.com/boykush/dotfiles) の `agents/AGENTS.md`）の手順がこの名前を前提にしているので、変えない。

## 配る中身

`https://wiki-mcp.boykush.com/mcp` への参照だけ。実体は3つの repo に分かれている。

- **image** は wiki の CI が wiki の内容ごとビルドして `ghcr.io/boykush/wiki-mcp-server` へ push する。したがって引ける内容は **main に push 済みのもの**で、手元の未 push な編集は含まれない
- **動く場所と hostname** は [infrastructure-as-code](https://github.com/boykush/infrastructure-as-code) の `applications/remote-mcp-server/`。URL が変わるならここが先に変わる
- **参照** をこの package が配る

繋がらないときは公開サイト <https://boykush.github.io/wiki/> を見る。

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

Claude は `.mcp.json`、Codex は `.codex/config.toml` に入る。どちらも project scope なので、リポジトリに commit すればそのリポジトリで開いた全セッションに効く。

Claude の `.mcp.json` は本来リポジトリごとに承認プロンプトが出るが、`~/.claude/settings.json` の `enabledMcpjsonServers` に `scraps` が入っているので出ない。入っていないマシンでは1回承認する。
