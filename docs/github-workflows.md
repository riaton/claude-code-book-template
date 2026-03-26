# GitHub Workflows ドキュメント

このドキュメントでは、`.github/workflows` 配下に設定されているGitHub Actionsワークフローについて説明します。

## ワークフロー一覧

| ファイル名 | ワークフロー名 | 概要 |
|---|---|---|
| `claude.yml` | Claude Code | イシュー・PRコメントでの`@claude`メンションに対応 |
| `claude-code-review.yml` | Claude Code Review | PRの自動コードレビューを実行 |

---

## claude.yml

### 概要

イシューやプルリクエストのコメントで `@claude` とメンションすることで、ClaudeにタスクをAIアシスタントとして実行させるワークフローです。

### トリガー条件

以下のイベントが発生したときにワークフローが起動します。

| イベント | 条件 |
|---|---|
| `issue_comment` (created) | コメント本文に `@claude` が含まれる場合 |
| `pull_request_review_comment` (created) | コメント本文に `@claude` が含まれる場合 |
| `pull_request_review` (submitted) | レビュー本文に `@claude` が含まれる場合 |
| `issues` (opened, assigned) | イシュー本文またはタイトルに `@claude` が含まれる場合 |

### 権限

```yaml
permissions:
  contents: read
  pull-requests: read
  issues: read
  id-token: write
  actions: read
```

### 使用アクション

- `actions/checkout@v4`: リポジトリのチェックアウト
- `anthropics/claude-code-action@v1`: Claude Codeの実行

### 必要なシークレット

| シークレット名 | 説明 |
|---|---|
| `CLAUDE_CODE_OAUTH_TOKEN` | Claude CodeのOAuthトークン |

### 使い方

イシューまたはPRのコメントに `@claude` と記述し、実行させたいタスクを続けて記述します。

```
@claude このPRのテストを書いてください
```

### カスタマイズ

`claude.yml` には以下のオプション設定があります（コメントアウトされています）。

- **`prompt`**: Claudeに渡すカスタムプロンプトを固定する場合に指定します。未指定の場合、コメント内容がそのままプロンプトとして使用されます。
- **`claude_args`**: Claude CLIの追加オプション（例: 許可するツールの制限など）を指定します。

---

## claude-code-review.yml

### 概要

プルリクエストが作成・更新されたときに、Claudeが自動でコードレビューを実施するワークフローです。

### トリガー条件

以下のプルリクエストイベントが発生したときにワークフローが起動します。

| イベント | 説明 |
|---|---|
| `opened` | PRが新規作成されたとき |
| `synchronize` | PRに新しいコミットがプッシュされたとき |
| `ready_for_review` | ドラフトPRがレビュー可能状態になったとき |
| `reopened` | クローズされたPRが再オープンされたとき |

### 権限

```yaml
permissions:
  contents: read
  pull-requests: read
  issues: read
  id-token: write
```

### 使用アクション

- `actions/checkout@v4`: リポジトリのチェックアウト
- `anthropics/claude-code-action@v1`: Claude Code Reviewの実行

### 必要なシークレット

| シークレット名 | 説明 |
|---|---|
| `CLAUDE_CODE_OAUTH_TOKEN` | Claude CodeのOAuthトークン |

### カスタマイズ

`claude-code-review.yml` には以下のオプション設定があります（コメントアウトされています）。

- **`if` 条件**: PRの作成者でフィルタリングすることができます。例えば、外部コントリビューターや初めてのコントリビューターのPRのみレビューするよう制限できます。
- **`paths`**: 特定のファイル変更があった場合のみワークフローを実行するよう制限できます。

---

## セットアップ手順

1. Claude CodeのOAuthトークンを取得します（[Claude Code](https://claude.ai/code) にてログイン）。
2. GitHubリポジトリの **Settings > Secrets and variables > Actions** にて `CLAUDE_CODE_OAUTH_TOKEN` を登録します。
3. `.github/workflows/claude.yml` および `.github/workflows/claude-code-review.yml` をリポジトリに追加します。

## 参考リンク

- [claude-code-action リポジトリ](https://github.com/anthropics/claude-code-action)
- [Claude Code Action 使用方法](https://github.com/anthropics/claude-code-action/blob/main/docs/usage.md)
- [Claude Code CLI リファレンス](https://code.claude.com/docs/en/cli-reference)
