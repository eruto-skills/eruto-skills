---
name: mod-idea-dig
description: >
  minecraft-club の次回ワールド向け MOD/datapack 候補を発掘する半自動スキル。
  常設ソースを巡回 → 重複除去 → 設計意図8分類 → 候補ファイル下書き → 「要判断」リスト化、までを回す。
  `選定基準.md` をフィルタに使い、最終採否は人間に残す。
  「mod発掘」「候補を掘って」「mod-selection の dig を回して」「dig mod ideas」で使う。
user-invocable: true
allowed-tools: Read, Write, Edit, Grep, Glob, Bash, Agent, WebSearch
argument-hint: "[鉱脈名 or 空欄=全ソース]"
---

# mod-idea-dig

`minecraft-club/mod-selection/` の候補発掘を半自動で回すスキル。
**機械的な発掘・分類・下書きはこのスキルが、基準の定義と最終採否は人間が持つ。**

対象ディレクトリ: `c:/@projects/minecraft-club/mod-selection/`

## 入力（毎回まず読む）

- **基準（フィルタ）**: `選定基準.md`
- **既存候補（重複除外用）**: `README.md` の全ディレクトリ表 ＋ `統合版仕様アイデアプール.md` ＋ `アイデア発掘ノート.md`
- **常設ソース表**: `アイデア発掘ノート.md` の「常設ソース」節

## 手順

1. **基準と既存候補を読む**。重複除外リスト（既存の全候補ファイル名＋2プールの項目）を把握する。
2. **常設ソースを巡回**（引数で鉱脈を絞れる。空欄なら全ソース）。fetch-web戦略（`curl -s 'https://r.jina.ai/<URL>'` 主、失敗時 `fetch-page.js`）／WebSearch。
   - Vanilla Tweaks datapacks／Modrinth follows順（API `api.modrinth.com/v2/search`）／公式フィードバック votes順／r/minecraftsuggestions top（末尾 `.json`）／CurseForge 人気／Planet Minecraft。
   - 多ソース時は **Agent subagent を並列**で起動し、各ソースをキュレーションさせる。
3. **重複除去**: 既存候補＋2プールと突き合わせ、既出は捨てる。
4. **基準でフィルタ**（`選定基準.md`）:
   - 除外: 常設PvP／世界観不一致／C分類(グリッチ)／B分類(システム的・原則)／創作専用・プラットフォーム依存。
   - **主役の圧では絞らない**（未定のため pressure 候補は全部生かす）。
   - 各候補に **「datapackで足りるか / MODが要るか」** を明記。
5. **設計意図8分類へ仕分け**し、面白い順にキュレーション。
6. **下書き**（house style は既存ファイル踏襲。1ファイル＝作る1つの単位、関連は束ねてよい）:
   - 既存ディレクトリ入り → 該当 `N-dir/NN-<slug>.md` を作成し、`README.md` の当該表に行追加。
   - 統合版由来 → `統合版仕様アイデアプール.md`、他鉱脈 → `アイデア発掘ノート.md` のバックログに追記。
7. **「要判断」リストを出力**（採否は人間）: 世界観フィット△／主役の圧・総量に関わる／常設PvP・大型で世界を変える／版を上げれば入る、もの。
8. **検証**: lint（`）|` → `） |`）と壊れリンクを Grep で確認（`\.\./\.\.` や旧パスが残っていないか）。ディレクトリ名は番号プレフィックス付き（`1-pressure/` 等）。

## 出力

追加/更新したファイル一覧 ＋ **「要判断」リスト** をユーザーに提示。ユーザーが採否を打ったら、採用分を本表へ確定・不採用分は下書き削除。

## house style（候補ファイルの型）

やりたいこと → 統合版仕様/先例（既存MOD・版・ローダー）→ 自作方針（必要なら）→ **どうアレンジするか**（部活ワールド向け）→ 設定できるべき項目 → バランス・要検討 → クロスリンク。
見本: `1-pressure/01-thin-air.md`／`5-exploration/01-water-breathing.md`。

## 注意

- 番号プレフィックス: ディレクトリ `N-slug/`、ファイル `NN-english-slug.md`（ディレクトリ内で振り直し）。
- Markdown lint: テーブルはパイプ前にスペース（`） |`）、リスト/コードフェンスは前後に空行。
- 応答言語は日本語。
