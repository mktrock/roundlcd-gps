# gps-pwa — スマホのGPS位置を RoundLcd へ送る Web アプリ

Android の Chrome で開く単体ページ（`index.html`）。`navigator.geolocation` で取った位置を、Web Bluetooth で RoundLcd の
GATT 特性（`GpsFix` 16バイト）へ約1Hzで書き込みます。RoundLcd の MAP 画面がそれを地図に重ねて表示します。
値とバイト配置は `shared/include/gpsProtocol.h` と手動で同期します（`index.html` の先頭付近の定数と `encodeFix()`）。

## 使い方

1. RoundLcd の MAP 画面で **DOWN を長押し**して BLE を開始する（`WAITING` と機器名 `RoundLcd` が出る）。
2. スマホの Chrome でこのページを **https で** 開く（Web Bluetooth と位置情報は https か localhost でしか使えない）。
3. 「RoundLcd に接続して送信を開始」→ 機器 `RoundLcd` を選ぶ → 位置情報を許可する。
4. RoundLcd の画面が `NO FIX`（接続済み・測位待ち）から地図に変わる。走行中はこのページを前面に開いたままにする（画面は消灯しない）。

## 目的地
ページの「目的地を設定する」を開くと、RoundLcd に目的地を1か所だけ設定できます（電源を切っても RoundLcd の NVS に残ります）。

- **地図をタップ**して選ぶ（国土地理院の地図。読み込みにインターネットが必要）、または **座標（`35.6812,139.7671`）／Google マップの長い URL を貼って**「入力から場所を選ぶ」。
  Google マップの短縮リンク（`maps.app.goo.gl`）は読めません（リンクを開いて出る長い URL か、場所を長押しして出る座標を貼ってください）。
- 「この場所を目的地にする」で送信。「現在地を目的地にする」「目的地を解除する」もあります。RoundLcd に接続していない間に選んだ場所は、接続したときに送ります。
- RoundLcd の MAP 画面に、目的地の印（地図の外なら縁に矢印）と直線距離（`DEST 3.2km`、50m以内は `ARRIVED`）が出ます。案内（道順）はしません。
  操作モード（DOWN長押し）の4つ目 `DEST` で、UP短押し＝目的地が中心に来るまで地図を動かす、DOWN短押し＝目的地を解除。

切れたときは3秒おきに自動で再接続します。iPhone の Safari は Web Bluetooth に対応していないため使えません。

## 配信

`index.html` / `manifest.json` / `icon-*.png` を https で配信できる場所へ置きます（GitHub Pages など。ホーム画面に追加するとアプリのように開ける）。

開発中は PC で `python -m http.server 8000`（このフォルダで）→ USB 接続した Android で `adb reverse tcp:8000 tcp:8000` →
スマホの Chrome で `http://localhost:8000/` を開けば、localhost は安全なコンテキストとして扱われるため試せます。
