![Chapter 04: Agents and Custom Instructions](images/chapter-header.png)

> **Python コードレビュアー、テスト専門家、セキュリティレビュアーを…たった一つのツールで雇えるとしたら？**

Chapter 03 では、コードレビュー、リファクタリング、デバッグ、テスト生成、git 連携といった基本的なワークフローを習得しました。これらによって GitHub Copilot CLI の生産性は大きく向上しました。さあ、さらにその先へ進みましょう。

これまでは Copilot CLI を汎用アシスタントとして使ってきました。Agent（エージェント）を使うと、特定の役割と組み込みの基準を持たせることができます。たとえば、型ヒントや PEP 8 を強制するコードレビュアーや、pytest のテストケースを書くテストヘルパーなどです。同じプロンプトでも、専門的な指示を持つ Agent が処理すると、目に見えて良い結果が得られることを体験できます。

## 🎯 学習目標

この章を終えると、以下のことができるようになります：

- ビルトイン Agent の使用：Plan（`/plan`）、Code-review（`/review`）、自動 Agent（Explore、Task）の理解
- Agent ファイル（`.agent.md`）を使った専門 Agent の作成
- ドメイン特化タスクへの Agent の活用
- `/agent` や `--agent` による Agent の切り替え
- プロジェクト固有の基準に合わせたカスタムインストラクションファイルの作成

> ⏱️ **所要時間の目安**：約55分（読む時間 20分 + ハンズオン 35分）

---

## 🧩 身近なたとえ：専門家を雇う

家に問題が発生したとき、「なんでも屋」は呼びませんよね。専門家を呼びます：

| 問題 | 専門家 | 理由 |
|---------|------------|-----|
| 水漏れ | 配管工 | 配管の規格を知っており、専用の工具を持っている |
| 配線工事 | 電気工事士 | 安全要件を理解し、法規に準拠できる |
| 屋根の張り替え | 屋根職人 | 材料や地域の気候条件を熟知している |

Agent もまったく同じ仕組みです。汎用の AI の代わりに、特定のタスクに特化し、正しいプロセスを知っている Agent を使いましょう。一度指示を設定すれば、コードレビュー、テスト、セキュリティ、ドキュメント作成など、その専門性が必要なときにいつでも再利用できます。

<img src="images/hiring-specialists-analogy.png" alt="Hiring Specialists Analogy - Just as you call specialized tradespeople for house repairs, AI agents are specialized for specific tasks like code review, testing, security, and documentation" width="800" />

---

# Agent を使う

ビルトイン Agent やカスタム Agent をすぐに使い始めましょう。

---

## *Agent が初めて？* ここから始めよう！
Agent を使ったり作ったりしたことがない方へ。このコースで始めるために必要なことはこれだけです。

1. **今すぐ *ビルトイン* Agent を試してみましょう：**
   ```bash
   copilot
   > /plan Add input validation for book year in the book app
   ```
   これは Plan Agent を呼び出して、ステップバイステップの実装プランを作成します。

2. **カスタム Agent のサンプルを見てみましょう：** Agent の指示を定義するのはとても簡単です。提供されている [python-reviewer.agent.md](../.github/agents/python-reviewer.agent.md) ファイルを見て、パターンを確認してください。

3. **コアコンセプトを理解しましょう：** Agent は、ジェネラリストではなくスペシャリストに相談するようなものです。「フロントエンド Agent」はアクセシビリティやコンポーネントパターンに自動的に注力します。Agent の指示にすでに記述されているので、わざわざ指示する必要がありません。


## Built-in Agents

**You've already used some built-in agents in Chapter 03 Development Workflow!**
<br>`/plan` and `/review` are actually built-in agents. Now you know what's happening under the hood. Here's the full list:

