# UbuntuでoTreeを動かす

- 公式ドキュメント [https://otree.readthedocs.io/en/latest/server/ubuntu.html](https://otree.readthedocs.io/en/latest/server/ubuntu.html)
- 情報が古いが...
    - [https://otree-server-setup.readthedocs.io/en/latest/index.html](https://otree-server-setup.readthedocs.io/en/latest/index.html)
    - [https://otreecb.netlify.app/reference/ubuntu_server_setup.html](https://otreecb.netlify.app/reference/ubuntu_server_setup.html)


## 目的

- Windowsではなく，LinuxやmacOSなどのUNIX系OSでoTreeを動かしたい．Windowsがダメなのではないが......

#### Linuxの使い方に慣れましょう
- サーバ用のOSはLinuxを採用することが多い．
- 社研ラボのoTree用サーバのOSはUbuntu（Linux OSの中で一番有名？）．
- AWSのEC2やGCPのCompute EngineのOSもたいていLinux．

#### 屋上屋を架する愚は避ける
- oTreeの使い方を解説する（初級者向け）ドキュメントはたいていWindowsを使っている．
- Windowsに直接Python・oTreeを入れる方法は公式ドキュメント [https://otree.readthedocs.io/en/latest/install-windows.html#install-windows](https://otree.readthedocs.io/en/latest/install-windows.html#install-windows) を参照してください．
- UNIXな環境を前提としたドキュメントは少ない．公式ドキュメントでも説明が雑（MacやLinuxが使える人に詳しい説明は不要と思われている？その推論はいくらか正しいとしても......）．


## マシンを用意してLinuxを入れる

- isoファイルを公式Webページからダウンロードし，USBメモリに（ブータブルな形式で）コピーする．それを使って実機にインストール．
- [Ubuntuの仮想環境の構築](../ubuntu/README.md)


## シェルBashの使い方

- [https://learnxinyminutes.com/docs/bash/](https://learnxinyminutes.com/docs/bash/)
- 以下のコマンドが使えるようになっておく．
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

## テキストエディタVimの使い方

- [https://learnxinyminutes.com/docs/vim/](https://learnxinyminutes.com/docs/vim/)
- `vimtutor` コマンドでチュートリアル開始．
- `vim test.txt` でVimを起動しtest.txtを編集する．
- `i` を押してインプットモードに入る．
- `Ecs` を押してインプットモードからノーマルモードに戻る．
- ノーマルモードで `:q!` を押して編集内容を破棄して終了．
- ノーマルモードで `:x` を押して編集内容を保存して終了．


<hr>

## Python を pyenv (+ virtualenv) で導入

<iframe width="560" height="315" src="https://www.youtube.com/embed/xOPHDOUsg0c" frameborder="0" allowfullscreen></iframe>

- UbuntuはシステムにデフォルトでPythonが入っているので，それを使っても良いが，おすすめしない．
- Pythonを導入するのに pyenv + virtualenv を必ず使わなければならないわけではない．


1. Pythonのビルド環境（build-essential等）をaptでインストール  
    [https://github.com/pyenv/pyenv/wiki](https://github.com/pyenv/pyenv/wiki)

    ```bash
    sudo apt-get update
    sudo apt-get install make build-essential libssl-dev zlib1g-dev libbz2-dev libreadline-dev libsqlite3-dev wget curl llvm libncursesw5-dev xz-utils tk-dev libxml2-dev libxmlsec1-dev libffi-dev liblzma-dev
    ```


2. pyenvを導入  
    [https://github.com/pyenv/pyenv](https://github.com/pyenv/pyenv)
    - Pythonを（直接入れるのではなく）pyenvを使って入れるメリット:
        - pythonのバージョンをすぐに切り替えられる（oTree3はPython3.8まででしか動かない）．
        - ホームディレクトリにPythonを入れるので，システムのデフォルトのPythonに干渉しない．

    ```bash
    git clone https://github.com/pyenv/pyenv.git ~/.pyenv
    cd ~/.pyenv && src/configure && make -C src
    echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.bashrc
    echo 'command -v pyenv >/dev/null || export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.bashrc
    echo 'eval "$(pyenv init -)"' >> ~/.bashrc
    echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.profile
    echo 'command -v pyenv >/dev/null || export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.profile
    echo 'eval "$(pyenv init -)"' >> ~/.profile
    ```

    - 導入が終わったら，いったんシェルを開き直す．


3. pyenvのプラグインpyenv-virtualenvを導入  
    [https://github.com/pyenv/pyenv-virtualenv](https://github.com/pyenv/pyenv-virtualenv)
    - virtualenvを使うメリット:
        - oTree専用（あるいは特定のプロジェクト専用）の環境を構築できる（Pythonのパッケージをいろいろ入れるとわけが分からなくなる）．
        - pyenv自体はPythonのバージョンの切り替えをするもの．virtualenvは同一バージョンで複数の環境を構築できる．

    ```bash
    git clone https://github.com/pyenv/pyenv-virtualenv.git $(pyenv root)/plugins/pyenv-virtualenv
    echo 'eval "$(pyenv virtualenv-init -)"' >> ~/.bashrc
    ```

    - 導入が終わったら，いったんシェルを開き直す．


4. pyenvでPythonをインストール

    ```bash
    pyenv install 3.9.11
    ```


5. pyenv-virtualenvでoTree専用の環境（myotree5と名付ける）を作成

    ```bash
    pyenv virtualenv 3.9.11 myotree5
    ```


6. 用意した環境myotree5を使うように切り替える

    ```bash
    pyenv global myotree5
    ```

    - `python -V` でインストールしたバージョン（3.9.11）が表示されたら，ちゃんと設定できている．


7. パッケージpsycopg2を入れるために必要なライブラリlibpq-devをインストール（PostgreSQLが必要な場合）  
    [https://www.psycopg.org/docs/](https://www.psycopg.org/docs/)

    ```bash
    sudo apt-get install libpq-dev
    ```


- pipでoTree（とpsycopg2）をインストール

    ```bash
    pip install otree psycopg2
    ```

    - パッケージのバージョンを特定する場合は `otree==5.8.1` などとする．
    - `which otree` でパスが表示されたら，ちゃんとインストールできている．
    - `otree startproject my_oTree` でoTreeの空プロジェクトを作成（サンプルゲームも入れる）．
    - この時点で `otree devserver` でサーバーが動くのか確認．my_oTreeディレクトリに移動してからコマンドを叩く．


## データベースサーバーソフトウェアの PostgreSQL を導入

<iframe width="560" height="315" src="https://www.youtube.com/embed/uYNYo1IICvA" frameborder="0" allowfullscreen></iframe>

- oTree公式はPostgreSQLを使うことを推奨
- PostgreSQLを使わない場合，SQLite3を使う．
- SQLite3でもパフォーマンスは悪くない？（要検証）
- Herokuでの動作環境を再現するにはPostgreSQLを使う（HerokuはPostgreSQL必須）．


## Redis は不要

- oTreeの以前のバージョンではRedisの導入が必須だったが，v3.3.4以降（しれっと）不要になった．


## 環境変数の設定

- （Bashを使っている場合は）.bashrcに環境変数を設定．
- Vim（vi）を使って.bashrcを編集できるようにしておいたほうが良い．nanoでも良いが．

    ```bash
    export OTREE_ADMIN_PASSWORD=000999
    export OTREE_PRODUCTION=1
    export OTREE_AUTH_LEVEL=STUDY
    export DATABASE_URL=postgres://user01:0099@localhost/otreedb
    ```

- `otree prodserver` で本番開始．
- PostgreSQLのURLにパスワード（例では0099）を書いても良い．そのときはPostgreSQLのパスワードファイル.pgpassが不要．
- oTreeの公式ドキュメントでは言及されていないが，settings.pyの `SECRET_KEY` も環境変数から受け取るように変えたほうが良い気がする．


## ネットワークの設定

- サーバーのIPアドレスを固定する．
- ファイアウォール（ufw）で開放するポートを設定．
    - LAN内で完結する場合（学内のラボなど）はあまり気にしなくて良い．
    - インターネットにさらす場合は慎重に設定する．oTreeのデフォルトのポート8000は開放しない．
- インターネットにさらす場合は...
    - ドメイン取得（SSL証明書の発行のために必須）．
    - nginx（Webサーバーソフトウェア）をリバースプロキシ（インターネットからはポート443で受け入れて，それをlocalhost:8000に渡す）として設定．
    - HTTPSで通信できるようにcertbotで証明書発行．
- oTreeサーバー稼働中のプロセス管理
    - nohupを使うとSSHの接続が切れてもプロセスが閉じない．
    - SupervisorやCircusを使うのが良い，らしい．