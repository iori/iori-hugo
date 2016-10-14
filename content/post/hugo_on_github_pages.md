+++
categories = ["Development", "Hugo", "GitHub Pages"]
date = "2015-04-17T09:00:00+09:00"
description = "hugoでGitHubPagesにBlogを開設する手順のまとめ"
title = "Hugo on GitHub Pages."
slug = "Hugo on GitHub Pages"

+++

hugoでGitHubPagesにBlogを開設する手順のまとめ。

## GitHub Pages & Hugo

GitHub Pagesは **自分のアカウント名.github.io** というURLで静的サイトを公開できるサービス。無料！やったね！

今回はhugoというGo言語製の静的サイトジェネレーターを使って、GitHub Pagesに静的サイトを構築します。hugoでは

* テーマの選択
* MarkDownでの記事の記述

が出来ます。

## 構築手順

今回は

1. hugoのインストール
2. hugoでテンプレートを作成する
3. テーマを使用する
4. 記事を作成する
5. slugを利用してURLを変更する
6. localで表示を確認する
7. deployする

までやります

### 1. hugoのインストール

macの場合は

```bash
$ brew install hugo
```

でインストール出来ます。他のOSの方は、[こちら](https://github.com/spf13/hugo#choose-how-to-install)を御覧ください。

### 2. hugoでテンプレートを作成する

```bash
$ hugo new site site_name
```

で作成されます。

#### リポジトリの作成

次にGitHubで

* hugoのテンプレートを管理するリポジトリの作成(これは名前は何でもOK)
* あなたのアカウント名.github.io というリポジトリの作成

をして下さい。 **アカウント名.github.io** リポジトリではhugoで生成した静的なファイルをpushします。hugoで作成したプロジェクトは別のリポジトリで管理するためです。リポジトリの作成が完了したら

```bash
$ git init
$ echo public/ >> .gitignore # hugoが生成する静的なファイルはこのリポジトリでは管理しない
$ git remote add git@github.com:あなたのアカウント名/テンプレート管理用のリポジトリ名.git
$ git add .
$ git commit -am 'first commit'
```

をして下さい。

### 3. テーマを使用する

https://github.com/spf13/hugoThemes/#theme-list
ここから利用したいテーマを探して下さい。今回は ` hyde-x`を利用します。hyde-xのリポジトリに移り、SSH Clone URLをコピーし

```bash
$ git submodule add git@github.com:zyro/hyde-x.git themes/hyde-x
```

します。これでthemesにhyde-xがコピーされ、submoduleが登録されます。` config.toml`に` theme = "hyde-x"`を追記して下さい。追記したらまた` git commit`しておきましょう。

### 4. 記事を作成する

次は実際に記事を作成して見ましょう。hugoでは

```bash
$ hugo new post/example.md
```

で記事を作成出来ます。ファイルは` content/`に生成されます。記事の内容は

```
+++
categories = []
date = "2015-07-31T14:52:15+09:00"
description = ""
keywords = []
title = "example"

+++
```

のようになります。` +++`の下からMardDownで記事を書くことが出来ます。` +++`内は設定です。

### 5. slugを利用してURLを変更する

slugを指定してURLを変更することが出来ます。これはファイル名がURLになるのを防ぐことができます。` example.md`の` +++`内に

```
slug = "hoge"
```

と追記して下さい。また、` config.toml`に

```
[permalinks]
  post = "/entry/:slug/"
```

と追記して下さい。これで` content/post/`以下のファイルは` /entry/(ファイル名|slugで指定した文字列`がURLとなります。

### 6. localで表示を確認する

ここまで来たらもう少しです。次のコマンドを実行して下さい。

```bash
$ hugo server -w
```

これで` localhost:1313`でHOMEにアクセス出来、`example`というタイトルの` localhost:1313/entry/hoge`という記事も生成されている筈です。

` -w`は、記事が保存されたら自動的にブラウザの内容を更新してくれるので便利です。ここまでで` git commit`しておきましょう。

### deployする

deployするにはまず、` public/`をgitで新たに管理します。

```bash
$ cd public
$ git init
$ git remote add git@github.com:あなたのアカウント名/あなたのアカウント名.git
```

このようにして下さい。これでpublicが` アカウント名.github.io`というリポジトリで管理されます。

hugoではdeploy用のコマンドが用意されていないので、rootに以下の` deploy.sh`を作成します

```sh
#deploy.shの中身
#!/bin/bash

echo -e "\033[0;32mDeploying updates to GitHub...\033[0m"

# Build the project. 
hugo

# Go To Public folder
cd public

# Add changes to git.
git add -A

# Commit changes.
msg="rebuilding site `date`"
if [ $# -eq 1 ]
  then msg="$1"
fi
git commit -m "$msg"

#Push source and build repos.
git push origin master

# Come Back
cd ..
```

` deploy.sh`を作成したら、`git commit`しておきましょう。最後に、

```bash
$ ./deploy.sh
```

をすると、自動的に` public/`に静的ファイルを生成し、githubのリポジトリにpushしてくれます。これで ` 貴方のアカウント名.github.io`でサイトが構築され実際にアクセスすることが出来ます。
