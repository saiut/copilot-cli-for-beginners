![Chapter 03: Development Workflows](images/chapter-header.png)

> **もし AI が、あなた自身が気づいていないバグまで見つけてくれるとしたら？**

この章では、GitHub Copilot CLI を日常的な開発ツールとして活用していきます。テスト、リファクタリング、デバッグ、Git など、毎日使っているワークフローの中で Copilot CLI を使いこなしましょう。

## 🎯 学習目標

この章を終えると、以下のことができるようになります：

- Copilot CLI を使った包括的なコードレビューの実行
- レガシーコードの安全なリファクタリング
- AI を活用したデバッグ
- テストの自動生成
- Copilot CLI と git ワークフローの連携

> ⏱️ **所要時間の目安**: 約60分（読み物15分 + ハンズオン45分）

---

## 🧩 身近なたとえ：大工のワークフロー

大工は道具の使い方を知っているだけではなく、作業ごとに*ワークフロー*を持っています：

<img src="images/carpenter-workflow-steps.png" alt="Craftsman workshop showing three workflow lanes: Building Furniture (Measure, Cut, Assemble, Finish), Fixing Damage (Assess, Remove, Repair, Match), and Quality Check (Inspect, Test Joints, Check Alignment)" width="800"/>

開発者も同じように、タスクごとにワークフローがあります。GitHub Copilot CLI はこれらのワークフローをそれぞれ強化し、日々の開発作業をより効率的かつ効果的にしてくれます。

---

# 5つのワークフロー

<img src="images/five-workflows.png" alt="Five glowing neon icons representing code review, testing, debugging, refactoring, and git integration workflows" width="800"/>

各ワークフローは独立しています。今の自分に合ったものを選んでもいいですし、全部通して取り組んでもかまいません。

---

## 自分に合ったワークフローを選ぼう

この章では、開発者が日常的に使う5つのワークフローを取り上げます。**ただし、一度に全部読む必要はありません！** 各ワークフローは下の折りたたみセクションで独立しています。今の自分やプロジェクトに合ったものを選んでください。残りはいつでも戻って確認できます。

<img src="images/five-workflows-swimlane.png" alt="Five Development Workflows: Code Review, Refactoring, Debugging, Test Generation, and Git Integration shown as horizontal swimlanes" width="800"/>

