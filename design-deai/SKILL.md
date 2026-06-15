---
name: design-deai
description: >
  AIコーディングツール（Claude Code・Cursor・v0・Lovable等）が生成したUIコードに含まれる
  「AIスロップ」パターン（紫グラデ・Inter固定・rounded-2xl乱用・グラスモーフィズム全面適用等）を
  ソースコード静的解析で検出し、DESIGN.md を生成して今後のAI出力を脱スロップ化するスキル。
  「AI臭いUI直して」「デザインスロップ直して」「DESIGN.md作って」「vibe coding UI改善」
  「AI臭さ消したい」「脱AI化」「anti-slop」「AIデザインアンチパターン直して」
  「AIが作ったUIがダサい」など、AI生成UIのデザイン品質改善を求める発言では必ずこのスキルを使うこと。
  ui-visual-reviewerが実描画ベースのWCAG監査であるのに対し、
  design-deaiはソースコードのパターン検出と予防的DESIGN.md生成に特化しており補完関係にある。
user-invocable: true
allowed-tools: Read, Write, Glob, Grep, Bash
argument-hint: "[プロジェクトディレクトリパス（省略時はカレントディレクトリを解析）]"
---

# design-deai

AIコーディングツールが生成したUIコードの「AI臭さ」を診断し、脱スロップ化のための DESIGN.md を生成するスキル。

## なぜこのスキルが必要か

Claude Code・Cursor・v0 等のAIツールは学習データの統計的平均に収束するため、指示なしだと毎回同じパターンを出力する。
このスロップを防ぐ根本的な解決策が「DESIGN.md」だ。プロジェクトのカラー・フォント・スペーシングルールをファイルに明示し、AIコーディングツールが毎セッションそれを参照することで、スロップが再生産されなくなる。

| 典型的AIスロップ | 原因 |
|----------------|------|
| `bg-indigo-500` / 紫グラデーション | Tailwind デフォルト色が学習データを汚染 |
| Inter フォント固定 | Figma→Tailwind→Next.js エコシステムの最頻出フォント |
| `rounded-2xl` 全要素適用 | shadcn/ui デフォルト値の統計的再現 |
| グラスモーフィズム全面適用 | 2020年代前半の SaaS テンプレートの平均 |
| Empty/Error State なし | ハッピーパスのみが学習データに多い |

→ アンチパターン詳細: [antipattern-catalog.md](references/antipattern-catalog.md)

## ワークフロー

### Phase 1: スコープ確認

解析対象を決定する。

- 引数でパスが指定された場合はそのディレクトリを対象とする
- 引数なしの場合はカレントディレクトリを対象とする
- 対象ファイル: `**/*.{css,scss,tsx,jsx,ts,js}` および `tailwind.config.*`

対象ファイルが 200 件を超える場合は `src/` 以下に絞って解析する旨をユーザーに伝えてから続行する。

### Phase 2: AIスロップパターン検出

以下の9カテゴリを Grep で検出する。検出パターンの詳細は [antipattern-catalog.md](references/antipattern-catalog.md) を参照。

#### 2-1. カラー・グラジエント（重症度: 高）

```bash
grep -rn "indigo-[0-9]\|purple-[0-9]\|violet-[0-9]\|from-indigo\|to-purple\|to-violet\|from-purple" \
  --include="*.tsx" --include="*.jsx" --include="*.ts" --include="*.js" --include="*.css" .
```

```bash
grep -rn "backdrop-blur\|backdrop-filter\|bg-gradient\|rgba.*0\.[12][^0-9]" \
  --include="*.tsx" --include="*.jsx" --include="*.css" --include="*.scss" .
```

#### 2-2. タイポグラフィ（重症度: 中）

```bash
grep -rn "Inter\|Roboto\|Open.Sans\|Noto.Sans\|font-sans\b" \
  --include="*.tsx" --include="*.jsx" --include="*.css" --include="*.scss" .
```

#### 2-3. 角丸過剰（重症度: 低〜中）

```bash
grep -rn "rounded-2xl\|rounded-3xl\|rounded-full\|border-radius.*2[0-9]px" \
  --include="*.tsx" --include="*.jsx" --include="*.css" --include="*.scss" .
```

#### 2-4. シャドウ・グロー（重症度: 中）

```bash
grep -rn "shadow-lg\|shadow-xl\|shadow-2xl\|box-shadow.*rgba.*0\.[3-9]\|glow\|drop-shadow-lg" \
  --include="*.tsx" --include="*.jsx" --include="*.css" --include="*.scss" .
```

#### 2-5. 状態設計（重症度: 高）

```bash
# Empty/Error State の存在確認（見つからなければ欠如を記録）
grep -rn "EmptyState\|ErrorState\|empty.state\|error.state\|NoData\|エラー画面" \
  --include="*.tsx" --include="*.jsx" .
```

