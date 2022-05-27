# シェルで使用するoTreeサブコマンド

## `browser_bots`

## `create_session`
- コマンドでセッションを作成する．
- セッション名，人数（，ルーム）の指定しかできないため使いどころは不明．より詳細な設定でセッション作成を自動化したい場合はREST APIを使った方が良い．
[https://otree.readthedocs.io/en/latest/misc/rest_api.html#create-sessions-endpoint](https://otree.readthedocs.io/en/latest/misc/rest_api.html#create-sessions-endpoint)
- 使い方
    - ` otree create_session session_config_name num_participants`
    - ` otree create_session --room ROOM_NAME session_config_name num_participants`
- `session_config_name`: `settings.py` の `SESSION_CONFIGS` で設定しているセッション名を1つ選ぶ．
- `num_participants`: セッションの人数を設定．
- `ROOM_NAME`: `settings.py` の `ROOMS` で設定しているルーム名を1つ選ぶ．

## `devserver`
- 開発中の動作環境のためにサーバーを起動するコマンド．
- `prodserver` コマンドとは異なり，稼働中に `settings.py` や `__init__.py` などを編集すると自動的にサーバーを再起動して編集分を反映してくれる．
- 使い方
    - `otree devserver`
    - `otree devserver 8000`
    - `otree devserver 0.0.0.0:8000`

## `prodserver`
- 実験本番のためにサーバーを起動するコマンド．
- `devserver` コマンドとは異なり，稼働中にWebサーバーのログがシェル標準出力される．
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
- GitHubで管理するときにはアプリ単位ではなくプロジェクト単位の方が良い．プロジェクトで1つのリポジトリであれば，そのままHerokuにデプロイできる．

## `test`

## `unzip`

## `update_my_code`

## `zip`

## `zipserver`
