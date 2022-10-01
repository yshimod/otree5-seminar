# Ubuntuで oTree を動かす

## Linuxマシンを用意する  
以下のいずれかの方法で．

- 実機を使う場合，isoファイルを公式WebページからダウンロードしてUSBメモリに（ブータブルな形式で）コピーし，それを使ってインストール．
- VPS（ConoHa）やIaaS（Amazon EC2）を契約してインスタンスを作成．
- [Ubuntuの仮想環境の構築](../ubuntu/README.md)


## Linux（Ubuntu）をインストールした後にやっておくこと

以下は備忘録．一度も以下のような作業を経験したことがなければ，より詳しい文献に当たるべし．

- ユーザーの作成
    - 作業用のユーザー（たとえば `lab` ）を作成する．
      ```
      sudo adduser lab
      ```
    - sudo権限を付与する．
      ```
      sudo usermod -aG sudo lab
      ```

- SSHの設定
    - SSH公開鍵認証で接続できるように準備する．
        - （Ubuntuサーバー上ではなく）ローカルのPCで鍵を作成する．
          ```
          ssh-keygen -t ed25519
          ```
        - 生成した秘密鍵（たとえば `id_ed25519` ）の権限を `600` にする．
          ```
          chmod 600 id_ed25519
          ```
        - 生成した公開鍵（ `*.pub`，たとえば `id_ed25519.pub` ）を，Ubuntuサーバーに転送する．
            - `*.pub` ファイルを何らかの方法で（実機があればUSBメモリを使う，すでにSSH接続できるのであれば `scp` コマンドを使うなどして）Ubuntuサーバーの自分のホームディレクトリかどこかに転送しておく．
            - ホームディレクトリの中に `.ssh` ディレクトリを（無ければ）作成．
            - `.ssh` ディレクトリに `authorized_keys` なるテキストファイルを作成し，そこに公開鍵の中身を書く．
                - `authorized_keys` ファイルが存在していなければ， `*.pub` ファイルを `authorized_keys` にリネームしてしまう．
                  ```
                  mv id_ed25519.pub authorized_keys
                  ```
                - `authorized_keys` ファイルが存在していれば，追記する．
                  ```
                  cat id_ed25519.pub >> authorized_keys
                  rm id_ed25519.pub
                  ```
            - `authorized_keys` の権限は `600` にする．
              ```
              chmod 600 authorized_keys
              ```
    - `/etc/ssh/sshd_config` を編集する．
      ```
      sudo vim /etc/ssh/sshd_config
      ```
        - SSH用のポートをデフォルト（`22`）から変更する．
          ```
          Port 2222
          ```
            - 「 `Port 22` 」と書かれている行はコメントアウトしておく．
        - SSHでrootにログインできないようにする．
          ```
          PermitRootLogin no
          ```
        - パスワードでログインできないようにする．
          ```
          PasswordAuthentication no
          ```
        - 公開鍵でログインできるようにする．
          ```
          PubkeyAuthentication yes
          ```
    - 諸々の設定の終了後，シェルから出たりマシンを再起動したりする前に，SSHでアクセスできるかどうか確認しておく．SSHの設定が間違っているとアクセスできなくなってしまう．
        - アドレスが `133.1.112.999`， ユーザー名が `lab`， ポート番号が `2222`， 秘密鍵が `~/.ssh/id_ed25519` の場合，以下のコマンドでログインできる．
          ```
          ssh -i ~/.ssh/id_ed25519 -p 2222 lab@133.1.112.999
          ```

- ファイアウォールの設定
    - ufwを起動する．
      ```
      sudo ufw enable
      ```
    - すべてのポートを閉じる（`deny` にする... デフォルトの設定 `reject` でも良いけど）．
      ```
      sudo ufw default deny
      ```
    - SSHのデフォルトのポート番号（`22`）を削除する．
      ```
      sudo ufw delete allow 22
      ```
    または
      ```
      sudo ufw delete allow "OpenSSH"
      ```
    - 自分で設定したSSHのポート番号（たとえば `2222`）を許可する．
      ```
      sudo ufw allow 2222
      ```
    - oTree 用のポート番号（たとえば `8000`）を許可しておいても良い．
      ```
      sudo ufw allow 8000
      ```
    - 設定が終わったら反映させる．
      ```
      sudo ufw reload
      ```
    - 諸々の設定の終了後，シェルから出たりマシンを再起動したりする前に，SSHでアクセスできるかどうか確認しておく．ファイアウォールの設定が間違っているとアクセスできなくなってしまう．


