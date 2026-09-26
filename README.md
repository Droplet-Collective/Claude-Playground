# Claude-Playground

Claude と一緒に作ったものを置いておく遊び場（プレイグラウンド）です。

## Projects

- [GPU流体シミュレーション (WebGL2)](fluid/) — `fluid/index.html`
- [粘菌 (Physarum) シミュレーション (WebGL2)](physarum/) — `physarum/index.html` — <https://droplet-collective.github.io/Claude-Playground/physarum/>

---

## GPU流体シミュレーション (WebGL2)

ブラウザ上で動くリアルタイムの Navier–Stokes 流体シミュレーションです。
いわゆる「Stable Fluids」法を WebGL2 のフラグメントシェーダーで実装しており、外部ライブラリや CDN には一切依存しない単一の HTML ファイルです。

- ドラッグ（マウス / マルチタッチ）で速度と染料を注入
- クリック（ドラッグなし）で放射状のバースト
- 約 3 秒操作がないと、ゆるやかなランダム噴射が自動で入り、画面が止まって見えることがありません
- 起動時には大量のスプラットで派手に始まります
- 4 種類のカラーパレット（Neon / Sakura / Ocean / Mono）と、スプラットごとに巡る HSV の色相
- ブルーム（グロー）とビネットのポストプロセス、FPS 表示

### 動かし方

1. ローカル: `fluid/index.html` をブラウザで開くだけです（ビルド不要）。
2. GitHub Pages: `main` に push すると `.github/workflows/pages.yml` が自動でデプロイします。
   公開 URL: <https://droplet-collective.github.io/Claude-Playground/fluid/>

> **初回のみ設定が必要です。** リポジトリの **Settings → Pages → Build and deployment → Source** を **GitHub Actions** にしてください。
> これをしないとワークフローが Pages 環境にデプロイできません。

対応ブラウザ: WebGL2 と `EXT_color_buffer_float`（または `EXT_color_buffer_half_float`）が使える最近の Chrome / Edge / Firefox / Safari。
非対応の場合は真っ黒な画面ではなく、その旨のメッセージを表示します。

### 操作

| 操作 | 内容 |
| --- | --- |
| ドラッグ | 速度と染料を注入（マルチタッチ対応） |
| クリック / タップ | その場でバースト |
| `Space` | ランダムバースト |
| `C` | 画面をクリア |
| `S` | スクリーンショットを PNG でダウンロード |
| `P` | パレットを切り替え（Neon → Sakura → Ocean → Mono） |
| `B` | ブルームの ON / OFF |
| `Esc` | 一時停止 / 再開 |
| `H` | UI の表示 / 非表示 |

左上のガラス風パネルでは、染料の減衰・速度の減衰・圧力の反復回数・渦度（Curl）の強さ・スプラット半径をスライダーで調整でき、
ブルーム / 一時停止 / 自動スプラットの ON / OFF も切り替えられます。キーボードだけでも操作できます。

### 技術メモ

各フレームで以下のパスをフラグメントシェーダーとして順に実行します（シミュレーション格子は 256、染料テクスチャは 1024、いずれも半精度浮動小数点テクスチャ）。

1. **Curl** — 速度場の回転（渦度）を計算
2. **Vorticity confinement** — 渦度を強調する力を速度に加え、小さな渦が消えにくくする
3. **Divergence** — 速度場の発散を計算
4. **Pressure (Jacobi)** — ポアソン方程式を Jacobi 法で約 20 回反復して圧力を求める
5. **Gradient subtraction** — 圧力勾配を引いて速度場を非圧縮にする
6. **Advection** — 速度で速度自身と染料を移流（半ラグランジュ法、減衰付き）
7. **Bloom** — しきい値抽出 → ダウンサンプルぼかし → 加算アップサンプル
8. **Display** — 染料 + ブルーム + 簡易シェーディング + ビネットを画面に合成

