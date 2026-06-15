# DESIGN.md テンプレート

AIコーディングツールに参照させる DESIGN.md の生成テンプレート。
検出されたアンチパターンに基づいて各セクションを埋める。

---

## 生成ルール

- `<!-- TODO: ... -->` は検出できなかった項目・ユーザーが後から埋める箇所
- スロップスコアが低い（0〜30）場合は「禁止事項」セクションを簡素化してよい
- 既存コードからブランドカラーが検出された場合は `YOUR_COLOR_HERE` を実際の値に置換する
- OKLCH で色を定義することを強く推奨する（HSLより知覚的均一性が高い）

---

## テンプレート本文

````markdown
# DESIGN.md — [プロジェクト名]

> このファイルはAIコーディングツールへの設計指示書です。
> UIを生成・修正する前に必ずこのファイルを読んでください。

## What NOT to Use（禁止事項）

以下はAI生成UIの「スロップ」パターンです。これらを使用しないでください。

### 色・グラジエント
- `bg-indigo-500` / `bg-purple-*` / `bg-violet-*` — ブランド指定の色のみ使用すること
- `from-indigo-* to-purple-*` などの紫グラデーション — 禁止
- `backdrop-blur-xl` を複数要素に同時適用 — 1ページあたり最大1要素

### フォント
- `font-family: 'Inter', sans-serif` の単独使用 — 下記のフォント定義を使うこと
- Google Fonts からの動的ロード — パフォーマンス問題。self-hosted または variable font を使うこと

### レイアウト・スタイル
- `rounded-2xl` / `rounded-3xl` を全要素に一律適用 — 下記のradius スケールを使うこと
- `shadow-xl` / `shadow-2xl` を全カードに適用 — 最大 `shadow-md` を基準にする
- グラスモーフィズム（`backdrop-blur` + 半透明）の全面適用 — 強調用途のみ

### コピー・アイコン
- "Get Started", "Streamline your workflow", "Enterprise-grade" などの汎用フレーズ — 禁止
- Lucide + Inter の組み合わせ — 即座にAI生成と識別される。下記のアイコン設定を使うこと

---

## Color System（カラーシステム）

```css
:root {
  /* Brand Colors */
  --brand-primary:    <!-- TODO: oklch(XX% X.XX XXX) --> ;
  --brand-secondary:  <!-- TODO: oklch(XX% X.XX XXX) --> ;
  --brand-accent:     <!-- TODO: oklch(XX% X.XX XXX) --> ;

  /* Semantic Colors */
  --color-success:    oklch(60% 0.20 145);   /* green */
  --color-warning:    oklch(75% 0.18 70);    /* amber */
  --color-error:      oklch(55% 0.22 25);    /* red */
  --color-info:       oklch(60% 0.18 250);   /* blue */

  /* Surface */
  --surface-base:     <!-- TODO: oklch(97% 0.005 XX) --> ;  /* ページ背景 */
  --surface-raised:   <!-- TODO: oklch(99% 0.003 XX) --> ;  /* カード背景 */
  --surface-overlay:  <!-- TODO: oklch(95% 0.008 XX) --> ;  /* ホバー・選択背景 */

  /* Text */
  --text-primary:     oklch(20% 0.005 260);  /* メインテキスト */
  --text-secondary:   oklch(45% 0.005 260);  /* サブテキスト */
  --text-placeholder: oklch(65% 0.005 260);  /* プレースホルダー */
}
```

---

## Typography（タイポグラフィ）