| Agent | How to Invoke | What It Does |
|-------|---------------|--------------|
| **Plan** | `/plan` or `Shift+Tab` (cycle modes) | Creates step-by-step implementation plans before coding |
| **Code-review** | `/review` | Reviews staged/unstaged changes with focused, actionable feedback |
| **Init** | `/init` | Generates project configuration files (instructions, agents) |
| **Explore** | *Automatic* | Used internally when you ask Copilot to explore or analyze the codebase |
| **Task** | *Automatic* | Executes commands like tests, builds, lints, and dependency installs |

<br>

**Built-in agents in action** - Examples of invoking Plan, Code-review, Explore, and Task

```bash
copilot

# Invoke the Plan agent to create an implementation plan
> /plan Add input validation for book year in the book app

# Invoke the Code-review agent on your changes
> /review

# Explore and Task agents are invoked automatically when relevant:
> Run the test suite        # Uses Task agent

> Explore how book data is loaded    # Uses Explore agent
```

What about the Task Agent? It works behind the scenes to manage and track what is going on and to report back in a clean and clear format:

| Outcome | What You See |
|---------|--------------|
| ✅ **Success** | Brief summary (e.g., "All 247 tests passed", "Build succeeded") |
| ❌ **Failure** | Full output with stack traces, compiler errors, and detailed logs |


