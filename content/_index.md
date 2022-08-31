---
title: Introduction
type: docs
bookToc: false
---

# XS-Leaks Wiki
## Overview

Cross-site leaks (別名 XS-Leaks、XSLeaks) は、Web プラットフォームに組み込まれたサイドチャネル [^side-channel] に由来する脆弱性分類です。 それらは、Web サイトが相互にやり取りできるようにするという Web のコア原則を利用し、正当なメカニズム [^browser-features] を悪用してユーザーに関する情報を推測します。 XS-Leaks の見方として、クロスサイトリクエストフォージェリ (CSRF [^csrf]) 技術との類似性を強調する方法があります。XS-Leaksは、他の Web サイトがユーザーに代わってアクションを実行するのではなく、ユーザーに関する情報を推測するために使用されます。

例えばブラウザは、ウェブサイトが、サブリソースをロードしたり、ナビゲートしたり、他のアプリケーションにメッセージを送信することを可能にします。このような動作は、一般的にウェブプラットフォームに組み込まれたセキュリティメカニズム（例えば、[same-origin policy]（https://developer.mozilla.org/en-US/docs/Web/Security/Same-origin_policy））によって制限されますが、XS-Leaksはウェブサイト間の相互作用中に露出する小さな情報の断片を利用します。

XS-Leaksの原理は、ウェブ上で利用可能なこのようなサイドチャネルを利用して、他のウェブアプリケーション内のデータ、ローカル環境の詳細、接続されている内部ネットワークなど、ユーザーの機密情報を明らかにすることです。

## Cross-site オラクル

XS-Leakに使用される情報の断片は、通常バイナリ形式を持ち、「オラクル」と呼ばれます。
オラクルは一般的に、攻撃者から見える形で巧妙に用意された質問に対して*YES*または*NO*で答えます。

例えば、オラクルは次のように問われることがあります。

> 他のウェブアプリケーションでユーザーの検索結果に*secret*という単語が表示されるか？

これは、という問いかけに等しいかもしれません:

> クエリ *?query=secret* は *HTTP 200* ステータスコードを返すか？

また、[Error Events]({{< ref "./docs/attacks/error-events.md" >}})で*HTTP 200*のステータスコードを検出することができるため、この問い合わせと同じ効果となります:

> アプリケーションで *?query=secret* からリソースを読み込むと、*onload* イベントが発生するか？

上記のクエリーは、攻撃者によって多くの異なるキーワードで繰り返される可能性があり、その結果、ユーザーのデータに関する機密情報を推測するためにレスポンスが使用される可能性があります。

ブラウザは、善意でありながら、少量のクロスオリジン情報を漏らしてしまう可能性のある、さまざまなAPIを幅広く提供しています。
これらのAPIについては、このWikiの中で詳しく説明します。

## Example

ウェブサイトは、他のウェブサイトのデータに直接アクセスすることはできませんが、他のウェブサイトからリソースをロードし、その副作用を観察することは可能です。たとえば、*evil.com*は*bank.com*からの応答を明示的に読むことを禁じられていますが、*evil.com*は*bank.com*からスクリプトをロードしようと試み、それがうまくロードされたかどうかを判断することができます。

{{< hint example >}}

例えば、*bank.com* が、ある種の取引におけるユーザーの領収書に関するデータを返す API エンドポイントを持っているとする。

1. *evil.com*は、スクリプトとしてURL *bank.com/my_receipt?q=groceries*をロードしようとすることができます。デフォルトでは、ブラウザはリソースをロードする際にクッキーを添付するので、*bank.com*へのリクエストはユーザーのクレデンシャルを運ぶことになります。
2. ユーザーが最近食料品を購入した場合、スクリプトは *HTTP 200* ステータスコードで正常にロードされます。ユーザーが食料品を購入していない場合、リクエストはHTTP 404*ステータスコードでロードに失敗し、[エラーイベント]({{< ref "./docs/attacks/error-events.md" >}})がトリガーされます。
3. エラーイベントをリスニングし、異なるクエリでこのアプローチを繰り返すことで、攻撃者はユーザーの取引履歴に関するかなりの量の情報を推論することができます。
{{< /hint >}}

