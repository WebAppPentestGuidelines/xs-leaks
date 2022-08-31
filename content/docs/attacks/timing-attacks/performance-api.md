+++
title = "Performance API"
description = ""
date = "2020-10-01"
category = [
    "Attack",
]
abuse = [
    "window.performance",
]
defenses = [
    "SameSite Cookies",
    "CORB",
]
menu = "main"
weight = 2
+++
## Performance API
[`Performance API`](https://developer.mozilla.org/en-US/docs/Web/API/Performance)は、[`Resource Timing API`](https://developer.mozilla.org/en-US/docs/Web/API/Resource_Timing_API)のデータによって強化されたパフォーマンス関連の情報へのアクセスを提供します。このAPIは、持続時間のようなネットワークリクエストの時間情報を提供しますが、サーバーから送られた`Timing-Allow-Origin: *`ヘッダーがある場合、転送サイズとドメイン検索時間も提供されます。 
このデータには、 [`performance.getEntries`](https://developer.mozilla.org/en-US/docs/Web/API/Performance/getEntries)または[`performance.getEntriesByName`](https://developer.mozilla.org/en-US/docs/Web/API/Performance/getEntriesByName)を使ってアクセスすることができます。また、[`performance.now()`](https://developer.mozilla.org/en-US/docs/Web/API/Performance/now)の差分を使って実行時間を取得することもできますが、これはミリ秒しか提供しないため、Chromeのfetchでは精度が低い可能性があります。

## ネットワークのduration
リクエストのNetwork durationを `performance` API から取得することができます。

以下のスニペットはリクエストを実行し、200ms後に `performance` オブジェクトから持続時間を取得します。

```javascript
async function getNetworkDuration(url) {
    let href = new URL(url).href;
	// duration = 0 のリクエストがあるため、fetch()の代わりに画像を使用する。
    let image = new Image().src = href;
	// performance.getEntriesByName()に追加されるリクエストを待ちます。
    await new Promise(r => setTimeout(r, 200));
    // 最後に追加された時間を取得する
    let res = performance.getEntriesByName(href).pop();
    console.log("Request duration: " + res.duration);
    return res.duration
}

await getNetworkDuration('https://example.org');
```
{{< hint info >}} 他のブラウザと異なり、Firefoxはミリ秒単位で測定値を提供します。 {{< /hint >}}

## X-Frame-Optionsを検知する
埋め込み内にページを表示する場合 (たとえば、`X-Frame-Options` ヘッダーのため)、Chrome の `performance` オブジェクトに追加されません。

```javascript
async function isFrameBlocked(url) {
    let href = new URL(url).href;
	// この関数が実行される前に、このURLに対するリクエストがあるかもしれません。
    let start_count = performance.getEntriesByName(href).length;
    let embed = document.createElement('embed');
    embed.setAttribute("hidden", true);
    embed.src = href;
    document.body.appendChild(embed);
	// performance.getEntriesByName()に追加されるリクエストを待ちます。
    await new Promise(r => setTimeout(r, 1000));
	// テスト用エンベッドの削除
    document.body.removeChild(embed)
    return performance.getEntriesByName(href).length === start_count;
}

await isFrameBlocked('https://example.org');
```
{{< hint note >}} この手法はChromiumベースのブラウザでのみ有効なようです。 {{< /hint >}}

# キャッシュされたリソースを検知する

`performance`API を使用すると、リソースがキャッシュされたかどうかを検出することができます。
[Cross-Origin Read Blocking]({{< ref "../../defenses/secure-defaults/corb.md" >}})が発動されない限り（リソースはhtml）、チェックの過程でリソースはキャッシュされます。 
```javascript
async function ifCached2(url) {
    let href = new URL(url).href;
    await fetch(href, {mode: "no-cors", credentials: "include"});
    // performance.getEntriesByName()に追加されるリクエストを待ちます。
    await new Promise(r => setTimeout(r, 200));
    // 最後に追加された時間を取得する
    let res = performance.getEntriesByName(href).pop();
    console.log("Request duration: " + res.duration);
    // 304かどうかチェックする
    if (res.encodedBodySize > 0 && res.transferSize > 0 && res.transferSize < res.encodedBodySize) return true
    if (res.transferSize > 0) return false;
    if (res.decodedBodySize > 0) return true;
    // Timing-Allow-Origin ヘッダがない場合、duration を使用する。
    return res.duration < 10;
}
```

## 通信速度

オクテット単位で通信速度を測定することができます。
```javascript
async function getSpeed(count = 10) {
    var total = 0;
	// 複数回リクエストを行い、平均値を取得する
    for (let i = 0; i < count; i++) {
        // キャッシュをバイパスして現在のオリジンにリクエストを行う
        await fetch(location.href, {cache: "no-store"});
        // 追加されるタイミングを待つ
        await new Promise(r => setTimeout(r, 200));
        // locationの最新のタイミングを取得する
        let page = window.performance.getEntriesByName(location.href).pop();
        // レスポンスタイムをtransferSizeで割って取得
        total += (page.responseEnd - page.responseStart) / page.transferSize;
    }
    // リクエストの平均レスポンスタイムを取得
    return total/count
}

await averageSpeed = getSpeed();
```
