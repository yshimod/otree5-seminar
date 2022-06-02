# UbuntuでoTreeを動かす

- [UbuntuでoTreeを動かす](#ubuntuでotreeを動かす)
  - [目的](#目的)
      - [Linuxの使い方に慣れましょう](#linuxの使い方に慣れましょう)
      - [屋上屋を架する愚は避ける](#屋上屋を架する愚は避ける)
  - [参考](#参考)
  - [Linuxマシンを用意する](#linuxマシンを用意する)
  - [Python を pyenv (+ virtualenv) で導入 <a id="pythoninstallation"></a>](#python-を-pyenv--virtualenv-で導入-)
  - [pipでoTreeのインストール <a id="otreeinstallation"></a>](#pipでotreeのインストール-)
  - [本番としてoTreeを動かすために必要なこと <a id="postinstallation"></a>](#本番としてotreeを動かすために必要なこと-)
      - [データベースサーバーソフトウェアの PostgreSQL を導入](#データベースサーバーソフトウェアの-postgresql-を導入)
      - [Redis は不要](#redis-は不要)
      - [環境変数の設定 <a id="envvar"></a>](#環境変数の設定-)
      - [ネットワークの設定 <a id="webserver"></a>](#ネットワークの設定-)


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


## 参考
- 公式ドキュメント [https://otree.readthedocs.io/en/latest/server/ubuntu.html](https://otree.readthedocs.io/en/latest/server/ubuntu.html)
- 情報が古いが...
    - [https://otree-server-setup.readthedocs.io/en/latest/index.html](https://otree-server-setup.readthedocs.io/en/latest/index.html)
    - [https://otreecb.netlify.app/reference/ubuntu_server_setup.html](https://otreecb.netlify.app/reference/ubuntu_server_setup.html)


## Linuxマシンを用意する  
以下のいずれかの方法で．

- isoファイルを公式Webページからダウンロードし，USBメモリに（ブータブルな形式で）コピーする．それを使って実機にインストール．
- VPSやIaaSを契約してインスタンスを作成．
- [Ubuntuの仮想環境の構築](../ubuntu/README.md)


## Python を pyenv (+ virtualenv) で導入 <a id="pythoninstallation"></a>

<iframe width="560" height="315" src="https://www.youtube-nocookie.com/embed/xOPHDOUsg0c?rel=0" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

- UbuntuはシステムにデフォルトでPythonが入っているので，それを使っても良いが，おすすめしない．
- Pythonを導入するのに pyenv + virtualenv を必ず使わなければならないわけではない．
    - Pythonを（直接入れるのではなく）pyenvを使って入れるメリット:
        - pythonのバージョンをすぐに切り替えられる（oTree3はPython3.8まででしか動かない）．
        - ホームディレクトリにPythonを入れるので，システムのデフォルトのPythonに干渉しない．
    - virtualenvを使うメリット:
        - oTree専用（あるいは特定のプロジェクト専用）の環境を構築できる（Pythonのパッケージをいろいろ入れるとわけが分からなくなる）．
        - pyenv自体はPythonのバージョンの切り替えをするもの．virtualenvは同一バージョンで複数の環境を構築できる．


1. Pythonのビルド環境（build-essential等）をaptでインストール  
    [https://github.com/pyenv/pyenv/wiki](https://github.com/pyenv/pyenv/wiki)
    ```bash
    sudo apt-get update
    sudo apt-get install make build-essential libssl-dev zlib1g-dev libbz2-dev libreadline-dev libsqlite3-dev wget curl llvm libncursesw5-dev xz-utils tk-dev libxml2-dev libxmlsec1-dev libffi-dev liblzma-dev
    ```
1. pyenvを導入  
    [https://github.com/pyenv/pyenv](https://github.com/pyenv/pyenv)
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
1. pyenvのプラグインpyenv-virtualenvを導入  
    [https://github.com/pyenv/pyenv-virtualenv](https://github.com/pyenv/pyenv-virtualenv)
    ```bash
    git clone https://github.com/pyenv/pyenv-virtualenv.git $(pyenv root)/plugins/pyenv-virtualenv
    echo 'eval "$(pyenv virtualenv-init -)"' >> ~/.bashrc
    ```
    - 導入が終わったら，いったんシェルを開き直す．
1. pyenvでPythonをインストール
    ```bash
    pyenv install 3.9.11
    ```
1. pyenv-virtualenvでoTree専用の環境（myotree5と名付ける）を作成
    ```bash
    pyenv virtualenv 3.9.11 myotree5
    ```
1. 用意した環境myotree5を使うように切り替える
    ```bash
    pyenv global myotree5
    ```
    - `python -V` でインストールしたバージョン（3.9.11）が表示されたら，ちゃんと設定できている．


## pipでoTreeのインストール <a id="otreeinstallation"></a>

1. （PostgreSQLが必要な場合）パッケージpsycopg2を入れるために必要なライブラリlibpq-devをインストール  
    [https://www.psycopg.org/docs/](https://www.psycopg.org/docs/)
    ```bash
    sudo apt-get install libpq-dev
    ```
1. pipでoTree（とpsycopg2）をインストール
    ```bash
    pip install otree psycopg2
    ```
    - パッケージのバージョンを特定する場合は `otree==5.8.1` などとする．
    - `which otree` でパスが表示されたら，ちゃんとインストールできている．
    - 何か必要なものが入っていないと，あるいは依存関係の問題で，psycopg2のインストールは失敗するかも．
        - 根気強くエラーを読んで，検索して，必要なものを入れてから再チャレンジ．
        - psycopg2のバージョンを少し古いものに固定してみる．
        - PostgreSQLを使う予定がない場合にはpsycopg2は入れる必要なし．とりあえず諦める．
1. oTreeを動かしてみる
    - まずはシェルでoTreeのバージョンを確認．
        ```bash
        otree --version
        ```
    - 試しに新規プロジェクトの作成（たとえば新規プロジェクトの名前を `my_oTree` とする）．
        ```bash
        otree startproject my_oTree
        ```
        - とりあえずサンプルゲームも入れる（「Include sample games?」聞かれたら `y` と答える）．
    - `my_oTree` なるディレクトリが作成されるので，その中に入る．
        ```bash
        # my_oTree に入る
        cd my_oTree
        # my_oTreeの中身を確認する
        ls -la
        ```
    - oTreeがサーバーとして動くのか確認．
        ```bash
        otree prodserver
        ```
        - ブラウザで `http://localhost:8000` にアクセスできるか？
            - WSL2上で動かしている場合はlocalhostでアクセスできるはず．  
                （何となれば... [https://docs.microsoft.com/ja-jp/windows/wsl/networking#accessing-linux-networking-apps-from-windows-localhost](https://docs.microsoft.com/ja-jp/windows/wsl/networking#accessing-linux-networking-apps-from-windows-localhost)）
            - Vagrantで入れたUbuntuや，遠隔地のサーバー（SSHで接続中）で動かしている場合は，localhostではなく動かしているサーバーのIPアドレスを陽に指定してアクセスしてみる．


## 本番としてoTreeを動かすために必要なこと <a id="postinstallation"></a>

#### データベースサーバーソフトウェアの PostgreSQL を導入

<iframe width="560" height="315" src="https://www.youtube-nocookie.com/embed/uYNYo1IICvA?rel=0" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

- oTree公式はPostgreSQLを使うことを推奨
- PostgreSQLを使わない場合，SQLite3を使う．
- SQLite3でもパフォーマンスは悪くない？（要検証）
- Herokuでの動作環境を再現するにはPostgreSQLを使う（HerokuはPostgreSQL必須）．


#### Redis は不要

- oTreeの以前のバージョンではRedisの導入が必須だったが，v3.3.4以降（しれっと）不要になった．


#### 環境変数の設定 <a id="envvar"></a>

- （Bashを使っている場合は）.bashrcに環境変数を設定．
    ```bash
    # oTree実験者画面へのパスワードをたとえば000999にしてみる．本当はもっと複雑なものにする．
    # ユーザー名のデフォルトは admin
    export OTREE_ADMIN_PASSWORD=000999

    # OTREE_AUTH_LEVEL=STUDY とするとパスワード保護が有効になる．
    # パスワードをOTREE_ADMIN_PASSWORDで設定していないのにOTREE_AUTH_LEVEL=STUDYとするとログインできなくなる．
    export OTREE_AUTH_LEVEL=STUDY

    # OTREE_PRODUCTIONが未定義であったり値が0であったりするとDEBUGモードになり，デバッグ用の情報が表示される．
    # OTREE_PRODUCTION=1 とすることでデバッグ用の情報を非表示にする．
    export OTREE_PRODUCTION=1

    # PostgreSQL が設定済みであれば，
    #    データベースにアクセスするロール名（たとえばuser01）
    #    パスワード（たとえば0099）
    #    アドレス（oTreeと同じマシンで動かす場合はlocalhost）
    #    データベース名（たとえばotreedb）
    # をoTreeに環境変数DATABASE_URLで教える．
    export DATABASE_URL=postgres://user01:0099@localhost/otreedb
    # DATABASE_URLを設定しなければoTreeはSQLiteを使う．
    ```
    - Vim（vi）を使って.bashrcを編集できるようにしておいた方が良い．nanoでも良いが．
    - PostgreSQLのURL（`DATABASE_URL`）にパスワード（例では0099）を書いても良い．そのときはPostgreSQLのパスワードファイル.pgpassが不要．
    - oTreeの公式ドキュメントでは言及されていないが，settings.pyの `SECRET_KEY` も環境変数から受け取るように変えた方が良い気がする．


#### ネットワークの設定 <a id="webserver"></a>
- [https://otreecb.netlify.app/reference/ubuntu_server_setup.html#step-3-install-nginx](https://otreecb.netlify.app/reference/ubuntu_server_setup.html#step-3-install-nginx) が詳しい．
- WSL2のUbuntuを使っている場合はポートフォワードを設定する必要がある．
    - [https://docs.microsoft.com/ja-jp/windows/wsl/networking](https://docs.microsoft.com/ja-jp/windows/wsl/networking)
    - [https://bayashi.net/diary/2020/1121](https://bayashi.net/diary/2020/1121)
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
