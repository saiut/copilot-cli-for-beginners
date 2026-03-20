![Chapter 06: MCP Servers](images/chapter-header.png)

> **もし Copilot がターミナルから GitHub の Issue を読んだり、データベースを確認したり、PR を作成したりできたら？**

これまで Copilot は、`@` で参照したファイル、会話の履歴、そして自身の学習データなど、直接渡された情報しか扱えませんでした。でも、もし Copilot が自分から GitHub リポジトリを確認したり、プロジェクトファイルを閲覧したり、ライブラリの最新ドキュメントを調べたりできるとしたら？

それを実現するのが MCP（Model Context Protocol）です。MCP は、Copilot を外部サービスに接続して、リアルタイムの実データにアクセスできるようにする仕組みです。Copilot が接続する各サービスは「MCP サーバー」と呼ばれます。この章では、いくつかの接続をセットアップし、Copilot がどれほど便利になるかを体験します。

> 💡 **MCP をすでにご存知ですか？** [クイックスタートに進む](#-use-the-built-in-github-mcp)で動作を確認し、サーバーの設定を始めましょう。

## 🎯 学習目標

この章を終えると、以下のことができるようになります：

- MCP とは何か、なぜ重要かを理解する
- `/mcp` コマンドを使って MCP サーバーを管理する
- GitHub、ファイルシステム、ドキュメント用の MCP サーバーを設定する
- ブックアプリプロジェクトで MCP を活用したワークフローを使う
- カスタム MCP サーバーの構築方法とタイミングを知る（オプション）

> ⏱️ **所要時間の目安**: 約50分（読む時間15分 + ハンズオン35分）

---

## 🧩 身近なたとえ：ブラウザ拡張機能

<img src="images/browser-extensions-analogy.png" alt="MCP Servers are like Browser Extensions" width="800"/>

MCP サーバーはブラウザの拡張機能のようなものです。ブラウザ単体でも Web ページは表示できますが、拡張機能を追加すると、さまざまなサービスに接続できるようになります：

| ブラウザ拡張機能 | 接続先 | MCP での対応 |
|-------------------|---------------------|----------------|
| パスワードマネージャー | パスワード保管庫 | **GitHub MCP** → リポジトリ、Issue、PR |
| Grammarly | 文章分析サービス | **Context7 MCP** → ライブラリのドキュメント |
| ファイルマネージャー | クラウドストレージ | **Filesystem MCP** → ローカルプロジェクトファイル |

拡張機能がなくてもブラウザは便利ですが、拡張機能があると格段にパワフルになります。MCP サーバーは Copilot に対して同じ役割を果たします。リアルタイムのデータソースに接続することで、GitHub の Issue を読んだり、ファイルシステムを探索したり、最新のドキュメントを取得したりできるようになります。

***MCP サーバーは Copilot を外の世界（GitHub、リポジトリ、ドキュメントなど）につなげます***

> 💡 **ポイント**: MCP がなければ、Copilot は `@` で明示的に共有したファイルしか見えません。MCP があれば、プロジェクトを積極的に探索し、GitHub リポジトリを確認し、ドキュメントを調べることが自動的にできるようになります。

---

<img src="images/quick-start-mcp.png" alt="Power cable connecting with bright electrical spark surrounded by floating tech icons representing MCP server connections" width="800"/>

# クイックスタート：30秒で MCP を体験

## 組み込みの GitHub MCP サーバーを使ってみよう
何も設定する前に、まず MCP を実際に動かしてみましょう。
GitHub MCP サーバーはデフォルトで含まれています。次のコマンドを試してください：

```bash
copilot
> List the recent commits in this repository
```

Copilot が実際のコミットデータを返してきたら、MCP の動作を確認できたことになります。これは GitHub MCP サーバーがあなたの代わりに GitHub にアクセスしている結果です。ただし、GitHub は数ある MCP サーバーの *1つ* にすぎません。この章では、ファイルシステムアクセスや最新ドキュメントなど、さらにサーバーを追加して Copilot をもっと便利にする方法を紹介します。

---

## `/mcp show` コマンド

`/mcp show` を使って、どの MCP サーバーが設定されていて、有効になっているかを確認できます：

```bash
copilot

> /mcp show

MCP Servers:
✓ github (enabled) - GitHub integration
✓ filesystem (enabled) - File system access
```

> 💡 **GitHub サーバーしか表示されない？** 正常です！追加の MCP サーバーをまだ設定していなければ、GitHub だけが表示されます。次のセクションでサーバーを追加していきます。

> 📚 **すべての `/mcp` コマンドを見たい？** サーバーの追加、編集、有効化、削除のためのコマンドがあります。この章の最後にある[コマンドリファレンス](#-additional-mcp-commands)をご覧ください。

<details>
<summary>🎬 実際に動かしてみよう！</summary>

![MCP Status Demo](images/mcp-status-demo.gif)

*デモの出力は環境によって異なります。モデル、ツール、応答はここに表示されているものと異なる場合があります。*

</details>

---

## MCP で何が変わるのか？

MCP があるとないとでは、実際にこれだけの違いがあります：

**MCP なしの場合：**
```bash
> What's in GitHub issue #42?

"I don't have access to GitHub. You'll need to copy and paste the issue content."
```

**MCP ありの場合：**
```bash
> What's in GitHub issue #42 of this repository?

Issue #42: Login fails with special characters
Status: Open
Labels: bug, priority-high
Description: Users report that passwords containing...
```

MCP によって、Copilot があなたの実際の開発環境を把握できるようになります。

> 📚 **公式ドキュメント**: [About MCP](https://docs.github.com/copilot/concepts/context/mcp) で MCP が GitHub Copilot とどのように連携するか詳しく解説されています。

---

# MCP サーバーの設定

<img src="images/configuring-mcp-servers.png" alt="Hands adjusting knobs and sliders on a professional audio mixing board representing MCP server configuration" width="800"/>

MCP の動作を確認できたので、次は追加のサーバーをセットアップしましょう。このセクションでは、設定ファイルの形式と新しいサーバーの追加方法を説明します。

---

## MCP 設定ファイル

MCP サーバーは `~/.copilot/mcp-config.json`（ユーザーレベル、すべてのプロジェクトに適用）または `.vscode/mcp.json`（プロジェクトレベル、現在のワークスペースのみに適用）で設定します。

```json
{
  "mcpServers": {
    "server-name": {
      "type": "local",
      "command": "npx",
      "args": ["@package/server-name"],
      "tools": ["*"]
    }
  }
}
```

*ほとんどの MCP サーバーは npm パッケージとして配布され、`npx` コマンドで実行します。*

<details>
<summary>💡 <strong>JSON が初めてですか？</strong> 各フィールドの意味を確認するにはここをクリック</summary>

| フィールド | 意味 |
|-------|---------------|
| `"mcpServers"` | すべての MCP サーバー設定を格納するコンテナ |
| `"server-name"` | 任意の名前（例：「github」「filesystem」） |
| `"type": "local"` | サーバーがローカルマシンで動作する |
| `"command": "npx"` | 実行するプログラム（npx は npm パッケージを実行する） |
| `"args": [...]` | コマンドに渡す引数 |
| `"tools": ["*"]` | このサーバーのすべてのツールを許可する |

**JSON の重要なルール：**
- 文字列にはダブルクォート `"` を使用する（シングルクォートは不可）
- 最後の項目の後にカンマを付けない
- ファイルは有効な JSON でなければならない（不安な場合は [JSON バリデーター](https://jsonlint.com/) を使用）

</details>

---

## MCP サーバーの追加

GitHub MCP サーバーは組み込みで、セットアップ不要です。以下は追加できるサーバーです。**興味のあるものを選ぶか、順番に進めてください。**

| やりたいこと | リンク先 |
|---|---|
| Copilot にプロジェクトファイルを閲覧させたい | [Filesystem サーバー](#filesystem-server) |
| 最新のライブラリドキュメントを取得したい | [Context7 サーバー](#context7-server-documentation) |
| オプションの追加機能を試したい（カスタムサーバー、web_fetch） | [基本を超えて](#beyond-the-basics) |

<details>
<summary><strong>Filesystem サーバー</strong> - Copilot にプロジェクトファイルを探索させる</summary>
<a id="filesystem-server"></a>

### Filesystem サーバー

```json
{
  "mcpServers": {
    "filesystem": {
      "type": "local",
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "."],
      "tools": ["*"]
    }
  }
}
```

> 💡 **`.` パスについて**: `.` は「カレントディレクトリ」を意味します。Copilot は起動した場所を基準にファイルにアクセスできます。Codespace では、これがワークスペースのルートになります。必要に応じて `/workspaces/copilot-cli-for-beginners` のような絶対パスを指定することもできます。

この設定を `~/.copilot/mcp-config.json` に追加して、Copilot を再起動してください。

</details>

<details>
<summary><strong>Context7 サーバー</strong> - 最新のライブラリドキュメントを取得する</summary>
<a id="context7-server-documentation"></a>

### Context7 サーバー（ドキュメント）

Context7 は、人気のフレームワークやライブラリの最新ドキュメントに Copilot がアクセスできるようにします。古くなっている可能性のある学習データに頼る代わりに、Copilot が実際の最新ドキュメントを取得します。

```json
{
  "mcpServers": {
    "context7": {
      "type": "local",
      "command": "npx",
      "args": ["-y", "@upstash/context7-mcp"],
      "tools": ["*"]
    }
  }
}
```

- ✅ **API キー不要** 
- ✅ **アカウント不要** 
- ✅ **コードはローカルに保持**

この設定を `~/.copilot/mcp-config.json` に追加して、Copilot を再起動してください。

</details>

<details>
<summary><strong>基本を超えて</strong> - カスタムサーバーと Web アクセス（オプション）</summary>
<a id="beyond-the-basics"></a>

ここからは、上記のコアサーバーに慣れてから試すオプション機能です。

### Microsoft Learn MCP サーバー

ここまで見てきた MCP サーバー（filesystem、Context7）はすべてローカルマシンで動作していました。しかし、MCP サーバーはリモートでも動作できます。つまり、Copilot CLI に URL を指定するだけで、残りはすべて自動で処理されます。`npx` も `python` も、ローカルプロセスも、インストールする依存関係も不要です。

[Microsoft Learn MCP サーバー](https://github.com/microsoftdocs/mcp) はその好例です。Copilot CLI が公式の Microsoft ドキュメント（Azure、Microsoft Foundry やその他の AI トピック、.NET、Microsoft 365 など）に直接アクセスでき、モデルの学習データに頼る代わりに、ドキュメントの検索、ページ全体の取得、公式コードサンプルの検索ができます。

- ✅ **API キー不要** 
- ✅ **アカウント不要** 
- ✅ **ローカルインストール不要**

**`/plugin install` で簡単インストール：**

JSON 設定ファイルを手動で編集する代わりに、1つのコマンドでインストールできます：

```bash
copilot

> /plugin install microsoftdocs/mcp
```

サーバーと関連するエージェントスキルが自動的に追加されます。インストールされるスキルには以下が含まれます：

- **microsoft-docs**: コンセプト、チュートリアル、事実の検索
- **microsoft-code-reference**: API リファレンス、コードサンプル、トラブルシューティング
- **microsoft-skill-creator**: Microsoft テクノロジーに関するカスタムスキルを生成するメタスキル

**使い方：**
```bash
copilot

> What's the recommended way to deploy a Python app to Azure App Service? Search Microsoft Learn.
```

📚 詳しくはこちら: [Microsoft Learn MCP Server overview](https://learn.microsoft.com/training/support/mcp-get-started)

### `web_fetch` による Web アクセス

Copilot CLI には、任意の URL からコンテンツを取得できる組み込みの `web_fetch` ツールがあります。ターミナルを離れずに README、API ドキュメント、リリースノートを取り込むのに便利です。MCP サーバーは不要です。

アクセス可能な URL は `~/.copilot/config.json`（Copilot の一般設定）で制御できます。これは `~/.copilot/mcp-config.json`（MCP サーバー定義）とは別のファイルです。

```json
{
  "permissions": {
    "allowedUrls": [
      "https://api.github.com/**",
      "https://docs.github.com/**",
      "https://*.npmjs.org/**"
    ],
    "blockedUrls": [
      "http://**"
    ]
  }
}
```

**使い方：**
```bash
copilot

> Fetch and summarize the README from https://github.com/facebook/react
```

### カスタム MCP サーバーの構築

Copilot を自分の API、データベース、または内部ツールに接続したいですか？ Python でカスタム MCP サーバーを構築できます。既製のサーバー（GitHub、filesystem、Context7）がほとんどのユースケースをカバーするため、これは完全にオプションです。

📖 ブックアプリを使った完全なチュートリアルは [カスタム MCP サーバーガイド](mcp-custom-server.md) をご覧ください。

📚 詳しい背景情報は [MCP for Beginners コース](https://github.com/microsoft/mcp-for-beginners) をご覧ください。

</details>

<a id="complete-configuration-file"></a>

### 完全な設定ファイル

filesystem と Context7 サーバーを含む完全な `mcp-config.json` は以下の通りです：

> 💡 **注意:** GitHub MCP は組み込みです。設定ファイルに追加する必要はありません。

```json
{
  "mcpServers": {
    "filesystem": {
      "type": "local",
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "."],
      "tools": ["*"]
    },
    "context7": {
      "type": "local",
      "command": "npx",
      "args": ["-y", "@upstash/context7-mcp"],
      "tools": ["*"]
    }
  }
}
```

`~/.copilot/mcp-config.json` に保存するとグローバルに適用、`.vscode/mcp.json` に保存するとプロジェクト固有の設定になります。

---

# MCP サーバーを使う

MCP サーバーの設定ができたので、何ができるか見てみましょう。

<img src="images/using-mcp-servers.png" alt="Using MCP Servers - Hub-and-spoke diagram showing a Developer CLI connected to GitHub, Filesystem, Context7, and Custom/Web Fetch servers" width="800" />

---

## サーバーの使用例

**試したいサーバーを選ぶか、順番に進めてください。**

| 試したいこと | リンク先 |
|---|---|
| GitHub のリポジトリ、Issue、PR | [GitHub サーバー](#github-server-built-in) |
| プロジェクトファイルの閲覧 | [Filesystem サーバーの使い方](#filesystem-server-usage) |
| ライブラリドキュメントの検索 | [Context7 サーバーの使い方](#context7-server-usage) |
| カスタムサーバー、Microsoft Learn MCP、web_fetch の使い方 | [基本を超えた使い方](#beyond-the-basics-usage) |

<details>
<summary><strong>GitHub サーバー（組み込み）</strong> - リポジトリ、Issue、PR などにアクセス</summary>
<a id="github-server-built-in"></a>

### GitHub サーバー（組み込み）

GitHub MCP サーバーは**組み込み**です。Copilot にログイン済み（初期セットアップで実施済み）であれば、すでに動作しています。設定は不要です！

> 💡 **動作しない場合** `/login` を実行して GitHub に再認証してください。

<details>
<summary><strong>Dev Container での認証</strong></summary>

- **GitHub Codespaces**（推奨）: 認証は自動的に行われます。`gh` CLI が Codespace のトークンを引き継ぎます。操作は不要です。
- **ローカル Dev Container（Docker）**: コンテナ起動後に `gh auth login` を実行し、Copilot を再起動してください。

**認証のトラブルシューティング：**
```bash
# 認証状態の確認
gh auth status

# 未認証の場合、ログイン
gh auth login

# GitHub MCP の接続確認
copilot
> /mcp show
```

</details>

| 機能 | 例 |
|---------|----------|
| **リポジトリ情報** | コミット、ブランチ、コントリビューターの表示 |
| **Issue** | Issue の一覧表示、作成、検索、コメント |
| **プルリクエスト** | PR の表示、差分の確認、PR の作成、ステータス確認 |
| **コード検索** | リポジトリ全体のコード検索 |
| **Actions** | ワークフロー実行とステータスの照会 |

```bash
copilot

# このリポジトリの最近のアクティビティを確認
> List the last 5 commits in this repository

Recent commits:
1. abc1234 - Update chapter 05 skills examples (2 days ago)
2. def5678 - Add book app test fixtures (3 days ago)
3. ghi9012 - Fix typo in chapter 03 README (4 days ago)
...

# リポジトリの構造を探索
> What branches exist in this repository?

Branches:
- main (default)
- chapter6 (current)

# リポジトリ全体でコードパターンを検索
> Search this repository for files that import pytest

Found 1 file:
- samples/book-app-project/tests/test_books.py
```

> 💡 **自分のフォークで作業中ですか？** このコースのリポジトリをフォークした場合は、Issue やプルリクエストの作成などの書き込み操作も試せます。後の演習で実践します。

> ⚠️ **結果が表示されない場合** GitHub MCP はリモートリポジトリ（github.com 上）に対して動作し、ローカルファイルだけではありません。リポジトリにリモートが設定されていることを確認してください：`git remote -v` で確認できます。

</details>

<details>
<summary><strong>Filesystem サーバー</strong> - プロジェクトファイルの閲覧と分析</summary>
<a id="filesystem-server-usage"></a>

### Filesystem サーバー

設定が完了すると、filesystem MCP は Copilot が自動的に使えるツールを提供します：

```bash
copilot

> How many Python files are in the book-app-project directory?

Found 3 Python files in samples/book-app-project/:
- book_app.py
- books.py
- utils.py

> What's the total size of the data.json file?

samples/book-app-project/data.json: 2.4 KB

> Find all functions that don't have type hints in the book app

Found 2 functions without type hints:
- samples/book-app-project/utils.py:10 - get_user_choice()
- samples/book-app-project/utils.py:14 - get_book_details()
```

</details>

<details>
<summary><strong>Context7 サーバー</strong> - ライブラリドキュメントの検索</summary>
<a id="context7-server-usage"></a>

### Context7 サーバー

```bash
copilot

> What are the best practices for using pytest fixtures?

From pytest Documentation:

Fixtures - Use fixtures to provide a fixed baseline for tests:

    import pytest

    @pytest.fixture
    def sample_books():
        return [
            {"title": "1984", "author": "George Orwell", "year": 1949},
            {"title": "Dune", "author": "Frank Herbert", "year": 1965},
        ]

    def test_find_by_author(sample_books):
        # fixture は自動的に引数として渡される
        results = [b for b in sample_books if "Orwell" in b["author"]]
        assert len(results) == 1

Best practices:
- Use fixtures instead of setup/teardown methods
- Use tmp_path fixture for temporary files
- Use monkeypatch for modifying environment
- Scope fixtures appropriately (function, class, module, session)

> How can I apply this to the book app's test file?

# Copilot は公式の pytest パターンを把握しているので、
# samples/book-app-project/tests/test_books.py に適用できる
```

</details>

<details>
<summary><strong>基本を超えた使い方</strong> - カスタムサーバーと web_fetch の使い方</summary>
<a id="beyond-the-basics-usage"></a>

### 基本を超えた使い方

**カスタム MCP サーバー**: [カスタム MCP サーバーガイド](mcp-custom-server.md) から book-lookup サーバーを構築した場合、ブックコレクションを直接照会できます：

```bash
copilot

> Look up information about "1984" using the book lookup server. Search for books by George Orwell
```

**Microsoft Learn MCP**: [Microsoft Learn MCP サーバー](#microsoft-learn-mcp-server)をインストールした場合、公式の Microsoft ドキュメントを直接検索できます：

```bash
copilot

> How do I configure managed identity for an Azure Function? Search Microsoft Learn.
```

**Web Fetch**: 組み込みの `web_fetch` ツールを使って、任意の URL からコンテンツを取得できます：

```bash
copilot

> Fetch and summarize the README from https://github.com/facebook/react
```

</details>

---

## マルチサーバーワークフロー

これらのワークフローは、開発者が「もうこれなしでは仕事できない」と言う理由を示しています。各例では、1つのセッション内で複数の MCP サーバーを組み合わせて使います。

<img src="images/issue-to-pr-workflow.png" alt="Issue to PR Workflow using MCP - Shows the complete flow from getting a GitHub issue through creating a pull request" width="800"/>

*完全な MCP ワークフロー：GitHub MCP がリポジトリデータを取得し、Filesystem MCP がコードを見つけ、Context7 MCP がベストプラクティスを提供し、Copilot が分析を行います*

以下の各例は独立しています。**興味のあるものを選ぶか、すべて読んでください。**

| 見たいもの | リンク先 |
|---|---|
| 複数のサーバーの連携 | [マルチサーバー探索](#multi-server-exploration) |
| Issue から PR まで1つのセッションで | [Issue から PR へのワークフロー](#issue-to-pr-workflow) |
| プロジェクトのクイックヘルスチェック | [ヘルスダッシュボード](#health-dashboard) |

<details>
<summary><strong>マルチサーバー探索</strong> - filesystem、GitHub、Context7 を1つのセッションで組み合わせる</summary>
<a id="multi-server-exploration"></a>

#### 複数の MCP サーバーでブックアプリを探索する

```bash
copilot

# ステップ 1：filesystem MCP を使ってブックアプリを探索
> List all Python files in samples/book-app-project/ and summarize
> what each file does

Found 3 Python files:
- book_app.py: CLI entry point with command routing (list, add, remove, find)
- books.py: BookCollection class with data persistence via JSON
- utils.py: Helper functions for user input and display

# ステップ 2：GitHub MCP を使って最近の変更を確認
> What were the last 3 commits that touched files in samples/book-app-project/?

Recent commits affecting book app:
1. abc1234 - Add test fixtures for BookCollection (2 days ago)
2. def5678 - Add find_by_author method (5 days ago)
3. ghi9012 - Initial book app setup (1 week ago)

# ステップ 3：Context7 MCP でベストプラクティスを確認
> What are Python best practices for JSON data persistence?

From Python Documentation:
- Use context managers (with statements) for file I/O
- Handle JSONDecodeError for corrupted files
- Use dataclasses for structured data
- Consider atomic writes to prevent data corruption

# ステップ 4：推奨事項をまとめる
> Based on the book app code and these best practices,
> what improvements would you suggest?

Suggestions:
1. Add input validation in add_book() for empty strings and invalid years
2. Consider atomic writes in save_books() to prevent data corruption
3. Add type hints to utils.py functions (get_user_choice, get_book_details)
```

<details>
<summary>🎬 MCP ワークフローの実際の動作を見る！</summary>

![MCP Workflow Demo](images/mcp-workflow-demo.gif)

*デモの出力は環境によって異なります。モデル、ツール、応答はここに表示されているものと異なる場合があります。*

</details>

**結果**: コード探索 → 履歴確認 → ベストプラクティス検索 → 改善計画。**1つのターミナルセッションから、3つの MCP サーバーを組み合わせて実現。**

</details>

<details>
<summary><strong>Issue から PR へのワークフロー</strong> - ターミナルを離れずに GitHub Issue からプルリクエストまで</summary>
<a id="issue-to-pr-workflow"></a>

#### Issue から PR へのワークフロー（自分のリポジトリで）

これは書き込みアクセスのある自分のフォークやリポジトリで最もうまく動作します：

> 💡 **今すぐ試せなくても大丈夫です。** 読み取り専用のクローンの場合、この操作は課題で実践します。今はフローを理解するために読み進めてください。

```bash
copilot

> Get the details of GitHub issue #1

Issue #1: Add input validation for book year
Status: Open
Description: The add_book function accepts any year value...

> @samples/book-app-project/books.py Fix the issue described in issue #1

[Copilot が add_book() に年の入力検証を実装]

> Run the tests to make sure the fix works

All 8 tests passed ✓

> Create a pull request titled "Add year validation to book app"

✓ Created PR #2: Add year validation to book app
```

**コピペゼロ。コンテキストスイッチゼロ。1つのターミナルセッションですべて完了。**

</details>

<details>
<summary><strong>ヘルスダッシュボード</strong> - 複数のサーバーを使ったプロジェクトのクイックヘルスチェック</summary>
<a id="health-dashboard"></a>

#### ブックアプリ ヘルスダッシュボード

```bash
copilot

> Give me a health report for the book app project:
> 1. List all functions across the Python files in samples/book-app-project/
> 2. Check which functions have type hints and which don't
> 3. Show what tests exist in samples/book-app-project/tests/
> 4. Check the recent commit history for this directory

Book App Health Report
======================

📊 Functions Found:
- books.py: 8 methods in BookCollection (all have type hints ✓)
- book_app.py: 6 functions (4 have type hints, 2 missing)
- utils.py: 3 functions (1 has type hints, 2 missing)

🧪 Test Coverage:
- test_books.py: 8 test functions covering BookCollection
- Missing: no tests for book_app.py CLI functions
- Missing: no tests for utils.py helper functions

📝 Recent Activity:
- 3 commits in the last week
- Most recent: added test fixtures

Recommendations:
- Add type hints to utils.py functions
- Add tests for book_app.py CLI handlers
- All files well-sized (<100 lines) - good structure!
```

**結果**: 複数のデータソースが数秒で集約されます。手動でやると、grep の実行、行数のカウント、git log の確認、テストファイルの閲覧が必要で、15分以上はかかるでしょう。

</details>

---

# 実践練習

<img src="../images/practice.png" alt="Warm desk setup with monitor showing code, lamp, coffee cup, and headphones ready for hands-on practice" width="800"/>

**🎉 基本はマスターしました！** MCP を理解し、サーバーの設定方法を学び、実際のワークフローを見てきました。次は自分で試してみましょう。

---

## ▶️ やってみよう

それでは演習を通じて、ブックアプリプロジェクトで MCP サーバーの使い方を実践してみましょう。

### 演習 1：MCP のステータスを確認する

まず、利用可能な MCP サーバーを確認しましょう：

```bash
copilot

> /mcp show
```

GitHub サーバーが有効として表示されるはずです。表示されない場合は `/login` を実行して認証してください。

---

### 演習 2：Filesystem MCP でブックアプリを探索する

filesystem サーバーを設定済みの場合、ブックアプリを探索してみましょう：

```bash
copilot

> How many Python files are in samples/book-app-project/?
> What functions are defined in each file?
```

**期待される結果**: Copilot が `book_app.py`、`books.py`、`utils.py` とそれぞれの関数を一覧表示します。

> 💡 **filesystem MCP をまだ設定していない場合** 上の[完全な設定ファイル](#complete-configuration-file)セクションから設定ファイルを作成してください。その後 Copilot を再起動してください。

---

### 演習 3：GitHub MCP でリポジトリの履歴を照会する

組み込みの GitHub MCP を使って、このコースのリポジトリを探索しましょう：

```bash
copilot

> List the last 5 commits in this repository

> What branches exist in this repository?
```

**期待される結果**: Copilot が GitHub リモートの最近のコミットメッセージとブランチ名を表示します。

> ⚠️ **Codespace を使用中ですか？** 自動的に動作します。認証は引き継がれます。ローカルクローンの場合は、`gh auth status` でログイン状態を確認してください。

---

### 演習 4：複数の MCP サーバーを組み合わせる

filesystem と GitHub MCP を1つのセッションで組み合わせてみましょう：

```bash
copilot

> Read samples/book-app-project/data.json and tell me what books are
> in the collection. Then check the recent commits to see when this
> file was last modified.
```

**期待される結果**: Copilot が JSON ファイルを読み（filesystem MCP）、「The Hobbit」「1984」「Dune」「To Kill a Mockingbird」「Mysterious Book」の5冊を一覧表示し、GitHub でコミット履歴を照会します。

**セルフチェック**: "リポジトリのコミット履歴を確認して" が、手動で `git log` を実行してプロンプトに貼り付けるよりも優れている理由を説明できれば、MCP を理解できています。

---

## 📝 課題

### メインチャレンジ：ブックアプリ MCP 探索

MCP サーバーを組み合わせてブックアプリプロジェクトで実践しましょう。以下のステップを1つの Copilot セッションで完了してください：

1. **MCP の動作確認**: `/mcp show` を実行して、少なくとも GitHub サーバーが有効であることを確認する
2. **filesystem MCP のセットアップ**（未設定の場合）: filesystem サーバー設定を含む `~/.copilot/mcp-config.json` を作成する
3. **コードの探索**: filesystem サーバーを使って以下を Copilot に質問する：
   - `samples/book-app-project/books.py` のすべての関数を一覧表示する
   - `samples/book-app-project/utils.py` で型ヒントが欠けている関数を確認する
   - `samples/book-app-project/data.json` を読んでデータ品質の問題を特定する（ヒント：最後のエントリに注目）
4. **リポジトリのアクティビティを確認**: GitHub MCP を使って Copilot に質問する：
   - `samples/book-app-project/` のファイルに関する最近のコミットを一覧表示する
   - オープンな Issue やプルリクエストがあるか確認する
5. **サーバーを組み合わせる**: 1つのプロンプトで Copilot に以下を質問する：
   - `samples/book-app-project/tests/test_books.py` のテストファイルを読む
   - テストされている関数と `books.py` のすべての関数を比較する
   - テストカバレッジの不足を要約する

**成功基準**: filesystem と GitHub MCP のデータを1つの Copilot セッションでシームレスに組み合わせでき、各 MCP サーバーがレスポンスにどう貢献したかを説明できること。

<details>
<summary>💡 ヒント（クリックして展開）</summary>

**ステップ 1：MCP の確認**
```bash
copilot
> /mcp show
# github が有効として表示されるはず
# 未認証の場合: /login を実行
```

**ステップ 2：設定ファイルの作成**

上の[完全な設定ファイル](#complete-configuration-file)セクションの JSON を使用し、`~/.copilot/mcp-config.json` として保存してください。

**ステップ 3：データ品質の問題の確認ポイント**

`data.json` の最後の書籍は以下の通りです：
```json
{
  "title": "Mysterious Book",
  "author": "",
  "year": 0,
  "read": false
}
```
著者が空で、年が 0。これがデータ品質の問題です！

**ステップ 5：テストカバレッジの比較**

`test_books.py` のテストは `add_book`、`mark_as_read`、`remove_book`、`get_unread_books`、`find_book_by_title` をカバーしています。`load_books`、`save_books`、`list_books` などの関数には直接のテストがありません。`book_app.py` の CLI 関数と `utils.py` のヘルパー関数にもテストがまったくありません。

**MCP が動作しない場合：** 設定ファイルを編集した後に Copilot を再起動してください。

</details>

### ボーナスチャレンジ：カスタム MCP サーバーを構築する

もっと深く学びたいですか？ [カスタム MCP サーバーガイド](mcp-custom-server.md) に従って、任意の API に接続する Python の MCP サーバーを自分で構築してみましょう。

---

<details>
<summary>🔧 <strong>よくある間違いとトラブルシューティング</strong>（クリックして展開）</summary>

### よくある間違い

| 間違い | 何が起こるか | 対処法 |
|---------|--------------|-----|
| GitHub MCP が組み込みだと知らない | 手動でインストール・設定しようとする | GitHub MCP はデフォルトで含まれています。「List the recent commits in this repo」と試してみてください |
| 設定ファイルを間違った場所で探す | MCP の設定を見つけられない・編集できない | ユーザーレベルの設定は `~/.copilot/mcp-config.json`、プロジェクトレベルは `.vscode/mcp.json` |
| 設定ファイルの JSON が不正 | MCP サーバーの読み込みに失敗する | `/mcp show` で設定を確認。JSON の構文を検証してください |
| MCP サーバーの認証を忘れる | "Authentication failed" エラーが出る | 一部の MCP は個別の認証が必要です。各サーバーの要件を確認してください |

### トラブルシューティング

**「MCP server not found」** - 以下を確認してください：
1. npm パッケージが存在するか: `npm view @modelcontextprotocol/server-github`
2. 設定ファイルが有効な JSON であるか
3. サーバー名が設定と一致しているか

`/mcp show` を使って現在の設定を確認してください。

**「GitHub authentication failed」** - 組み込みの GitHub MCP は `/login` の認証情報を使用します。以下を試してください：

```bash
copilot
> /login
```

これで GitHub に再認証されます。問題が続く場合は、アクセスしようとしているリポジトリに対して GitHub アカウントに必要な権限があるか確認してください。

**「MCP server failed to start」** - サーバーログを確認してください：
```bash
# サーバーコマンドを手動で実行してエラーを確認
npx -y @modelcontextprotocol/server-github
```

**MCP ツールが利用できない** - サーバーが有効になっているか確認してください：
```bash
copilot

> /mcp show
# サーバーが一覧表示され、有効になっているか確認
```

サーバーが無効になっている場合は、下の[追加の `/mcp` コマンド](#-additional-mcp-commands)で再度有効にする方法を確認してください。

</details>

---

<details>
<summary>📚 <strong>追加の <code>/mcp</code> コマンド</strong>（クリックして展開）</summary>
<a id="-additional-mcp-commands"></a>

`/mcp show` 以外にも、MCP サーバーを管理するためのコマンドがいくつかあります：

| コマンド | 説明 |
|---------|--------------|
| `/mcp show` | 設定されたすべての MCP サーバーとそのステータスを表示 |
| `/mcp add` | 新しいサーバーを追加するインタラクティブセットアップ |
| `/mcp edit <server-name>` | 既存のサーバー設定を編集 |
| `/mcp enable <server-name>` | 無効化されたサーバーを有効にする |
| `/mcp disable <server-name>` | サーバーを一時的に無効にする |
| `/mcp delete <server-name>` | サーバーを完全に削除する |

このコースでは、ほとんどの場面で `/mcp show` だけで十分です。その他のコマンドは、管理するサーバーが増えてきたときに役立ちます。

</details>

---

# まとめ

## 🔑 重要ポイント

1. **MCP** は Copilot を外部サービス（GitHub、ファイルシステム、ドキュメント）に接続します
2. **GitHub MCP は組み込み** - 設定不要、`/login` するだけです
3. **Filesystem と Context7** は `~/.copilot/mcp-config.json` で設定します
4. **マルチサーバーワークフロー** により、1つのセッションで複数のデータソースを組み合わせられます
5. **サーバーのステータス確認** は `/mcp show` で行います（追加のコマンドも利用可能）
6. **カスタムサーバー** で任意の API を接続できます（オプション、付録のガイドで解説）

> 📋 **クイックリファレンス**: コマンドとショートカットの完全な一覧は [GitHub Copilot CLI command reference](https://docs.github.com/en/copilot/reference/cli-command-reference) をご覧ください。

---

## ➡️ 次のステップ

これで、モード、コンテキスト、ワークフロー、エージェント、スキル、そして MCP と、すべての構成要素が揃いました。次はこれらをすべて組み合わせていきます。

**[Chapter 07: すべてをまとめる](../07-putting-it-together/README.md)** では、以下を学びます：

- エージェント、スキル、MCP を統合したワークフロー
- アイデアからマージされた PR までの完全な機能開発
- フックによる自動化
- チーム環境でのベストプラクティス

---

**[← Chapter 05 に戻る](../05-skills/README.md)** | **[Chapter 07 へ進む →](../07-putting-it-together/README.md)**
