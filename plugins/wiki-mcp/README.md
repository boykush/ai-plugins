# wiki-mcp

[boykush/wiki](https://github.com/boykush/wiki) を引くための remote MCP サーバーを1つ持つだけの plugin。

サーバーの実体は `https://wiki-mcp.boykush.com/mcp`。manifest は [boykush/infrastructure-as-code](https://github.com/boykush/infrastructure-as-code)、image は wiki の CI が wiki の内容ごとビルドして GHCR へ push する。したがって引ける内容は **main に push 済みのもの**で、手元の未 push な編集は含まれない。繋がらないときは公開サイト <https://boykush.github.io/wiki/> を見る。

ローカルで scraps を動かす経路は持たない（[scraps 本体の `mcp-server` plugin](https://github.com/boykush/scraps/tree/main/plugins/mcp-server) がその役)。ここは参照先を remote の1つに保つための plugin。

## 使う

リポジトリの `.claude/settings.json`:

```json
{
  "extraKnownMarketplaces": {
    "ai-plugins": {
      "source": {
        "source": "github",
        "repo": "boykush/ai-plugins"
      }
    }
  },
  "enabledPlugins": {
    "wiki-mcp@ai-plugins": true
  }
}
```

サーバー名は `scraps`。tool は `mcp__scraps__*` として見える（`search_scraps` / `get_scrap` / `lookup_scrap_links` など）。`.mcp.json` 由来のサーバーと違って project ごとの承認プロンプトは出ない——plugin を有効にした時点で同意している扱いになる。

いつ wiki を引くかの指針は `~/.claude/CLAUDE.md`（[dotfiles](https://github.com/boykush/dotfiles) の `agents/AGENTS.md`）にある。
