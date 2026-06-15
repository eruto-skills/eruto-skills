# AIスロップ アンチパターンカタログ

AIコーディングツールが生成するUIコードに共通して現れる9カテゴリのアンチパターン一覧。
各カテゴリにはGrepパターン・根本原因・改善策を記載している。

## カテゴリ1: カラー・グラジエント（重症度: 高）

### パープルグラジエント症候群

**Grep パターン:**
```
indigo-[0-9]|purple-[0-9]|violet-[0-9]|from-indigo|to-purple|to-violet|from-purple
bg-gradient-to|linear-gradient.*purple|linear-gradient.*indigo
```

**根本原因:** Tailwind CSS の `bg-indigo-500` が数年間デフォルトボタン色として膨大なチュートリアル・OSSに使われ続けた。LLMはそのコードを学習し「モダンUI = インディゴ」と統計的に収束する。Tailwind作者Adam Wathanが2025年8月に謝罪。

**改善策:** OKLCHでブランド固有のカラーランプを定義し、Tailwindのデフォルト色を`:root`で上書きする。
```css
:root {
  --brand-primary: oklch(55% 0.18 142);
}
```

### グラジエントオーブ / ブロブ背景

**Grep パターン:**
```
backdrop-blur|backdrop-filter|bg-white\/1[0-9]|bg-white\/[1-9]\b|border-white\/2[0-9]
```

**改善策:** 背景装飾は1ページあたり1箇所に限定し、SVGグレインテクスチャ等の代替を検討する。

### ピュアブラック背景

**Grep パターン:**
```
bg-black\b|background.*#000000|background.*rgb\(0,\s*0,\s*0\)
```

**改善策:** `#000000` は避け、`#121212`〜`#1E1E1E` 程度を使う。

---

## カテゴリ2: タイポグラフィ（重症度: 中）

### Inter / Roboto デフォルト固定

**Grep パターン:**
```
'Inter'|"Inter"|font-family.*Inter|font-inter|@import.*Inter|@import.*Roboto
Roboto|Open.Sans|Noto.Sans|font-sans\b
```

**根本原因:** Figma→Tailwind→Next.js エコシステムがInterをデフォルト化した。「InterはAIのComic Sans」と揶揄されるほど、選択に意図がないことを露呈する。

**改善策:** ブランドキャラクターに合わせた書体を選定する。
- テック/モダン: Geist（Vercel）、Satoshi
- フレンドリー: Plus Jakarta Sans、Figtree
- B2B/スタートアップ: Plus Jakarta Sans

### 同一ウェイト問題

**Grep パターン:**
```
font-weight.*600|font-semibold.*font-semibold|font-medium.*font-medium
```

**改善策:** 見出し（700〜800）・本文（400〜500）・ラベル（500〜600）でウェイトを使い分ける。

---

## カテゴリ3: 角丸過剰（重症度: 低〜中）

### rounded-2xl 全面適用

**Grep パターン:**
```
rounded-2xl|rounded-3xl|rounded-full|border-radius:\s*2[0-9]px
```

**根本原因:** Bootstrap/Tailwindの`rounded-lg`が「カードのデフォルト」として学習データに定着した。

**改善策:**
- 内側のradius = 外側のradius - padding（ネストの黄金律）
- 要素サイズに応じたスケールを設定する（sm: 4px, md: 8px, lg: 12px）

---

## カテゴリ4: シャドウ・グロー（重症度: 中）

### shadow-lg / shadow-xl 多用

**Grep パターン:**
```
shadow-lg|shadow-xl|shadow-2xl|box-shadow.*rgba.*0\.[3-9]|glow|drop-shadow-lg
```

**根本原因:** AIは「浮いている感」をシャドウで表現しようとし、強いシャドウを乱用する。

**改善策:** 多層・低透明度のシャドウが自然に見える。
```css
box-shadow: 0 1px 3px rgba(0,0,0,0.08), 0 4px 12px rgba(0,0,0,0.04);
```

---

## カテゴリ5: 状態設計欠如（重症度: 高）

### Empty / Error State なし

**存在確認 Grep:**
```
EmptyState|ErrorState|empty.state|error.state|NoData|エラー画面|データがありません
```

**根本原因:** NN/Gの調査（2025年）では50のAI生成ダッシュボードの92%がEmpty Stateを持たず、78%がError Stateを持たなかった。AIはハッピーパスのみを学習データから再現する。

**改善策:** 以下3状態を必ず設計する。
- **Empty State**: データがない場合の表示とアクション誘導
- **Error State**: エラー時のメッセージと回復アクション
- **Loading State**: スケルトンローダーまたはタイピングインジケーター

---

## カテゴリ6: ローディングパターン（重症度: 中）

### スピナー乱立

**Grep パターン:**
```
Spinner|spinner|<CircularProgress|<LoadingIcon|loading.*spinner
```

**スケルトン確認:**
```
Skeleton|skeleton|pulse\b|animate-pulse
```

**改善策:**
- LLM推論のような長時間処理: ストリーミング出力
- リスト読み込み: スケルトンローダー（出力形状に合わせる）
- 会話AI: `...` タイピングインジケーター
- 汎用スピナーは1秒未満の短時間のみ使う

---

## カテゴリ7: デザイントークン不在（重症度: 高）

### インライン値乱立

**存在確認 Grep:**
```
var(--|:root\s*\{
```

**Tailwind カスタムテーマ確認:**
```
theme.*extend|colors:.*\{
```

**根本原因:** AIは毎セッションでデザイントークンを参照せず、コンテキストドリフトで値が毎回変わる。

**改善策:** DESIGN.md + CSS カスタムプロパティでトークンを定義し、AIが毎回それを参照する体制を作る。
```css
:root {
  --color-brand-500: oklch(55% 0.18 142);
  --space-4: 1rem;
  --radius-md: 8px;
}
```

---

## カテゴリ8: 汎用コピー（重症度: 低）

**Grep パターン:**
```
Get Started|Streamline|Supercharge|Enterprise.grade|World.class|lorem ipsum|placeholder text
Discover|Transform|Revolutionize|Unleash|Empower.*team
```

**根本原因:** 統計的平均のマーケティングコピーが学習データに多い。

**改善策:** 問題ベースのコピーと具体的な数値を使う。
- NG: "Streamline your workflow"
- OK: "応答時間を72時間から15分に短縮する"

---

## カテゴリ9: アイコン汎用化（重症度: 低）

**Grep パターン:**
```
lucide-react|@heroicons|spark.*icon|SparklesIcon|MagicIcon
```

**根本原因:** v0・LovableのデフォルトアイコンライブラリがLucideであり、学習データにも多い。「Inter + Lucide の組み合わせ」が揃った時点でデザイナーにはAI生成と即座に見抜かれる。

**改善策:** Phosphor Icons・Radix Icons・独自SVGへの切り替えを検討する。
アイコン単体ではなく「アイコン + 文脈」でセマンティクスを成立させる。
