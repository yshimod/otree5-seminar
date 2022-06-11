{% raw %}

# `__init__.py` の書き方


## ファイルの冒頭
- シバン（shebang... `#!/usr/bin/env python3` みたいなやつ）は不要．
- 文字コード宣言（ `# -*- coding: utf-8 -*-` みたいなやつ）は不要（むしろ非推奨）．
- `otree.api` のインポート
    ```python
    from otree.api import *
    ```
    - oTree3ではインポートするものを細かく指定していた． [https://otree.readthedocs.io/en/latest/misc/noself.html#about-the-new-format](https://otree.readthedocs.io/en/latest/misc/noself.html#about-the-new-format)
- 他にも使いたいモジュール（`time`，`random`，`json`，`numpy`など）があれば，冒頭でインポートしておく．


## 定数: `C` クラス
[https://otree.readthedocs.io/en/latest/models.html#constants](https://otree.readthedocs.io/en/latest/models.html#constants)

- アプリ内で参照する変数（定数）を定義する．
- クラス `C` で設定するものはCSVデータ出力には含まれない．CSVデータに記録しなくても平気な定数を定義する．
    - セッションごと（トリートメントごと）変化しうる変数はクラス `C` ではなく `settings.py` の `SESSION_CONFIGS` の中で定義するべき． `SESSION_CONFIGS` で定義した変数（ `session.変数` ）はCSVデータ出力に含まれる．
- 公式ドキュメントには，辞書型の変数はメソッドで定義せよ，とあるが，辞書型の変数も普通に定義できそう．
- マナーとして変数名は大文字にしておくと良い？
- oTree3では `Constants` なるクラスで設定していた．

### `NAME_IN_URL`
- URLにアプリ名として表示するものを設定．
- 必ず設定しなければならないが，アプリ名（ `__name__` で取得できる）と異なっていても良い．

### `PLAYERS_PER_GROUP`
- ゲーム実験の場合，各グループの人数を設定する．
- 必ず設定する．
- プレーヤーの相互作用が無い場合（アンケートなど）は `1` ではなく `None` とする．
- セッションを作成するとき，人数は `PLAYERS_PER_GROUP` の整数倍でなければならない．
- グループ内でプレイヤーに番号（ `player.id_in_group` ）が振られる．この番号はグループにおける役割のインデックスとして使うと良い．

### `NUM_ROUNDS`
- アプリを繰り返す場合，繰り返す（最大）回数を設定する．
- 1以上の整数を必ず設定する．

### `*_ROLE`
- 変数名の末尾に `_ROLE` ，または先頭に `ROLE_` をつけたものは， `player.id_in_group` のラベルとして使うことができる．
- たとえば `PLAYERS_PER_GROUP = 2` のとき，
    ```python
    BUYER_ROLE = "買い手"
    SELLER_ROLE = "売り手"
    ```
    とすると， `player.id_in_group == 1` のプレイヤーの `player.role` は `買い手` ， `player.id_in_group == 2` のプレイヤーの `player.role` は `売り手` となる．
- `player.id_in_group` のラベルとして使いたくない場合， `C` のクラス変数の名前に `_ROLE` や `ROLE_` をつけてはいけない．



## データモデル
[https://otree.readthedocs.io/en/latest/models.html](https://otree.readthedocs.io/en/latest/models.html)


### `Subsession` クラス
[https://otree.readthedocs.io/en/latest/models.html#subsession](https://otree.readthedocs.io/en/latest/models.html#subsession)

#### メソッド・変数
- `round_number`
    - （当該 player ないし group が属している） subsession のラウンド数（自然数）．
    - subsession は player， group の上位概念なので，たとえば，ある player がラウンド1にいて，もう一人の player がラウンド2にいるとき，果たして `subsession.round_number` は1と2のどちらなのか，と疑問に思うかもしれない．答えは，前者の player について， `player.subsession.round_number` は1，後者の player について， `player.subsession.round_number` は2，となる．ラウンドごとに subsession が定義されるため，2人の player が属する subsession が異なっていることに注意．
- `get_groups()`
    - 返り値: subsession に属する group のリスト．
- `get_players()`
    - 返り値: subsession に属する player のリスト．
- `in_all_rounds()`
    - 返り値: すべてのラウンドの subsession のリスト．
- `in_previous_rounds()`
    - 返り値: （当該ラウンドを含まず）以前のすべてのラウンドの subsession のリスト．
- `in_rounds(first, last)`
    - 返り値: `first` ラウンド目から `last` ラウンド目までの（両端を含む）subsession のリスト．
- `in_round(round_number)`
    - 返り値: `round_number` ラウンド目の subsession．
- `get_group_matrix()`
    - 返り値: `player.id_in_subsession` （自然数）の2次元配列．
    - `get_group_matrix()[i][j]` は， `group.id_in_subsession` が `i+1` で `player.id_in_group` が `j+1` である player の `player.id_in_subsession`．
- `field_maybe_none()`
    - 引数: フォームの変数名（文字列）．
    - 返り値: 引数の変数の値．変数の値が `None` であれば `None` がエラーを出すことなく返ってくる．
    - [https://otree.readthedocs.io/en/latest/misc/tips_and_tricks.html#field-maybe-none](https://otree.readthedocs.io/en/latest/misc/tips_and_tricks.html#field-maybe-none)

#### 組み込み関数 `creating_session()` の中で使うメソッド
- `group_randomly()`
    - player をランダムに group 分けする．
    - [https://otree.readthedocs.io/en/latest/multiplayer/groups.html#group-randomly](https://otree.readthedocs.io/en/latest/multiplayer/groups.html#group-randomly)
    - 引数で `fixed_id_in_group = True` を取ると，役割（ `id_in_group` ） を維持したまま group 分けする．
    - 返り値なし．
- `group_like_round(round_number)`
    - 以前のラウンド（ `round_number` ラウンド目）の group 分けを引き継ぐ．
    - [https://otree.readthedocs.io/en/latest/multiplayer/groups.html#group-like-round](https://otree.readthedocs.io/en/latest/multiplayer/groups.html#group-like-round)
- `set_group_matrix(new_structure)`
    - group 分けを自分で決めて設定する．
    - [https://otree.readthedocs.io/en/latest/multiplayer/groups.html#group-set-players](https://otree.readthedocs.io/en/latest/multiplayer/groups.html#group-set-players)
    - 引数は `player.id_in_subsession` （自然数）の2次元リスト．
    - たとえば， `PLAYERS_PER_GROUP = 2` で，全ての player 数（ `subsession.session.num_participants` ）が6のとき，  
        ```python
        new_structure = [
            [1, 3],
            [5, 2],
            [4, 6]
        ]
        subsession.set_group_matrix(new_structure)
        ```
        とすれば，以下のような group 分けになる．

        <table>
            <thead>
            <tr>
            <th><code>group.id_in_subsession</code></th>
            <th><code>player.id_in_group == 1</code></th>
            <th><code>player.id_in_group == 2</code></th>
            </tr>
            </thead>
            <tbody>
            <tr>
            <td>1</td>
            <td>1</td>
            <td>3</td>
            </tr>
            <tr>
            <td>2</td>
            <td>5</td>
            <td>2</td>
            </tr>
            <tr>
            <td>3</td>
            <td>4</td>
            <td>6</td>
            </tr>
            </tbody>
        </table>


#### アクセスできる上層のオブジェクト
- session



### クラス `Group`
[https://otree.readthedocs.io/en/latest/models.html#group](https://otree.readthedocs.io/en/latest/models.html#group)

#### メソッド・変数
- `id_in_subsession`
    - group の番号（自然数）．
- `round_number`
    - group のラウンド数（自然数）．
- `get_players()`
    - 返り値: group に属する player のリスト．
- `in_all_rounds()`
    - 返り値: 同一 `group.id_in_subsession` である，すべてのラウンドの group のリスト．
- `in_previous_rounds()`
    - 返り値: 同一 `group.id_in_subsession` である，（当該ラウンドを含まず）以前のすべてのラウンドの group のリスト．
- `in_rounds(first, last)`
    - 返り値: 同一 `group.id_in_subsession` である， `first` ラウンド目から `last` ラウンド目までの（両端を含む）group のリスト．
- `in_round(round_number)`
    - 返り値: 同一 `group.id_in_subsession` である， `round_number` ラウンド目の group．
- `get_player_by_role(role)`
    - 返り値: group に属する player の中で， `player.role` が 引数 `role` （文字列）と等しい player．
    - `C` クラスで `_ROLE` が定義されていないと使えない．
    - [https://otree.readthedocs.io/en/latest/multiplayer/groups.html#roles](https://otree.readthedocs.io/en/latest/multiplayer/groups.html#roles)
- `get_player_by_id(id_in_group)`
    - 返り値: group に属する player の中で， `player.id_in_group` が 引数 `id_in_group` （自然数）と等しい player．
- `set_players(new_structure)`
    - group 内の役割（ `id_in_group` ）を自分で決めて設定する．
    - 引数は player オブジェクトのリスト．自然数のリスト（たとえば `id_in_group` のリスト）ではないことに注意．
    - 組み込み関数 `creating_session()` の中で使うのが良い？
- `field_maybe_none()`
    - 引数: フォームの変数名（文字列）．
    - 返り値: 引数の変数の値．変数の値が `None` であれば `None` がエラーを出すことなく返ってくる．
    - [https://otree.readthedocs.io/en/latest/misc/tips_and_tricks.html#field-maybe-none](https://otree.readthedocs.io/en/latest/misc/tips_and_tricks.html#field-maybe-none)

#### アクセスできる上層のオブジェクト
- subsession
- session



### クラス `Player`
[https://otree.readthedocs.io/en/latest/models.html#player](https://otree.readthedocs.io/en/latest/models.html#player)


- `id_in_subsession`
    - subsession でユニークな player の番号（自然数）．
    - 管理者画面で番号は 「P1」「P2」，... と表示される．
    - `player.participant.id_in_session` と等しい．
- `id_in_group`
    - group でユニークな player の役割番号．
- `role`
    - `C` クラスで `_ROLE` が定義されている場合， `id_in_group` に対応するラベルの文字列．
- `payoff`
    - 報酬額を記録する組み込みのフィールド．
    - [https://otree.readthedocs.io/en/latest/currency.html#payoffs](https://otree.readthedocs.io/en/latest/currency.html#payoffs)
    - 全 subsession の `player.payoff` の合計が自動的に計算され，管理者画面の Payments で確認できる．
    - 自動的にoTreeの通貨型に変換されるので注意．
- `round_number`
    - player のラウンド数（自然数）．
- `in_all_rounds()`
    - 返り値: 同一 participant である，すべてのラウンドの player のリスト．
- `in_previous_rounds()`
    - 返り値: 同一 participant である，（当該ラウンドを含まず）以前のすべてのラウンドの player のリスト．
- `in_rounds(first, last)`
    - 返り値: 同一 participant である， `first` ラウンド目から `last` ラウンド目までの（両端を含む）player のリスト．
- `in_round(round_number)`
    - 返り値: 同一 participant である， `round_number` ラウンド目の player．
- `get_others_in_subsession()`
    - 返り値: player が属する subsession に属する，当該 player 以外の player のリスト．
- `get_others_in_group()`
    - 返り値: player が属する group に属する，当該 player 以外の player のリスト．
- `field_maybe_none()`
    - 引数: フォームの変数名（文字列）．
    - 返り値: 引数の変数の値．変数の値が `None` であれば `None` がエラーを出すことなく返ってくる．
    - [https://otree.readthedocs.io/en/latest/misc/tips_and_tricks.html#field-maybe-none](https://otree.readthedocs.io/en/latest/misc/tips_and_tricks.html#field-maybe-none)

#### アクセスできる上層のオブジェクト
- participant
- group
- subsession
- session


### フィールドの型
[https://otree.readthedocs.io/en/latest/models.html#fields](https://otree.readthedocs.io/en/latest/models.html#fields)

- `models.BooleanField`
    - `choices`
    - `widget`
    - `initial`
    - `label`
    - `doc`
    - `blank`
- `models.CurrencyField`
    - `choices`
    - `widget`
    - `initial`
    - `label`
    - `doc`
    - `min`
    - `max`
    - `blank`
- `models.IntegerField`
    - `choices`
    - `widget`
    - `initial`
    - `label`
    - `doc`
    - `min`
    - `max`
    - `blank`
- `models.FloatField`
    - `choices`
    - `widget`
    - `initial`
    - `label`
    - `doc`
    - `min`
    - `max`
    - `blank`
- `models.StringField`
    - `choices`
    - `widget`
    - `initial`
    - `label`
    - `doc`
    - `max_length=10000`
    - `blank`
- `models.LongStringField`
    - `initial`
    - `label`
    - `doc`
    - `max_length=None`
    - `blank`


### widget として使えるもの
[https://otree.readthedocs.io/en/latest/forms.html#widgets](https://otree.readthedocs.io/en/latest/forms.html#widgets)

- `CheckboxInput`
- `RadioSelect`
- `RadioSelectHorizontal`


### 入力フォームとの対応
[https://otree.readthedocs.io/en/latest/forms.html#raw-html-widgets](https://otree.readthedocs.io/en/latest/forms.html#raw-html-widgets)

1. データを参加者に入力させたいページのクラスにおいて `form_model` と `form_fields` を設定する．
1. フィールド名（記憶するデータの変数名）を，HTMLテンプレートの `<input>` タグの `name` 属性に設定する．
    - ↑ テンプレートタグを使えば自動でやってくれる．
1. HTMLタグを直書きして入力フォームを作る場合も，テンプレートタグ `{{ formfield_errors '変数名' }}` を書いておくと，oTreeによる検証のエラーメッセージを表示することができる．



## ページ
[https://otree.readthedocs.io/en/latest/pages.html](https://otree.readthedocs.io/en/latest/pages.html)


- クラス `Page` を継承する．
- クラス名がURLに表示される．

### ページクラスの変数
- `template_name`
    - テンプレートファイルのパスを渡せば，ページクラスの名称とファイル名が一致していなくても良い．
    - 一つのテンプレートファイルを複数のページクラスで使い回すことも可能．
    - パスはアプリディレクトリから指定する．
        - `template_name = 'app_name/MyPage.html'`
        - 同一アプリ内であれば， `template_name = __name__ + '/MyPage.html'` と書いても良い．
- `form_model`
    - 当該ページで使う入力フォームのモデルを `player`， `group`， `subsession` から選んで文字列を渡す．
    - どれか一つしか選べない．一つのページで player モデルの変数と group モデルの変数の両方の入力フォームを置くことはできない．たとえばとりあえず全部 player モデルでデータを記録しておき， oTree 内部で group モデルの変数に転記する，などで対処する．
    - 動的に変更することはできない．
- `form_fields`
    - 当該ページで入力フォームの変数名の文字列をリストで渡す．
    - `form_model` で指定したモデルで定義していない変数を指定してはいけない．
    - `form_fields` で複数要素を指定していて，かつテンプレートタグ `{{ formfields }}` を使う場合，リストの順番通り入力フォームが作られる．
    - 動的に変更する場合には組み込みメソッド `get_form_fields()` を使う．
- `timeout_seconds`
    - 整数で秒数を渡すと，当該ページでの時間制限を設定できる．
    - 動的に変更する場合には組み込みメソッド `get_timeout_seconds()` を使う．
- `timer_text`
    - 時間制限を設定したときに表示されるメッセージ（文字列）を指定する．
    - デフォルトでは「このページでの残り時間」と表示される．


### ページクラスの組み込みメソッド

関数を定義する前にデコレータ `@staticmethod` を記述しておく．
ただし，デコレータをつけずにおいてインスタンスメソッドとして（第1引数を `self` にして）使うことはできなさそう．


- `live_method()`
    - 引数: `(player: Player, data)`
    - 返り値: 辞書型．キーは `id_in_group` とする．
        - group の全員に送信する場合のキーは `0`．
        - 他の group や subsession 全体へは送信できない．
    - [https://otree.readthedocs.io/en/latest/live.html](https://otree.readthedocs.io/en/latest/live.html)
- `get_form_fields()`
    - 引数: `(player: Player)`
    - 返り値: 変数名の文字列のリスト． `form_fields` と同様．
- `vars_for_template()`
    - 引数: `(player: Player)`
    - 返り値: 辞書型．
    - [https://otree.readthedocs.io/en/latest/pages.html#vars-for-template](https://otree.readthedocs.io/en/latest/pages.html#vars-for-template)
    - ページを読み込む度に実行される．
    - `js_vars()` よりも先に実行される．
- `js_vars()`
    - 引数: `(player: Player)`
    - 返り値: 辞書型．
    - [https://otree.readthedocs.io/en/latest/templates.html#passing-data-from-python-to-javascript-js-vars](https://otree.readthedocs.io/en/latest/templates.html#passing-data-from-python-to-javascript-js-vars)
    - ページを読み込む度に実行される．
    - `vars_for_template()` の後に実行される．
- `before_next_page()`
    - 引数: `(player: Player, timeout_happened)`
        - `timeout_happened` はブール値．時間制限でページが進んだ場合にフラグが立つ．
    - 返り値: 何も返してはいけない．
    - ページが進められたとき（フォームが送信されたとき），次のページを表示する前に実行される．
    - 当該ページが `is_displayed()` でスキップされていれば，実行されない．
- `is_displayed()`
    - 引数: `(player: Player)`
    - 返り値: ブール値．デフォルトでは `True` を返している．
    - ページを表示する前に実行される．
    - 返り値が `False` の場合，（ `page_sequence` で指定した順番の）次のページへスキップされる．
- `error_message()`
    - 引数: `(player: Player, values)`
        - `values` はフォームの変数名をキーとする辞書で，入力された値が取り出せる．
    - 返り値: フォームの変数名をキーとする辞書型で，値は各フォームに対応するエラーメッセージの文字列．エラーが無い場合は何も返さないか，空の辞書を返す．
    - ページが進められたとき（フォームが送信されたとき），次のページを表示する前に実行され，エラーがあれば同一ページにエラーメッセージを追加したものを表示する．
    - フォームごと個別に検証用の関数を定義することもできる．ページクラスの外側で `*_error_message()` を定義する．
- `get_timeout_seconds()`
    - 引数: `(player: Player)`
    - 返り値: 整数で秒数を渡すと，当該ページでの時間制限を設定できる． `timeout_seconds` と同様．
- `app_after_this_page()`
    - 引数: `(player: Player, upcoming_apps)`
        - `upcoming_apps` は `settings.py` で設定した `SESSION_CONFIGS.app_sequence` で，当該アプリより後のアプリ名（文字列）が並んだリスト．
    - 返り値: スキップ先のアプリ名（ `upcoming_apps` の要素である文字列）．
    - 複数のラウンドが設定してあるときも，アプリごとスキップする．同一アプリで次のラウンドの最初のページへスキップするのではない．


## 待機ページ
[https://otree.readthedocs.io/en/latest/multiplayer/waitpages.html](https://otree.readthedocs.io/en/latest/multiplayer/waitpages.html)


- クラス `WaitPage` を継承する．
- クラス名がURLに表示される．

### 待機ページの変数
- `template_name`
    - 待機ページもカスタマイズしようと思えばできる．そのときはテンプレートファイルのパスを渡す．
    - 使用例: [https://www.otreehub.com/projects/otree-snippets/](https://www.otreehub.com/projects/otree-snippets/) の "wait_page_timeout"
- `group_by_arrival_time`
    - `True` を渡せば，次のページ以降の group 分けが，当該待機ページに到達した順の group 分けになる．
    - 当該待機ページが `page_sequence` の先頭にある場合のみ `group_by_arrival_time = True` を設定できる．
    - `group_by_arrival_time = True` を設定している場合，全 player が当該待機ページを通るようにする． `is_displayed()` で特定の役割の player だけスキップする，というようなことをしてはいけない． `is_displayed()` で特定のラウンド以外はスキップする，という使い方はOK．
    - より細かい設定をしたい場合は， `group_by_arrival_time = True` を設定した上で，クラスの外側で `group_by_arrival_time_method()` を定義する．
    - [https://otree.readthedocs.io/en/latest/multiplayer/waitpages.html#group-by-arrival-time](https://otree.readthedocs.io/en/latest/multiplayer/waitpages.html#group-by-arrival-time)
    - 待機中，参加者のウィンドウがアクティブでない場合（たとえば他のタブを開いて遊んでいる場合），ドロップアウトとみなし group に参加させない． [https://groups.google.com/g/otree/c/XsFMNoZR7PY](https://groups.google.com/g/otree/c/XsFMNoZR7PY)
- `wait_for_all_groups`
    - `True` を渡せば，待機ページにおいて group のメンバーではなく， subsession の全メンバーを待機する．
    - `wait_for_all_groups = True` としたときは，組み込みメソッド `after_all_players_arrive()` の引数が subsession オブジェクトとなることに注意．
- `title_text`
    - 文字列を渡せば，待機ページで表示される「しばらくお待ちください」の部分を変更することができる．
- `body_text`
    - 文字列を渡せば，待機ページで表示される「他の参加者をお待ちください」の部分を変更することができる．


### 待機ページの組み込みメソッド

関数を定義する前にデコレータ `@staticmethod` を記述しておく．
ただし，デコレータをつけずにおいてインスタンスメソッドとして（第1引数を `self` にして）使うことはできなさそう．


- `vars_for_template()`
    - 引数: `(player: Player)`
    - 返り値: 辞書型．
    - `template_name` にパスを渡して独自のテンプレートファイルを使うときのみ設定する．
- `js_vars()`
    - 引数: `(player: Player)`
    - 返り値: 辞書型．
    - `template_name` にパスを渡して独自のテンプレートファイルを使うときのみ設定する．
- `is_displayed()`
    - 引数: `(player: Player)`
    - 返り値: ブール値．デフォルトでは `True` を返している．
    - 逐次手番ゲームにおいて，プレイヤーの役割別に表示させるページを `is_displayed()` 制御しているとき，待機ページで `is_displayed()` を使う必要はない．
- `app_after_this_page()`
    - 引数: `(player: Player, upcoming_apps)`
    - 返り値: スキップ先のアプリ名（ `upcoming_apps` の要素である文字列）．
- `after_all_players_arrive()`
    - 引数: `(group: Group)`
    - `wait_for_all_groups = True` とした場合の引数は `(subsession: Subsession)`．
    - 返り値: 何も返してはいけない．
    - モジュールレベルで関数（たとえば group オブジェクトを引数に取る `set_payoffs()` ）を独自に定義しておき，以下のように呼び出す設定をしても良い．引数に注意．
        ```python
        after_all_players_arrive = 'set_payoffs'
        ```
        ```python
        after_all_players_arrive = set_payoffs
        ```
        ```python
        def after_all_players_arrive(group: Group):
            set_payoffs(group)
        ```
    - [https://otree.readthedocs.io/en/latest/multiplayer/waitpages.html#after-all-players-arrive](https://otree.readthedocs.io/en/latest/multiplayer/waitpages.html#after-all-players-arrive)



## モジュールレベルの組み込み関数

データモデルクラスやページクラスの外側で定義する．引数に注意．


### `creating_session()`
- `Subsession` クラスの下側で定義する．
- 引数: `(subsession: Subsession)`
- 返り値: 何も返してはいけない．
- セッションの作成時に実行される． subsession ごと実行される．
- [https://otree.readthedocs.io/en/latest/treatments.html](https://otree.readthedocs.io/en/latest/treatments.html)

### `custom_export()`
- 引数: `players` （ `player` のリスト）
- 返り値: `yeild` で出力したいデータをリストで返す．
- "CSV Data Export" で出力するときに実行される．
- [https://otree.readthedocs.io/en/latest/admin.html#custom-data-exports](https://otree.readthedocs.io/en/latest/admin.html#custom-data-exports)

### `group_by_arrival_time_method()`
- `Subsession` クラスの下側で定義する．
- 引数: `(subsession: Subsession, waiting_players)`
    - `waiting_players` 到達して待機中の player オブジェクトが入ったリスト．
- 返り値: player オブジェクトが入ったリスト．リストの要素のメンバーで group となる．
- 待機ページにプレイヤーが到達する度に実行される．
- [https://otree.readthedocs.io/en/latest/multiplayer/waitpages.html#group-by-arrival-time-method](https://otree.readthedocs.io/en/latest/multiplayer/waitpages.html#group-by-arrival-time-method)

### `vars_for_admin_report()`
- 引数: `(subsession: Subsession)`
- 返り値: 辞書型．
- 定義する場合は，当該アプリのディレクトリに `admin_report.html` なるファイル名でテンプレートファイルを作成しておく．
- [https://otree.readthedocs.io/en/latest/admin.html#customizing-the-admin-interface-admin-reports](https://otree.readthedocs.io/en/latest/admin.html#customizing-the-admin-interface-admin-reports)

### `*_min()`
- `*` の部分にフィールドの変数名を入れる．
- 引数: `(player: Player)`
- 返り値: フィールドの最小値．

### `*_max`
- `*` の部分にフィールドの変数名を入れる．
- 引数: `(player: Player)`
- 返り値: フィールドの最大値．

### `*_choices`
- `*` の部分にフィールドの変数名を入れる．
- 引数: `(player: Player)`
- 返り値: フィールドの選択肢のリスト．

### `*_error_message`
- `*` の部分にフィールドの変数名を入れる．
- 引数: `(player: Player, value)`
    - `value` は入力されたフィールドの値．
- 返り値: エラーメッセージの文字列．


## 自作の関数

- モジュールレベルやクラスの中で自分で関数を定義して使える．
- 自作の関数なので当然自分で引数を設定して良いが，呼び出す場所次第では引数の指定がある．たとえば， `after_all_players_arrive` に自作の関数を渡すときに引数は `group` （または `subsession` ）となる．
- 当然のこととして，自作の関数は陽に呼び出さないと実行されない．


## `page_sequence`

- ページの順番をリストで渡す．ページ名の文字列ではなく，ページクラスのクラスオジェクトが要素．


{% endraw %}
