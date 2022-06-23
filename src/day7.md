{% raw %}
【第7回】 2022年6月23日


- 今後の予定
    - 第7回: 質問紙調査（画面のデザイン，JavaScript，CSS，Bootstrap）
    - 6月30日・7月7日: 演習の時間
    - 第8回（7/14）: 連続時間ダブルオークション（ライブページとExtraModel）
    - 第回: 補遺



# （Qualtricsのような）質問紙調査を作る方法

- 今日のゴール
    - コード [https://github.com/yshimod/otree_survey](https://github.com/yshimod/otree_survey)
    - デモページ [https://otree-seminar-survey-sample.herokuapp.com/demo](https://otree-seminar-survey-sample.herokuapp.com/demo)


## 定数 `C` クラスに質問紙調査の「素材」を定義しておく

- 質問文・選択肢・正答などを1箇所にまとめて定義しておいたほうが管理しやすい．

- 辞書型で定義すると良い．

  ```python
  ## Frederick, S. (2005).
  ## "Cognitive reflection and decision making,"
  ## Journal of Economic perspectives, 19(4), 25-42.
  materials_crt = {
      "field": models.FloatField,
      "items": {
          "crt1": {
              "question": """
                  A bat and a ball cost $1.10 in total.
                  The bat costs $1.00 more than the ball.
                  How much does the ball cost?
              """,
              "unit": "cent(s)",
              "correct": 5.0
          },
          "crt2": {
              "question": """
                  If it takes 5 machines 5 minutes to make 5 widgets,
                  how long would it take 100 machines to make 100 widgets?
              """,
              "unit": "minute(s)",
              "correct": 5.0
          },
          "crt3": {
              "question": """
                  In a lake, there is a patch of lily pads.
                  Every day, the patch doubles in size.
                  If it takes 48 days for the patch to cover the entire lake,
                  how long would it take for the patch to cover half of the lake?
              """,
              "unit": "day(s)",
              "correct": 47
          }
      }
  }

  ## Yamagishi, T. & Yamagishi, M. (1994).
  ## "Trust and commitment in the United States and Japan,"
  ## Motivation and Emotion, 18, 129-166.
  materials_gentrust = {
      "field": models.IntegerField,
      "opts": [
          [1, "Strongly Disagree"],
          [2, "Disagree"],
          [3, "Neutral"],
          [4, "Agree"],
          [5, "Strongly Agree"]
      ],
      "items": {
          "gentrust1": "Most people are basically honest.",
          "gentrust2": "Most people are trustworthy.",
          "gentrust3": "Most people are basically good and kind.",
          "gentrust4": "Most people are trustful of others.",
          "gentrust5": "I am trustful.",
          "gentrust6": "Most people will respond in kind when they are trusted by others."
      }
  }
  ```


- Python で辞書型のオブジェクトを定義するには，JSONのように `{ }` の中に `"key": value` をコンマで区切って並べる．あるいは， `dict(key1 = value1, key2 = value2)` というように定義しても良い．

- 質問文など長文を定義するとき，三重引用符 `""" """` で括れば途中に改行を入れても良い．文字列には改行文字（ `\n` ）やインデント用に入れてある空白文字がそのまま入る．ただし，その文字列がテンプレートで展開されるときには改行文字は無視される．



## データモデルでたくさんの変数を定義する

- [https://otree.readthedocs.io/en/latest/misc/tips_and_tricks.html#how-to-make-many-fields](https://otree.readthedocs.io/en/latest/misc/tips_and_tricks.html#how-to-make-many-fields)

- `Player` クラスの外側（下側）で， `setattr(Player, "変数名", models.*Field())` をforループで回す．

    - `setattr()` は，第2引数の文字列の名前の変数に，第3引数の値（たとえば `models.IntegerField()` ）を代入したものを，第1引数のオブジェクト（たとえば `Player` クラス）に追加する．

    ```python
    for k, v in C.materials_crt["items"].items():
        setattr(
            Player,
            k,
            C.materials_crt["field"](
                label = v["question"],
                help_text = v["unit"]
            )
        )

    for k, v in C.materials_gentrust["items"].items():
        setattr(
            Player,
            k,
            C.materials_gentrust["field"](
                label = v,
                choices = C.materials_gentrust["opts"],
                widget = widgets.RadioSelectHorizontal
            )
        )
    ```

    - ↑ は `C` クラスで定義しておいた辞書オブジェクト `materials_crt` の `items` なる要素について， `.items()` メソッドでキーと値をそれぞれ `k` と `v` としてforループで取り出し，ループの中で `setattr()` を使う．

    - `setattr()` は `materials_crt` の `field` なる要素で定義したもの（ `models.FloatField` ）を呼び出して使っているが，これは以下の記述と等価．

    ```python
    setattr(
        Player,
        k,
        models.FloatField(
            label = v["question"],
            help_text = v["unit"]
        )
    )
    ```



## 入力フォームの順番をランダム化する

- [https://otree.readthedocs.io/en/latest/forms.html#customizing-a-field-s-appearance](https://otree.readthedocs.io/en/latest/forms.html#customizing-a-field-s-appearance)


- テンプレートで `{{ formfields }}` タグなどを使って入力フォームを作るとき，入力フォームの順番は，ページクラスの `form_fields` に渡したリストの順番となる． `get_form_fields()` を定義した場合はその返り値のリストの順番が優先される．


- 乱数を引く処理を行う場合，乱数は一度だけ引き，その結果を記録しておくと良い．たとえば， `creating_session()` の中でリストを並び替え，その結果を `json.dumps()` を使ってJSON文字列に変換し， player モデルの変数（ `order_crt = models.LongStringField()` と定義してある）に記録しておく．

  ```python
  # import random
  # import json
  def creating_session(subsession: Subsession):
      list_crt = [k for k in C.materials_crt["items"].keys()]

      ## 全ての player について乱数を引いて結果を保存しておく
      for p in subsession.get_players():
          p.order_crt = json.dumps(
              random.sample(list_crt, len(list_crt))
          )
  ```

    - 「JSON文字列」とは， `'["crt3", "crt2", "crt1"]'` のように変換された文字列．


- `get_form_fields()` において，JSON文字列として記録してある変数名のリストを `json.loads()` でパース（Pythonがリストとして扱えるように変換）してから返す．

  ```python
  # import json
  @staticmethod
  def get_form_fields(player: Player):
      return json.loads(player.order_crt)
  ```


- したがって，入力フォームの順番をランダム化するには， `get_form_fields()` の中で乱数を引いてリストの要素を並び替えたものを返せば良さそう．たとえば以下のような実装が考えられる（が，おすすめできない）．

  ```python
  # import random
  @staticmethod
  def get_form_fields(player: Player):
      list_crt = [k for k in C.materials_crt["items"].keys()]    ## 変数名（文字列）のリスト
      new_list_crt = random.sample(list_crt, len(list_crt))    ## 破壊的に並び替える random.shuffle() を使っても良い

      return new_list_crt
  ```

    - `get_form_fields()` はページを読み込む度に実行されるため，↑ の実装だと，画面を更新する度にフォームの順番が並び替えられてしまう．


- セッション作成時に呼び出される `creating_session()` ではなく，どうしても `get_form_fields()` の中で乱数を引きたい場合，結果が記録されていない場合（ `player.field_maybe_none("order_crt") == None` ）にのみ乱数を引くような実装にしたほうが良い．たとえば以下のような実装．

  ```python
  # import random
  # import json
  @staticmethod
  def get_form_fields(player: Player):
      if player.field_maybe_none("order_crt") == None:
          list_crt = [k for k in C.materials_crt["items"].keys()]

          player.order_crt = json.dumps(
              random.sample(list_crt, len(list_crt))
          )

      return json.loads(player.order_crt)
  ```

    - 変数が `None` か否かを判定するときに直接 `player.order_crt == None` として比較すると，（本当に `None` の場合）エラーとなるので，組み込みメソッド `.field_maybe_none()` を使う．



## ページの順番をランダム化する

- テンプレートファイル（たとえば `survey_template.html` ）を複数ページで共通化する．必要に応じて，テンプレートの中で `{{ if }}` タグを使い，変数を使った条件分岐で中身を変える．

    - とりあえず `survey_template.html` のブロックコンテンツには最低限 `{{ formfields }}` と `{{ next_button }}` だけ書いておく．


- ページクラスで共通のテンプレートファイルを読み込むように設定する．

  ```python
  class Survey1(Page):
      template_name = __name__ + "/survey_template.html"
      form_model = "player"
  
  class Survey2(Page):
      template_name = __name__ + "/survey_template.html"
      form_model = "player"
  ```


- 各ページのクラスで定義する `get_form_fields()` を，共通化する．

    - ページクラスの外側で，自前の関数として以下を定義する．

    ```python
    # import json
    def my_get_form_fields(player: Player, idx):
        if json.loads(player.order_pages)[idx - 1] == "crt":
            return json.loads(player.order_crt)
        else:
            return json.loads(player.order_gentrust)
    ```

    - 各ページのクラスで，自前の関数を組み込みメソッド `get_form_fields()` の中で呼び出す．

    ```python
    class Survey1(Page):
        template_name = __name__ + "/survey_template.html"
        form_model = "player"

        @staticmethod
        def get_form_fields(player: Player):
            return my_get_form_fields(player, 1)

    class Survey2(Page):
        template_name = __name__ + "/survey_template.html"
        form_model = "player"

        @staticmethod
        def get_form_fields(player: Player):
            return my_get_form_fields(player, 2)
    ```


- ページの順番のリストを `creating_session()` の中で乱数を引いて決定する．なお，あらかじめ順番を記録しておく変数 `order_pages` と `order_crt` と `order_gentrust` を `models.LongStringField` で定義しておく．

  ```python
  # import random
  # import json
  def creating_session(subsession: Subsession):
      list_crt = [k for k in C.materials_crt["items"].keys()]
      list_gentrust = [k for k in C.materials_gentrust["items"].keys()]

      ## 全ての player について乱数を引いて結果を保存しておく
      for p in subsession.get_players():
          p.order_pages = json.dumps(
              random.sample(["crt", "gentrust"], 2)
          )

          p.order_crt = json.dumps(
              random.sample(list_crt, len(list_crt))
          )

          p.order_gentrust = json.dumps(
              random.sample(list_gentrust, len(list_gentrust))
          )
  ```


<p class="ytubevideo"><iframe width="560" height="315" src="https://www.youtube.com/embed/hvYpAj7eA_0?rel=0&enablejsapi=1&origin=https://yshimod.github.io/" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe></p>



## CSS

- テンプレートで `{{ formfields }}` タグを使って入力フォームを作ると，あまりにもデザインがダサく辟易してしまう．


- CSS を指定してスタイルを変更する．

    - [https://otree.readthedocs.io/en/latest/templates.html#javascript-and-css](https://otree.readthedocs.io/en/latest/templates.html#javascript-and-css)


- CSS を指定するには，以下の3通り．

    - テンプレートファイルの HTML タグに `style` 属性を加える．

    ```html
    <p style="font-weight: bold;">以下の質問にご回答ください．</p>
    ```

    - テンプレートファイルに `{{ block styles }} {{ endblock }}` を置き，その中に`<style>` タグで CSS を書く．

    ```html
    {{ block title }}
        タイトル
    {{ endblock }}

    {{ block styles }}
        <style type="text/css">
            .mypstyle{
                font-weight: bold;
            }
        </style>
    {{ endblock }}

    {{ block content }}
        <p class="mypstyle">以下の質問にご回答ください．</p>
        {{ formfields }}
        {{ next_button }}
    {{ endblock }}
    ```

        - `{{ block content }} {{ endblock }}` の中に直接 `<style>` タグを書いても良い．ただし読み込まれる順番が遅くなる．

    - CSS ファイル（ `mystyles.css` ）を作成して， `_static/global` などに入れておき，それを読み込む．

    ```html
    {{ block title }}
        タイトル
    {{ endblock }}

    {{ block styles }}
        <link rel="stylesheet" href="{{ static 'global/mystyles.css' }}">
    {{ endblock }}

    {{ block content }}
        <p class="mypstyle">以下の質問にご回答ください．</p>
        {{ formfields }}
        {{ next_button }}
    {{ endblock }}
    ```

    CSS ファイル `mystyles.css` の中身は以下．

    ```css
    .mypstyle{
        font-weight: bold;
    }
    ```


- ブラウザで HTML の要素を検証し， oTree 組み込みの CSS セレクタを見つけ，それをカスタマイズすればよい．

    - [https://otree.readthedocs.io/en/latest/templates.html#customizing-the-theme](https://otree.readthedocs.io/en/latest/templates.html#customizing-the-theme)

    - たとえば「次へ」ボタンを右寄せにしたい場合，以下のように CSS を設定する．

    ```css
    .otree-btn-next {
        display: block;
        margin: 0 0 0 auto;
    }
    ```


<p class="ytubevideo"><iframe width="560" height="315" src="https://www.youtube.com/embed/tX7Twpkillk?rel=0&enablejsapi=1&origin=https://yshimod.github.io/" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe></p>




## Bootstrap

- oTree ではデフォルトで Bootstrap が読み込まれている．

    - [Bootstrap のドキュメント](https://getbootstrap.com/docs/5.0/getting-started/introduction/)

    - Bootstrap のバージョンは 5.0.1 （oTree v5.8.5 において）．

- デフォルトで読み込まれているので，自分で読み込んではいけない．異なるバージョンを使うことはできない（？）．

- ドキュメントで使いたい機能を検索して，コードの例をコピーする．

- たとえば（いわゆる目玉のようなラジオボタンではなく）四角のボタンでラジオボタンを実装するには以下．

  ```html
  <div class="btn-group" role="group" aria-label="Basic radio toggle button group">
      <input type="radio" class="btn-check" name="{{ 変数名 }}" id="{{ 変数名 }}-1" autocomplete="off">
      <label class="btn btn-outline-primary" for="{{ 変数名 }}-1">Radio 1</label>

      <input type="radio" class="btn-check" name="{{ 変数名 }}" id="{{ 変数名 }}-2" autocomplete="off">
      <label class="btn btn-outline-primary" for="{{ 変数名 }}-2">Radio 2</label>

      <input type="radio" class="btn-check" name="{{ 変数名 }}" id="{{ 変数名 }}-3" autocomplete="off">
      <label class="btn btn-outline-primary" for="{{ 変数名 }}-3">Radio 3</label>
  </div>
  ```

    - [https://getbootstrap.com/docs/5.0/components/button-group/#checkbox-and-radio-button-groups](https://getbootstrap.com/docs/5.0/components/button-group/#checkbox-and-radio-button-groups)


- 要素（たとえば入力フォームと説明文と画像）を画面に敷き詰める場合， Bootstrap の Grid system が有用．

    - [https://getbootstrap.com/docs/5.0/layout/grid/](https://getbootstrap.com/docs/5.0/layout/grid/)


<p class="ytubevideo"><iframe width="560" height="315" src="https://www.youtube.com/embed/deJDSzgFY68?rel=0&enablejsapi=1&origin=https://yshimod.github.io/" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe></p>



{% endraw %}
