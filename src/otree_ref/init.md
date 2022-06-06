# `__init__.py` の書き方


## ファイルの冒頭
- シバン（shebang... `#!/usr/bin/env python3` みたいなやつ）は不要．
- 文字コード宣言（ `# -*- coding: utf-8 -*-` みたいなやつ）は不要（むしろ非推奨）．
- `otree.api` のインポート
    ```python
    from otree.api import *
    ```
    - oTree3ではインポートするものを細かく指定していた．
- 他にも使いたいモジュール（`time`，`random`，`json`，`numpy`など）があれば，冒頭でインポートしておく．


## 定数: クラス `C`
[https://otree.readthedocs.io/en/latest/models.html#constants](https://otree.readthedocs.io/en/latest/models.html#constants)


- アプリ内で参照する変数（定数）を定義する．
- クラス `C` で設定するものはCSVデータ出力には含まれない．CSVデータに記録しなくても平気な定数を定義する．
    - セッションごと（トリートメントごと）変化しうる変数はクラス `C` ではなく `settings.py` の `SESSION_CONFIGS` の中で定義するべき． `SESSION_CONFIGS` で定義した変数（ `session.変数` ）はCSVデータ出力に含まれる．
- 公式ドキュメントには，dict型の変数はメソッドで定義せよ，とあるが，dict型の変数も普通に定義できそう．
- マナーとして変数名は大文字にしておくと良い？
- oTree3では `Constants` なるクラスで設定していた．

#### `NAME_IN_URL`
- URLにアプリ名として表示するものを設定．
- 必ず設定しなければならないが，アプリ名（ `__name__` で取得できる）と異なっていても良い．

#### `PLAYERS_PER_GROUP`
- ゲーム実験の場合，各グループの人数を設定する．
- 必ず設定する．
- プレーヤーの相互作用が無い場合（アンケートなど）は `1` ではなく `None` とする．
- セッションを作成するとき，人数は `PLAYERS_PER_GROUP` の整数倍でなければならない．
- グループ内でプレイヤーに番号（ `player.id_in_group` ）が振られる．この番号はグループにおける役割のインデックスとして使うと良い．

#### `NUM_ROUNDS`
- アプリを繰り返す場合，繰り返す（最大）回数を設定する．
- 1以上の整数を必ず設定する．

#### `*_ROLE`
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

#### クラス `Subsession`
[https://otree.readthedocs.io/en/latest/models.html#subsession](https://otree.readthedocs.io/en/latest/models.html#subsession)


- round_number
- get_groups()
- get_players()
- group_randomly()
- group_like_round()
- get_group_matrix()
- set_group_matrix()
- in_all_rounds()
- in_previous_rounds()
- in_rounds(first, last)
- in_round(round_number)


##### アクセスできる上層のデータ
- session


#### クラス `Group`
[https://otree.readthedocs.io/en/latest/models.html#group](https://otree.readthedocs.io/en/latest/models.html#group)


- round_number
- in_all_rounds()
- in_previous_rounds()
- in_rounds(first, last)
- in_round(round_number)
- get_player_by_role(role)
- get_player_by_id(id_in_group)
- get_players()
- set_players()


##### アクセスできる上層のデータ
- subsession
- session



#### クラス `Player`
[https://otree.readthedocs.io/en/latest/models.html#player](https://otree.readthedocs.io/en/latest/models.html#player)


- id_in_group
- payoff
- round_number
- in_all_rounds()
- in_previous_rounds()
- in_rounds(first, last)
- in_round(round_number)
- get_others_in_subsession()
- get_others_in_group()


##### アクセスできる上層のデータ
- participant
- group
- subsession
- session


#### フィールドの型
- `models.BooleanField` (for true/false and yes/no values)
- `models.CurrencyField` for currency amounts; see Currency.
- `models.IntegerField`
- `models.FloatField` (for real numbers)
- `models.StringField` (for text strings)
- `models.LongStringField` (for long text strings; its form widget is a multi-line textarea)

#### 入力フォームとの対応
1. データを参加者に入力させたいページのクラスにおいて `form_model` と `form_fields` を設定する．
1. フィールド名（記憶するデータの変数名）を，HTMLテンプレートの `<input>` タグの `name` 属性に設定する．
    - ↑ oTreeのマスタッシュ記法を使えば自動でやってくれる．

## ページ
[https://otree.readthedocs.io/en/latest/pages.html](https://otree.readthedocs.io/en/latest/pages.html)


- クラス `Page` を継承する．
- クラス名がURLに表示される．

### 変数
#### `round_number`

#### `template_name`

#### `form_model`

#### `form_fields`

#### `timeout_seconds`

#### `timer_text`


### 組み込み関数
#### `live_method`
- (player: Player, data)

#### `get_form_fields`
- (player: Player)

#### `vars_for_template`
- (player: Player)

#### `js_vars`
- (player: Player)

#### `before_next_page`
- (player: Player, timeout_happened)

#### `is_displayed`
- (player: Player)

#### `error_message`
- (player: Player, values)

#### `get_timeout_seconds`
- (player: Player)

#### `app_after_this_page`
- (player: Player, upcoming_apps)


## 待機ページ
[https://otree.readthedocs.io/en/latest/multiplayer/waitpages.html](https://otree.readthedocs.io/en/latest/multiplayer/waitpages.html)


- クラス `WaitPage` を継承する．
- クラス名がURLに表示される．

### 変数
#### `round_number`

#### `template_name`

#### `group_by_arrival_time`

#### `wait_for_all_groups`

#### `title_text`

#### `body_text`

### 組み込み関数
#### `vars_for_template`
- (player: Player)

#### `js_vars`
- (player: Player)

#### `is_displayed`
- (player: Player)

#### `app_after_this_page`
- (player: Player, upcoming_apps)

#### `after_all_players_arrive`
- (group: Group)



## 組み込み関数
#### `creating_session`

#### `custom_export`

#### `group_by_arrival_time_method`

#### `vars_for_admin_report`

#### `{field_name}_min`

#### `{field_name}_max`

#### `{field_name}_choices`

#### `{field_name}_error_message`


## 自作の関数

## `page_sequence`
