# CLAUDE.md

スマホカメラで照度を推定するハンディ照度計風 Web アプリ（PWA）。ビルド工程なしの静的サイトで、GitHub Pages（`main` 直下）にそのまま公開している。

- 公開URL: https://lux-meter.morilab-garage.com/
- リポジトリ: https://github.com/m-masaki72/web-lux-meter

## 構成

- `index.html` — HTML/CSS/JS をすべて1ファイルに収録。外部ライブラリ・外部リクエストなし。液晶風の等幅フォントは `fonts/` に置いた Share Tech Mono の ASCII サブセット（SIL OFL 1.1、CSS 上は別名 `LX Mono`）
  - `PLACES` / `SCENES`：場所の早見表と時間帯・用途の判定基準
  - 測定：`startSensor()`（AmbientLightSensor）→ 失敗したら `startCamera()`。露出を手動制御できれば `mode = "manual"`、できなければ `"auto"`（相対値、LCD に REL）
  - 表示：7セグは `buildSeg()` で SVG を生成し `drawText()` で点灯。バーグラフは `drawBar()`
  - 非同期の起動処理は `sess`（世代番号）で管理。電源OFF・カメラ切替で `sess++` し、await 後に `my !== sess` なら中断する
  - 画面が隠れたら `powerOff()`、戻ったら `powerOn(true)` で自動再開（復帰時は置き方の案内トーストを省く）
  - `mode === "auto"`（REL）では場所の目安・用途の判定・調光アドバイス・補正・共有文の場所名を出さない（自動露出で明るさが揃えられた値のため）
  - 色温度：ホワイトバランスを手動 6500K 付近に固定できた端末だけ `cctOf()`（McCamy 近似）で推定
  - 文言は `T.ja` / `T.en`。日本語の静的文言は HTML の `data-i18n` 要素から起動時に取り込むので、HTML 側を正とする
  - テーマは `data-theme`（未指定ならシステム設定）。色は CSS 変数で、ライト用は2か所（`[data-theme=light]` とメディアクエリ）を揃えて直す
  - 状態の保存は localStorage（補正係数 `luxCal:<機種>:<user|environment|als>`、単位 `luxUnit`、言語 `luxLang`、テーマ `luxTheme`）
  - ショートカット：`?start=1` で自動測定開始
- `sw.js` — Service Worker。**アセットや index.html を変えたら `CACHE` のバージョンを上げる**
- `manifest.webmanifest` / `icons/` / `og.png` — PWA・SNS 用。アイコン PNG と og.png は `icons/icon.svg` などから puppeteer で書き出したもの

## ルール

- 照度は「あくまで参考値」である旨の注意書きを画面上（本体直下と末尾の注記）に必ず残す
- JSON-LD の FAQPage の内容は、ページ内「よくある質問」の文言と一致させる（片方だけ変えない）。英語版の FAQ は `T.en.faqBody`
- 本体（受光部〜POWER）は縦持ちスマホの1画面目に収める。受光部の大きさ `--ring` を画面の高さから決めているので、本体の高さを変えたら 360×640 と 390×844 で確認する
- URL を変えるときは canonical・hreflang・OGP・JSON-LD・共有テキスト・`sitemap.xml` の URL もまとめて更新する。大きく内容を変えたら `sitemap.xml` の `lastmod` も更新
- 主な対象はスマホ縦画面（幅 360〜430px）。変更後はスマホ幅で横スクロールが出ないことを確認する

## 動作確認

カメラは HTTPS か localhost でのみ動く。ローカル確認は `npx serve .` などで配信して開く。
ヘッドレス Chrome で確認するときは `--use-fake-device-for-media-stream --use-fake-ui-for-media-stream` で疑似カメラを使える（疑似カメラは露出を変えても映像が変わらないため、数値の正しさは実機でしか確認できない）。
