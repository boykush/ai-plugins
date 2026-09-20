# ai-plugins

`boykush` owner のリポジトリに横断して使う AI 向けの設定を、plugin として配る marketplace。

## 目的

MCP サーバーや skill の定義を、リポジトリごとに書き写さない。

置き場所は今まで2つしか無かった。global（`~/.mcp.json` や `~/.claude/settings.json`）に置けば全セッションに効くが、リポジトリが「これに依存している」と宣言できない。リポジトリの `.mcp.json` に置けば依存は明示できるが、同じ JSON が増えていく——wiki の MCP サーバーは実際に dotfiles と `adr` と `livt` の3箇所に同じ内容で置かれていた。

ここに置いた plugin を各リポジトリが `enabledPlugins` で参照すれば、定義は1つのまま依存だけがリポジトリ側に残る。

## 使う側

リポジトリの `.claude/settings.json` に marketplace と plugin を宣言する。リポジトリに commit するので、そのリポジトリで開いた全セッションに同じ依存が効く。

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

## 置いてある plugin

| plugin | 中身 |
| --- | --- |
| [wiki-mcp](plugins/wiki-mcp) | [boykush/wiki](https://github.com/boykush/wiki) を引く remote MCP サーバー |

## 置く / 置かない

- **置く**: 2つ以上のリポジトリで使う MCP サーバー・skill・command・agent。対象は `boykush` owner のうち fork と archive を除いたもの（[github-management](https://github.com/boykush/github-management) の fan-out と同じ範囲）
- **置かない**: マシンに紐づく設定。hooks の shell script、statusline、helix 連携のような手元の環境ありきの物は [dotfiles](https://github.com/boykush/dotfiles) に残す
- **置かない**: 特定のプロダクトの利用者に配る plugin。[scraps](https://github.com/boykush/scraps) の `llm-wiki` / `mcp-server` のように、そのプロダクトの repo が自前の marketplace で配る

## plugin を足す

1. `plugins/<name>/.claude-plugin/plugin.json` を書く。MCP サーバーを持つなら `mcpServers` に `./.mcp.json` を指す
2. `.claude-plugin/marketplace.json` の `plugins` に `{ "name": "<name>", "source": "./plugins/<name>" }` を足す
3. `plugins/<name>/README.md` に何を配るかと使い方を書く

リポジトリ自体の設定（visibility、ruleset、topics）は [github-management](https://github.com/boykush/github-management) の `repositories/ai-plugins.tf` が持つ。