| やりたいこと | ジャンプ先 |
|---|---|
| マージ前にコードをレビューしたい | [ワークフロー 1: コードレビュー](#workflow-1-code-review) |
| 散らかったコードやレガシーコードを整理したい | [ワークフロー 2: リファクタリング](#workflow-2-refactoring) |
| バグを追跡して修正したい | [ワークフロー 3: デバッグ](#workflow-3-debugging) |
| コードのテストを生成したい | [ワークフロー 4: テスト生成](#workflow-4-test-generation) |
| より良いコミットやPRを書きたい | [ワークフロー 5: Git 連携](#workflow-5-git-integration) |
| コーディング前にリサーチしたい | [クイックヒント：計画やコーディングの前にリサーチ](#quick-tip-research-before-you-plan-or-code) |
| バグ修正ワークフローの全体像を見たい | [すべてを組み合わせる](#putting-it-all-together-bug-fix-workflow) |

**下のワークフローをクリックして展開し**、GitHub Copilot CLI がその分野の開発プロセスをどのように強化できるかを確認しましょう。

---

<a id="workflow-1-code-review"></a>
<details>
<summary><strong>ワークフロー 1: コードレビュー</strong> - ファイルのレビュー、/review エージェントの使用、重要度別チェックリストの作成</summary>

<img src="images/code-review-swimlane-single.png" alt="Code review workflow: review, identify issues, prioritize, generate checklist." width="800"/>

### 基本的なレビュー

この例では `@` シンボルを使ってファイルを参照し、Copilot CLI がその内容に直接アクセスしてレビューできるようにします。

```bash
copilot

> Review @samples/book-app-project/book_app.py for code quality
```

---

<details>
<summary>🎬 実際に見てみよう！</summary>

![Code Review Demo](images/code-review-demo.gif)

*デモの出力は実行ごとに異なります。使用するモデル、ツール、レスポンスはここに表示されているものと異なる場合があります。*

</details>

---

### 入力バリデーションのレビュー

Copilot CLI に特定の観点（ここでは入力バリデーション）に焦点を当てたレビューを依頼するには、気になるカテゴリをプロンプトに列挙します。

```text
copilot

> Review @samples/book-app-project/utils.py for input validation issues. Check for: missing validation, error handling gaps, and edge cases
```


### 複数ファイルのプロジェクトレビュー

`@` でディレクトリ全体を参照すると、Copilot CLI がプロジェクト内のすべてのファイルを一度にスキャンできます。

```bash
copilot

> @samples/book-app-project/ Review this entire project. Create a markdown checklist of issues found, categorized by severity
```

### インタラクティブなコードレビュー

マルチターン会話を使って、より深く掘り下げます。まず大まかなレビューから始め、セッションを再開せずにフォローアップの質問をしましょう。

```bash
copilot

> @samples/book-app-project/book_app.py Review this file for:
> - Input validation
> - Error handling
> - Code style and best practices

# Copilot CLI が詳細なレビューを提供

> The user input handling - are there any edge cases I'm missing?

# Copilot CLI が空文字列や特殊文字などの潜在的な問題を表示

> Create a checklist of all issues found, prioritized by severity

# Copilot CLI が優先度付きのアクション項目を生成
```

### レビューチェックリストのテンプレート

Copilot CLI に出力を特定のフォーマットで構造化するよう依頼します（ここでは、Issue に貼り付けられる重要度別のマークダウンチェックリスト）。

```bash
copilot

> Review @samples/book-app-project/ and create a markdown checklist of issues found, categorized by:
> - Critical (data loss risks, crashes)
> - High (bugs, incorrect behavior)
> - Medium (performance, maintainability)
> - Low (style, minor improvements)
```

### Git の変更を理解する（/review を使う前に重要）

`/review` コマンドを使う前に、git における2種類の変更を理解しておく必要があります：

| 変更の種類 | 意味 | 確認方法 |
|-------------|---------------|------------|
| **ステージ済みの変更** | `git add` で次のコミットに含めたファイル | `git diff --staged` |
| **未ステージの変更** | 変更したがまだ add していないファイル | `git diff` |

```bash
# クイックリファレンス
git status           # ステージ済みと未ステージの両方を表示
git add file.py      # ファイルをステージ（コミット対象に追加）
git diff             # 未ステージの変更を表示
git diff --staged    # ステージ済みの変更を表示
```

### /review コマンドの使い方

`/review` コマンドは、組み込みの**コードレビューエージェント**を起動します。このエージェントは、ステージ済みおよび未ステージの変更を、ノイズの少ない高品質な出力で分析するように最適化されています。自由形式のプロンプトを書く代わりに、スラッシュコマンドで特化型の組み込みエージェントを呼び出しましょう。

```bash
copilot

> /review
# ステージ済み・未ステージの変更に対してコードレビューエージェントを起動
# 焦点を絞った実用的なフィードバックを提供

> /review Check for security issues in authentication
# 特定の観点に絞ったレビューを実行
```

> 💡 **ヒント**: コードレビューエージェントは、保留中の変更があるときに最も効果を発揮します。より焦点を絞ったレビューのために、`git add` でファイルをステージしておきましょう。

</details>

---

<a id="workflow-2-refactoring"></a>
<details>
<summary><strong>ワークフロー 2: リファクタリング</strong> - コードの再構成、関心の分離、エラーハンドリングの改善</summary>

<img src="images/refactoring-swimlane-single.png" alt="Refactoring workflow: assess code, plan changes, implement, verify behavior." width="800"/>

### シンプルなリファクタリング

> **まずはこれを試してみよう：** `@samples/book-app-project/book_app.py The command handling uses if/elif chains. Refactor it to use a dictionary dispatch pattern.`

簡単な改善から始めましょう。これらのプロンプトをブックアプリで試してみてください。各プロンプトは `@` ファイル参照と具体的なリファクタリング指示を組み合わせることで、Copilot CLI が何を変更すべきか正確に把握できるようにしています。

```bash
copilot

> @samples/book-app-project/book_app.py The command handling uses if/elif chains. Refactor it to use a dictionary dispatch pattern.

> @samples/book-app-project/utils.py Add type hints to all functions

> @samples/book-app-project/book_app.py Extract the book display logic into utils.py for better separation of concerns
```

> 💡 **リファクタリングが初めての方へ：** 複雑な変換に取り組む前に、型ヒントの追加や変数名の改善など、簡単なリクエストから始めてみましょう。

---

<details>
<summary>🎬 実際に見てみよう！</summary>

![Refactor Demo](images/refactor-demo.gif)

*デモの出力は実行ごとに異なります。使用するモデル、ツール、レスポンスはここに表示されているものと異なる場合があります。*

</details>

---

### 関心の分離

1つのプロンプトで複数のファイルを `@` で参照すると、Copilot CLI がリファクタリングの一環としてファイル間でコードを移動できます。

```bash
copilot

> @samples/book-app-project/utils.py @samples/book-app-project/book_app.py
> The utils.py file has print statements mixed with logic. Refactor to separate display functions from data processing.
```

### エラーハンドリングの改善

関連する2つのファイルを提供し、横断的な関心事を説明することで、Copilot CLI が両方のファイルにわたって一貫した修正を提案できるようにします。

```bash
copilot

> @samples/book-app-project/utils.py @samples/book-app-project/books.py
> These files have inconsistent error handling. Suggest a unified approach using custom exceptions.
```

### ドキュメントの追加

各 docstring に含めるべき内容を詳細なリストで指定します。

```bash
copilot

> @samples/book-app-project/books.py Add comprehensive docstrings to all methods:
> - Include parameter types and descriptions
> - Document return values
> - Note any exceptions raised
> - Add usage examples
```

### テスト付きの安全なリファクタリング

マルチターン会話で2つの関連リクエストを連続して行います。まずテストを生成し、それをセーフティネットとしてリファクタリングを行います。

```bash
copilot

> @samples/book-app-project/books.py Before refactoring, generate tests for current behavior

# まずテストを取得

> Now refactor the BookCollection class to use a context manager for file operations

# 安心してリファクタリング - テストで動作が保持されていることを検証
```

</details>

---

<a id="workflow-3-debugging"></a>
<details>
<summary><strong>ワークフロー 3: デバッグ</strong> - バグの追跡、セキュリティ監査、複数ファイルにわたる問題のトレース</summary>

<img src="images/debugging-swimlane-single.png" alt="Debugging workflow: understand error, locate root cause, fix, test." width="800"/>

### シンプルなデバッグ

> **まずはこれを試してみよう：** `@samples/book-app-buggy/books_buggy.py Users report that searching for "The Hobbit" returns no results even though it's in the data. Debug why.`

まず、何が問題なのかを説明しましょう。バグ入りブックアプリで試せる一般的なデバッグパターンを紹介します。各プロンプトは `@` ファイル参照と明確な症状の説明を組み合わせることで、Copilot CLI がバグを特定して診断できるようにしています。

```bash
copilot

# パターン：「X を期待したが Y になった」
> @samples/book-app-buggy/books_buggy.py Users report that searching for "The Hobbit" returns no results even though it's in the data. Debug why.

# パターン：「想定外の動作」
> @samples/book-app-buggy/book_app_buggy.py When I remove a book that doesn't exist, the app says it was removed. Help me find why.

# パターン：「間違った結果」
> @samples/book-app-buggy/books_buggy.py When I mark one book as read, ALL books get marked. What's the bug?
```

> 💡 **デバッグのコツ**: *症状*（何が起きているか）と*期待値*（本来どうなるべきか）を説明しましょう。Copilot CLI が残りを見つけてくれます。

---

<details>
<summary>🎬 実際に見てみよう！</summary>

![Fix Bug Demo](images/fix-bug-demo.gif)

*デモの出力は実行ごとに異なります。使用するモデル、ツール、レスポンスはここに表示されているものと異なる場合があります。*

</details>

---

### 「バグ探偵」 - AI が関連するバグも発見

ここがコンテキストを活用したデバッグの真骨頂です。バグ入りブックアプリでこのシナリオを試してみましょう。`@` でファイル全体を提供し、ユーザーから報告された症状だけを説明します。Copilot CLI が根本原因をたどり、近くにある追加のバグも見つけることがあります。

```bash
copilot

> @samples/book-app-buggy/books_buggy.py
>
> Users report: "Finding books by author name doesn't work for partial names"
> Debug why this happens
```

**Copilot CLI の出力**:
```
Root Cause: Line 80 uses exact match (==) instead of partial match (in).

Line 80: return [b for b in self.books if b.author == author]

The find_by_author function requires an exact match. Searching for "Tolkien"
won't find books by "J.R.R. Tolkien".

Fix: Change to case-insensitive partial match:
return [b for b in self.books if author.lower() in b.author.lower()]
```

**なぜこれが重要か**: Copilot CLI はファイル全体を読み取り、バグレポートのコンテキストを理解し、明確な説明とともに具体的な修正を提供してくれます。

> 💡 **おまけ**: Copilot CLI はファイル全体を分析するため、聞いていない*別の*問題も見つけることがよくあります。たとえば、著者検索を修正している最中に、`find_book_by_title` の大文字小文字の区別に関するバグも指摘するかもしれません！

### セキュリティに関する実践的な補足

自分のコードのデバッグは重要ですが、本番アプリケーションにおけるセキュリティの脆弱性を理解することも不可欠です。この例を試してください：Copilot CLI に見慣れないファイルを渡して、セキュリティの問題を監査してもらいます。

```bash
copilot

> @samples/buggy-code/python/user_service.py Find all security vulnerabilities in this Python user service
```

このファイルには、本番アプリで実際に遭遇する現実的なセキュリティパターンが含まれています。

> 💡 **よく出てくるセキュリティ用語：**
> - **SQL インジェクション**: ユーザー入力がデータベースのクエリに直接挿入され、攻撃者が悪意のあるコマンドを実行できてしまうこと
> - **パラメータ化クエリ**: 安全な代替手段 - プレースホルダ（`?`）でユーザーデータと SQL コマンドを分離する方法
> - **レースコンディション**: 2つの操作が同時に発生し、互いに干渉してしまうこと
> - **XSS（クロスサイトスクリプティング）**: 攻撃者が Web ページに悪意のあるスクリプトを注入すること

---

### エラーの理解

スタックトレースをプロンプトに直接貼り付け、`@` ファイル参照と一緒に渡すことで、Copilot CLI がエラーをソースコードにマッピングできます。

```bash
copilot

> I'm getting this error:
> AttributeError: 'NoneType' object has no attribute 'title'
>     at show_books (book_app.py:19)
>
> @samples/book-app-project/book_app.py Explain why and how to fix it
```

### テストケースを使ったデバッグ

正確な入力と観察された出力を記述して、Copilot CLI に具体的で再現可能なテストケースを提供します。

```bash
copilot

> @samples/book-app-buggy/books_buggy.py The remove_book function has a bug. When I try to remove "Dune",
> it also removes "Dune Messiah". Debug this: explain the root cause and provide a fix.
```

### コード全体を通じた問題のトレース

複数のファイルを参照し、Copilot CLI にファイル間のデータフローを追跡させて、問題の発生箇所を特定します。

```bash
copilot

> Users report that the book list numbering starts at 0 instead of 1.
> @samples/book-app-buggy/book_app_buggy.py @samples/book-app-buggy/books_buggy.py
> Trace through the list display flow and identify where the issue occurs
```

### データの問題を理解する

データファイルとそれを読み込むコードを一緒に提供することで、Copilot CLI がエラーハンドリングの改善を提案する際に全体像を把握できます。

```bash
copilot

> @samples/book-app-project/data.json @samples/book-app-project/books.py
> Sometimes the JSON file gets corrupted and the app crashes. How should we handle this gracefully?
```

</details>

---

<a id="workflow-4-test-generation"></a>
<details>
<summary><strong>ワークフロー 4: テスト生成</strong> - 包括的なテストとエッジケースを自動的に生成</summary>

<img src="images/test-gen-swimlane-single.png" alt="Test Generation workflow: analyze function, generate tests, include edge cases, run." width="800"/>

> **まずはこれを試してみよう：** `@samples/book-app-project/books.py Generate pytest tests for all functions including edge cases`

### 「テストの爆発」 - 2件のテスト vs 15件以上のテスト

手作業でテストを書く場合、通常は2〜3つの基本的なテストを作成します：
- 正常な入力のテスト
- 不正な入力のテスト
- エッジケースのテスト

Copilot CLI に包括的なテストを生成させると何が起きるか見てみましょう！このプロンプトは、`@` ファイル参照と構造化されたリストを使って、Copilot CLI を徹底的なテストカバレッジに導きます：

```bash
copilot

> @samples/book-app-project/books.py Generate comprehensive pytest tests. Include tests for:
> - Adding books
> - Removing books
> - Finding by title
> - Finding by author
> - Marking as read
> - Edge cases with empty data
```

---

<details>
<summary>🎬 実際に見てみよう！</summary>

![Test Generation Demo](images/test-gen-demo.gif)

*デモの出力は実行ごとに異なります。使用するモデル、ツール、レスポンスはここに表示されているものと異なる場合があります。*

</details>

---

**結果**: 以下を含む15件以上の包括的なテストが得られます：

```python
class TestBookCollection:
    # 正常系
    def test_add_book_creates_new_book(self):
        ...
    def test_list_books_returns_all_books(self):
        ...

    # 検索操作
    def test_find_book_by_title_case_insensitive(self):
        ...
    def test_find_book_by_title_returns_none_when_not_found(self):
        ...
    def test_find_by_author_partial_match(self):
        ...
    def test_find_by_author_case_insensitive(self):
        ...

    # エッジケース
    def test_add_book_with_empty_title(self):
        ...
    def test_remove_nonexistent_book(self):
        ...
    def test_mark_as_read_nonexistent_book(self):
        ...

    # データの永続化
    def test_save_books_persists_to_json(self):
        ...
    def test_load_books_handles_missing_file(self):
        ...
    def test_load_books_handles_corrupted_json(self):
        ...

    # 特殊文字
    def test_add_book_with_unicode_characters(self):
        ...
    def test_find_by_author_with_special_characters(self):
        ...
```

**結果**: 30秒で、考え抜いて書くのに1時間はかかるようなエッジケースのテストが手に入ります。

---

### ユニットテスト

単一の関数をターゲットにし、テストしたい入力カテゴリを列挙して、Copilot CLI に焦点を絞った綿密なユニットテストを生成させます。

```bash
copilot

> @samples/book-app-project/utils.py Generate comprehensive pytest tests for get_book_details covering:
> - Valid input
> - Empty strings
> - Invalid year formats
> - Very long titles
> - Special characters in author names
```

### テストの実行

ツールチェーンについて平易な言葉で質問しましょう。Copilot CLI が適切なシェルコマンドを生成してくれます。

```bash
copilot

> How do I run the tests? Show me the pytest command.

# Copilot CLI の応答：
# cd samples/book-app-project && python -m pytest tests/
# 詳細出力の場合: python -m pytest tests/ -v
# print文を表示する場合: python -m pytest tests/ -s
```

### 特定のシナリオのテスト

Copilot CLI が正常系を超えたテストを生成するよう、高度なシナリオやトリッキーなケースをリストアップします。

```bash
copilot

> @samples/book-app-project/books.py Generate tests for these scenarios:
> - Adding duplicate books (same title and author)
> - Removing a book by partial title match
> - Finding books when collection is empty
> - File permission errors during save
> - Concurrent access to the book collection
```

### 既存ファイルへのテスト追加

単一の関数に対する*追加の*テストを依頼して、既存のテストを補完する新しいケースを Copilot CLI に生成させます。

```bash
copilot

> @samples/book-app-project/books.py
> Generate additional tests for the find_by_author function with edge cases:
> - Author name with hyphens (e.g., "Jean-Paul Sartre")
> - Author with multiple first names
> - Empty string as author
> - Author name with accented characters
```

</details>

---

<a id="workflow-5-git-integration"></a>
<details>
<summary><strong>ワークフロー 5: Git 連携</strong> - コミットメッセージ、PR の説明文、/delegate、/diff</summary>

<img src="images/git-integration-swimlane-single.png" alt="Git Integration workflow: stage changes, generate message, commit, create PR." width="800"/>

> 💡 **このワークフローは git の基本操作**（ステージング、コミット、ブランチ）を前提としています。git が初めての方は、先に他の4つのワークフローを試してみてください。

### コミットメッセージの生成

> **まずはこれを試してみよう：** `copilot -p "Generate a conventional commit message for: $(git diff --staged)"` — 変更をステージしてからこれを実行すると、Copilot CLI がコミットメッセージを書いてくれます。

この例では、`-p` インラインプロンプトフラグとシェルのコマンド置換を使って、`git diff` の出力を直接 Copilot CLI にパイプし、ワンショットでコミットメッセージを生成します。`$(...)` 構文は括弧内のコマンドを実行し、その出力を外側のコマンドに挿入します。

```bash

# 何が変わったか確認
git diff --staged

# [Conventional Commit](../GLOSSARY.md#conventional-commit) 形式でコミットメッセージを生成
# （"feat(books): add search" や "fix(data): handle empty input" のような構造化メッセージ）
copilot -p "Generate a conventional commit message for: $(git diff --staged)"

# 出力: "feat(books): add partial author name search
#
# - Update find_by_author to support partial matches
# - Add case-insensitive comparison
# - Improve user experience when searching authors"
```

---

<details>
<summary>🎬 実際に見てみよう！</summary>

![Git Integration Demo](images/git-integration-demo.gif)

*デモの出力は実行ごとに異なります。使用するモデル、ツール、レスポンスはここに表示されているものと異なる場合があります。*

</details>

---

### 変更の説明

`git show` の出力を `-p` プロンプトにパイプして、最新のコミットの内容を平易な言葉で要約してもらいます。

```bash
# このコミットでは何が変わった？
copilot -p "Explain what this commit does: $(git show HEAD --stat)"
```

### PR の説明文

`git log` の出力と構造化されたプロンプトテンプレートを組み合わせて、完全なプルリクエストの説明文を自動生成します。

```bash
# ブランチの変更からPR説明文を生成
copilot -p "Generate a pull request description for these changes:
$(git log main..HEAD --oneline)

Include:
- Summary of changes
- Why these changes were made
- Testing done
- Breaking changes? (yes/no)"
```

### プッシュ前のレビュー

`git diff main..HEAD` を `-p` プロンプト内で使い、プッシュ前にブランチ全体の変更をすばやくチェックします。

```bash
# プッシュ前の最終チェック
copilot -p "Review these changes for issues before I push:
$(git diff main..HEAD)"
```

### /delegate でバックグラウンドタスクを実行

`/delegate` コマンドは、作業を GitHub 上の Copilot コーディングエージェントに引き渡します。`/delegate` スラッシュコマンド（または `&` ショートカット）を使って、明確に定義されたタスクをバックグラウンドエージェントに任せましょう。

```bash
copilot

> /delegate Add input validation to the login form

# または & プレフィックスのショートカットを使用：
> & Fix the typo in the README header

# Copilot CLI の動作：
# 1. 変更を新しいブランチにコミット
# 2. ドラフトプルリクエストを作成
# 3. GitHub 上でバックグラウンドで作業
# 4. 完了したらレビューをリクエスト
```

これは、他の作業に集中している間に完了させたい、明確に定義されたタスクに最適です。

### /diff でセッション中の変更をレビュー

`/diff` コマンドは、現在のセッション中に行われたすべての変更を表示します。コミット前に Copilot CLI が変更したすべての内容をビジュアル diff で確認するには、このスラッシュコマンドを使いましょう。

```bash
copilot

# いくつか変更を加えた後...
> /diff

# このセッションで変更されたすべてのファイルのビジュアル diff を表示
# コミット前のレビューに最適
```

</details>

---

## クイックヒント：計画やコーディングの前にリサーチ

ライブラリの調査、ベストプラクティスの理解、馴染みのないトピックの探索が必要なとき、コードを書く前に `/research` を使って深いリサーチ調査を実行しましょう：

```bash
copilot

> /research What are the best Python libraries for validating user input in CLI apps?
```

Copilot は GitHub リポジトリやウェブソースを検索し、参考文献付きの要約を返します。新しい機能の開発に取りかかる前に、情報に基づいた判断をしたい場合に便利です。結果は `/share` で共有できます。

> 💡 **ヒント**: `/research` は `/plan` の*前に*使うと効果的です。まずアプローチをリサーチし、その後で実装を計画しましょう。

---

## すべてを組み合わせる：バグ修正ワークフロー

報告されたバグを修正するための完全なワークフローはこちらです：

```bash

# 1. バグ報告を理解する
copilot

> Users report: 'Finding books by author name doesn't work for partial names'
> @samples/book-app-project/books.py Analyze and identify the likely cause

# 2. 問題をデバッグする（同じセッションで続行）
> Based on the analysis, show me the find_by_author function and explain the issue

> Fix the find_by_author function to handle partial name matches

# 3. 修正に対するテストを生成する
> @samples/book-app-project/books.py Generate pytest tests specifically for:
> - Full author name match
> - Partial author name match
> - Case-insensitive matching
> - Author name not found

# 4. コミットメッセージを生成する
copilot -p "Generate commit message for: $(git diff --staged)"

# 出力: "fix(books): support partial author name search"
```

### バグ修正ワークフローのまとめ

| ステップ | アクション | Copilot コマンド |
|------|--------|-----------------|
| 1 | バグを理解する | `> [バグの説明] @relevant-file.py Analyze the likely cause` |
| 2 | 詳細な分析を取得 | `> Show me the function and explain the issue` |
| 3 | 修正を実装する | `> Fix the [具体的な問題]` |
| 4 | テストを生成する | `> Generate tests for [具体的なシナリオ]` |
| 5 | コミットする | `copilot -p "Generate commit message for: $(git diff --staged)"` |

---

# 実践練習

<img src="../images/practice.png" alt="Warm desk setup with monitor showing code, lamp, coffee cup, and headphones ready for hands-on practice" width="800"/>

さあ、これらのワークフローを実際に使ってみましょう。

---

## ▶️ 自分で試してみよう

デモを完了したら、次のバリエーションを試してみましょう：

1. **バグ探偵チャレンジ**: `samples/book-app-buggy/books_buggy.py` の `mark_as_read` 関数を Copilot CLI にデバッグさせてください。なぜ1冊だけでなくすべての本が既読になるのか、説明してくれましたか？

2. **テストチャレンジ**: ブックアプリの `add_book` 関数のテストを生成してください。あなた自身では思いつかなかったであろうエッジケースが、Copilot CLI によっていくつ含まれているか数えてみましょう。

3. **コミットメッセージチャレンジ**: ブックアプリのファイルに何か小さな変更を加え、ステージし（`git add .`）、次のコマンドを実行してください：
   ```bash
   copilot -p "Generate a conventional commit message for: $(git diff --staged)"
   ```
   生成されたメッセージは、あなたがさっと書くものより良いですか？

**セルフチェック**: 「このバグをデバッグして」が「バグを探して」より強力である理由（コンテキストが重要！）を説明できれば、開発ワークフローを理解できています。

---

## 📝 課題

### メインチャレンジ：リファクタリング、テスト、そしてリリース

ハンズオンの例では `find_book_by_title` とコードレビューに焦点を当てました。ここでは、`book-app-project` の別の関数で同じワークフロースキルを練習しましょう：

1. **レビュー**: `books.py` の `remove_book()` のエッジケースや潜在的な問題を Copilot CLI にレビューしてもらいましょう：
   `@samples/book-app-project/books.py Review the remove_book() function. What happens if the title partially matches another book (e.g., "Dune" vs "Dune Messiah")? Are there any edge cases not handled?`
2. **リファクタリング**: 大文字小文字を区別しないマッチングや、本が見つからない場合に有用なフィードバックを返すなど、エッジケースに対応するよう `remove_book()` の改善を Copilot CLI に依頼しましょう
3. **テスト**: 改善した `remove_book()` 関数に特化した pytest テストを生成しましょう。以下をカバーします：
   - 存在する本の削除
   - 大文字小文字を区別しないタイトルマッチング
   - 存在しない本は適切なフィードバックを返す
   - 空のコレクションからの削除
4. **レビュー**: 変更をステージし、`/review` を実行して残りの問題をチェックしましょう
5. **コミット**: Conventional Commit メッセージを生成しましょう：
   `copilot -p "Generate a conventional commit message for: $(git diff --staged)"`

<details>
<summary>💡 ヒント（クリックで展開）</summary>

**各ステップのサンプルプロンプト：**

```bash
copilot

# ステップ 1: レビュー
> @samples/book-app-project/books.py Review the remove_book() function. What edge cases are not handled?

# ステップ 2: リファクタリング
> Improve remove_book() to use case-insensitive matching and return a clear message when the book isn't found. Show me the before and after code.

# ステップ 3: テスト
> Generate pytest tests for the improved remove_book() function, including:
> - Removing a book that exists
> - Case-insensitive matching ("dune" should remove "Dune")
> - Book not found returns appropriate response
> - Removing from an empty collection

# ステップ 4: レビュー
> /review

# ステップ 5: コミット
> Generate a conventional commit message for this refactor
```

**ヒント:** `remove_book()` を改善した後、Copilot CLI に「Are there any other functions in this file that could benefit from the same improvements?」と聞いてみましょう。`find_book_by_title()` や `find_by_author()` にも同様の改善を提案してくれるかもしれません。

</details>

### ボーナスチャレンジ：Copilot CLI でアプリケーションを作成する

> 💡 **注意**: この GitHub Skills の演習では、Python ではなく **Node.js** を使用します。ここで練習する GitHub Copilot CLI のテクニック（Issue の作成、コードの生成、ターミナルからのコラボレーション）は、どの言語にも適用できます。

この演習では、GitHub Copilot CLI を使って Issue の作成、コードの生成、ターミナルからのコラボレーションを行いながら、Node.js の電卓アプリを構築する方法を学びます。CLI のインストール、テンプレートやエージェントの使用、反復的なコマンドライン駆動の開発を練習します。

##### <img src="../images/github-skills-logo.png" width="28" align="center" /> [「Copilot CLI でアプリケーションを作成する」Skills 演習を始める](https://github.com/skills/create-applications-with-the-copilot-cli)

---

<details>
<summary>🔧 <strong>よくある間違いとトラブルシューティング</strong>（クリックで展開）</summary>

### よくある間違い

| 間違い | 何が起きるか | 対処法 |
|---------|--------------|-----|
| 「このコードをレビューして」のような曖昧なプロンプト | 具体的な問題を見逃す汎用的なフィードバック | 具体的に: 「SQL インジェクション、XSS、認証の問題をレビューして」 |
| コードレビューに `/review` を使わない | 最適化されたコードレビューエージェントを活用できない | ノイズの少ない高品質な出力に最適化された `/review` を使いましょう |
| コンテキストなしで「バグを探して」と依頼する | Copilot CLI はどのバグを経験しているかわからない | 症状を説明する: 「ユーザーが Y をすると X が起きると報告している」 |
| フレームワークを指定せずにテストを生成 | 間違った構文やアサーションライブラリのテストになる可能性 | 指定する: 「Jest を使ってテストを生成して」や「pytest を使って」 |

### トラブルシューティング

**レビューが不完全に見える場合** - 何を確認すべきかをより具体的にしましょう：

```bash
copilot

# こうではなく：
> Review @samples/book-app-project/book_app.py

# こうしましょう：
> Review @samples/book-app-project/book_app.py for input validation, error handling, and edge cases
```

**テストが自分のフレームワークと合わない場合** - フレームワークを指定しましょう：

```bash
copilot

> @samples/book-app-project/books.py Generate tests using pytest (not unittest)
```

**リファクタリングで動作が変わってしまう場合** - 動作の保持を Copilot CLI に依頼しましょう：

```bash
copilot

> @samples/book-app-project/book_app.py Refactor command handling to use dictionary dispatch. IMPORTANT: Maintain identical external behavior - no breaking changes
```

</details>

---

# まとめ

## 🔑 重要ポイント

<img src="images/specialized-workflows.png" alt="Specialized Workflows for Every Task: Code Review, Refactoring, Debugging, Testing, and Git Integration" width="800"/>

1. **コードレビュー**は具体的なプロンプトで包括的になります
2. **リファクタリング**は先にテストを生成するとより安全になります
3. **デバッグ**はエラーとコードの両方を Copilot CLI に見せることで効果を発揮します
4. **テスト生成**にはエッジケースとエラーシナリオを含めるべきです
5. **Git 連携**でコミットメッセージや PR の説明文を自動化できます

> 📋 **クイックリファレンス**: コマンドとショートカットの完全な一覧は [GitHub Copilot CLI コマンドリファレンス](https://docs.github.com/en/copilot/reference/cli-command-reference) をご覧ください。

---

## ✅ チェックポイント：基本をマスターしました

**おめでとうございます！** GitHub Copilot CLI を使って生産的に作業するためのコアスキルがすべて身につきました：

| スキル | 章 | できるようになったこと |
|-------|---------|----------------|
| 基本コマンド | Ch 01 | インタラクティブモード、プランモード、プログラマティックモード（-p）、スラッシュコマンドの使用 |
| コンテキスト | Ch 02 | `@` でファイルを参照、セッション管理、コンテキストウィンドウの理解 |
| ワークフロー | Ch 03 | コードレビュー、リファクタリング、デバッグ、テスト生成、git との連携 |

第04〜06章では、さらなるパワーを追加する機能を扱います。学ぶ価値は十分あります。

---

## 🛠️ 自分だけのワークフローを構築する

GitHub Copilot CLI の「正しい」使い方は1つではありません。自分なりのパターンを開発する際のヒントをいくつか紹介します：

> 📚 **公式ドキュメント**: GitHub 推奨のワークフローとヒントについては [Copilot CLI ベストプラクティス](https://docs.github.com/copilot/how-tos/copilot-cli/cli-best-practices) をご覧ください。

- **重要なことには `/plan` から始めましょう。** 実行前にプランを洗練させましょう - 良いプランはより良い結果につながります。
- **うまくいったプロンプトを保存しましょう。** Copilot CLI がミスをしたときは、何が問題だったかメモしましょう。時間とともに、これが自分だけのプレイブックになります。
- **自由に実験しましょう。** 長くて詳細なプロンプトを好む開発者もいれば、短いプロンプトにフォローアップを重ねる開発者もいます。さまざまなアプローチを試して、自分にとって自然な方法を見つけましょう。

> 💡 **次の内容**: 第04章と第05章では、ベストプラクティスをカスタムインストラクションやスキルとして体系化し、Copilot CLI に自動的に読み込ませる方法を学びます。

---

## ➡️ 次のステップ

残りの章では、Copilot CLI の機能を拡張する追加機能を扱います：

| 章 | 内容 | こんなときに使いたくなる |
|---------|----------------|---------------------|
| Ch 04: エージェント | 特化型 AI ペルソナの作成 | ドメインエキスパート（フロントエンド、セキュリティ）が欲しいとき |
| Ch 05: スキル | タスク用インストラクションの自動読み込み | 同じプロンプトを繰り返し使うとき |
| Ch 06: MCP | 外部サービスとの接続 | GitHub やデータベースからのライブデータが必要なとき |

**おすすめ**: まず1週間コアワークフローを使ってみて、具体的なニーズが出てきたら第04〜06章に戻りましょう。

---

## 追加トピックに進む

**[第04章：エージェントとカスタムインストラクション](../04-agents-custom-instructions/README.md)** では、以下を学びます：

- 組み込みエージェント（`/plan`、`/review`）の使い方
- `.agent.md` ファイルによる特化型エージェント（フロントエンドエキスパート、セキュリティ監査人）の作成
- マルチエージェントコラボレーションパターン
- プロジェクト標準のためのカスタムインストラクションファイル

---

**[← 第02章に戻る](../02-context-conversations/README.md)** | **[第04章に進む →](../04-agents-custom-instructions/README.md)**