## Python を pyenv (+ virtualenv) で導入

<p class="ytubevideo"><iframe width="560" height="315" src="https://www.youtube.com/embed/xOPHDOUsg0c?rel=0&enablejsapi=1&origin=https://yshimod.github.io/" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe></p>

- UbuntuはシステムにデフォルトでPythonが入っているので，それを使っても良いが，おすすめしない．
- Pythonを導入するのに pyenv + virtualenv を必ず使わなければならないわけではない．
    - Pythonを（直接入れるのではなく）pyenvを使って入れるメリット:
        - pythonのバージョンをすぐに切り替えられる（ oTree 3 はPython3.8まででしか動かない）．
        - ホームディレクトリにPythonを入れるので，システムのデフォルトのPythonに干渉しない．
    - virtualenvを使うメリット:
        - oTree 専用（あるいは特定のプロジェクト専用）の環境を構築できる（Pythonのパッケージをいろいろ入れるとわけが分からなくなる）．
        - pyenv自体はPythonのバージョンの切り替えをするもの．virtualenvは同一バージョンで複数の環境を構築できる．


1. Pythonのビルド環境（build-essential等）をaptでインストール  
[https://github.com/pyenv/pyenv/wiki](https://github.com/pyenv/pyenv/wiki)
  ```
  sudo apt-get update
  sudo apt-get install make build-essential libssl-dev zlib1g-dev libbz2-dev libreadline-dev libsqlite3-dev wget curl llvm libncursesw5-dev xz-utils tk-dev libxml2-dev libxmlsec1-dev libffi-dev liblzma-dev
  ```
1. pyenvを導入  
[https://github.com/pyenv/pyenv](https://github.com/pyenv/pyenv)
  ```
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
  ```
  git clone https://github.com/pyenv/pyenv-virtualenv.git $(pyenv root)/plugins/pyenv-virtualenv
  echo 'eval "$(pyenv virtualenv-init -)"' >> ~/.bashrc
  ```
    - 導入が終わったら，いったんシェルを開き直す．
1. pyenvでPythonをインストール
  ```
  pyenv install 3.9.11
  ```
1. pyenv-virtualenvでoTree専用の環境（myotree5と名付ける）を作成
  ```
  pyenv virtualenv 3.9.11 myotree5
  ```
1. 用意した環境myotree5を使うように切り替える
  ```
  pyenv global myotree5
  ```
    - `python -V` でインストールしたバージョン（3.9.11）が表示されたら，ちゃんと設定できている．


## pipで oTree のインストール

1. （PostgreSQLが必要な場合）パッケージpsycopg2を入れるために必要なライブラリlibpq-devをインストール  
[https://www.psycopg.org/docs/](https://www.psycopg.org/docs/)
  ```
  sudo apt-get install libpq-dev
  ```
1. pipで oTree （とpsycopg2）をインストール
  ```
  pip install otree psycopg2
  ```
    - パッケージのバージョンを特定する場合は `otree==5.8.1` などとする．
    - `which otree` でパスが表示されたら，ちゃんとインストールできている．
    - 何か必要なものが入っていないと，あるいは依存関係の問題で，psycopg2のインストールは失敗するかも．
        - 根気強くエラーを読んで，検索して，必要なものを入れてから再チャレンジ．
        - psycopg2のバージョンを少し古いものに固定してみる．
        - PostgreSQLを使う予定がない場合にはpsycopg2は入れる必要なし．とりあえず諦める．
1. oTree を動かしてみる
    - まずはシェルでoTreeのバージョンを確認．
      ```
      otree --version
      ```
    - 試しに新規プロジェクトの作成（たとえば新規プロジェクトの名前を `my_oTree` とする）．
      ```
      otree startproject my_oTree
      ```
        - とりあえずサンプルゲームも入れる（「Include sample games?」聞かれたら `y` と答える）．
    - `my_oTree` なるディレクトリが作成されるので，その中に入る．
      ```
      # my_oTree に入る
      cd my_oTree
      # my_oTreeの中身を確認する
      ls -la
      ```
    - oTreeがサーバーとして動くのか確認．
      ```
      otree prodserver
      ```
        - ブラウザで `http://localhost:8000` にアクセスできるか？
            - WSL2上で動かしている場合はlocalhostでアクセスできるはず．  
                （何となれば... [https://docs.microsoft.com/ja-jp/windows/wsl/networking#accessing-linux-networking-apps-from-windows-localhost](https://docs.microsoft.com/ja-jp/windows/wsl/networking#accessing-linux-networking-apps-from-windows-localhost)）
            - Vagrantで入れたUbuntuや，遠隔地のサーバー（SSHで接続中）で動かしている場合は，localhostではなく動かしているサーバーのIPアドレスを陽に指定してアクセスしてみる．


## 本番としてoTreeを動かすために必要なこと

### データベースサーバーソフトウェアの PostgreSQL を導入

<p class="ytubevideo"><iframe width="560" height="315" src="https://www.youtube.com/embed/uYNYo1IICvA?rel=0&enablejsapi=1&origin=https://yshimod.github.io/" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe></p>

- oTree公式はPostgreSQLを使うことを推奨
- PostgreSQLを使わない場合，SQLite3を使う．
- SQLite3でもパフォーマンスは悪くない？（要検証）
- Herokuでの動作環境を再現するにはPostgreSQLを使う（HerokuはPostgreSQL必須）．


### Redis は不要

- oTreeの以前のバージョンではRedisの導入が必須だったが，v3.3.4以降（しれっと）不要になった．


### 環境変数の設定

- （Bashを使っている場合は）.bashrcに環境変数を設定．
  ```
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
  # を oTree に環境変数DATABASE_URLで教える．
  export DATABASE_URL=postgres://user01:0099@localhost/otreedb
  # DATABASE_URLを設定しなければ oTree はSQLiteを使う．
  ```
    - Vim（vi）を使って.bashrcを編集できるようにしておいた方が良い．nanoでも良いが．
    - PostgreSQLのURL（`DATABASE_URL`）にパスワード（例では0099）を書いても良い．そのときはPostgreSQLのパスワードファイル.pgpassが不要．
    - oTree の公式ドキュメントでは言及されていないが，settings.pyの `SECRET_KEY` も環境変数から受け取るように変えた方が良い気がする．


## インターネット（LAN外）からアクセスできるようにする

- oTreeが動いているマシンにグローバルIPアドレスが割り当てられていて，oTreeのポート（8000）がインターネットに開放されていれば，特別な設定をしなくてもインターネットからoTreeにアクセスできる．
- しかし，そのような状況は稀である．
    - 何も設定していないのにインターネットから接続できてしまっている場合は逆に改めたほうが良い．

- WSL2のUbuntuを使っている場合は設定が難しそうなので，ここでは諦める．
    - LAN内でアクセスできるようにするだけでもポートフォワーディングの設定が必要．

- 参考: [https://otreecb.netlify.app/reference/ubuntu_server_setup.html#step-3-install-nginx](https://otreecb.netlify.app/reference/ubuntu_server_setup.html#step-3-install-nginx)


### グローバルIPアドレスの確保
- インターネットからアクセスするためには固定されたグローバルIPアドレスが必要．
- 自宅サーバーで固定されたグローバルIPアドレスが無い場合はDDNSを契約する．
- 学内サーバーの場合は担当者（計算機室，情報基盤センター的な部局の窓口）にお願いする．
- グローバルIPアドレスが割り当てられているマシンの下にLANがあり（要するに，LANのゲートウェイにグローバルIPアドレスが割り当てられている），そのLAN内のマシンを oTree のサーバーにする場合は，ポートフォワーディングの設定が必要．


### ドメインの取得
- HTTPS通信をするためにはSSL証明書が必要．SSL証明書を発行するためにはドメインが必要．
- 学内サーバーの場合は担当者（計算機室，情報基盤センター的な部局の窓口）にお願いする．
- Amazon EC2 を使う場合，Amazon Route 53 でドメインを取得すると良い．
- ドメインを取得後，DNSにグローバルIPアドレスとドメインの対応関係を登録する．
- 以下では `myotree.iser.osaka-u.ac.jp` なる（サブ）ドメインを取得したとする．


### Nginx の設定
- Nginx とは Webサーバーアプリケーション．「エンジンエックス」と読む．「エヌジンクス」と読むとバカにされる．
- そもそも oTree には Webサーバーの機能が含まれているものの，HTTP通信しかできない．
- Nginx をリバースプロキシとして使い， oTree サーバーとクライアント（ブラウザ）がHTTPS通信できるようにする．
    - インターネットから oTree サーバーが動いているマシンまでのHTTPS通信を，一旦 Nginx が受け取り，それを oTree に（`localhost:8000` に）渡す．


1. Nginx のインストール
  ```
  sudo apt install nginx
  ```

2. Nginx が自動起動するように設定
  ```
  sudo systemctl enable nginx
  ```

3. Nginx の通信のポートを開放する．
  ```
  sudo ufw allow 'Nginx Full'
  sudo ufw reload
  ```

4. Nginx のデフォルトの設定を削除する．
  ```
  sudo unlink /etc/nginx/sites-enabled/default
  ```
    - シンボリックリンクを削除しているだけで，設定ファイル自体は `/etc/nginx/sites-available/default` に残っているので安心して良い．

5. Nginx の設定ファイルを作成する．
  ```
  sudo vim /etc/nginx/sites-available/otree-reverse-proxy.conf
  ```
内容は以下．コピペすれば良い．
  ```
  map $http_upgrade $connection_upgrade {
      default upgrade;
      '' close;
  }

  server {
      access_log /var/log/nginx/otree-reverse-access.log;
      error_log /var/log/nginx/otree-reverse-error.log;

      ## 取得したドメインを設定する．
      server_name myotree.iser.osaka-u.ac.jp;

      location / {
          proxy_buffering off;
          proxy_pass http://localhost:8000;
          proxy_http_version 1.1;
          proxy_set_header Upgrade $http_upgrade;
          proxy_set_header Connection $connection_upgrade; 
          proxy_set_header HOST $host;
          proxy_set_header X-Real-Ip $remote_addr;
          proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
          proxy_set_header X-Forwarded-Host $server_name;
          proxy_set_header X-Forwarded-Proto $scheme;
          proxy_set_header X-Forwarded-Port $server_port;
      }
  }
  ```

6. 作成した設定ファイルをアクティブにする（`/etc/nginx/sites-enabled` ディレクトリにシンボリックリンクを張る）．
  ```
  sudo ln -s /etc/nginx/sites-available/otree-reverse-proxy.conf /etc/nginx/sites-enabled/otree-reverse-proxy.conf
  ```

7. 作成した設定ファイルを反映させる．
まず作成した設定ファイルに問題がないかテストする．
  ```
  sudo nginx -t
  ```
問題がなければ Nginx を再起動して設定を反映させる．
  ```
  sudo systemctl restart nginx
  ```


### SSL の設定
- HTTPS通信のためにSSL証明書を取得する．
- Let's Encrypt を使って，無料のTLS/SSL証明書を取得する．


1. Cerbotのインストール
  ```
  sudo apt install certbot python3-certbot-nginx
  ```

2. SSL証明書の取得
  ```
  sudo certbot --nginx -d myotree.iser.osaka-u.ac.jp
  ```
    - コマンド実行後，同意を求められたら同意する（`A` を入力してEnter）．
    - メールアドレスを聞いてもよいか質問されたらNoを返す（`N` を入力してEnter）で良い．
    - "Please choose whether or not to redirect HTTP traffic to HTTPS, removing HTTP access." が出たら Redirect を選ぶ（`2` を入力してEnter）．
        - 自動的に 先ほど作成した Nginx の設定ファイルを良きに計らって書き換えてくれる．

3. Nginx を再起動して設定を反映させる．
  ```
  sudo systemctl restart nginx
  ```

4. Let’s Encryptの証明書の有効期限は90日間．期限が残り30日になったタイミングでcertbotは自動的に証明書を更新してくれる．特に設定は必要ないが，念の為，改めて自動更新を有効にしておく．
  ```
  sudo systemctl enable certbot.timer
  ```


### oTree の起動
- Nginxの設定が終わったら，通常通り， oTree のプロジェクトディレクトリに移動して， oTree を起動する．
  ```
  otree prodserver 8000
  ```
- ブラウザで `https://myotree.iser.osaka-u.ac.jp` にアクセスして，いつもの oTree の画面が表示されれば成功．
    - Chrome であれば，アドレスバーの左側に南京錠のアイコンが表示されていれば正しくHTTPS通信ができている．
- SSH接続していて oTree を `otree prodserver` コマンドで起動している場合，SSH接続が切断してしまうと oTree も終了してしまう．これを防ぐために，`nohup` コマンドで oTree をバックグラウンドで実行する．
  ```
  nohup otree prodserver 8000 &
  ```
    - 終了するときには `ps` コマンドで oTree のプロセスID（PID）を探し， `kill` コマンドで終了させる．
- MTurkでの実験など，一定期間 oTree を起動したままほったらかしにする場合には，より高度なプロセス管理が必要．
    - SupervisorやCircusを使うのが良い，らしい．
