{% raw %}
【第6回】 2022年6月16日


- 逐次手番ゲーム（参加者ごと表示させる画面を変える）
- 有限回繰り返しゲーム（プレイヤーのシャッフル，タイムアウト）
- 無限回繰り返しゲーム（確率的に繰り返しを終了する）



# oTree の組み込み関数の使い方

- oTree の組み込み関数は [ `__init__.py` の書き方](otree_ref/init.md) で網羅しているつもりです．


- 組み込み関数の使い方については，公式ドキュメントで関数名を検索してください．

    - たとえば，[`is_displayed` で検索](https://otree.readthedocs.io/en/latest/search.html?q=is_displayed)．


- [oTree Hub の Example code](https://www.otreehub.com/code/) で（ブラウザの検索機能を使って）検索して使用例を見るのも有用です．



## 逐次手番ゲーム（信頼ゲーム）

- `otree startproject` コマンド実行で手に入るサンプルゲームの「trust」アプリ．

- [公式ドキュメントのチュートリアル「信頼ゲーム」](https://otree.readthedocs.io/ja/latest/tutorial/part3.html) を参照．

- ソースコード [https://github.com/oTree-org/oTree/tree/lite/trust](https://github.com/oTree-org/oTree/tree/lite/trust)

- デモページ [https://otree-demo.herokuapp.com/demo/trust](https://otree-demo.herokuapp.com/demo/trust)


### 意思決定データを Group に記録する

- 信頼ゲームでは役割（先手と後手）で行う意思決定が異なり，group でユニークな値である．

    - 先手は後手に預けるポイント数 `sent_amount` を決定する．

    - 後手は先手に返すポイント数 `sent_back_amount` を決定する．

- player の変数として定義するより，group の変数として定義した方が良い．

    - テンプレートで `sent_amount` を展開するとき...

        - group の変数であれば `{{ group.sent_amount }}`．

        - player の変数であれば `vars_for_template()` で `player.group.get_player_by_id(1).sent_amount` を何らかの変数として渡す必要がある．

            - テンプレートで直接 `{{ group.get_player_by_id(1).sent_amount }}` としたいところだが，エラーとなる．

            - 先手のみに表示する部分であれば `{{ player.sent_amount }}` でよいが，後手には使えない．


<p class="ytubevideo"><iframe width="560" height="315" src="https://www.youtube.com/embed/sOhoXLiNCQg?rel=0&enablejsapi=1&origin=https://yshimod.github.io/" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe></p>

- あえて信頼ゲームの意思決定データを Player に保存するにはどうすれば良いか，と質問を受けました．

    - たとえば `sent_amount` と `sent_back_amount` を `Player` クラスで定義する場合， `__init__.py` はたとえば以下のように書く．

    %accordion%`__init__.py`%accordion%
    ```python
    from otree.api import *

    class C(BaseConstants):
        NAME_IN_URL = 'trust'
        PLAYERS_PER_GROUP = 2
        NUM_ROUNDS = 1
        INSTRUCTIONS_TEMPLATE = 'trust/instructions.html'
        ENDOWMENT = cu(100)
        MULTIPLIER = 3

    class Subsession(BaseSubsession):
        pass

    class Group(BaseGroup):
        pass

    class Player(BasePlayer):
        sent_amount = models.CurrencyField(
            min=0,
            max=C.ENDOWMENT,
            doc="""Amount sent by P1""",
            label="Please enter an amount from 0 to 100:"
        )
        sent_back_amount = models.CurrencyField(
            min=cu(0),
            doc="""Amount sent back by P2"""
        )

    def sent_back_amount_max(player: Player):
        """
        引数が group ではなく player であることに注意！
        """
        group: Group = player.group    ## ← 「: Group」 は消しても良い
        return group.get_player_by_id(1).sent_amount * C.MULTIPLIER

    def set_payoffs(group: Group):
        p1: Player = group.get_player_by_id(1)    ## ← 「: Player」 は消しても良い
        p2: Player = group.get_player_by_id(2)    ## ← 「: Player」 は消しても良い
        p1.payoff = C.ENDOWMENT - p1.sent_amount + p2.sent_back_amount
        p2.payoff = p1.sent_amount * C.MULTIPLIER - p2.sent_back_amount

    class Introduction(Page):
        pass

    class Send(Page):
        form_model = 'player'
        form_fields = ['sent_amount']

        @staticmethod
        def is_displayed(player: Player):
            return player.id_in_group == 1

    class SendBackWaitPage(WaitPage):
        pass

    class SendBack(Page):
        form_model = 'player'
        form_fields = ['sent_back_amount']

        @staticmethod
        def is_displayed(player: Player):
            return player.id_in_group == 2

        @staticmethod
        def vars_for_template(player: Player):
            group: Group = player.group
            p1: Player = group.get_player_by_id(1)

            tmp_sent_amount = p1.sent_amount

            return dict(
                tmp_sent_amount = tmp_sent_amount,    ## ← {{ group.sent_amount }} を {{ tmp_sent_amount }} に置換せよ
                tripled_amount = tmp_sent_amount * C.MULTIPLIER
            )

    class ResultsWaitPage(WaitPage):
        after_all_players_arrive = set_payoffs

    class Results(Page):
        @staticmethod
        def vars_for_template(player: Player):
            group: Group = player.group
            p1: Player = group.get_player_by_id(1)
            p2: Player = group.get_player_by_id(2)
            ## ↑ group を定義せず，いちいち
            ## p1 = player.group.get_player_by_id(1)
            ## p2 = player.group.get_player_by_id(2)
            ## とすると，パフォーマンスが低下する．
            ## https://otree.readthedocs.io/en/latest/misc/tips_and_tricks.html#improving-code-performance

            tmp_sent_amount = p1.sent_amount
            tmp_sent_back_amount = p2.sent_back_amount

            return dict(
                tmp_sent_amount = tmp_sent_amount,    ## ← {{ group.sent_amount }} を {{ tmp_sent_amount }} に置換せよ
                tmp_sent_back_amount = tmp_sent_back_amount,    ## ← {{ group.sent_back_amount }} を {{ tmp_sent_back_amount }} に置換せよ
                tripled_amount = tmp_sent_amount * C.MULTIPLIER
            )

    page_sequence = [
        Introduction,
        Send,
        SendBackWaitPage,
        SendBack,
        ResultsWaitPage,
        Results,
    ]
    ```
    %/accordion%

    - テンプレートでは，特定のプレイヤーの変数を直接展開することができないため， `vars_for_template()` を使ってテンプレートに変数を渡す必要がある．この手間がデメリットといえばデメリット．

    - `sent_amount` と `sent_back_amount` を `Player` クラスで定義する場合，出力されるCSVファイルの1行（あるプレイヤーのデータ）には，先手プレイヤーの場合に `sent_amount` 列は数値が入っているが， `sent_asent_back_amountmount` 列は空欄（ `None` ）となっている．後手プレイヤーの場合には `sent_asent_back_amountmount` 列は数値が入っているが， `sent_amount` 列は空欄となっている．

        - 実験実施だけに注目すれば（変数を呼び出すのが少々面倒くさいだけで）問題ないが，実験データの分析のことを考慮すると，1行に当該プレイヤーの意思決定と相手プレイヤーの意思決定の両方が入っている方が便利かも．

    - `Player` クラスの変数として入力フォームを実装し， oTree サーバー側で（たとえば `before_next_page()` を使って） `Group` クラスの変数に "転記" する方法も一案．

    %accordion%`__init__.py`%accordion%
    ```python
    from otree.api import *

    class C(BaseConstants):
        NAME_IN_URL = 'trust'
        PLAYERS_PER_GROUP = 2
        NUM_ROUNDS = 1
        INSTRUCTIONS_TEMPLATE = 'trust/instructions.html'
        ENDOWMENT = cu(100)
        MULTIPLIER = 3

    class Subsession(BaseSubsession):
        pass

    class Group(BaseGroup):
        sent_amount = models.CurrencyField()
        sent_back_amount = models.CurrencyField()

    class Player(BasePlayer):
        p_sent_amount = models.CurrencyField(
            min=0,
            max=C.ENDOWMENT,
            doc="""Amount sent by P1""",
            label="Please enter an amount from 0 to 100:"
        )
        p_sent_back_amount = models.CurrencyField(
            min=cu(0),
            doc="""Amount sent back by P2"""
        )

    def p_sent_back_amount_max(player: Player):
        """
        引数が group ではなく player であることに注意！
        """
        group: Group = player.group    ## ← 「: Group」 は消しても良い
        return group.sent_amount * C.MULTIPLIER

    def set_payoffs(group: Group):
        p1: Player = group.get_player_by_id(1)    ## ← 「: Player」 は消しても良い
        p2: Player = group.get_player_by_id(2)    ## ← 「: Player」 は消しても良い
        p1.payoff = C.ENDOWMENT - group.sent_amount + group.sent_back_amount
        p2.payoff = group.sent_amount * C.MULTIPLIER - group.sent_back_amount

    class Introduction(Page):
        pass

    class Send(Page):
        form_model = 'player'
        form_fields = ['p_sent_amount']

        @staticmethod
        def is_displayed(player: Player):
            return player.id_in_group == 1

        @staticmethod
        def before_next_page(player: Player, timeout_happened):
            group: Group = player.group

            if player.id_in_group == 1:
                """
                is_displayed() によって id_in_group != 1 なるプレイヤーはスキップされ，
                後手プレイヤーにはこの before_next_page() は実行されない．
                したがって，このif文は必ず True で通るはずである．
                （必ず True になるのだったら，if文を噛まさなくてもいいかも．）
                """
                group.sent_amount = player.p_sent_amount

    class SendBackWaitPage(WaitPage):
        pass

    class SendBack(Page):
        form_model = 'player'
        form_fields = ['p_sent_back_amount']

        @staticmethod
        def is_displayed(player: Player):
            return player.id_in_group == 2

        @staticmethod
        def vars_for_template(player: Player):
            group: Group = player.group

            return dict(
                tripled_amount = group.sent_amount * C.MULTIPLIER
            )

        @staticmethod
        def before_next_page(player: Player, timeout_happened):
            group: Group = player.group

            if player.id_in_group == 2:
                """
                is_displayed() によって id_in_group != 2 なるプレイヤーはスキップされ，
                先手プレイヤーにはこの before_next_page() は実行されない．
                """
                group.sent_back_amount = player.p_sent_back_amount

    class ResultsWaitPage(WaitPage):
        after_all_players_arrive = set_payoffs

    class Results(Page):
        @staticmethod
        def vars_for_template(player: Player):
            group: Group = player.group

            return dict(
                tripled_amount = group.sent_amount * C.MULTIPLIER
            )

    page_sequence = [
        Introduction,
        Send,
        SendBackWaitPage,
        SendBack,
        ResultsWaitPage,
        Results,
    ]
    ```
    %/accordion%


### ページ表示をスキップする

- ページクラスの組み込みメソッド `is_displayed()` を定義すれば，返り値で渡す真偽値でページを表示するか否かを設定できる．

- ラウンド数（ `player.round_number` ）やプレイヤーの役割（ `player.id_in_group` ）で条件分岐させることが多い．

    - 複数ラウンドを設定していて，アプリ内にインストラクションページが含まれるとき，インストラクションは最初の1回だけ表示するには， `is_displayed()` で `player.round_number == 1` を返せばよい．

- [https://otree.readthedocs.io/en/latest/pages.html#is-displayed](https://otree.readthedocs.io/en/latest/pages.html#is-displayed)


- ページクラスの組み込みメソッド `app_after_this_page()` で返り値をアプリ名とすれば，返り値のアプリまでスキップされる．

- ページクラスの組み込みメソッド `get_timeout_seconds()` で返り値を `0` とすれば，0秒で自動的にページが遷移するため，ページをスキップさせる手段として使えなくもない．ただし一瞬はページが表示されることに注意．



#### 逐次手番ゲームでの `is_displayed()` の使い方

- 信頼ゲームでは役割（先手と後手）で行う意思決定が異なる．それぞれを別のページとして実装する．

    - 先手が後手に預けるポイント数を入力するページとして `Send`．

    - 後手が，先手から受け取ったポイント数のうち先手に返すポイント数を入力するページとして `SendBack`．

    - ゲームの結果を表示するページとして `Results`．

- `is_displayed()` を使って，`Send` ページを先手だけに，`SendBack` ページを後手だけに表示する．

    - `is_displayed()` の引数は `player` オブジェクト．返り値が `True` のとき，その player に表示される．

    - group 内の役割は `player.id_in_group` の値で定義する（定数 `C` クラスで `*_ROLE` を定義して役割のラベルを設定しても良い）．

        - `player.id_in_group == 1` なる player を先手，`player.id_in_group == 2` なる player を後手とする．

        - `Send` クラスにおいて， `is_displayed()` を定義し，返り値を `player.id_in_group == 1` （の真偽値）とする．

        - `SendBack` クラスにおいて， `is_displayed()` を定義し，返り値を `player.id_in_group == 2` （の真偽値）とする．

- ページ順を `page_sequence = [Send, SendBack, Results]` と設定しているとき...

    - 先手が `Send` ページで意思決定している間，後手には `Send` ページが表示されず，次の `SendBack` ページが表示されてしまう．`SendBack` ページにおいて `Send` ページにおける先手の意思決定の結果を表示させるような実装をしている場合，意図しない挙動となるかエラーが出る．

    - 後手が `SendBack` ページで意思決定している間，先手には `Results` ページが表示されてしまう．後手の意思決定が終わっていなければ，当然利得の計算もまだ行われていないため， `Results` ページで利得を表示することはできない．

- 一方のプレイヤーが意思決定している間，他方のプレイヤーは待機ページで待機する必要がある．

    - 先手が `Send` ページで意思決定している間，後手には `SendBackWaitPage` なる待機ページを表示する．先手の意思決定が終わったタイミングで特に行うべき処理は無いため， `SendBackWaitPage` クラスは定義するだけで中身は `pass` とだけ書いておけば良い．

    - 後手が `SendBack` ページで意思決定している間，先手には `ResultsWaitPage` なる待機ページを表示する．後手の意思決定が終わったタイミングで利得を計算するため，`after_all_players_arrive` を定義する．


<p class="ytubevideo"><iframe width="560" height="315" src="https://www.youtube.com/embed/AiWlUOnbZvI?rel=0&enablejsapi=1&origin=https://yshimod.github.io/" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe></p>



### `*_max()` で入力フォーム検証の条件を動的に設定する

- 信頼ゲームで後手が行う意思決定（先手に返すポイント数: `sent_back_amount`）の上限は，先手が預けたポイント数（を何倍かしたもの）．

- 先手の意思決定によって，後手の意思決定の上限が変動する．

- モジュールレベルで `sent_back_amount_max()` を定義して，返り値を `sent_back_amount` の最大値とする．

    - [https://otree.readthedocs.io/en/latest/forms.html#field-name-max](https://otree.readthedocs.io/en/latest/forms.html#field-name-max)


<p class="ytubevideo"><iframe width="560" height="315" src="https://www.youtube.com/embed/3smlOBGiRV8?rel=0&enablejsapi=1&origin=https://yshimod.github.io/" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe></p>




## 有限回繰り返しゲーム（マッチングペニー）

- `otree startproject` コマンド実行で手に入るサンプルゲームの「matching_pennies」アプリ．

- ソースコード [https://github.com/oTree-org/oTree/tree/lite/matching_pennies](https://github.com/oTree-org/oTree/tree/lite/matching_pennies)

- デモページ [https://otree-demo.herokuapp.com/demo/matching_pennies](https://otree-demo.herokuapp.com/demo/matching_pennies)



### `creating_session()` を使ったプレイヤーのシャッフル

- モジュールレベルで組み込み関数 `creating_session()` を定義すると，subsession の最初のページが表示されるタイミングで group の編成を定義できる．

- 一つのアプリを何回も繰り返す設定にしている場合（`NUM_ROUNDS` が 1よりも大きい場合），繰り返す度に（つまり subsession ごとに） group 編成を変更することができる．

- subsession の途中で group 編成を変更することはできない．

    - ただし， subsession の途中でも， group 内での役割（`id_in_group`）は group の `set_players()` メソッドで変更できる．

- `creating_session()` はセッションを作成するタイミングで実行されるので， `creating_session()` では意思決定に応じて group 編成を変更することはできない．

    - （待機ページクラスで `group_by_arrival_time = True` とした上で） `group_by_arrival_time_method()` を使えば，そこで柔軟に group 編成を定義できる．

- group の編成は subsession のメソッド `set_group_matrix()` で設定できる．引数に2次元配列で記述した新しい group 編成を渡す．

    - z-Tree とは異なり absolute stranger マッチングは実装されていないので，自分で実装する．

        - [https://www.sciencedirect.com/science/article/pii/S0165176516302324](https://www.sciencedirect.com/science/article/pii/S0165176516302324)

- [https://otree.readthedocs.io/en/latest/treatments.html](https://otree.readthedocs.io/en/latest/treatments.html)

- [https://otree.readthedocs.io/en/latest/multiplayer/groups.html#group-matching](https://otree.readthedocs.io/en/latest/multiplayer/groups.html#group-matching)


<p class="ytubevideo"><iframe width="560" height="315" src="https://www.youtube.com/embed/Bsrq7v5Lz90?rel=0&enablejsapi=1&origin=https://yshimod.github.io/" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe></p>


- （Python 標準機能としての）リストの操作は以下を参照．  
    [https://docs.python.org/3.9/library/stdtypes.html#mutable-sequence-types](https://docs.python.org/3.9/library/stdtypes.html#mutable-sequence-types)

    - 作例中で使っている `.reverse()` メソッドはリスト自体を破壊的に変更（逆転）する．`.reverse()` 自体の返り値は `None` であることに注意．

    - （雑談） R と Python の挙動の違いに注意！

        - R ではデータをいじると，勝手にコピーしたものを変えてくれる．

        ```r
        a <- c(0:4)
        print(a)
        # [1] 0 1 2 3 4

        b <- a
        b[3] <- 999
        print(b)
        # [1]   0   1 999   3   4

         print(a)
        # [1] 0 1 2 3 4
        ```

        - 一方で，Pythonはコピーせず，もとのデータも変えてくれる．

        ```python
        a = list(range(5))
        print(a)
        # [0, 1, 2, 3, 4]

        b = a
        b[2] = 999
        print(b)
        # [0, 1, 999, 3, 4]

        print(a)
        # [0, 1, 999, 3, 4]
        ```


#### 【Tips】 関数内でインポートしてよいか？

- oTree の公式ドキュメントでは，パッケージやモジュールをインポートするとき，関数定義の中で `import` していることが多い．

    - たとえば [https://otree.readthedocs.io/en/latest/treatments.html?highlight=import](https://otree.readthedocs.io/en/latest/treatments.html?highlight=import)

- PEP (Python Enhancement Proposal) では以下のように説明されている．

    > Imports are always put at the top of the file, just after any module comments and docstrings, and before module globals and constants.

    - [https://peps.python.org/pep-0008/#imports](https://peps.python.org/pep-0008/#imports)

    - 原則は `.py` ファイルの冒頭で `import` しなければならない．

- ただし，`otree startproject` で入るサンプルゲームでの実装から分かるように，関数定義の中で `import` しても（ただし関数の中でのみ）モジュールの関数が使える．

- 名前が衝突してしまう場合には，やむを得ず関数定義の中で `import` して衝突を回避する．

- 冒頭で `import` すると，具体的にどの箇所でモジュールの関数を使っているのかがわかりにくくなってしまう．だから，モジュールの関数を使いたい場所の直前で `import` したい．

    - そのような動機の場合，自分でモジュールを作り（つまり，関数定義を別の `.py` ファイルに記述し），その中で `import` すれば良い．



### 意思決定画面で時間制限を設定してみる

- ページクラスで変数 `timeout_seconds` に整数を定義すると，その `timeout_seconds` 秒の時間制限を設定できる．

- 動的に時間制限を設定するには，ページクラスの組み込みメソッド `get_timeout_seconds()` を定義し，返り値を残り秒数とする．

- デフォルトでは，画面に「このページでの残り時間 mm:ss」と表示される．

- タイムアウトするとページのフォームが自動送信される．

    - どんな値が自動送信されても，エラーが表示されることなく次のページへ遷移する．

    - 入力フォームに数値などが途中まで入力してあって放置されている状態のとき，タイムアウトの時点で入力されていた値が送信される．

    - 値として適切である場合のみ（たとえば `models.FloatField` としてある入力フォームに文字列ではなく数値が入力されている場合など）ちゃんと記録される．

    - 入力フォームが空の場合，あるいは値として不適切な場合，記録されるデータは，（`initial` が設定されていなければ） `BooleanField` なら `False`， `IntegerField` や `FloatField` なら `0`， `StringField` なら `""`．

    - 自動送信されて記録された値を変更したい場合は，ページクラスの組み込みメソッド `before_next_page()` の中で `timeout_happened == True` なる player に対し処理を行う．

    - `otree prodserver` でサーバーを起動している場合のみ，クライアントがブラウザーを閉じているときにタイムアウトが発生すると，クライアントの代わりに，サーバーが自分宛てにフォームを送信する．

- 「Advance slowest user(s)」ボタンで次のページへ遷移させた場合も，当該ページでタイムアウトしたのと同じ処理が行われる．

    - ただし，サーバーが自分宛てにフォームを送信するので，クライアントで入力フォームに何か入力されていても，その値を取得することはできない．

- [https://otree.readthedocs.io/en/latest/timeouts.html](https://otree.readthedocs.io/en/latest/timeouts.html)


<p class="ytubevideo"><iframe width="560" height="315" src="https://www.youtube.com/embed/Of6yiqbz97Q?rel=0&enablejsapi=1&origin=https://yshimod.github.io/" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe></p>


#### 【Tips】 「 `@staticmethod` 」は何か？

- 関数デコレータで，すぐ下で定義している関数をスタティックメソッドに変換するもの．

    - [https://docs.python.org/3.9/library/functions.html#staticmethod](https://docs.python.org/3.9/library/functions.html#staticmethod)

- 純粋な Python の機能の説明...

    - 用語:

        - 「メソッド」: クラスの中で定義した関数．

        - 「インスタンス（オブジェクト）」: （操作的な定義） `Kurasu` という名前のクラスが定義してあるとき， `Kurasu()` はインスタンスオブジェクト．

            - oTree でよく出てくる `player` や `group` はインスタンスオブジェクトが代入されたもの．つまり `player = Player()` ， `group = Group()` ．

    - たとえば，以下のようなクラスが定義してあるとする．

    ```python
    class Kurasu:
        mytext = "こんにちは"

        def testfunc1(self, nanika):
            print(self.mytext)
            print(nanika)
    ```

    - 関数 `testfunc1()` を `Kurasu` クラスのインスタンス（ `Kurasu()` ）のメソッドとして呼び出すとき，第1引数に自身のインスタンスオブジェクト（ `self = Kurasu()` ）を受け取った状態の関数になっており，第2引数の `nanika` だけ渡せば良い．

        - たとえば，

        ```python
        kurasu = Kurasu()
        kurasu.testfunc1("あいうえお")
        # こんにちは
        # あいうえお
        ```

        とすると，「こんにちは」と「あいうえお」が出力される．
        `testfunc1()` の引数に，自分では `self` として何のオブジェクトも渡しておらず， `nanika` だけしか渡していないのに，ちゃんと動いていることに注目されたい．

    - 関数 `testfunc1()` をクラスから直接呼び出すとき， `testfunc1()` の定義通り，ちゃんと第1引数に `self` として自身のインスタンスオブジェクト，第2引数に `nanika` を渡さなければならない．

        - たとえば，先ほどと同様，引数 `nanika` だけしか渡さない場合，

        ```python
        Kurasu.testfunc1("あいうえお")
        # TypeError: testfunc1() missing 1 required positional argument: 'nanika'
        ```

        とエラーが出る． `nanika` に渡したつもりの文字列が `self` に取られ，もう一つの引数が足らない状態である．

        - ちゃんと引数としてインスタンスオブジェクトと `nanika` の2つを渡してやると上手くいく．

        ```python
        kurasu = Kurasu()
        Kurasu.testfunc1(kurasu, "あいうえお")
        # こんにちは
        # あいうえお
        ```

    - `@staticmethod` がついた関数 `testfunc2()` で `Kurasu` クラスのインスタンス（ `Kurasu()` ）のメソッドとして呼び出すとき，関数の定義の通り，引数は `nanika` だけで渡せば良い．

    ```python
    class Kurasu:
        mytext = "こんにちは"
        
        @staticmethod
        def testfunc2(nanika):
            print(nanika)

    kurasu = Kurasu()
    kurasu.testfunc2("かきくけこ")
    # かきくけこ
    ```

    とすると，「かきくけこ」が出力される．
    なお，関数の定義で `self` を引数に取っていないので， `testfunc2()` の中では `self` を使えない（ `self.mytext` で値を受け取れない）．

    - 関数 `testfunc2()` の定義の直前に書いてある `@staticmethod` を消した上で， `testfunc2()` をインスタンスメソッドとして呼び出すと，エラーが出る．

    ```python
    class Kurasu:
        def testfunc2(nanika):
            print(nanika)

    kurasu = Kurasu()
    kurasu.testfunc2("かきくけこ")
    # TypeError: testfunc2() takes 1 positional argument but 2 were given
    ```

    曰く「`testfunc2()` は引数を1つ（ `nanika` ）しか取らない関数のはずなのに，2個渡されましたけど，どういうこと？」と．
    この場合， `testfunc2()` がインスタンスメソッドとして呼び出されているので，第1変数として暗黙のうちに `Kurasu()` が引数として渡されている．
    しかし， `testfunc2()` を定義するときに `nanika` しか受け取らない，ということにしていたので，エラーが出た．

    - 関数 `testfunc2()` をクラスから直接呼び出すとき， `@staticmethod` がついているか否かによらず，（ `Kurasu.testfunc1()` の場合と同様に） `testfunc2()` の定義で記述した引数をすべて渡さなければならない．

        - `@staticmethod` がついている場合．

        ```python
        class Kurasu:
            @staticmethod
            def testfunc2(nanika):
                print(nanika)

        Kurasu.testfunc2("かきくけこ")
        # かきくけこ
        ```

        - `@staticmethod` がついていない場合．

        ```python
        class Kurasu:
            def testfunc2(nanika):
                print(nanika)

        Kurasu.testfunc2("かきくけこ")
        # かきくけこ
        ```

    - 関数 `testfunc2()` をクラスから直接呼び出すとき， `@staticmethod` がついているか否かによらず，インスタンスメソッドとして呼び出したとき（ `kurasu.testfunc2()` のとき）と同じ挙動（引数 `nanika` だけ渡せば良い）となる．

- **クラスの中で定義する関数で，自身のインスタンスオブジェクト（ `self` ）を第1引数として受け取りたくない場合， `@staticmethod` をつけなければならない**．

    - oTree で，たとえばページクラスの中で `is_displayed()` を定義するとき，ページクラス自身のインスタンスオブジェクトは不要であり，別クラスである `Player` の インスタンスオブジェクト `player` を引数に取る（ oTree 本体は `is_displayed()` なる関数が `player` のみを引数に取る，ということを前提としてコードが書かれている）．

    - oTree 5 で組み込み関数（メソッド）はもはや `self` を引数にとる必要がなくなったため， `@staticmethod` をつけるべきである．

- **ところが，実のところ oTree は，ページクラスで組み込みのメソッドを定義する際に `@staticmethod` をつけなくても，エラーは出ず普通に動いてしまう．なぜ？**

    - oTree 本体は，たとえば `MyPage` クラスで定義した `is_displayed()` を呼び出すとき，

    ```python
    getattr(MyPage, "is_displayed")(Player())
    ```

    として呼び出している．これは，

    ```python
    MyPage.is_displayed(Player())
    ```

    と等価である．
    つまり， oTree の本体は `is_displayed()` をインスタンスのメソッドとしてではなく，クラスから直接呼び出していて，引数は `player = Player()` のみを渡している．

    - ページクラスの中で `@staticmethod` をつけて定義した関数は，クラスから直接呼び出されても，あるいは仮にインスタンスのメソッドとして呼び出されたとしても， `player` オブジェクトだけを受け取って想定した挙動をしてくれる．

    - ページクラスの中で `@staticmethod` をつけずに定義した関数をクラスから直接呼び出すとき，通常は第1引数に当該クラスのインスタンスオブジェクト（ `self = MyPage()` ）を渡さないといけないが， oTree 本体は有無を言わさず `player = Player()` を渡す．引数に `self` を受け取って処理を行うような関数の定義をしていればエラーとなるが，公式ドキュメントの指示通り （ `self` ではなく） `player` オブジェクトを受け取るように定義していれば，想定した挙動をしてくれる．

- **つまり，関数を呼び出すときに `@staticmethod` の有無で挙動が変わるような呼び出し方をしていないため， `@staticmethod` をつけなくても，エラーが出ずに動く**．

- では， `@staticmethod` はつけなくても良いか？

    - 公式ドキュメントではつけなくても良いと言っている．  
    [https://otree.readthedocs.io/en/latest/install-nostudio.html#about-staticmethod-etc](https://otree.readthedocs.io/en/latest/install-nostudio.html#about-staticmethod-etc)

    - Python 学習者は，クラスで `self` を第1引数で受け取らない関数には `@staticmethod` をつけるクセをつけたほうが良い．

    - 動けば良い，ではトラブルシューティングで苦労する．

- oTree 3 では，ページクラス自身のインスタンスオブジェクト（ `self` ）を受け取り，そこから `player` オブジェクトを取り出していた．

    - oTree 5 へのアップデートで，組み込みの関数において実験データにアクセスする際にいちいち `self` から取り出す必要がなくなったことは（利点かどうかは別として）大きな特徴である．oTree の著者も， v5 のスタイルを「 no-self format 」と呼んでいる．

- なお，oTree 3 のような `self` を引数に取るスタイルでの関数定義は oTree 5 でも可能といえば可能だが，スタイルを完全に oTree 3 のスタイルにしなければならない．重要な点は `__init__.py` にデータモデルのクラスとページクラスの両方を定義するのではなく， `models.py` と `pages.py` に分離して定義しなければならないところである．実のところ，「 no-self format 」か否かの判定は `__init__.py` に `import` の文字列が含まれているかどうかで，含まれていれば「 no-self format 」として処理をする． `@staticmethod` がついているかどうか，とか，関数の引数が `self` か `player` か，とかはまったく関係ない．引数に `self` と書いてあっても，「 no-self format 」の場合は容赦なく `player` オブジェクトを渡してくる．


## 無限回繰り返しゲーム（繰り返しPD）

- [https://github.com/snunnari/otree_repeated_prisoner](https://github.com/snunnari/otree_repeated_prisoner)

- ただし， oTree のバージョンが古い．バージョン5の書き方に翻訳したものはこちら:  
[https://github.com/iserExperiment/otree_repeated_prisoner](https://github.com/iserExperiment/otree_repeated_prisoner)


### 確率的に繰り返しを終了するとき `NUM_ROUNDS` は最大数を設定する

- oTree はサーバーを立ち上げる際にデータベースの列を作成している．

- 一度データベースの枠を多めに作っておいてから，使わずに空欄のままにしておく分にはどうとでもなる．

    - `NUM_ROUNDS = 100` としておくと，サーバーを立ち上げたときデータベースにラウンド数分の列を生成する．

    - subsession の途中で引き出した乱数の値によって条件分岐し，終了ラウンド以降を `is_displayed()` や `app_after_this_page()` を使ってページをスキップしてしまえば，確率的にラウンドの繰り返しを終わらせることができる．

        -  subsession の途中でも REST を使って外部から `session.vars` に値を渡すことができるので，ラボでサイコロを振って確率的に繰り返しの終了を決めることもできる．  
        [https://otree.readthedocs.io/en/latest/misc/rest_api.html#session-vars-endpoint](https://otree.readthedocs.io/en/latest/misc/rest_api.html#session-vars-endpoint)

    - 途中でラウンド数を増やすことはできないため， `NUM_ROUNDS` で設定するラウンド数は大きくしないといけない．しかし， `NUM_ROUNDS` を増やすほどデータベースの列数は増え，処理に時間がかかりパフォーマンスは悪化する（？）．

- サーバーを立ち上げる際に（定数 `C` クラスにおいて）乱数を引き出して，確率的に決定されるラウンド数を予め決定してしまう，というのも手．


- 最大数が定義できない場合や，データベースの列数が増えることによるパフォーマンス低下を避ける場合，ライブページと ExtraModel を使って実装するのが良い．

    - [https://www.otreehub.com/projects/otree-more-demos/](https://www.otreehub.com/projects/otree-more-demos/) の「supergames_indefinite」アプリ．

        -  ソースコード [https://github.com/oTree-org/more-demos/tree/master/supergames_indefinite](https://github.com/oTree-org/more-demos/tree/master/supergames_indefinite)

        - デモページ [https://otree-more-demos.herokuapp.com/demo/supergames_indefinite](https://otree-more-demos.herokuapp.com/demo/supergames_indefinite)


<p class="ytubevideo"><iframe width="560" height="315" src="https://www.youtube.com/embed/NZfKz8p9iF4?rel=0&enablejsapi=1&origin=https://yshimod.github.io/" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe></p>


- この作例のポイントは...

    - oTree サーバーを立ち上げた時点で乱数を引き，幾何分布から一つ一つのスーパーゲームの長さ（ステージゲームの回数）を決めて，定数としている点．

        - `C` クラスの中で乱数を引いた場合，サーバーを再起動しない限り，いくらセッションを作り直しても同じスーパーゲームの長さとなる．

        - スーパーゲームの長さは定数で定義されているため， group によってスーパーゲームの長さを変えたい場合には違う実装を考えなければならない．

        - Heroku を使う場合，自動的に再起動されるので（乱数のシードを設定していない限り）意図しないタイミングでスーパーゲームの長さが変わってしまうことに注意．

    - スーパーゲームの回数は最大数を定数で定義して，指定時間を超えた以降のスーパーゲームは表示させない点．

        - 指定時間が来る前に設定した最大回数が終わってしまった場合，追加ラウンドを行うことはできない（急いでサーバーを立ち上げ直した後セッションをもう一度始める，で良いならそれでいいが）．

        - この作例では，指定時間を超えた場合にのみ表示する最終ページ（ `End` ）を用意し，そのページに「次へ」ボタンを置かないことにより，そこで終了（したことに）している．

            - あくまでラウンド繰り返しの途中で「次へ」進めなくしているだけなので，うっかり「Advance slowest user(s)」ボタンを押してしまうと，続きの新たなラウンドが始まってしまう．

        - 次のアプリに進める必要がある場合（たとえば繰り返しゲーム課題の後に質問紙調査がある場合），ちゃんと `is_displayed()` や `app_after_this_page()` を使ってページをスキップさせる必要がある．

            - 指定時間を超えた場合にのみ表示する最終ページで `app_after_this_page()` を定義すれば，以降の余分なラウンドを一気にスキップできる．




{% endraw %}
