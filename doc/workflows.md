# GitHub Actions ワークフロー設定ドキュメント

`.github/workflows/` 以下に設定されているワークフローについて説明します。

---

## 1. Claude Code (`claude.yml`)

### 概要

Issue やプルリクエストのコメントで `@claude` とメンションすることで、Claude AI を呼び出して各種タスクを実行させるワークフローです。

### トリガー条件

| イベント | 条件 |
|---|---|
| `issue_comment` | コメントが作成されたとき |
| `pull_request_review_comment` | PRのレビューコメントが作成されたとき |
| `issues` | Issue がオープンまたはアサインされたとき |
| `pull_request_review` | PRレビューが提出されたとき |

コメント本文または Issue 本文に `@claude` が含まれる場合のみジョブが実行されます。

### 必要なシークレット

| シークレット名 | 説明 |
|---|---|
| `CLAUDE_CODE_OAUTH_TOKEN` | Claude Code の OAuth トークン |

### パーミッション

```yaml
permissions:
  contents: read
  pull-requests: read
  issues: read
  id-token: write
  actions: read
```

### 主な動作

1. リポジトリをチェックアウト
2. `anthropics/claude-code-action@v1` を使用して Claude を起動
3. コメントの内容に応じてコード変更、質問への回答、コードレビューなどを実行

### カスタマイズ可能な設定

- **`prompt`**: Claude へのカスタム指示（省略時はコメント内容を使用）
- **`claude_args`**: Claude の動作を制御する追加引数（例: 利用可能ツールの制限）
- **`additional_permissions`**: 追加パーミッション（例: CI結果の読み取り）

### 使用例

Issue やPRのコメントで以下のように記述すると Claude が動作します:

```
@claude このコードをレビューしてください
@claude バグを修正してください
@claude テストを追加してください
```

---

## 2. Claude Code Review (`claude-code-review.yml`)

### 概要

プルリクエストが作成・更新されたときに自動的に Claude によるコードレビューを実行するワークフローです。

### トリガー条件

| イベント | 種別 |
|---|---|
| `pull_request` | `opened`（新規作成） |
| `pull_request` | `synchronize`（コミット追加） |
| `pull_request` | `ready_for_review`（ドラフト解除） |
| `pull_request` | `reopened`（再オープン） |

### 必要なシークレット

| シークレット名 | 説明 |
|---|---|
| `CLAUDE_CODE_OAUTH_TOKEN` | Claude Code の OAuth トークン |

### パーミッション

```yaml
permissions:
  contents: read
  pull-requests: read
  issues: read
  id-token: write
```

### 主な動作

1. リポジトリをチェックアウト
2. `anthropics/claude-code-action@v1` を使用して Claude を起動
3. `/code-review` プラグインを使用してPRの差分を自動レビュー

### カスタマイズ可能な設定

- **特定ファイルのみ対象にする**: `paths` フィルタを使用してレビュー対象ファイルを絞り込める

  ```yaml
  paths:
    - "src/**/*.ts"
    - "src/**/*.tsx"
  ```

- **特定のPR作成者のみ対象にする**: `if` 条件でフィルタリングできる

  ```yaml
  if: |
    github.event.pull_request.user.login == 'external-contributor' ||
    github.event.pull_request.author_association == 'FIRST_TIME_CONTRIBUTOR'
  ```

---

## セットアップ

両ワークフローを動作させるには、リポジトリの **Settings > Secrets and variables > Actions** に以下のシークレットを登録する必要があります:

| シークレット名 | 取得方法 |
|---|---|
| `CLAUDE_CODE_OAUTH_TOKEN` | [Claude Code](https://claude.ai/code) にログインし、OAuth トークンを発行 |

## 参考リンク

- [claude-code-action リポジトリ](https://github.com/anthropics/claude-code-action)
- [使用方法ドキュメント](https://github.com/anthropics/claude-code-action/blob/main/docs/usage.md)
- [FAQ](https://github.com/anthropics/claude-code-action/blob/main/docs/faq.md)
