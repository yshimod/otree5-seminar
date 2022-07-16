{% raw %}

# `__init__.py` の書き方


## ファイルの冒頭

- シバン（shebang... `#!/usr/bin/env python3` みたいなやつ）は不要．

- 文字コード宣言（ `# -*- coding: utf-8 -*-` みたいなやつ）は不要（むしろ非推奨）．

- `otree.api` のインポート
  ```python
  from otree.api import *
  ```

    - oTree 3 ではインポートするものを細かく指定していた．  
    [https://otree.readthedocs.io/en/latest/misc/noself.html#about-the-new-format](https://otree.readthedocs.io/en/latest/misc/noself.html#about-the-new-format)


- 他にも使いたいモジュール（`time`，`random`，`json`，`numpy`など）があれば，冒頭でインポートしておく．



## 定数 `C` クラス
[https://otree.readthedocs.io/en/latest/models.html#constants](https://otree.readthedocs.io/en/latest/models.html#constants)

- アプリ内で参照する変数（定数）を定義する．

- `C` クラスで設定するものはCSVデータ出力には含まれない．CSVデータに記録するまでもないような定数（当面は実験の条件として変更しないようなもの）を定義する．

    - セッションごと（トリートメントごと）変化しうる変数は `C` クラスではなく `settings.py` の `SESSION_CONFIGS` の中で定義するべき． `SESSION_CONFIGS` で定義した変数はCSVデータ出力に含まれる．


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
    - 引数: フィールド名（文字列）．
    - 返り値: 引数のフィールドの値．フィールドの値が `None` であれば `None` がエラーを出すことなく返ってくる．
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
    - 引数: フィールド名（文字列）．
    - 返り値: 引数のフィールドの値．フィールドの値が `None` であれば `None` がエラーを出すことなく返ってくる．
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
    - 引数: フィールド名（文字列）．
    - 返り値: 引数のフィールドの値．フィールドの値が `None` であれば `None` がエラーを出すことなく返ってくる．
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

- 自分で変数を定義するには， `settings.py` の `SESSION_FIELDS` に変数名の文字列をリストで渡しておく．

- session の下位概念（ subsession， group， player ）からアクセスできる．

    - たとえば player オブジェクトを引数で受け取る関数の中では `player.session.変数名` とする．


- （All apps の）CSV出力で，自分で定義した変数と組み込みの変数も出力される．

- 組み込みの変数・メソッド

    - `code`

        - セッションで固有なID．

        - `participant.session_id` でもアクセスできる．

    - `num_participants`

    - `config`

        - `settings.py` の `SESSION_CONFIGS` （の当該セッションの要素内） と `SESSION_CONFIG_DEFAULTS` で設定した変数が辞書オブジェクトに入っている．

            - `config['participation_fee']`

            - `config['real_world_currency_per_point']`

    - `get_subsessions()`

    - `get_participants()`

    - `vars`

        - `SESSION_FIELDS` で変数名を定義しなくても， `session.vars["変数名"] = なにか` として記録できる．

        - `vars` の要素として記録したものは，（ `custom_export()` を使わない限り）CSV出力で出力されない．



#### participant

- participant オブジェクトは，セッションに参加している各参加者についてユニークな変数を記録できる．

- 自分で変数を定義するには， `settings.py` の `PARTICIPANT_FIELDS` に変数名の文字列をリストで渡しておく．

- participant の下位概念である player からアクセスできる．

    - たとえば player オブジェクトを引数で受け取る関数の中では `player.participant.変数名` とする．

- （All appsの）CSV出力で，自分で定義した変数と組み込みの変数も出力される．

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

    - `vars`

        - `PARTICIPANT_FIELDS` で変数名を定義しなくても， `participant.vars["変数名"] = なにか` として記録できる．

        - `vars` の要素として記録したものは，（ `custom_export()` を使わない限り）CSV出力で出力されない．



### ExtraModel
[https://otree.readthedocs.io/en/latest/misc/advanced.html#extramodel](https://otree.readthedocs.io/en/latest/misc/advanced.html#extramodel)

- データベースにおいて `Player`， `Group`， `Subsession` の各テーブルのレコード（1行）は，それぞれ player， group， subsession に関してユニークである．つまり，（1つのラウンドで）1つのフィールドには1つの値しか入れることができない．

- ExtraModel を使えば，独自にテーブルのフィールドを定義することができる． ExtraModel のテーブルは player （，group，subsession ）あたり1レコード，の制約がなく，1人の player について複数のレコードを作ることができる．

- ExtraModel のデータは自動には CSV ファイル出力ができないため， [`custom_export()`](#customexport) を使って出力するように設定する．

- 使い方:

    1. `ExtraModel` クラスを継承したクラスを定義する．定義しただけテーブルが作られるため，複数のクラスを定義することができる．

      ```python
      class MyModel(ExtraModel):
          # なにか
      ```

    1. `Player` クラスなどと同様にフィールドを定義する．

      ```python
      var1 = models.IntegerField()
      var2 = models.StringField()
      ```

    1. フィールドに記録したデータが，どの group の，どの player のデータなのかを記録するためのフィールドを定義する．対戦相手 player が誰であるのかを記録するためのフィールドも定義できる．

      ```python
      me = models.Link(Player)
      opponent = models.Link(Player)
      group = models.Link(Group)
      subsession = models.Link(Subsession)
      ```

    1. たとえば，クラスの定義は以下のようになる．

      ```python
      class MyModel(ExtraModel):
          me = models.Link(Player)
          opponent = models.Link(Player)
          var1 = models.IntegerField()
          var2 = models.BooleanField()
      ```

    1. データを記録するためには `create()` メソッドでレコードを作る．
        - 引数で記録したいデータを渡す． 
        - player （など）とレコードとの紐付けのためのフィールドには， `player` オブジェクトそのものを渡す．
        - 定義してあるフィールドすべての値を渡す必要はない．渡さなければ，そのフィールドは `None` となる．

        - たとえば `before_next_page()` の中で以下のように使う．

        ```python
        @staticmethod
        def before_next_page(player: Player, timeout_happened):
            MyModel.create(
                me = player,
                opponent = player.get_others_in_group()[0],    ## opponent には相手 player のオブジェクトを渡す．
                var1 = 123
            )
        ```

    1. すでに作られたレコードを取り出すためには `filter()` メソッドでクエリにかける．
        - 返り値は定義した `ExtraModel` クラスのインスタンスオブジェクトのリストである．
        - クエリの条件に合致するレコードが1件であってもリストで返ってくることに注意．
        - クエリの対象はテーブル全体であり，セッションごと `resetdb` をしていない場合は別のセッションでのデータにもアクセスできる．逆に言えば，クエリの条件で player， group， subsession を使わない場合，別のセッションで作られたレコードも引っかかる．
        - 既に存在するレコードのフィールドの値を更新することもできる．

        - たとえば `vars_for_template()` の中で以下のように使う．
        ```python
        @staticmethod
        def vars_for_template(player: Player):
            ## opponent が 自分である（対戦相手の）レコードを取り出す．
            retrieved_record = MyModel.filter(opponent = player)[0]

            her_var1 = retrieved_record.var1
            retrieved_record.var2 = True

            ## 他人の player オブジェクトを取り出すことも可能．
            opponent: Player = retrieved_record.me
            opponent_payoff = opponent.payoff

            return dict(
                her_var1 = her_var1,
                opponent_payoff = opponent_payoff
            )
        ```



### フィールドの型
[https://otree.readthedocs.io/en/latest/models.html#fields](https://otree.readthedocs.io/en/latest/models.html#fields)

- `models.BooleanField`
    - 真偽値型．
    - CSVファイルを出力するときは0/1で表示される．
    - `choices`
        - `[データベースに記録する値, テンプレートで展開するラベル]` の組（リスト）のリストを渡す．
        - `True` のラベルを「協力」とする場合は以下のように記述する．
        ```python
        choices = [
            [True, "協力"],
            [False, "非協力"]
        ]
        ```
        - テンプレートタグで入力フォームを作るとき，（ `widget = widgets.RadioSelect` と指定しなくても）ラジオボタンで入力フォームが生成され，ラベルは `choices` で設定したものが表示される．
    - `widget`
        - `choices` を指定せずに `widget = widgets.CheckboxInput` と指定すると，チェックボックスで入力フォームが生成される．
    - `initial`
        - 初期値を設定する場合は指定する．
        - `initial` を指定していないとき，タイムアウトが発生すると `False` が入る．
        - 入力する機会がなければ `None` のまま．
    - `label`
        - テンプレートタグで入力フォームを作るとき，`label` に指定した文字列が （ `<input>` タグに対応する） `<label>` タグで生成される．
        - 以前は，指定した文字列の後にコロンが自動的に追加されて， `label = ""` の場合であってもコロンだけが表示されていた．しかしいつの間にか改善され，コロンが追加される実装は廃止となり， `label = ""` の場合は `<label>` タグ自体生成されない．
    - `doc`
        - ドキュメントを記述してもよいが，（アプリなどのドキュメントとは異なり）管理者画面で自動的にドキュメントが表示されるような仕組みは無さそう．
        - [https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Column.params.doc](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Column.params.doc)
    - `help_text`
        - 参加者への指示などの文章を記述しておくと，テンプレートタグで入力フォームを作るときに小さい灰色の字で `help_text` に書いたものが入力フォームの下側に表示される．
        - テンプレートで `{{ form.フィールド名.description }}` と記述すると `help_text` に書いた文字列が展開される．
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
    - `min` や `max` を設定すると `<input>` 要素の `min`， `max` 属性にも値が渡され，ブラウザでも最小値，最大値の検証が行われる．
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
    - `min` や `max` を設定すると `<input>` 要素の `min`， `max` 属性にも値が渡されるが， `type="text"` であるため，結局，ブラウザでの最小値，最大値の検証は機能しないことに注意．
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
        - ブラウザでは検証されない．文字列の長さの検証を実装するには HTML タグを直書きして `<input>` 要素の `maxlength` 属性を設定する．
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



### テンプレートタグで入力フォームを作る
[https://otree.readthedocs.io/en/latest/forms.html#widgets](https://otree.readthedocs.io/en/latest/forms.html#widgets)

1. データを参加者に入力させたいページのクラスにおいて `form_model` と `form_fields` を設定する．

1. テンプレートで `{{ formfields }}` タグを記述する．一つずつ特定して設置する場合には `{{ formfield "フィールド名" }}` と記述する．

1. テンプレートタグで入力フォームを設置するとき，HTML タグが生成される．


- `models.BooleanField()` を特に設定せず使った場合:

  ```python
  test = models.BooleanField(
      label="ラベル"
  )
  ```

  <p class="codepen" data-height="300" data-theme-id="dark" data-default-tab="html,result" data-slug-hash="rNdVWGO" data-editable="true" data-user="yshimod" style="height: 300px; box-sizing: border-box; display: flex; align-items: center; justify-content: center; border: 2px solid; margin: 1em 0; padding: 1em;">
      <span>See the Pen <a href="https://codepen.io/yshimod/pen/rNdVWGO">
      oTree Boolean</a> by yshimod (<a href="https://codepen.io/yshimod">@yshimod</a>)
      on <a href="https://codepen.io">CodePen</a>.</span>
  </p>


- `models.BooleanField()` で選択肢を設定した場合:

  ```python
  test = models.BooleanField(
      label="ラベル",
      choices=[
          [True, "協力する"],
          [False, "裏切る"]
      ]
  )
  ```

  <p class="codepen" data-height="300" data-theme-id="dark" data-default-tab="html,result" data-slug-hash="poLJNWM" data-editable="true" data-user="yshimod" style="height: 300px; box-sizing: border-box; display: flex; align-items: center; justify-content: center; border: 2px solid; margin: 1em 0; padding: 1em;">
      <span>See the Pen <a href="https://codepen.io/yshimod/pen/poLJNWM">
      oTree Boolean Labeled</a> by yshimod (<a href="https://codepen.io/yshimod">@yshimod</a>)
      on <a href="https://codepen.io">CodePen</a>.</span>
  </p>


- `models.BooleanField()` で `widgets.RadioSelectHorizontal` を使った場合:

  ```python
  test = models.BooleanField(
      label="ラベル",
      widget=widgets.RadioSelectHorizontal
  )
  ```

  <p class="codepen" data-height="300" data-theme-id="dark" data-default-tab="html,result" data-slug-hash="bGvdBmJ" data-editable="true" data-user="yshimod" style="height: 300px; box-sizing: border-box; display: flex; align-items: center; justify-content: center; border: 2px solid; margin: 1em 0; padding: 1em;">
      <span>See the Pen <a href="https://codepen.io/yshimod/pen/bGvdBmJ">
      oTree Boolean ckbox</a> by yshimod (<a href="https://codepen.io/yshimod">@yshimod</a>)
      on <a href="https://codepen.io">CodePen</a>.</span>
  </p>


- `models.BooleanField()` で `widgets.CheckboxInput` を使った場合:

  ```python
  test = models.BooleanField(
      label="ラベル",
      widget=widgets.CheckboxInput
  )
  ```

  <p class="codepen" data-height="300" data-theme-id="dark" data-default-tab="html,result" data-slug-hash="WNzvogB" data-editable="true" data-user="yshimod" style="height: 300px; box-sizing: border-box; display: flex; align-items: center; justify-content: center; border: 2px solid; margin: 1em 0; padding: 1em;">
      <span>See the Pen <a href="https://codepen.io/yshimod/pen/WNzvogB">
      oTree Boolean Checkbox</a> by yshimod (<a href="https://codepen.io/yshimod">@yshimod</a>)
      on <a href="https://codepen.io">CodePen</a>.</span>
  </p>


- `models.IntegerField()` を特に設定せず使った場合:

  ```python
  test = models.IntegerField(
      label="ラベル"
  )
  ```

  <p class="codepen" data-height="300" data-theme-id="dark" data-default-tab="html,result" data-slug-hash="rNdVWoL" data-editable="true" data-user="yshimod" style="height: 300px; box-sizing: border-box; display: flex; align-items: center; justify-content: center; border: 2px solid; margin: 1em 0; padding: 1em;">
      <span>See the Pen <a href="https://codepen.io/yshimod/pen/rNdVWoL">
      Untitled</a> by yshimod (<a href="https://codepen.io/yshimod">@yshimod</a>)
      on <a href="https://codepen.io">CodePen</a>.</span>
  </p>


- `models.IntegerField()` で最大値を設定した場合:

  ```python
  test = models.IntegerField(
      label="ラベル",
      max=100
  )
  ```

  <p class="codepen" data-height="300" data-theme-id="dark" data-default-tab="html,result" data-slug-hash="wvmaoNZ" data-editable="true" data-user="yshimod" style="height: 300px; box-sizing: border-box; display: flex; align-items: center; justify-content: center; border: 2px solid; margin: 1em 0; padding: 1em;">
      <span>See the Pen <a href="https://codepen.io/yshimod/pen/wvmaoNZ">
      oTree Integer Max</a> by yshimod (<a href="https://codepen.io/yshimod">@yshimod</a>)
      on <a href="https://codepen.io">CodePen</a>.</span>
  </p>


- `models.IntegerField()` で選択肢を設定した場合:

  ```python
  test = models.IntegerField(
      label="ラベル",
      choices = [
          [1, "全く同意できない"],
          [2, "同意できない"],
          [3, "どちらともいえない"],
          [4, "同意できる"],
          [5, "非常に同意できる"],
          [99, "わからない"]
      ]
  )
  ```

  <p class="codepen" data-height="300" data-theme-id="dark" data-default-tab="html,result" data-slug-hash="VwXLmRX" data-editable="true" data-user="yshimod" style="height: 300px; box-sizing: border-box; display: flex; align-items: center; justify-content: center; border: 2px solid; margin: 1em 0; padding: 1em;">
      <span>See the Pen <a href="https://codepen.io/yshimod/pen/VwXLmRX">
      oTree Integer Labeled</a> by yshimod (<a href="https://codepen.io/yshimod">@yshimod</a>)
      on <a href="https://codepen.io">CodePen</a>.</span>
  </p>


- `models.IntegerField()` で選択肢を設定した上で `widgets.RadioSelect` を使った場合:

  ```python
  test = models.IntegerField(
      label="ラベル",
      choices = [
          [1, "全く同意できない"],
          [2, "同意できない"],
          [3, "どちらともいえない"],
          [4, "同意できる"],
          [5, "非常に同意できる"],
          [99, "わからない"]
      ],
      widget=widgets.RadioSelect
  )
  ```

  <p class="codepen" data-height="300" data-theme-id="dark" data-default-tab="html,result" data-slug-hash="PoRqbgX" data-editable="true" data-user="yshimod" style="height: 300px; box-sizing: border-box; display: flex; align-items: center; justify-content: center; border: 2px solid; margin: 1em 0; padding: 1em;">
      <span>See the Pen <a href="https://codepen.io/yshimod/pen/PoRqbgX">
      Untitled</a> by yshimod (<a href="https://codepen.io/yshimod">@yshimod</a>)
      on <a href="https://codepen.io">CodePen</a>.</span>
  </p>


- `models.FloatField()` を特に設定せず使った場合:

  ```python
  test = models.FloatField(
      label="ラベル"
  )
  ```

  <p class="codepen" data-height="300" data-theme-id="dark" data-default-tab="html,result" data-slug-hash="yLKNVde" data-editable="true" data-user="yshimod" style="height: 300px; box-sizing: border-box; display: flex; align-items: center; justify-content: center; border: 2px solid; margin: 1em 0; padding: 1em;">
      <span>See the Pen <a href="https://codepen.io/yshimod/pen/yLKNVde">
      oTree Float</a> by yshimod (<a href="https://codepen.io/yshimod">@yshimod</a>)
      on <a href="https://codepen.io">CodePen</a>.</span>
  </p>


- `models.FloatField()` で最小値を設定した場合:

  ```python
  test = models.FloatField(
      label="ラベル",
      min=10/3
  )
  ```

  <p class="codepen" data-height="300" data-theme-id="dark" data-default-tab="html,result" data-slug-hash="xxWGRvL" data-editable="true" data-user="yshimod" style="height: 300px; box-sizing: border-box; display: flex; align-items: center; justify-content: center; border: 2px solid; margin: 1em 0; padding: 1em;">
      <span>See the Pen <a href="https://codepen.io/yshimod/pen/xxWGRvL">
      oTree Float Min</a> by yshimod (<a href="https://codepen.io/yshimod">@yshimod</a>)
      on <a href="https://codepen.io">CodePen</a>.</span>
  </p>

    - ブラウザでの最小値の検証は機能しないことに注意．


- `models.StringField()` を特に設定せず使った場合:

  ```python
  test = models.StringField(
      label="ラベル"
  )
  ```

  <p class="codepen" data-height="300" data-theme-id="dark" data-default-tab="html,result" data-slug-hash="VwXLPYd" data-editable="true" data-user="yshimod" style="height: 300px; box-sizing: border-box; display: flex; align-items: center; justify-content: center; border: 2px solid; margin: 1em 0; padding: 1em;">
      <span>See the Pen <a href="https://codepen.io/yshimod/pen/VwXLPYd">
      oTree String</a> by yshimod (<a href="https://codepen.io/yshimod">@yshimod</a>)
      on <a href="https://codepen.io">CodePen</a>.</span>
  </p>


- `models.LongStringField()` を特に設定せず使った場合:

  ```python
  test = models.LongStringField(
      label="ラベル"
  )
  ```

  <p class="codepen" data-height="300" data-theme-id="dark" data-default-tab="html,result" data-slug-hash="RwMPKrq" data-editable="true" data-user="yshimod" style="height: 300px; box-sizing: border-box; display: flex; align-items: center; justify-content: center; border: 2px solid; margin: 1em 0; padding: 1em;">
      <span>See the Pen <a href="https://codepen.io/yshimod/pen/RwMPKrq">
      oTree LongString</a> by yshimod (<a href="https://codepen.io/yshimod">@yshimod</a>)
      on <a href="https://codepen.io">CodePen</a>.</span>
  </p>





### HTML タグを直書きして入力フォームを作る
[https://otree.readthedocs.io/en/latest/forms.html#raw-html-widgets](https://otree.readthedocs.io/en/latest/forms.html#raw-html-widgets)

1. データを参加者に入力させたいページのクラスにおいて `form_model` と `form_fields` を設定する．

1. フィールド名を， `<input>` 要素の `name` 属性に設定する．

1. テンプレートタグ `{{ formfield_errors 'フィールド名' }}` を書いておくと， oTree による検証のエラーメッセージを表示することができる．




## ページ
[https://otree.readthedocs.io/en/latest/pages.html](https://otree.readthedocs.io/en/latest/pages.html)

- クラス `Page` を継承する．

- クラス名がURLに表示される．

- 当該ページで使うテンプレートファイルのファイル名は `クラス名.html` にする．大文字・小文字を厳密に区別しておく．



### ページクラスの変数

- `template_name`
    - テンプレートファイルのパスを渡せば，ページクラスの名称とファイル名が一致していなくても良い．
    - 一つのテンプレートファイルを複数のページクラスで使い回すことも可能．
    - パスはアプリディレクトリから指定する．
        - `template_name = 'アプリ名/MyPage.html'`
        - 同一アプリ内であれば， `template_name = __name__ + '/MyPage.html'` と書いても良い．
    - （ファイルシステムによってはファイル名の大文字と小文字の区別をしないが，）大文字・小文字を厳密に区別しておく．


- `form_model`
    - 当該ページで使う入力フォームのモデルを `"player"`， `"group"`， `"subsession"` から選んで文字列を渡す．
    - どれか一つしか選べない．一つのページで player モデルのフィールドと group モデルのフィールドの両方の入力フォームを置くことはできない．たとえばとりあえず全部 player モデルでデータを記録しておき， oTree 内部で group モデルのフィールドに転記する，などで対処する．
    - 動的に変更することはできない．


- `form_fields`
    - 当該ページで入力フォームのフィールド名の文字列をリストで渡す．
    - `form_model` で指定したモデルで定義していないフィールドを指定してはいけない．
    - `form_fields` で複数要素を指定していて，かつテンプレートタグ `{{ formfields }}` を使う場合，リストの順番通り入力フォームが作られる．
    - 動的に変更する場合には組み込みメソッド `get_form_fields()` を使う．
        - `form_fields` で設定してあっても， `get_form_fields()` の設定の方が優先される．
    - `form_fields` で入力フォームのフィールド名を渡した場合，当該フィールドの定義で `blank = True` と設定しない限り，（ `initial` を設定してあっても，）ページの入力フォームから適切な値が送信されなければ次のページへ進めない．


- `timeout_seconds`
    - 整数で秒数を渡すと，当該ページでの時間制限を設定できる．
    - 動的に変更する場合には組み込みメソッド `get_timeout_seconds()` を使う．


- `timer_text`
    - 時間制限を設定したときに表示されるメッセージ（文字列）を指定する．
    - デフォルトでは「このページでの残り時間」と表示される．



### ページクラスの組み込みメソッド

関数を定義する前にデコレータ `@staticmethod` を記述しておく．
ただし，デコレータをつけずにおいてインスタンスメソッド（第1引数を `self` に受け取る関数）として使うことはできない．


- `live_method()`
    - 引数: `(player: Player, data)`
    - 返り値: 辞書型．キーは `id_in_group` の自然数とする．
        - group の全員に送信する場合のキーは `0`．
        - 他の group や subsession 全体へは送信できない．
    - [https://otree.readthedocs.io/en/latest/live.html](https://otree.readthedocs.io/en/latest/live.html)


- `get_form_fields()`
    - 引数: `(player: Player)`
    - 返り値: フィールド名の文字列のリスト． `form_fields` と同様．
    - ページを読み込む度に実行される（しかも2回実行される？）．
    - `vars_for_template()` よりも先に実行される．
    - [https://otree.readthedocs.io/en/latest/forms.html#determining-form-fields-dynamically](https://otree.readthedocs.io/en/latest/forms.html#determining-form-fields-dynamically)


- `vars_for_template()`
    - 引数: `(player: Player)`
    - 返り値: 辞書型．
    - ページを読み込む度に実行される．
    - `js_vars()` よりも先に実行される．
    - [https://otree.readthedocs.io/en/latest/pages.html#vars-for-template](https://otree.readthedocs.io/en/latest/pages.html#vars-for-template)


- `js_vars()`
    - 引数: `(player: Player)`
    - 返り値: 辞書型．
    - ページを読み込む度に実行される．
    - `vars_for_template()` の後に実行される．
    - 返り値のオブジェクトが， HTML のコンテンツブロックの要素の直前において， `js_vars` なる変数名で定義される．したがって， `js_vars` から値を取り出せば良い．
        - たとえば `js_vars()` の返り値が `return {"testlist" = list(range(0, 10, 2))}` であるとき， oTree サーバーは自動的に以下の要素を展開する．
        ```html
        <script>var js_vars = {"testlist": [0, 2, 4, 6, 8]};</script>
        ```
    - [https://otree.readthedocs.io/en/latest/templates.html#passing-data-from-python-to-javascript-js-vars](https://otree.readthedocs.io/en/latest/templates.html#passing-data-from-python-to-javascript-js-vars)


- `before_next_page()`
    - 引数: `(player: Player, timeout_happened)`
        - `timeout_happened` は真偽値．時間制限でページが進んだ場合にフラグが立つ．
    - 返り値: 何も返してはいけない．
    - ページが進められたとき（フォームが送信されたとき），次のページを表示する前に実行される．
    - 当該ページが `is_displayed()` でスキップされていれば，実行されない．
    - [https://otree.readthedocs.io/en/latest/pages.html#before-next-page](https://otree.readthedocs.io/en/latest/pages.html#before-next-page)


- `is_displayed()`
    - 引数: `(player: Player)`
    - 返り値: 真偽値．デフォルトでは `True` を返している．
    - ページを表示する前に実行される．
    - 返り値が `False` の場合，（ `page_sequence` で指定した順番の）次のページへスキップされる．
    - [https://otree.readthedocs.io/en/latest/pages.html#is-displayed](https://otree.readthedocs.io/en/latest/pages.html#is-displayed)


- `error_message()`
    - 引数: `(player: Player, values)`
        - `values` は入力フォームのフィールド名をキーとする辞書で，入力された値が取り出せる．
        - 検証したい入力フォームが定義してあるモデルが player でなくても（当該ページのクラスで `form_model = "player"` と設定してある場合以外であっても），1つ目の引数は `player` ．
            - ページに入力フォームが無い場合であっても， `error_message()` は使える．
    - 返り値: 入力フォームのフィールド名をキーとする辞書型で，値は各入力フォームに対応するエラーメッセージの文字列．
        - エラーが無い場合は何も返さないか，空の辞書を返す．
        - 辞書型ではなく，単なる文字列を返すとき，画面のタイトルの直下にその文字列が表示される．
            - 作例: [https://www.otreehub.com/projects/otree-snippets/](https://www.otreehub.com/projects/otree-snippets/) の「experimenter_input」．
    - ページが進められたとき（フォームが送信されたとき），次のページを表示する前に実行され，エラーがあれば同一ページにエラーメッセージを追加したものを表示する．
    - 入力フォームごと個別に検証用の関数を定義することもできる．ページクラスの外側で `フィールド名_error_message()` を定義する．
    - [https://otree.readthedocs.io/en/latest/misc/tips_and_tricks.html#avoid-duplicated-validation-methods](https://otree.readthedocs.io/en/latest/misc/tips_and_tricks.html#avoid-duplicated-validation-methods)


- `get_timeout_seconds()`
    - 引数: `(player: Player)`
    - 返り値: 整数で秒数を渡すと，当該ページでの時間制限を設定できる． `timeout_seconds` と同様．
    - [https://otree.readthedocs.io/en/latest/timeouts.html#get-timeout-seconds](https://otree.readthedocs.io/en/latest/timeouts.html#get-timeout-seconds)


- `app_after_this_page()`
    - 引数: `(player: Player, upcoming_apps)`
        - `upcoming_apps` は `settings.py` で設定した `SESSION_CONFIGS` の `app_sequence` で，当該アプリより後のアプリ名（文字列）が並んだリスト．
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
    - 待機中，参加者のウィンドウがアクティブでない場合（たとえば他のタブを開いて遊んでいる場合），ドロップアウトとみなし group に参加させない．  
    [https://groups.google.com/g/otree/c/XsFMNoZR7PY](https://groups.google.com/g/otree/c/XsFMNoZR7PY)
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
ただし，デコレータをつけずにおいてインスタンスメソッドとして（第1引数を `self` にして）使うことはできない．


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
    - 返り値: 真偽値．デフォルトでは `True` を返している．
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

- group の編成（各 player がどの group に属し， `id_in_group` は何か）をカスタマイズするときに使う．

- 刺激呈示などで乱数を使う場合， `creating_session()` で乱数を引き出して player のフィールドなどに記録しておくと良い．

- 引数: `(subsession: Subsession)`

- 返り値: 何も返してはいけない．

- セッションの作成時に実行される．
    - 直前のアプリや直前のラウンド（ subsession ）における行動に応じた処理はできない．
    - ↑ そのような処理をしたい場合は `group_by_arrival_time_method()` を使う．


- subsession の数だけ（ `NUM_ROUNDS` だけ）実行される．
    - `creating_session()` において group 分けを実装するとき， 陽に `group_randomly()` が呼び出されないとデフォルトの group 分け（ `player.id_in_subsession` の昇順で `PLAYERS_PER_GROUP` 人ずつの group ）になる．
    - たとえば 1ラウンド目（ `if subsession.round_number == 1` ）に `group_randomly()` を使って group 分けして，以降のラウンドでは1ラウンド目の group を継承したい場合は， `if subsession.round_number > 1` なる条件において陽に `group_like_round(1)` を呼び出す必要がある．


- [組み込み関数 `creating_session()` の中で使うメソッド](#組み込み関数-creatingsession-の中で使うメソッド)

- [https://otree.readthedocs.io/en/latest/treatments.html](https://otree.readthedocs.io/en/latest/treatments.html)



### `custom_export()`

- 独自の構成でデータを CSV ファイルとして出力するときに使う．
    - `ExtraModel` のフィールドに記録したデータはそのままでは CSV ファイルとして出力されないため， `custom_export()` で出力する．
    - 通常のフィールドに JSON 文字列でデータが保存してある場合，これを `custom_export()` の中でパースして，一つのセルに一つのデータが入るようにする．


- 引数: (`players: list[Player]`) 
    - `players` は `player` オブジェクトが入ったリスト．


- 返り値: `yield` で CSV ファイルで出力したいデータをリストで返す．
    - たとえば，各 player の `player.payoff` を出力したい場合，以下のようにすれば良い．
    ```python
    def custom_export(players: list[Player]):
        ## まず CSV ファイルの1行目に列名を出力すると良い．
        yield ["session.code", "player.id_in_subsession", "group.id_in_subsession", "id_in_group", "payoff"]

        for p in players:
            ## forループで yield を使うと，リストのデータが CSV ファイルの行として次々追記される．
            yield [p.session.code, p.id_in_subsession, p.group.id_in_subsession, p.id_in_group, p.payoff]
    ```


- セッションごと CSV ファイルを出力することはできず，データベースに保存されているすべてのデータが出力される．したがって，各行にはセッションに固有なID（ `player.session.code` ）を含めておくと良い．

- "CSV Data Export" で出力する度に関数が実行される．

- [https://otree.readthedocs.io/en/latest/admin.html#custom-data-exports](https://otree.readthedocs.io/en/latest/admin.html#custom-data-exports)



### `group_by_arrival_time_method()`

- 引数: `(subsession: Subsession, waiting_players: list[Player])`
    - `waiting_players` 到達して待機中の player オブジェクトが入ったリスト．


- 返り値: player オブジェクトが入ったリスト．リストの要素のメンバーで group となる．

- 待機ページにプレイヤーが到達する度に実行される．
    - `page_sequence` の先頭に置かれた待機ページのクラスで `group_by_arrival_time = True` が設定されている場合にのみ呼び出される．
    - `creating_session()` と同様の用途で使える． `creating_session()` とは異なり，直前のアプリや直前のラウンド（ subsession ）における行動に応じた処理を実装できる．


- [https://otree.readthedocs.io/en/latest/multiplayer/waitpages.html#group-by-arrival-time-method](https://otree.readthedocs.io/en/latest/multiplayer/waitpages.html#group-by-arrival-time-method)



### `vars_for_admin_report()`

- 独自の管理者画面を使いたいときに使う．

- 引数: `(subsession: Subsession)`

- 返り値: 辞書型．

- 定義する場合は，当該アプリのディレクトリに `admin_report.html` なるファイル名でテンプレートファイルを作成しておく．

- [https://otree.readthedocs.io/en/latest/admin.html#customizing-the-admin-interface-admin-reports](https://otree.readthedocs.io/en/latest/admin.html#customizing-the-admin-interface-admin-reports)



### `フィールド名_max()`

- `フィールド名` の部分にフィールドのフィールド名を入れる．

- oTree サーバーを立ち上げる段階ではフィールドの最大値が決められない場合に，この関数を使う．

- 引数: 当該フィールドが定義されているクラスのインスタンス．
    - たとえば player のフィールドであれば，引数は `player` ．


- 返り値: フィールドの最大値．

- [https://otree.readthedocs.io/en/latest/forms.html#field-name-max](https://otree.readthedocs.io/en/latest/forms.html#field-name-max)



### `フィールド名_min()`

- `フィールド名` の部分にフィールドのフィールド名を入れる．

- `フィールド名_max()` の最小値版．

- 引数: 当該フィールドが定義されているクラスのインスタンス．
    - たとえば player のフィールドであれば，引数は `player` ．


- 返り値: フィールドの最小値．

- [https://otree.readthedocs.io/en/latest/forms.html#field-name-max](https://otree.readthedocs.io/en/latest/forms.html#field-name-max)



### `フィールド名_choices`

- `フィールド名` の部分にフィールドのフィールド名を入れる．

- oTree サーバーを立ち上げる段階では入力フォームの選択肢が決められない場合に，この関数を使う．

- 引数: 当該フィールドが定義されているクラスのインスタンス．
    - たとえば player のフィールドであれば，引数は `player` ．


- 返り値: フィールドの選択肢のリスト．

- [https://otree.readthedocs.io/en/latest/forms.html#field-name-choices](https://otree.readthedocs.io/en/latest/forms.html#field-name-choices)



### `フィールド名_error_message`

- `フィールド名` の部分にフィールドのフィールド名を入れる．

- 入力フォームの oTree サーバー側での検証に引っかかった場合に表示するエラーメッセージをカスタマイズするときに使う．

- 引数: `(*, value)`
    - `value` は入力されたフィールドの値．
    - `*` には当該フィールドが定義されているクラスのインスタンスを書く．
        - たとえば player のフィールドであれば，引数は `player` ．


- 返り値: エラーメッセージの文字列．

- [https://otree.readthedocs.io/en/latest/forms.html#field-name-error-message](https://otree.readthedocs.io/en/latest/forms.html#field-name-error-message)



## 自作の関数

- モジュールレベルやクラスの中で自分で関数を定義して使える．

- 自作の関数なので当然自分で引数を設定して良いが，呼び出す場所次第では引数の指定がある．たとえば， `after_all_players_arrive` に自作の関数を渡すときに引数は `group` （または `subsession` ）となる．

- 当然のこととして，自作の関数は陽に呼び出さないと実行されない．

- [https://otree.readthedocs.io/en/latest/misc/tips_and_tricks.html#avoid-duplicated-page-functions](https://otree.readthedocs.io/en/latest/misc/tips_and_tricks.html#avoid-duplicated-page-functions)



## `page_sequence`

- ページの順番をリストで渡す．ページ名の文字列ではなく，ページクラスのクラスオジェクトが要素．




<script async src="https://cpwebassets.codepen.io/assets/embed/ei.js"></script>
{% endraw %}
