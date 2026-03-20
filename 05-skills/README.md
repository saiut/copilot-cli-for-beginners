![Chapter 05: スキルシステム](images/chapter-header.png)

> **チームのベストプラクティスを毎回説明しなくても、Copilot が自動的に適用してくれたら？**

この章では、Agent Skills（エージェントスキル）について学びます。スキルとは、タスクに関連するときに Copilot が自動的に読み込む指示のフォルダです。エージェントが Copilot の「考え方」を変えるのに対し、スキルは Copilot に「特定のタスクの完了方法」を教えます。セキュリティに関する質問をすると自動的に適用されるセキュリティ監査スキルの作成、一貫したコード品質を保証するチーム標準のレビュー基準の構築、Copilot CLI・VS Code・Copilot コーディングエージェントでのスキルの動作について学びます。


## 🎯 学習目標

この章を終えると、以下のことができるようになります：

- Agent Skills の仕組みと使いどころを理解する
- SKILL.md ファイルを使ってカスタムスキルを作成する
- 共有リポジトリのコミュニティスキルを利用する
- スキル・エージェント・MCP の使い分けを判断する

> ⏱️ **所要時間の目安**: 約55分（読む時間 20分 + ハンズオン 35分）

---

## 🧩 たとえ話：電動工具

汎用のドリルは便利ですが、専用のアタッチメントがあればもっと強力になります。
<img src="images/power-tools-analogy.png" alt="Power Tools - Skills Extend Copilot's Capabilities" width="800"/>


スキルも同じ仕組みです。作業に応じてドリルのビットを交換するように、さまざまなタスクに合わせて Copilot にスキルを追加できます：

| スキル（アタッチメント） | 用途 |
|------------|---------|
| `commit` | 一貫したコミットメッセージを生成する |
| `security-audit` | OWASP の脆弱性をチェックする |
| `generate-tests` | 包括的な pytest テストを作成する |
| `code-checklist` | チームのコード品質基準を適用する |



*スキルは Copilot の機能を拡張する専用アタッチメントです*

---

# スキルの仕組み

<img src="images/how-skills-work.png" alt="Glowing RPG-style skill icons connected by light trails on a starfield background representing Copilot skills" width="800"/>

スキルとは何か、なぜ重要か、エージェントや MCP との違いを学びます。

---

## *スキルが初めての方は* ここから始めましょう！

1. **利用可能なスキルを確認する：**
   ```bash
   copilot
   > /skills list
   ```
   プロジェクトフォルダと個人フォルダにある全スキルが表示されます。

2. **実際のスキルファイルを見てみましょう：** 提供されている [code-checklist SKILL.md](../.github/skills/code-checklist/SKILL.md) を確認してください。YAML frontmatter と Markdown の指示文だけで構成されています。

3. **基本コンセプトを理解する：** スキルはタスク固有の指示で、プロンプトがスキルの説明に一致すると Copilot が*自動的に*読み込みます。手動で有効化する必要はなく、自然に質問するだけで大丈夫です。


## スキルを理解する

Agent Skills は、タスクに**関連するときに Copilot が自動的に読み込む**指示、スクリプト、リソースを含むフォルダです。Copilot はプロンプトを読み取り、一致するスキルがあるか確認し、該当する指示を自動的に適用します。

```bash
copilot

> Check books.py against our quality checklist
# Copilot が「code-checklist」スキルに一致すると判断し、
# Python 品質チェックリストを自動的に適用します

> Generate tests for the BookCollection class
# Copilot が「pytest-gen」スキルを読み込み、
# 設定されたテスト構造を適用します

> What are the code quality issues in this file?
# Copilot が「code-checklist」スキルを読み込み、
# チームの基準に照らしてチェックします
```

> 💡 **ポイント**: スキルは、プロンプトがスキルの説明に一致すると**自動的にトリガー**されます。自然に質問するだけで、Copilot が裏側で関連するスキルを適用します。次に紹介するように、スキルを直接呼び出すこともできます。

