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
- [heroku_deploy_notify](#8-heroku_deploy_notify) - Herokuデプロイ完了/失敗をPRに通知
- [claude-dependabot-review](#9-claude-dependabot-review) - Dependabot更新PRをCI緑でも危うい観点でClaudeレビュー
- [claude-review-cli](#10-claude-review-cli) - @aki77/claude-code-review CLIによるClaude Code自動コードレビュー
- [pr-review-guide](#11-pr-review-guide) - PRのレビューガイドHTMLを生成しPR本文にリンクをupsert

## 利用可能なワークフロー

### 1. copytuner_deploy

<details>
<summary>Copytune翻訳管理ツールへの翻訳情報をデプロイするワークフローです。</summary>

**使用方法:**
```yaml
uses: SonicGarden/workflows/.github/workflows/copytuner_deploy.yml@main
secrets:
  apiKey: ${{ secrets.COPYTUNER_API_KEY }}
```

**必要なシークレット:**
- `apiKey`: CopytunerのAPIキー

</details>

### 2. gem_changelog

<details>
<summary>Gemfile.lockの変更を検出し、更新されたgemのchangelogをPRコメントに投稿するワークフローです。</summary>

**使用方法:**
```yaml
uses: SonicGarden/workflows/.github/workflows/gem_changelog.yml@main
secrets:
  rubygemsToken: ${{ secrets.RUBYGEMS_TOKEN }} # オプション
```

**オプションのシークレット:**
- `rubygemsToken`: RubyGems.orgのAPIトークン（スコープ: `Index rubygems`）

</details>

### 3. new_gems

<details>
<summary>Gemfileの変更を検出し、新しく追加されたgemをPRコメントに投稿するワークフローです。</summary>

**使用方法:**
```yaml
uses: SonicGarden/workflows/.github/workflows/new_gems.yml@main
```

</details>

### 4. new_npm

<details>
<summary>package.jsonの変更を検出し、新しく追加されたnpmパッケージをPRコメントに投稿するワークフローです。</summary>

**使用方法:**
```yaml
uses: SonicGarden/workflows/.github/workflows/new_npm.yml@main
with:
  runs_on: "ubuntu-slim" # オプション、デフォルトは "ubuntu-slim"
```

**パラメータ:**
- `runs_on`: GitHub Actions runner の指定（デフォルト: `ubuntu-slim`）

</details>

### 5. npm_changelog

<details>
<summary>npm/yarnのロックファイルの変更を検出し、更新されたパッケージのchangelogをPRコメントに投稿するワークフローです。</summary>

**使用方法:**
```yaml
uses: SonicGarden/workflows/.github/workflows/npm_changelog.yml@main
with:
  lockPath: "yarn.lock" # オプション、デフォルトは "yarn.lock"
  runs_on: "ubuntu-slim" # オプション、デフォルトは "ubuntu-slim"
```

**パラメータ:**
- `lockPath`: ロックファイルのパス（デフォルト: `yarn.lock`）
- `runs_on`: GitHub Actions runner の指定（デフォルト: `ubuntu-slim`）

</details>

### 6. staging_deploy

<details>
<summary>指定されたブランチから staging ブランチへの自動デプロイPRを作成・管理するワークフローです。</summary>

**使用方法:**
```yaml
uses: SonicGarden/workflows/.github/workflows/staging_deploy.yml@main
with:
  source_branch: "release-candidate" # オプション、デフォルトは "release-candidate"
  target_branch: "staging" # オプション、デフォルトは "staging"
  runs_on: "ubuntu-slim" # オプション、デフォルトは "ubuntu-slim"
  enable_auto_merge: true # オプション、デフォルトは true
```

**パラメータ:**
- `source_branch`: デプロイ元のブランチ名（デフォルト: `release-candidate`）
- `target_branch`: デプロイ先のブランチ名（デフォルト: `staging`）
- `runs_on`: GitHub Actions runner の指定（デフォルト: `ubuntu-slim`）
- `enable_auto_merge`: CI通過後にPRを自動マージするか（デフォルト: `true`）。`false` にするとPRの作成・更新のみ行い、自動マージは設定しない

**機能:**
- 既存のPRがあるかをチェック
- 新しいPRの作成または既存PRの更新
- 自動マージの設定（`enable_auto_merge: true` の場合のみ）
- `[skip deploy]` がコミットメッセージに含まれている場合はスキップ

**GitHub上での設定（CIが通ってから自動マージするため）:**

> **Note:** 以下の設定は `enable_auto_merge: true`（デフォルト）で自動マージを利用する場合に必要です。`false` の場合は不要です。

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

</details>

### 7. claude-review

<details>
<summary>Claude Code を使用してPRの自動コードレビューを実行するワークフローです。</summary>

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

</details>

### 8. heroku_deploy_notify

<details>
<summary>Heroku-GitHub 自動デプロイ対象ブランチへの push を契機に、Heroku Platform API をポーリングしてリリースの完了/失敗を検知し、そのマージコミットに紐づく PR にデプロイ結果をコメントするワークフローです。デプロイ本体には介入しない外形監視です。</summary>

**使用方法:**

呼び出し側でブランチごとに job を分け、`heroku_app_name` / `env_name` を渡します。`push` トリガーと `concurrency` は呼び出し側で定義します。

```yaml
name: Deploy Notify
on:
  push:
    branches: [staging, main]
concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true
jobs:
  notify-production:
    if: github.ref == 'refs/heads/main'
    uses: SonicGarden/workflows/.github/workflows/heroku_deploy_notify.yml@main
    with:
      heroku_app_name: my-app
      env_name: production
    secrets:
      heroku_api_key: ${{ secrets.HEROKU_API_KEY }}
  notify-staging:
    if: github.ref == 'refs/heads/staging'
    uses: SonicGarden/workflows/.github/workflows/heroku_deploy_notify.yml@main
    with:
      heroku_app_name: my-app-staging
      env_name: staging
    secrets:
      heroku_api_key: ${{ secrets.HEROKU_API_KEY }}
```

**パラメータ:**
- `heroku_app_name`: Heroku アプリ名（必須）
- `env_name`: PR コメントに表示する環境名（必須、例: `production` / `staging`）
- `poll_interval_seconds`: ポーリング間隔（秒）（デフォルト: `15`）
- `poll_max_attempts`: 最大試行回数（デフォルト: `48`。`poll_interval_seconds × この値`が監視時間。ランナーの15分上限内に収めること）
- `release_fetch_count`: 取得する最新 release 件数（デフォルト: `20`）
- `notify_on_success`: 成功通知の ON/OFF（デフォルト: `true`）
- `notify_on_failure`: 失敗通知の ON/OFF（デフォルト: `true`）
- `notify_on_timeout`: release を検知できずタイムアウトした場合の通知 ON/OFF（デフォルト: `true`）
- `include_release_log`: PR コメントに Heroku release phase ログ（折りたたみ）を添付する ON/OFF（デフォルト: `true`）

**必要なシークレット:**
- `heroku_api_key`: read-protected スコープの Heroku API トークン（read スコープだと releases の取得が 403 になります）

**前提:**
- 対象リポジトリで Heroku-GitHub の自動デプロイが有効であること
- Heroku release の `description` が `Deploy <短縮SHA>` 形式であること（コミット SHA 先頭7文字で照合します）

**機能:**
- Heroku release を最大 `poll_max_attempts` 回ポーリングして完了/失敗を検知
- マージコミットに紐づく PR にデプロイ結果（成功/失敗/タイムアウト）をコメント
- PR コメントは `push` イベント時のみ（手動実行などでは誤爆防止でスキップ）

</details>

### 9. claude-dependabot-review

<details>
<summary>Dependabot の依存更新 PR について、CI（テスト）が緑でも見逃しがちな「ランタイム挙動の変化・deprecation」を CHANGELOG ベースで Claude に評価させ、PR へ要約コメントを 1 本残すワークフローです。検出/分類は Dependabot に、「壊れたか」は CI に任せ、本WFは「緑でも危ういもの」を拾う役に徹します。言語・フレームワークには依存しません。</summary>

**使用方法:**

再利用可能ワークフローでは `workflow_run` トリガーを定義できないため、トリガー（`on: workflow_run`）と発火条件（CI 成功 & Dependabot 起点）は呼び出し側で定義し、`head_sha` を渡します。

```yaml
name: Claude Dependabot Review
on:
  workflow_run:
    workflows: ["Test"]   # CI(テスト本体)のワークフロー名に合わせる
    types: [completed]
concurrency:
  # 同一 head_branch の再実行で多重コメントを防ぐ
  group: ${{ github.workflow }}-${{ github.event.workflow_run.head_branch }}
  cancel-in-progress: true
jobs:
  dependabot-review:
    # Dependabot 起点の PR、かつ CI が成功した場合のみ
    if: >
      github.event.workflow_run.event == 'pull_request' &&
      github.event.workflow_run.conclusion == 'success' &&
      github.event.workflow_run.actor.login == 'dependabot[bot]'
    uses: SonicGarden/workflows/.github/workflows/claude-dependabot-review.yml@main
    with:
      head_sha: ${{ github.event.workflow_run.head_sha }}
    secrets:
      # 以下どちらかが必須
      claude_oauth_token: ${{ secrets.CLAUDE_OAUTH_TOKEN }}
      anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
```

**パラメータ:**
- `head_sha`: レビュー対象 PR の HEAD コミット SHA（必須。呼び出し側で `github.event.workflow_run.head_sha` を渡す）
- `model`: 使用する Claude モデル（オプション、デフォルト: `sonnet`。空文字を明示指定するとアクション側のデフォルトになる）
- `runs_on`: GitHub Actions runner の指定（デフォルト: `ubuntu-24.04-arm`）

**必要なシークレット:**
- `claude_oauth_token`: Claude OAuth トークン（推奨）
- `anthropic_api_key`: Anthropic API キー（claude_oauth_tokenがない場合のフォールバック）

**前提:**
- 呼び出し側の `workflows: ["Test"]` を、待ち受けたい CI ワークフローの `name` に合わせること
- Dependabot が PR 本文に Release notes / Changelog を展開していること（一次ソースとして読む）

**機能:**
- CI（テスト）が緑でも見逃す「ランタイム挙動の変化・deprecation」を CHANGELOG ベースで評価
- ベースブランチ（更新適用前の現状コード）のみを checkout し、信頼コンテキストに untrusted な PR HEAD を置かない
- PR へのコメントは 1 本に集約（既存の Claude コメントを自動削除して重複を防止）
- `head_sha` に紐づく open PR が見つからない場合はスキップ

</details>

### 10. claude-review-cli

<details>
<summary>`@aki77/claude-code-review` CLI を `npx` で実行してPRの自動コードレビューを行うワークフローです。GitHub MCPツールを直接呼び出す `claude-review` とは異なり、CLI が gh CLI 経由で diff を取得しレビューする方式です。`/review` PRコメントによる再実行、`reviewed-by-claude` ラベルでの二重レビュー防止を備えています。</summary>

**使用方法:**

再利用可能ワークフローでは `pull_request` / `issue_comment` トリガーを定義できないため、呼び出し側でトリガーと `concurrency` を定義します。

```yaml
name: Claude Code Review
on:
  pull_request:
    types: [opened, synchronize, ready_for_review]
  issue_comment:
    types: [created]

concurrency:
  # NOTE: PR更新時のレビューと/reviewコメントは同じgroupで同時実行を防ぐ (例: workflow--pr-123)
  # NOTE: 単なるコメントでもアクション自体は発動してしまうので、/review以外のコメントは個別groupで無関係なキャンセルを防ぐ (例: workflow--comment-456)
  group: |
    ${{
      github.event_name == 'pull_request' && format('{0}--pr-{1}', github.workflow, github.event.pull_request.number) ||
      contains(github.event.comment.body, '/review') && format('{0}--pr-{1}', github.workflow, github.event.issue.number) ||
      format('{0}--comment-{1}', github.workflow, github.event.comment.id)
    }}
  cancel-in-progress: true

jobs:
  claude-review:
    uses: SonicGarden/workflows/.github/workflows/claude-review-cli.yml@main
    secrets:
      # 以下どちらかが必須
      claude_oauth_token: ${{ secrets.CLAUDE_OAUTH_TOKEN }}
      anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
```

**パラメータ:**
- `runs_on`: GitHub Actions runner の指定（デフォルト: `ubuntu-24.04-arm`）

**必要なシークレット:**
- `claude_oauth_token`: Claude OAuth トークン（推奨）
- `anthropic_api_key`: Anthropic API キー（claude_oauth_tokenがない場合のフォールバック）

**機能:**
- `@aki77/claude-code-review` CLIによる自動レビュー（インラインコメント）
- PRコメントで `/review` と投稿すると再レビューを実行
- `reviewed-by-claude` ラベルを自動作成・付与し、二重レビューを防止
- 既存のClaude botコメントを自動削除して重複を防止
- fork PR対策としてベースブランチを明示的にfetch
- ドラフトPR、`[skip review]` を含むタイトル、Dependabot起点のPRは自動的にスキップ（Dependabot更新PRは `claude-dependabot-review` を利用）

**前提:**
- 呼び出し側リポジトリに `pull_request` / `issue_comment` トリガーと `concurrency` を定義すること
- リポジトリ固有の除外条件（特定ブランチ間のPRを除外するなど）が必要な場合は、呼び出し側のジョブに `if` 条件を追加すること

</details>

### 11. pr-review-guide

<details>
<summary>PR ごとに「初見のレビュアー向け解説ページ（自己完結 HTML）」を Claude に生成させ、artifact の URL を PR 本文へ marker 方式で upsert するワークフローです。Claude を読み取り専用トークンで動かす `generate` ジョブと、書き込みを行う `publish` ジョブに分離しています。`/guide` PRコメントによる再生成、`review-guide-done` ラベルでの二重生成防止を備えています。</summary>

**使用方法:**

再利用可能ワークフローでは `pull_request` / `issue_comment` トリガーを定義できないため、呼び出し側でトリガーと `concurrency` を定義します。

```yaml
name: PR Review Guide
on:
  pull_request:
    # WARNING: types に edited を入れてはいけない。本ワークフロー自身が PR 本文を編集する
    #          ため無限ループになる。labeled も同様(publish がラベルを付ける)。
    types: [opened, synchronize, ready_for_review]
  issue_comment:
    types: [created]

concurrency:
  # NOTE: PR更新時の生成と/guideコメントは同じgroupで同時実行を防ぐ (例: PR Review Guide--pr-123)
  # NOTE: 単なるコメントでもワークフロー自体は発動してしまうので、/guide以外のコメントは個別groupで無関係なキャンセルを防ぐ (例: PR Review Guide--comment-456)
  group: |
    ${{
      github.event_name == 'pull_request' && format('{0}--pr-{1}', github.workflow, github.event.pull_request.number) ||
      startsWith(github.event.comment.body, '/guide') && format('{0}--pr-{1}', github.workflow, github.event.issue.number) ||
      format('{0}--comment-{1}', github.workflow, github.event.comment.id)
    }}
  cancel-in-progress: true

jobs:
  pr-review-guide:
    # WARNING: この permissions ブロックは必須。再利用可能ワークフローの job は呼び出し側の
    #          GITHUB_TOKEN を縮小しかできず拡大できないため、ここで write を渡さないと
    #          publish ジョブが `gh label create` / `gh pr edit` で 403 になる。
    #          Claude を動かす generate ジョブは自身の permissions で read のみへ絞り込むので、
    #          ここで write を渡しても Claude に書き込み権限は渡らない。
    permissions:
      contents: read
      pull-requests: write
      issues: write
    uses: SonicGarden/workflows/.github/workflows/pr-review-guide.yml@main
    secrets:
      # 以下どちらかが必須
      claude_oauth_token: ${{ secrets.CLAUDE_OAUTH_TOKEN }}
      anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
```

**パラメータ:**
- `retention_days`: 生成した HTML artifact の保持日数（デフォルト: `7`）。PR 本文に表示する保持期間の文言も同じ値から生成されます
- `runs_on`: GitHub Actions runner の指定（デフォルト: `ubuntu-24.04-arm`）

**必要なシークレット:**
- `claude_oauth_token`: Claude OAuth トークン（推奨）
- `anthropic_api_key`: Anthropic API キー（claude_oauth_tokenがない場合のフォールバック）

**機能:**
- `aki77/skills` の `pr-review-guide` スキルで、初見のレビュアー向け解説ページ（自己完結 HTML）を生成
- 生成した HTML を zip 化せず単一ファイルの artifact としてアップロードし、ブラウザでそのまま表示可能
- artifact の URL を PR 本文へ marker 方式で upsert（作者の文章はバイト単位で保持し、再実行しても重複しない）
- Claude を動かす `generate` ジョブは読み取り専用トークンで実行し、書き込みは Claude が動かない `publish` ジョブに隔離
- PRコメントで `/guide` と投稿すると再生成を実行（OWNER / MEMBER / COLLABORATOR のみ）
- `review-guide-done` ラベルを自動作成・付与し、二重生成を防止
- 生成に失敗した場合はラベルを付けずに終了し、次の push で自動的に再挑戦
- ドラフトPR、`[skip guide]` を含むタイトル、Dependabot起点のPRは自動的にスキップ
- fork PR では secrets が渡らないため、ジョブを緑のまま静かにスキップ

**前提:**
- 呼び出し側リポジトリに `pull_request` / `issue_comment` トリガーと `concurrency` を定義すること
- 呼び出し側ジョブに `permissions:`（`contents: read` / `pull-requests: write` / `issues: write`）を定義すること。再利用可能ワークフローは呼び出し側の権限を縮小しかできないため、これが無いと `publish` が 403 になる
- トリガーの `types` に `edited` / `labeled` を含めないこと。本ワークフロー自身が PR 本文とラベルを更新するため無限ループになる
- リポジトリ固有の除外条件（特定ブランチ間のPRを除外するなど）が必要な場合は、呼び出し側のジョブに `if` 条件を追加すること
- runner に `gh` 2.92 以上が必要（スキル導入に使う `gh skill install` が preview 機能。GitHub ホストランナーは条件を満たす）

</details>
