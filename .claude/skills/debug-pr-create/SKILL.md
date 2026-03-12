---
name: debug-pr-create
description: デバッグ実行結果をもとにデバッグ用PRをdraftで作成する
tags: [workflow, debug, github, pr]
---

# デバッグ用PR作成スキル

デバッグプランファイルの情報をもとに、デバッグ用ブランチからdraft PRを作成します。
元PRへの参照、デバッグレポート、スクリーンショット・GIFのエビデンスをPR descriptionに含めます。

## 使い方

```bash
/debug-pr-create <debug-plan-ファイル名>
```

### 入力形式

| 形式         | 例                                                | 説明                                  |
| ------------ | ------------------------------------------------- | ------------------------------------- |
| ファイル名   | `debug-plan-20260311-1430.md`                     | outputs/debug-plans/ 配下のファイル名 |
| ファイルパス | `outputs/debug-plans/debug-plan-20260311-1430.md` | 相対パスまたは絶対パス                |

## 前提条件

- `debug-plan` スキルで作成されたデバッグプランファイルが存在すること
- `debug-implement` スキルでデバッグ実行が完了していること
- デバッグ用ブランチにチェックアウトしていること

## 実行フロー

### 1. デバッグプランの読み込み

引数からデバッグプランファイルを特定し、読み込みます。

**ファイル検索の優先順位:**

1. 引数がフルパスの場合 → そのまま読み込み
2. 引数がファイル名の場合 → `outputs/debug-plans/` 配下を検索

読み込み後、以下の情報を抽出:

- **対象PR番号**: 元PR番号
- **対象PR URL**: 元PR URL
- **対象PRタイトル**: 元PRタイトル
- **元ブランチ**: ベースブランチとして使用
- **デバッグ用ブランチ**: 現在のブランチと一致するか確認

さらに、元PRのフォーク情報を取得してベースブランチを決定:

```bash
# 元PRのフォーク判定とブランチ情報を取得
gh pr view <元PR番号> --json isCrossRepository,headRefName,baseRefName
```

- **同一リポジトリPR** (`isCrossRepository: false`): `headRefName` をベースブランチに使用
- **フォークPR** (`isCrossRepository: true`): `baseRefName` をベースブランチに使用
  - フォークPRの `headRefName` はフォーク側のブランチ名であり、`origin` に存在しないため

### 2. デバッグレポートの読み込み

```bash
# デバッグレポートを読み込み
cat outputs/debug-implements/<debug-plan-ファイル名（拡張子なし）>/debug-report.md
```

レポートから以下を抽出:

- 実行結果サマリー（OK / NG / 要確認 / スキップの件数）
- 不具合一覧
- スキップ項目

### 3. 事前確認

```bash
# 現在のブランチがデバッグ用ブランチであるか確認
git branch --show-current

# コミットされていない変更がないか確認
git status
```

- デバッグ用ブランチにいない場合 → ユーザーに確認の上チェックアウト
- 未コミットの変更がある場合 → ユーザーにコミットするか確認

### 4. エビデンスファイルのコミット

スクリーンショットやGIFなどのデバッグエビデンスがあることを確認します。

```bash
# 未コミットのエビデンスファイルを確認
git status outputs/debug-implements/
```

未コミットのエビデンスがある場合、ユーザーにコミットするか確認します。
コミットする場合:

```bash
git add outputs/debug-implements/<debug-plan-ファイル名（拡張子なし）>/
git add outputs/debug-plans/<debug-plan-ファイル名>
git commit -m "debug: PR #<PR番号> のデバッグエビデンスを追加"
```

### 5. リモートへのプッシュ

```bash
git push -u origin <デバッグ用ブランチ名>
```

**必ずユーザーに確認してからプッシュすること。**

### 6. Draft PRの作成

以下の形式でdraft PRを作成します。

**PRタイトル:**

```bash
[デバッグ] PR #<元PR番号> <元PRタイトル> の動作確認
```

**ベースブランチの決定:**

元PRがフォークPRかどうかで、ベースブランチを切り替える:

- **同一リポジトリPR** (`isCrossRepository: false`): 元PRの `headRefName` をベースブランチに指定
- **フォークPR** (`isCrossRepository: true`): 元PRの `baseRefName` をベースブランチに指定

**PR Bodyテンプレート:** `.claude/templates/debug-pr-body-template.md`

**PR作成コマンド:**

```bash
gh pr create \
  --draft \
  --base <ベースブランチ名> \
  --title "[デバッグ] PR #<元PR番号> <元PRタイトル> の動作確認" \
  --body "$(cat .claude/templates/debug-pr-body-template.md)"
# <ベースブランチ名> は Step 1 で決定した値を使用
# 同一リポジトリPR → headRefName / フォークPR → baseRefName
```

テンプレート内のプレースホルダー（`<PR番号>`, `<plan名>` 等）を実際の値に置換してからbodyに渡すこと。

### 7. ユーザーへの完了報告

```bash
デバッグ用PRを作成しました:

- PR URL: <作成したPR URL>
- タイトル: [デバッグ] PR #<元PR番号> <元PRタイトル> の動作確認
- ベースブランチ: <元ブランチ名>
- ステータス: Draft

このPRはデバッグエビデンスの共有用です。確認後クローズしてください。
```

## 注意事項

- **Draft PRとして作成**: 誤ってマージされないよう、必ずdraftで作成する
- **ベースブランチはPRの種類で決定**: 同一リポジトリPRでは `headRefName`、フォークPRでは `baseRefName` をベースにする（フォークPRの `headRefName` は `origin` に存在しないため）
- **コミット・プッシュは必ずユーザーに確認**: エビデンスのコミットとプッシュは必ずユーザーの許可を得てから行う
- **不具合がある場合は強調表示**: NG項目がある場合は、PR descriptionで目立つように記載する