> 🧰 **すぐ使えるテンプレート**: [.github/skills](../.github/skills/) フォルダに、コピー＆ペーストで試せるシンプルなスキルが用意されています。

### スラッシュコマンドによる直接呼び出し

自動トリガーがスキルの主な動作方法ですが、スキル名をスラッシュコマンドとして使って**直接呼び出す**こともできます：

```bash
> /generate-tests Create tests for the user authentication module

> /code-checklist Check books.py for code quality issues

> /security-audit Check the API endpoints for vulnerabilities
```

特定のスキルを確実に使いたいときに、明示的にコントロールできます。

> 📝 **スキルとエージェントの呼び出しの違い**: スキルの呼び出しとエージェントの呼び出しを混同しないでください：
> - **スキル**: `/skill-name <プロンプト>`（例：`/code-checklist Check this file`）
> - **エージェント**: `/agent`（リストから選択）または `copilot --agent <名前>`（コマンドライン）
>
> 同じ名前のスキルとエージェントがある場合（例：「code-reviewer」）、`/code-reviewer` と入力するとエージェントではなく**スキル**が呼び出されます。

### スキルが使われたかどうか確認するには？

Copilot に直接聞くことができます：

```bash
> What skills did you use for that response?

> What skills do you have available for security reviews?
```

### スキル vs エージェント vs MCP

スキルは GitHub Copilot の拡張モデルの一部にすぎません。エージェントや MCP サーバーとの比較を見てみましょう。

> *MCP についてはまだ心配しなくて大丈夫です。[Chapter 06](../06-mcp-servers/) で詳しく説明します。スキルが全体像のどこに位置するかを把握するために、ここに含めています。*

<img src="images/skills-agents-mcp-comparison.png" alt="Comparison diagram showing the differences between Agents, Skills, and MCP Servers and how they combine into your workflow" width="800"/>

| 機能 | 何をするか | いつ使うか |
|---------|--------------|-------------|
| **エージェント** | AI の思考方法を変える | 多くのタスクにわたる専門知識が必要なとき |
| **スキル** | タスク固有の指示を提供する | 詳細なステップのある、特定の繰り返しタスク |
| **MCP** | 外部サービスに接続する | API からのライブデータが必要なとき |

幅広い専門知識にはエージェント、特定のタスクの指示にはスキル、外部データには MCP を使いましょう。エージェントは会話中に1つ以上のスキルを使用できます。たとえば、エージェントにコードをチェックしてもらうと、`security-audit` スキルと `code-checklist` スキルの両方が自動的に適用されることがあります。

