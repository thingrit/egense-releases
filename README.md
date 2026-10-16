# Egense

Coding Agent（Claude Code または Codex）に作業を頼み、出てきた変更を Mac の画面でレビューして、
コミット・マージまで進めるアプリです。

このリポジトリには配布物だけを置いています。ソースは非公開です。
入っているアプリは、ここの最新のリリースを見て自分を更新します。

## 必要なもの

- macOS（Apple Silicon・Intel のどちらでも動きます）
- `git`
- `claude`（Claude Code）か `codex`（Codex）の少なくとも 1 つ
- `gh`（GitHub CLI）: Pull Request をレビューするときだけ使います。`gh auth login` でログインしておきます

コマンドは、ログインシェルの `PATH` と、`/opt/homebrew/bin`・`/usr/local/bin`・`~/.local/bin` などの
よく使われる場所から探します。見つからないときは Settings の Agent の欄に
「使える Agent が見つかりません」と出ます。

Claude Code のアカウントは、Settings の「Claude のアカウント」→「アカウントを追加」からアプリの中でログインできます。

## 入れ方

1. [最新のリリース](https://github.com/thingrit/egense-releases/releases/latest) から
   `Egense_<版>_universal.dmg` をダウンロードします。
2. dmg を開き、`Egense.app` を「アプリケーション」フォルダへ移します。
3. Egense を起動します。

`Egense.app.tar.gz` と `latest.json` は自動更新に使うファイルです。手でダウンロードする必要はありません。

## 使い方

### リポジトリを追加する

左の「Workspace」で「Git リポジトリを選択」（または「Open…」）を押し、作業する Git リポジトリのフォルダを選びます。
Task はリポジトリごとに作ります。

### Task を作る

1. リポジトリを選び、「新しい Task を入力…」の欄に依頼の内容を書きます。
   `/` で Skill を、`@` でファイルを指定できます。⌘N でこの欄へ移れます。
2. 「Create task」を押すか Enter を押します。

作ると、Egense は次のことを行います。

- `~/.egense/worktrees/<リポジトリ名>/` の下に、Task 専用の作業ツリーと `egense/` で始まる branch を作ります
- 書いた依頼をそのまま Coding Agent に渡し、作業を始めさせます
- Task の名前を依頼の内容から付けます

作った直後の 5 秒間は、Esc で作成を取り消せます。

入力欄の横のチェックボックスで、作り方を変えられます。

- 「Work tree」: 外すと、作業ツリーを作らずにリポジトリ本体の今の branch で作業します
- 「Review mode」: 入れると、入力した Pull Request の番号か URL を取り込み、AI にレビューさせます

### 変更をレビューする

Task を開くと、次の 3 つが並びます。

- Changes: 変更したファイルの一覧と差分。ファイルごとに確認済みの印を付けられます
- Interaction: Coding Agent とのやりとり。続けて依頼を送れます
- Checks: プロジェクトの検査の結果と、AI レビューの指摘

指摘ごとに「直す」「直さない」「今回は見送る」を選びます。「AI に直させる」で修正を頼むこともできます。
差分の行を選ぶと、その箇所について「説明してもらう」「調べてもらう」「レビューしてもらう」と AI に頼めます。

レビューでの判断が溜まると、Egense は次からの判断に使う Rule の候補を作ります。
候補は上の「Policy」で確かめ、使うかどうかを決めます。

### Task を終える

「Task の完了」を押し、行う手順を選びます。コミット、マージ先の branch へのマージ、
作業ツリーの削除、branch の削除などを 1 回で行えます。マージで衝突したときは、AI に解決を頼めます。

変更を残さずにやめるときは「Task の破棄」を使います。コミットもマージもせずに、作業ツリーと branch と Task を消します。

### iPhone から使う

Settings の「外部から使用」で「接続の受け付け」を「受け付ける」にすると、「接続先」にアドレスが出ます。
同じネットワーク（または VPN）にある iPhone のブラウザでそのアドレスを開くと、
Task の一覧を見る、新しい Task を作る、Agent に答える、Task を完了にする、といった操作ができます。
差分のレビューと Terminal は Mac でだけ使えます。

アドレスにはトークンが入っています。人に渡したり、画面の写真に写したりしないでください。

## 更新

起動したときに新しい版を確かめます。新しい版があると Settings のアイコンに印が付きます。
Settings の「更新」で「更新して再起動」を押すと入れ替わります。
Agent や Terminal のコマンドが動いているときは、止めてよいかの確認が出ます。「停止して更新」で進めます。

## データの置き場所

データは `~/.egense` にあります。

| 場所 | 中身 |
|---|---|
| `egense.db` | Task や判断の記録（SQLite） |
| `worktrees/` | Task ごとの作業ツリー |
| `backups/` | データベースの形を変える前に取った控え。新しいものから 5 個を残します |
| `attachments/` | 依頼に添えたファイル |
| `claude-accounts/` | アプリの中でログインした Claude Code のアカウント |

## 前の版に戻す

新しい版で不具合が出たときは、次の手順で戻せます。

1. Egense を終了します（⌘Q）。
2. [リリースの一覧](https://github.com/thingrit/egense-releases/releases) から戻したい版の dmg をダウンロードし、
   「アプリケーション」フォルダの `Egense.app` と入れ替えます。
3. 新しい版でデータベースの形が変わっていた場合は、控えを書き戻します。
   `~/.egense/backups/` に、新しい版を入れたときの日時の付いた控え（`egense-v<番号>-<日時>.db`）があれば、
   形が変わっています。無ければこの手順は要りません。

   ```sh
   # いまのデータベースを控えておく（書き戻しをやり直せるように）
   sqlite3 ~/.egense/egense.db "VACUUM INTO '$HOME/.egense/backups/egense-before-restore-$(date +%Y%m%d-%H%M%S).db'"
   # 控えを書き戻す
   cp ~/.egense/backups/egense-v<番号>-<日時>.db ~/.egense/egense.db
   rm -f ~/.egense/egense.db-wal ~/.egense/egense.db-shm
   ```

   控えを取ったあとに作った Task や判断の記録は消えます。
4. Egense を起動します。

戻した版のあいだも、アプリは新しい版があることを知らせ続けます。戻した状態で使い続けるあいだは、更新を入れないでください。
