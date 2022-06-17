{% raw %}

# `__init__.py` の書き方


## ファイルの冒頭
- シバン（shebang... `#!/usr/bin/env python3` みたいなやつ）は不要．
- 文字コード宣言（ `# -*- coding: utf-8 -*-` みたいなやつ）は不要（むしろ非推奨）．
- `otree.api` のインポート
    ```python
    from otree.api import *
    ```
    - oTree 3 ではインポートするものを細かく指定していた． [https://otree.readthedocs.io/en/latest/misc/noself.html#about-the-new-format](https://otree.readthedocs.io/en/latest/misc/noself.html#about-the-new-format)
- 他にも使いたいモジュール（`time`，`random`，`json`，`numpy`など）があれば，冒頭でインポートしておく．


## 定数 `C` クラス
[https://otree.readthedocs.io/en/latest/models.html#constants](https://otree.readthedocs.io/en/latest/models.html#constants)

- アプリ内で参照する変数（定数）を定義する．
- クラス `C` で設定するものはCSVデータ出力には含まれない．CSVデータに記録しなくても平気な定数を定義する．
    - セッションごと（トリートメントごと）変化しうる変数はクラス `C` ではなく `settings.py` の `SESSION_CONFIGS` の中で定義するべき． `SESSION_CONFIGS` で定義した変数（ `session.変数` ）はCSVデータ出力に含まれる．
- 公式ドキュメントには，辞書型の変数はメソッドで定義せよ，とあるが，辞書型の変数も普通に定義できそう．
- マナーとして変数名は大文字にしておくと良い？
- oTree 3 では `Constants` なるクラスで設定していた．

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

#### `subsession.*` （メソッド・変数）
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
    - 引数で `objects=True` を渡すと，返り値のリストの要素が `id_in_subsession` （自然数）ではなく `player` オブジェクトそのものになる．
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
    - `creating_session()` において `set_group_matrix()` を使って group 分けを設定するとき， `PLAYERS_PER_GROUP` は設定せずに `None` のままでも良い． `set_group_matrix()` の設定の方が優先される．
    - 返り値はリストのリストであればよく，行列である必要はない．つまり， group のサイズが統一されていなくてもよい．たとえば以下のような設定も可能．
        ```python
        subsession.set_group_matrix(
            [
                [1],
                [2, 3],
                [4, 5, 6],
                [7, 8, 9, 10]
            ]
        )
        ```

#### subsession からアクセスできる上層のオブジェクト
- session



### `Group` クラス
[https://otree.readthedocs.io/en/latest/models.html#group](https://otree.readthedocs.io/en/latest/models.html#group)

#### `group.*` （メソッド・変数）
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
- `field_maybe_none()`
    - 引数: フォームの変数名（文字列）．
    - 返り値: 引数の変数の値．変数の値が `None` であれば `None` がエラーを出すことなく返ってくる．
    - [https://otree.readthedocs.io/en/latest/misc/tips_and_tricks.html#field-maybe-none](https://otree.readthedocs.io/en/latest/misc/tips_and_tricks.html#field-maybe-none)

#### group からアクセスできる上層のオブジェクト
- subsession
- session



### `Player` クラス
[https://otree.readthedocs.io/en/latest/models.html#player](https://otree.readthedocs.io/en/latest/models.html#player)

#### `player.*` （メソッド・変数）
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

#### player からアクセスできる上層のオブジェクト
- participant
- group
- subsession
- session



### session オブジェクトと participant オブジェクト
- アプリをまたいで変数を受け渡したい場合，アプリ（ subsession ）よりも上位概念のオブジェクトにデータを記録する必要がある．
- アプリの `__init__.py` で（たとえば `Player` クラスのように）クラスを定義する必要はない．というより，アプリよりも上位概念なのでアプリ内では定義できない．
- [https://otree.readthedocs.io/en/latest/rounds.html#passing-data-between-rounds-or-apps](https://otree.readthedocs.io/en/latest/rounds.html#passing-data-between-rounds-or-apps)

#### session
- session オブジェクトは，セッションに参加している全参加者で共通の変数を記録できる．
- 変数を定義するには， `settings.py` の `SESSION_FIELDS` に変数名の文字列をリストで渡しておく．
- session の下位概念（ subsession， group， player ）からアクセスできる．
    - たとえば player オブジェクトを引数で受け取る関数の中では `player.session.変数名` とする．
- `SESSION_FIELDS` で 変数名を定義しなくても， `session.vars["変数名"]` で定義することも可能．
- 組み込みの変数・メソッド
    - `code`
        - セッションで固有なID．
        - `participant.session_id` でもアクセスできる．
    - `num_participants`
    - `config`
        - `settings.py` の `SESSION_CONFIGS` （の当該セッションの要素内） と `SESSION_CONFIG_DEFAULTS` で設定した変数がdictに入っている．
            - `config['participation_fee']`
            - `config['real_world_currency_per_point']`
    - `get_subsessions()`
    - `get_participants()`

#### participant
- participant オブジェクトは，セッションに参加している各参加者についてユニークな変数を記録できる．
- 変数を定義するには， `settings.py` の `PARTICIPANT_FIELDS` に変数名の文字列をリストで渡しておく．
- participant の下位概念である player からアクセスできる．
    - たとえば player オブジェクトを引数で受け取る関数の中では `player.participant.変数名` とする．
- `PARTICIPANT_FIELDS` で 変数名を定義しなくても， `participant.vars["変数名"]` で定義することも可能．
- 組み込みの変数・メソッド
    - `session_id`
        - セッションで固有なID．
    - `label`
        - ルームに名簿（ `participant_label_file` ）が設定してある場合，各参加者に対応する名簿上の文字列（ラベル）が得られる．
        - 管理者画面で得られるセッションワイドリンクにクエリパラメータ `?participant_label=123456789` などとつけると， `participant.label` に `123456789` が入る．
            - 参加者プール（ORSEEなど）やその他システム（Qualtricsなど）で発行されたIDを oTree に引き継ぐ場合に使うと良い．
    - `code`
        - oTreeが発行した，参加者で固有な（データベース内で固有な）ID．
        - アルファベットと数字のランダム文字列．
    - `id_in_session`
        - oTreeが割り振った，参加者で固有な通し番号（自然数）．
        - 管理者画面で番号は「P1」「P2」と表示される．
        - `player.id_in_subsession` でもアクセスできる．
    - `payoff`
        - 各 subsession における `player.payoff` の和．
    - `payoff_in_real_world_currency()`
    - `payoff_plus_participation_fee()`



### フィールドの型
[https://otree.readthedocs.io/en/latest/models.html#fields](https://otree.readthedocs.io/en/latest/models.html#fields)

- `models.BooleanField`
    - ブール型．
    - CSVファイルを出力するときは0/1で表示される．
    - `choices`
        - `True` のラベルを「協力」とする場合は以下のように記述する．
            ```python
            choices = [
                [True, "協力"],
                [False, "非協力"]
            ]
            ```
        - テンプレートタグで入力フォームを作るとき，（ `widget = widgets.RadioSelect` と指定しなくても）ラジオボタンでフォームが生成され，ラベルは `choices` で設定したものが表示される．
    - `widget`
        - `choices` を指定せずに `widget = widgets.CheckboxInput` と指定すると，チェックボックスでフォームが生成される．
    - `initial`
        - 初期値を設定する場合は指定する．
        - `initial` を指定していないとき，タイムアウトが発生すると `False` が入る．
        - 入力する機会がなければ `None` のまま．
    - `label`
        - テンプレートタグで入力フォームを作るとき，`label` に指定した文字列が （ `<input>` タグに対応する） `<label>` タグで生成される．
        - `label = ""` としてあるとき，以前はもコロンだけが必ず表示されていたが，いつの間にか改善され，最新バージョンでは `<label>` タグ自体生成されない．
    - `doc`
        - ドキュメントを記述してもよいが，（アプリなどのドキュメントとは異なり）管理者画面で自動的にドキュメントが表示されるような仕組みは無さそう．
        - [https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Column.params.doc](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Column.params.doc)
    - `help_text`
        - 参加者への指示などの文章を記述しておくと，テンプレートタグで入力フォームを作るときに小さい灰色の字で `help_text` に書いたものがフォームの下側に表示される．
        - テンプレートで `{{ form.変数名.description }}` と記述すると `help_text` に書いた文字列が展開される．
    - `blank`
        - 強制回答にしない場合は `blank = True` とする．
- `models.CurrencyField`
    - 通貨型．
    - 整数型や浮動小数点型の値を渡すと通貨型に変換される．
    - `choices`
        - `choices` を指定して，テンプレートタグで入力フォームを作るとき，デフォルトではドロップダウンメニューが生成される．
        - 組み込み関数 `currency_range(first, last, increment)` を使うと便利かも．
            - `cu(first)` ポイントから（最大） `cu(last)` ポイントまで `cu(increment)` ポイント刻みの等差数列（リスト）が返ってくる．
            - たとえば `choices = currency_range(0, 10, 3)` とすると「0ポイント」，「3ポイント」，「6ポイント」，「9ポイント」なる選択肢が生成される．
        - ラベルを設定するとき，以下のように記述できる．
            ```python
            choices = [
                [cu(0), "利己的"],
                [cu(300), "効率"],
                [cu(500), "平等"]
            ]
            ```
        - `choices` を指定した上で入力フォームを HTML タグ直打ちで実装する場合，送信される値を通貨型に変換した（丸められた）ものが `choices` で指定したリストに含まれないと oTree サーバーの検証に引っかかる．
    - `widget`
        - `choices` を指定した上で `widget = widgets.RadioSelect` と指定すると，選択肢がラジオボタンで生成される．
    - `initial`
        - 初期値を設定する場合は指定する．
        - `initial` を指定していないとき，タイムアウトが発生すると `cu(0)` が入る．
    - `label`
    - `doc`
    - `help_text`
    - `min`
        - 最小値．
        - 通貨型の設定でポイント表示を使っているときに `min = 0` として，かつ入力フォームを HTML タグ直打ちで実装する場合，入力フォームの値が「-0.49」でも oTree サーバーの検証は通過する．ただしデータベースには「0」で記録される．
    - `max`
        - 最大値．
    - `blank`
- `models.IntegerField`
    - 整数型．
    - 浮動小数点型の値を渡すとエラーとなる（入力フォームの送信時は oTree サーバーの検証に引っかかる）．
    - テンプレートタグで自由記述の入力フォームを作るとき， `type="number"` とした `<input>` で生成されるため，数字以外は入力できない．
    - `choices`
        - リッカート尺度の入力フォームを実装する場合，たとえば以下のように記述する．
            ```python
            choices = [
                [1, "全く同意できない"],
                [2, "同意できない"],
                [3, "どちらともいえない"],
                [4, "同意できる"],
                [5, "非常に同意できる"],
                [99, "わからない"]
            ]
            ```
    - `widget`
    - `initial`
    - `label`
    - `doc`
    - `help_text`
    - `min`
    - `max`
    - `blank`
- `models.FloatField`
    - 浮動小数点型．
    - Pythonの float と同様，15桁の有効桁数までは正確に表現できる．
    - テンプレートタグで自由記述の入力フォームを作るとき， `type="text"` とした `<input>` で生成されるため，数字以外も入力できてしまうが，文字列が送信されると oTree サーバーの検証に引っかかる．
    - `choices`
    - `widget`
    - `initial`
    - `label`
    - `doc`
    - `help_text`
    - `min`
    - `max`
    - `blank`
- `models.StringField`
    - 文字列型．
    - 10000字（`max_length` 字）を超えると， oTree サーバーの検証に引っかかる．
    - テンプレートタグで自由記述の入力フォームを作るとき， `type="text"` とした `<input>` で生成される．
    - たとえば `input1 = models.StringField()` と定義して，入力されたものをテンプレートで展開して表示するとき， `{{ input1 | escape }}` というように `escape` フィルターを使ったほうが良い（ oTree 3 ではデフォルトでエスケープされていた）．
        - たとえば参加者が「`<script>alert('あなたは対策不足です！');</script>`」と入力したとき， `{{ input1 }}` が展開されると， `<script>`で記述した JavaScript コードが実際に実行される（アラートが表示される）．
    - `choices`
    - `widget`
    - `initial`
    - `label`
    - `doc`
    - `help_text`
    - `max_length`
        - デフォルトは `max_length=10000`．
    - `blank`
- `models.LongStringField`
    - 可変長文字列型．
    - `models.StringField` と異なり，文字数制限なし．
    - テンプレートタグで自由記述の入力フォームを作るとき， `<textarea>` タグで生成される．複数行入力できるが，入力された値に含まれる改行コードは空白に置換される．
    - `choices` は使用できない．
    - 以上のこと以外は `models.StringField` と同じ．
    - `initial`
    - `label`
    - `doc`
    - `help_text`
    - `max_length`
        - デフォルトは `max_length=None`．
    - `blank`


### widget として使えるもの
[https://otree.readthedocs.io/en/latest/forms.html#widgets](https://otree.readthedocs.io/en/latest/forms.html#widgets)

- `widgets.CheckboxInput`
- `widgets.RadioSelect`
- `widgets.RadioSelectHorizontal`


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
    - 当該ページで使う入力フォームのモデルを `"player"`， `"group"`， `"subsession"` から選んで文字列を渡す．
    - どれか一つしか選べない．一つのページで player モデルの変数と group モデルの変数の両方の入力フォームを置くことはできない．たとえばとりあえず全部 player モデルでデータを記録しておき， oTree 内部で group モデルの変数に転記する，などで対処する．
    - 動的に変更することはできない．
- `form_fields`
    - 当該ページで入力フォームの変数名の文字列をリストで渡す．
    - `form_model` で指定したモデルで定義していない変数を指定してはいけない．
    - `form_fields` で複数要素を指定していて，かつテンプレートタグ `{{ formfields }}` を使う場合，リストの順番通り入力フォームが作られる．
    - 動的に変更する場合には組み込みメソッド `get_form_fields()` を使う．
        - `form_fields` で設定してあっても， `get_form_fields()` の設定の方が優先される．
    - `form_fields` で入力フォームの変数名を渡した場合，当該変数で `blank = True` と設定しない限り，（ `initial` を設定してあっても，）ページの入力フォームから適切な値が送信されなければ次のページへ進めない．
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
    - [https://otree.readthedocs.io/en/latest/forms.html#determining-form-fields-dynamically](https://otree.readthedocs.io/en/latest/forms.html#determining-form-fields-dynamically)
- `vars_for_template()`
    - 引数: `(player: Player)`
    - 返り値: 辞書型．
    - [https://otree.readthedocs.io/en/latest/pages.html#vars-for-template](https://otree.readthedocs.io/en/latest/pages.html#vars-for-template)
    - ページを読み込む度に実行される．
    - `js_vars()` よりも先に実行される．
    - [https://otree.readthedocs.io/en/latest/pages.html#vars-for-template](https://otree.readthedocs.io/en/latest/pages.html#vars-for-template)
- `js_vars()`
    - 引数: `(player: Player)`
    - 返り値: 辞書型．
    - [https://otree.readthedocs.io/en/latest/templates.html#passing-data-from-python-to-javascript-js-vars](https://otree.readthedocs.io/en/latest/templates.html#passing-data-from-python-to-javascript-js-vars)
    - ページを読み込む度に実行される．
    - `vars_for_template()` の後に実行される．
    - [https://otree.readthedocs.io/en/latest/templates.html#passing-data-from-python-to-javascript-js-vars](https://otree.readthedocs.io/en/latest/templates.html#passing-data-from-python-to-javascript-js-vars)
- `before_next_page()`
    - 引数: `(player: Player, timeout_happened)`
        - `timeout_happened` はブール値．時間制限でページが進んだ場合にフラグが立つ．
    - 返り値: 何も返してはいけない．
    - ページが進められたとき（フォームが送信されたとき），次のページを表示する前に実行される．
    - 当該ページが `is_displayed()` でスキップされていれば，実行されない．
    - [https://otree.readthedocs.io/en/latest/pages.html#before-next-page](https://otree.readthedocs.io/en/latest/pages.html#before-next-page)
- `is_displayed()`
    - 引数: `(player: Player)`
    - 返り値: ブール値．デフォルトでは `True` を返している．
    - ページを表示する前に実行される．
    - 返り値が `False` の場合，（ `page_sequence` で指定した順番の）次のページへスキップされる．
    - [https://otree.readthedocs.io/en/latest/pages.html#is-displayed](https://otree.readthedocs.io/en/latest/pages.html#is-displayed)
- `error_message()`
    - 引数: `(player: Player, values)`
        - `values` はフォームの変数名をキーとする辞書で，入力された値が取り出せる．
    - 返り値: フォームの変数名をキーとする辞書型で，値は各フォームに対応するエラーメッセージの文字列．エラーが無い場合は何も返さないか，空の辞書を返す．
    - ページが進められたとき（フォームが送信されたとき），次のページを表示する前に実行され，エラーがあれば同一ページにエラーメッセージを追加したものを表示する．
    - フォームごと個別に検証用の関数を定義することもできる．ページクラスの外側で `*_error_message()` を定義する．
    - [https://otree.readthedocs.io/en/latest/misc/tips_and_tricks.html#avoid-duplicated-validation-methods](https://otree.readthedocs.io/en/latest/misc/tips_and_tricks.html#avoid-duplicated-validation-methods)
- `get_timeout_seconds()`
    - 引数: `(player: Player)`
    - 返り値: 整数で秒数を渡すと，当該ページでの時間制限を設定できる． `timeout_seconds` と同様．
    - [https://otree.readthedocs.io/en/latest/timeouts.html#get-timeout-seconds](https://otree.readthedocs.io/en/latest/timeouts.html#get-timeout-seconds)
- `app_after_this_page()`
    - 引数: `(player: Player, upcoming_apps)`
        - `upcoming_apps` は `settings.py` で設定した `SESSION_CONFIGS.app_sequence` で，当該アプリより後のアプリ名（文字列）が並んだリスト．
    - 返り値: スキップ先のアプリ名（ `upcoming_apps` の要素である文字列）．
    - 複数のラウンドが設定してあるときも，アプリごとスキップする．同一アプリで次のラウンドの最初のページへスキップするのではない．
    - [https://otree.readthedocs.io/en/latest/pages.html#app-after-this-page](https://otree.readthedocs.io/en/latest/pages.html#app-after-this-page)



## 待機ページ
[https://otree.readthedocs.io/en/latest/multiplayer/waitpages.html](https://otree.readthedocs.io/en/latest/multiplayer/waitpages.html)


- クラス `WaitPage` を継承する．
- クラス名がURLに表示される．

### 待機ページの変数
- `template_name`
    - 待機ページもカスタマイズしようと思えばできる．そのときはテンプレートファイルのパスを渡す．
    - 使用例: [https://www.otreehub.com/projects/otree-snippets/](https://www.otreehub.com/projects/otree-snippets/) の "wait_page_timeout"
    - [https://otree.readthedocs.io/en/latest/misc/advanced.html#wait-pages](https://otree.readthedocs.io/en/latest/misc/advanced.html#wait-pages)
- `group_by_arrival_time`
    - `True` を渡せば，次のページ以降の group 分けが，当該待機ページに到達した順の group 分けになる．
    - 当該待機ページが `page_sequence` の先頭にある場合のみ `group_by_arrival_time = True` を設定できる．
    - `group_by_arrival_time = True` を設定している場合，全 player が当該待機ページを通るようにする． `is_displayed()` で特定の役割の player だけスキップする，というようなことをしてはいけない． `is_displayed()` で特定のラウンド以外はスキップする，という使い方はOK．
    - より細かい設定をしたい場合は， `group_by_arrival_time = True` を設定した上で，クラスの外側で `group_by_arrival_time_method()` を定義する．
    - 待機中，参加者のウィンドウがアクティブでない場合（たとえば他のタブを開いて遊んでいる場合），ドロップアウトとみなし group に参加させない． [https://groups.google.com/g/otree/c/XsFMNoZR7PY](https://groups.google.com/g/otree/c/XsFMNoZR7PY)
    - [https://otree.readthedocs.io/en/latest/multiplayer/waitpages.html#group-by-arrival-time](https://otree.readthedocs.io/en/latest/multiplayer/waitpages.html#group-by-arrival-time)
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
    - `title_text` と `body_text` それぞれのキーで値を渡すと，待機ページで表示されるタイトルと本文をデフォルトから変更できる．
    - `template_name` にパスを渡して独自のテンプレートファイルを使うときには，自分でキーを設定して値をテンプレートに渡せる．
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
- セッションの作成時に実行される．
    - 直前のアプリや直前のラウンド（ subsession ）における行動に応じた処理はできない．
    - ↑ そのような処理をしたい場合は（想定された使い方ではないが） `group_by_arrival_time_method()` を使う．
- subsession の数だけ（ `NUM_ROUNDS` だけ）実行される．
    - `creating_session()` において group 分けを実装するとき， 陽に `group_randomly()` が呼び出されないとデフォルトの group 分け（ `player.id_in_subsession` の昇順で `PLAYERS_PER_GROUP` 人ずつの group ）になる．
    - たとえば 1ラウンド目（ `if subsession.round_number == 1` ）に `group_randomly()` を使って group 分けして，以降のラウンドでは1ラウンド目の group を継承したい場合は， `if subsession.round_number > 1` なる条件において陽に `group_like_round(1)` を呼び出す必要がある．
- [組み込み関数 `creating_session()` の中で使うメソッド](#組み込み関数-creatingsession-の中で使うメソッド)
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
    - `page_sequence` の先頭に置かれた待機ページのクラスで `group_by_arrival_time = True` が設定されている場合にのみ呼び出される．
    - `creating_session()` と同様の用途で使える． `creating_session()` とは異なり，直前のアプリや直前のラウンド（ subsession ）における行動に応じた処理を実装できる．
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
- [https://otree.readthedocs.io/en/latest/forms.html#field-name-max](https://otree.readthedocs.io/en/latest/forms.html#field-name-max)


### `*_max`
- `*` の部分にフィールドの変数名を入れる．
- 引数: `(player: Player)`
- 返り値: フィールドの最大値．
- [https://otree.readthedocs.io/en/latest/forms.html#field-name-max](https://otree.readthedocs.io/en/latest/forms.html#field-name-max)


### `*_choices`
- `*` の部分にフィールドの変数名を入れる．
- 引数: `(player: Player)`
- 返り値: フィールドの選択肢のリスト．
- [https://otree.readthedocs.io/en/latest/forms.html#field-name-choices](https://otree.readthedocs.io/en/latest/forms.html#field-name-choices)


### `*_error_message`
- `*` の部分にフィールドの変数名を入れる．
- 引数: `(player: Player, value)`
    - `value` は入力されたフィールドの値．
- 返り値: エラーメッセージの文字列．
- [https://otree.readthedocs.io/en/latest/forms.html#field-name-error-message](https://otree.readthedocs.io/en/latest/forms.html#field-name-error-message)



## 自作の関数

- モジュールレベルやクラスの中で自分で関数を定義して使える．
- 自作の関数なので当然自分で引数を設定して良いが，呼び出す場所次第では引数の指定がある．たとえば， `after_all_players_arrive` に自作の関数を渡すときに引数は `group` （または `subsession` ）となる．
- 当然のこととして，自作の関数は陽に呼び出さないと実行されない．
- [https://otree.readthedocs.io/en/latest/misc/tips_and_tricks.html#avoid-duplicated-page-functions](https://otree.readthedocs.io/en/latest/misc/tips_and_tricks.html#avoid-duplicated-page-functions)



## `page_sequence`

- ページの順番をリストで渡す．ページ名の文字列ではなく，ページクラスのクラスオジェクトが要素．


{% endraw %}
