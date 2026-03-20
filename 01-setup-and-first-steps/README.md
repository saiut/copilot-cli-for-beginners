![Chapter 01: First Steps](images/chapter-header.png)

> **AI がバグを瞬時に見つけ、分かりにくいコードを解説し、動作するスクリプトを生成する様子を体験しましょう。さらに、GitHub Copilot CLI の 3 つの使い方を学びます。**

この章からいよいよ本番です！開発者たちが GitHub Copilot CLI を「いつでも相談できるシニアエンジニア」と表現する理由を、実際に体験していただきます。AI がセキュリティバグを数秒で見つけ、複雑なコードをわかりやすく解説し、動作するスクリプトを即座に生成する様子をご覧ください。そして、3 つのインタラクションモード（Interactive、Plan、Programmatic）をマスターして、どんなタスクにどのモードを使えばよいかを理解しましょう。

> ⚠️ **前提条件**: 先に **[Chapter 00: Quick Start](../00-quick-start/README.md)** を完了してください。以下のデモを実行するには、GitHub Copilot CLI がインストールされ、認証が完了している必要があります。

## 🎯 学習目標

この章を終えると、以下のことができるようになります：

- ハンズオンデモを通じて GitHub Copilot CLI がもたらす生産性向上を体感する
- タスクに応じて適切なモード（Interactive、Plan、Programmatic）を選択する
- slash commands を使ってセッションを操作する

> ⏱️ **所要時間の目安**: 約 45 分（読み物 15 分 + ハンズオン 30 分）

---

# 初めての Copilot CLI 体験

<img src="images/first-copilot-experience.png" alt="Developer sitting at a desk with code on the monitor and glowing particles representing AI assistance" width="800"/>

さっそく Copilot CLI にできることを見てみましょう。

---

## ウォーミングアップ：最初のプロンプト

印象的なデモに入る前に、今すぐ試せる簡単なプロンプトから始めましょう。**コードリポジトリは不要です**！ターミナルを開いて Copilot CLI を起動するだけです：

```bash
copilot
```

以下の初心者向けプロンプトを試してみてください：

```
> Explain what a dataclass is in Python in simple terms

> Write a function that sorts a list of dictionaries by a specific key

> What's the difference between a list and a tuple in Python?

> Give me 5 best practices for writing clean Python code
```

Python を使っていない方もご安心ください！お好みの言語について質問すれば大丈夫です。

同僚に話しかけるように自然に質問できることに気づくでしょう。探索が終わったら、`/exit` と入力してセッションを終了します。

**ここがポイント**: GitHub Copilot CLI は会話型です。特別な構文は必要ありません。普通の言葉で質問するだけで始められます。

## 実際に動かしてみよう

開発者たちが「いつでも相談できるシニアエンジニア」と呼ぶ理由を見てみましょう。

> 📖 **例の読み方**: `>` で始まる行は、Interactive な Copilot CLI セッション内で入力するプロンプトです。`>` が付いていない行は、ターミナルで実行するシェルコマンドです。

> 💡 **出力例について**: このコース全体で示される出力例はあくまで参考です。Copilot CLI の応答は毎回異なるため、表現、フォーマット、詳細度は変わります。正確なテキストではなく、返される*情報の種類*に注目してください。

### デモ 1：数秒でコードレビュー

このコースには、意図的にコード品質の問題を含むサンプルファイルが用意されています。そのうちの 1 つをレビューしてみましょう：

```bash
# ローカルで作業していてまだクローンしていない場合は、コースリポジトリをクローンする
git clone https://github.com/github/copilot-cli-for-beginners
cd copilot-cli-for-beginners

# Copilot を起動する
copilot
```

Interactive セッションに入ったら：

```
> Review @samples/book-app-project/book_app.py for code quality issues and suggest improvements
```

