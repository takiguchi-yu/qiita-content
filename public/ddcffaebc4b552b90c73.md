---
title: WebSocket でランダム指数バックオフ（エクスポネンシャルバックオフ）の実装
tags:
  - websocket
  - リトライ
  - 再接続
private: false
updated_at: '2023-09-24T12:18:08+09:00'
id: ddcffaebc4b552b90c73
organization_url_name: null
slide: false
ignorePublish: false
---
### Exponential Backoff（指数バックオフ）とは

Exponential Backoff とはリトライの仕組みの一種で、リトライ間隔を徐々に広げていくアルゴリズムになります。1秒→3秒→9秒→27秒... のように指数関数的にリトライ間隔を広げることでアプリケーションの信頼性を向上させて、開発者の運用コストを削除します。

### アルゴリズム

### WebSocket接続処理のソースコード全体（実装例）

```js
const endpoint = 'ws://localhost:8080'

// 指数バックオフ設定値
let initialReconnectDelay = 1000;
let currentReconnectDelay = initialReconnectDelay;
let maxReconnectDelay = 16000;

function wsConnection(endpoint) {
  ws = new WebSocket(endpoint);
  let s = (l) => console.log(l);

  // コネクション確立
  ws.onopen = m => {
    s(" CONNECTED")
    currentReconnectDelay = initialReconnectDelay;
  }

  // メッセージ受信
  ws.onmessage = m => s(" RECEIVED: " + JSON.stringify(m.data, null, 3))

  // コネクションエラー
  ws.onerror = e => s(" ERROR")

  // コネクションクローズ (クローズしたら再接続)
  ws.onclose = e => {
    s(" CONNECTION CLOSED");

    setTimeout((function() {
      reconnectToWebsocket()
    }).bind(this), currentReconnectDelay + Math.floor(Math.random() * 3000))  // ランダム指数バックオフ
  }

  reconnectToWebsocket = () => {
    if(currentReconnectDelay < maxReconnectDelay) {
      currentReconnectDelay*=2;
      s(" RECONNECTION...");
      wsConnection(endpoint);
    }
  }  
}

wsConnection(endpoint);
```

#### バックオフをランダムにする
バックオフをランダムにすることのメリットは 10,000 のクライアントが接続されているときにサーバーがダウンし、すぐに復旧した場合はどうなるでしょうか? そうです、これらのクライアントはすべて、まったく同じ秒で再接続を試みます。これにより、サーバーでスパイクが発生し、最悪の場合はサーバーがダウンする可能性があります。

このスパイクを防ぐために、バックオフ アルゴリズムをランダムにします。これにより、すべてのクライアントが同時に再接続するわけではなくなり、サーバーを急上昇させることなく、すべてのクライアントが正常に再接続できるようになります。
（これをジッターというらしい）

### 参考サイト

https://dev.to/jeroendk/how-to-implement-a-random-exponential-backoff-algorithm-in-javascript-18n6
