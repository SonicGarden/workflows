# workflows
Reusable GitHub Actions workflows

## ワークフロー一覧

- [copytuner_deploy](#1-copytuner_deploy) - Copytuner翻訳管理ツールへのデプロイ
- [gem_changelog](#2-gem_changelog) - Gemfile.lockの変更検出とchangelog投稿
- [new_gems](#3-new_gems) - 新規追加gemの検出とコメント投稿
- [new_npm](#4-new_npm) - 新規追加npmパッケージの検出とコメント投稿
- [npm_changelog](#5-npm_changelog) - npm/yarnロックファイルの変更検出とchangelog投稿
- [staging_deploy](#6-staging_deploy) - staging環境への自動デプロイPR作成
- [claude-review](#7-claude-review) - Claude Codeによる自動コードレビュー

## 利用可能なワークフロー

### 1. copytuner_deploy
Copytune翻訳管理ツールへの翻訳情報をデプロイするワークフローです。

**使用方法:**
```yaml
uses: SonicGarden/workflows/.github/workflows/copytuner_deploy.yml@main
secrets:
  apiKey: ${{ secrets.COPYTUNER_API_KEY }}
```

**必要なシークレット:**
- `apiKey`: CopytunerのAPIキー

### 2. gem_changelog
Gemfile.lockの変更を検出し、更新されたgemのchangelogをPRコメントに投稿するワークフローです。

**使用方法:**
```yaml
uses: SonicGarden/workflows/.github/workflows/gem_changelog.yml@main
secrets:
  rubygemsToken: ${{ secrets.RUBYGEMS_TOKEN }} # オプション
```

**オプションのシークレット:**
- `rubygemsToken`: RubyGems.orgのAPIトークン（スコープ: `Index rubygems`）

### 3. new_gems
Gemfileの変更を検出し、新しく追加されたgemをPRコメントに投稿するワークフローです。

**使用方法:**
```yaml
uses: SonicGarden/workflows/.github/workflows/new_gems.yml@main
```

### 4. new_npm
package.jsonの変更を検出し、新しく追加されたnpmパッケージをPRコメントに投稿するワークフローです。

**使用方法:**
```yaml
uses: SonicGarden/workflows/.github/workflows/new_npm.yml@main
with:
  runs_on: "ubuntu-24.04-arm" # オプション、デフォルトは "ubuntu-24.04-arm"
```

**パラメータ:**
- `runs_on`: GitHub Actions runner の指定（デフォルト: `ubuntu-24.04-arm`）

### 5. npm_changelog
npm/yarnのロックファイルの変更を検出し、更新されたパッケージのchangelogをPRコメントに投稿するワークフローです。

**使用方法:**
```yaml
uses: SonicGarden/workflows/.github/workflows/npm_changelog.yml@main
with:
  lockPath: "yarn.lock" # オプション、デフォルトは "yarn.lock"
  runs_on: "ubuntu-24.04-arm" # オプション、デフォルトは "ubuntu-24.04-arm"
```

**パラメータ:**
- `lockPath`: ロックファイルのパス（デフォルト: `yarn.lock`）
- `runs_on`: GitHub Actions runner の指定（デフォルト: `ubuntu-24.04-arm`）

### 6. staging_deploy
指定されたブランチから staging ブランチへの自動デプロイPRを作成・管理するワークフローです。

**使用方法:**
```yaml
uses: SonicGarden/workflows/.github/workflows/staging_deploy.yml@main
with:
  source_branch: "release-candidate" # オプション、デフォルトは "release-candidate"
  target_branch: "staging" # オプション、デフォルトは "staging"
  runs_on: "ubuntu-24.04-arm" # オプション、デフォルトは "ubuntu-24.04-arm"
```

**パラメータ:**
- `source_branch`: デプロイ元のブランチ名（デフォルト: `release-candidate`）
- `target_branch`: デプロイ先のブランチ名（デフォルト: `staging`）
- `runs_on`: GitHub Actions runner の指定（デフォルト: `ubuntu-24.04-arm`）

**機能:**
- 既存のPRがあるかをチェック
- 新しいPRの作成または既存PRの更新
- 自動マージの設定
- `[skip deploy]` がコミットメッセージに含まれている場合はスキップ

**GitHub上での設定（CIが通ってから自動マージするため）:**

このワークフローでPRが自動的にマージされるようにするには、リポジトリで以下の設定が必要です：

1. **ブランチ保護ルールの設定:**
   - リポジトリの Settings > Branches に移動
   - staging ブランチのブランチ保護ルールを追加
   - 「Require status checks to pass before merging」をチェック
   - 必要なCIチェック（例：テスト、linter など）を選択

2. **自動マージ機能の有効化:**
   - リポジトリの Settings > General に移動
   - 「Allow auto-merge」をチェック

これらの設定により、CIが全て通過した後に自動的にPRがマージされます。

### 7. claude-review
Claude Code を使用してPRの自動コードレビューを実行するワークフローです。

**使用方法:**
```yaml
uses: SonicGarden/workflows/.github/workflows/claude-review.yml@main
with:
  guideline_files: |
    doc/agent/code-review.md
    doc/agent/code-review-rails.md
  minimum_changed_lines: 10
secrets:
  # 以下どちらかが必須
  claude_oauth_token: ${{ secrets.CLAUDE_OAUTH_TOKEN }}
  anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
```

**パラメータ:**
- `guideline_files`: コードレビューのガイドラインファイルのパス一覧（改行区切り）
- `model`: 使用するClaudeモデル（オプション、デフォルトは `sonnet`。空文字を明示指定するとアクション側のデフォルトになる）
- `minimum_changed_lines`: 前回レビューからの最小変更行数（デフォルト: 0、0の場合は常に実行）
- `runs_on`: GitHub Actions runner の指定（デフォルト: `ubuntu-24.04-arm`）

**必要なシークレット:**
- `claude_oauth_token`: Claude OAuth トークン（推奨）
- `anthropic_api_key`: Anthropic API キー（claude_oauth_tokenがない場合のフォールバック）

**機能:**
- PRの変更内容を日本語で自動レビュー
- インラインコメントによる具体的な改善提案
- コード品質、セキュリティ、パフォーマンスなど多角的な観点からのレビュー
- 既存のClaude botコメントを自動削除して重複を防止
- `[skip review]` がコミットメッセージに含まれている場合はレビューをスキップ

## 使用例

他のリポジトリからこれらのワークフローを使用する場合は、`.github/workflows/` ディレクトリに以下のようなファイルを作成してください：

```yaml
name: Example Workflow
on:
  pull_request:
    paths:
      - 'Gemfile.lock'

jobs:
  gem-changelog:
    uses: SonicGarden/workflows/.github/workflows/gem_changelog.yml@main
    secrets:
      rubygemsToken: ${{ secrets.RUBYGEMS_TOKEN }}
```
