# jaeger-remote-mcp

クラスタの workload が出した trace を引くための remote MCP サーバーへの参照を1つ持つだけの package。名前の `remote` は他の package に合わせたもので、port-forward で手元から引く経路は [infrastructure-as-code](https://github.com/boykush/infrastructure-as-code) の README が持つ。

サーバー名は `jaeger`。tool は `mcp__jaeger__*` として見える。読み取りだけ。

## 配る中身

`https://jaeger-mcp.boykush.com/mcp` への参照だけ。実体は [infrastructure-as-code](https://github.com/boykush/infrastructure-as-code) にある。

- **動く場所** は `applications/jaeger/`。MCP は Jaeger 自身が UI と同じポートに出す（`config.yaml` の `ai.mcp`）
- **外向きの hostname と path** は `applications/agentgateway/config.yaml`。UI の hostname（`jaeger.boykush.com`）からは MCP に入れない。URL が変わるならここが先に変わる
- **認証** は `terraform/access.tf`。Jaeger にログインは無く、前段の Cloudflare Access が token を出し、agentgateway がそれを確かめる
- **参照** をこの package が配る

## 使う

リポジトリの `apm.yml`:

```yaml
dependencies:
  apm:
    - boykush/ai-plugins/plugins/jaeger-remote-mcp
```

```bash
apm install
```

Claude は `.mcp.json`、Codex は `.codex/config.toml` に入る。どちらも project scope なので、リポジトリに commit すればそのリポジトリで開いた全セッションに効く。

## 引っかかり所

- **他の package と違って認証がある**。初回だけクライアントの OAuth（Claude なら `/mcp`）で GitHub ログインを通す。通るのは Access の owner policy に載っている本人だけなので、CI のような無人のセッションでは使えない
- クラスタは毎晩 01:37〜06:37 JST に落ちる。その間は Cloudflare が 530 を返す。trace 自体は volume に残るので、朝に戻れば引ける
