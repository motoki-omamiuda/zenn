---
title: "Git｜手を動かして仕組みを理解する！"
emoji: "🕌"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [Git]
published: true
---
:::details バージョン管理システムの歴史

コラムとして、VCS（Version Control System）の歴史を紹介しておきます。  
これを知ることで、いかにしてGitの仕組みが出来上がったのかを知ることができます


### 1982年：RCS (Revision Control System)
最初期のバージョン管理システムです。「ファイル単位」でしか管理できず、プロジェクト全体をまとめて管理する概念はありませんでした。また、複数のユーザーが同時に同じファイルを編集することも想定されていなかったため、後に登場するCVSに主役を譲ることになります。

### 1990年：CVS (Concurrent Versions System)
ネットワーク経由での共同作業を可能にした、初の本格的なソースコード管理システムです。  オープンソース（フリーウェア）だったため、1990年代に爆発的に普及しました。しかし、「ファイルやディレクトリの移動・名前変更・削除がうまく扱えない」、「日本語などの文字コード（SJIS/EUC等）がファイル名やマージの際に化けてしまう」といった問題があり、Subversionへと移行していきました。

### 2004年：Subversion (SVN)
CVSの弱点だった「ファイルやディレクトリの名前変更・削除」を完璧に行えるようにした、正統進化系のシステムです。2000年代後半の主流となりました。しかし、すべてのデータを一つのサーバーで管理する「中央集権型」だったため、「サーバーが落ちると全員の作業が止まる（単一障害点）」、「ネットに繋がっていないとコミット（履歴の保存）すらできない」という限界があり、Gitに取って代わられました。

### 2005年：Git
これまでのすべての課題を解決する「分散型」バージョン管理システムとして登場しました。自分のパソコン内だけで履歴管理が完結し、複数人での膨大な分岐（ブランチ）やマージも高速・安全に行えます。その圧倒的な便利さから普及が進み、2025年時点ではバージョン管理システム市場の約87%を占める絶対的なスタンダードとなっています。
:::

## この記事で扱う内容
この記事はGitの配管コマンド（低レベルのコマンド）を使って、Git内部の仕組みをきちんと理解しよう、というものです。便利なGitコマンドの紹介やコミットメッセージの実務的な書き方の紹介、等は行わず、Gitのオブジェクトや情報管理の方法など、根幹となる内部の仕組み解説を目的としています。

# 【解説】Gitのデータ管理方法
まず初めに、Gitは`.git/objects`フォルダ配下で、Git管理しているファイルの内容・ディレクトリの構造情報・コミットの情報を保存しています。

