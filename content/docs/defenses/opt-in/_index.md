---
title: "Opt-In Mechanisms"
weight: 1
---

# Opt-In Mechanisms

XS-Leak攻撃から防御するために、アプリケーションが展開できるさまざまなオプトインメカニズムがあります。
メカニズムは、防御するテクニックの点で重複する可能性があることに注意してください。

* [Fetch Metadata]({{< ref "./fetch-metadata.md" >}})を使用すると、アプリケーションは要求が開始された方法と理由を判別できるため、悪意のある要求を拒否できます。
* [Cross-Origin-Opener-Policy]({{< ref "./coop.md" >}}) を使用すると、アプリケーションは、window.openまたはwindow.openerを介した他のWebサイトとのやり取りを防ぐことができます。
* [Cross-Origin-Resource-Policy]({{< ref "./corp.md" >}}) を使用すると、アプリケーションは他のサイトに特定のリソースが含まれないようにすることができます。
* [Framing Protections]({{< ref "./xfo.md" >}}) を使用すると、アプリケーションでフレーミングを許可するサイトを定義できます。
* [SameSite Cookies]({{< ref "./same-site-cookies.md" >}}) を使用すると、アプリケーションはサードパーティサイトからのリクエストにCookieを含めるかを判断できます。