canvas のサイズは `devicePixelRatio`（最大 2）とウィンドウサイズに追従し、リサイズ時はテクスチャを内容ごと作り直します。
スクリーンショットは `preserveDrawingBuffer: true` で `canvas.toBlob()` から取得しています。

---

## 粘菌 (Physarum) シミュレーション (WebGL2)

100 万体のエージェントが GPU 上で粘菌（*Physarum polycephalum*）のように振る舞い、脈打つ血管状のネットワークを自己組織化していくシミュレーションです。
Jeff Jones の Physarum モデルを WebGL2 だけで実装した、外部依存のない単一の HTML ファイルです。

- 各エージェントは前方・左右 3 点で「フェロモン（トレイル）」の濃さを感じ、濃い方へ向きを変えて進み、自分もトレイルを残します
- トレイルは毎フレームぼかされて薄れていくため、よく通る道だけが太い脈として残ります
- 5 種類のプリセット（Nervous system / Coral / Galaxy / Cells / Web）で、神経網・珊瑚・渦・細胞・蜘蛛の巣のような模様に切り替わります
- 5 種類のカラーパレット（Bioluminescent / Ember / Ice / Neon / Mono）とグロー、FPS 表示
- マウス / タッチで押している間、その位置が「餌」になってエージェントが集まってきます
- エージェント数は 250k / 500k / 1M から選択（URL に `?agents=250k` を付けても指定できます）

公開 URL: <https://droplet-collective.github.io/Claude-Playground/physarum/>

対応ブラウザは流体シミュレーションと同じく、WebGL2 と `EXT_color_buffer_float`（または `EXT_color_buffer_half_float`）が使える最近のブラウザです。
非対応の場合は日本語のメッセージを表示します。

### 操作

| 操作 | 内容 |
| --- | --- |
| 押しながら動かす | その位置に餌を置く（エージェントが集まる） |
| `Space` | Scatter — エージェントを再配置（Random → Ring → Center の順に巡る） |
| `C` | トレイルをクリア |
| `S` | スクリーンショットを PNG でダウンロード |
| `P` | パレットを切り替え |
| `N` | プリセットを切り替え |
| `Esc` | 一時停止 / 再開 |
| `H` | UI の表示 / 非表示 |

左上のガラス風パネルでは、センサー角度・センサー距離・回転角・移動量・堆積量・減衰・拡散の強さ・明るさをスライダーで調整でき、
プリセット / パレット / エージェント数をメニューから選べます。キーボードだけでも操作できます。

### 技術メモ

- エージェント（x, y, 向き, 速度のばらつき）は `RGBA32F` テクスチャ（1M 体で 1024×977）に格納し、フラグメントシェーダーでピンポン更新します（32F が描き込めない環境では半精度にフォールバック）。
- 更新パスでは各エージェントがトレイルテクスチャを前方と ±センサー角の 3 点でサンプリングし、最も濃い方へ回転角だけ向きを変えてから移動量ぶん進みます（端はトーラス状にラップ）。
- 堆積パスでは `gl_VertexID` からエージェントテクスチャを `texelFetch` し、`GL_POINTS` を加算ブレンドで `R16F` のトレイルテクスチャに描き込みます（半精度なら `EXT_float_blend` なしでブレンドできます）。
- トレイルは毎フレーム 3×3 ぼかし（拡散）と乗算による減衰をピンポンで適用します。
- 表示パスは濃度を `1 - exp(-d * exposure)` でトーンマップし、5 色のカラーランプに通します。1/4 解像度のぼかしテクスチャを足してグローとビネットを加えています。

### ファイル構成

```
index.html                 # プロジェクト一覧のランディングページ
fluid/index.html           # 流体シミュレーション本体（単一ファイル）
physarum/index.html        # 粘菌シミュレーション本体（単一ファイル）
.github/workflows/pages.yml# GitHub Pages デプロイ
```
