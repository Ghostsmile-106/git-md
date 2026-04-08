# Git × GitHub CLI セットアップガイド
## gh コマンドを使った最短セットアップ手順

---

## はじめに

このガイドでは GitHub公式CLIツール **`gh`** を使った方法で、ローカルリポジトリの作成からGitHubへの公開まで解説します。`gh` を使うことで、ブラウザでGitHubを開く手間なく、ターミナルだけで完結できます。

---

## STEP 1 — GitHub CLI（gh）をインストールする

### なぜ必要か

`gh` はGitHub公式のCLIツールです。通常はブラウザで行う「リポジトリの作成」「PR作成」などをターミナルから実行できます。Git本体とは別のツールなので、別途インストールが必要です。

### コマンド

```bash
# macOS（Homebrew）
brew install gh

# Windows（winget）
winget install --id GitHub.cli

# Ubuntu / Debian
sudo apt install gh
```

### インストール確認

```bash
gh --version
# 例: gh version 2.x.x
```

---

## STEP 2 — gh でGitHubにログインする

### なぜ必要か

`gh` コマンドがGitHubのAPIを操作するために、あなたのGitHubアカウントと紐づける必要があります。ここで認証しておくことで、以降のコマンドでブラウザやパスワードの入力が不要になります。

### コマンド

```bash
gh auth login
```

### 対話プロンプトへの回答例

```
? What account do you want to log into?
  → GitHub.com を選択

? What is your preferred protocol for Git operations?
  → SSH を選択

? Generate a new SSH key to add to your GitHub account?
  → Yes を選択（SSH鍵をまだ作っていない場合）

? Enter a passphrase for your new SSH key (Optional):
  → パスフレーズを設定（推奨）

? How would you like to authenticate GitHub CLI?
  → Login with a web browser を選択

→ ブラウザが開くので、表示されたコードを入力して認証
```

### ポイント

- **SSH鍵の生成も `gh auth login` が自動でやってくれます。** 別途 `ssh-keygen` を実行する必要はありません
- 生成された公開鍵はGitHubアカウントに自動登録されます
- すでにSSH鍵がある場合は既存の鍵を使うかどうか選択できます
- 認証状態の確認：`gh auth status`

---

## STEP 3 — ユーザー情報を設定する

### なぜ必要か

Gitはコミット（変更の記録）に「誰がいつ変更したか」を必ず記録します。`user.name` と `user.email` を設定しないと、コミットに不正確な情報が記録されてしまいます。GitHubはメールアドレスでアカウントと紐づけるため、正しいアドレスの設定が重要です。

### コマンド

```bash
# このリポジトリ専用の設定（複数アカウントを使い分ける場合に推奨）
git config user.name  "Your Name"
git config user.email "you@example.com"

# 全リポジトリ共通にしたい場合は --global を追加
git config --global user.name  "Your Name"
git config --global user.email "you@example.com"

# 設定を確認
git config --list
```

### `--global` との使い分け

| オプション | 適用範囲 | 向いている場面 |
|---|---|---|
| なし（ローカル） | 現在のリポジトリのみ | 仕事用・個人用でアカウントを使い分ける場合 |
| `--global` | そのPCの全リポジトリ | 1つのアカウントしか使わない場合 |

---

## STEP 4 — ローカルリポジトリを作成する

### なぜ必要か

`git init` は、プロジェクトフォルダをGitの管理下に置くための最初の一歩です。実行すると `.git/` という隠しディレクトリが作られ、変更履歴・ブランチ情報・設定などがすべてここに保存されます。

### コマンド

```bash
mkdir my-project && cd my-project
# プロジェクトフォルダを作成して移動

git init
# Gitリポジトリを初期化
```

### 実行結果

```
Initialized empty Git repository in /path/to/my-project/.git/
```

---

## STEP 5 — ファイルを追加してコミットする

### なぜ必要か

`gh repo create` でGitHubにプッシュするには、**最低1つのコミットが必要**です。コミットがない状態ではプッシュするものがないためエラーになります。`git add` でファイルをステージング（記録対象として登録）し、`git commit` で変更を確定します。

### コマンド

```bash
# ファイルを作成
echo "# my-project" >> README.md

# すべての変更をステージング
git add .

# コミット（変更を記録）
git commit -m "Initial commit"
```

### コミットメッセージについて

コミットメッセージは「何をしたか」が一目でわかる内容にするのがベストプラクティスです。

```bash
# 悪い例
git commit -m "fix"
git commit -m "update"

# 良い例
git commit -m "Add README"
git commit -m "Fix login bug"
```

---

## STEP 6 — gh でGitHubにリポジトリを作成してプッシュする

### なぜ必要か

ここが `gh` を使う最大のメリットです。通常であれば「ブラウザでGitHubを開く → リポジトリを作成 → URLをコピー → `git remote add` で登録 → `git push`」という手順が必要ですが、`gh repo create` を使うとこれをすべて1コマンドで完結できます。

### コマンド

```bash
gh repo create <GitHubユーザー名>/<リポジトリ名> --public --source=. --remote=origin --push
```

### 実際の例

```bash
gh repo create Ghostsmile-106/my-project --public --source=. --remote=origin --push
```

### オプションの意味

| オプション | 意味 |
|---|---|
| `--public` | リポジトリを公開にする（非公開にする場合は `--private`） |
| `--source=.` | 現在のディレクトリをソースとして使う |
| `--remote=origin` | リモート名を `origin` として登録する |
| `--push` | 作成後すぐにプッシュする |

### 実行結果

```
✓ Created repository Ghostsmile-106/my-project on GitHub
✓ Added remote origin
✓ Pushed commits to git@github.com:Ghostsmile-106/my-project.git
```

---

## 補足 — 複数アカウントの使い分け（社用PC・自宅PCなど）

### SSH鍵はPCごとに発行する

SSH鍵はそのPCにしか存在しないため、**別のPCからアクセスする場合はそのPCで再度 `gh auth login` を実行**します。GitHubは1アカウントに複数の公開鍵を登録できるので、社用PCと自宅PCの両方の鍵を登録しておけばどちらからもアクセス可能です。

```
社用PC  → gh auth login → 公開鍵A をGitHubに登録
自宅PC  → gh auth login → 公開鍵B をGitHubに登録（追加）
→ どちらのPCからもアクセス可能
```

### 仕事用・個人用でアカウントが異なる場合

`~/.ssh/config` を使って、接続先ごとに鍵を切り替えられます。

```
# ~/.ssh/config

# 仕事用
Host github-work
  HostName github.com
  User git
  IdentityFile ~/.ssh/id_ed25519_work

# 個人用
Host github-personal
  HostName github.com
  User git
  IdentityFile ~/.ssh/id_ed25519_personal
```

---

## まとめ — 全手順の流れ

```
1. gh のインストール
         ↓
2. gh auth login        ← GitHubと紐づけ（SSH鍵生成も自動）
         ↓
3. git config user.*    ← コミット情報の設定
         ↓
4. git init             ← ローカルリポジトリの初期化
         ↓
5. git add / commit     ← 最初のコミット
         ↓
6. gh repo create ... --push  ← GitHub上にリポジトリ作成 ＋ プッシュ（1コマンドで完結）
```

2回目以降の日常的な操作は `git add / commit / push` だけで開発を進められます。