> 💡 **`@` って何？** `@` シンボルは Copilot CLI にファイルを読み込むよう指示します。これについては Chapter 02 で詳しく学びます。今はコマンドをそのままコピーしてください。

---

<details>
<summary>🎬 動作を見てみよう！</summary>

![Code Review Demo](images/code-review-demo.gif)

*デモの出力は毎回異なります。使用されるモデル、ツール、応答はここに表示されているものとは異なります。*

</details>

---

**ここがポイント**: プロフェッショナルなコードレビューが数秒で完了します。手動でレビューすると……まあ、もっと時間がかかりますね！

---

### デモ 2：分かりにくいコードを解説

コードを見て「何をしてるんだろう？」と思ったことはありませんか？Copilot CLI セッションで試してみましょう：

```
> Explain what @samples/book-app-project/books.py does in simple terms
```

---

<details>
<summary>🎬 動作を見てみよう！</summary>

![Explain Code Demo](images/explain-code-demo.gif)

*デモの出力は毎回異なります。使用されるモデル、ツール、応答はここに表示されているものとは異なります。*

</details>

---

**何が起こるか**: （出力は毎回異なります）Copilot CLI がファイルを読み込み、コードを理解し、わかりやすい言葉で解説してくれます。

```
This is a book collection management module using Python dataclasses.

Think of it like a digital bookshelf:
- Book is a dataclass - a simple way to store book information (title, author, year, read status)
- BookCollection manages the entire collection and handles saving/loading

Key components:

1. @dataclass decorator (line 8)
   - Automatically creates __init__, __repr__, etc.
   - Clean way to define data structures in Python

2. BookCollection class (line 16)
   - Maintains a list of Book objects
   - Handles persistence with JSON file I/O
   - load_books() reads from data.json
   - save_books() writes to data.json using asdict()

3. Book operations:
   - add_book() - creates and saves new books
   - find_book_by_title() - searches collection
   - mark_as_read() - updates read status
   - find_by_author() - filters by author name

Common pattern: Read from JSON → Work with Python objects → Write back to JSON
```

**ここがポイント**: 複雑なコードを、まるで忍耐強いメンターのように解説してくれます。

---

### デモ 3：動作するコードを生成

普段なら 15 分かけて検索するような関数が必要ですか？セッションを続けて：

```
> Write a Python function that takes a list of books and returns statistics: 
  total count, number read, number unread, oldest and newest book
```

---

<details>
<summary>🎬 動作を見てみよう！</summary>

![Generate Code Demo](images/generate-code-demo.gif)

*デモの出力は毎回異なります。使用されるモデル、ツール、応答はここに表示されているものとは異なります。*

</details>

---

**何が起こるか**: 完全に動作する関数が数秒で生成され、すぐにコピー＆ペーストして実行できます。

探索が終わったら、セッションを終了します：

```
> /exit
```

**ここがポイント**: 即座に結果が得られ、1 つの連続したセッションで全てを行えました。

---

# モードとコマンド

<img src="images/modes-and-commands.png" alt="Futuristic control panel with glowing screens, dials, and equalizers representing Copilot CLI modes and commands" width="800"/>

Copilot CLI にできることを見てきました。次は、これらの機能を*どのように*効果的に使うかを理解しましょう。ポイントは、状況に応じて 3 つのインタラクションモードのどれを使うかを知ることです。

> 💡 **補足**: Copilot CLI には **Autopilot** モードもあり、あなたの入力を待たずにタスクを進めてくれます。強力ですが、完全な権限の付与が必要で、premium requests を自律的に消費します。このコースでは以下の 3 つのモードに焦点を当てます。基本に慣れたら Autopilot をご紹介します。

---

## 🧩 身近なたとえ：外食に行く

GitHub Copilot CLI の使い方を外食にたとえてみましょう。お店に行く計画から注文まで、状況によってアプローチは変わりますよね：

