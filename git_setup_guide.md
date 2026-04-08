# Git セットアップ完全ガイド
## リポジトリ作成からSSH設定まで

---

## はじめに

Gitを使い始めるには、いくつかの初期設定が必要です。このガイドでは「なぜその設定が必要なのか」という理由とともに、一連の手順をわかりやすく解説します。

---

## STEP 1 — ローカルリポジトリを作成する

### なぜ必要か

`git init` は、プロジェクトフォルダをGitの管理下に置くための最初の一歩です。このコマンドを実行することで、フォルダ内に `.git/` という隠しディレクトリが作られ、変更履歴・ブランチ情報・設定などがすべてここに保存されます。これがないとGitの機能を一切使えません。

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

### ポイント

- 既存フォルダに対して実行しても問題ありません（上書きされません）
- `.git/` を削除するとGit管理が完全に消えます（通常は削除しないこと）

---

## STEP 2 — ユーザー情報を設定する

### なぜ必要か

Gitはコミット（変更の記録）に「誰がいつ変更したか」を必ず記録します。`user.name` と `user.email` を設定しないと、コミットに不正確な情報が記録されてしまいます。チームで開発する場合、この情報がないと誰の変更かわからなくなります。

また、GitHubなどのサービスでは **メールアドレスでアカウントと紐づけ** るため、正しいメールアドレスの設定が重要です。

### コマンド

```bash
# このリポジトリ専用の設定（推奨）
git config user.name "Your Name"
git config user.email "you@example.com"

# 設定を確認する
git config --list
```

### `--global` との違い

| オプション | 適用範囲 | 用途 |
|---|---|---|
| なし（ローカル） | 現在のリポジトリのみ | 仕事用と個人用を使い分けたい場合 |
| `--global` | そのPCの全リポジトリ | 1つのアカウントしか使わない場合 |

```bash
# 全リポジトリ共通にしたい場合
git config --global user.name "Your Name"
git config --global user.email "you@example.com"
```

### ポイント

- 仕事用と個人用でGitHubアカウントが別の場合は、`--global` を使わずリポジトリごとに設定するのがベストプラクティスです
- `git config --list --show-origin` でどのファイルから設定が読み込まれているか確認できます

---

## STEP 3 — SSH鍵を生成する

### なぜSSHが必要か

GitHubへのアクセスには **HTTPS** と **SSH** の2つの方式があります。

| 方式 | 認証 | 特徴 |
|---|---|---|
| HTTPS | ユーザー名＋パスワード（またはトークン） | 設定が簡単だが、毎回認証が必要になることがある |
| SSH | 公開鍵暗号方式 | 一度設定すれば自動認証。パスワード不要でセキュア |

**SSH（Secure Shell）は公開鍵暗号を使った認証方式**です。「秘密鍵」と「公開鍵」のペアを作り、公開鍵だけをGitHubに登録します。GitHubはあなたの公開鍵で接続を検証するため、パスワードを送信する必要がなく、**フィッシングやパスワード漏洩のリスクがありません**。

特にCI/CDパイプラインや自動化スクリプトでは、パスワード入力を求められないSSHが事実上必須です。

### コマンド

```bash
ssh-keygen -t ed25519 -C "you@example.com"
# Ed25519アルゴリズムで鍵を生成
```

#### 対話プロンプトへの回答

```
Enter file in which to save the key: （Enterで ~/.ssh/id_ed25519 に保存）
Enter passphrase: （パスフレーズを設定。空白でもOKだが設定を推奨）
Enter same passphrase again: （同じパスフレーズをもう一度）
```

### アルゴリズムについて

| アルゴリズム | 特徴 |
|---|---|
| `ed25519` | 現在の推奨。高速・安全・鍵が短い |
| `rsa -b 4096` | 古い環境との互換性が必要な場合に使用 |

### 生成されるファイル

```
~/.ssh/id_ed25519      # 秘密鍵（絶対に外部に漏らさないこと）
~/.ssh/id_ed25519.pub  # 公開鍵（GitHubに登録する）
```

### ポイント

- **秘密鍵（`.ssh/id_ed25519`）は絶対に共有しない**こと。これが漏れると第三者があなたとして認証できてしまいます
- パスフレーズを設定すると、秘密鍵ファイルが盗まれても悪用されにくくなります

---

## STEP 4 — ssh-agentに鍵を登録する

### なぜ必要か

パスフレーズを設定した場合、SSH接続のたびにパスフレーズの入力が求められます。これは安全ですが不便です。**ssh-agentは秘密鍵をメモリ上に一時保管するバックグラウンドプロセス**で、一度パスフレーズを入力すれば、セッション中は自動的に鍵を使ってくれます。

パスフレーズなしで鍵を作成した場合でも、ssh-agentに登録しておくとSSH設定の管理が一元化されて便利です。

### コマンド

```bash
# ssh-agentを起動
eval "$(ssh-agent -s)"

# 秘密鍵をssh-agentに登録
ssh-add ~/.ssh/id_ed25519
```

### 実行結果

```
Agent pid 12345
Identity added: /Users/you/.ssh/id_ed25519
```

