---
title: "Isolation Policies"
weight: 3
---

# Isolation Policies

このセクションでは、異なる種類のクロスサイトインタラクションに対する、_isolation policies_ の形で提示された対策について説明します:

* [Fetch Metadata]({{< ref "../opt-in/fetch-metadata.md">}})を使った共通のリソース（スクリプト、画像、フェッチなど）に対するクロスサイトリクエストを防御するには、[Resource Isolation Policy]({{< ref "./resource-isolation.md" >}})を確認してください。
* [Fetch Metadata]({{< ref "../opt-in/fetch-metadata.md">}})でクロスサイトのフレーミングから守るには、[Framing Isolation Policy]({{< ref "./framing-isolation.md" >}})を確認してください。
* [Fetch Metadata]({{< ref "../opt-in/fetch-metadata.md">}}) を使ったクロスサイト ナビゲーショナル リクエストを防御するには、[Navigation Isolation Policy]({{< ref "./navigation-isolation.md" >}})を確認してください。
* [Fetch Metadata]({{< ref "../opt-in/fetch-metadata.md">}})、SameSite Cookie、またはRefererヘッダーのいずれかによるすべてのクロスサイトの相互作用を防御するには、[Strict Isolation Policy]({{< ref "./strict-isolation.md" >}})を確認してください。