上記の例では、2つの異なるオリジンのウェブサイト（*evil.com* と *bank.com* ）が、ブラウザがウェブサイトの使用を許可している API を介して、相互に作用しています。 この相互作用は、ブラウザや*bank.com*の脆弱性を悪用するものではありませんでしたが、*evil.com*が*bank.com*上のユーザーのデータに関する情報を取得することを可能にしました。  

## XS-Leaksの根本的な原因

ほとんどのXS-Leaksの根本原因は、ウェブのデザインに内在するものです。多くの場合、アプリケーションは何も間違っていないのに、XS-Leaksによる情報漏洩の危険にさらされています。XS-Leaksの根本的な原因をブラウザレベルで修正することは、多くの場合、既存のウェブサイトを壊してしまうため困難です。このため、ブラウザは様々な防御機構({{< ref "defenses" >}})を実装し、この困難を克服しています。これらの防御の多くは、通常、特定の HTTP ヘッダ (例: *[Cross-Origin-Opener-Policy]({{< ref "./docs/defenses/opt-in/coop.md">}}): same-origin*) を使うことによって、より厳しいセキュリティモデルを選ぶようウェブサイトに要求していますが、望ましい結果を得るためにはしばしば組み合わせる必要があります。

XS-Leaksの情報源は、以下のように区別することができます：

1. ブラウザAPI (例： [Frame Counting]({{< ref "frame-counting.md" >}}) と [Timing Attacks]({{< ref "timing-attacks.md" >}}))
2. ブラウザの細かい実装とバグ (例：　[Connection Pooling]({{< ref "./docs/attacks/timing-attacks/connection-pool.md" >}}) や [typeMustMatch]({{< ref "./docs/attacks/historical/content-type.md#typemustmatch" >}}))
3. ハードウェアのバグ(例： Speculative Execution Attacks [^spectre])

## ちょっとした歴史

XS-Leaksは長い間ウェブプラットフォームの一部でした。ユーザーのウェブ活動に関する情報を漏らす[タイミング攻撃]({{< ref "network-timing.md" >}})は少なくとも [2000](https://dl.acm.org/doi/10.1145/352600.352606)年には知られていました。

このクラスの問題は、その影響力を高める新しいテクニックが発見されるにつれて、着実に注目を集めるようになりました[^old-wiki]。
2015年、GelernterとHerzbergは「Cross-Site Search Attacks」[^xs-search-first]を発表し、タイミング攻撃を悪用して、GoogleとMicrosoftが構築したWebアプリケーションに対して影響力の大きいXS-Search攻撃を実装しました。それ以来、さらに多くの XS-Leak テクニックが発見され、テストされています。

近年、ブラウザは様々な新しい[防御機構]({{< ref "defenses" >}}) を実装して、アプリケーションを XS-Leaks からより簡単に保護できるようにしました。

## このWikiについて

この wiki は、読者に XS-Leaks を紹介することと、XS-Leaks を悪用する経験豊富な研究者のためのリファレンス ガイドとして機能することの両方を目的としています。 この wiki にはさまざまな手法に関する情報が含まれていますが、新しい手法は常に出現しています。
 新しいテクニックを追加するか、既存のページを拡張するかに関係なく、改善は常に高く評価されます!

このウィキに貢献する方法を見つけて、 [Contributions]({{< ref "contributions.md" >}}) の記事で貢献者のリストを参照してください。

※なお、和訳版についての要望やコミットは別途Issue等にて受け付けます。


## References
[^side-channel]: Side Channel Vulnerabilities on the Web - Detection and Prevention, [link](https://owasp.org/www-pdf-archive/Side_Channel_Vulnerabilities.pdf)
[^csrf]: Cross Site Request Forgery (CSRF), [link](https://owasp.org/www-community/attacks/csrf)
[^browser-features]: In some cases, these features are maintained to preserve backwards compatibility. But, in other cases, new features are added to browsers regardless of the fact that they introduce potential cross-site leaks (e.g. [Scroll to Text Fragment]({{< ref "scroll-to-text-fragment.md" >}})), as the benefits are considered to outweigh the downsides.
[^old-wiki]: Browser Side Channels, [link](https://github.com/xsleaks/xsleaks/wiki/Browser-Side-Channels)
[^xs-search-first]: Cross-Site Search Attacks, [link](https://446h.cybersec.fun/xssearch.pdf)
[^spectre]: Meltdown and Spectre, [link](https://spectreattack.com/)