> 📚 **Official Documentation**: [GitHub Copilot CLI Agents](https://docs.github.com/copilot/how-tos/use-copilot-agents/use-copilot-cli#use-custom-agents)

---

# Copilot CLI に Agent を追加する

独自の Agent をワークフローに組み込むのはとても簡単です！一度定義したら、あとは指示するだけ！

<img src="images/using-agents.png" alt="Four colorful AI robots standing together, each with different tools representing specialized agent capabilities" width="800"/>

## 🗂️ Agent を追加する 

Agent ファイルは `.agent.md` 拡張子を持つ Markdown ファイルです。YAML frontmatter（メタデータ）と Markdown の指示文の 2 つのパートで構成されています。

> 💡 **YAML frontmatter が初めて？** ファイルの先頭に `---` で囲まれた小さな設定ブロックです。YAML は単なる `key: value` のペアです。それ以外は通常の Markdown です。

最小限の Agent の例：

```markdown
---
name: my-reviewer
description: Code reviewer focused on bugs and security issues
---

# Code Reviewer

You are a code reviewer focused on finding bugs and security issues.

When reviewing code, always check for:
- SQL injection vulnerabilities
- Missing error handling
- Hardcoded secrets
```

> 💡 **必須 vs 任意**：`description` フィールドは必須です。`name`、`tools`、`model` などのフィールドは任意です。

## Agent ファイルの配置場所

| 配置場所 | スコープ | 最適な用途 |
|----------|-------|----------|
| `.github/agents/` | プロジェクト固有 | プロジェクトの規約を含むチーム共有 Agent |
| `~/.copilot/agents/` | グローバル（全プロジェクト） | どのプロジェクトでも使う個人用 Agent |

**このプロジェクトには [.github/agents/](../.github/agents/) フォルダーにサンプル Agent ファイルが含まれています**。独自に作成することも、提供されているものをカスタマイズすることもできます。

<details>
<summary>📂 このコースのサンプル Agent を見る</summary>

| ファイル | 説明 |
|------|-----|
| `hello-world.agent.md` | 最小限の例 - まずここから始めましょう |
| `python-reviewer.agent.md` | Python コード品質レビュアー |
| `pytest-helper.agent.md` | Pytest テストスペシャリスト |

```bash
# または個人の Agent フォルダーにコピー（すべてのプロジェクトで利用可能）
cp .github/agents/python-reviewer.agent.md ~/.copilot/agents/
```

その他のコミュニティ Agent は [github/awesome-copilot](https://github.com/github/awesome-copilot) をご覧ください

</details>


## 🚀 カスタム Agent の 2 つの使い方

### インタラクティブモード
インタラクティブモード内で `/agent` を使って Agent 一覧を表示し、使いたい Agent を選択します。
選択した Agent と会話を続けることができます。

```bash
copilot
> /agent
```

別の Agent に切り替えたり、デフォルトモードに戻すには、再度 `/agent` コマンドを使います。

### プログラマティックモード

Agent を指定して新しいセッションを直接開始します。

```bash
copilot --agent python-reviewer
> Review @samples/book-app-project/books.py
```

> 💡 **Agent の切り替え**：`/agent` または `--agent` を使えば、いつでも別の Agent に切り替えられます。標準の Copilot CLI に戻るには、`/agent` で **no agent** を選択してください。

---

# Agent をさらに活用する

<img src="images/creating-custom-agents.png" alt="Robot being assembled on a workbench surrounded by components and tools representing custom agent creation" width="800"/>

> 💡 **このセクションは任意です。** ビルトイン Agent（`/plan`、`/review`）はほとんどのワークフローで十分な機能を持っています。カスタム Agent は、作業全体で一貫して適用したい専門的な知識が必要なときに作りましょう。

以下の各トピックはそれぞれ独立しています。**興味のあるものを選んでください - 一度に全部読む必要はありません。**

| やりたいこと... | ジャンプ先 |
|---|---|
| Agent が汎用プロンプトより優れている理由を見る | [スペシャリスト vs ジェネラリスト](#specialist-vs-generic-see-the-difference) |
| 複数の Agent を組み合わせて機能を開発 | [複数 Agent の連携](#working-with-multiple-agents) |
| Agent の整理、命名、共有 | [Agent の整理と共有](#organizing--sharing-agents) |
| プロジェクトコンテキストの常時適用設定 | [プロジェクト用の Copilot 設定](#configuring-your-project-for-copilot) |
| YAML プロパティとツールのリファレンス | [Agent ファイルリファレンス](#agent-file-reference) |

以下のシナリオをクリックして展開してください。

---

<a id="specialist-vs-generic-see-the-difference"></a>
<details>
<summary><strong>スペシャリスト vs ジェネラリスト：違いを見てみよう</strong> - Agent が汎用プロンプトより良い出力を生む理由</summary>

## スペシャリスト vs ジェネラリスト：違いを見てみよう

ここで Agent の価値が証明されます。その違いを見てみましょう：

### Agent なし（汎用 Copilot）

```bash
copilot

> Add a function to search books by year range in the book app
```

**汎用の出力**：
```python
def search_by_year_range(books, start_year, end_year):
    results = []
    for book in books:
        if book['year'] >= start_year and book['year'] <= end_year:
            results.append(book)
    return results
```

基本的です。動きます。でも足りないものがたくさんあります。

---

### Python Reviewer Agent を使った場合

```bash
copilot

> /agent
# "python-reviewer" を選択

> Add a function to search books by year range in the book app
```

**スペシャリストの出力**：
```python
from typing import List, Dict, Any

def search_by_year_range(
    books: List[Dict[str, Any]],
    start_year: int,
    end_year: int
) -> List[Dict[str, Any]]:
    """
    Search for books within a specified year range.

    Args:
        books: List of book dictionaries
        start_year: Starting year (inclusive)
        end_year: Ending year (inclusive)

    Returns:
        List of books published within the year range

    Raises:
        ValueError: If start_year is greater than end_year
        TypeError: If year values are not integers
    """
    if not isinstance(start_year, int) or not isinstance(end_year, int):
        raise TypeError("Year values must be integers")

    if start_year > end_year:
        raise ValueError(f"Start year ({start_year}) cannot be greater than end year ({end_year})")

    return [
        book for book in books
        if isinstance(book.get('year'), int)
        and start_year <= book['year'] <= end_year
    ]
```

**python-reviewer Agent が自動的に含めるもの**：
- ✅ すべてのパラメータと戻り値に型ヒント
- ✅ Args/Returns/Raises を含む包括的な docstring
- ✅ 適切なエラーハンドリングを伴う入力バリデーション
- ✅ パフォーマンス向上のためのリスト内包表記
- ✅ エッジケースの処理（年の値が欠落・無効な場合）
- ✅ PEP 8 準拠のフォーマット
- ✅ 防御的プログラミングの実践

**違い**：同じプロンプトで、劇的に良い出力が得られます。Agent は、あなたが言い忘れるような専門知識を自動的に提供します。

</details>

---

<a id="working-with-multiple-agents"></a>
<details>
<summary><strong>複数 Agent の連携</strong> - スペシャリストの組み合わせ、セッション中の切り替え、Agent-as-tools</summary>

## 複数 Agent の連携

真の力は、スペシャリストが協力して一つの機能を開発するときに発揮されます。

### 例：シンプルな機能を作る

```bash
copilot

> I want to add a "search by year range" feature to the book app

# python-reviewer で設計
> /agent
# "python-reviewer" を選択

> @samples/book-app-project/books.py Design a find_by_year_range method. What's the best approach?

# pytest-helper に切り替えてテスト設計
> /agent
# "pytest-helper" を選択

> @samples/book-app-project/tests/test_books.py Design test cases for a find_by_year_range method.
> What edge cases should we cover?

# 両方の設計を統合
> Create an implementation plan that includes the method implementation and comprehensive tests.
```

**重要なポイント**：あなたがアーキテクトとしてスペシャリストを指揮します。スペシャリストが詳細を担当し、あなたがビジョンを担当します。

<details>
<summary>🎬 実際の動作を見てみましょう！</summary>

![Python Reviewer Demo](images/python-reviewer-demo.gif)

*デモの出力は異なる場合があります - 使用するモデル、ツール、応答はここに表示されているものと異なります。*

</details>

### Agent as Tools

Agent が設定されていると、Copilot は複雑なタスク中にそれらをツールとして呼び出すこともできます。フルスタックの機能を依頼すると、Copilot が自動的に適切なスペシャリスト Agent に部分を委任することがあります。

</details>

---

<a id="organizing--sharing-agents"></a>
<details>
<summary><strong>Agent の整理と共有</strong> - 命名、ファイル配置、インストラクションファイル、チーム共有</summary>

## Agent の整理と共有

### Agent の命名

Agent ファイルを作成するとき、名前は重要です。`/agent` や `--agent` の後に入力する名前であり、チームメイトが Agent 一覧で見る名前でもあります。

| ✅ 良い名前 | ❌ 避けるべき名前 |
|--------------|----------|
| `frontend` | `my-agent` |
| `backend-api` | `agent1` |
| `security-reviewer` | `helper` |
| `react-specialist` | `code` |
| `python-backend` | `assistant` |

**命名規則：**
- 小文字とハイフンを使用：`my-agent-name.agent.md`
- ドメインを含める：`frontend`、`backend`、`devops`、`security`
- 必要に応じて具体的に：`react-typescript` vs 単なる `frontend`

---

### チームとの共有

Agent ファイルを `.github/agents/` に置くと、バージョン管理されます。リポジトリにプッシュすれば、チームメンバー全員が自動的に利用できます。ただし、Agent は Copilot がプロジェクトから読み取るファイルの一種類に過ぎません。Copilot は、誰も `/agent` を実行しなくてもすべてのセッションで自動的に適用される**インストラクションファイル**もサポートしています。

こう考えてください：Agent は必要なときに呼び出すスペシャリスト、インストラクションファイルは常に有効なチームルールです。

### ファイルの配置場所

主な 2 つの配置場所はすでに学びました（上記の [Agent ファイルの配置場所](#where-to-put-agent-files) を参照）。以下のディシジョンツリーで選びましょう：

<img src="images/agent-file-placement-decision-tree.png" alt="Decision tree for where to put agent files: experimenting → current folder, team use → .github/agents/, everywhere → ~/.copilot/agents/" width="800"/>

**まずはシンプルに：** プロジェクトフォルダーに `*.agent.md` ファイルを 1 つ作成します。満足したら永続的な場所に移動しましょう。

Agent ファイル以外にも、Copilot はプロジェクトレベルの**インストラクションファイル**を自動的に読み込みます（`/agent` 不要）。`AGENTS.md`、`.instructions.md`、`/init` については、以下の [プロジェクト用の Copilot 設定](#configuring-your-project-for-copilot) をご覧ください。

</details>

---

<a id="configuring-your-project-for-copilot"></a>
<details>
<summary><strong>プロジェクト用の Copilot 設定</strong> - AGENTS.md、インストラクションファイル、/init セットアップ</summary>

## プロジェクト用の Copilot 設定

Agent は必要なときに呼び出すスペシャリストです。**プロジェクト設定ファイル**はそれとは異なります：Copilot がすべてのセッションで自動的に読み取り、プロジェクトの規約、技術スタック、ルールを理解します。誰も `/agent` を実行する必要はなく、リポジトリで作業する全員にコンテキストが常に有効です。

### /init でクイックセットアップ

最も簡単な始め方は、Copilot に設定ファイルを自動生成させることです：

```bash
copilot
> /init
```

Copilot がプロジェクトをスキャンし、プロジェクトに合わせたインストラクションファイルを作成します。後から編集することもできます。

### インストラクションファイルの形式

| ファイル | スコープ | 備考 |
|------|-------|-------|
| `AGENTS.md` | プロジェクトルートまたはネスト | **クロスプラットフォーム標準** - Copilot や他の AI アシスタントで動作 |
| `.github/copilot-instructions.md` | プロジェクト | GitHub Copilot 専用 |
| `.github/instructions/*.instructions.md` | プロジェクト | トピック別の細かいインストラクション |
| `CLAUDE.md`、`GEMINI.md` | プロジェクトルート | 互換性のためサポート |

> 🎯 **まずはこれから？** プロジェクトのインストラクションには `AGENTS.md` を使いましょう。他の形式は必要に応じて後から探索できます。

### AGENTS.md

`AGENTS.md` は推奨される形式です。Copilot や他の AI コーディングツールで動作する[オープン標準](https://agents.md/)です。リポジトリのルートに配置すると、Copilot が自動的に読み取ります。このプロジェクトの [AGENTS.md](../AGENTS.md) が実際の動作例です。

一般的な `AGENTS.md` には、プロジェクトのコンテキスト、コードスタイル、セキュリティ要件、テスト基準を記述します。`/init` で生成するか、提供されている例を参考に独自に作成してください。

### カスタムインストラクションファイル（.instructions.md）

より細かい制御が必要なチームは、インストラクションをトピック別のファイルに分割できます。各ファイルが 1 つの関心事をカバーし、自動的に適用されます：

```
.github/
└── instructions/
    ├── python-standards.instructions.md
    ├── security-checklist.instructions.md
    └── api-design.instructions.md
```

> 💡 **注意**：インストラクションファイルはどの言語でも動作します。この例ではコースプロジェクトに合わせて Python を使っていますが、TypeScript、Go、Rust など、チームが使用する任意の技術で同様のファイルを作成できます。

**コミュニティのインストラクションファイルを探す**：[github/awesome-copilot](https://github.com/github/awesome-copilot) で、.NET、Angular、Azure、Python、Docker など多くの技術向けの既製インストラクションファイルを見つけられます。

### カスタムインストラクションの無効化

Copilot にプロジェクト固有の設定をすべて無視させたい場合（デバッグや動作比較に便利）：

```bash
copilot --no-custom-instructions
```

</details>

---

<a id="agent-file-reference"></a>
<details>
<summary><strong>Agent ファイルリファレンス</strong> - YAML プロパティ、ツールエイリアス、完全な例</summary>

## Agent ファイルリファレンス

### より詳細な例

上記で[最小限の Agent 形式](#-agent-を追加する)を見ました。ここでは `tools` プロパティを使用する、より包括的な Agent を紹介します。`~/.copilot/agents/python-reviewer.agent.md` を作成します：

```markdown
---
name: python-reviewer
description: Python code quality specialist for reviewing Python projects
tools: ["read", "edit", "search", "execute"]
---

# Python Code Reviewer

You are a Python specialist focused on code quality and best practices.

**Your focus areas:**
- Code quality (PEP 8, type hints, docstrings)
- Performance optimization (list comprehensions, generators)
- Error handling (proper exception handling)
- Maintainability (DRY principles, clear naming)

**Code style requirements:**
- Use Python 3.10+ features (dataclasses, type hints, pattern matching)
- Follow PEP 8 naming conventions
- Use context managers for file I/O
- All functions must have type hints and docstrings

**When reviewing code, always check:**
- Missing type hints on function signatures
- Mutable default arguments
- Proper error handling (no bare except)
- Input validation completeness
```

### YAML プロパティ

| プロパティ | 必須 | 説明 |
|----------|----------|-----|
| `name` | いいえ | 表示名（デフォルトはファイル名） |
| `description` | **はい** | Agent の役割 - Copilot がいつ提案すべきか理解するのに役立つ |
| `tools` | いいえ | 許可するツールのリスト（省略 = 全ツール利用可）。以下のツールエイリアスを参照。 |
| `target` | いいえ | `vscode` または `github-copilot` のみに制限 |

### ツールエイリアス

`tools` リストでは以下の名前を使用します：
- `read` - ファイルの内容を読み取る
- `edit` - ファイルを編集する
- `search` - ファイルを検索（grep/glob）
- `execute` - シェルコマンドを実行（別名：`shell`、`Bash`）
- `agent` - 他のカスタム Agent を呼び出す

> 📖 **公式ドキュメント**：[Custom agents configuration](https://docs.github.com/copilot/reference/custom-agents-configuration)
>
> ⚠️ **VS Code のみ**：`model` プロパティ（AI モデルの選択）は VS Code では動作しますが、GitHub Copilot CLI ではサポートされていません。クロスプラットフォーム対応の Agent ファイルに含めても問題ありません。GitHub Copilot CLI はそれを無視します。

### その他の Agent テンプレート

> 💡 **初心者の方へ**：以下の例はテンプレートです。**具体的な技術は、あなたのプロジェクトで使うものに置き換えてください。** 重要なのは Agent の*構造*であり、記載されている具体的な技術ではありません。

このプロジェクトには [.github/agents/](../.github/agents/) フォルダーに動作する例が含まれています：
- [hello-world.agent.md](../.github/agents/hello-world.agent.md) - 最小限の例、まずここから
- [python-reviewer.agent.md](../.github/agents/python-reviewer.agent.md) - Python コード品質レビュアー
- [pytest-helper.agent.md](../.github/agents/pytest-helper.agent.md) - Pytest テストスペシャリスト

コミュニティ Agent は [github/awesome-copilot](https://github.com/github/awesome-copilot) をご覧ください。

</details>

---

# 実践練習

<img src="../images/practice.png" alt="Warm desk setup with monitor showing code, lamp, coffee cup, and headphones ready for hands-on practice" width="800"/>

独自の Agent を作成して、実際に動かしてみましょう。

---

## ▶️ 試してみよう

```bash

# agents ディレクトリを作成（存在しない場合）
mkdir -p .github/agents

# コードレビュアー Agent を作成
cat > .github/agents/reviewer.agent.md << 'EOF'
---
name: reviewer
description: Senior code reviewer focused on security and best practices
---

# Code Reviewer Agent

You are a senior code reviewer focused on code quality.

**Review priorities:**
1. Security vulnerabilities
2. Performance issues
3. Maintainability concerns
4. Best practice violations

**Output format:**
Provide issues as a numbered list with severity tags:
[CRITICAL], [HIGH], [MEDIUM], [LOW]
EOF

# ドキュメント Agent を作成
cat > .github/agents/documentor.agent.md << 'EOF'
---
name: documentor
description: Technical writer for clear and complete documentation
---

# Documentation Agent

You are a technical writer who creates clear documentation.

**Documentation standards:**
- Start with a one-sentence summary
- Include usage examples
- Document parameters and return values
- Note any gotchas or limitations
EOF

# Agent を使ってみる
copilot --agent reviewer
> Review @samples/book-app-project/books.py

# または Agent を切り替える
copilot
> /agent
# "documentor" を選択
> Document @samples/book-app-project/books.py
```

---

## 📝 課題

### メインチャレンジ：専門 Agent チームを作る

ハンズオン例で `reviewer` と `documentor` Agent を作成しました。次は、別のタスク——ブックアプリのデータバリデーション改善——のために Agent を作成・活用する練習です：

1. 3 つの Agent ファイル（`.agent.md`）をブックアプリ向けに作成し、`.github/agents/` に配置
2. 作成する Agent：
   - **data-validator**：`data.json` の欠落や不正データ（空の著者名、year=0、欠落フィールド）をチェック
   - **error-handler**：Python コードの一貫性のないエラーハンドリングをレビューし、統一的なアプローチを提案
   - **doc-writer**：docstring や README コンテンツを生成・更新
3. 各 Agent をブックアプリで使用：
   - `data-validator` → `@samples/book-app-project/data.json` を監査
   - `error-handler` → `@samples/book-app-project/books.py` と `@samples/book-app-project/utils.py` をレビュー
   - `doc-writer` → `@samples/book-app-project/books.py` に docstring を追加
4. Agent の連携：`error-handler` でエラーハンドリングの欠落を特定し、次に `doc-writer` で改善されたアプローチをドキュメント化

**成功基準**：3 つの動作する Agent が一貫性のある高品質な出力を生成し、`/agent` でそれらを切り替えられること。

<details>
<summary>💡 ヒント（クリックで展開）</summary>

**スターターテンプレート**：`.github/agents/` に Agent ごとに 1 ファイル作成：

`data-validator.agent.md`:
```markdown
---
description: Analyzes JSON data files for missing or malformed entries
---

You analyze JSON data files for missing or malformed entries.

**Focus areas:**
- Empty or missing author fields
- Invalid years (year=0, future years, negative years)
- Missing required fields (title, author, year, read)
- Duplicate entries
```

`error-handler.agent.md`:
```markdown
---
description: Reviews Python code for error handling consistency
---

You review Python code for error handling consistency.

**Standards:**
- No bare except clauses
- Use custom exceptions where appropriate
- All file operations use context managers
- Consistent return types for success/failure
```

`doc-writer.agent.md`:
```markdown
---
description: Technical writer for clear Python documentation
---

You are a technical writer who creates clear Python documentation.

**Standards:**
- Google-style docstrings
- Include parameter types and return values
- Add usage examples for public methods
- Note any exceptions raised
```

**Agent のテスト：**

> 💡 **注意：** `samples/book-app-project/data.json` はこのリポジトリのローカルコピーに含まれているはずです。万一見つからない場合は、ソースリポジトリからオリジナルをダウンロードしてください：
> [data.json](https://github.com/github/copilot-cli-for-beginners/blob/main/samples/book-app-project/data.json)

```bash
copilot
> /agent
# リストから "data-validator" を選択
> @samples/book-app-project/data.json Check for books with empty author fields or invalid years
```

**ヒント：** YAML frontmatter の `description` フィールドは Agent が動作するために必須です。

</details>

### ボーナスチャレンジ：インストラクションライブラリ

ここまで、必要なときに呼び出す Agent を作りました。次は反対側：`/agent` 不要で Copilot がすべてのセッションで自動的に読み取る**インストラクションファイル**を試してみましょう。

`.github/instructions/` フォルダーを作成し、最低 3 つのインストラクションファイルを作りましょう：
- `python-style.instructions.md`：PEP 8 と型ヒント規約の強制
- `test-standards.instructions.md`：テストファイルでの pytest 規約の強制
- `data-quality.instructions.md`：JSON データエントリのバリデーション

各インストラクションファイルをブックアプリのコードでテストしましょう。

---

<details>
<summary>🔧 <strong>よくある間違いとトラブルシューティング</strong>（クリックで展開）</summary>

### よくある間違い

| 間違い | 何が起こるか | 修正方法 |
|---------|---------|-----|
| Agent の frontmatter に `description` がない | Agent が読み込まれない、または検出されない | YAML frontmatter に必ず `description:` を含める |
| Agent ファイルの配置場所が間違っている | 使おうとしても Agent が見つからない | `~/.copilot/agents/`（個人）または `.github/agents/`（プロジェクト）に配置 |
| `.agent.md` ではなく `.md` を使っている | ファイルが Agent として認識されない可能性 | `python-reviewer.agent.md` のように命名 |
| Agent のプロンプトが長すぎる | 30,000 文字制限に引っかかる可能性 | Agent の定義は簡潔に保ち、詳細な指示にはスキルを使用 |

### トラブルシューティング

**Agent が見つからない** - Agent ファイルが以下のいずれかの場所にあるか確認してください：
- `~/.copilot/agents/`
- `.github/agents/`

利用可能な Agent の一覧を表示：

```bash
copilot
> /agent
# 利用可能なすべての Agent が表示されます
```

**Agent が指示に従わない** - プロンプトをより明確にし、Agent の定義に詳細を追加しましょう：
- 具体的なフレームワーク/ライブラリとバージョン
- チームの規約
- コードパターンの例

**カスタムインストラクションが読み込まれない** - プロジェクトで `/init` を実行してプロジェクト固有のインストラクションをセットアップ：

```bash
copilot
> /init
```

または無効化されていないか確認：
```bash
# カスタムインストラクションを読み込む場合は --no-custom-instructions を使わない
copilot  # デフォルトでカスタムインストラクションが読み込まれます
```

</details>

---

# まとめ

## 🔑 重要ポイント

1. **ビルトイン Agent**：`/plan` と `/review` は直接呼び出し、Explore と Task は自動で動作
2. **カスタム Agent** は `.agent.md` ファイルで定義されるスペシャリスト
3. **良い Agent** には明確な専門性、基準、出力形式がある
4. **複数 Agent の連携**により、専門知識を組み合わせて複雑な問題を解決
5. **インストラクションファイル**（`.instructions.md`）はチームの基準をエンコードして自動適用
6. **一貫性のある出力**は、よく定義された Agent の指示から生まれる

> 📋 **クイックリファレンス**：コマンドとショートカットの完全な一覧は [GitHub Copilot CLI コマンドリファレンス](https://docs.github.com/en/copilot/reference/cli-command-reference) をご覧ください。

---

## ➡️ 次のステップ

Agent は Copilot がコードに*どのようにアプローチし、対象を絞ったアクションを取る*かを変えます。次は、*どのようなステップを踏む*かを変える **スキル** について学びます。Agent とスキルの違いが気になりますか？Chapter 05 で正面から取り上げます。

**[Chapter 05: スキルシステム](../05-skills/README.md)** では、以下を学びます：

- プロンプトからスキルが自動トリガーされる仕組み（スラッシュコマンド不要）
- コミュニティスキルのインストール
- SKILL.md ファイルでのカスタムスキル作成
- Agent、スキル、MCP の違い
- それぞれをいつ使うべきか

---

**[← Chapter 03 に戻る](../03-development-workflows/README.md)** | **[Chapter 05 へ進む →](../05-skills/README.md)**