#### 2-6. ローディングパターン（重症度: 中）

```bash
grep -rn "Skeleton\|skeleton\|Spinner\|spinner\|loading" \
  --include="*.tsx" --include="*.jsx" .
```

#### 2-7. デザイントークン（重症度: 高）

```bash
# CSS カスタムプロパティの使用状況
grep -rn "var(--\|:root\s*{" --include="*.css" --include="*.scss" .
# Tailwind テーマ拡張
grep -rn "theme.*extend\|colors:" tailwind.config.* 2>/dev/null || echo "tailwind.config なし"
```

#### 2-8. 汎用コピー（重症度: 低）

```bash
grep -rn "Get Started\|Streamline\|Supercharge\|Enterprise.grade\|World.class\|lorem ipsum\|placeholder" \
  --include="*.tsx" --include="*.jsx" --include="*.html" .
```

#### 2-9. DESIGN.md 不在確認

```bash
ls DESIGN.md 2>/dev/null || echo "DESIGN.md なし"
```

### Phase 3: スロップスコア算出

検出結果を集計してスロップスコア（0〜100）を算出し、重症度ごとに分類する。

| スコア | 判定 | 対応方針 |
|-------|------|---------|
| 0〜30 | クリーン | DESIGN.md で防止ルールを明文化するだけで十分 |
| 31〜60 | 要注意 | 主要パターンが定着。DESIGN.md + 段階的リファクタ推奨 |
| 61〜100 | ヘビースロップ | 全面的な DESIGN.md 制約 + 優先箇所の即時修正が必要 |

算出方法:
- **高重症度** パターン（カラー・状態設計欠如・トークン不在）: 各最大30点
- **中重症度** パターン（タイポグラフィ・シャドウ・ローディング）: 各最大10点
- **低重症度** パターン（角丸・汎用コピー）: 各最大5点

検出件数が 10 件未満なら重症度のウェイトの半分、10 件以上なら全点。

### Phase 4: DESIGN.md 生成

スロップスコアと検出結果に基づいて `DESIGN.md` を生成する。
テンプレートは [design-md-template.md](references/design-md-template.md) を参照。

DESIGN.md には以下のセクションを含める:

1. **禁止事項 (What NOT to use)**: 検出されたスロップパターンを明示禁止
2. **カラーシステム**: 既存コードから抽出した色、またはOKLCHによる推奨パレット
3. **タイポグラフィ**: 使用フォントと代替フォントの指定
4. **スペーシング・角丸**: 許可する値の一覧
5. **状態設計ルール**: Empty/Error/Loading State の必須化
6. **コンポーネントルール**: shadcn/ui デフォルト外観の上書き方針

既存コードにブランドカラーやカスタムフォントが見つかった場合は、それを DESIGN.md に反映する。
見つからない場合は「未設定」として TODO を記載し、ユーザーが後から埋められるようにする。

### Phase 5: 改善レポートの出力

以下の形式でレポートをユーザーに提示する:

```
## design-deai 診断レポート

**スロップスコア: XX/100** （判定: 要注意）

### 検出されたアンチパターン（優先度順）

1. [HIGH] カラー: `bg-indigo-500` が N 箇所検出
   → DESIGN.md の --brand-primary に置換し、AI生成時に参照させる
2. [HIGH] デザイントークン不在: CSS カスタムプロパティが未定義
   → DESIGN.md の :root セクションに変数を定義する
3. [MEDIUM] タイポグラフィ: Inter 固定が N 箇所
   → Satoshi または Geist への変更を DESIGN.md に明記する
...

### 生成したファイル
- DESIGN.md （プロジェクトルート）

### 次のステップ
1. DESIGN.md を確認し、TODO 箇所を埋める
2. AIコーディングツールのセッション開始時に DESIGN.md を読ませる
3. ui-visual-reviewer で実描画の WCAG 対比を検証する（推奨）
```

## Cross-Skill Integration

| スキル | 役割 | 連携タイミング |
|-------|------|-------------|
| `ui-visual-reviewer` | 実描画のカラー対比・WCAG監査 | design-deai 完了後に実行し視覚的に確認 |
| `design-brief` | デザイン方向性・ブランドアイデンティティの決定 | DESIGN.md 生成前に方向性を固める場合 |

## 制約・既知の限界

- **動的クラス**: `cn()` や `clsx()` での動的クラス生成は grep では完全に捕捉できない
- **Tailwind v4**: CSS-first 設定（`@theme` 構文）は v3 系の grep パターンでは検出できない場合がある
- **画像・SVG**: 視覚的要素はソースコード検出の対象外。視覚審査は `ui-visual-reviewer` を使うこと
- **False Positive**: `from-indigo` が意図的なブランドカラーである場合もある。検出件数と文脈を確認すること
