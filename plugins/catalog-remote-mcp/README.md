# catalog-remote-mcp

[boykush/github-management](https://github.com/boykush/github-management) の `catalog/`——repo をまたいで作用する関係のカタログ——を引くための remote MCP サーバーへの参照を1つ持つだけの package。名前の `remote` は他の2つに合わせたもので、手元で Backstage を立てる経路は用意していない。

サーバー名は `catalog`。tool は `mcp__catalog__*` として見える（`catalog.query-catalog-entities` / `catalog.get-catalog-entity` / `catalog.get-catalog-model-description` / `catalog.validate-entity`。名前に plugin の `catalog.` が付く）。読み取りだけで、entity を登録・削除する action は配信側で出していない。

## 配る中身

`https://catalog-mcp.boykush.com/mcp` への参照だけ。実体は2つの repo に分かれている。

- **中身** は github-management の `catalog/`。image（`ghcr.io/boykush/github-management-catalog`）に build されるので、引けるのは **main に push 済みのもの**
- **動く場所** は [infrastructure-as-code](https://github.com/boykush/infrastructure-as-code) の `applications/backstage/`。Backstage が catalog を読んで関係を解決し、read-only の action に絞った MCP として出す
- **外向きの hostname と path** は infrastructure-as-code の `applications/agentgateway/config.yaml`。URL が変わるならここが先に変わる
- **認証** は infrastructure-as-code の `terraform/access.tf`。前段の Cloudflare Access が token を出し、agentgateway がそれを確かめる。初回だけ `/mcp` から GitHub でログインする
- **参照** をこの package が配る

## 何のために引くか

repo の中だけを見て「この変更はここで閉じている」と判断しないため。全 repo に配られる設定、決定の適用範囲、MCP サーバーの提供元と利用先が entity と関係として入っているので、**着手前に「誰に届くか」を聞ける**。

## 使う

リポジトリの `apm.yml`:

```yaml
dependencies:
  apm:
    - boykush/ai-plugins/plugins/catalog-remote-mcp
```

```bash
apm install
```

Claude は `.mcp.json`、Codex は `.codex/config.toml` に入る。どちらも project scope なので、リポジトリに commit すればそのリポジトリで開いた全セッションに効く。

## 引っかかり所

- クラスタは毎晩 01:37〜06:37 JST に落ちる。その間は Cloudflare が 530 を返す。catalog を見たいだけなら Backstage の UI（`https://backstage.boykush.com`、Cloudflare Access のワンタイム PIN）か、github-management の `catalog/` を直接読む
- `~/.claude/settings.json` の `enabledMcpjsonServers` に `catalog` は入っていないので、**Claude では初回に承認プロンプトが出る**。出したくなければ [dotfiles](https://github.com/boykush/dotfiles) 側に足す
