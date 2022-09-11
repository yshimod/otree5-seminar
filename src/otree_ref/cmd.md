# シェルで使用する oTree サブコマンド


## `browser_bots`



## `create_session`

- コマンドでセッションを作成する．

- セッション名，人数（，ルーム）の指定しかできないため使いどころは不明．より詳細な設定でセッション作成を自動化したい場合はREST APIを使った方が良い．
[https://otree.readthedocs.io/en/latest/misc/rest_api.html#create-sessions-endpoint](https://otree.readthedocs.io/en/latest/misc/rest_api.html#create-sessions-endpoint)

- 使い方
    - `otree create_session session_config_name num_participants`
    - `otree create_session --room ROOM_NAME session_config_name num_participants`

- `session_config_name`: `settings.py` の `SESSION_CONFIGS` で設定しているセッション名を1つ選ぶ．

- `num_participants`: セッションの人数を設定．

- `ROOM_NAME`: `settings.py` の `ROOMS` で設定しているルーム名を1つ選ぶ．



## `devserver`

- 開発中の動作確認のためにサーバーを起動するコマンド．

- 起動後（ポートを特定しなければ） `http://localhost:8000` で管理者画面にアクセスできる．

- `prodserver` コマンドとは異なり，稼働中に `settings.py` や `__init__.py` などを編集すると自動的にサーバーを再起動して編集分を反映してくれる．

- 使い方
    - `otree devserver`
    - `otree devserver 8000`
    - `otree devserver 0.0.0.0:8000`



## `prodserver`

- 実験本番のためにサーバーを起動するコマンド．

- 起動後（ポートを特定しなければ） `http://localhost:8000` で管理者画面にアクセスできる．

- `devserver` コマンドとは異なり，稼働中にWebサーバーのログがシェル標準出力される．

- `devserver` コマンドとは異なり，クライアントがブラウザーを閉じているときにタイムアウトが発生すると，クライアントの代わりに，サーバーが自分宛てにフォームを送信する．これは時間制限 + 6秒のタイミングで行われる．

- 使い方
    - `otree prodserver`
    - `otree prodserver 8000`
    - `otree prodserver 0.0.0.0:8000`



## `prodserver1of2`

- `prodserver` コマンドの正体．

- Heroku用の `Procfile` で Web dyno のコマンドとして設定されている．



## `prodserver2of2`

- Heroku用の `Procfile` で Worker dyno のコマンドとして設定されているが，実のところ何もしない．互換性対応で残っている．



## `remove_self`

- [https://otree.readthedocs.io/en/latest/misc/noself.html](https://otree.readthedocs.io/en/latest/misc/noself.html)



## `resetdb`

- データベースをリセットする．



## `startapp`

- 既存のプロジェクトに新規のアプリを作成する．

- このコマンドで生成されるファイル:
  ```
  .
  ├── MyPage.html
  ├── Results.html
  ├── __init__.py
  └── __pycache__
      └── __init__.cpython-39.pyc
  ```

- 使い方
    1. `cd プロジェクトのディレクトリ`
    1. `otree startapp アプリ名`



## `startproject`

- プロジェクトを新規に作成する．

- このコマンドで生成されるファイル:
  ```
  .
  ├── Procfile
  ├── __pycache__
  │   └── settings.cpython-39.pyc
  ├── _static
  │   └── global
  │       └── empty.css
  ├── _templates
  │   └── global
  │       └── Page.html
  ├── requirements.txt
  └── settings.py
  ```

- 使い方
    - `otree startproject プロジェクト名`

- サンプルゲームを追加する場合は `Include sample games? (y or n)` と聞かれたときに `y` と答える．

- サンプルゲームを追加しない場合，`startapp` コマンドで1つずつ空のアプリを追加していく．

- サンプルゲームは自動的に https://github.com/oTree-org/oTree/archive/lite.zip からダウンロードしてくる．したがってインターネットに接続していないとサンプルゲームの追加に失敗する．

- GitHubで管理するときにはアプリ単位ではなくプロジェクト単位の方が良い．プロジェクトで1つのリポジトリであれば，そのままHerokuにデプロイできる．



## `test`



## `unzip`

- `.otreezip` ファイルを解凍する．



## `update_my_code`

- oTree バージョン3スタイルのコードをバージョン5のスタイルに変更するコマンド．ただし，コマンドを実行して成功したように見えても，実際のところ不備が残っているかもしれないので，確認を要する．

- あえて  `update_my_code` でコードを書き換えなくてもそのまま動くケースは多い．

- `update_my_code` の後に， `remove_self` や `upcase_constants` の実行をおすすめされる．



## `zip`

- oTree プロジェクトのディレクトリで `otree zip` を実行すると，ディレクトリ内の `.git`， `db.sqlite3`， `.pyo`， `.pyc`， `.pyd`， `.idea`， `.DS_Store`， `.otreezip`， `venv`， `_static_root`， `staticfiles`， `__pycache__`， `.env` を除外して zipファイルに圧縮する．

- `requirements.txt` が無い場合，エラーとなる．

- `requirements.txt` に記述されているパッケージのバージョンが古い場合，最新バージョンを使うように書き換えるか聞いてくる．

- `runtime.txt` が無い場合，勝手に生成して追加する．存在する場合も勝手に書き換える．

- 圧縮されたファイルは `.otreezip` なる拡張子で保存される．解凍するには `otree unzip ファイルパス`．



## `zipserver`

- `.otreezip` ファイルのまま，サーバーを起動できる．
