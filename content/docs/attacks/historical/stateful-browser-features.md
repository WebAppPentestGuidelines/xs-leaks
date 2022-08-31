+++
title = "Stateful Browser Features"
description = ""
date = "2020-10-01"
category = "Historical"
defenses = [
    "Browser Fix"
]
menu = "main"
+++

ブラウザの機能/拡張の中には、ブラウザによって生成された特定のウェブサイトの状態に応じて、リクエストの処理方法を変更するものがあります。攻撃者は、時に、このプロセス全体を観察し、ブラウザを混乱させ、これらの状態に対して副作用をもたらすアクションを引き起こすことができます。

## WebKit – ITP

[Intelligent Tracking Prevention](https://webkit.org/tracking-prevention/) (ITP) は、[WebKit Tracking Prevention technologies](https://webkit.org/tracking-prevention/) の一部であるプライバシー機能です。これは、いくつかの機能を組み合わせたもので、ウェブサイトが第三者のコンテキストの下でユーザーを追跡することを防ぐことを目的としています。残念ながら、残念ながら、初期の設計では新しいXS-Leak[^1]が登場し、ITPが暗黙的に作り出した状態を攻撃者が悪用して、ウェブサイトをトラッカーとすることができました。

### 根本的な原因

ITPは、Webサイトにトラッキング機能があるかどうかを分類するために、リソースの負荷や、クリック、タップ、テキスト入力など、Webサイトに対するユーザーのインタラクションに関する統計情報を収集しています。ITPは、これらの統計情報を分類し、トラッキング機能があると思われるWebサイトにはストライクを与えます。3回ストライクを受けると、そのウェブサイトは拒否リストに登録され、今後のリクエストの際にブラウザによって扱いが変更されます。

#### 問題

ITPの問題の1つは、攻撃者がITPを操作して、特定の動作を任意に強制できることです。たとえば、攻撃者はITPを操作して、あるドメインにストライクを与え、そのドメイ ンが拒否リストに入ったかどうかをチェックすることができます。この情報は、たとえば次のようなさまざまな方法で活用することができます。

- ドメインが拒否リストに入るために必要なストライクの数に基づいて、ユーザーのブラウジング習慣をリークします。
- 拒否リストを利用して、結果があるときだけクロスサイトリソースを取り込むページに対してXS-Search攻撃を実行します。

### 修正

 [この問題を修正するため](https://webkit.org/blog/9661/preventing-tracking-prevention-tracking/)、ITPは分類をやめ、すべてのサイトをデフォルトで「トラッキング」サイトと見なすようになりました。これにより、攻撃者が特定のITPの動作を検出することを可能にしていた暗黙的状況がなくなりました。

## 参考文献

[^1]: Information Leaks via Safari’s Intelligent Tracking Prevention, [link](https://arxiv.org/pdf/2001.07421.pdf)
[^2]: Preventing Tracking Prevention Tracking, [link](https://webkit.org/blog/9661/preventing-tracking-prevention-tracking/)
