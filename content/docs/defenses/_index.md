---
weight: 20
bookFlatSection: true
title: "対策の仕組み"
---

# 対策の仕組み

[XS-Leaks Attack Vectors]({{< ref "../attacks/" >}}) をすべて対策することは容易ではありません。攻撃ベクトルの一つ一つは異なるウェブやブラウザのコンポーネントに影響を与え、そしてそれには癖があります。Google VRPのような一部のバグバウンティプログラムは彼らが大規模な改修に注力するために新しいXS-Leaksの報告に対して支払いを停止しました。[^1] Googleなどの他の多くの企業はXS-Leaksを修正するには、アプリケーションがXS-Leaks全カテゴリを緩和するのに有効な [new large scale mitigations and changes to the web platform]({{< relref "_index.md#opt-in-mechanisms" >}}) に時間とエンジニアリングを投資が正しいアプローチだと信じています。

現在ブラウザはXS-Leaksを軽減するために使用できるオプトインの仕組みを多数提供しています。これらは強力な保護機能を提供しますが、すべてのブラウザでまだ十分にサポートされていないことが欠点です。XS-Leaksを効果的に対策するためには、様々な手法を組み合わせる必要があります。

## オプトインの仕組み

これらの [defense mechanisms]({{< ref "opt-in/_index.md" >}}) によってアプリケーションは類似した XS-Leaks に一括に対応することができます。これらの防御は、アプリケーションがブラウザの挙動を変更できるようにするか、あるいはアプリケーション自身の動作を変更するために使用できる追加情報を提供することです。

{{< hint tip >}}
オプトインによる防御の仕組みはデフォルトの戦略であるべきです。XS-Leaksに対してだけでなく、XSSI、クリックジャッキング、CSRFといった他の脆弱性に対しての対策にもなります。
{{< /hint >}}

{{< hint important >}}
ブラウザのサポートに依存する緩和策を使用する場合、ユーザのブラウザで十分にサポートされていることを必ず確認してください。例えば `fetch metadata` ヘッダは素晴らしいツールですが現在はChromiumベースのブラウザだけでしかサポートされていません。異なる仕様に対するブラウザサポートの最新情報については、[MDN](https://developer.mozilla.org/en-US/)を確認してください。
{{< /hint >}}

## アプリケーションの設計

アプリケーションの設計[手法]({{< ref "design-protections/_index.md" >}})はXS-Leaksをアプリケーションの設計段階で防止する方法に重点を置いています。これは、より強力で全体的な保護をすぐに有効にすることが現実的でない場合に、非常に有用なアプローチです。もう一つの大きな利点は、最新のブラウザ仕様をサポートしていない古いブラウザでも、注意深くアプリケーションを設計することでXS-Leaksを阻止できることです。

{{< hint note >}}
アプリケーションの設計技術を使って、アプリケーション全体ですべてのXS-Leak技術を防御することは極めて困難です。 アプリケーション設計の手法は深刻なリークを防ぐのに有効ですが、ブラウザが提供する[opt-in mechanisms]({{< relref "_index.md#opt-in-mechanisms" >}}) は全体的な解決策として優れています
{{< /hint >}}

## Secure Defaults

ブラウザベンダーは、このWikiで言及されている いくつかのXS-Leaksの影響を緩和するために[default behaviors]({{< ref "secure-defaults/_index.md" >}})を変更することに積極的に取り組んでいます。ブラウザのデフォルトの動作を変更することはセキュリティの向上と後方互換性維持の間でバランスをとることです。
{{< hint important >}}
セキュアデフォルトは素晴らしいものです！開発者が更に努力することなくアプリケーションとユーザを保護することができます。しかし、XS-Leaksを完全に防ぐことはできないので注意してください。
{{< /hint >}}

## 参考文献

[^1]: Google Bughunter University - XSLeaks and XS-Search, [link](https://sites.google.com/site/bughunteruniversity/nonvuln/xsleaks)
