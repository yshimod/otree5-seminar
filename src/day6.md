{% raw %}
【第6回】 2022年6月16日


- 今後の予定
    - 第6回（今日）: 
        - 逐次手番ゲーム（参加者ごと表示させる画面を変える）
        - 繰り返しゲーム（プレイヤーのシャッフル，タイムアウト）
    - 第回: 質問紙調査（画面のデザイン，JavaScript，CSS，Bootstrap）
    - 第回: 連続時間ダブルオークション（ライブページとExtraModel）
    - 第回: 補遺


この勉強会を始める時点で，どのレベルの知識を知っているかという前提と，何ができるようになるかという目標を明確にしていなかったため，初心者向けを装いながら，これまでのところオタク向けの内容になっています．

[第4回](day4.md)で，どのようにコードを書いていくのかの手順を説明したつもりではあるのですが，やはり初心者には難解ではないかと思われます．
文字資料だけでは何をどうしたらよいか分からないという方は動画を見て勉強するのが良いでしょう．
たとえば，[Jonas Freyさん](https://sites.google.com/site/jonasfreysite/)によるoTreeチュートリアルの動画は有用です．
- [Part 1: introduction](https://www.youtube.com/watch?v=OzkFvVhoHr0&list=PLBL9eqPcwzGPli11Yighw5LWwzIifEFd_&index=1)
- [Part 2: Install oTree with Pycharm](https://www.youtube.com/watch?v=criiiSiEtUw&list=PLBL9eqPcwzGPli11Yighw5LWwzIifEFd_&index=2)
- [Part 3: Overview over oTree](https://www.youtube.com/watch?v=7f5HgW4vgvU&list=PLBL9eqPcwzGPli11Yighw5LWwzIifEFd_&index=3)
- [Part 4: A first Simple Game](https://www.youtube.com/watch?v=eozwgQ21Kg0&list=PLBL9eqPcwzGPli11Yighw5LWwzIifEFd_&index=4)
- [Part 5: Real Effort Task with Multiple Rounds](https://www.youtube.com/watch?v=js2hsUwOFR0&list=PLBL9eqPcwzGPli11Yighw5LWwzIifEFd_&index=5)
- [Part 6: Multiplayer Games - Prisoners Dilemma](https://www.youtube.com/watch?v=ewCDAk4DxWo&list=PLBL9eqPcwzGPli11Yighw5LWwzIifEFd_&index=6)
- [Part 7: Deploying oTree Experiments on Heroku](https://www.youtube.com/watch?v=VrPdBEghYEM&list=PLBL9eqPcwzGPli11Yighw5LWwzIifEFd_&index=7)

第4回では，ゼロからコードを書いていく方法を解説しています．
しかし初心者には，すでに完成しているコードについて，少しいじっては動作を確認し，という試行錯誤を繰り返しながら自分が作りたいものに変えていく方法で勉強するのが良いでしょう．

今回は完成したコードを読み下していきながら，特に oTree の組み込み関数の使い方を勉強していきます．



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
                - なぜなら，メソッド（ `.get_player_by_id()` ）を使った後に変数を呼び出すことができないため．
            - 先手のみに表示する部分であれば `{{ player.sent_amount }}` でよいが，後手には使えない．


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


### `*_max()` で入力フォーム検証の条件を動的に設定する

- 信頼ゲームで後手が行う意思決定（先手に返すポイント数: `sent_back_amount`）の上限は，先手が預けたポイント数（を何倍かしたもの）．
- 先手の意思決定によって，後手の意思決定の上限が変動する．
- モジュールレベルで `sent_back_amount_max()` を定義して，返り値を `sent_back_amount` の最大値とする．



## 有限回繰り返しゲーム（マッチングペニー）

- `otree startproject` コマンド実行で手に入るサンプルゲームの「matching_pennies」アプリ．
- ソースコード [https://github.com/oTree-org/oTree/tree/lite/matching_pennies](https://github.com/oTree-org/oTree/tree/lite/matching_pennies)
- デモページ [https://otree-demo.herokuapp.com/demo/matching_pennies](https://otree-demo.herokuapp.com/demo/matching_pennies)


### `creating_session()` を使ったプレイヤーのシャッフル

- モジュールレベルで組み込み関数 `creating_session()` を定義すると，subsession の最初のページが表示されるタイミングで group の編成を定義できる．
- 一つのアプリを何回も繰り返す設定にしている場合（`NUM_ROUNDS` が 1よりも大きい場合），繰り返す度に（つまり subsession ごとに） group 編成を変更することができる．
- subsession の途中で group 編成を変更することはできない．ただし，group 内での役割（`id_in_group`）は変更できる．
- `creating_session()` はセッションを作成するタイミングで実行されるので， `creating_session()` では意思決定に応じて group 編成を変更することはできない．
    - （待機ページクラスで `group_by_arrival_time = True` とした上で） `group_by_arrival_time_method()` を使えば，そこで柔軟に group 編成を定義できる．
- group の編成は subsession のメソッド `set_group_matrix()` で設定できる．引数に2次元配列で記述した新しい group 編成を渡す．
    - zTree とは異なり absolute stranger マッチングは実装されていないので，自分で実装する．
        - [https://www.sciencedirect.com/science/article/pii/S0165176516302324](https://www.sciencedirect.com/science/article/pii/S0165176516302324)
- [https://otree.readthedocs.io/en/latest/treatments.html](https://otree.readthedocs.io/en/latest/treatments.html)
- [https://otree.readthedocs.io/en/latest/multiplayer/groups.html#group-matching](https://otree.readthedocs.io/en/latest/multiplayer/groups.html#group-matching)


### 意思決定画面で時間制限を設定してみる

- ページクラスで変数 `timeout_seconds` に整数を定義すると，その `timeout_seconds` 秒の時間制限を設定できる．
- 動的に時間制限を設定するには，ページクラスの組み込みメソッド `get_timeout_seconds()` を定義し，返り値を残り秒数とする．
- デフォルトでは，画面に「このページでの残り時間 mm:ss」と表示される．
- タイムアウトするとページのフォームが自動送信される．
    - 送信されるデータは，（`initial` が設定されていなければ） `BooleanField` なら `False`， `IntegerField` や `FloatField` なら `0`， `StringField` なら `""`．
        - この値を変更したい場合は，ページクラスの組み込みメソッド `before_next_page()` の中で `timeout_happened == True` なる player に対し処理を行う．
    - `otree prodserver` でサーバーを起動している場合のみ，クライアントがブラウザーを閉じているときにタイムアウトが発生すると，クライアントの代わりに，サーバーが自分宛てにフォームを送信する．
- [https://otree.readthedocs.io/en/latest/timeouts.html](https://otree.readthedocs.io/en/latest/timeouts.html)



## 無限回繰り返しゲーム（繰り返しPD）

- [https://github.com/snunnari/otree_repeated_prisoner](https://github.com/snunnari/otree_repeated_prisoner)
- ただし，oTree のバージョンが古い．バージョン5の書き方に翻訳したものはこちら:  
    [https://github.com/iserExperiment/otree_repeated_prisoner/tree/yshimod/updating](https://github.com/iserExperiment/otree_repeated_prisoner/tree/yshimod/updating)．


### 確率的に繰り返しを終了するとき `NUM_ROUNDS` は最大数を設定する
- oTree はサーバーを立ち上げる際にデータベースの列を作成している．
- 一度データベースの枠を作っておいてから，使わずに空欄のままにしておく分にはどうとでもなる．
    - `NUM_ROUNDS = 100` としておくと，サーバーを立ち上げたときデータベースにラウンド数分の列を生成する．
    - subsession の途中で引き出した乱数の値によって条件分岐し，終了ラウンド以降を `is_displayed()` や `app_after_this_page()` を使ってページをスキップしてしまえば，確率的にラウンドの繰り返しを終わらせることができる．
        -  subsession の途中でも REST を使って外部から `session.vars` に値を渡すことができるので，ラボでサイコロを振って確率的に繰り返しの終了を決めることもできる． [https://otree.readthedocs.io/en/latest/misc/rest_api.html#session-vars-endpoint](https://otree.readthedocs.io/en/latest/misc/rest_api.html#session-vars-endpoint)
    - 途中でラウンド数を増やすことはできないため， `NUM_ROUNDS` で設定するラウンド数は大きくしないといけない．しかし， `NUM_ROUNDS` を増やすほどデータベースの列数は増え，処理に時間がかかりパフォーマンスは悪化する（？）．
- サーバーを立ち上げる際に（定数 `C` クラスにおいて）乱数を引き出して，確率的に決定されるラウンド数を予め決定してしまう，というのも手といえば手．


- 最大数が定義できない場合や，データベースの列数が増えることによるパフォーマンス低下を避ける場合，ライブページと ExtraModel を使って実装するのが良い．
    - [https://www.otreehub.com/projects/otree-more-demos/](https://www.otreehub.com/projects/otree-more-demos/) の「supergames_indefinite」．
        -  ソースコード [https://github.com/oTree-org/more-demos/tree/master/supergames_indefinite](https://github.com/oTree-org/more-demos/tree/master/supergames_indefinite)
        - デモページ [https://otree-more-demos.herokuapp.com/demo/supergames_indefinite](https://otree-more-demos.herokuapp.com/demo/supergames_indefinite)


{% endraw %}
