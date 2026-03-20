![Chapter 00: クイックスタート](images/chapter-header.png)

ようこそ！この章では、GitHub Copilot CLI（コマンドラインインターフェース）のインストール、GitHubアカウントでのサインイン、そしてすべてが正しく動作することの確認を行います。これはセットアップのための短い章です。準備が整ったら、Chapter 01 から本格的なデモが始まります！

## 🎯 学習目標

この章を終えると、以下のことが完了しています：

- GitHub Copilot CLI のインストール
- GitHub アカウントでのサインイン
- 簡単なテストによる動作確認

> ⏱️ **所要時間の目安**: 約10分（読む時間5分 + ハンズオン5分）

---

## ✅ 前提条件

- Copilot にアクセスできる **GitHub アカウント**。[サブスクリプションの選択肢を確認](https://github.com/features/copilot/plans)。学生・教員は [GitHub Education](https://education.github.com/pack) から Copilot Pro を無料で利用できます。
- **ターミナルの基本操作**: `cd` や `ls` などのコマンドに慣れていること

### 「Copilot アクセス」とは

GitHub Copilot CLI には有効な Copilot サブスクリプションが必要です。[github.com/settings/copilot](https://github.com/settings/copilot) でステータスを確認できます。以下のいずれかが表示されるはずです：

- **Copilot Individual** - 個人サブスクリプション
- **Copilot Business** - 組織経由
- **Copilot Enterprise** - エンタープライズ経由
- **GitHub Education** - 認証済み学生・教員向け無料プラン

「You don't have access to GitHub Copilot」と表示される場合は、無料オプションを利用するか、プランに加入するか、アクセスを提供している組織に参加する必要があります。

---

## インストール

> ⏱️ **所要時間の目安**: インストールに2〜5分、認証にさらに1〜2分かかります。

### 推奨: GitHub Codespaces（セットアップ不要）

前提条件を何もインストールしたくない場合は、GitHub Codespaces を使用できます。GitHub Copilot CLI がすぐに使える状態で（サインインは必要）、Python 3.13、pytest、GitHub CLI がプリインストールされています。

1. [このリポジトリをフォーク](https://github.com/github/copilot-cli-for-beginners/fork)して自分の GitHub アカウントに追加
2. **Code** > **Codespaces** > **Create codespace on main** を選択
3. コンテナのビルドが完了するまで数分待機
4. 準備完了！Codespace 環境でターミナルが自動的に開きます。

> 💡 **Codespace での確認**: `cd samples/book-app-project && python book_app.py help` を実行して、Python とサンプルアプリが動作することを確認してください。

### 代替手段: ローカルインストール

> 💡 **どれを選べばいい？** Node.js がインストールされていれば `npm` を使いましょう。それ以外の場合は、お使いのシステムに合った方法を選んでください。

> 💡 **デモには Python が必要です**: このコースでは Python のサンプルアプリを使用します。ローカルで作業する場合は、デモを始める前に [Python 3.10+](https://www.python.org/downloads/) をインストールしてください。

> **注意:** コース全体を通して使用する主なサンプルは Python（`samples/book-app-project`）ですが、JavaScript（`samples/book-app-project-js`）と C#（`samples/book-app-project-cs`）のバージョンも用意されています。各サンプルには、その言語でアプリを実行するための手順が書かれた README があります。

お使いのシステムに合った方法を選んでください：

### 全プラットフォーム (npm)

```bash
# Node.js がインストールされていれば、これが最も手軽な方法です
npm install -g @github/copilot
```

### macOS/Linux (Homebrew)

```bash
brew install copilot-cli
```

### Windows (WinGet)

```bash
winget install GitHub.Copilot
```

### macOS/Linux (インストールスクリプト)

```bash
curl -fsSL https://gh.io/copilot-install | bash
```

---

## 認証

`copilot-cli-for-beginners` リポジトリのルートでターミナルウィンドウを開き、CLI を起動してフォルダへのアクセスを許可します。

```bash
copilot
```

リポジトリが含まれるフォルダを信頼するかどうか尋ねられます（まだ信頼していない場合）。1回だけ信頼するか、今後のすべてのセッションで信頼するかを選べます。

<img src="images/copilot-trust.png" alt="Copilot CLI でフォルダ内のファイルを信頼する" width="800"/>

フォルダを信頼した後、GitHub アカウントでサインインできます。

```
> /login
```

**次に起こること:**

1. Copilot CLI がワンタイムコード（例: `ABCD-1234`）を表示します
2. ブラウザで GitHub のデバイス認証ページが開きます。まだサインインしていなければ GitHub にサインインしてください。
3. プロンプトが表示されたらコードを入力します
4. 「Authorize」を選択して GitHub Copilot CLI にアクセスを許可します
5. ターミナルに戻ります - サインイン完了です！

<img src="images/auth-device-flow.png" alt="デバイス認証フロー - ターミナルでのログインからサインイン確認までの5ステッププロセス" width="800"/>

*デバイス認証フロー: ターミナルでコードが生成され、ブラウザで確認し、Copilot CLI が認証されます。*

**ヒント**: サインインはセッション間で保持されます。トークンが期限切れになったり、明示的にサインアウトしない限り、この操作は1回だけで済みます。

---

## 動作確認

### ステップ 1: Copilot CLI のテスト

サインインが完了したので、Copilot CLI が正しく動作するか確認しましょう。ターミナルで CLI を起動します（まだの場合）：

```bash
> Say hello and tell me what you can help with
```

応答を受け取ったら、CLI を終了できます：

```bash
> /exit
```

---

<details>
<summary>🎬 実際の動作を見てみましょう！</summary>

![Hello Demo](images/hello-demo.gif)

*デモの出力は異なる場合があります。使用するモデル、ツール、応答はここに表示されているものとは異なります。*

</details>

---

**期待される出力**: Copilot CLI の機能を一覧するフレンドリーな応答。

### ステップ 2: サンプルブックアプリの実行

このコースでは、CLI を使ってコース全体を通して探索・改善していくサンプルアプリが用意されています *(コードは /samples/book-app-project にあります)*。始める前に、*Python 蔵書コレクションターミナルアプリ* が動作することを確認しましょう。お使いのシステムに応じて `python` または `python3` を実行してください。

> **注意:** コース全体を通して使用する主なサンプルは Python（`samples/book-app-project`）ですが、JavaScript（`samples/book-app-project-js`）と C#（`samples/book-app-project-cs`）のバージョンも用意されています。各サンプルには、その言語でアプリを実行するための手順が書かれた README があります。

```bash
cd samples/book-app-project
python book_app.py list
```

**期待される出力**: "The Hobbit"、"1984"、"Dune" を含む5冊の本のリスト。

### ステップ 3: ブックアプリで Copilot CLI を試す

まずリポジトリのルートに戻ります（ステップ 2 を実行した場合）：

```bash
cd ../..   # 必要に応じてリポジトリのルートに戻る
copilot 
> What does @samples/book-app-project/book_app.py do?
```

**期待される出力**: ブックアプリの主要な機能とコマンドの要約。

エラーが表示された場合は、下の[トラブルシューティング](#トラブルシューティング)セクションを確認してください。

完了したら Copilot CLI を終了できます：

```bash
> /exit
```

---

## ✅ 準備完了！

インストールはこれで完了です。Chapter 01 から本格的な内容が始まります：

- AI がブックアプリをレビューし、コード品質の問題を即座に発見する様子を見る
- Copilot CLI の3つの使い方を学ぶ
- 自然な日本語や英語から動作するコードを生成する

**[Chapter 01: はじめの一歩へ進む →](../01-setup-and-first-steps/README.md)**

---

## トラブルシューティング

### 「copilot: command not found」

CLI がインストールされていません。別のインストール方法を試してください：

```bash
# brew が失敗した場合は npm を試す:
npm install -g @github/copilot

# またはインストールスクリプト:
curl -fsSL https://gh.io/copilot-install | bash
```

### 「You don't have access to GitHub Copilot」

1. [github.com/settings/copilot](https://github.com/settings/copilot) で Copilot サブスクリプションがあることを確認
2. 職場アカウントを使用している場合は、組織が CLI アクセスを許可しているか確認

### 「Authentication failed」

再認証を行います：

```bash
copilot
> /login
```

### ブラウザが自動的に開かない場合

[github.com/login/device](https://github.com/login/device) に手動でアクセスし、ターミナルに表示されたコードを入力してください。

### トークンの期限切れ

`/login` を再度実行するだけです：

```bash
copilot
> /login
```

### それでも解決しない場合

- [GitHub Copilot CLI ドキュメント](https://docs.github.com/copilot/concepts/agents/about-copilot-cli)を確認
- [GitHub Issues](https://github.com/github/copilot-cli/issues) を検索

---

## 🔑 重要ポイント

1. **GitHub Codespace はすぐに始められる方法です** - Python、pytest、GitHub Copilot CLI がすべてプリインストールされているので、すぐにデモに取りかかれます
2. **複数のインストール方法** - お使いのシステムに合ったものを選択（Homebrew、WinGet、npm、またはインストールスクリプト）
3. **認証は1回だけ** - トークンが期限切れになるまでログインは保持されます
4. **ブックアプリは動作確認済み** - コース全体を通して `samples/book-app-project` を使用します

> 📚 **公式ドキュメント**: インストールオプションと要件については [Copilot CLI のインストール](https://docs.github.com/copilot/how-tos/copilot-cli/cli-getting-started) を参照してください。

> 📋 **クイックリファレンス**: コマンドとショートカットの完全なリストは [GitHub Copilot CLI コマンドリファレンス](https://docs.github.com/en/copilot/reference/cli-command-reference) を参照してください。

---

**[Chapter 01: はじめの一歩へ進む →](../01-setup-and-first-steps/README.md)**