> 📚 **詳細はこちら**: スキルの形式やベストプラクティスの完全なリファレンスは、公式ドキュメント [About Agent Skills](https://docs.github.com/copilot/concepts/agents/about-agent-skills) を参照してください。

---

## 手動プロンプトから自動的な専門知識へ

スキルの作成方法に入る前に、スキルを学ぶ*意義*を見てみましょう。一貫性の向上を実感すれば、「作り方」も自然に理解できます。

### スキルなし：一貫性のないレビュー

コードレビューのたびに、何かを忘れてしまうかもしれません：

```bash
copilot

> Review this code for issues
# 一般的なレビュー - チーム固有の懸念点を見落とすかもしれません
```

あるいは、毎回長いプロンプトを書くことになります：

```bash
> Review this code checking for bare except clauses, missing type hints,
> mutable default arguments, missing context managers for file I/O,
> functions over 50 lines, print statements in production code...
```

入力時間：**30秒以上**。一貫性：**記憶力次第**。

### スキルあり：自動的なベストプラクティス

`code-checklist` スキルがインストールされていれば、自然に質問するだけです：

```bash
copilot

> Check the book collection code for quality issues
```

**裏側で起きていること**：
1. Copilot がプロンプト内の「code quality」と「issues」を検出
2. スキルの説明を確認し、`code-checklist` スキルが一致すると判断
3. チームの品質チェックリストを自動的に読み込み
4. すべてのチェック項目を、一覧を入力することなく適用

<img src="images/skill-auto-discovery-flow.png" alt="How Skills Auto-Trigger - 4-step flow showing how Copilot automatically matches your prompt to the right skill" width="800"/>

*自然に質問するだけです。Copilot がプロンプトを適切なスキルに一致させ、自動的に適用します。*

**出力例**：
```
## Code Checklist: books.py

### Code Quality
- [PASS] All functions have type hints
- [PASS] No bare except clauses
- [PASS] No mutable default arguments
- [PASS] Context managers used for file I/O
- [PASS] Functions are under 50 lines
- [PASS] Variable and function names follow PEP 8

### Input Validation
- [FAIL] User input is not validated - add_book() accepts any year value
- [FAIL] Edge cases not fully handled - empty strings accepted for title/author
- [PASS] Error messages are clear and helpful

### Testing
- [FAIL] No corresponding pytest tests found

### Summary
3 items need attention before merge
```

**違い**: チームの基準が毎回、入力なしで自動的に適用されます。

---

<details>
<summary>🎬 動作を確認しましょう！</summary>

![Skill Trigger Demo](images/skill-trigger-demo.gif)

*デモの出力は環境により異なります。使用するモデル、ツール、レスポンスはここに表示されているものと異なる場合があります。*

</details>

---

## スケールでの一貫性：チーム PR レビュースキル

チームに10項目の PR チェックリストがあるとします。スキルがなければ、すべての開発者が10項目すべてを覚えなければならず、誰かが必ず何か1つを忘れます。`pr-review` スキルがあれば、チーム全体で一貫したレビューが行えます：

```bash
copilot

> Can you review this PR?
```

Copilot がチームの `pr-review` スキルを自動的に読み込み、10項目すべてをチェックします：

```
PR Review: feature/user-auth

## Security ✅
- No hardcoded secrets
- Input validation present
- No bare except clauses

## Code Quality ⚠️
- [WARN] print statement on line 45 - remove before merge
- [WARN] TODO on line 78 missing issue reference
- [WARN] Missing type hints on public functions

## Testing ✅
- New tests added
- Edge cases covered

## Documentation ❌
- [FAIL] Breaking change not documented in CHANGELOG
- [FAIL] API changes need OpenAPI spec update
```

**スキルの力**: チーム全員が自動的に同じ基準を適用できます。新入社員もチェックリストを暗記する必要がありません。スキルが代わりにやってくれます。

---

# カスタムスキルの作成

<img src="images/creating-managing-skills.png" alt="Human and robotic hands building a wall of glowing LEGO-like blocks representing skill creation and management" width="800"/>

SKILL.md ファイルから独自のスキルを構築します。

---

## スキルの保存場所

スキルは `.github/skills/`（プロジェクト固有）または `~/.copilot/skills/`（ユーザーレベル）に保存されます。

### Copilot がスキルを見つける方法

Copilot は以下の場所を自動的にスキャンしてスキルを探します：

| 場所 | スコープ |
|----------|-------|
| `.github/skills/` | プロジェクト固有（git でチームと共有） |
| `~/.copilot/skills/` | ユーザー固有（個人用スキル） |

### スキルの構造

各スキルは独自のフォルダに `SKILL.md` ファイルを持ちます。オプションでスクリプト、サンプル、その他のリソースを含めることもできます：

```
.github/skills/
└── my-skill/
    ├── SKILL.md           # 必須: スキルの定義と指示
    ├── examples/          # オプション: Copilot が参照できるサンプルファイル
    │   └── sample.py
    └── scripts/           # オプション: スキルが使用できるスクリプト
        └── validate.sh
```

> 💡 **ヒント**: ディレクトリ名は SKILL.md の frontmatter にある `name` と一致させましょう（小文字でハイフン区切り）。

### SKILL.md の形式

スキルは YAML frontmatter 付きのシンプルな Markdown 形式を使用します：

```markdown
---
name: code-checklist
description: Comprehensive code quality checklist with security, performance, and maintainability checks
license: MIT
---

# Code Checklist

When checking code, look for:

## Security
- SQL injection vulnerabilities
- XSS vulnerabilities
- Authentication/authorization issues
- Sensitive data exposure

## Performance
- N+1 query problems (running one query per item instead of one query for all items)
- Unnecessary loops or computations
- Memory leaks
- Blocking operations

## Maintainability
- Function length (flag functions > 50 lines)
- Code duplication
- Missing error handling
- Unclear naming

## Output Format
Provide issues as a numbered list with severity:
- [CRITICAL] - Must fix before merge
- [HIGH] - Should fix before merge
- [MEDIUM] - Should address soon
- [LOW] - Nice to have
```

**YAML プロパティ：**

| プロパティ | 必須 | 説明 |
|----------|----------|-------------|
| `name` | **はい** | 一意の識別子（小文字、スペースはハイフン） |
| `description` | **はい** | スキルの内容と Copilot が使用すべきタイミング |
| `license` | いいえ | このスキルに適用されるライセンス |

> 📖 **公式ドキュメント**: [About Agent Skills](https://docs.github.com/copilot/concepts/agents/about-agent-skills)

### 最初のスキルを作成する

OWASP Top 10 の脆弱性をチェックするセキュリティ監査スキルを作りましょう：

```bash
# スキルディレクトリを作成
mkdir -p .github/skills/security-audit

# SKILL.md ファイルを作成
cat > .github/skills/security-audit/SKILL.md << 'EOF'
---
name: security-audit
description: Security-focused code review checking OWASP (Open Web Application Security Project) Top 10 vulnerabilities
---

# Security Audit

Perform a security audit checking for:

## Injection Vulnerabilities
- SQL injection (string concatenation in queries)
- Command injection (unsanitized shell commands)
- LDAP injection
- XPath injection

## Authentication Issues
- Hardcoded credentials
- Weak password requirements
- Missing rate limiting
- Session management flaws

## Sensitive Data
- Plaintext passwords
- API keys in code
- Logging sensitive information
- Missing encryption

## Access Control
- Missing authorization checks
- Insecure direct object references
- Path traversal vulnerabilities

## Output
For each issue found, provide:
1. File and line number
2. Vulnerability type
3. Severity (CRITICAL/HIGH/MEDIUM/LOW)
4. Recommended fix
EOF

# スキルをテストする（スキルはプロンプトに基づいて自動的に読み込まれます）
copilot

> @samples/book-app-project/ Check this code for security vulnerabilities
# Copilot が「security vulnerabilities」がスキルに一致すると検出し、
# OWASP チェックリストを自動的に適用します
```

**期待される出力**（結果は環境により異なります）：

```
Security Audit: book-app-project

[HIGH] Hardcoded file path (book_app.py, line 12)
  File path is hardcoded rather than configurable
  Fix: Use environment variable or config file

[MEDIUM] No input validation (book_app.py, line 34)
  User input passed directly to function without sanitization
  Fix: Add input validation before processing

✅ No SQL injection found
✅ No hardcoded credentials found
```

---

## 良いスキル説明文の書き方

SKILL.md の `description` フィールドは非常に重要です！ Copilot がスキルを読み込むかどうかを決定する手段だからです：

```markdown
---
name: security-audit
description: Use for security reviews, vulnerability scanning,
  checking for SQL injection, XSS, authentication issues,
  OWASP Top 10 vulnerabilities, and security best practices
---
```

> 💡 **ヒント**: 普段の質問の仕方に一致するキーワードを含めましょう。「security review」と質問するなら、説明文にも「security review」を含めてください。

### スキルとエージェントの組み合わせ

スキルとエージェントは連携して動作します。エージェントが専門知識を提供し、スキルが具体的な指示を提供します：

```bash
# code-reviewer エージェントで開始
copilot --agent code-reviewer

> Check the book app for quality issues
# code-reviewer エージェントの専門知識と
# code-checklist スキルのチェックリストが組み合わされます
```

---

# スキルの管理と共有

インストール済みスキルの確認、コミュニティスキルの発見、自分のスキルの共有方法を学びます。

<img src="images/managing-sharing-skills.png" alt="Managing and Sharing Skills - showing the discover, use, create, and share cycle for CLI skills" width="800" />

---

## `/skills` コマンドでスキルを管理する

`/skills` コマンドを使ってインストール済みのスキルを管理できます：

| コマンド | 機能 |
|---------|--------------|
| `/skills list` | インストール済みのすべてのスキルを表示 |
| `/skills info <名前>` | 特定のスキルの詳細を表示 |
| `/skills add <名前>` | スキルを有効化（リポジトリやマーケットプレイスから） |
| `/skills remove <名前>` | スキルを無効化またはアンインストール |
| `/skills reload` | SKILL.md ファイル編集後にスキルを再読み込み |

> 💡 **覚えておこう**: プロンプトごとにスキルを「有効化」する必要はありません。インストールされているスキルは、プロンプトが説明に一致すると**自動的にトリガー**されます。これらのコマンドは、利用可能なスキルの管理用であり、使用するためのものではありません。

### 例：スキルを確認する

```bash
copilot

> /skills list

Available skills:
- security-audit: Security-focused code review checking OWASP Top 10
- generate-tests: Generate comprehensive unit tests with edge cases
- code-checklist: Team code quality checklist
...

> /skills info security-audit

Skill: security-audit
Source: Project
Location: .github/skills/security-audit/SKILL.md
Description: Security-focused code review checking OWASP Top 10 vulnerabilities
```

---

<details>
<summary>動作を確認しましょう！</summary>

![List Skills Demo](images/list-skills-demo.gif)

*デモの出力は環境により異なります。使用するモデル、ツール、レスポンスはここに表示されているものと異なる場合があります。*

</details>

---

### `/skills reload` を使うタイミング

スキルの SKILL.md ファイルを作成または編集した後、Copilot を再起動せずに変更を反映するには `/skills reload` を実行します：

```bash
# スキルファイルを編集した後、
# Copilot で以下を実行：
> /skills reload
Skills reloaded successfully.
```

> 💡 **知っておくと便利**: `/compact` で会話履歴を要約した後もスキルは有効です。コンパクト化の後に再読み込みする必要はありません。

---

## コミュニティスキルの発見と利用

### プラグインでスキルをインストールする

> 💡 **プラグインとは？** プラグインは、スキル、エージェント、MCP サーバー設定を一緒にバンドルできるインストール可能なパッケージです。Copilot CLI 用の「アプリストア」の拡張機能のようなものです。

`/plugin` コマンドでこれらのパッケージを閲覧・インストールできます：

```bash
copilot

> /plugin list
# インストール済みプラグインを表示

> /plugin marketplace
# 利用可能なプラグインを閲覧

> /plugin install <プラグイン名>
# マーケットプレイスからプラグインをインストール
```

プラグインは複数の機能をバンドルできます。1つのプラグインに、連携して動作する関連スキル、エージェント、MCP サーバー設定が含まれることがあります。

### コミュニティスキルリポジトリ

コミュニティリポジトリから既製のスキルも入手できます：

- **[Awesome Copilot](https://github.com/github/awesome-copilot)** - スキルのドキュメントやサンプルを含む GitHub Copilot の公式リソース

### コミュニティスキルを手動でインストールする

GitHub リポジトリでスキルを見つけたら、そのフォルダをスキルディレクトリにコピーします：

```bash
# awesome-copilot リポジトリをクローン
git clone https://github.com/github/awesome-copilot.git /tmp/awesome-copilot

# 特定のスキルをプロジェクトにコピー
cp -r /tmp/awesome-copilot/skills/code-checklist .github/skills/

# すべてのプロジェクトで個人用に使う場合
cp -r /tmp/awesome-copilot/skills/code-checklist ~/.copilot/skills/
```

> ⚠️ **インストール前に確認**: プロジェクトにコピーする前に、必ずスキルの `SKILL.md` を読んでください。スキルは Copilot の動作を制御するため、悪意のあるスキルは有害なコマンドの実行や予期しないコード変更を指示する可能性があります。

---

# 実践練習

<img src="../images/practice.png" alt="Warm desk setup with monitor showing code, lamp, coffee cup, and headphones ready for hands-on practice" width="800"/>

学んだことを活かして、独自のスキルを構築・テストしましょう。

---

## ▶️ 自分で試してみよう

### さらにスキルを作成する

異なるパターンを示す2つのスキルを紹介します。先ほどの「最初のスキルを作成する」と同じ `mkdir` + `cat` のワークフローに従うか、スキルを適切な場所にコピー＆ペーストしてください。その他のサンプルは [.github/skills](../.github/skills) にあります。

### pytest テスト生成スキル

コードベース全体で一貫した pytest 構造を確保するスキルです：

```bash
mkdir -p .github/skills/pytest-gen

cat > .github/skills/pytest-gen/SKILL.md << 'EOF'
---
name: pytest-gen
description: Generate comprehensive pytest tests with fixtures and edge cases
---

# pytest Test Generation

Generate pytest tests that include:

## Test Structure
- Use pytest conventions (test_ prefix)
- One assertion per test when possible
- Clear test names describing expected behavior
- Use fixtures for setup/teardown

## Coverage
- Happy path scenarios
- Edge cases: None, empty strings, empty lists
- Boundary values
- Error scenarios with pytest.raises()

## Fixtures
- Use @pytest.fixture for reusable test data
- Use tmpdir/tmp_path for file operations
- Mock external dependencies with pytest-mock

## Output
Provide complete, runnable test file with proper imports.
EOF
```

### チーム PR レビュースキル

チーム全体で一貫した PR レビュー基準を適用するスキルです：

```bash
mkdir -p .github/skills/pr-review

cat > .github/skills/pr-review/SKILL.md << 'EOF'
---
name: pr-review
description: Team-standard PR review checklist
---

# PR Review

Review code changes against team standards:

## Security Checklist
- [ ] No hardcoded secrets or API keys
- [ ] Input validation on all user data
- [ ] No bare except clauses
- [ ] No sensitive data in logs

## Code Quality
- [ ] Functions under 50 lines
- [ ] No print statements in production code
- [ ] Type hints on public functions
- [ ] Context managers for file I/O
- [ ] No TODOs without issue references

## Testing
- [ ] New code has tests
- [ ] Edge cases covered
- [ ] No skipped tests without explanation

## Documentation
- [ ] API changes documented
- [ ] Breaking changes noted
- [ ] README updated if needed

## Output Format
Provide results as:
- ✅ PASS: Items that look good
- ⚠️ WARN: Items that could be improved
- ❌ FAIL: Items that must be fixed before merge
EOF
```

### さらに挑戦

1. **スキル作成チャレンジ**: 3項目のチェックリストを行う `quick-review` スキルを作成しましょう：
   - ベアな except 句（bare except clauses）
   - 型ヒントの欠落
   - 分かりにくい変数名

   テスト方法：「Do a quick review of books.py」と質問してください

2. **スキルの比較**: 手動で詳細なセキュリティレビュープロンプトを書く時間を計測してみましょう。次に「Check for security issues in this file」と質問して、security-audit スキルに自動的に読み込ませてください。スキルでどれだけ時間を節約できましたか？

3. **チームスキルチャレンジ**: チームのコードレビューチェックリストについて考えてみましょう。スキルとしてエンコードできますか？ スキルが常にチェックすべき項目を3つ書き出してください。

**セルフチェック**: `description` フィールドがなぜ重要なのかを説明できれば、スキルを理解しています（Copilot がスキルを読み込むかどうかを決定する手段だからです）。

---

## 📝 課題

### メインチャレンジ：ブックサマリースキルを作成する

上記の例では `pytest-gen` と `pr-review` スキルを作成しました。次は、まったく異なる種類のスキルの作成を実践しましょう：データからフォーマット済み出力を生成するスキルです。

1. 現在のスキルを一覧表示する：Copilot を実行して `/skills list` を渡してください。`ls .github/skills/` でプロジェクトのスキルを、`ls ~/.copilot/skills/` で個人スキルを確認することもできます。
2. `.github/skills/book-summary/SKILL.md` に `book-summary` スキルを作成し、ブックコレクションのフォーマット済み Markdown サマリーを生成するようにします
3. スキルには以下を含めてください：
   - 明確な name と description（description はマッチングに不可欠です！）
   - 具体的なフォーマット規則（例：タイトル、著者、出版年、読了状態を含む Markdown テーブル）
   - 出力規則（例：読了状態に ✅/❌ を使用、年で並び替え）
4. スキルをテストする：`@samples/book-app-project/data.json Summarize the books in this collection`
5. `/skills list` でスキルが自動トリガーされることを確認する
6. `/book-summary Summarize the books in this collection` で直接呼び出しを試す

**成功基準**: ブックコレクションについて質問したときに Copilot が自動的に適用する `book-summary` スキルが動作していること。

<details>
<summary>💡 ヒント（クリックで展開）</summary>

**スターターテンプレート**: `.github/skills/book-summary/SKILL.md` を作成してください：

```markdown
---
name: book-summary
description: Generate a formatted markdown summary of a book collection
---

# Book Summary Generator

Generate a summary of the book collection following these rules:

1. Output a markdown table with columns: Title, Author, Year, Status
2. Use ✅ for read books and ❌ for unread books
3. Sort by year (oldest first)
4. Include a total count at the bottom
5. Flag any data issues (missing authors, invalid years)

Example:
| Title | Author | Year | Status |
|-------|--------|------|--------|
| 1984 | George Orwell | 1949 | ✅ |
| Dune | Frank Herbert | 1965 | ❌ |

**Total: 2 books (1 read, 1 unread)**
```

**テストする：**
```bash
copilot
> @samples/book-app-project/data.json Summarize the books in this collection
# description のマッチングに基づいてスキルが自動トリガーされるはずです
```

**トリガーされない場合：** `/skills reload` を実行してから再度質問してください。

</details>

### ボーナスチャレンジ：コミットメッセージスキル

1. 一貫したフォーマットの Conventional Commit メッセージを生成する `commit-message` スキルを作成してください
2. 変更をステージングして「Generate a commit message for my staged changes」と質問してテストしてください
3. スキルをドキュメント化し、`copilot-skill` トピック付きで GitHub に共有してください

---

<details>
<summary>🔧 <strong>よくある間違いとトラブルシューティング</strong>（クリックで展開）</summary>

### よくある間違い

| 間違い | 何が起きるか | 修正方法 |
|---------|--------------|-----|
| ファイル名を `SKILL.md` 以外にする | スキルが認識されない | ファイル名は正確に `SKILL.md` でなければなりません |
| `description` フィールドが曖昧 | スキルが自動的に読み込まれない | description は主要な検出メカニズムです。具体的なトリガーワードを使用してください |
| frontmatter に `name` または `description` がない | スキルの読み込みに失敗する | YAML frontmatter に両方のフィールドを追加してください |
| フォルダの場所が間違っている | スキルが見つからない | `.github/skills/skill-name/`（プロジェクト）または `~/.copilot/skills/skill-name/`（個人用）を使用してください |

### トラブルシューティング

**スキルが使われない場合** - Copilot が期待どおりにスキルを使わない場合：

1. **description を確認する**: 質問の仕方と一致していますか？
   ```markdown
   # 悪い例: 曖昧すぎる
   description: Reviews code

   # 良い例: トリガーワードを含む
   description: Use for code reviews, checking code quality,
     finding bugs, security issues, and best practice violations
   ```

2. **ファイルの場所を確認する**：
   ```bash
   # プロジェクトスキル
   ls .github/skills/

   # ユーザースキル
   ls ~/.copilot/skills/
   ```

3. **SKILL.md の形式を確認する**: frontmatter は必須です：
   ```markdown
   ---
   name: skill-name
   description: What the skill does and when to use it
   ---

   # 指示をここに記述
   ```

**スキルが表示されない場合** - フォルダ構造を確認してください：
```
.github/skills/
└── my-skill/           # フォルダ名
    └── SKILL.md        # 正確に SKILL.md でなければなりません（大文字小文字区別あり）
```

スキルの作成や編集後は `/skills reload` を実行して、変更が反映されることを確認してください。

**スキルが読み込まれているかテストする** - Copilot に直接聞いてみましょう：
```bash
> What skills do you have available for checking code quality?
# Copilot が見つけた関連スキルを説明してくれます
```

**スキルが実際に動作しているかどうかを確認するには？**

1. **出力形式を確認する**: スキルが出力形式を指定している場合（`[CRITICAL]` タグなど）、レスポンスにそれが含まれているか確認してください
2. **直接聞く**: レスポンスを受け取った後、「Did you use any skills for that?」と聞いてください
3. **有効/無効の比較**: `--no-custom-instructions` を付けて同じプロンプトを試し、違いを確認してください：
   ```bash
   # スキルあり
   copilot --allow-all -p "Review @file.py for security issues"

   # スキルなし（ベースライン比較）
   copilot --allow-all -p "Review @file.py for security issues" --no-custom-instructions
   ```
4. **特定のチェック項目を確認する**: スキルに特定のチェック（「50行を超える関数」など）が含まれている場合、出力にそれが現れるか確認してください

</details>

---

# まとめ

## 🔑 重要ポイント

1. **スキルは自動的**: プロンプトがスキルの description に一致すると、Copilot が自動的に読み込みます
2. **直接呼び出し**: `/skill-name` のスラッシュコマンドでスキルを直接呼び出すこともできます
3. **SKILL.md の形式**: YAML frontmatter（name、description、オプションの license）に Markdown の指示を加えた構成です
4. **場所が重要**: `.github/skills/` はプロジェクト/チーム共有用、`~/.copilot/skills/` は個人用です
5. **description がカギ**: 普段の質問の仕方に一致する description を書きましょう

> 📋 **クイックリファレンス**: コマンドとショートカットの完全な一覧は [GitHub Copilot CLI command reference](https://docs.github.com/en/copilot/reference/cli-command-reference) を参照してください。

---

## ➡️ 次のステップ

スキルは、自動読み込みされる指示で Copilot の機能を拡張しました。しかし、外部サービスへの接続はどうでしょうか？ それが MCP の出番です。

**[Chapter 06: MCP サーバー](../06-mcp-servers/README.md)** では、以下を学びます：

- MCP（Model Context Protocol）とは何か
- GitHub、ファイルシステム、ドキュメントサービスへの接続
- MCP サーバーの設定方法
- マルチサーバーワークフロー

---

**[← Chapter 04 に戻る](../04-agents-custom-instructions/README.md)** | **[Chapter 06 へ進む →](../06-mcp-servers/README.md)**
