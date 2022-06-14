+++
title = "CSS Tricks"
description = ""
date = "2020-10-01"
category = [
    "Attack"
]
abuse = [
    "CSS"
]
defenses = [
    "Framing Protections"
]
menu = "main"
weight = 2
+++

## CSS Tricks
CSSを利用して視覚的な変化を起こすことで、ユーザーが埋め込みピクセル値などの情報を暴露するように騙すことができます。

## ユーザのヒストリーを取得する
CSSの[`:visited`](https://developer.mozilla.org/en-US/docs/Web/CSS/:visited)セレクタを使うと、訪問したことのある URL に対して異なるスタイルを適用することができます。 
以前は[`getComputedStyle()`](https://developer.mozilla.org/en-US/docs/Web/API/Window/getComputedStyle)を使ってこの違いを検出することができましたが、昨今のブラウザでは常にリンクが訪問されたかのような値を返すことでこれを防ぎ、セレクタを使って適用できるスタイルを制限しています。[^changes-1]   
そこで、CSSが効いた領域をクリックするようにユーザーを誘導する必要があるかもしれませんが、これは[`mix-blend-mode`](https://developer.mozilla.org/en-US/docs/Web/CSS/mix-blend-mode)を使って行うことができます。[^blend-mode]
また、レンダリングタイミングを悪用することで、ユーザーとのインタラクションなしに、リンクを別の色に塗り替える方法もあります。[^render-timings-bug]
複数のリンクを使用して時間差を増加させることで動作するPoCがchromium report上で提供されました。[^render-timings-bug]

{{< hint info >}} [^leak-1] shows an example of this attack using a whack a mole game to trick the user into clicking areas of the page, multiple bugs were reported about this issue: [^bug-1](https://bugs.chromium.org/p/chromium/issues/detail?id=712246), [^bug-2](https://bugs.chromium.org/p/chromium/issues/detail?id=713521), [^bug-3](https://bugzilla.mozilla.org/show_bug.cgi?id=147777){{< /hint >}}

## Captchaの悪用
CSSを使えば、埋め込みをコンテキストから外部に取り出せます。 
この例として、[^leak-2]に見られるようなcaptchaのふりをすることが挙げられます。 
これは、埋め込み部分の幅と高さを設定することで、狙った文字だけが表示されるようにするものです。
複数の埋め込みを利用して、表示される文字の順番を変えて、何の情報を提供しているのかをユーザーが分かりづらくすることも可能です。

## オートコンプリートの悪用
テキスト入力を使用するウェブサイトで、```autocomplete="off"```を使用して[オートコンプリート](https://developer.mozilla.org/en-US/docs/Web/HTML/Attributes/autocomplete)を無効にしない場合、電子メールアドレスなどのデータをリークできる場合があります。javascriptを使用したテキスト入力のオートコンプリート機能を利用するために、ユーザーを騙して、キーを押させることができます。
Chromeの場合、上または下矢印キーを押してダイアログを開き、値を選択した後、EnterキーまたはTabキーを押して値をページに挿入するようにユーザーを誘導することが必要です。

```javascript
let input = document.createElement("input");
input.type = "email";
input.autocomplete = "email";
input.name = "email";
input.size = "1";
input.style = "position:absolute;right:-500px;bottom:-21.9px";
input.onkeypress = e => {
    e.preventDefault();
}
window.onmousedown = e => {
    // マウスのクリックを無視する
    e.preventDefault();
}
input.onchange = e => {
    alert(e.srcElement.value);
    e.srcElement.value = "";
}
document.body.appendChild(input);
setInterval(() => {
    input.focus({preventScroll: true});
}, 1000);
```

## カスタムカーソル
データを直接的にリークすることはできないかもしれませんが、ユーザーを騙すのに役立つかもしれません。巨大なカーソルは、オートコンプリートダイアログや他のネイティブUIに重なるかもしれないからです。

```html
<style>
:root {
  cursor: url('https://www.google.com/favicon.ico'), auto;
}
</style>
```

## 対策

XFOは、埋め込みが攻撃されることを防ぎます。これは、(XFOによって)コンテンツ表示されなくなることで、視覚的な違いがなくなるためです。ユーザのヒストリーを取得するタイプの攻撃は、ユーザー側で防ぐしかありません。ブラウザの履歴を無効にしたり、Firefoxの場合、`about:config`パネルで`layout.css.visited_links_enabled`を`false`に設定すれことで、防ぐことができます。

| [SameSite Cookies (Lax)]({{< ref "/docs/defenses/opt-in/same-site-cookies.md" >}}) | [COOP]({{< ref "/docs/defenses/opt-in/coop.md" >}}) | [Framing Protections]({{< ref "/docs/defenses/opt-in/xfo.md" >}}) |                  [Isolation Policies]({{< ref "/docs/defenses/isolation-policies" >}})                   |
| :--------------------------------------------------------------------------------: | :-------------------------------------------------: | :---------------------------------------------------------------: | :------------------------------------------------------------------------------------------------------: |
|                                         ❌                                          |                          ❌                          |                                 ✔️                                 | ❌  |
## 参考文献
[^leak-1]: Whack a mole game, [link](https://lcamtuf.coredump.cx/whack/)  
[^changes-1]: Privacy and the :visited selector, [link](https://developer.mozilla.org/en-US/docs/Web/CSS/Privacy_and_the_:visited_selector)  
[^blend-mode]: CSS mix-blend-mode is bad for your browsing history, [link](https://lcamtuf.blogspot.com/2016/08/css-mix-blend-mode-is-bad-for-keeping.html)  
[^render-timings]: Pixel Perfect Timing Attacks with HTML5, [link](https://go.contextis.com/rs/140-OCV-459/images/Pixel_Perfect_Timing_Attacks_with_HTML5_Whitepaper%20%281%29.pdf)  
[^render-timings-bug]: Visited links can be detected via redraw timing, [link](https://bugs.chromium.org/p/chromium/issues/detail?id=252165)
[^leak-2]: The Human Side Channel, [link](https://ronmasas.com/posts/the-human-side-channel)  
