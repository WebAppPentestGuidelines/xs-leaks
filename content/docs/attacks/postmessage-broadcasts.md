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

アプリケーションは、他のオリジンと情報を共有するために、しばしば[postMessage broadcasts](https://developer.mozilla.org/en-US/docs/Web/API/Window/postMessage)を使用する。
`postMessage` を使うと、2種類の XS-Leaks につながる可能性がある。

* 信頼できない発信元と機密性の高いメッセージを共有すること
    * `postMessage` API は `targetOrigin` パラメータをサポートしており、これを使用してメッセージを受信できるオリジンを制限することができる。メッセージに機密性の高いデータが含まれている場合、このパラメータを使用することが重要である。 

* 変化するコンテンツやブロードキャストの存在に基づいた情報のリーク
    * 他のXS-Leakのテクニックと同様に、これはオラクルを形成するために使われる可能性がある。例えば、あるアプリケーションが、与えられたユーザ名を持つユーザが存在する場合にのみ、「Page Loaded」という postMessage ブロードキャストを送信すると、これを利用して情報をリークすることができる。
    
## 対策

このXS-Leakは、postMessageのブロードキャスト送信の目的に深く依存するため、明確な解決策は存在しない。
アプリケーションは、postMessageの通信を既知の送信元グループに制限する必要がある。
これが不可能な場合、通信はユーザの状態に関係なく一貫して同じ動作をし、攻撃者が通信間の違いに基づいて情報を推論するのを防ぐべきである。


## References

[^1]: Cross-Origin State Inference (COSI) Attacks: Leaking Web Site States through XS-Leaks, [link](https://arxiv.org/pdf/1908.02204.pdf)
