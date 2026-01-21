---
description: GitHub Issueを受け取って実装を行う
allowed-tools: Bash(gh issue view:*), Bash(gh issue comment:*), Bash(git:*), Read, Write, Edit, Glob, Grep, Task, EnterPlanMode, ExitPlanMode, TodoWrite
---

# Implement Issue Command

GitHub Issueの内容を分析し、実装を行います。

## 使用方法

```
/implement-issue <issue番号>
```

または Issue URL を指定:
```
/implement-issue https://github.com/owner/repo/issues/123
```

## 実行手順

### 0. ブランチを切る
最新のmainブランチからブランチを切って作業を開始する。ブランチ名は適切に命名する。

### 1. Issueの取得と分析

まず、指定されたIssueの詳細を取得します：

```bash
gh issue view $ARGUMENTS --json number,title,body,labels,assignees,comments
```

取得した情報から以下を分析：
- **Issue の目的**: 何を達成すべきか
- **要件**: 具体的な要件・仕様
- **受け入れ条件**: 完了の判断基準（あれば）
- **技術的な制約**: 使用すべき技術やパターン
- **関連コード**: 既存のコードベースとの関連

### 2. コードベースの調査

Issue に関連するコードを調査：
- 既存の実装パターンを確認
- 変更が必要なファイルを特定
- 影響範囲を把握

### 3. Plan Mode に入る

`EnterPlanMode` ツールを使用して Claude Code の plan mode に入る。
plan mode では以下を行う：
- コードベースをさらに詳しく調査
- 実装アプローチの設計
- 計画ファイル（`.plan.md`）への実装計画の記述

### 4. 実装計画の作成（Plan Mode 内）

plan mode 内で `.plan.md` ファイルに詳細な実装計画を記述：

```markdown
## 概要
- Issue の要約
- 解決するべき課題

## 実装アプローチ
- 選択したアプローチとその理由
- 代替案があれば記載

## 変更するファイル
- 各ファイルの変更内容を具体的に

## 実装ステップ
1. ステップ1: 〇〇
2. ステップ2: 〇〇
...

## テスト計画
- 追加・修正するテスト

## リスク・懸念事項
- 予想される課題や注意点
```

### 5. ユーザー承認（ExitPlanMode）

`ExitPlanMode` ツールを使用してユーザーに計画の承認を求める：
- 実装に必要な Bash 権限があれば `allowedPrompts` で指定
- ユーザーが計画を確認し、承認または修正を指示
- 承認されるまで実装には移らない

### 6. 実装

承認された計画に基づいて実装を進める：
- TodoWrite で各ステップを管理
- 計画に沿って順番に実装
- 必要に応じてテストも作成
- lint/format を適用

### 7. 完了報告

実装完了後：
- 変更内容のサマリーを報告
- 確認が必要な点があれば提示
- 次のアクション（PR作成など）を提案

## 注意事項

- **実装前に必ず計画を提示**: いきなり実装を始めず、まず計画をユーザーに確認すること
- **既存のコーディング規約に従う**: プロジェクトのコーディング規約やパターンを確認して従う
- **段階的な実装**: 大きな変更は小さなステップに分割して進める
- **テストを考慮**: 可能であればテストも追加する
- **コミットは個別に作成しない**: 実装完了後、ユーザーがコミットを判断する

## Issueコメントへの返信（オプション）

ユーザーが希望する場合、実装開始時や完了時にIssueにコメントを追加：

```bash
gh issue comment $ARGUMENTS --body "実装を開始しました。
```
