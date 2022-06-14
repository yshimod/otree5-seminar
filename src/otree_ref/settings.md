# `settings.py` の書き方


## `SESSION_CONFIGS`
- セッションの構成を辞書型で定義する．
    - `name`: セッションの名前
    - `display_name`: 管理者画面で表示するセッション名（指定しなければ `name` が表示される）
    - `app_sequence`: アプリの順序をlistで設定
    - `num_demo_participants`: デモページでの人数
    - 独自のセッション変数（定数）
        - たとえば `time_pressure=True` と設定した場合，アプリのスクリプトの中では `インスタンスオブジェクト.session.config["time_pressure"]` で変数にアクセスできる（インスタンスオブジェクト ∈ { `player`, `group`, `subsession` } ）．
        - [https://otree.readthedocs.io/en/latest/treatments.html](https://otree.readthedocs.io/en/latest/treatments.html)
- 例
    ```python
    SESSION_CONFIGS = [
        dict(
            name="guess_two_thirds",
            display_name="Guess 2/3 of the Average",
            app_sequence=[
                "guess_two_thirds",
                "payment_info"
            ],
            num_demo_participants=3,
            time_pressure=True
        ),
        dict(
            name="survey",
            app_sequence=[
                "survey",
                "payment_info"
            ],
            num_demo_participants=1
        )
    ]
    ```

## `SESSION_CONFIG_DEFAULTS`
- `SESSION_CONFIGS` で複数のセッションを設定している場合，すべての要素に共通して設定したい変数があれば `SESSION_CONFIG_DEFAULTS` で設定する．
- `SESSION_CONFIGS` に同じ変数名で定義されていれば， `SESSION_CONFIGS` の定義が優先される．
- `real_world_currency_per_point` （実験ポイントと通貨の間のレート） と `participation_fee` （固定報酬） は `SESSION_CONFIG_DEFAULTS` か `SESSION_CONFIGS` のどちらかで必ず設定しなければならない．
    - `participation_fee` で固定報酬を設定した場合，SessionPaymentsページで表示される参加者の合計報酬額（ `participant.payoff_plus_participation_fee` ）は，どんな参加者であれ `participation_fee.payoff` が加算される．たとえばオンライン実験で実験中に脱落した場合にも固定報酬が加算されてしまう．
- 例
    ```python
    SESSION_CONFIG_DEFAULTS = dict(
        real_world_currency_per_point=1.00,    # この変数は必ず設定せよ
        participation_fee=0.00,    # この変数は必ず設定せよ
        doc=""
    )
    ```

## `PARTICIPANT_FIELDS`
- 参加者一人一人について，アプリをまたいで1つの変数を使いたい場合，変数名の一覧を `PARTICIPANT_FIELDS` にlistで渡す．
- たとえば `effort_score` なる変数をアプリのスクリプトで使う場合， `participant.effort_score` または `participant.vars['effort_score']` で変数にアクセスする．変数の値の更新も可能．
- 例
    ```python
    PARTICIPANT_FIELDS = [
        'effort_score',
        'lottery_score'
    ]
    ```

## `SESSION_FIELDS`
- アプリをまたいでセッション参加者全員に共通な変数を使いたい場合，変数名の一覧を `SESSION_FIELDS` にlistで渡す．
- たとえば `effort_score_best` なる変数をアプリのスクリプトで使う場合， `session.effort_score_best` または `session.vars['effort_score_best']` で変数にアクセスする．変数の値の更新も可能．
- 例
    ```python
    SESSION_FIELDS = [
        'effort_score_best',
        'lottery_score_best'
    ]
    ```

## `LANGUAGE_CODE`
- 参加者に表示するデフォルトメッセージの言語を選択する．
    - たとえば次のページへ進めるボタンのテキストは，英語は「Next」日本語は「次へ」．
- 管理者画面は翻訳されない．
- 例
    ```python
    # ISO-639 code
    # for example: de, fr, ja, ko, zh-hans
    LANGUAGE_CODE = 'ja'    ## 日本語
    ```

## `REAL_WORLD_CURRENCY_CODE`
- Currency型で使う通貨を選択する．
- 日本円の場合，Currency型は整数になる．
- テンプレートで値を表示するとき，数字の後に「円」が表示される．
    - 「〇〇円」表記を「¥〇〇」表記に変更することはできない．
- 例
    ```python
    # e.g. EUR, GBP, CNY, JPY
    REAL_WORLD_CURRENCY_CODE = 'JPY'    ## 日本円
    ```

## `USE_POINTS`
- Currency型で通貨ではなくポイント（トークン，ECU）を使う場合， `True` を渡す．
- テンプレートで値を表示するとき，数字の後に「ポイント」が表示される．
- 例
    ```python
    USE_POINTS = True
    ```

## `ROOMS`
- Roomを使用する場合，dictで定義する．
- 参加者の一覧が手元にある場合:
    - 参加者ラベルの一覧はテキストファイル（たとえば `econ101.txt` ）で作る．ラベル（ `participant_label` ）を改行区切りで記述する．
    - 参加者ラベルの一覧を記述したテキストファイルのパス（ `_rooms/econ101.txt` ）を `participant_label_file` で指定する．
    ```python
    ROOMS = [
        dict(
            name='econ101',
            display_name='Econ 101 class',
            participant_label_file='_rooms/econ101.txt',
        )
    ]
    ```
- 参加者の一覧が手元に無い場合:
    - たとえば，参加者にラベルを入力させる，URLクエリパラメータでラベルを設定する，などの場合．
    - `participant_label_file` を設定しない．
    ```python
    ROOMS = [
        dict(
            name='live_demo',
            display_name='Room for live demo (no participant labels)'
        )
    ]
    ```

## `ADMIN_USERNAME`
- 管理者のユーザー名．
- 例
    ```python
    ADMIN_USERNAME = 'admin'
    ```

## `ADMIN_PASSWORD`
- 管理者のパスワード． `settings.py` 内で直接設定できなくもないが，環境変数（ `OTREE_ADMIN_PASSWORD` ）を設定する方が良い．
- デフォルト
    ```python
    # for security, best to set admin password in an environment variable
    ADMIN_PASSWORD = environ.get('OTREE_ADMIN_PASSWORD')
    ```

## `DEMO_PAGE_INTRO_HTML`
- デモページで実験者向けに表示する説明文．
- 例
    ```python
    DEMO_PAGE_INTRO_HTML = """Here are some oTree games."""
    ```

## `SECRET_KEY`
- URLに付与されるランダム文字列の生成に使われる．
- `otree startproject` でプロジェクトを作成したときに文字列が自動的に生成される．
- `SECRET_KEY` があれば不正にアクセスできてしまう？
- GitHubでプロジェクトを管理する場合， `settings.py` の中には書かず環境変数で設定した方が良い？（Djangoの `SECRET_KEY` はGitHubにプッシュすると警告される．）
- 例
    ```python
    SECRET_KEY = '1234567890'
    ```
- 環境変数で設定する例
    ```python
    SECRET_KEY = environ.get('OTREE_SECRET_KEY')
    ```
    - 環境変数は `export OTREE_SECRET_KEY=1234567890` などのように設定．

## `INSTALLED_APPS`
- デフォルトで以下が設定されているが， oTree 5 では不要？
    ```python
    INSTALLED_APPS = ['otree']
    ```

## `DEBUG`
- oTree本体では `DEBUG = os.environ.get('OTREE_PRODUCTION') in [None, '', '0']`
- つまり，環境変数で `export OTREE_PRODUCTION=1` とするか `settings.py` で以下の設定をしない限りデバッグモードで起動する．
    ```python
    DEBUG = False
    ```

## `AUTH_LEVEL`
- oTree本体では `AUTH_LEVEL = os.environ.get('OTREE_AUTH_LEVEL')`
- 環境変数で `export OTREE_AUTH_LEVEL=STUDY` とするか `settings.py` で以下の設定すると本番モード（管理者画面全体がパスワード保護）で起動する．
    ```python
    OTREE_AUTH_LEVEL = "STUDY"
    ```
- `STUDY` を `DEMO` に変えると，管理者画面のデモページ以外がパスワード保護で起動する．
