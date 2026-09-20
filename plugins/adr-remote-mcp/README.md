# adr-remote-mcp

[boykush/adr](https://github.com/boykush/adr) の決定を引くための remote MCP サーバーへの参照を1つ持つだけの package。名前の `remote` は、手元で adi を立てる経路と区別するため。

サーバー名は `adr`。tool は `mcp__adr__*` として見える。wiki 側が `scraps`（サーバーを提供するツールの名前）なのに対してこちらを `adr` にしたのは、package 名と hostname と揃えたかったから——ツール名で揃えるなら `adi` になる。

## 配る中身

`https://adr-mcp.boykush.com/mcp` への参照だけ。実体は2つの repo に分かれている。

- **image** は adr の CI が `ghcr.io/boykush/adr-mcp-server` へ push する。[ad-guidance-tool](https://github.com/boykush/ad-guidance-tool) の fork からビルドした adi に、決定と `.rule` を同梱したもの。したがって引ける内容は **main に push 済みのもの**
- **動く場所と hostname** は [infrastructure-as-code](https://github.com/boykush/infrastructure-as-code) の `applications/remote-mcp-server/adr/`。URL が変わるならここが先に変わる
- **参照** をこの package が配る

読み取り専用で、前段に認証は無い（決定は元から公開）。落ちているときは repo の `decisions/` を直接読む。

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

Claude は `.mcp.json`、Codex は `.codex/config.toml` に入る。どちらも project scope なので、リポジトリに commit すればそのリポジトリで開いた全セッションに効く。

`~/.claude/settings.json` の `enabledMcpjsonServers` には `scraps` しか入っていないので、**Claude では初回に `adr` の承認プロンプトが出る**。出したくなければ [dotfiles](https://github.com/boykush/dotfiles) 側に `adr` を足す。