| モード | 外食のたとえ | 使うタイミング |
|------|----------------|-------------|
| **Plan** | レストランまでの GPS ルート | 複雑なタスク - ルートを計画し、確認ポイントを見直し、計画に同意してから実行する |
| **Interactive** | ウェイターと会話する | 探索と反復 - 質問し、カスタマイズし、リアルタイムでフィードバックを得る |
| **Programmatic** | ドライブスルーで注文 | 素早く具体的なタスク - 自分の環境にとどまり、結果をすぐに得る |

外食と同じように、どのアプローチがふさわしいか、自然と分かるようになります。

<img src="images/ordering-food-analogy.png" alt="Three Ways to Use GitHub Copilot CLI - Plan Mode (GPS route to restaurant), Interactive Mode (talking to waiter), Programmatic Mode (drive-through)" width="800"/>

*タスクに応じてモードを選びましょう：Plan は事前に計画を立てたいとき、Interactive はやり取りしながら共同作業したいとき、Programmatic はすぐに結果が欲しいとき*

### どのモードから始めればいい？

**Interactive mode から始めましょう。**
- 試行錯誤や追加の質問ができます
- 会話を通じて自然にコンテキストが蓄積されます
- 間違えても `/clear` で簡単にやり直せます

慣れたら以下を試してみてください：
- **Programmatic mode**（`copilot -p "<your prompt>"`）で素早い一回限りの質問に
- **Plan mode**（`/plan`）でコーディング前にじっくり計画を立てたいときに

---

## 3 つのモード

### モード 1：Interactive Mode（まずはここから）

<img src="images/interactive-mode.png" alt="Interactive Mode - Like talking to a waiter who can answer questions and adjust the order" width="250"/>

**最適な用途**: 探索、反復、複数ターンの会話。質問に答え、フィードバックを受け取り、その場で注文を調整できるウェイターと話すようなものです。

Interactive セッションを開始します：

```bash
copilot
```

ここまで見てきたように、自然に入力できるプロンプトが表示されます。利用可能なコマンドのヘルプを見るには、以下を入力するだけです：

```
> /help
```

**ここがポイント**: Interactive mode はコンテキストを保持します。各メッセージが前のメッセージの上に積み重なり、まさに本物の会話のようです。

#### Interactive Mode の例

```bash
copilot

> Review @samples/book-app-project/utils.py and suggest improvements

> Add type hints to all functions

> Make the error handling more robust

> /exit
```

各プロンプトが前の回答の上に構築されていることに注目してください。毎回最初からやり直すのではなく、会話を続けているのです。

---

### モード 2：Plan Mode

<img src="images/plan-mode.png" alt="Plan Mode - Like planning a route before a trip using GPS" width="250"/>

**最適な用途**: 実行前にアプローチを確認したい複雑なタスク。GPS を使って旅行前にルートを計画するのと似ています。

Plan mode は、コードを書く前にステップごとの計画を作成するのに役立ちます。`/plan` コマンドを使うか、**Shift+Tab** を押して Plan Mode に切り替えます：

> 💡 **ヒント**: **Shift+Tab** でモードが切り替わります：Interactive → Plan → Autopilot。Interactive セッション中いつでも押してコマンドを入力せずにモードを切り替えられます。

```bash
copilot

> /plan Add a "mark as read" command to the book app
```

**Plan mode の出力:** （出力は毎回異なります）

```
📋 Implementation Plan

Step 1: Update the command handler in book_app.py
  - Add new elif branch for "mark" command
  - Create handle_mark_as_read() function

Step 2: Implement the handler function
  - Prompt user for book title
  - Call collection.mark_as_read(title)
  - Display success/failure message

Step 3: Update help text
  - Add "mark" to available commands list
  - コマンドの使い方をドキュメント化する

Step 4: Test the flow
  - 本を追加する
  - 既読にする
  - リスト出力でステータスが変わっていることを確認する

Proceed with implementation? [Y/n]
```

