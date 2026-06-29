# Skill 改善スキル

Skill のベストプラクティスを元に、対象のスキルがそれに沿えているかをチェックし、改善案を提示する。

参考ドキュメントについては`docs/improving-skills/README.md`に記載している。

## インストール方法

```
npx skills add https://github.com/kazushi-tsuji/skills --skill improving-skills
```

## 設計思想

### `allowed-tools`

- スキルを検索, 読み込むのに必要なツールを指定
- 改善を適用するための Edit/Write はあえて含めず、ユーザーに確認を取る
- `Bash(ls *)` / `Bash(find *)` は意図的に広く取っている。絞りすぎると実際の探索コマンドに一致せず活用されないため。危険な操作は `disallowed-tools` 側でブロックしている

### 対象スキルの特定

ユースケースとして自作のスキルを改善するケースを一番に想定している。

そのため、実際にスキルが適用される優先順位(enterprise > personal > project)ではなく、 project > personal の優先順位で改善対象のスキルを特定する。

また、プラグインスコープ(`<plugin>/skills/<name>/SKILL.md`)は `~/.claude/plugins/cache/` 配下のキャッシュに配置され、ユーザーが管理するリポジトリではない。改善結果をリポジトリへコミットする本スキルの想定ワークフローに馴染まないため、探索対象から除外している。
