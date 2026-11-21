# WithGaraku — GLSL Effect Exporter

イラストの演出に使うエフェクトを GLSL で書いて、**背景透過の PNG（単体・連番）**で書き出すためのブラウザツールです。

公開ページ: https://yamasaki4c6.github.io/WithGaraku.io/

## 使い方

1. 左のエディタに Shadertoy と同じ `mainImage(out vec4 fragColor, in vec2 fragCoord)` 形式でシェーダーを書きます（Ctrl/⌘+Enter でコンパイル。自動コンパイルもあります）。
2. `fragColor` のアルファがそのまま PNG の透過になります。
3. 解像度・fps・長さを決めて「連番をZIPで書き出し」を押します。Chrome / Edge なら「フォルダへ書き出し」で、指定したフォルダに直接保存することもできます。

### レイヤー

右側のレイヤーパネルで、**画像レイヤー**と **GLSL レイヤー**を重ねられます（上にあるものが手前）。

- 追加：「＋GLSL」「＋画像」。画像はプレビューへのドラッグ＆ドロップや Ctrl/⌘+V の貼り付けでも追加できます
- 並び替え：レイヤーをドラッグ。目のアイコンで表示／非表示、ダブルクリックで名前の変更
- 不透明度と合成モード（通常・加算（発光）・スクリーン・乗算・置き換え）をレイヤーごとに設定できます
- 画像レイヤーは、プレビュー上でドラッグすると移動、ホイールで拡大縮小できます（100% = キャンバスに収まる大きさ）
- GLSL レイヤーの `iChannel0` は「そのレイヤーより下の合成結果」です。合成を「置き換え」にすると、下の絵を歪ませるようなエフェクトが作れます（サンプル「陽炎」）
- 書き出されるのは**表示中のレイヤーだけ**です。エフェクトだけを透過で書き出したいときは、イラストのレイヤーを非表示にしてください
- レイヤー構成と画像はブラウザ内に自動保存されます

### アルファの扱い

| モード | 書き方 | 向いているもの |
|---|---|---|
| straight | `vec4(col, a)` | 煙・墨・影など、不透明感のあるもの |
| premultiplied | `vec4(col * a, a)` | 光を足し算で重ねていくシェーダー |
| 黒背景（出力設定） | — | 発光・火花・ブルーム。クリスタで「加算（発光）」「スクリーン」にして乗せる |

### 使える uniform

`iResolution` `iTime` `iTimeDelta` `iFrame` `iFrameRate` `iMouse` `iDate` `iChannel0`（下のレイヤーの合成結果） `iChannelResolution[0]`、独自に `iDuration`（書き出す長さの秒数）があります。`fract(iTime / iDuration)` を使うと、ループがきれいに繋がります。

### プレビュー

プレビューの背景（市松模様・黒・白・単色）は確認用で、書き出しには含まれません。

## 連番を動画にする（ffmpeg）

```bash
# ProRes 4444（透過あり）
ffmpeg -framerate 30 -i frame_%04d.png -c:v prores_ks -profile:v 4444 -pix_fmt yuva444p10le out.mov
# WebM VP9（透過あり）
ffmpeg -framerate 30 -i frame_%04d.png -c:v libvpx-vp9 -pix_fmt yuva420p -b:v 0 -crf 20 out.webm
```

## ローカルで動かす

`index.html` 1枚だけで動きます。ダブルクリックで開いても動きますが、画像の読み込みで問題が出る場合は `npx serve` などのローカルサーバー経由で開いてください。
