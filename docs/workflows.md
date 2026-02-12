# GitHub Actions ワークフロー ドキュメント

このリポジトリには、Claude Code を活用した2つの GitHub Actions ワークフローが含まれています。

---

## 1. Claude Code (`claude.yml`)

### 概要

Issue やプルリクエストのコメントで `@claude` とメンションすることで、Claude が自動的にタスクを実行するワークフローです。

### トリガー条件

以下のイベントが発生し、かつ本文またはタイトルに `@claude` が含まれる場合に起動します。

| イベント | タイプ |
|---|---|
| `issue_comment` | `created` |
| `pull_request_review_comment` | `created` |
| `pull_request_review` | `submitted` |
| `issues` | `opened`, `assigned` |

### 必要な権限

```yaml
permissions:
  contents: read
  pull-requests: read
  issues: read
  id-token: write
  actions: read
```

### 必要なシークレット

| シークレット名 | 説明 |
|---|---|
| `CLAUDE_CODE_OAUTH_TOKEN` | Claude Code の認証トークン |

### 動作の仕組み

1. リポジトリをチェックアウト（`actions/checkout@v4`）
2. `anthropics/claude-code-action@v1` を実行し、`@claude` へのメンションに応じた処理を行う

### カスタマイズ

- **`prompt`**: カスタムプロンプトを指定することで、コメント内容に関係なく固定のタスクを実行させることができます
- **`claude_args`**: 利用可能なツールの制限など、Claude の動作を細かく設定できます（例: `--allowed-tools Bash(gh pr:*)`）
- **`additional_permissions`**: CI 結果の読み取りなど、追加の権限を付与できます

---

## 2. Claude Code Review (`claude-code-review.yml`)

### 概要

プルリクエストが作成・更新された際に、Claude が自動でコードレビューを行うワークフローです。

### トリガー条件

プルリクエストに対して以下のアクションが発生した場合に起動します。

| イベント | タイプ |
|---|---|
| `pull_request` | `opened`, `synchronize`, `ready_for_review`, `reopened` |

特定ファイルパスの変更のみを対象にする設定（`paths` フィルター）もコメントアウトで用意されています。

### 必要な権限

```yaml
permissions:
  contents: read
  pull-requests: read
  issues: read
  id-token: write
```

### 必要なシークレット

| シークレット名 | 説明 |
|---|---|
| `CLAUDE_CODE_OAUTH_TOKEN` | Claude Code の認証トークン |

### 動作の仕組み

1. リポジトリをチェックアウト（`actions/checkout@v4`）
2. `anthropics/claude-code-action@v1` を実行し、`code-review` プラグインを使ってコードレビューを行う
3. レビュー結果はプルリクエストのコメントとして投稿される

### カスタマイズ

- **PR 作成者フィルター**: `if` 条件を使って特定のユーザー（例: 外部コントリビューター、新しい開発者）のPRのみレビューするよう設定できます
- **パスフィルター**: `paths` を設定することで、特定のファイルが変更された場合のみワークフローを起動できます

---

## セットアップ手順

1. [Claude Code](https://claude.ai/code) にサインインし、OAuth トークンを取得します
2. リポジトリの **Settings > Secrets and variables > Actions** で `CLAUDE_CODE_OAUTH_TOKEN` シークレットを登録します
3. 以上でセットアップ完了です。Issue やプルリクエストで `@claude` とメンションするか、プルリクエストを作成するとワークフローが動作します

## 参考リンク

- [claude-code-action ドキュメント](https://github.com/anthropics/claude-code-action/blob/main/docs/usage.md)
- [Claude Code CLI リファレンス](https://code.claude.com/docs/en/cli-reference)
