+++
title = "postMessage Broadcasts"
description = ""
date = "2020-10-01"
category = [
    "Attack",
]
abuse = [
    "postMessage",
]
defenses = [
    "Application Fix",
]
menu = "main"
weight = 3
+++

アプリケーションは、他のオリジンと情報を共有するために、しばしば [postMessage broadcasts](https://developer.mozilla.org/en-US/docs/Web/API/Window/postMessage) を使用します。
`postMessage` を使うと、2種類の XS-Leaks につながる可能性があります。

* 信頼できない発信元と機密性の高いメッセージを共有すること
    * `postMessage` API は `targetOrigin` パラメータをサポートしており、これを使用してメッセージを受信できるオリジンを制限することができます。メッセージに機密性の高いデータが含まれている場合、このパラメータを使用することが重要です。 

* 変化するコンテンツやブロードキャストの存在に基づいた情報のリーク
    * 他の XS-Leak のテクニックと同様に、これはオラクルを形成するために使われる可能性があります。例えば、特定のユーザ名を持つユーザが存在する場合にのみ、アプリケーションが「Page Loaded」という postMessage ブロードキャストを送信すると、これを利用して情報をリークすることができます。
    
## 対策

この XS-Leak は、postMessage のブロードキャスト送信の目的に深く依存するため、明確な解決策はありません。
アプリケーションは、postMessage の通信を既知の送信元グループに制限する必要があります。
これが不可能な場合、攻撃者が通信間の違いに基づいて情報を推論するのを防ぐために、ユーザの状態に関係なく通信が一貫して同じ動作をする必要があります。


## References

[^1]: Cross-Origin State Inference (COSI) Attacks: Leaking Web Site States through XS-Leaks, [link](https://arxiv.org/pdf/1908.02204.pdf)