### ポイント

- macOSでは `~/.ssh/config` に `AddKeysToAgent yes` を追記すると、起動時に自動登録されます
- `ssh-add -l` で登録済みの鍵一覧を確認できます

---

## STEP 5 — 公開鍵をGitHubに登録する

### なぜ必要か

SSH認証では、**サーバー（GitHub）があなたの公開鍵を知っている必要があります**。あなたが接続してきたとき、GitHubは「この接続は登録済みの公開鍵と対応する秘密鍵を持っているか」を検証します。公開鍵が登録されていないと、認証に失敗して接続が拒否されます。

### 手順

**1. 公開鍵をコピーする**

```bash
cat ~/.ssh/id_ed25519.pub
# 出力された内容をすべてコピー
```

**2. GitHubに登録する**

1. GitHub → 右上のアイコン → **Settings**
2. 左メニュー → **SSH and GPG keys**
3. **New SSH key** をクリック
4. Title（わかりやすい名前）と Key（コピーした公開鍵）を貼り付けて **Add SSH key**

**3. 接続テスト**

```bash
ssh -T git@github.com
```

### 実行結果

```
Hi username! You've successfully authenticated, but GitHub does not provide shell access.
```

このメッセージが表示されれば設定成功です。

### ポイント

- `Permission denied (publickey)` と表示された場合は、鍵の登録が正しくできていないかssh-agentに登録されていません
- `ssh -vT git@github.com` で詳細なデバッグ情報を確認できます

---

## STEP 6 — リモートリポジトリと紐づける

### なぜ必要か

ローカルのGitリポジトリは、そのままでは自分のPC内にしか存在しません。**リモートリポジトリ（GitHubなど）と紐づけることで、変更をクラウドに保存したり、他の人と共有したりできるようになります。**

`origin` は慣習的に使われるリモートの名前です（変更可能ですが、originが標準）。

### コマンド

```bash
# GitHubでリポジトリを作成後、SSHのURLを使って登録
git remote add origin git@github.com:username/my-project.git

# 登録内容を確認
git remote -v
```

### 実行結果

```
origin  git@github.com:username/my-project.git (fetch)
origin  git@github.com:username/my-project.git (push)
```

### HTTPSとSSHのURL比較

| 方式 | URL形式 |
|---|---|
| HTTPS | `https://github.com/username/my-project.git` |
| SSH | `git@github.com:username/my-project.git` |

SSH設定が済んでいるので、必ず **SSH形式のURL** を使いましょう。

### ポイント

- すでに登録済みのリモートURLを変更したい場合：`git remote set-url origin <新しいURL>`
- リモートの削除：`git remote remove origin`

---

## STEP 7 — 最初のコミットとプッシュ

### なぜ必要か

リモートリポジトリを作っただけでは中身がありません。ローカルの変更を `git add` でステージングし、`git commit` で記録し、`git push` でリモートに送信することで、はじめてGitHubに反映されます。

初回は `-u` オプション（`--set-upstream`）をつけることで、以降は `git push` だけで同じリモートブランチに送信できるようになります。

### コマンド

```bash
# ファイルを作成
echo "# my-project" >> README.md

# すべての変更をステージング
git add .

# コミット（変更を記録）
git commit -m "Initial commit"

# デフォルトブランチをmainに設定
git branch -M main

# リモートにプッシュ（初回は -u でトラッキングを設定）
git push -u origin main
```

### コミットメッセージについて

良いコミットメッセージは変更の「何を」「なぜ」を伝えます。

```
# 悪い例
git commit -m "fix"
git commit -m "update"

# 良い例
git commit -m "Add user authentication feature"
git commit -m "Fix null pointer error in login form"
```

### ポイント

- `git add .` はすべてのファイルを追加します。特定ファイルだけ追加したい場合は `git add ファイル名`
- `.gitignore` ファイルを作成して、追跡不要なファイル（`node_modules/`, `.env` など）を除外しましょう

---

## 補足 — 複数アカウントの使い分け

仕事用と個人用でGitHubアカウントが異なる場合、`~/.ssh/config` を使って接続先ごとに鍵を切り替えられます。

### ~/.ssh/config の設定

```
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

### リモートURLの指定

```bash
# 仕事用アカウントのリポジトリ
git remote add origin git@github-work:work-org/project.git

# 個人用アカウントのリポジトリ
git remote add origin git@github-personal:your-name/project.git
```

各リポジトリで `git config user.email` をそれぞれのアカウントのメールアドレスに設定することも忘れずに。

---

## まとめ — セットアップの全体像

```
1. git init              ← リポジトリを初期化
2. git config user.*     ← コミット情報の設定
3. ssh-keygen            ← SSH鍵ペアの生成
4. ssh-add               ← エージェントへの登録
5. GitHubに公開鍵を登録  ← GitHub側の設定
6. git remote add        ← リモートと紐づけ
7. git push -u origin main ← 初回プッシュ
```

一度この設定を完了すれば、以降の日常的な操作は `git add / commit / push / pull` だけで開発を進められます。