**ここがポイント**: Plan mode では、コードが書かれる前にアプローチを確認・修正できます。計画が完成したら、後で参照できるようにファイルに保存するよう Copilot CLI に指示することもできます。たとえば、「Save this plan to `mark_as_read_plan.md`」と言えば、計画の詳細を含む Markdown ファイルが作成されます。

> 💡 **もっと複雑なものを試したいですか？** `/plan Add search and filter capabilities to the book app` を試してみてください。Plan mode はシンプルな機能から本格的なアプリケーションまで対応します。

> 📚 **Autopilot mode**: Shift+Tab で **Autopilot** という 3 つ目のモードに切り替わることに気づいたかもしれません。Autopilot mode では、Copilot が各ステップでの入力を待たずに計画全体を実行します。同僚にタスクを渡して「終わったら教えて」と言うようなものです。一般的なワークフローは「計画 → 承認 → Autopilot」で、つまり良い計画を書く力がまず必要です。Interactive mode と Plan mode に慣れてから、準備ができたら [公式ドキュメント](https://docs.github.com/copilot/concepts/agents/copilot-cli/autopilot) をご覧ください。

---

### モード 3：Programmatic Mode

<img src="images/programmatic-mode.png" alt="Programmatic Mode - Like using a drive-through for a quick order" width="250"/>

**最適な用途**: 自動化、スクリプト、CI/CD、ワンショットコマンド。ウェイターと話す必要のないドライブスルーのようなものです。

`-p` フラグを使って、対話が不要な一回限りのコマンドを実行します：

```bash
# コードを生成する
copilot -p "Write a function that checks if a number is even or odd"

# 簡単なヘルプを得る
copilot -p "How do I read a JSON file in Python?"
```

**ここがポイント**: Programmatic mode は即座に回答して終了します。会話はなく、入力 → 出力のみです。

<details>
<summary>📚 <strong>さらに詳しく：Programmatic Mode をスクリプトで使う</strong>（クリックで展開）</summary>

慣れてきたら、`-p` をシェルスクリプト内で使えます：

```bash
#!/bin/bash

# コミットメッセージを自動生成する
COMMIT_MSG=$(copilot -p "Generate a commit message for: $(git diff --staged)")
git commit -m "$COMMIT_MSG"

# ファイルをレビューする
copilot --allow-all -p "Review @myfile.py for issues"
```
> ⚠️ **`--allow-all` について**: このフラグはすべての権限プロンプトをスキップし、Copilot CLI がファイルの読み取り、コマンドの実行、URL へのアクセスを確認なしで行えるようにします。Programmatic mode（`-p`）では操作を承認する対話セッションがないため、このフラグが必要です。`--allow-all` は自分で書いたプロンプトと信頼できるディレクトリでのみ使用してください。信頼できない入力や機密性の高いディレクトリでは決して使用しないでください。

</details>

---

## 基本の Slash Commands

これらのコマンドは Interactive mode で使えます。**まずはこの 4 つだけ覚えましょう** - 日常の 90% をカバーします：

| コマンド | 機能 | 使うタイミング |
|---------|--------------|-------------|
| `/help` | 利用可能なすべてのコマンドを表示する | コマンドを忘れたとき |
| `/clear` | 会話をクリアして最初からやり直す | トピックを切り替えるとき |
| `/plan` | コーディングの前に作業を計画する | より複雑な機能に取り組むとき |
| `/research` | GitHub や Web ソースを使った詳細な調査 | コーディング前にトピックを調査したいとき |
| `/model` | AI モデルの表示または切り替え | AI モデルを変更したいとき |
| `/exit` | セッションを終了する | 作業が終わったとき |

入門としてはこれで十分です！慣れてきたら、追加のコマンドを探索してみましょう。

> 📚 **公式ドキュメント**: [CLI command reference](https://docs.github.com/copilot/reference/cli-command-reference) でコマンドとフラグの完全なリストを確認できます。

<details>
<summary>📚 <strong>追加のコマンド</strong>（クリックで展開）</summary>

> 💡 上記の 5 つのコマンドで日常的に使う場面のほとんどをカバーできます。このリファレンスは、さらに探索する準備ができたときのためのものです。

### エージェント環境

| コマンド | 機能 |
|---------|--------------|
| `/init` | リポジトリの Copilot 設定を初期化する |
| `/agent` | 利用可能なエージェントを閲覧・選択する |
| `/skills` | スキルを管理して機能を拡張する |
| `/mcp` | MCP サーバーの設定を管理する |

> 💡 スキルについては [Chapter 05](../05-skills/README.md) で詳しく解説します。MCP サーバーについては [Chapter 06](../06-mcp-servers/README.md) で解説します。

### モデルとサブエージェント

| コマンド | 機能 |
|---------|--------------|
| `/model` | AI モデルの表示または切り替え |
| `/delegate` | GitHub 上の Copilot coding agent（クラウド上のエージェント）にタスクを引き渡す |
| `/fleet` | 複雑なタスクを並列サブタスクに分割して高速化する |
| `/tasks` | バックグラウンドのサブエージェントとデタッチされたシェルセッションを表示する |

### コード

| コマンド | 機能 |
|---------|--------------|
| `/diff` | 現在のディレクトリで行われた変更をレビューする |
| `/review` | コードレビューエージェントを実行して変更を分析する |
| `/research` | GitHub や Web ソースを使った詳細な調査を実行する |
| `/terminal-setup` | 複数行入力サポートを有効にする（Shift+Enter と Ctrl+Enter） |

### 権限

| コマンド | 機能 |
|---------|--------------|
| `/allow-all` | このセッションのすべての権限プロンプトを自動承認する |
| `/add-dir <directory>` | 許可リストにディレクトリを追加する |
| `/list-dirs` | 許可されたすべてのディレクトリを表示する |
| `/cwd`, `/cd [directory]` | 作業ディレクトリの表示または変更 |

> ⚠️ **注意して使用してください**: `/allow-all` は確認プロンプトをスキップします。信頼できるプロジェクトには便利ですが、信頼できないコードでは注意が必要です。

### セッション

| コマンド | 機能 |
|---------|--------------|
| `/resume` | 別のセッションに切り替える（オプションでセッション ID を指定） |
| `/rename` | 現在のセッション名を変更する |
| `/context` | コンテキストウィンドウのトークン使用量と可視化を表示する |
| `/usage` | セッションの使用状況メトリクスと統計を表示する |
| `/session` | セッション情報とワークスペースの概要を表示する |
| `/compact` | 会話を要約してコンテキスト使用量を削減する |
| `/share` | セッションを Markdown ファイルまたは GitHub gist としてエクスポートする |

### ヘルプとフィードバック

| コマンド | 機能 |
|---------|--------------|
| `/help` | 利用可能なすべてのコマンドを表示する |
| `/changelog` | CLI バージョンの変更履歴を表示する |
| `/feedback` | GitHub にフィードバックを送信する |
| `/theme` | ターミナルテーマの表示または設定 |

### クイックシェルコマンド

`!` を先頭に付けることで、AI を経由せずにシェルコマンドを直接実行できます：

```bash
copilot

> !git status
# AI をバイパスして git status を直接実行する

> !python -m pytest tests/
# pytest を直接実行する
```

### モデルの切り替え

Copilot CLI は OpenAI、Anthropic、Google などの複数の AI モデルに対応しています。利用できるモデルはサブスクリプションのレベルとリージョンによって異なります。`/model` を使って利用可能なオプションを確認し、モデルを切り替えましょう：

```bash
copilot
> /model

# 利用可能なモデルを表示して選択します。Sonnet 4.5 を選択します。
```

> 💡 **ヒント**: モデルによって消費する "premium requests" の数が異なります。**1x** と表示されているモデル（Claude Sonnet 4.5 など）は優れたデフォルトの選択肢です。高性能で効率的です。倍率が高いモデルは premium request のクォータをより早く消費するので、本当に必要なときのために取っておきましょう。

</details>

---

# 実践練習

<img src="../images/practice.png" alt="Warm desk setup with monitor showing code, lamp, coffee cup, and headphones ready for hands-on practice" width="800"/>

学んだことを実際に試してみましょう。

---

## ▶️ やってみよう

### Interactive で探索する

Copilot を起動し、フォローアップのプロンプトを使って book app を繰り返し改善してみましょう：

```bash
copilot

> Review @samples/book-app-project/book_app.py - what could be improved?

> Refactor the if/elif chain into a more maintainable structure

> Add type hints to all the handler functions

> /exit
```

### 機能を計画する

`/plan` を使って、コードを書く前に Copilot CLI に実装を計画させてみましょう：

```bash
copilot

> /plan Add a search feature to the book app that can find books by title or author

# 計画をレビューする
# 承認または修正する
# ステップごとに実装される様子を見る
```

### Programmatic Mode で自動化する

`-p` フラグを使うと、Interactive mode に入らずにターミナルから直接 Copilot CLI を実行できます。以下のスクリプトをリポジトリのルートから（Copilot の中ではなく）ターミナルにコピー＆ペーストして、book app の全 Python ファイルをレビューしましょう。

```bash
# book app の全 Python ファイルをレビューする
for file in samples/book-app-project/*.py; do
  echo "Reviewing $file..."
  copilot --allow-all -p "Quick code quality review of @$file - critical issues only"
done
```

**PowerShell (Windows):**

```powershell
# book app の全 Python ファイルをレビューする
Get-ChildItem samples/book-app-project/*.py | ForEach-Object {
  $relativePath = "samples/book-app-project/$($_.Name)";
  Write-Host "Reviewing $relativePath...";
  copilot --allow-all -p "Quick code quality review of @$relativePath - critical issues only" 
}
```

---

デモを完了したら、以下のバリエーションを試してみてください：

1. **Interactive チャレンジ**: `copilot` を起動して book app を探索しましょう。`@samples/book-app-project/books.py` について質問し、3 回連続で改善を依頼してみてください。

2. **Plan Mode チャレンジ**: `/plan Add rating and review features to the book app` を実行してみましょう。計画をよく読んでみてください。理にかなっていますか？

3. **Programmatic チャレンジ**: `copilot --allow-all -p "List all functions in @samples/book-app-project/book_app.py and describe what each does"` を実行してみましょう。一発でうまくいきましたか？

---

## 📝 課題

### メインチャレンジ：Book App のユーティリティを改善する

ハンズオンの例では `book_app.py` のレビューとリファクタリングに焦点を当てました。次は同じスキルを別のファイル `utils.py` で練習しましょう：

1. Interactive セッションを開始する: `copilot`
2. Copilot CLI にファイルの要約を依頼する: `@samples/book-app-project/utils.py What does each function in this file do?`
3. 入力バリデーションの追加を依頼する: "Add validation to `get_user_choice()` so it handles empty input and non-numeric entries"
4. エラーハンドリングの改善を依頼する: "What happens if `get_book_details()` receives an empty string for the title? Add guards for that."
5. docstring を依頼する: "Add a comprehensive docstring to `get_book_details()` with parameter descriptions and return values"
6. プロンプト間でコンテキストがどのように引き継がれるか観察する。各改善が前の改善の上に積み重なっていきます
7. `/exit` で終了する

**成功の基準**: 入力バリデーション、エラーハンドリング、docstring が追加された改善版 `utils.py` が、マルチターンの会話を通じて完成していること。

<details>
<summary>💡 ヒント（クリックで展開）</summary>

**試してみるプロンプト例:**
```bash
> @samples/book-app-project/utils.py What does each function in this file do?
> Add validation to get_user_choice() so it handles empty input and non-numeric entries
> What happens if get_book_details() receives an empty string for the title? Add guards for that.
> Add a comprehensive docstring to get_book_details() with parameter descriptions and return values
```

**よくある問題:**
- Copilot CLI が確認の質問をしてきたら、自然に答えてください
- コンテキストは引き継がれるので、各プロンプトは前のものの続きになります
- やり直したい場合は `/clear` を使ってください

</details>

### ボーナスチャレンジ：モードを比較する

例では `/plan` で検索機能を計画し、`-p` でバッチレビューを行いました。今度は 3 つすべてのモードを 1 つの新しいタスクで試してみましょう：`BookCollection` クラスに `list_by_year()` メソッドを追加します。

1. **Interactive**: `copilot` → ステップバイステップでメソッドの設計と構築を依頼する
2. **Plan**: `/plan Add a list_by_year(start, end) method to BookCollection that filters books by publication year range`
3. **Programmatic**: `copilot --allow-all -p "@samples/book-app-project/books.py Add a list_by_year(start, end) method that returns books published between start and end year inclusive"`

**振り返り**: どのモードが最も自然に感じましたか？それぞれどんな場面で使いますか？

---

<details>
<summary>🔧 <strong>よくあるミスとトラブルシューティング</strong>（クリックで展開）</summary>

### よくあるミス

| ミス | 何が起こるか | 修正方法 |
|---------|--------------|-----|
| `/exit` の代わりに `exit` と入力する | Copilot CLI が "exit" をコマンドではなくプロンプトとして扱う | slash commands は必ず `/` で始まります |
| `-p` をマルチターンの会話に使う | 各 `-p` 呼び出しは独立しており、前の呼び出しの記憶がない | コンテキストを引き継ぐ会話には Interactive mode（`copilot`）を使う |
| `$` や `!` を含むプロンプトをクォートで囲み忘れる | Copilot CLI に渡される前にシェルが特殊文字を解釈してしまう | プロンプトをクォートで囲む: `copilot -p "What does $HOME mean?"` |

### トラブルシューティング

**"Model not available"** - お使いのサブスクリプションにすべてのモデルが含まれていない可能性があります。`/model` で利用可能なモデルを確認してください。

**"Context too long"** - 会話がコンテキストウィンドウの上限に達しました。`/clear` でリセットするか、新しいセッションを開始してください。

**"Rate limit exceeded"** - 数分待ってから再試行してください。バッチ操作には遅延を入れた Programmatic mode の使用を検討してください。

</details>

---

# まとめ

## 🔑 重要ポイント

1. **Interactive mode** は探索と反復のためのモードです。コンテキストが引き継がれます。それまでに話した内容を覚えている相手と会話するようなものです。
2. **Plan mode** は通常、より複雑なタスクに使います。実装前に計画をレビューできます。
3. **Programmatic mode** は自動化のためのモードです。対話は不要です。
4. **4 つの基本コマンド**（`/help`、`/clear`、`/plan`、`/exit`）で日常の大部分をカバーできます。

> 📋 **クイックリファレンス**: [GitHub Copilot CLI command reference](https://docs.github.com/en/copilot/reference/cli-command-reference) でコマンドとショートカットの完全なリストを確認できます。

---

## ➡️ 次のステップ

3 つのモードを理解したところで、次は Copilot CLI にコードのコンテキストを与える方法を学びましょう。

**[Chapter 02: Context and Conversations](../02-context-conversations/README.md)** では、以下を学びます：

- ファイルやディレクトリを参照する `@` 構文
- `--resume` と `--continue` によるセッション管理
- コンテキスト管理が Copilot CLI を真に強力にする理由

---

**[← コースホームに戻る](../README.md)** | **[Chapter 02 へ進む →](../02-context-conversations/README.md)**
