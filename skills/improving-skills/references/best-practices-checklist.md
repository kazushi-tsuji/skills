# ベストプラクティスチェックリスト

出典: [Claude Code Skills docs](https://code.claude.com/docs/en/skills), [Agent Skills Best Practices](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices), [agentskills.io](https://agentskills.io/skill-creation/best-practices), [Optimizing descriptions](https://agentskills.io/skill-creation/optimizing-descriptions), [Evaluating skills](https://agentskills.io/skill-creation/evaluating-skills)

## 目次

1. [description（最重要）](#1-description最重要)
2. [naming（命名）](#2-naming命名)
3. [conciseness（簡潔さ）](#3-conciseness簡潔さ)
4. [progressive disclosure（段階的開示）](#4-progressive-disclosure段階的開示)
5. [control calibration（制御のキャリブレーション）](#5-control-calibration制御のキャリブレーション)
6. [structure patterns（構造パターン）](#6-structure-patterns構造パターン)
7. [code & scripts（コードとスクリプト）](#7-code--scriptsコードとスクリプト)
8. [content guidelines（コンテンツガイドライン）](#8-content-guidelinesコンテンツガイドライン)
9. [testing（テスト）](#9-testingテスト)
10. [frontmatter とツール権限](#10-frontmatter-とツール権限)
11. [改善優先度の判断基準](#改善優先度の判断基準)

---

## 1. description（最重要）

**目的**: description はスキルの発見メカニズム。Claude はここを見てスキルを呼び出すか判断する。

チェック項目:
- [ ] **三人称**で書かれているか（"I can..." / "You can..." は NG。"Processes Excel files..." の形式）
- [ ] **命令形フレーズ**で書かれているか（"Use this skill when..." / 「〜の場合は必ず使用すること」形式）
- [ ] ユーザーの**意図**（何を達成したいか）にフォーカスしているか（実装詳細ではなく）
- [ ] スキルが**何をするか**具体的に記述されているか
- [ ] **いつ使うべきか**のトリガー条件・キーワードが含まれているか
- [ ] アンダートリガーを防ぐため適度に "pushy"（能動的）な表現か
- [ ] 1024文字以内、XMLタグなし
- [ ] Claude Code の skill listing では `description` + `when_to_use` の合計が1,536文字で切り詰められる。主要なユースケースを先頭に配置しているか
- [ ] 主要なキーワードが含まれているか（ユーザーが実際に使いそうな表現）
- [ ] should-trigger / should-not-trigger クエリによる実測記録（evals内のトリガーテスト結果等）が対象スキルにあるか。**記録が存在しない場合はFailではなくN/Aとする**（improving-skills自身は対象スキルを反復実行してトリガー率を計測する手段を持たない）
  - 記録がある場合、以下の手法が踏まれているかを記録から確認する:
    - 合計 **20件程度**（should-trigger 8-10件 + should-not-trigger 8-10件）用意されているか
    - should-trigger: フレーズの多様性（フォーマル/カジュアル/略語）、明示的/暗示的、詳細度・複雑度に幅があるか
    - should-not-trigger: キーワードは共通だが別タスクになる **near-miss** を中心にしているか（「明らかに無関係」なクエリばかりでないか）
    - クエリに具体性があるか（ファイルパス、個人的な文脈、カジュアルな口語・誤字）
    - クエリが **60% トレイン / 40% バリデーション** に分割されているか
    - 各クエリを**3回実行**した trigger rate（発火した割合）が記録され、**閾値0.5**でPASS/FAIL判定されているか（目視確認のみで済ませていないか）

**良い例**:
```yaml
description: Excel ファイルを分析してピボットテーブル・グラフを生成する。Excel ファイル、
  スプレッドシート、表形式データ、.xlsx ファイルを扱う際に使用すること。
```

**悪い例**:
```yaml
description: ドキュメントを処理します
```

---

## 2. naming（命名）

チェック項目:
- [ ] 小文字・数字・ハイフンのみ（最大64文字）
- [ ] **動名詞形**（processing-pdfs）または**アクション形**（process-pdfs）を使用
- [ ] 予約語（"anthropic", "claude"）を含まない
- [ ] 意味が明確（"helper", "utils", "tools" などの曖昧な名前でない）

---

## 3. conciseness（簡潔さ）

**原則**: Claude はすでに賢い。Claude が既に知っていることは書かない。

チェック項目:
- [ ] SKILL.md 本文が **500行未満**か（目安として5,000トークン未満）
- [ ] compaction 後も重要な指示が残るよう、**最重要な指示を SKILL.md の冒頭**に配置しているか（compaction 後は先頭 5,000 トークンのみ保持される）
- [ ] 一度ロードされたスキル内容はセッション中保持され続け、後続ターンで再読込されない。ワンタイムの手順ではなく、**タスク全体を通じて有効な標準指示**として書かれているか
- [ ] スキルのスコープが**適切に定義**されているか（狭すぎず・広すぎず）
  - 狭すぎ: 単一タスクごとにスキルが乱立し、1タスクに複数スキルが必要になる
  - 広すぎ: 無関係な機能が1スキルにまとまり、正確なトリガーが困難になる
- [ ] Claude が既に知っていることの説明がないか
  - 悪い例: "PDF（Portable Document Format）はよく使われるファイル形式です..."
  - 良い例: ライブラリ名と具体的なコードスニペットだけ示す
- [ ] 各段落・各文が付加価値を持っているか（「必要か？」と自問する）
- [ ] 詳細な参照資料は別ファイルに分離されているか

---

## 4. progressive disclosure（段階的開示）

**原則**: SKILL.md はコアのみ。追加情報は必要時に読み込む。

チェック項目:
- [ ] 大量コンテンツは `references/` や `assets/` に分離されているか
- [ ] 参照ファイルへのリンクに「**いつ読むべきか**」が明示されているか
  - 良い例: "Form 入力については [FORMS.md](FORMS.md) を参照"（いつ読むか明確）
  - 悪い例: "詳細は references/ を参照"（曖昧）
- [ ] 参照が **1段階のみ**か（ネストした参照は避ける）
- [ ] 100行超の参照ファイルには **目次**があるか
- [ ] 複数ドメインを扱うスキルは、ドメインごとに参照ファイルを分割しているか（例: `reference/finance.md`, `reference/sales.md`）
- [ ] 参照ファイル名は内容を表す具体的な名前か（`doc2.md` ではなく `form_validation_rules.md`）

---

## 5. control calibration（制御のキャリブレーション）

**原則**: タスクの脆弱性・一貫性要件に応じて自由度を調整する。

チェック項目:
- [ ] **高脆弱タスク**（DBマイグレーション等）には具体的な指示があるか
- [ ] **低脆弱タスク**（コードレビュー等）には大まかな方針のみで過剰な制約がないか
- [ ] 複数の選択肢ではなく**デフォルト＋例外**の形式になっているか
  - 悪い例: "pypdf、pdfplumber、PyMuPDF、pdf2image のいずれかを..."
  - 良い例: "テキスト抽出には pdfplumber を使用。OCR が必要なスキャン文書は pdf2image+pytesseract を使用"
- [ ] **手順（Procedures）** が**宣言（Declarations）** より重視されているか
  - 手順: 問題の種類に対してどう対処するか（再利用可能）
  - 宣言: 特定インスタンスの答え（使い捨て）

---

## 6. structure patterns（構造パターン）

### Gotchas セクション

最も価値の高いコンテンツ。合理的な推測が外れる環境固有の落とし穴。

チェック項目:
- [ ] 合理的な推測が外れる**具体的な落とし穴**が記載されているか
- [ ] 一般論ではなく**このプロジェクト固有**の事実が含まれているか

良い例:
```markdown
## Gotchas
- `users` テーブルはソフトデリート方式。WHERE deleted_at IS NULL が必須
- user_id（DB）、uid（認証サービス）、accountId（請求API）はすべて同じ値を指す
```

### テンプレート

- [ ] 出力フォーマットが必要な場合、**具体的なテンプレート**が提供されているか（説明文より確実）

### チェックリスト

- [ ] 多段階ワークフローで Claude が進捗を追跡できる**チェックリスト**があるか

### バリデーションループ

- [ ] 品質が重要な操作で「実行→検証→修正→再検証」のループが含まれているか
- [ ] 破壊的・バッチ操作に **Plan-Validate-Execute** パターンが使われているか
  - 計画ファイル（JSON等）を作成 → バリデーションスクリプトで検証 → 実行

### Examples パターン

- [ ] 出力のスタイル・トーンの再現性が重要な場合、input/output のペア例が示されているか（説明文より具体例の方が伝わりやすい）

---

## 7. code & scripts（コードとスクリプト）

**`scripts/` ディレクトリが存在しない場合は本セクション全体を N/A とする。**

チェック項目:
- [ ] **Windowsスタイルのパス（`\`）**を使っていないか（常に `/` を使用）
- [ ] 必要なパッケージが**明示**されているか（「インストール済み前提」でない）
- [ ] スクリプトが問題を解決しているか（Claude に丸投げしていないか）
- [ ] スクリプトがエラーケースを自前で処理しているか（FileNotFound・権限エラー等を Claude に任せていないか）
- [ ] **マジックナンバー**にコメントで説明があるか（なぜその値か）
- [ ] MCP ツール参照に**完全修飾名**（`Server:tool_name` 形式）を使用しているか
- [ ] スクリプトを「実行する」のか「参照として読む」のか指示が明確か（例: "Run analyze_form.py" と "See analyze_form.py for the algorithm" を混同していないか）
- [ ] 繰り返し登場する共通処理が `scripts/` にバンドルされているか
- [ ] `--help` でスクリプトのインターフェース（フラグ・使用例）が確認できるか
- [ ] データ出力（stdout）と診断メッセージ（stderr）が分離されているか
- [ ] 破壊的・バッチ操作に冪等性（再実行しても安全）への配慮があるか

---

## 8. content guidelines（コンテンツガイドライン）

チェック項目:
- [ ] **時間依存の情報**がないか（"Before August 2025..." 等）
  - 必要なら `<details>` タグで "Old patterns" セクションとして格納
- [ ] **用語が一貫**しているか（同じ概念に複数の用語を使っていない）
  - 悪い例: "API endpoint"、"URL"、"API route"、"path" を混在
- [ ] サンプルや例が**具体的**か（抽象的でない）

---

## 9. testing（テスト）

**原則**: スキルは評価駆動で開発・改善する。大量のドキュメントを書く前に評価シナリオを作成し（eval-first）、実際の使用前に性能を検証する。

**`evals/` や実行記録が対象スキルに存在しない場合、本セクションは Fail ではなく N/A として扱う。一部の記録しかない場合（例: `evals.json` はあるが `grading.json`/`timing.json` がない）も、該当する記録が存在しない項目は個別にN/Aとする。**

**作業順序や実行時の一時情報（いつ assertions を追加したか、いつ timing.json を保存したか等）に関する項目は、対象スキル側にそれを示す記録（コミット履歴・ドキュメント上の言及等）がない限り遡って確認できない。判定不能な場合はFailにせずN/Aとする。**

チェック項目:
- [ ] 最低3件の evaluation シナリオ（入力・期待動作）が作成されているか
- [ ] 対象モデル（Haiku・Sonnet・Opus）で動作確認済みか
- [ ] テストシナリオだけでなく実際のユースケースで確認済みか
- [ ] Claude の実際の動作を観察したか（予期しないファイル読み込み順序・参照リンクの見落とし・特定セクションへの過依存・アクセスされないファイルがないか）
- [ ] チームフィードバックを収集したか（該当する場合）
- [ ] 実行中に発生したエラー・想定外の挙動を **Gotchas に追記したか**（observe → fix → add gotcha のサイクル）
- [ ] **最初のラン結果を確認してから** assertions（合否を判定できる具体的な検証基準）を各 eval シナリオに追加したか（ラン前に定義すると実態と乖離した検証基準になりやすい）
- [ ] スキルあり / スキルなし の出力を比較し、スキルの付加価値を確認したか
- [ ] eval ワークスペースを `iteration-N/eval-<name>/with_skill/` 形式で整理したか
- [ ] 各実行後に `timing.json`（total_tokens, duration_ms）を**即時**保存したか（後から取得不可）
- [ ] `grading.json` の assertion_results が `text` / `passed` / `evidence` フィールドを使っており、`evidence` に具体的な引用があるか（`name`/`met`/`details` でない、かつラベルはあるが内容が薄いだけで安易にPASSにしていないか）
- [ ] `benchmark.json` で with_skill / without_skill の pass_rate・time・tokens の **delta** を算出したか
- [ ] **どちらでも常に PASS** するアサーション（非弁別的）を除外したか
- [ ] **どちらでも常に FAIL** するアサーションを修正したか（アサーション自体の問題）
- [ ] スキルありのみ PASS するアサーションを把握し、スキルが実際に貢献しているか確認したか
- [ ] 実行ログを読んで無駄な処理・曖昧な指示を特定し、インストラクションをスリム化したか
- [ ] 人間レビューの feedback を次イテレーションに反映したか（空フィードバック=問題なし）
- [ ] スキル改善指示を LLM に与える際、「特定例へのパッチ」ではなく「根本原因への汎化」を求めたか
- [ ] 同一evalの結果が実行ごとに大きくばらつく場合（stddevが高い）、eval設計のフレーキーさかスキル指示の曖昧さかを切り分けたか
- [ ] 時間・トークンが突出して大きいevalがあれば、実行ログ（transcript）を読んで原因を特定したか

---

## 10. frontmatter とツール権限

**目的**: フロントマターは呼び出し制御・権限範囲を左右する。誤設定は意図しない自動実行や過剰な権限付与につながる。

チェック項目:
- [ ] 副作用のある操作（deploy, commit, 外部送信等）を行うスキルに `disable-model-invocation: true` が設定されているか（Claude が判断で自動実行すべきでない）
- [ ] ユーザーが直接呼び出す意味がない背景知識スキルに `user-invocable: false` が設定されているか
- [ ] `allowed-tools` が必要最小限か（ワイルドカードで過剰な権限を自己付与していないか）
- [ ] プロジェクトスコープのスキルは `allowed-tools` がワークスペース信頼後に有効になる点を踏まえ、リポジトリ由来のスキルとして妥当な権限か
- [ ] `context: fork` 使用時、会話履歴を持たない独立実行を前提とした自己完結的なタスク記述になっているか
- [ ] `context: fork` 使用時、`agent` フィールドが指定されている場合はタスクに適した subagent か（省略時は `general-purpose` になる点も踏まえる）
- [ ] `$ARGUMENTS` / `$N` / `arguments`+`$name` 等の文字列置換を使う場合、宣言と使用箇所が整合しているか（`arguments` 未宣言のまま `$name` を使っていないか等）
- [ ] モノレポ等で特定ディレクトリ配下のみで有効化すべき場合、`paths` フィールドの活用を検討したか

**悪い例**:
```yaml
allowed-tools: Bash(*)
```

**良い例**:
```yaml
name: deploy
description: Deploy the application to production
disable-model-invocation: true
allowed-tools: Bash(git push *) Bash(npm run deploy)
```

---

## 改善優先度の判断基準

| カテゴリ | 影響範囲 | 優先度 |
|---|---|---|
| description の問題 | スキルが全くトリガーされない | 🔴 高 |
| frontmatter/権限の問題 | 意図しない自動実行・過剰な権限付与 | 🔴 高 |
| 構造的問題（500行超等） | パフォーマンス劣化 | 🔴 高 |
| 冗長コンテンツ | コンテキスト圧迫 | 🟡 中 |
| 制御キャリブレーション不足 | 出力品質のばらつき | 🟡 中 |
| Gotchas/パターン不足 | エラー発生率 | 🟡 中 |
| テスト不足 | 品質の検証漏れ | 🟡 中 |
| 用語不一貫・形式問題 | 軽微な混乱 | 🟢 低 |
