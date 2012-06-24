gitslaveで複数のリポジトリを管理する

この記事は [Git Advent Calendar / Jun.](http://qiita.com/advent-calendar/git) 25日目の記事です！
22日目は[rosylilly](http://qiita.com/users/rosylilly)さんの「[複数人での Git 開発に便利な 3 つのコマンド](http://qiita.com/items/9648ad2c8aa53465372b)」でした。

## gitslaveとは

gitslaveは[こちら](http://gitslave.sourceforge.net/)で公開されているツールで、あるgitリポジトリの中にサブディレクトリとして別の
gitリポジトリを置いて管理するためのツールです。

似たようなツールとしてgit submodule([sotarok](http://qiita.com/users/sotarok)さんの「[Git submodule の基礎](http://qiita.com/items/0d525e568a6088f6f6bb)」がとてもわかりやすい!)
がありますが、submoduleで管理しているリポジトリ内も頻繁に更新するような使い方だと、以下のような操作が
結構煩雑です。

* 親リポジトリとsubmoduleで管理している子リポジトリに一緒に変更を加えてcommitやpushをしたい。
* 親リポジトリと子リポジトリで同じ名前で一緒にブランチを切りたい。
* 親リポジトリと子リポジトリで同じ名前で一緒にタグを打ちたい。

gitslaveでは上記のような親リポジトリだけでなく子リポジトリにも頻繁にcommit、push、ブランチ作成などを
行うようなプロジェクトに向いています。


## インストール

ではまずはインストールしてみましょう。
Macをお使いであれば、brewで一発でインストールできます。

```bash:
$ brew install gitslave
```

Mac以外の環境では[こちら](http://sourceforge.net/projects/gitslave/files/)からアーカイブをダウンロードしてきましょう。

## 使い方

ここからは例としてparentというリポジトリ内にTwitter Bootstrapを配置することにします。
([sotarok](http://qiita.com/users/sotarok)さんの真似です:P)

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
[master 0fe079a] gits adding "https://github.com/twitter/bootstrap.git" "./lib/bootstrap" (.gitignore)
 1 file changed, 1 insertion(+)
 create mode 100644 .gitignore
[master 436ce68] gits adding "https://github.com/twitter/bootstrap.git" "./lib/bootstrap"
 1 file changed, 1 insertion(+)
```

これで、parent/lib/bootstrapにTwitter Bootstrapが取得できました。

## 変更してコミットする


