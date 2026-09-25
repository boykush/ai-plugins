# doc-placement

事実の**置き場**を決める順番と、既に散文へ写されている事実を棚卸しする手順を持つ package。MCP サーバーは配らず、skill `doc-placement` だけを配る。

## 目標は機械化

事実を、機械が読める1つの持ち主（宣言・設定・スキーマ・カタログの entity と edge）に置き、文書はそれを指す。写しが減るのは結果で、目的ではない。

実際に起きた drift を1週間分追うと、修正はほぼすべて「他のファイルが既に持っている事実の写し」だった。共通点は2つで、どちらも書き方ではなく置き場の問題だった。

- 機械可読な持ち主（変数・宣言・検索条件）があるのに、散文が一覧や件数を写していた
- 同じ契約が2つのリポジトリの散文に両方あり、互いに「相手が持っている」と書いていた

renderer を変えても直らない。散文を綺麗にするのではなく、持ち主を作って散文を減らす方向に効く。

## 外部に向けた文書は別

公開 repo の README や `docs/` は、その repo を**使う人**に向けたもの。CLI のオプションや設定ファイルの形式のように、持ち主がコード側にあっても書く——「詳細は `<file>` のコメント」は repo の中を読む人にしか届かない。これを skill 自身が言うので、OSS の repo に無理に当たらない。

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