```css
:root {
  /* Font Families */
  --font-heading: <!-- TODO: 'Satoshi', 'Geist', 'Plus Jakarta Sans' のどれか --> , sans-serif;
  --font-body:    <!-- TODO: フォント名 --> , sans-serif;
  --font-mono:    'Geist Mono', 'JetBrains Mono', monospace;

  /* Type Scale（流体タイポグラフィ） */
  --text-xs:   clamp(0.75rem,  0.7rem  + 0.25vw, 0.875rem);
  --text-sm:   clamp(0.875rem, 0.8rem  + 0.375vw, 1rem);
  --text-base: clamp(1rem,     0.9rem  + 0.5vw,   1.125rem);
  --text-lg:   clamp(1.125rem, 1rem    + 0.625vw, 1.25rem);
  --text-xl:   clamp(1.25rem,  1.1rem  + 0.75vw,  1.5rem);
  --text-2xl:  clamp(1.5rem,   1.2rem  + 1.5vw,   2.25rem);
  --text-3xl:  clamp(1.875rem, 1.5rem  + 1.875vw, 3rem);
  --text-4xl:  clamp(2.25rem,  1.5rem  + 3.75vw,  4.5rem);
}

h1, h2, h3 { font-family: var(--font-heading); }
body { font-family: var(--font-body); }

/* Weight Scale */
/* body: 400, ui-label: 500, subheading: 600, heading: 700-800 */
```

---

## Spacing & Radius（スペーシング・角丸）

```css
:root {
  /* Spacing（8px ベースグリッド） */
  --space-1:  0.25rem;   /* 4px */
  --space-2:  0.5rem;    /* 8px */
  --space-3:  0.75rem;   /* 12px */
  --space-4:  1rem;      /* 16px */
  --space-6:  1.5rem;    /* 24px */
  --space-8:  2rem;      /* 32px */
  --space-12: 3rem;      /* 48px */
  --space-16: 4rem;      /* 64px */

  /* Border Radius Scale */
  --radius-sm:   4px;     /* インプット、タグ、バッジ */
  --radius-md:   8px;     /* ボタン、小カード */
  --radius-lg:   12px;    /* メインカード、パネル */
  --radius-xl:   16px;    /* モーダル、ダイアログ */
  --radius-pill: 9999px;  /* ピル型バッジのみ */
  /* rounded-2xl (16px以上) を全要素に適用しないこと */
}
```

---

## Component Rules（コンポーネントルール）

### 必須: 3状態の実装

**すべての非同期データ取得を伴うコンポーネントに以下3状態を実装すること:**

```tsx
// Empty State — データがない場合
<EmptyState
  icon={<IconName />}
  title="[具体的な説明]"
  action={<Button>アクション</Button>}
/>

// Error State — エラー時
<ErrorState
  message="[ユーザーフレンドリーなエラーメッセージ]"
  retry={handleRetry}
/>

// Loading State — スケルトンローダーを使うこと（スピナー禁止）
<Skeleton className="h-[N]px w-full" />
```

### shadcn/ui の上書きルール

shadcn/ui を使用する場合、以下のデフォルト値を必ず上書きすること:
- `Button` の `variant="default"` 色: `bg-primary` を `var(--brand-primary)` にマッピング
- カード系コンポーネントの `rounded-lg` → `var(--radius-lg)` を使うこと
- `className="font-sans"` → `font-family: var(--font-body)` を継承させること

---

## Icons（アイコン）

```
使用するアイコンライブラリ: <!-- TODO: Phosphor Icons / Radix Icons / カスタムSVG -->
禁止: Lucide + Inter の組み合わせ（AI生成の指紋になるため）
```

アイコン単体でセマンティクスを成立させないこと。必ず隣接するテキストラベルか aria-label を付ける。

---

## Copy Guidelines（コピー指針）

- **禁止フレーズ**: "Get Started", "Streamline", "Supercharge", "Enterprise-grade", "World-class", "Cutting-edge", "Innovative"
- **推奨**: 問題ベースのコピー（例: "72時間から15分に短縮する"）
- CTAボタン: 具体的なアウトカムを示す（例: "最初のプロジェクトを開始する"）

---

## AI Coding Instructions

このファイルを参照してUIを生成する際の指示:

1. 色は必ず `var(--brand-primary)` 等のカスタムプロパティを使うこと。`bg-indigo-*` 等の Tailwind デフォルト色は禁止
2. フォントは `var(--font-heading)` / `var(--font-body)` を使うこと。`font-sans` を直接使わない
3. 角丸は `var(--radius-md)` 等のトークンを使うこと。`rounded-2xl` を全要素に適用しない
4. 非同期データを扱うコンポーネントは Empty / Error / Loading の3状態を必ず実装すること
5. shadcn/ui を使う場合はデフォルト外観を上書きし、このファイルのトークンを使うこと
````
