# doc-placement

文書に書く事実の**置き場**を決める基準と、既にある写しを棚卸しする手順を持つ package。MCP サーバーは配らず、skill `doc-placement` だけを配る。

## なぜ置き場なのか

エージェントに記録を任せると文書は増え、同じ事実が2箇所に書かれた時点から片方が古びる。実際に起きた drift を1週間分追うと、修正はほぼすべて「他のファイルが既に持っている事実の写し」だった。共通点は2つで、どちらも書き方ではなく置き場の問題だった。

- 機械可読な持ち主（変数・宣言・検索条件）があるのに、散文が一覧や件数を写していた
- 同じ契約が2つのリポジトリの散文に両方あり、互いに「相手が持っている」と書いていた

renderer を変えても直らない種類の問題なので、この skill が持つのは置き場の基準と、写しの見つけ方だけ。

## 規範と手順を分ける

**基準はこの skill が持ち、AGENTS.md には「文書を書くとき・消すときはこの skill の基準に従う」だけを置く。** AGENTS.md は毎セッション context に載る push の面なので短くなければならず、基準そのものを両方に書けばそれ自体が写しになる。呼ばれたときだけ読まれる skill 側は、手順やコマンドまで持てる。

command ではなく skill にしてあるのは、apm から見て command が Codex へ届かないため。

## 使う

リポジトリの `apm.yml`:

```yaml
dependencies:
  apm:
    - boykush/ai-plugins/plugins/doc-placement
```

```bash
apm install
```

Claude は `.claude/skills/`、Codex は `.agents/skills/` に入る。マシンの全セッション（リポジトリの外や `boykush` 配下以外も含む）に効かせるなら、[dotfiles](https://github.com/boykush/dotfiles) の user scope で宣言する。基準にリポジトリ固有の話は入れていないので、どちらでも効く。
