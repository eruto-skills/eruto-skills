---
name: cook-idea
description: "料理ナレッジVault（cooking/）の laws から新作料理を発想し、自己批判と機序検証まで回す。laws を category 横断で組み合わせ、制約を与えて多案生成し、各案を法則の conditions/evidence と突き合わせて絞り、ideas/ に書き出す。Trigger on: 「新しい料理を考えて」「lawから発想」「cook-idea」「料理の新作」。取り込み（動画→dish）は対象外——それは cooking/CLAUDE.md の取り込みパイプライン。"
user-invocable: true
allowed-tools: Read, Write, Edit, Grep, Glob, WebSearch, Bash, Task
argument-hint: "[制約: 食材/シーン/調理法]（省略可）"
---

# cook-idea

料理ナレッジVault `c:/@projects/cooking/` の **laws（法則）から新作 idea を発想し、検証まで回す**スキル。
`cooking/meta/automation-plan.md` の設計を実装したもの。Vaultの作法は `cooking/CLAUDE.md` に従う。

- 入力: `cooking/laws/*.md`
- 出力: `cooking/ideas/*.md`

## Workflow

1. **制約の決定** — 引数があれば食材/シーン/調理法として使う。無ければ新規性が出る制約を1つ選ぶ（例: 夏／弁当／酒の肴、鶏むね、レンジ完結、少材料）。ユーザーに一言確認してもよい。
2. **law の読み込み** — `Grep`/`Read` で `laws/*.md`（`_template.md` は除く）の frontmatter を集める（`claim`/`category`/`conditions`/`confidence`/`evidence`）。
   - 優先: `confidence` は 検証済 > 検証中 > 仮説、`evidence` は 裏づけあり を厚めに。
   - **反証あり/要注意 の law は捨てない。「正しい機序で使う」**（例: [[すりおろし香味野菜はとろみで絡みを作る]] は"とろみ"でなく"パルプ被膜による絡み"として使う）。
3. **組み合わせ** — `category`（味のバランス/テクスチャ/温度/香り/技法/味の設計/文脈/設計）の**異なる law を 3〜5 本**選ぶ。意図的に散らすと新規性が出る。制約を注入。
4. **多案生成** — 3〜5 案。似すぎる案は 1 つに畳む。
5. **自己批判（このスキルの肝）** — 各案を、適用した law の `conditions` と `evidence` に**機械的に突き合わせる**。
   - `conditions` に反する案 → 落とすか設計を直す。
   - `evidence: 反証あり/要注意` の law → 誤った機序を前提にしていないか点検し、正しい機序に設計し直す。
   - 案ごとに残った懸念を「想定リスク」に落とす。弱い案は捨てる。
6. **新規機序の検証** — idea が laws に無い**新しい機序主張**を持ち込むなら、`cooking/CLAUDE.md`「法則の検証（機序の裏づけ）」の手順で確認（WebSearch → 良質ソース、WebFetch禁止、俗説に注意）。強い裏づけが取れたら新 law 化を検討。
7. **出力** — 残った案を `cooking/ideas/_template.md` 形式で書く。
   - `applied_laws` に `[[リンク]]`、`## 自己批判（conditions / evidence との突き合わせ）` 節に突き合わせ結果、`status: 案`。
   - ファイル名は料理名の一意な basename（日本語可）。
8. **フィードバックの導線** — 各 idea 末尾に「作ったらどの law が裏づく/覆るか」を1行添える（試作結果を dishes/laws に書き戻す入口）。

## 出力形式（idea）

`cooking/ideas/_template.md` に準拠しつつ、次の節を必ず持つ:

```markdown
## 適用した法則
- [[法則名]] … どう効かせたか

## 自己批判（conditions / evidence との突き合わせ）
- **法則名**（evidence: …）: conditions/落とし穴に照らした点検結果と、設計への反映

## 想定リスク・確かめたい点
## 試作の結果
```

## 良い発想のコツ

- **category を散らす**より深掘りが効く題材もある。単調なら散らし、狙いが明確なら同カテゴリで攻める。
- 検証で"条件つき/落とし穴"の付いた law（例: 酸は振り切る、急冷はアルミ、香りは後入れだがスパイスは先入れ）は、**その条件ごと使うと失敗を先回りで避けられる**。
- 反証された機序（例: うま味調味料で乳化）を発想の根拠にしない。料理が"効く"事実と"なぜ効く"を分ける。

## Project Integration / Dependencies

- 対象 Vault: `c:/@projects/cooking/`（laws を入力、ideas を出力）。
- `WebSearch`（機序検証時）、`curl`（fetch-web フォールバック）。取り込み系ツール（yt-dlp等）は不要。
- 関連: 発想の設計思想は `cooking/meta/automation-plan.md`、検証ルールは `cooking/CLAUDE.md`。
