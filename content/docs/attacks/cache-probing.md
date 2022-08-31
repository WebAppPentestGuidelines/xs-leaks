+++
title = "Cache Probing"
description = ""
category = "Attack"
abuse = [
    "window.open",
    "Error Events",
    "Cache",
    "iframes",
    "AbortController"
]
defenses = [
    "SameSite Cookies",
    "Vary: Sec-Fetch-Site",
    "Subresource Protections",
]
menu = "main"
weight = 2
+++

 Cache Probingは、あるリソースがブラウザによってキャッシュされているかどうかを検知する手法です。そのコンセプトは、初期のWeb[^4]から知られており、当初はタイミングの差異を検知することに基づいていました。

ユーザがWebサイトに訪れると、画像、スクリプト、そしてHTMLコンテンツなどの様々なリソースが取得され、それらは(特定の条件下で)ブラウザによってキャッシュされます。この最適化により、ブラウザはこれらのリソースを再度要求することなくディスクから提供するため、その後のナビゲーションが高速化されます。もし攻撃者がどのリソースがキャッシュされているかを検知できれば、この情報を基に、ユーザが特定のページにアクセスしたことがあるかどうかをリークできます。

Cache Probingのバリエーションとして、[Error Events]({{< ref "../attacks/error-events.md" >}})を悪用することにより、より正確でインパクトのある攻撃を実行することができます。

## 攻撃の原理

攻撃者は、あるユーザーがあるSNSを訪問したかどうかを知りたいと考えています。

1. そのSNSにアクセスすると、いくつかのサブリソースがキャッシュされます。
2. ユーザが、SNSが通常取得するリソースを取得する攻撃者のページを訪問する。
3. [Network Timing XS-Leak]({{< ref "../attacks/timing-attacks/network-timing.md" >}})の手法を用いて、攻撃者のページはキャッシュからの応答（つまりステップ1が起こった）とネットワークからの応答（つまりステップ1が起こらなかった）の違いを検出することができます。（応答がキャッシュからの場合、遅延は顕著に短くなります。）

## Error EventsによるCache Probing

[Error Events]({{< ref "../attacks/error-events.md" >}})[^2]を利用したキャッシュプロービングは、より正確な攻撃を可能にします。このアプローチでは、時間の計測に頼らず、[Error Events]({{< ref "../attacks/error-events.md" >}})といくつかのサーバサイドの挙動を活用して、あるリソースがキャッシュされたかどうかを検出します。この攻撃は、以下のステップを必要とします。