ファイル内容を保存するためのオブジェクトを[Blobオブジェクト](#blobオブジェクト)、ディレクトリ構造を保存するためのオブジェクトを[Treeオブジェクト](#treeオブジェクト)、コミット情報を保存するためのオブジェクトを[Commitオブジェクト](#commitオブジェクト)と呼びます。

各オブジェクトは「内容をハッシュ化したもの」をKey、「内容を復元可能な形（Deflate圧縮アルゴリズム[^2]）で圧縮したもの」をValueとして保存しています。

## ４つのオブジェクトの詳細

### Blobオブジェクト
Gitではファイル内容をBlobというオブジェクトとして管理しています。前述の通り、これは「ファイルの内容をハッシュ化したもの」をKey、「ファイルの内容を復元可能な形で圧縮したもの」をValueとして保存しており、ファイル名は保存されません。

また、Blobオブジェクトはインデックス（ステージング）と呼ばれるファイルに登録することができ、そこから後述するTreeオブジェクトを作成することができます。

```bash
# 便利な配管コマンド
$ git hash-object -w <file> # 特定のファイルに対応したBlobオブジェクトの作成
$ git cat-file -p <blob-hash> # BlobオブジェクトのValueを見る配管コマンド
```

#### 【課題】  
上記コマンドを使って、Git管理するフォルダを作成し、特定のファイルに対するBlobオブジェクトを作成し、中身を確認してみましょう。

:::details 【回答】
```bash
# 1. 使用するフォルダの作成する
$ mkdir git-hands-on-01

# 2. フォルダに入る
$ cd git-hands-on-01

# 3. Git管理を開始する
$ git init .

# 4. .git/objectsを確認する
$ ls -la .git/objects/
total 16
drwxr-xr-x 4 motoki motoki 4096 May 26 10:30 .
drwxr-xr-x 7 motoki motoki 4096 May 26 10:30 ..
drwxr-xr-x 2 motoki motoki 4096 May 26 10:30 info
drwxr-xr-x 2 motoki motoki 4096 May 26 10:30 pack

# 5. Git管理するファイルの作成
$ echo "hello" > text.txt

# 6. ファイルのBlobオブジェクト作成
$ git hash-object -w text.txt
ce013625030ba8dba906f756967f9e9ca394464a

# 7. .git/objectsを確認する
$ ls -la .git/objects/
total 20
drwxr-xr-x 5 motoki motoki 4096 May 26 10:34 .
drwxr-xr-x 7 motoki motoki 4096 May 26 10:30 ..
drwxr-xr-x 2 motoki motoki 4096 May 26 10:34 ce # Blobオブジェクトが作成された！
drwxr-xr-x 2 motoki motoki 4096 May 26 10:30 info
drwxr-xr-x 2 motoki motoki 4096 May 26 10:30 pack

# 8. Blobオブジェクトの中身を確認
$ git cat-file -p ce013625030ba8dba906f756967f9e9ca394464a
hello # 作成したファイルの中身が出力された！
```
:::


### Treeオブジェクト
Gitでは、Treeというオブジェクトとして、各プロジェクトのディレクトリ構造のデータを保存しています。各Treeオブジェクトには、BlobオブジェクトやTreeオブジェクトを紐づけることができ、紐づいたオブジェクトは子要素としてTreeオブジェクトに格納されていることになります。

![](https://static.zenn.studio/user-upload/468ca2f9c77b-20260526.png)
*Treeオブジェクトのイメージ[^1]*

また、基本的にTreeオブジェクトはインデックス（ステージング）ファイルを元に作成されます。

```bash
# 便利な配管コマンド
$ git update-index --add --cacheinfo <number> <hash> <name> # Blobオブジェクトを特定の名前でインデックスに登録する
# 基本的な<number>の例
# 100644：普通のファイル（読み書き可能）  
# 100755：実行可能ファイル（スクリプトなど）  
# 120000：シンボリックリンク  

$ git write-tree  # インデックスを元に、Treeオブジェクトを作る配管コマンド
```

#### 【課題】  
上記コマンドを使って、前の課題で作成したBlobオブジェクトをインデックスに登録し、Treeオブジェクトを作成してみましょう。

:::details 【回答】
```bash
# 1. Blobオブジェクトmain.txtという名前でインデックスに登録
$ git update-index --add --cacheinfo 100644 ce013625030ba8dba906f756967f9e9ca394464a main.txt

# 2. Treeオブジェクトの作成
$ git write-tree
38051ae48accaf0025a257eaa7a3a328e1f0fe56

# 3. .git/objectsを確認する
$ ls -la .git/objects/
total 24
drwxr-xr-x 6 motoki motoki 4096 May 26 11:01 .
drwxr-xr-x 7 motoki motoki 4096 May 26 11:01 ..
drwxr-xr-x 2 motoki motoki 4096 May 26 11:01 38 # Treeオブジェクトが作成された！
drwxr-xr-x 2 motoki motoki 4096 May 26 10:34 ce
drwxr-xr-x 2 motoki motoki 4096 May 26 10:30 info
drwxr-xr-x 2 motoki motoki 4096 May 26 10:30 pack

# 4. Treeオブジェクトの中身を確認
$ git cat-file -p 38051ae48accaf0025a257eaa7a3a328e1f0fe56
100644 blob ce013625030ba8dba906f756967f9e9ca394464a main.txt # 格納したオブジェクトが出力された！
```
:::

### Commitオブジェクト
Gitでは、誰が・いつ・なんのTreeオブジェクトを作成したのか、という情報をCommitオブジェクトとして保存しています。Commitオブジェクトには、最上層（ルート）のTreeオブジェクトや親となるCommitオブジェクトが紐づいています。

また、Commitオブジェクトには任意のコメントを製作者が記載することができます。

![](https://static.zenn.studio/user-upload/61b7d6f5db2a-20260526.png)
*Commitオブジェクトのイメージ[^1]*

```bash
# 便利な配管コマンド
$ git commit-tree <tree-hash> # TreeオブジェクトからCommitオブジェクトを作成する
```

#### 【課題】  
前の課題で作成したTreeオブジェクトを元に、コメントを付けたCommitオブジェクトを作成してみましょう。（上記コマンドを使います）

:::details 【回答】
```bash
# 1. Treeオブジェクトを元に、コメントをつけてCommitオブジェクトを作成する
$ git commit-tree 38051ae48accaf0025a257eaa7a3a328e1f0fe56 -m "add: ファイルの追加"
088bf5b6a738a9d088a97273c8f81b6dd9fb5a49

# 2. .git/objectsを確認する
$ ls -la .git/objects/
total 28
drwxr-xr-x 7 motoki motoki 4096 May 26 11:14 .
drwxr-xr-x 7 motoki motoki 4096 May 26 11:01 ..
drwxr-xr-x 2 motoki motoki 4096 May 26 11:14 08 # Commitオブジェクトが作成された！
drwxr-xr-x 2 motoki motoki 4096 May 26 11:01 38
drwxr-xr-x 2 motoki motoki 4096 May 26 10:34 ce
drwxr-xr-x 2 motoki motoki 4096 May 26 10:30 info
drwxr-xr-x 2 motoki motoki 4096 May 26 10:30 pack

# 3. Commitオブジェクトの中身を確認
$ git cat-file -p 088bf5b6a738a9d088a97273c8f81b6dd9fb5a49
tree 38051ae48accaf0025a257eaa7a3a328e1f0fe56
author Motoki Omamiuda <omamiuda1011@gmail.com> 1779761658 +0900
committer Motoki Omamiuda <omamiuda1011@gmail.com> 1779761658 +0900

add: ファイルの追加 # しっかりとCommit作成者の情報が記載されている！
```

:::

### Tagオブジェクト
特定のコミットに固定の目印をつけるオブジェクトです。「注釈付きタグ」を作ると、作成者や署名情報が入った専用の「タグオブジェクト」が生成されます。

## ブランチとHEAD

### ブランチ
Gitのブランチは巷で言われるような「歴史の枝分かれ」のような大層なものではありません。ブランチとは、commitオブジェクトのハッシュが記載されたテキストファイルです。`.git/refs/heads/main` というファイルを開くと、commitオブジェクトのハッシュが書かれていること確認できます。

```bash
$ git update-ref <branch> <commit-hash> # ブランチを更新する配管コマンド
```

### HEAD
GitのHEADとは、今どのブランチにいるかを示すポインタです。`.git/HEAD`というファイルを開くと、今いるブランチが記載されていることが確認できます。ブランチを移動するとは、このテキストファイルを書き換えるだけの処理です。

```bash
$ git symbolic-ref HEAD  # HEADを確認する配管コマンド
```

# 【実践】低レベルコマンドでコミット
上記内容を踏まえて、よく使うコマンドの意味を理解してみましょう。普段の業務では、以下のようなコマンドでファイルを管理している人が多いでしょう。

```bash
# 普段の業務で使うコマンド

# 1. ファイルをBlobオブジェクトとして登録
# 2. Blobオブジェクトをインデックス（ステージング）に紐づけ
# 3. インデックスの情報を元にTreeオブジェクトを作成
$ git add .

# 4. 作成したTreeオブジェクトとメッセージを合体させてCommitオブジェクトを作成
# 5. HEADがあるブランチにCommitオブジェクトを紐づけ
$ git commit -m "add: ファイルの追加"
```

このコマンドを、配管コマンドを使って書き直していきます。

## 準備
改めて新規のフォルダを作成し、そこで作業を行ってください。

```bash
# 1. 使用するフォルダの作成する
$ mkdir git-hands-on-02

# 2. フォルダに入る
$ cd git-hands-on-02

# 3. Git管理を開始する
$ git init .

# 4. Git管理するファイルの作成
$ echo "hello" > text.txt
```

## `git add` の分解
```bash
# 1. ファイルをgitデータベースにblobオブジェクトとして登録
$ git hash-object -w text.txt
ce013625030ba8dba906f756967f9e9ca394464a

# 2. blobオブジェクトをインデックス（ステージング）に紐づけ

# 詳細
# hashでblobオブジェクトを、普通のファイル（100644）として、text.txtという名前でインデックスに紐づけ

# 番号の意味
# 100644：普通のファイル（読み書き可能）  
# 100755：実行可能ファイル（スクリプトなど）  
# 120000：シンボリックリンク  
$ git update-index --add --cacheinfo 100644 ce013625030ba8dba906f756967f9e9ca394464a text.txt

# 3. インデックスの情報を元にtreeオブジェクトを作成
$ git write-tree
b0739a0d4ff07ce11d15ec008086e0236759dc69
```

## `git commit` の分解
```bash
# 1. 作成したtreeオブジェクトとメッセージを合体させてcommitオブジェクトを作成
$ git commit-tree b0739a0d4ff07ce11d15ec008086e0236759dc69 -m "add: ファイルの追加"
088bf5b6a738a9d088a97273c8f81b6dd9fb5a49

# 2. 今いるブランチを確認
$ git symbolic-ref HEAD
refs/heads/main

# 3. HEADがあるブランチにcommitオブジェクトを紐づけ
$ git update-ref refs/heads/main 088bf5b6a738a9d088a97273c8f81b6dd9fb5a49
```

最後にGitのログを表示してみましょう。しっかりコミットが作成されていることが確認できます。
```bash
$ git log
commit 088bf5b6a738a9d088a97273c8f81b6dd9fb5a49 (HEAD -> main)
Author: Motoki Omamiuda <omamiuda1011@gmail.com>
Date:   Tue May 26 11:14:18 2026 +0900

    add: ファイルの追加
```
 
[^1]: [*Pro Git book -chapter10（Git official book）*](https://git-scm.com/book/ja/v2)  
[^2]: [*Deflate圧縮アルゴリズム（wiki）*](https://ja.wikipedia.org/wiki/Deflate)  
