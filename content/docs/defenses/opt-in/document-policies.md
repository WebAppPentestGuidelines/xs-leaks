+++
title = "Document Policies"
category = [
    "Defense",
]
menu = "main"
+++

`Document-Policy`は、別の実験的な機能ポリシー[^1]と同様の実験的なメカニズムであり、ドキュメントの構成やドキュメント、フレームからの機能の削除（サンドボックス化）に関する機能をカバーするために使用されます。[^2]
たとえば、次の例に示すように、レスポンスヘッダーに設定できます。 

{{< hint example >}}
Document-Policy: unsized-media=?0, document-write=?0, max-image-bpp=2.0, frame-loading=lazy
{{< /hint >}}

# ForceLoadAtTop 
ForceLoadAtTop機能は、プライバシーに配慮したサイトに対して、[Scroll To Text]({{< ref "/docs/attacks/experiments/scroll-to-text-fragment.md" >}}) (と他のload-on-scroll動作) のオプトアウト機能を提供します。
この機能により、サイトは常にページの一番上に読み込まれ、テキストフラグメントやエレメントフラグメントを含むあらゆるscroll-on-loadの動作をブロックすることを示すことができます。
この機能は `Document-Policy: force-load-at-top` レスポンスヘッダーで設定することができます。

この機能は [ID Attribute]({{< ref "/docs/attacks/id-attribute.md" >}}) や [Scroll to Text Fragment]({{< ref "/docs/attacks/experiments/scroll-to-text-fragment.md" >}}) などの攻撃を防止するために有用です。

# References
[^1]: Document-Policy proposal, [link](https://github.com/wicg/document-policy/blob/main/document-policy-explainer.md)
[^2]: Feature Policy, [link](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Feature-Policy)
[^3]: Document-Policy: force-load-at-top, [link](https://www.chromestatus.com/feature/5744681033924608)
