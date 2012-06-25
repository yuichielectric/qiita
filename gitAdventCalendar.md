gitslaveで複数のリポジトリを管理する

この記事は [Git Advent Calendar / Jun.](http://qiita.com/advent-calendar/git) 25日目の記事です！
24日目は[rosylilly](http://qiita.com/users/rosylilly)さんの「[複数人での Git 開発に便利な 3 つのコマンド](http://qiita.com/items/9648ad2c8aa53465372b)」でした。

## gitslaveとは

gitslaveは[こちら](http://gitslave.sourceforge.net/)で公開されているツールで、あるgitリポジトリの中にサブディレクトリとして別のgitリポジトリを置いて管理するためのツールです。

似たようなツールとしてgit submodule([sotarok](http://qiita.com/users/sotarok)さんの「[Git submodule の基礎](http://qiita.com/items/0d525e568a6088f6f6bb)」がとてもわかりやすい！)があります。
しかし、submoduleで管理しているリポジトリ内も頻繁に更新するような使い方だと、以下のような操作が結構煩雑です。

* 親リポジトリとsubmoduleで管理している子リポジトリに一緒に変更を加えてcommitやpushをしたい。
* 親リポジトリと子リポジトリで同じ名前で一緒にブランチを切りたい。
* 親リポジトリと子リポジトリで同じ名前で一緒にタグを打ちたい。

gitslaveでは上記のような親リポジトリだけでなく子リポジトリにも頻繁にcommit、push、ブランチ作成などを行うようなプロジェクトに向いています。


## インストール

ではまずはインストールしてみましょう。
Macをお使いであれば、brewで一発でインストールできます。

```bash:
$ brew install gitslave
```

Mac以外の環境では[こちら](http://sourceforge.net/projects/gitslave/files/)からアーカイブをダウンロードしてきましょう。

## 使い方

ここからは例としてparentというリポジトリ内にTwitter Bootstrapを配置することにします([sotarok](http://qiita.com/users/sotarok)さんの真似です:P)。

gitslaveは`gits`というコマンドになっています。
紛らわしいのでご注意を。

まずは、gitslaveを使用する準備として`gits prepare`を実行すると、.gitslaveというファイルがコミットされます。

```bash:
# parentにて
$ gits prepare
master (root-commit) 4989262] gits creating .gitslave
 0 files changed
 create mode 100644 .gitslave
```

まだ、.gitslaveの中身は空です。

次に、`gits attach`コマンドでBootstrapリポジトリを取得します。

```bash:
$ gits attach https://github.com/twitter/bootstrap.git ./lib/bootstrap
Cloning into './lib/bootstrap'...
[master 1a5d1ba] gits adding "https://github.com/twitter/bootstrap.git" "./lib/bootstrap" (.gitignore)
 1 file changed, 1 insertion(+)
 create mode 100644 .gitignore
[master c618e90] gits adding "https://github.com/twitter/bootstrap.git" "./lib/bootstrap"
 1 file changed, 1 insertion(+)
```

parent/lib/bootstrapにTwitter Bootstrapが取得できました。

attachコマンドを実行すると、2つのコミットが実行されます。

```bash:
$ git log
commit c618e90e2ac4bdebe863fa6d29e988c55f7c5431
Author: yuichielectric <yuichielectric@xxx.com>
Date:   Mon Jun 25 08:42:22 2012 +0900

    gits adding "https://github.com/twitter/bootstrap.git" "./lib/bootstrap"

commit 1a5d1ba1d8a67fed9ba8edcb4dd42e614e7a8283
Author: yuichielectric <yuichielectric@xxx.com>
Date:   Mon Jun 25 08:42:22 2012 +0900

    gits adding "https://github.com/twitter/bootstrap.git" "./lib/bootstrap" (.gitignore)
```

1つ目のコミットで./lib/bootstrapを.gitignoreに追加して、2つ目のコミットで実際のbootstrapを取得しています。

## 変更してコミットする

次にparent/README.mdを追加し、さらにbootstrap下のファイルにも変更を加えてみます。
その後、普通に`git status`すると

```bash:
$ git status
# On branch master
# Untracked files:
#   (use "git add <file>..." to include in what will be committed)
#
#	README.md
```

bootstrap下の変更は見えません。
しかし、`gits`コマンドを使用すると

```bash:
$ gits status
# On branch master
# Changes not staged for commit:
#   (use "git add <file>..." to update what will be committed)
#   (use "git checkout -- <file>..." to discard changes in working directory)
#
#       modified:   ./lib/bootstrap/js/bootstrap-tooltip.js
#
# Untracked files:
#   (use "git add <file>..." to include in what will be committed)
#
#       ./README.md # Do not git(s) add this path
#
```

このようにREADME.mdとlib/bootstrap/js/bootstrap-tooltip.jsの両方に変更があることがわかります。
あとは、addしてcommitです。

```bash:
$ gits add -A
$ gits status
On: ./lib/bootstrap:
  # On branch master
  # Changes to be committed:
  #   (use "git reset HEAD <file>..." to unstage)
  #
  #     modified:   js/bootstrap-tooltip.js
  #
On: (parent):
  # On branch master
  # Changes to be committed:
  #   (use "git reset HEAD <file>..." to unstage)
  #
  #     new file:   README.md
  #
$ gits commit -m "Modified some files."
On: ./lib/bootstrap:
  [master 3e23fe1] Modified some files.
   1 file changed, 1 insertion(+), 1 deletion(-)
On: (parent):
  [master d5180de] Modified some files.
   1 file changed, 3 insertions(+)
   create mode 100644 README.md
$ gits log -1
On: ./lib/bootstrap:
  commit 3e23fe1c4c86922d8badf1b2d827739e47def1bf
  Author: yuichi_tanaka <yuichi_tanaka@cybozu.co.jp>
  Date:   Mon Jun 25 08:55:50 2012 +0900
  
      Modified some files.
On: (parent):
  commit d5180deb90e375f6cadf4cb2b7f0d4ceee083d5c
  Author: yuichi_tanaka <yuichi_tanaka@cybozu.co.jp>
  Date:   Mon Jun 25 08:55:50 2012 +0900
  
      Modified some files.
```

これで、parentリポジトリとbootstrapリポジトリの両方にコミットが出来ました。

## ブランチ作成

次に、parentとbootstrapの両方にまとめてブランチを切ってみます。

```bash:
$ gits checkout -b branch
On: (parent), ./lib/bootstrap:
  Switched to a new branch 'branch'
```

これで両方のリポジトリ上でbranchというブランチが切れました。
この後、修正、コミット、pushは前述の通り実行すれば、あたかも一つのリポジトリ上にいるかのように操作ができます。

## タグ打ち

タグも同様にしてまとめて打つことができます。

```bash:
$ gits tag test_tag
$ gits tag
On: ./lib/bootstrap:
  test_tag
  v1.0.0
  v1.1.0
  v1.1.1
  v1.2.0
  v1.3.0
  v1.4.0
  v2.0.0
  v2.0.1
  v2.0.2
  v2.0.3
  v2.0.4
On: (parent):
  test_tag
```

## gitslaveを利用しているリポジトリのclone

では、最後にgitslaveを利用しているリポジトリを他の人がcloneしてくるときの操作の紹介です。

```bash:
$ gits clone <path-to-repository> parent2
Cloning into 'parent2'...
done.
Cloning into './lib/bootstrap'...
```

これで、bootstrapリポジトリの取得も同時に実行してくれます。

## pull

ここまでくれば大体わかってくると思いますが、pullも本家と同様です。

```bash:
$ gits pull --rebase
On: (parent2):
  %REPO%From%REPO% /%REPO%home%REPO%/%REPO%yuichi%REPO%/%REPO%workspace%REPO%/./%REPO%parent%REPO%
     %REPO%3dedceb%REPO%..%REPO%f18308b%REPO%  %REPO%branch%REPO%     -> %REPO%origin%REPO%/%REPO%branch%REPO%
  %REPO%First%REPO%, %REPO%rewinding%REPO% %REPO%head%REPO% %REPO%to%REPO% %REPO%replay%REPO% %REPO%your%REPO% %REPO%work%REPO
  %REPO%Fast%REPO%-%REPO%forwarded%REPO% %REPO%branch%REPO% %REPO%to%REPO% %REPO%f18308b46994637c81035dcc49fcc26b4e0f02f5%REPO
On: ./lib/bootstrap:
  From /home/yuichi/workspace/./parent/lib/%REPO%
    branch     -> origin/branch
  First, rewinding head to replay your work on top of it...
  Fast-forwarded branch
```

これで、parentとbootstrapそれぞれのoriginから変更をpullしてくることができます。

※ parent2の表示が変になっているのは原因不明です。。

## まとめ

以上、非常に簡単ですがgitslaveについてまとめてみました。
gitslaveは以下のような利用用途に向いています。

* 複数のリポジトリを一つのリポジトリ上で利用したい。
* 親リポジトリだけでなく子リポジトリも同じタイミングで修正したり、ブランチを切ったり、タグを打ったりする。

もし、gitslave実際の運用で使っているよ！等ありましたら、是非教えてください！！
