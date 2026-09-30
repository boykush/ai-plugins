---
name: wiki-query
description: ユーザーが「wiki」と言って wiki を見るよう求めたときだけ使う（「wikiにあったっけ」「wiki見て」など）。個人 wiki（boykush/wiki）を MCP の scraps で探し、見つけたものをどう扱うかを持つ。「wiki」と言われていなければ、wiki に書いてありそうな話題でも引かない。
---

# wiki を引く

個人 wiki（[boykush/wiki](https://github.com/boykush/wiki)）はユーザーが書き溜めた理解の1次ソースで、MCP の `scraps` がそれを読む。

## 探す

- ユーザーは題名を覚えていないので、言い回しは曖昧になる。`search_scraps` を語を変えて何度か試す。検索の広げ方・絞り方と関連の辿り方は `scraps` の MCP の instructions に従う
- 見つからなければ「wiki にない」と明示する

## 使う

- 見つけた scrap は `[[Title]]`（ctx 付きなら `[[Ctx/Title]]`）で指し、ユーザーの理解と揃える
- wiki にある語はユーザーが把握済みなので、定義から説明し直さない
- wiki は書いた時点の理解のスナップショットで、リポジトリの現状ではない。コードと食い違ったらコードを正とし、食い違いを伝える

## 繋がらないとき

`scraps` は remote サーバーで、使えないセッションもある。そのときは公開サイト <https://boykush.github.io/wiki/> を読む。ローカルに wiki の複製は置いていないので、Grep / Read で代用しない。
