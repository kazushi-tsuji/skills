---
name: improving-skills
description: >
  既存スキルを読み込み、Agent Skills ベストプラクティスに基づいて分析・改善を施す。
  優先度付き分析レポート（🔴高/🟡中/🟢低）と before/after 形式の改善案を提示し、
  ユーザーの承認を得てから変更を適用する。
when_to_use: >
  「スキルを改善して」「description が弱い」「スキルが長すぎる」「スキルがトリガーされない」
  「スキルの品質を上げたい」「このスキルをレビューして」「スキルを見直したい」
  「スキルを評価して」「スキルを診断して」「スキルをチェックして」と言われたとき、
  また特定スキルに問題を感じたときに必ず使用すること。
allowed-tools:
  - Read(//**/.agents/skills/**)
  - Read(//**/.claude/skills/**)
  - Read(//**/skills/improving-skills/references/best-practices-checklist.md)
  - Bash(ls *)
  - Bash(find *)
disallowed-tools:
  - Bash(find * -delete *)
  - Bash(find * -exec *)
  - Bash(find * -execdir *)
  - Bash(find * -ok *)
  - Bash(find * -okdir *)
  - Bash(find * -fprintf *)
  - Bash(find * -fprint *)
  - Bash(find * -fls *)
---

# スキル改善

既存スキルを読み込み、ベストプラクティスに沿った分析と改善を行います。

作業開始時に以下をコピーして進捗管理します:

```
進捗:
- [ ] Step 1: 対象スキルの特定
- [ ] Step 2: スキル全体の読み込み
- [ ] Step 3: チェックリストによる分析
- [ ] Step 4: 改善案の提示・ユーザー確認
- [ ] Step 5: 改善の適用
- [ ] Step 6: 改善後の確認
```

## Step 1: 対象スキルの特定

スキルが指定されていない場合はユーザーに確認します。検索する優先順位：

1. プロジェクトスコープ: `.claude/skills/<name>/SKILL.md`（リポジトリルートまでの各階層、および作業対象ファイル配下のネストした `.claude/skills/` も含む）
2. ユーザースコープ: `~/.claude/skills/<name>/SKILL.md`

同名スキルが複数階層に存在する場合は project > personal の優先順位、またはディレクトリ修飾名（例: `apps/web:deploy`）で対象を一意に特定する。なお enterprise スコープ（managed settings 経由）はファイルパス走査で検索できないため、プラグインスコープ（`<plugin>/skills/<name>/SKILL.md`）は改善結果をリポジトリへコミットする本スキルの想定ワークフローに馴染まないため、いずれも対象外とする。

## Step 2: スキル全体の読み込み

Step 1 で特定したディレクトリの全ファイルを把握する。SKILL.md に加え、`references/`・`assets/`（参照コンテンツ）、`scripts/`（レビュー対象）、`evals/`（テスト観点）が存在する場合は内容を確認する。

## Step 3: ベストプラクティスチェックリストによる分析

[references/best-practices-checklist.md](references/best-practices-checklist.md) を読み込み、各チェック項目（□）を1つずつ対象スキルに当てはめて Pass / Fail を判定します。

## Step 4: 改善案の提示

分析結果を以下の形式でまとめてユーザーに提示します。

**改善点がある場合:**

```
## 分析結果: <スキル名>

### 🔴 重要度 高（description・トリガー・構造の根本的問題）
- ...

### 🟡 重要度 中（冗長性・制御キャリブレーション・パターン不足）
- ...

### 🟢 重要度 低（表現・用語の一貫性など）
- ...

### ✅ 良い点
- ...
```

**改善点がない場合:**

```
## 分析結果: <スキル名>

### ✅ 良い点
- ...

改善は不要です。
```

**記入例:**

```
## 分析結果: pdf-filling

### 🔴 重要度 高（description・トリガー・構造の根本的問題）
- description が「PDFを処理します」のみで、いつ使うべきかのトリガー条件が書かれていない。「PDFフォームへの入力」「PDF記入」等のキーワードを追加すべき

### 🟡 重要度 中（冗長性・制御キャリブレーション・パターン不足）
- SKILL.md 本文が650行あり、ライブラリ選定の背景説明が長い。pypdf/pdfplumber/PyMuPDF の比較部分を references/library-comparison.md に切り出すべき

### 🟢 重要度 低（表現・用語の一貫性など）
- 「フォーム」と「入力欄」が同じ概念に対して混在している

### ✅ 良い点
- Gotchas セクションにスキャンPDFのOCR精度に関する具体的な注意点が記載されている
```

## Step 5: 改善の適用

ユーザーの承認を得てから編集します。編集前に変更内容を before/after 形式で表示して確認を取ります。承認された変更のみ適用し、未承認の変更はスキップします。

**before/after 形式:**
```
before: description: スキルを処理します
after:  description: 既存スキルを分析し改善案を提示する。「スキルを改善して」と言われたときに使用すること。
```

## Step 6: 改善後の確認

改善を適用したら、[references/best-practices-checklist.md](references/best-practices-checklist.md) の各カテゴリを再確認します。残存する Fail 項目があれば before/after 形式で修正を提案し、ユーザーの確認を取ります。修正を適用したら Step 6 を繰り返します。ユーザーが「直さない」と判断した項目は承認済み例外として扱い、次の確認に進みます。全項目が Pass になった時点、または残存する Fail 項目がすべて承認済み例外になった時点で完了とします。

## Gotchas

- スキル名が指定されていない場合、会話履歴から勝手に推測しない — 必ずユーザーに確認する
- SKILL.md だけでなく `references/`・`scripts/` 等のファイルも把握してから分析する（SKILL.md 単体では評価が不完全になる）
- 「削除」は削除前にユーザーへ確認を取る。判断が難しい場合はユーザーに委ねる
- 「スキルが発火しない」という相談でも、対象タスクが単純な1ステップ作業（例:「このPDFを読んで」）の場合は description 強化だけでは発火率が上がらないことがある。Claude は基本ツールで対応できる単純タスクではスキルを介さない傾向があるため、タスクの複雑さ・専門性を踏まえて助言する
