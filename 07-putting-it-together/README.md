![Chapter 07: すべてをまとめよう](images/chapter-header.png)

> **これまで学んだすべてをここで組み合わせます。アイデアからマージされた PR まで、1 回のセッションで完結させましょう。**

この章では、これまで学んできたことをすべて組み合わせて、完全なワークフローを構築します。マルチエージェントのコラボレーションを使って機能を構築し、コミット前にセキュリティの問題を検出する pre-commit フックを設定し、Copilot を CI/CD パイプラインに統合し、機能のアイデアからマージされた PR まで 1 回のターミナルセッションで完了させます。ここが GitHub Copilot CLI が真の力を発揮するところです。

> 💡 **注意**: この章では、これまで学んだことをどう組み合わせるかを紹介します。**生産性を上げるのに agent、skill、MCP は必須ではありません（とても便利ですが）。** 「説明する → 計画する → 実装する → テストする → レビューする → 出荷する」というコアワークフローは、第 00〜03 章の組み込み機能だけで十分に機能します。

## 🎯 学習目標

この章を終えると、以下のことができるようになります：

- agent、skill、MCP（Model Context Protocol）を統合されたワークフローで組み合わせる
- マルチツールアプローチで完全な機能を構築する
- フックを使った基本的な自動化を設定する
- プロフェッショナルな開発のベストプラクティスを適用する

> ⏱️ **所要時間の目安**: 約 75 分（読み物 15 分 + ハンズオン 60 分）

---

## 🧩 身近なたとえ：オーケストラ

<img src="images/orchestra-analogy.png" alt="Orchestra Analogy - Unified Workflow" width="800"/>

交響楽団にはたくさんのセクションがあります：
- **弦楽器** は土台を担います（コアワークフローのようなもの）
- **金管楽器** はパワーを加えます（専門知識を持つ agent のようなもの）
- **木管楽器** は彩りを添えます（機能を拡張する skill のようなもの）
- **打楽器** はリズムを刻みます（外部システムと接続する MCP のようなもの）

個々のセクションだけでは限られた音しか出せません。しかし、うまく指揮をすれば、壮大なものが生まれます。

**この章で学ぶのはまさにそれです！**<br>
*オーケストラの指揮者のように、agent、skill、MCP を統合されたワークフローへとオーケストレーションします*

まず、コードの修正、テストの生成、レビュー、PR の作成をすべて 1 つのセッションで行うシナリオを見ていきましょう。

---

## アイデアからマージ済み PR まで 1 セッションで

