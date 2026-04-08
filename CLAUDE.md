# CLAUDE.md — FPの6つの係数プロジェクト

## プロジェクト概要

FP（ファイナンシャルプランニング）で使われる6つの係数を、グラフと表でインタラクティブに学べる単一ファイルWebアプリ。

- **起源**: 既存の「複利計算機」（index.html）を全面改造
- **構成**: HTML/CSS/JS 単一ファイル + Chart.js（CDN）
- **PWA対応**: manifest.json + sw.js（Service Worker）

---

## ファイル構成

```
index.html       # メインページ（全コード）
manifest.json    # PWA設定
sw.js            # Service Worker（HTML=ネットワーク優先）
icon.svg         # アプリアイコン（青背景 + 白FP + ゴールド6）
README.md        # 日本語ドキュメント
session-log.md   # 開発セッションログ
CLAUDE.md        # このファイル
.claude/
  launch.json    # 開発サーバー設定
```

## 開発サーバー

```bash
python3 -m http.server 3000
# → http://localhost:3000
```

`.claude/launch.json` に設定済み。`preview_start("Static Dev Server")` で起動可能。

---

## アーキテクチャ

### データ構造

`COEFS` 配列（6要素）が全係数を定義。各要素のフィールド：

```js
{
  name: '終価係数',
  formula: '(1+r)ⁿ',
  desc: '現在の1円が将来いくらになるか',
  desc2: '（ローン用途の説明）',   // 資本回収係数・年金現価係数のみ
  tags: ['一括', '将来'],          // [お金の動き, 時間方向]
  color: [66, 153, 225],           // RGB
  chartType: 'stacked_fv',
  calc: (r, n) => ...,
  headers: ['年', '終価係数'],
  rowFn: (r, n) => [...],
  meaningFn: (r, n) => '...',
}
```

### タブ順序（左から）

1. 終価係数 — `stacked_fv`
2. 現価係数 — `stacked_pv`
3. 年金終価係数 — `stacked_annuity`
4. 減債基金係数 — `stacked_sf`
5. 資本回収係数 — `stacked_cr`
6. 年金現価係数 — `stacked_pva`

### タグ体系

| | [将来] | [現在] |
|---|---|---|
| [一括] | 終価係数 | 現価係数 |
| [積立] | 年金終価係数 | 減債基金係数 |
| [取崩] | 資本回収係数 | 年金現価係数 |

### chartType 別グラフ仕様

| chartType | 青（下） | 緑（上） | 備考 |
|---|---|---|---|
| `stacked_fv` | 元本=1（固定） | (1+r)^t - 1 | 終価係数 |
| `stacked_pv` | 現価係数（固定） | pv×(1+r)^t - pv | N年目合計=1 |
| `stacked_annuity` | 積立累計=t | 年金終価 - t | 年金終価係数 |
| `stacked_sf` | 積立累計=pmt×t | 総額 - 積立累計 | N年目合計=1 |
| `stacked_cr` | 元金残=max(0, 1-pmt×(t-1)) | 残高 - 元金残 | 1年目=1から減少 |
| `stacked_pva` | 元金残=max(0, pv-(t-1)) | 残高 - 元金残 | 1年目=pvから減少 |

### 色ルール

- **青** `rgba(66,153,225,0.75)` — 元本・積立累計・元金残（全係数共通）
- **緑** `rgba(72,187,120,0.85)` — 運用益（全係数共通）
- 各タブのキーカラーは `c.color` で個別定義

### ツールチップ

```js
tooltip: {
  mode: 'index',           // バーのどの部分でも全データ表示
  itemSort: (a, b) => b.datasetIndex - a.datasetIndex,  // 運用益を上に
  callbacks: {
    label: ...,
    footer: items => `合計: ${合計値}`,
  }
}
```

### テーブル

- 7列（年 + 6係数）を1テーブルに集約
- `activeTab` の列を `active-col` クラスでハイライト
- 長い係数名は `breakAt` オブジェクトで `<br>` 挿入

### 注意書き（`stacked_cr` / `stacked_pva` のみ表示）

> ※ このグラフは概念理解のため、先に元金を減らす方式で表示しています。実務（元利均等返済など）では利息が先に支払われるため、元金の減り方は異なります。

`style.display` で直接制御（`classList` ではなく）。

---

## 各係数キーカラー

| 係数 | RGB | 印象 |
|---|---|---|
| 終価係数 | `[66, 153, 225]` | 青 |
| 現価係数 | `[185, 60, 130]` | クリムゾン |
| 年金終価係数 | `[72, 187, 120]` | 緑 |
| 減債基金係数 | `[237, 137, 54]` | オレンジ |
| 資本回収係数 | `[56, 178, 172]` | ティール |
| 年金現価係数 | `[229, 62, 62]` | 赤 |

## タグ色

| タグ | 背景 | 文字 |
|---|---|---|
| 一括 | `#bee3f8` | `#2c5282` |
| 積立 | `#c6f6d5` | `#276749` |
| 取崩 | `#fed7d7` | `#9b2c2c` |
| 将来 | `#e9d8fd` | `#553c9a` |
| 現在 | `#b2f5ea` | `#234e52` |

---

## GitHub

リポジトリ: `git@github.com:yto/fp-six-coefficients.git`
公開URL: `https://yto.github.io/fp-six-coefficients/`
ブランチ: `main`
