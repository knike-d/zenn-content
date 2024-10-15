---
title: "1回のコマンドでgit rebaseのfixupを行う方法"
emoji: "👷‍♂️"
type: "tech"
topics: ["git"]
published: true
published_at: 2024-10-15 07:00
---

# TL;DR

まずは、以下のコマンドで Git のグローバル設定ファイルである .gitconfig を開きましょう。

```
git config --global --edit
```

.gitconfig に alias を追加します。

```
[alias]
        fixup ="!f(){ git commit --fixup $1 && GIT_SEQUENCE_EDITOR=: git rebase -i --autosquash --autostash $1^;};f"
```

あとは修正したい変更をステージング（`git add`）した後に、以下のコマンドを実行するだけです。

```
git fixup <commit hash>
```

# 解説

git rebase で特定の commit を fixup する際には以下のようなステップを踏むことが多いと思います。

```
git commit --fixup <commit hash>
git rebase -i –autosquash <commit hash>^
```

これを 1 つのコマンドにまとめ、rebase 時のエディターを開かずに fixup を実行する方法として Git の alias を利用します。

## `alias`

Git の alias を使うことで、独自にコマンドを定義できます。
今回は`git fixup`というコマンドを作成しています。

## `!f(){ ... };f`

alias は先頭に`!`を付与すると、シェルコマンドとして認識させることができます。
シェルスクリプトとして関数の定義&即時実行させることで複数のコマンドをまとめて実行させることが可能です。

## `$1`

`$1`は関数の最初の引数を意味します。
今回の実行例`git fixup <commit hash>`では`<commit hash>`の部分が引数となります。

## `git commit --fixup $1`

これは指定した commit を修正するための fixup コミットを作成します。
fixup オプションを指定すると、先頭に `fixup!` の文字列が付与された commit メッセージで commit が作成されます。

```
fixup! <指定したcommitのcommitメッセージ>
```

この commit がある状態で rebase を autosquash オプション付きで実行すると、fixup コミットが自動で適切な位置に移動します。
このオプションがない場合は、手動でこの編集操作を行う必要があります。

## `&& GIT_SEQUENCE_EDITOR=:`

`&&`でコマンドを繋げます。

`GIT_SEQUENCE_EDITOR=:`は rebase の際にエディター編集をスキップする、という設定になります。
`GIT_SEQUENCE_EDITOR`という Git の環境変数に、何もしないコマンド`:`を設定しています。

## `git rebase -i --autosquash --autostash $1^`

`--autostash`は rebase するときに自動で作業中の変更を一時的に退避（`git stash`）するオプションです。
編集中のファイルがある場合は rebase を実行することができませんが、このオプションを設定することで編集中のファイルがある場合でも rebase を実行可能にします。
rebase が完了すると、退避した変更は元に戻ります。

`$1^`の`^`は 1 つ前の commit を指します。つまり、今回は fixup したい commit の 1 つ前の commit が指定されます。

# 補足: rebase オプションを global に設定する

今回の例では、rebase コマンドに autosquash や autostash オプションを指定していましたが、global に設定することもできます。
これにより、今回定義した`git fixup`コマンドを使わない場合でも、rebase 実行時に毎回オプションを指定する必要がなくなります。

設定したい場合は .gitconfig に以下を追加します。

```
[rebase]
        autostash = true
        autosquash = true
```

# まとめ

.gitconfig に alias を定義して、一連の動作を 1 回のコマンドで fixup する方法を紹介しました。
今回紹介した方法はちょっとした修正を過去の commit にまとめたい場合に便利なので、ぜひ活用してみてください。