エディター、ターミナル、テストランナー、GitHub UI を切り替えるたびにコンテキストを失う代わりに、すべてのツールを 1 つのターミナルセッションで組み合わせることができます。このパターンは後述の[統合パターン](#the-integration-pattern-for-power-users)セクションで詳しく説明します。

```bash
# Copilot をインタラクティブモードで起動する
copilot

> I need to add a "list unread" command to the book app that shows only
> books where read is False. What files need to change?

# Copilot が大まかな計画を作成...

# PYTHON-REVIEWER AGENT に切り替え
> /agent
# "python-reviewer" を選択

> @samples/book-app-project/books.py Design a get_unread_books method.
> What is the best approach?

# python-reviewer agent が以下を出力:
# - メソッドのシグネチャと戻り値の型
# - リスト内包表記を使ったフィルター実装
# - 空のコレクションに対するエッジケースの処理

# PYTEST-HELPER AGENT に切り替え
> /agent
# "pytest-helper" を選択

> @samples/book-app-project/tests/test_books.py Design test cases for
> filtering unread books.

# pytest-helper agent が以下を出力:
# - 空のコレクションのテストケース
# - 既読と未読が混在するテストケース
# - すべての本が既読のテストケース

# 実装
> Add a get_unread_books method to BookCollection in books.py
> Add a "list unread" command option in book_app.py
> Update the help text in the show_help function

# テスト
> Generate comprehensive tests for the new feature

# 以下のようなテストが複数生成される:
# - ハッピーパス（3 テスト）— 正しくフィルター、既読を除外、未読を含む
# - エッジケース（4 テスト）— 空のコレクション、すべて既読、未読なし、本が 1 冊
# - パラメータ化（5 ケース）— @pytest.mark.parametrize で既読/未読の割合を変えてテスト
# - 統合テスト（4 テスト）— mark_as_read、remove_book、add_book との連携とデータの整合性

# 変更をレビュー
> /review

# レビューが通れば、PR を作成（コースで学んだ GitHub MCP を使用）
> Create a pull request titled "Feature: Add list unread books command"
```

**従来のアプローチ**: エディター、ターミナル、テストランナー、ドキュメント、GitHub UI を切り替える。切り替えるたびにコンテキストが失われ、摩擦が生じます。

**重要な気づき**: あなたはアーキテクトのようにスペシャリストに指示を出しました。詳細は彼らが担当し、あなたはビジョンを担当しました。

> 💡 **さらに踏み込む**: このような大規模なマルチステッププランでは、`/fleet` を試して Copilot に独立したサブタスクを並列実行させてみましょう。詳しくは[公式ドキュメント](https://docs.github.com/copilot/concepts/agents/copilot-cli/fleet)をご覧ください。

---

# その他のワークフロー

<img src="images/combined-workflows.png" alt="People assembling a colorful giant jigsaw puzzle with gears, representing how agents, skills, and MCP combine into unified workflows" width="800"/>

第 04〜06 章を修了したパワーユーザーのために、agent、skill、MCP がいかに効果を倍増させるかを示すワークフローを紹介します。

## 統合パターン

すべてを組み合わせるためのメンタルモデルはこちらです：

<img src="images/integration-pattern.png" alt="The Integration Pattern - A 4-phase workflow: Gather Context (MCP), Analyze and Plan (Agents), Execute (Skills + Manual), Complete (MCP)" width="800"/>

---

## ワークフロー 1: バグ調査と修正

実際のバグ修正をツール統合で行います：

```bash
copilot

# フェーズ 1: GitHub からバグの詳細を取得（MCP が提供）
> Get the details of issue #1

# わかったこと: "find_by_author が部分名で動作しない"

# フェーズ 2: ベストプラクティスの調査（Web + GitHub ソースを使った深い調査）
> /research Best practices for Python case-insensitive string matching

# フェーズ 3: 関連コードを探す
> @samples/book-app-project/books.py Show me the find_by_author method

# フェーズ 4: エキスパートの分析を受ける
> /agent
# "python-reviewer" を選択

> Analyze this method for issues with partial name matching

# agent が特定: メソッドが部分文字列マッチングではなく完全一致を使用している

# フェーズ 5: agent のガイダンスに従って修正
> Implement the fix using lowercase comparison and 'in' operator

# フェーズ 6: テストを生成
> /agent
# "pytest-helper" を選択

> Generate pytest tests for find_by_author with partial matches
> Include test cases: partial name, case variations, no matches

# フェーズ 7: コミットと PR
> Generate a commit message for this fix

> Create a pull request linking to issue #1
```

---

## ワークフロー 2: コードレビューの自動化（オプション）

> 💡 **このセクションはオプションです。** pre-commit フックはチームにとって便利ですが、生産性を上げるために必須ではありません。始めたばかりの方はスキップしてください。
>
> ⚠️ **パフォーマンスに関する注意**: このフックはステージングされたファイルごとに `copilot -p` を呼び出すため、ファイルあたり数秒かかります。大きなコミットの場合は、重要なファイルに限定するか、`/review` で手動レビューを行うことを検討してください。

**git フック**とは、Git が特定のタイミングで自動的に実行するスクリプトのことです。例えば、コミットの直前に実行できます。これを使ってコードの自動チェックを行えます。Copilot によるコミットの自動レビューを設定する方法は以下の通りです：

```bash
# pre-commit フックを作成する
cat > .git/hooks/pre-commit << 'EOF'
#!/bin/bash

# ステージングされたファイルを取得（Python ファイルのみ）
STAGED=$(git diff --cached --name-only --diff-filter=ACM | grep -E '\.py$')

if [ -n "$STAGED" ]; then
  echo "Running Copilot review on staged files..."

  for file in $STAGED; do
    echo "Reviewing $file..."

    # ハングを防ぐためにタイムアウトを設定（ファイルあたり 60 秒）
    # --allow-all はファイルの読み書きを自動承認し、フックを無人で実行できるようにします。
    # 自動化スクリプトでのみ使用してください。対話セッションでは Copilot に許可を求めさせましょう。
    REVIEW=$(timeout 60 copilot --allow-all -p "Quick security review of @$file - critical issues only" 2>/dev/null)

    # タイムアウトが発生したか確認
    if [ $? -eq 124 ]; then
      echo "Warning: Review timed out for $file (skipping)"
      continue
    fi

    if echo "$REVIEW" | grep -qi "CRITICAL"; then
      echo "Critical issues found in $file:"
      echo "$REVIEW"
      exit 1
    fi
  done

  echo "Review passed"
fi
EOF

chmod +x .git/hooks/pre-commit
```

> ⚠️ **macOS ユーザーへ**: macOS にはデフォルトで `timeout` コマンドが含まれていません。`brew install coreutils` でインストールするか、`timeout 60` を外してタイムアウトガードなしで実行してください。

> 📚 **公式ドキュメント**: [フックの使用](https://docs.github.com/copilot/how-tos/copilot-cli/use-hooks)および[フック設定リファレンス](https://docs.github.com/copilot/reference/hooks-configuration)で完全なフック API を確認できます。
>
> 💡 **組み込みの代替手段**: Copilot CLI には、pre-commit などのイベントで自動実行できる組み込みフックシステム（`copilot hooks`）もあります。上記の手動 git フックは完全な制御を提供し、組み込みシステムはより簡単に設定できます。どちらのアプローチが合うかは上記のドキュメントを参照してください。

これで、すべてのコミットに簡単なセキュリティレビューが実行されます：

```bash
git add samples/book-app-project/books.py
git commit -m "Update book collection methods"

# 出力:
# Running Copilot review on staged files...
# Reviewing samples/book-app-project/books.py...
# Critical issues found in samples/book-app-project/books.py:
# - Line 15: File path injection vulnerability in load_from_file
#
# 問題を修正してもう一度試してください。
```

---

## ワークフロー 3: 新しいコードベースへのオンボーディング

新しいプロジェクトに参加するとき、コンテキスト、agent、MCP を組み合わせて素早くキャッチアップしましょう：

```bash
# Copilot をインタラクティブモードで起動
copilot

# フェーズ 1: コンテキストで全体像を把握する
> @samples/book-app-project/ Explain the high-level architecture of this codebase

# フェーズ 2: 特定のフローを理解する
> @samples/book-app-project/book_app.py Walk me through what happens
> when a user runs "python book_app.py add"

# フェーズ 3: agent でエキスパートの分析を受ける
> /agent
# "python-reviewer" を選択

> @samples/book-app-project/books.py Are there any design issues,
> missing error handling, or improvements you would recommend?

# フェーズ 4: 取り組むべきものを見つける（MCP が GitHub アクセスを提供）
> List open issues labeled "good first issue"

# フェーズ 5: 貢献を始める
> Pick the simplest open issue and outline a plan to fix it
```

このワークフローは `@` コンテキスト、agent、MCP を 1 回のオンボーディングセッションに組み合わせています。まさにこの章で先ほど紹介した統合パターンそのものです。

---

# ベストプラクティスと自動化

ワークフローをより効果的にするパターンと習慣を紹介します。

---

## ベストプラクティス

### 1. 分析の前にまずコンテキストを集める

分析を依頼する前に、必ずコンテキストを収集しましょう：

```bash
# 良い例
> Get the details of issue #42
> /agent
# python-reviewer を選択
> Analyze this issue

# あまり効果的でない例
> /agent
# python-reviewer を選択
> Fix login bug
# agent が issue のコンテキストを持っていない
```

### 2. agent、skill、カスタムインストラクションの違いを知る

それぞれのツールには得意分野があります：

```bash
# agent: 明示的に有効化する専門ペルソナ
> /agent
# python-reviewer を選択
> Review this authentication code for security issues

# skill: プロンプトが skill の説明文にマッチすると自動的に有効化されるモジュール型機能
#（事前に作成が必要 — 第 05 章参照）
> Generate comprehensive tests for this code
# テスト用の skill が設定されていれば、自動的に有効化される

# カスタムインストラクション (.github/copilot-instructions.md): 
# 切り替えやトリガーなしに、すべてのセッションに適用される常時有効なガイダンス
```

> 💡 **重要ポイント**: agent と skill はどちらもコードの分析と生成ができます。本当の違いは**有効化の方法**です — agent は明示的（`/agent`）、skill は自動的（プロンプトマッチ）、カスタムインストラクションは常時有効です。

### 3. セッションを集中させる

`/rename` でセッションにラベルを付け（履歴で見つけやすくなります）、`/exit` できれいに終了しましょう：

```bash
# 良い例: 1 セッションに 1 機能
> /rename list-unread-feature
# list unread の作業
> /exit

copilot
> /rename export-csv-feature
# CSV エクスポートの作業
> /exit

# あまり効果的でない例: すべてを 1 つの長いセッションで
```

### 4. Copilot でワークフローを再利用可能にする

ワークフローを Wiki にドキュメント化するだけでなく、Copilot が使えるようにリポジトリに直接エンコードしましょう：

- **カスタムインストラクション** (`.github/copilot-instructions.md`): コーディング規約、アーキテクチャルール、ビルド/テスト/デプロイの手順のための常時有効なガイダンス。すべてのセッションが自動的にこれに従います。
- **プロンプトファイル** (`.github/prompts/`): チームで共有できる再利用可能なパラメータ化されたプロンプト — コードレビュー、コンポーネント生成、PR の説明文のテンプレートのようなものです。
- **カスタム agent** (`.github/agents/`): チームの誰もが `/agent` で有効化できる専門ペルソナ（例：セキュリティレビュアーやドキュメントライター）をエンコードします。
- **カスタム skill** (`.github/skills/`): 関連するときに自動的に有効化される、ステップバイステップのワークフロー手順のパッケージです。

> 💡 **メリット**: 新しいチームメンバーがワークフローを無料で手に入れられます — 誰かの頭の中ではなく、リポジトリに組み込まれているからです。

---

## ボーナス: プロダクションパターン

以下のパターンはオプションですが、プロフェッショナルな環境で役立ちます。

### PR 説明文ジェネレーター

```bash
# 包括的な PR 説明文を生成する
BRANCH=$(git branch --show-current)
COMMITS=$(git log main..$BRANCH --oneline)

copilot -p "Generate a PR description for:
Branch: $BRANCH
Commits:
$COMMITS

Include: Summary, Changes Made, Testing Done, Screenshots Needed"
```

### CI/CD 統合

既存の CI/CD パイプラインを持つチーム向けに、GitHub Actions を使ってすべての Pull Request で Copilot レビューを自動化できます。レビューコメントの自動投稿やクリティカルな問題のフィルタリングも含まれます。

> 📖 **詳しくはこちら**: 完全な GitHub Actions ワークフロー、設定オプション、トラブルシューティングのヒントについては [CI/CD 統合](../appendices/ci-cd-integration.md) をご覧ください。

---

# 実践練習

<img src="../images/practice.png" alt="Warm desk setup with monitor showing code, lamp, coffee cup, and headphones ready for hands-on practice" width="800"/>

完全なワークフローを実践しましょう。

---

## ▶️ 自分で試してみよう

デモを完了したら、以下のバリエーションを試してみてください：

1. **エンドツーエンドチャレンジ**: 小さな機能（例：「未読の本を一覧表示」や「CSV にエクスポート」）を選びましょう。完全なワークフローを使います：
   - `/plan` で計画する
   - agent（python-reviewer、pytest-helper）で設計する
   - 実装する
   - テストを生成する
   - PR を作成する

2. **自動化チャレンジ**: コードレビュー自動化ワークフローの pre-commit フックを設定しましょう。意図的にファイルパスの脆弱性を含むコミットをしてみてください。ブロックされますか？

3. **あなたのプロダクションワークフロー**: よく行うタスク用の独自ワークフローを設計しましょう。チェックリストとして書き出してください。skill、agent、フックで自動化できる部分はどこですか？

**セルフチェック**: agent、skill、MCP がどのように連携するか、そしてそれぞれをいつ使うべきかを同僚に説明できれば、コースは修了です。

---

## 📝 課題

### メインチャレンジ: エンドツーエンドの機能開発

ハンズオンの例では「未読の本を一覧表示」機能の構築を見てきました。今度は別の機能で完全なワークフローを練習しましょう：**出版年の範囲で本を検索する**：

1. Copilot を起動してコンテキストを収集する: `@samples/book-app-project/books.py`
2. `/plan Add a "search by year" command that lets users find books published between two years` で計画する
3. `BookCollection` に `find_by_year_range(start_year, end_year)` メソッドを実装する
4. `book_app.py` に開始年と終了年を入力させる `handle_search_year()` 関数を追加する
5. テストを生成する: `@samples/book-app-project/books.py @samples/book-app-project/tests/test_books.py Generate tests for find_by_year_range() including edge cases like invalid years, reversed range, and no results.`
6. `/review` でレビューする
7. README を更新する: `@samples/book-app-project/README.md Add documentation for the new "search by year" command.`
8. コミットメッセージを生成する

進めながらワークフローをドキュメント化してください。

**成功基準**: Copilot CLI を使って、計画、実装、テスト、ドキュメント、レビューを含め、アイデアからコミットまで機能を完成させること。

> 💡 **ボーナス**: 第 04 章で agent を設定した方は、カスタム agent の作成と使用を試してみましょう。例えば、実装レビュー用の error-handler agent や README 更新用の doc-writer agent などです。

<details>
<summary>💡 ヒント（クリックで展開）</summary>

**この章の冒頭にある[「アイデアからマージ済み PR まで」](#アイデアからマージ済み-pr-まで-1-セッションで)の例のパターンに従いましょう。** 主なステップは以下の通りです：

1. `@samples/book-app-project/books.py` でコンテキストを収集
2. `/plan Add a "search by year" command` で計画
3. メソッドとコマンドハンドラを実装
4. エッジケース付きのテストを生成（無効な入力、結果なし、範囲の逆転）
5. `/review` でレビュー
6. `@samples/book-app-project/README.md` で README を更新
7. `-p` でコミットメッセージを生成

**考えるべきエッジケース：**
- ユーザーが "2000" と "1990" を入力した場合（範囲が逆）は？
- 範囲に該当する本がない場合は？
- ユーザーが数値以外を入力した場合は？

**重要なのはアイデア → コンテキスト → 計画 → 実装 → テスト → ドキュメント → コミットという完全なワークフローを練習すること**です。

</details>

---

<details>
<summary>🔧 <strong>よくある間違い</strong>（クリックで展開）</summary>

| 間違い | 起こること | 修正方法 |
|---------|-----------|----------|
| いきなり実装に飛びつく | 後で修正コストが高い設計上の問題を見逃す | まず `/plan` で考え方を整理する |
| 複数のツールが役立つのに 1 つだけ使う | 遅く、網羅性に欠ける結果になる | 組み合わせる: agent で分析 → skill で実行 → MCP で統合 |
| コミット前にレビューしない | セキュリティの問題やバグがすり抜ける | 常に `/review` を実行するか [pre-commit フック](#ワークフロー-2-コードレビューの自動化オプション)を使う |
| チームとワークフローを共有しない | 各メンバーが車輪の再発明をする | 共有の agent、skill、インストラクションにパターンをドキュメント化する |

</details>

---

# まとめ

## 🔑 重要ポイント

1. **統合 > 孤立**: ツールを組み合わせることでインパクトを最大化する
2. **まずコンテキスト**: 分析の前に必ず必要なコンテキストを収集する
3. **agent は分析、skill は実行**: 適切なツールを適切な場面で使う
4. **繰り返しを自動化する**: フックとスクリプトで効果を何倍にもする
5. **ワークフローをドキュメント化する**: 共有可能なパターンはチーム全体に恩恵をもたらす

> 📋 **クイックリファレンス**: コマンドとショートカットの完全なリストは [GitHub Copilot CLI コマンドリファレンス](https://docs.github.com/en/copilot/reference/cli-command-reference) をご覧ください。

---

## 🎓 コース修了！

おめでとうございます！以下のことを学びました：

| 章 | 学んだこと |
|-----|-----------|
| 00 | Copilot CLI のインストールとクイックスタート |
| 01 | 3 つのインタラクションモード |
| 02 | @ 構文によるコンテキスト管理 |
| 03 | 開発ワークフロー |
| 04 | 専門化された agent |
| 05 | 拡張可能な skill |
| 06 | MCP による外部接続 |
| 07 | 統合されたプロダクションワークフロー |

これで GitHub Copilot CLI を開発ワークフローにおける真の力の増幅器として使う準備が整いました。

## ➡️ 次のステップ

学びはここで終わりではありません：

1. **毎日使う**: 実際の作業で Copilot CLI を使いましょう
2. **カスタムツールを構築する**: 自分のニーズに合った agent と skill を作りましょう
3. **知識を共有する**: チームがこれらのワークフローを導入できるよう支援しましょう
4. **最新情報をフォローする**: GitHub Copilot のアップデートで新機能をチェックしましょう

### リソース

- [GitHub Copilot CLI ドキュメント](https://docs.github.com/copilot/concepts/agents/about-copilot-cli)
- [MCP Server Registry](https://github.com/modelcontextprotocol/servers)
- [コミュニティ Skills](https://github.com/topics/copilot-skill)

---

**お疲れ様でした！さあ、素晴らしいものを作りましょう。**

**[← 第 06 章に戻る](../06-mcp-servers/README.md)** | **[コースホームに戻る →](../README.md)**