1. [ブラウザキャッシュ]({{< ref "#invalidating-the-cache" >}})からのリソースを無効化する。このステップは、攻撃が別の訪問で以前にキャッシュされたリソースを考慮しないことを確認するために必要です。
2. ユーザーの状態によって異なる項目がキャッシュされるようなリクエストを実行する。例えば、ユーザーがログインしている場合にのみ、特定の画像を含むページをロードする。このリクエストは、`<link rel=prerender...`で対象のウェブサイトに移動したり、`iframe`でウェブサイトを埋め込んだり、`window.open`で新しいウィンドウを開くことで発生させることができる。
3. サーバが拒否するようなリクエストを引き起こす。例えば、[長大なRefererヘッダ](https://lists.archive.carbon60.com/apache/users/316239)を含んでいて、サーバーがリクエストを拒否するような場合です。もし、ステップ2でリソースがキャッシュされていれば、このリクエストはエラーイベントを発生させることなく、成功します。

### キャッシュの無効化
キャッシュからのリソースを無効にするには、攻撃者はそのサブリソースを取得する際にサーバーがエラーを返すようにする必要があります。これを実現するには、いくつかの方法があります。

- ブラウザによってリクエストが開始され、新しいコンテンツを受け取る前に`AbortController.abort()`で中止された `cache:'reload'`オプション付きのフェッチリクエスト
- [長大なRefererヘッダ](https://lists.archive.carbon60.com/apache/users/31623)と `'cache':'reload'`を持つリクエスト。ブラウザはこれを防ぐためにリファラの長さに[上限を設けている](https://github.com/whatwg/fetch/issues/903)ので、これはうまくいかないかもしれません。
- `POST`リクエストに `fetch` `no-cors` を指定した場合。エラーが返されない場合でも、ブラウザがキャッシュを無効化することがあります。
- Content-Type、Accept、Accept-Languageなど、サーバーを失敗させる可能性のあるリクエストヘッダ。（よりアプリケーションに依存します）
- その他のリクエストプロパティ

これらの方法のいくつかは、しばしばブラウザのバグとみなされていそうです。(例: [this bug](https://bugs.chromium.org/p/chromium/issues/detail?id=959789#c9)).

## Origin ReflectionによるCORS error

Origin Reflectionは、グローバルにアクセス可能なリソースに、リクエストを初期化したオリジンを反映した[Access-Control-Allow-Origin（ACAO）](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Access-Control-Allow-Origin)ヘッダを付与する挙動のことです。これは、CORSの設定ミス[^5]と考えることができ、ブラウザのキャッシュにリソースが存在するかどうかを検知するのに使用できます。

{{< hint info >}} 
例えば、Flask frameworkにおいて、origin refrectionは[デフォルトの動作](https://flask-cors.readthedocs.io/en/latest/api.htm)になっています。
{{< /hint >}}

もし `server.com` にホストされているリソースが `target.com` からリクエストされた場合、オリジンはレスポンスヘッダに次のように反映されるでしょう。`Access-Control-Allow-Origin: target.com` といった具合です。リソースがキャッシュされている場合、この情報はリソースと一緒にブラウザのキャッシュに保存されます。これにより、もし `attacker.com` が同じリソースを取得しようとした場合、2つの可能性があります。

- リソースがキャッシュにない場合：リソースは `Access-Control-Allow-Origin: attacker.com` ヘッダーとともにフェッチされ、保存される可能性があります。
- リソースがすでにキャッシュにある：フェッチ試行はキャッシュからリソースをフェッチしようとするが、要求しているオリジンと ACAO ヘッダーの値の不一致によりCORSエラーが起こる。（`target.com`が期待されているが、実際に提供されたのは`attacker.com` 。）以下に、この脆弱性を悪用して被害者のブラウザのキャッシュ状態を推測するコード例を示します。

```javascript
// この関数は、単にURLを受け取り、CORS モードでfetchします。
// fetchでエラーが発生した場合、attacker.comと犠牲者のIP間の
// オリジンの不一致によるCORSエラーになります。
function ifCached(url) {
    // fetchエラーの場合はtrueを、
	// 成功の場合はfalseに解決するプロミスを返します。
    return fetch(url, {
        mode: "cors"
    })
    .then(() => false)
    .catch(() => true);
}

// これは、server.comがorigin reflectionのCORS misconfigurationを
// 抱えていることを、攻撃者がすでに知っている場合にのみ有効。
var resource_url = "server.com/reflected_origin_resource.html"
var verdict = await ifCached(resource_url)
console.log("Resource was cached: " + verdict)
```

{{< hint tip >}}
これを軽減する最善の方法は、origin reflectionを排除して、 `Access-Control-Allow-Origin`ヘッダを使用することです。グローバルにアクセス可能で認証不要なリソースには `Access-Control-Allow-Origin: *` を使用します。
{{< /hint >}}

## Fetch with AbortController

以下のスニペットは、[AbortController](https://developer.mozilla.org/en-US/docs/Web/API/AbortController)インターフェースを*fetch*と*setTimeout*と組み合わせることで、リソースがキャッシュされているかどうかを検出し、ブラウザのキャッシュから特定のリソースを退避させる方法を示しています。このテクニックの良いところは、その過程で新しいコンテンツをキャッシュすることなく、プローブが行われることです。

```javascript
async function ifCached(url, purge = false) {
    var controller = new AbortController();
    var signal = controller.signal;
    // After 9ms, abort the request (before the request was finished).
    // The timeout might need to be adjusted for the attack to work properly.
    // Purging content seems to take slightly less time than probing
    var wait_time = (purge) ? 3 : 9;
    var timeout = await setTimeout(() => {
        controller.abort();
    }, wait_time);
    try {
        // credentials option is needed for Firefox
        let options = {
            mode: "no-cors",
            credentials: "include",
            signal: signal
        };
        // If the option "cache: reload" is set, the browser will purge
        // the resource from the browser cache
        if(purge) options.cache = "reload";

        await fetch(url, options);
    } catch (err) {
        // When controller.abort() is called, the fetch will throw an Exception
        if(purge) console.log("The resource was purged from the cache");
        else console.log("The resource is not cached");
        return false
    }
    // clearTimeout will only be called if this line was reached in less than
    // wait_time which means that the resource must have arrived from the cache
    clearTimeout(timeout);
    console.log("The resource is cached");

    return true;
}

// purge https://example.org from the cache
await ifCached('https://example.org', true);

// Put https://example.org into the cache
// Skip this step to simulate a case where example.org is not cached
open('https://example.org');

// wait 1 second (until example.org loads)
await new Promise(resolve => setTimeout(resolve, 1000));

// Check if https://example.org is in the cache
await ifCached('https://example.org');
```

## 対策

現在、Cache Probing攻撃からウェブサイトを完全に保護できるような優れた防御メカニズムはありません。それでも、以下のような[Cache Protections]({{< ref "cache-protections.md" >}})を導入することで、Webサイトのattack surfaceを縮小することは可能です。

- [Cache-controlヘッダ]({{< ref "cache-protections.md#cache-protection-via-cache-control-headers" >}})はリソースがキャッシュされるのを防ぐのに利用できます。
- [Random Tokens]({{< ref "cache-protections.md#cache-protection-via-random-tokens" >}})は攻撃者がURLを推測できないようにするために使用されます。
- [Vary: Sec-Fetch-Site]({{< ref "cache-protections.md#cache-protection-via-fetch-metadata" >}})はOriginのグループによってキャッシュを分離するために使用されます。
- ネットワーク要求が可能なユーザーコンテンツは、キャッシュを分割できるように、別ドメインまたは公開サフィックスリスト（適用可能な場合）を使用して、独自のeTLD+1上に配置すべきである。

キャッシュプロービング攻撃に対する有望な防御策は、要求元によってHTTPキャッシュを分割することです。このブラウザが提供する保護機能は、攻撃者のオリジンが他のオリジンのキャッシュされたリソースに干渉するのを防ぎます。

{{< hint info>}}
2021年9月現在、eTLD+1ごとにキャッシュを分割するPartitioned Cachesは、ほとんどのブラウザで利用できますが、アプリケーションはこれに依存できない状況です。
サブドメインからのリクエストや[window navigation]({{< ref "../attacks/navigations.md#partitioned-http-cache-bypass" >}})には保護が効きません。
{{< /hint >}}

## リアルワールドでの例

[Error Events Cache Probing]({{< ref "#cache-probing-with-error-events" >}})を利用した攻撃者は、ブラウザのキャッシュにビデオのサムネイルが残っているかどうかを確認することで、ユーザが特定のYouTubeビデオを視聴したかどうかを検出することができました[^3]。

## 参考文献

[^1]: Abusing HTTP Status Codes to Expose Private Information, [link](https://www.grepular.com/Abusing_HTTP_Status_Codes_to_Expose_Private_Information)
[^2]: HTTP Cache Cross-Site Leaks, [link](http://sirdarckcat.blogspot.com/2019/03/http-cache-cross-site-leaks.html)
[^3]: Mass XS-Search using Cache Attack, [link](https://terjanq.github.io/Bug-Bounty/Google/cache-attack-06jd2d2mz2r0/index.html#VIII-YouTube-watching-history)
[^4]: Timing Attacks on Web Privacy, [link](http://www.cs.jhu.edu/~fabian/courses/CS600.424/course_papers/webtiming.pdf)
[^5]: CORS misconfiguration, [link](https://web-in-security.blogspot.com/2017/07/cors-misconfigurations-on-large-scale.html)
