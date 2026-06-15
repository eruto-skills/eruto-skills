---
name: design-brief
description: >
  Remotionプレゼンテーションのビジュアルデザインを決める。コンテンツのアウトラインを読み、感情の流れとビジュアルメタファーを分析し、2〜3案のデザイン方向性（配色・書体・アニメーション言語・ムード）を提示する。発表者が案を選んだあと、`tech/remotion/src/themes/<name>.ts` を生成する。

  次のとき必ず使うこと: 「デザインを考えたい」「ビジュアルを決めたい」「テーマを作って」「配色どうしよう」「デザイン方向性を出して」「こだわりたい」「/design-brief」。
  新しいRemotionプレゼンを始めていてテーマがまだ決まっていないときも、自発的に使う。
user-invocable: true
allowed-tools: Read, Write, Glob, Grep, WebSearch
argument-hint: "[content-outline file path, or presentation description]"
---

# design-brief

コンテンツのアウトラインからビジュアル仕様を作るスキル。3フェーズで進む:

1. コンテンツを読んでビジュアル方向性を分析する
2. 2〜3案のデザイン方向性を提示し、発表者に選ばせる
3. 選ばれた案のRemotion Themeファイルを生成する

目的は「先にコードを書く」ことではなく「先に本当の選択肢を示す」こと。発表者が見て、自分のプレゼンがどう見えるか想像できる案を出す。

## フェーズ1: コンテンツ分析

引数があればそのファイルを Read する。なければ発表者にファイルパスまたはインライン説明を聞く。

内容から以下を読み取る（結果は直接出力せず、フェーズ2の案に反映する）:

- **感情アーク**: 各セクションで聴衆に何を感じさせたいか。例: 「疑問 → 驚き → 納得」
  - **重要**: コンテンツファイルがなく説明のみの場合、感情アークは架空の推測になる。この場合は表に出さず内部参考にとどめる。発表者が共感できるアークを作るには実際のコンテンツ（アウトライン・ナレーション）が必要。
- **キービジュアルメタファー**: このトピックを視覚化するとしたら何か。暗号なら「鍵・封筒・鍵かけ・数学式」など、具体的に
- **オーディエンスコンテキスト**: 誰が見るか・何を知っているか・何を求めているか
- **プレゼンのトーン**: 真剣↔ゆるい・テクニカル↔親しみやすい・講義↔物語、の軸でどこにいるか

必要に応じてWebSearchでビジュアルリファレンスを探す（例: 「cryptography presentation design」「security dark UI aesthetic」）。

## フェーズ2: デザイン方向性の提示

2〜3案をカード形式で提示する。各案は「単なる色違い」ではなく、異なるビジュアル人格・メタファーを持つこと。

→ カードの書き方の詳細は [design-directions-format.md](references/design-directions-format.md) を参照

提示のポイント:
- 案名は具体的なイメージが浮かぶ名前にする（「Dark Theme」ではなく「Ancient Codex」など）
- 案の間のトレードオフを明確にする。発表者が迷わず選べる情報を添える
- 各案は感情アークから逆算する。デザインはコンテンツに奉仕するもの
- 各案に @presentations/remotion-ds のアニメーション語彙を当てる（どの関数がこの案に合うか）

提示後: 「どの方向性が合いそうですか？要素を組み合わせることもできます。」と聞く。

## フェーズ3: Themeファイル生成

発表者が案を選んだら:

1. `tech/remotion/src/themes/<name>.ts` を生成する
   → 構造の詳細は [theme-file-template.md](references/theme-file-template.md) を参照

2. `tech/remotion/src/index.ts` に export を追加する

3. 以下を含む簡潔なシーン設計メモを出力する:
   - このプレゼンの主要なシーンタイプに対するレイアウト方針
   - この案のペース感に合うアニメーション関数の推奨（`animation.ts` から）
   - カスタムビジュアル要素が必要な場合はその概要（SVGアイコン・装飾パターン等）
   - 非標準フォントを使う場合は `Root.tsx` へのGoogle Fonts追加メモ

## プロジェクト連携

このスキルは `@presentations/remotion-ds` デザインシステム（`tech/remotion/`）と連携する。

| ファイル | 役割 |
|---|---|
| `tech/remotion/src/theme.ts` | Theme インターフェース定義 |
| `tech/remotion/src/themes/` | 既存テーマ（参考に読む） |
| `tech/remotion/src/animation.ts` | アニメーション語彙 |
| `tech/remotion/src/index.ts` | export 追加先 |
| `slides/<name>/video/src/Root.tsx` | Google Fonts 読み込み先 |

新しい案を作る前に `tech/remotion/src/themes/` の既存テーマを1〜2件読んで、トークン構造・命名の慣例を確認すること。
