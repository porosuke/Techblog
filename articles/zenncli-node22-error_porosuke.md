---
title: "Zenn-Cliが動かない！Node.js22＋GitBashでハマった話"
emoji: "😵"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["ZennCli", "Nodejs", "windows", "環境構築", "エラー解決"]
published: true
---
## 1. はじめに
Zennで技術記事を書こうと思い、Zenn-Cliの導入を進めていたところ、初手から環境構築でつまずいた。
同じようにGitBashで始めようとしている人の参考になればと思い、手順と原因をまとめておく。

## 2. 対象読者
- GitBashを使って最新版のZenn-Cliを導入しようとしている方。
- Windows環境にZenn-Cliを導入しようとしている方。

## 3. 結論
最終的に、Ubuntu環境にNode.js及びZenn-Cliを導入することで解決した。詳しくは4-4を参照。

## 4. 行ったこと
### 4-0. 準備
1. githubのアカウントを作成する
2. github上で空のリポジトリを作る（公開設定は不問）
3. Zennのアカウントを作成する
4. Zennとリポジトリを連携する
詳しくはここでは説明しない。こちらの記事を参考にしてほしい。
[公式解説サイト](https://zenn.dev/zenn/articles/connect-to-github)

### 4-1. 環境構築
まず、[このサイト](https://nodejs.org/ja/download)から Node.js ver22.21.0のsetup.exeファイルをダウンロードし、C直下に展開した。
次にGitBashを起動し、[このサイト](https://zenn.dev/zenn/articles/install-zenn-cli)を参考に、作業を進めようとした。コンテンツを管理するディレクトリはTechblogとした。
```bash
$ mkdir Techblog
$ cd Techblog
$ npm init --yes
$ npm install zenn-cli
``` 
ここまではうまく動いていたのだが......初期化で以下のようなエラーが。
```bash
$ npx zenn init
node:internal/url:1489
    throw new ERR_INVALID_FILE_URL_PATH('must be absolute', url);
    ^

TypeError [ERR_INVALID_FILE_URL_PATH]: File URL path must be absolute
    at getPathFromURLWin32 (node:internal/url:1489:11)
    at fileURLToPath (node:internal/url:1613:35)
...
Node.js v22.21.0
```

### 4-2. 原因
このエラーの原因はNode.js 22系がWindowsパスを絶対パスに変換できずに失敗するバグによるものだった。
Zenn-Cli内部でファイルURL→パス変換を行う箇所がありGitBashなどで発生する。
この問題はZenn-Cli側ではなくNode.js ver22の仕様変更・不具合に起因していると思われる。

### 4-3. 一時的な回避策？
Node.jsのバージョンが22に上がったためにエラーが起きるわけで、Node.jsのバージョンを落とせばいいのではないかと考えた。しかし、実際Node.js 20をインストールしてみたところ、以下の警告が出てしまった。
```bash
# 事前に古いNode.jsとzenn-cliをアンインストールしておく
$ npm install zenn-cli
npm WARN EBADENGINE Unsupported engine {
  package: 'zenn-cli@0.2.6',
  required: { node: '>=22.0.0' },
  current: { node: 'v20.19.5', npm: '9.6.3' }
}
```
これは「Zenn-Cliは22.0.0以上のバージョンのNode.jsを要求しています」という警告であり、これを解消するにはNode.js 20系でも動くバージョンの、古いZenn-Cliを入れなければならない。
ただ、今後アップデートが重ねられていくZenn-Cliにおいて、過去のバージョンに頼っていくのは根本的解決にならない。

### 4-4. 解決策
最終的に、Ubuntu環境にNode.js及びZenn-Cliを導入することで解決した。以下に導入の詳細を示す。
1. Node.js v22系を、**Ubuntu環境に**導入する。今回はVSCodeからWSL環境を操作することにした。
```bash
# Node.js 22 LTS を取得
$ curl -fsSL https://deb.nodesource.com/setup_22.x | sudo -E bash -
$ sudo apt-get install -y nodejs

# 動作確認
$ node -v   # v22.x が出ることを確認
$ npm -v    # npmも確認
```
2. Zenn-Cliの最新版を**Ubuntu環境に**インストールする。
```bash
$ npm install -g zenn-cli
```
:::details エラーが出たら
Zenn-CliをUbuntu環境にインストールしようとしたとき、以下のようなエラーが出ることがある。
```bash
npm error code EACCES
npm error syscall mkdir
npm error path /usr/lib/node_modules/zenn-cli
```
これは/usr/lib/node_modulesに書き込み権限がなく、普通のユーザーではグローバルパッケージをインストールできないために起こる。
これを解決するには管理者権限でインストールするか、
```bash
$ sudo npm install -g zenn-cli
```
ユーザー権限でグローバルインストールする必要がある。
```bash
# npm のグローバルディレクトリをユーザー領域に設定
$ mkdir ~/.npm-global
$ npm config set prefix '~/.npm-global'
# パスを通す
$ echo 'export PATH=$HOME/.npm-global/bin:$PATH' >> ~/.profile
$ source ~/.profile
# 改めてグローバルインストール
$ npm install -g zenn-cli
```
:::
3. Zenn-Cliを初期化する。
```bash
$ npx zenn init
$ dir
```
成功すると、articles/ と books/ フォルダ、README.md などが生成されるはず。

4. gitの設定を行う。
```bash
$ git init
$ git remote add origin https://github.com/<GitHubユーザー名>/Techblog.git
$ git add .
$ git commit -m "init: setup Zenn project"
$ git branch -M main
$ git push -u origin main
```
:::details エラーが出たら
gitに何らかの設定をしたとき、以下のようなエラーが出ることがある。
```bash
$ fatal: empty ident name (for <ユーザー名>) not allowed
```
これは利用者の名前とメールアドレスが登録されていないことによりgitの変更ができないことを意味する。
解決するには登録が必要になる。メールアドレスは非公開（no-reply）のものでも構わない。
```bash
$ git config --global user.name "任意の名前"
$ git config --global user.email "GitHubに登録しているメールアドレス"
```
:::

## 5. よくある関連問題
- 毎回Techblogディレクトリから始めたい。
→該当ディレクトリにいる状態でVSCodeのFile > Save Workspace As からワークスペースを任意の場所に保存する。次回からはそのワークスペースから起動することですぐに執筆を始められる。
- masterがどうのといったメッセージが出た
→「ブランチ名をmasterとして作成しましたが、今後のデフォルトをmainという名前にしたかったら設定してください」というものである。
```bash
hint: Using 'master' as the name for the initial branch. This default branch name
```
以下のコードでデフォルトを設定できる。
```bash
$ git config --global init.defaultBranch main
```
masterからmainに名前を変更するには以下のコード。
```bash
$ git branch -m main
```

## 6. まとめ
今回、まさか最新版のZenn-Cliを導入するのに苦戦するとは思わなかった。インストールに関する記事は多くあったものの、エラーについて触れている記事は見つからず、解決に時間をかけてしまった。Zenn-Cliは便利だが、Windows環境ではWSL経由が安定というのが今回の結論。