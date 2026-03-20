![Chapter 02: Context and Conversations](images/chapter-header.png)

> **もし AI が1ファイルだけでなく、コードベース全体を見渡せるとしたら？**

この章では、GitHub Copilot CLI の真の力である「コンテキスト」を解き放ちます。`@` 構文を使ってファイルやディレクトリを参照し、Copilot CLI にコードベースの深い理解を与える方法を学びます。セッションをまたいで会話を維持し、数日後に中断したところから正確に再開する方法、そしてクロスファイル分析が単一ファイルのレビューでは見落とすバグをどのように検出するかを見ていきます。

## 🎯 学習目標

この章を終えると、以下のことができるようになります：

- `@` 構文を使ってファイル、ディレクトリ、画像を参照する
- `--resume` と `--continue` で以前のセッションを再開する
- [context window](../GLOSSARY.md#context-window) の仕組みを理解する
- 効果的なマルチターン会話を書く
- マルチプロジェクトワークフローのためのディレクトリ権限を管理する

> ⏱️ **所要時間の目安**: 約50分（読み物20分 + ハンズオン30分）

---

## 🧩 現実世界のたとえ：同僚との仕事

<img src="images/colleague-context-analogy.png" alt="Context Makes the Difference - Without vs With Context" width="800"/>

*同僚と同じように、Copilot CLI も読心術は使えません。より多くの情報を提供すれば、人間も Copilot も的確なサポートができるようになります！*

同僚にバグを説明する場面を想像してみてください：

> **コンテキストなし**：「ブックアプリが動かないんだけど。」

> **コンテキストあり**：「`books.py` を見て、特に `find_book_by_title` 関数。大文字小文字を区別しないマッチングができていないんだ。」

Copilot CLI にコンテキストを提供するには、*`@` 構文* を使って Copilot CLI に特定のファイルを指し示します。

---

# 基本編：基本的なコンテキスト

<img src="images/essential-basic-context.png" alt="Glowing code blocks connected by light trails representing how context flows through Copilot CLI conversations" width="800"/>

このセクションでは、コンテキストを効果的に使うために必要なすべてをカバーします。まずはこの基本をマスターしましょう。

---

## @ 構文

`@` シンボルは、プロンプト内でファイルやディレクトリを参照するために使います。Copilot CLI に「このファイルを見て」と伝える方法です。

> 💡 **メモ**: このコースのすべての例はリポジトリに含まれる `samples/` フォルダを使用しているので、すべてのコマンドをそのまま試すことができます。

### 今すぐ試してみよう（セットアップ不要）

お使いのコンピュータにあるどんなファイルでも試すことができます：

```bash
copilot

# 手元にあるファイルを指定してみましょう
> Explain what @package.json does
> Summarize @README.md
> What's in @.gitignore and why?
```

> 💡 **手元にプロジェクトがない場合は？** テストファイルをサッと作りましょう：
> ```bash
> echo "def greet(name): return 'Hello ' + name" > test.py
> copilot
> > What does @test.py do?
> ```

### 基本的な @ パターン

| パターン | 機能 | 使用例 |
|---------|------|--------|
| `@file.py` | 単一ファイルを参照 | `Review @samples/book-app-project/books.py` |
| `@folder/` | ディレクトリ内のすべてのファイルを参照 | `Review @samples/book-app-project/` |
| `@file1.py @file2.py` | 複数ファイルを参照 | `Compare @samples/book-app-project/book_app.py @samples/book-app-project/books.py` |

### 単一ファイルを参照する

```bash
copilot

> Explain what @samples/book-app-project/utils.py does
```

---

<details>
<summary>🎬 実際の動作を見てみましょう！</summary>

![File Context Demo](images/file-context-demo.gif)

*デモの出力は実行ごとに異なります。モデル、ツール、レスポンスはここに表示されているものとは違う場合があります。*

</details>

---

### 複数ファイルを参照する

```bash
copilot

> Compare @samples/book-app-project/book_app.py and @samples/book-app-project/books.py for consistency
```

### ディレクトリ全体を参照する

```bash
copilot

> Review all files in @samples/book-app-project/ for error handling
```

---

## クロスファイルインテリジェンス

ここからコンテキストが超能力になります。単一ファイルの分析は便利ですが、クロスファイル分析は革命的です。

<img src="images/cross-file-intelligence.png" alt="Cross-File Intelligence - comparing single-file vs cross-file analysis showing how analyzing files together reveals bugs, data flow, and patterns invisible in isolation" width="800"/>

### デモ：複数ファイルにまたがるバグを見つける

```bash
copilot

> @samples/book-app-project/book_app.py @samples/book-app-project/books.py
>
> How do these files work together? What's the data flow?
```

> 💡 **上級者向けオプション**: セキュリティに焦点を当てたクロスファイル分析には、Python セキュリティの例を試してみてください：
> ```bash
> > @samples/buggy-code/python/user_service.py @samples/buggy-code/python/payment_processor.py
> > Find security vulnerabilities that span BOTH files
> ```

---

<details>
<summary>🎬 実際の動作を見てみましょう！</summary>

![Multi-File Demo](images/multi-file-demo.gif)

*デモの出力は実行ごとに異なります。モデル、ツール、レスポンスはここに表示されているものとは違う場合があります。*

</details>

---

**Copilot CLI が発見する内容**：

```
クロスモジュール分析
=====================

1. データフローパターン
   book_app.py が BookCollection インスタンスを作成してメソッドを呼び出す
   books.py が BookCollection クラスを定義してデータの永続化を管理する

   フロー: book_app.py (UI) → books.py (ビジネスロジック) → data.json (ストレージ)

2. 重複する表示関数
   book_app.py:9-21    show_books() 関数
   utils.py:28-36      print_books() 関数

   影響: ほぼ同一の関数が2つ同じことをしている。一方を更新（フォーマットの変更など）
   した場合、もう一方も更新する必要がある。

3. 一貫性のないエラーハンドリング
   book_app.py は年の変換で ValueError をハンドリングしている
   books.py はエラー時に暗黙的に None/False を返している

   パターン: モジュール間でエラーハンドリングの統一されたアプローチがない
```

**なぜこれが重要か**: 単一ファイルのレビューでは全体像を見落としてしまいます。クロスファイル分析だけが以下を明らかにします：
- 統合すべき**重複コード**
- コンポーネント間の相互作用を示す**データフローパターン**
- 保守性に影響する**アーキテクチャ上の問題**

---

### デモ：60秒でコードベースを理解する

<img src="images/codebase-understanding.png" alt="Split-screen comparison showing manual code review taking 1 hour versus AI-assisted analysis taking 10 seconds" width="800" />

プロジェクトに初めて触れますか？Copilot CLI を使って素早く把握しましょう。

```bash
copilot

> @samples/book-app-project/
>
> In one paragraph, what does this app do and what are its biggest quality issues?
```

**得られる結果**：
```
これは CLI ベースの書籍コレクション管理ツールで、JSON ファイルに保存された書籍の
追加、一覧表示、削除、検索ができます。最大の品質上の問題点は：

1. 表示ロジックの重複 - show_books() と print_books() が同じことをしている
2. 一貫性のないエラーハンドリング - 例外を投げるものとFalseを返すものが混在
3. 入力バリデーションなし - year に0が入り、title/author に空文字列が許容される
4. テスト不足 - find_book_by_title などの重要な関数にテストカバレッジがない

優先修正: 重複する表示関数の統合と入力バリデーションの追加。
```

**結果**: コードを1時間かけて読む作業が10秒に圧縮されます。どこに注力すべきか正確にわかります。

---

## 実践的な例

### 例1: コンテキスト付きコードレビュー

```bash
copilot

> @samples/book-app-project/books.py Review this file for potential bugs

# Copilot CLI がファイルの全内容を把握し、具体的なフィードバックを返します：
# "49行目: 大文字小文字を区別する比較で、書籍を見つけられない可能性が..."
# "29行目: JSON デコードエラーはキャッチされているが、データ破損はログに記録されていない..."

> What about @samples/book-app-project/book_app.py?

# book_app.py をレビューしつつ、books.py のコンテキストも把握した状態で
```

### 例2: コードベースの理解

```bash
copilot

> @samples/book-app-project/books.py What does this module do?

# Copilot CLI が books.py を読み込み、BookCollection クラスを理解します

> @samples/book-app-project/ Give me an overview of the code structure

# Copilot CLI がディレクトリをスキャンして要約します

> How does the app save and load books?

# Copilot CLI は既に見たコードをたどって追跡できます
```

<details>
<summary>🎬 マルチターン会話の実際の動作を見てみましょう！</summary>

![Multi-Turn Demo](images/multi-turn-demo.gif)

*デモの出力は実行ごとに異なります。モデル、ツール、レスポンスはここに表示されているものとは違う場合があります。*

</details>

### 例3: マルチファイルリファクタリング

```bash
copilot

> @samples/book-app-project/book_app.py @samples/book-app-project/utils.py
> I see duplicate display functions: show_books() and print_books(). Help me consolidate these.

# Copilot CLI が両方のファイルを見て、重複コードの統合方法を提案します
```

---

## セッション管理

セッションは作業中に自動的に保存されます。以前のセッションを再開して、中断したところから続けることができます。

### セッションの自動保存

すべての会話は自動的に保存されます。通常通り終了するだけです：

```bash
copilot

> @samples/book-app-project/ Let's improve error handling across all modules

[... 作業を行う ...]

> /exit
```

### 最新のセッションを再開する

```bash
# 中断したところから続ける
copilot --continue
```

### 特定のセッションを再開する

```bash
# セッション一覧からインタラクティブに選ぶ
copilot --resume

# または特定のセッション ID で再開する
copilot --resume abc123
```

> 💡 **セッション ID はどうやって調べるの？** 暗記する必要はありません。ID を指定せずに `copilot --resume` を実行すると、以前のセッションの一覧がインタラクティブに表示され、名前、ID、最終アクティブ日時が確認できます。使いたいセッションを選ぶだけです。
>
> **複数のターミナルがある場合は？** 各ターミナルウィンドウはそれぞれ独立したセッションとコンテキストを持ちます。3つのターミナルで Copilot CLI を開いている場合、3つの別々のセッションがあります。どのターミナルから `--resume` を実行しても、すべてのセッションを閲覧できます。`--continue` フラグは、どのターミナルで閉じたかに関係なく、最も最近閉じたセッションを取得します。
>
> **再起動せずにセッションを切り替えられる？** はい。アクティブなセッション内で `/resume` スラッシュコマンドを使います：
> ```
> > /resume
> # 切り替え先のセッション一覧が表示されます
> ```

### セッションを整理する

後から見つけやすいように、セッションにわかりやすい名前を付けましょう：

```bash
copilot

> /rename book-app-review
# セッション名が変更され、識別しやすくなります
```

### コンテキストの確認と管理

ファイルや会話を追加するにつれて、Copilot CLI の [context window](../GLOSSARY.md#context-window) が埋まっていきます。2つのコマンドでコントロールできます：

```bash
copilot

> /context
Context usage: 45,000 / 128,000 tokens (35%)

> /clear
# コンテキストを消去してやり直します。トピックを切り替えるときに使いましょう
```

> 💡 **`/clear` を使うタイミング**: `books.py` のレビューをしていて `utils.py` の話に切り替えたい場合は、先に `/clear` を実行しましょう。そうしないと、以前のトピックの古いコンテキストがレスポンスを混乱させる可能性があります。

---

### 中断したところから再開する

<img src="images/session-persistence-timeline.png" alt="Timeline showing how GitHub Copilot CLI sessions persist across days - start on Monday, resume on Wednesday with full context restored" width="800"/>

*セッションは終了時に自動保存されます。数日後にフルコンテキストで再開：ファイル、問題点、進捗すべてが記憶されています。*

複数日にわたるワークフローを想像してみてください：

```bash
# 月曜日: ブックアプリのレビューを開始
copilot

> /rename book-app-review
> @samples/book-app-project/books.py
> Review and number all code quality issues

発見された品質問題:
1. 重複する表示関数 (book_app.py & utils.py) - 中
2. 空文字列の入力バリデーションなし - 中
3. year に 0 や負数が入る - 低
4. すべての関数に型ヒントがない - 低
5. エラーログが欠落 - 低

> Fix issue #1 (duplicate functions)
# 修正に取り組む...

> /exit
```

```bash
# 水曜日: 中断したところから正確に再開
copilot --continue

> What issues remain unfixed from our book app review?

ブックアプリレビューの未修正の問題:
2. 空文字列の入力バリデーションなし - 中
3. year に 0 や負数が入る - 低
4. すべての関数に型ヒントがない - 低
5. エラーログが欠落 - 低

問題 #1 (重複関数) は月曜日に修正済み。

> Let's tackle issue #2 next
```

**これが強力な理由**: 数日後でも、Copilot CLI は以下を記憶しています：
- 作業していた正確なファイル
- 番号付きの問題リスト
- どの問題が既に対処済みか
- 会話のコンテキスト

再度説明する必要なし。ファイルを再度読み直す必要なし。ただ作業を続けるだけです。

---

**🎉 基本をマスターしました！** `@` 構文、セッション管理（`--continue`/`--resume`/`/rename`）、コンテキストコマンド（`/context`/`/clear`）だけで十分に生産性が高まります。以下はすべてオプションです。準備ができたら戻ってきてください。

---

# オプション：さらに深く学ぶ

<img src="images/optional-going-deeper.png" alt="Abstract crystal cave in blue and purple tones representing deeper exploration of context concepts" width="800"/>

これらのトピックは上記の基本の上に構築されています。**興味のあるものを選ぶか、[実践練習](#practice) まで飛ばしてください。**

| 学びたいこと... | ジャンプ先 |
|---|---|
| ワイルドカードパターンと高度なセッションコマンド | [追加の @ パターンとセッションコマンド](#additional-patterns) |
| 複数のプロンプトにまたがるコンテキスト構築 | [コンテキストアウェアな会話](#context-aware-conversations) |
| トークン制限と `/compact` | [context window を理解する](#understanding-context-windows) |
| 参照すべきファイルの選び方 | [何を参照すべきか選ぶ](#choosing-what-to-reference) |
| スクリーンショットやモックアップの分析 | [画像を扱う](#working-with-images) |

<details>
<summary><strong>追加の @ パターンとセッションコマンド</strong></summary>
<a id="additional-patterns"></a>

### 追加の @ パターン

パワーユーザー向けに、Copilot CLI はワイルドカードパターンと画像参照をサポートしています：

| パターン | 機能 |
|---------|------|
| `@folder/*.py` | フォルダ内のすべての .py ファイル |
| `@**/test_*.py` | 再帰ワイルドカード：任意の場所にあるすべてのテストファイルを検索 |
| `@image.png` | UI レビュー用の画像ファイル |

```bash
copilot

> Find all TODO comments in @samples/book-app-project/**/*.py
```

### セッション情報の表示

```bash
copilot

> /session
# 現在のセッション詳細とワークスペースの概要を表示します

> /usage
# セッションのメトリクスと統計情報を表示します
```

### セッションの共有

```bash
copilot

> /share file ./my-session.md
# セッションを Markdown ファイルとしてエクスポートします

> /share gist
# セッションの内容で GitHub gist を作成します
```

</details>

<details>
<summary><strong>コンテキストアウェアな会話</strong></summary>
<a id="context-aware-conversations"></a>

### コンテキストアウェアな会話

マルチターンの会話が互いに積み上がっていくとき、魔法が起こります。

#### 例：段階的な改善

```bash
copilot

> @samples/book-app-project/books.py Review the BookCollection class

Copilot CLI: "クラスは機能的ですが、以下の点に気づきました：
1. 一部のメソッドに型ヒントがない
2. 空の title/author に対するバリデーションがない
3. エラーハンドリングの改善が有効"

> Add type hints to all methods

Copilot CLI: "完全な型ヒントを追加したクラスはこちらです..."
[型付きバージョンを表示]

> Now improve error handling

Copilot CLI: "型付きバージョンをベースに、改善されたエラーハンドリングはこちらです..."
[バリデーションと適切な例外を追加]

> Generate tests for this final version

Copilot CLI: "型とエラーハンドリングを備えたクラスに基づいて..."
[包括的なテストを生成]
```

各プロンプトが前の作業の上に構築されていることに注目してください。これがコンテキストの力です。

</details>

<details>
<summary><strong>context window を理解する</strong></summary>
<a id="understanding-context-windows"></a>

### context window を理解する

基本編で `/context` と `/clear` は既に学びました。ここでは context window がどのように機能するかをより深く見ていきます。

すべての AI には「context window」があり、一度に考慮できるテキストの量のことです。

<img src="images/context-window-visualization.png" alt="Context Window Visualization" width="800"/>

*context window はデスクのようなものです：一度に置ける量には限りがあります。ファイル、会話履歴、システムプロンプトのすべてがスペースを占有します。*

#### 制限に達したときの動作

```bash
copilot

> /context

Context usage: 45,000 / 128,000 tokens (35%)

# ファイルや会話を追加するにつれて、使用量が増えていきます

> @large-codebase/

Context usage: 120,000 / 128,000 tokens (94%)

# 警告: コンテキスト制限に近づいています

> @another-large-file.py

Context limit reached. Older context will be summarized.
```

#### `/compact` コマンド

コンテキストがいっぱいになりつつあるが会話を失いたくない場合、`/compact` は履歴を要約してトークンを解放します：

```bash
copilot

> /compact
# 会話履歴を要約し、コンテキストスペースを解放します
# 重要な発見や決定は保持されます
```

#### コンテキスト効率化のヒント

| 状況 | アクション | 理由 |
|------|----------|------|
| 新しいトピックを開始 | `/clear` | 関係ないコンテキストを除去 |
| 長い会話 | `/compact` | 履歴を要約し、トークンを解放 |
| 特定のファイルが必要 | `@folder/` ではなく `@file.py` | 必要なものだけをロード |
| 制限に到達 | 新しいセッションを開始 | 新鮮な 128K コンテキスト |
| 複数のトピック | トピックごとに `/rename` | 適切なセッションを簡単に再開 |

#### 大規模コードベースのベストプラクティス

1. **具体的に指定する**: `@samples/book-app-project/` ではなく `@samples/book-app-project/books.py`
2. **トピック間でクリアする**: フォーカスを切り替えるときは `/clear` を使う
3. **`/compact` を使う**: 会話を要約してコンテキストを解放する
4. **複数のセッションを使う**: 機能やトピックごとに1セッション

</details>

<details>
<summary><strong>何を参照すべきか選ぶ</strong></summary>
<a id="choosing-what-to-reference"></a>

### 何を参照すべきか選ぶ

コンテキストに関して言えば、すべてのファイルが同じ価値を持つわけではありません。賢い選び方をしましょう：

#### ファイルサイズの考慮事項

| ファイルサイズ | おおよその[トークン](../GLOSSARY.md#token)数 | 戦略 |
|-----------|-------------------|----------|
| 小（100行未満） | 約500〜1,500トークン | 自由に参照 |
| 中（100〜500行） | 約1,500〜7,500トークン | 特定のファイルを参照 |
| 大（500行以上） | 7,500トークン以上 | 選択的に、特定のファイルを使用 |
| 非常に大きい（1000行以上） | 15,000トークン以上 | 分割するかセクションを絞ることを検討 |

**具体例：**
- ブックアプリの4つの Python ファイル合計 ≈ 2,000〜3,000トークン
- 典型的な Python モジュール（200行）≈ 3,000トークン
- Flask API ファイル（400行）≈ 6,000トークン
- package.json ≈ 200〜500トークン
- 短いプロンプト + レスポンス ≈ 500〜1,500トークン

> 💡 **コードのざっくり見積もり:** コード行数に約15を掛けるとおおよそのトークン数になります。あくまで目安であることを覚えておいてください。

#### 含めるべきもの vs 除外すべきもの

**価値が高い**（これらを含める）：
- エントリーポイント (`book_app.py`, `main.py`, `app.py`)
- 質問の対象となっている特定のファイル
- 対象ファイルが直接インポートしているファイル
- 設定ファイル (`requirements.txt`, `pyproject.toml`)
- データモデルやデータクラス

**価値が低い**（除外を検討）：
- 生成されたファイル（コンパイル出力、バンドルされたアセット）
- node modules やベンダーディレクトリ
- 大きなデータファイルやフィクスチャ
- 質問に関係のないファイル

#### 具体性のスペクトラム

```
抽象的 ────────────────────────► 具体的
@samples/book-app-project/                      @samples/book-app-project/books.py:47-52
     │                                       │
     └─ すべてをスキャン                      └─ 必要なものだけ
        (コンテキストを多く消費)                   (コンテキストを節約)
```

**広く指定すべきとき** (`@samples/book-app-project/`)：
- 初回のコードベース探索
- 多数のファイルにわたるパターン発見
- アーキテクチャレビュー

**具体的に指定すべきとき** (`@samples/book-app-project/books.py`)：
- 特定の問題のデバッグ
- 特定ファイルのコードレビュー
- 単一の関数についての質問

#### 実践例：段階的なコンテキストロード

```bash
copilot

# ステップ1: 構造から始める
> @package.json What frameworks does this project use?

# ステップ2: 回答に基づいて絞り込む
> @samples/book-app-project/ Show me the project structure

# ステップ3: 重要なものにフォーカスする
> @samples/book-app-project/books.py Review the BookCollection class

# ステップ4: 必要に応じて関連ファイルを追加する
> @samples/book-app-project/book_app.py @samples/book-app-project/books.py How does the CLI use the BookCollection?
```

この段階的なアプローチにより、コンテキストを効率的に絞り込めます。

</details>

<details>
<summary><strong>画像を扱う</strong></summary>
<a id="working-with-images"></a>

### 画像を扱う

`@` 構文を使って会話に画像を含めることができます。また、**クリップボードから貼り付ける**だけでもOKです（Cmd+V / Ctrl+V）。Copilot CLI はスクリーンショット、モックアップ、ダイアグラムを分析して、UI デバッグ、デザイン実装、エラー分析をサポートできます。

```bash
copilot

> @images/screenshot.png What is happening in this image?

> @images/mockup.png Write the HTML and CSS to match this design. Place it in a new file called index.html and put the CSS in styles.css.
```

> 📖 **もっと学ぶ**: サポートされるフォーマット、実践的なユースケース、画像とコードを組み合わせるヒントについては [Additional Context Features](../appendices/additional-context.md#working-with-images) を参照してください。

</details>

---

# 実践練習

<img src="../images/practice.png" alt="Warm desk setup with monitor showing code, lamp, coffee cup, and headphones ready for hands-on practice" width="800"/>

コンテキストとセッション管理のスキルを実践する時間です。

---

## ▶️ 自分で試してみよう

### プロジェクト全体のレビュー

このコースにはそのままレビューできるサンプルファイルが含まれています。copilot を起動して、次のプロンプトを実行してみましょう：

```bash
copilot

> @samples/book-app-project/ Give me a code quality review of this project

# Copilot CLI が以下のような問題を特定します：
# - 重複する表示関数
# - 入力バリデーションの欠如
# - 一貫性のないエラーハンドリング
```

> 💡 **自分のファイルで試したいですか？** 小さな Python プロジェクトを作成し（`mkdir -p my-project/src`）、いくつかの .py ファイルを追加して、`@my-project/src/` でレビューしてみましょう。サンプルコードの作成を copilot に頼むこともできます！

### セッションワークフロー

```bash
copilot

> /rename book-app-review
> @samples/book-app-project/books.py Let's add input validation for empty titles

[Copilot CLI がバリデーションのアプローチを提案]

> Implement that fix
> Now consolidate the duplicate display functions in @samples/book-app-project/
> /exit

# 後で - 中断したところから再開
copilot --continue

> Generate tests for the changes we made
```

---

デモを完了したら、以下のバリエーションを試してみましょう：

1. **クロスファイルチャレンジ**: book_app.py と books.py がどのように連携しているか分析する:
   ```bash
   copilot
   > @samples/book-app-project/book_app.py @samples/book-app-project/books.py
   > What's the relationship between these files? Are there any code smells?
   ```

2. **セッションチャレンジ**: セッションを開始し、`/rename my-first-session` で名前を付け、何か作業をして、`/exit` で終了し、`copilot --continue` を実行します。何をしていたか覚えていますか？

3. **コンテキストチャレンジ**: セッション中に `/context` を実行します。何トークン使っていますか？ `/compact` を試してもう一度確認しましょう。（`/compact` の詳細は「さらに深く学ぶ」の [context window を理解する](#understanding-context-windows) を参照してください。）

**セルフチェック**: `@folder/` が各ファイルを個別に開くよりもなぜ強力なのかを説明できれば、コンテキストを理解できています。

---

## 📝 課題

### メインチャレンジ：データフローを追跡する

ハンズオンの例ではコード品質レビューと入力バリデーションに焦点を当てました。ここでは同じコンテキストスキルを別のタスク、つまりアプリ内のデータの流れを追跡する練習をします：

1. インタラクティブセッションを開始する: `copilot`
2. `books.py` と `book_app.py` を一緒に参照する:
   `@samples/book-app-project/books.py @samples/book-app-project/book_app.py Trace how a book goes from user input to being saved in data.json. What functions are involved at each step?`
3. 追加のコンテキストとしてデータファイルを含める:
   `@samples/book-app-project/data.json What happens if this JSON file is missing or corrupted? Which functions would fail?`
4. クロスファイルの改善を依頼する:
   `@samples/book-app-project/books.py @samples/book-app-project/utils.py Suggest a consistent error-handling strategy that works across both files.`
5. セッション名を変更する: `/rename data-flow-analysis`
6. `/exit` で終了し、`copilot --continue` で再開してデータフローについてフォローアップの質問をする

**成功基準**: 複数ファイルにまたがるデータを追跡でき、名前付きセッションを再開でき、クロスファイルの提案を得られること。

<details>
<summary>💡 ヒント（クリックして展開）</summary>

**始め方：**
```bash
cd /path/to/copilot-cli-for-beginners
copilot
> @samples/book-app-project/books.py @samples/book-app-project/book_app.py Trace how a book goes from user input to being saved in data.json.
> @samples/book-app-project/data.json What happens if this file is missing or corrupted?
> /rename data-flow-analysis
> /exit
```

その後 `copilot --continue` で再開します。

**便利なコマンド：**
- `@file.py` - 単一ファイルを参照
- `@folder/` - フォルダ内のすべてのファイルを参照（末尾の `/` に注意）
- `/context` - コンテキストの使用量を確認
- `/rename <name>` - セッションに名前を付けて簡単に再開

</details>

### ボーナスチャレンジ：コンテキストの制限

1. `@samples/book-app-project/` でブックアプリのすべてのファイルを一度に参照する
2. 異なるファイル（`books.py`、`utils.py`、`book_app.py`、`data.json`）について詳細な質問をいくつかする
3. `/context` を実行して使用量を確認する。どのくらい素早く埋まりますか？
4. `/compact` を使ってスペースを回収する練習をし、その後会話を続ける
5. より具体的なファイル参照（フォルダ全体ではなく `@samples/book-app-project/books.py`）を試して、コンテキスト使用量への影響を確認する

---

<details>
<summary>🔧 <strong>よくある間違いとトラブルシューティング</strong>（クリックして展開）</summary>

### よくある間違い

| 間違い | 何が起こるか | 修正方法 |
|---------|------------|---------|
| ファイル名の前の `@` を忘れる | Copilot CLI が "books.py" をプレーンテキストとして扱う | ファイルを参照するには `@samples/book-app-project/books.py` を使う |
| セッションが自動的に保持されると思い込む | `copilot` を新しく起動すると以前のコンテキストがすべて失われる | `--continue`（最新セッション）または `--resume`（セッションを選択）を使う |
| カレントディレクトリ外のファイルを参照する | "Permission denied" や "File not found" エラー | `/add-dir /path/to/directory` でアクセスを許可する |
| トピック切り替え時に `/clear` を使わない | 古いコンテキストが新しいトピックのレスポンスを混乱させる | 別のタスクを始める前に `/clear` を実行する |

### トラブルシューティング

**"File not found" エラー** - 正しいディレクトリにいることを確認してください：

```bash
pwd  # カレントディレクトリを確認
ls   # ファイル一覧を表示

# copilot を起動して相対パスを使う
copilot

> Review @samples/book-app-project/books.py
```

**"Permission denied"** - 許可リストにディレクトリを追加する：

```bash
copilot --add-dir /path/to/directory

# またはセッション内で：
> /add-dir /path/to/directory
```

**コンテキストがすぐにいっぱいになる**：
- より具体的なファイル参照を使う
- 異なるトピック間で `/clear` を使う
- 作業を複数のセッションに分割する

</details>

---

# まとめ

## 🔑 重要ポイント

1. **`@` 構文** は Copilot CLI にファイル、ディレクトリ、画像のコンテキストを与えます
2. **マルチターン会話** はコンテキストが蓄積されるにつれて互いに積み上がっていきます
3. **セッションは自動保存**: `--continue` または `--resume` で中断したところから再開できます
4. **context window** には制限があります: `/context`、`/clear`、`/compact` で管理しましょう
5. **パーミッションフラグ** (`--add-dir`、`--allow-all`) がマルチディレクトリアクセスを制御します。賢く使いましょう！
6. **画像参照** (`@screenshot.png`) は UI の問題を視覚的にデバッグするのに役立ちます

> 📚 **公式ドキュメント**: コンテキスト、セッション、ファイル操作の完全なリファレンスは [Use Copilot CLI](https://docs.github.com/copilot/how-tos/copilot-cli/use-copilot-cli) を参照してください。

> 📋 **クイックリファレンス**: コマンドとショートカットの完全な一覧は [GitHub Copilot CLI command reference](https://docs.github.com/en/copilot/reference/cli-command-reference) を参照してください。

---

## ➡️ 次のステップ

Copilot CLI にコンテキストを与える方法がわかったので、実際の開発タスクに活用しましょう。ここで学んだコンテキスト技法（ファイル参照、クロスファイル分析、セッション管理）は、次の章で紹介する強力なワークフローの基盤となります。

**[Chapter 03: Development Workflows](../03-development-workflows/README.md)** では、以下を学びます：

- コードレビューのワークフロー
- リファクタリングパターン
- デバッグアシスタンス
- テスト生成
- Git 連携

---

**[← Chapter 01 に戻る](../01-setup-and-first-steps/README.md)** | **[Chapter 03 に進む →](../03-development-workflows/README.md)**
