【第1回】 2022年5月12日

# Ubuntuで oTree 本番環境を構築する


## 1. oTree に関する情報

<p class="ytubevideo"><iframe width="560" height="315" src="https://www.youtube.com/embed/pSNcDqts_Jg?rel=0&enablejsapi=1&origin=https://yshimod.github.io/" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe></p>


- [参考文献のリスト](references/README.md)



## 2. oTree の本番環境

<p class="ytubevideo"><iframe width="560" height="315" src="https://www.youtube.com/embed/SVMcdmlakxM?rel=0&enablejsapi=1&origin=https://yshimod.github.io/" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe></p>


- オンプレミス（e.g., 社研サーバー），IaaS（e.g., Amazon EC2, Google Compute Engine），VPS（e.g., ConoHa）などのLinuxサーバーで動かす．
    - [Ubuntuで oTree を動かす](server_setup/README.md)
- PaaS（e.g., Heroku）で動かす（ oTree 公式のおすすめ）．
    - [Herokuで oTree を動かす](heroku/README.md)
- Windowsで動かす（たとえば，ラボで z-Tree を動かしていたようなマシンで）．
    - [https://otree.readthedocs.io/en/latest/install-windows.html#install-windows](https://otree.readthedocs.io/en/latest/install-windows.html#install-windows)



## 3. Ubuntuに慣れる

<p class="ytubevideo"><iframe width="560" height="315" src="https://www.youtube.com/embed/ys42gwdtefE?rel=0&enablejsapi=1&origin=https://yshimod.github.io/" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe></p>


- シェルBashの使い方
    - [https://learnxinyminutes.com/docs/bash/](https://learnxinyminutes.com/docs/bash/)
    - 以下のコマンドが使えるようになっておくとよい．
        - 現在のディレクトリを表示 `pwd`
        - ファイルの表示 `ls`
        - ディレクトリの移動 `cd`
        - ファイル，ディレクトリのコピー `cp`
        - ファイル，ディレクトリの移動 `mv`
        - ファイルの中身の表示 `cat`, `more`, `less`
        - ディレクトリの生成 `mkdir`
        - ファイルの消去 `rm`
        - リダイレクト `>`（上書き）, `>>`（追記）
        - パイプ `|`

- テキストエディタVimの使い方
    - [https://learnxinyminutes.com/docs/vim/](https://learnxinyminutes.com/docs/vim/)
    - 以下の操作ができるようになっておくとよい．
        - `vim test.txt` でVimを起動しtest.txtを編集する．
        - `i` を押してインプットモードに入る．
        - `Ecs` を押してインプットモードからノーマルモードに戻る．
        - ノーマルモードで `:q!` を押して編集内容を破棄して終了．
        - ノーマルモードで `:x` を押して編集内容を保存して終了．
    - `vimtutor` コマンドでチュートリアル開始．
    - ドットインストール vim入門 [https://dotinstall.com/lessons/basic_vim](https://dotinstall.com/lessons/basic_vim)



## 4. Pythonの導入からpipで oTree をインストールするまで

<p class="ytubevideo"><iframe width="560" height="315" src="https://www.youtube.com/embed/AlKiPuN4gYg?rel=0&enablejsapi=1&origin=https://yshimod.github.io/" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe></p>


- [詳しくはこちら](server_setup/README.md#python-を-pyenv--virtualenv-で導入)

- 動画内で少々グダついてしまったので，補足します．  
    17:35あたりから，pipで oTree とpsycopg2をインストールしようとしていますが，psycopg2のインストールに失敗しています．
    状況を詳しく説明すると，[資料](server_setup/README.md#python-を-pyenv--virtualenv-で導入)ではまっさらなUbuntuにpyenv+virtualenvを使ってPythonをインストールし，そのPython（pip）を使って oTree とpsycopg2をインストールしようとしているのに対し，動画ではMacで，私が予めhomebrewを使って入れていたpyenv+virtualenvを使って，同様のことをしようとしています．
    Pythonのビルドに必要なもののインストールをhomebrewは勝手にやってくれるのですが，その勝手にやってくれていた部分に不具合があり（[https://qiita.com/sukapontan/items/4dccf3262bfd544e4204](https://qiita.com/sukapontan/items/4dccf3262bfd544e4204) 参照）psycopg2のインストールに失敗したようでした．
    結局動画では，psycopg2をインストールせず oTree のみをインストールして，先に進めました．

- 【フリーライド】立命館竹内研長瀬さんのチュートリアル動画も参考になります．
    - 「#2 oTree Macでのインストール」:  
        homebrewを使ってpyenvをインストールし，pyenvでPythonをインストールしています．
        Macユーザーはこの方法がベストかと思われます．
        （動画リンクは竹内研Slack内を検索してください．）



## 5. 本番として oTree を動かすために必要なこと

<p class="ytubevideo"><iframe width="560" height="315" src="https://www.youtube.com/embed/oTbznkpAvjM?rel=0&enablejsapi=1&origin=https://yshimod.github.io/" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe></p>


- [詳しくはこちら](server_setup/README.md#本番としてotreeを動かすために必要なこと)

- 「 oTree 勉強会」なる催し物の第1回でやるべき内容ではなかったかもしれませんが，お付き合いください．

- 動画の中で環境変数の設定について説明していますが，ボケたことを言っていますね．
    11:38あたりで「MacのBash」うんぬんと言っていますが，どんなOSであれBashはBashなので，おかしいです．
    **以下にBashにおける環境変数の設定について整理しました**．
    - 環境変数とは，シェル（Bash）における変数で，子プロセス（ oTree ）に引き継がれるもの．
    - Bashでは `export` コマンドで変数名を定義（かつ `=` で値を代入）する．E.g., `export OTREE_PRODUCTION=1`.
    - `otree` コマンドで oTree を起動する前に，一々 `export` コマンドで環境変数を設定しても良いが，毎度同じコマンドを入力するのは面倒．
    - シェル（Bash）を起動するたびに読み込む設定ファイルに環境変数の設定を書いておこう．
    - ログイン時に適用されるものは `~/.profile`，ターミナルでBashを起動した際に適用されるものは `~/.bashrc`．
    - ↑ のどちらかに書いておけば良い．
    - （参考 [https://techracho.bpsinc.jp/hachi8833/2021_07_08/66396](https://techracho.bpsinc.jp/hachi8833/2021_07_08/66396)）
    - Bashの場合のお話なので，Macユーザーでzshを使っている場合はzsh用の設定ファイルの仕様を確認してください（またはBashを使ってください）．



## 6. oTree サーバーをインターネットにさらす方法 (の概要)

<p class="ytubevideo"><iframe width="560" height="315" src="https://www.youtube.com/embed/9Hzzzg2-xX8?rel=0&enablejsapi=1&origin=https://yshimod.github.io/" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe></p>


- [詳しくはこちら](server_setup/README.md#ネットワークの設定)

- 「 oTree 勉強会」なる催し物の第1回でやるべき内容ではないし，Herokuで十分な人にとっても不要な内容かも分かりませんが，お付き合いください．

- もしも日本がどこぞに侵略戦争をしかけて各国からあらゆる制裁が加えられたときには自前のサーバーを建てなければならないでしょうから，そのときには．

- WSL2を使っている場合，ネットワークの設定の段階まで来るとややこしさが急激に上昇すると思われます．Microsoftのドキュメント（ [https://docs.microsoft.com/ja-jp/windows/wsl/networking](https://docs.microsoft.com/ja-jp/windows/wsl/networking) ）を参照して，ポートフォワーディングの設定等をする必要があります．



## 7. Herokuで oTree を動かす方法 (さわりだけ)

<p class="ytubevideo"><iframe width="560" height="315" src="https://www.youtube.com/embed/3Dzm6s8nPDA?rel=0&enablejsapi=1&origin=https://yshimod.github.io/" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe></p>


- [詳しくはこちら](heroku/README.md)

- oTree の著者たちにお金を払ってもよい場合は oTree Hub のHeroku連携機能を使っても良いでしょう．

- oTree Hub を使わない場合， oTree のプログラムを自分でHerokuのサーバーにアップロードする必要があります．この際に多かれ少なかれ，Gitの知識が必要となります．

- 「**Gitを使う必然性は何か？**」という質問をいただきました．
    - oTree Hubに課金できる場合には，Gitの知識がなくてもとりあえずプログラムをデプロイして実験は実施できます．
    - oTree Hubを使わずにHerokuを使う場合，プログラムのデプロイのためにGitの使用が必須となります．
    - ただし，（特にHeroku CLIを使ってHerokuに直接デプロイする場合は）ファイルのアップロードの手段としてGitを使っているという面が大きく，Gitの本来の目的（バージョン管理）からは逸れていることを認識しておくべきでしょう．
    - バージョン管理ツールとしてのGit，あるいはソーシャルコーディングのプラットフォームとしてのGitHubを使うべきか否か，については， oTree Hub 課金うんぬんとはまったく別に議論されるべきことだと思います．
    - 仮に oTree Hub に課金する場合であっても， oTree のプロジェクトはGitHubを使って管理するのが良いのではないかと思っています．
