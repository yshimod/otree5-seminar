{% raw %}
【第7回】 2022年6月23日


- 入力フォームの順番をランダム化する
- ページの順番をランダム化する
- Bootstrapで入力フォームの見た目をモダンにする



# （Qualtricsのような）質問紙調査を作る方法

- 今日のゴール
    - コード [https://github.com/yshimod/otree_survey](https://github.com/yshimod/otree_survey)
    - デモページ [https://otree-seminar-survey-sample.herokuapp.com/demo](https://otree-seminar-survey-sample.herokuapp.com/demo)


## 1. 定数 `C` クラスに質問紙調査の「素材」を定義しておく

- 質問文・選択肢・正答などを1箇所にまとめて定義しておいたほうが管理しやすい．


- 辞書型で定義すると良い．たとえば以下のように CRT と General Trust Scale （以下 Gen Trust ） 用の2つの辞書オブジェクトを定義しておく．

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


- Python で辞書型のオブジェクトを定義するには，JSONのように `{ }` の中に記述するか， `dict()` を使って記述しても良い．以下の2つは等価．

  ```python
  year = 2022
  mydict1 = {
      "year": year,
      "key1": "あいうえお"
  }
  ```
  ```python
  year = 2022
  mydict2 = dict(
      year = year,
      key1 = "あいうえお"
  )
  ```
  ```python
  mydict1 == mydict2
  # True
  ```


- 辞書オブジェクトから値を取り出すときは `辞書[キー]` とする．たとえば，コード例の `crt1` の質問文を取り出すには， `C.materials_crt["items"]["crt1"]["question"]` とする．


- 辞書オブジェクトからキーの一覧を取得するには， `.keys()` メソッドを使う．たとえばキーを一つずつ `print` するには，

  ```python
  for k in C.materials_crt["items"].keys():
      print(k)
  # crt1
  # crt2
  # crt3
  ```

    - `辞書.keys()` はリストではない（ dict_keys オブジェクトである）ため， `辞書.keys()[0]` として0番目のキーを取り出す，ということはできない．

    - リストに変換するためには， `辞書.keys()` をアンパックすればよい．つまり `[*辞書.keys()]` とすればキーの文字列が並んだリストが得られる．

        - アンパック以外に， `list()` 関数を使う（ `list(辞書.keys())` ），内包表記を使う（ `[k for k in 辞書.keys()]` ）方法があるが，アンパックを使った方が処理が速い．


- 辞書オブジェクトからキーと対応する値の両方を取得するには， `.items()` メソッドを使う．たとえばキーと値の組を一つずつ `print` するには，

  ```python
  for k, v in C.materials_gentrust["items"].items():
      print(k + " の値は " + v)
  # gentrust1 の値は Most people are basically honest.
  # gentrust2 の値は Most people are trustworthy.
  # gentrust3 の値は Most people are basically good and kind.
  # gentrust4 の値は Most people are trustful of others.
  # gentrust5 の値は I am trustful.
  # gentrust6 の値は Most people will respond in kind when they are trusted by others.
  ```

    - `辞書.items()` をリストに変換したとき，タプルにキーと値が入った状態でリストの要素になっている．


- 値として関数自体を入れることもできる．たとえば `C.materials_crt["field"]` の値は関数 `models.FloatField` であり，以下の2つは等価である．

  ```python
  class Player(BasePlayer):
      testvar1 = models.FloatField(label = "入力してください．")
  ```
  ```python
  class Player(BasePlayer):
      testvar1 = C.materials_crt["field"](label = "入力してください．")
  ```


- 質問文など長文を定義するとき，三重引用符 `""" """` で括れば途中に改行を入れても良い．文字列には改行文字（ `\n` ）やインデント用に入れてある空白文字がそのまま入る．たとえば `C.materials_crt["items"]["crt1"]["question"]` は以下が返ってくる．

  ```
  '\n                A bat and a ball cost $1.10 in total.\n                The bat costs $1.00 more than the ball.\n                How much does the ball cost?\n            '
  ```

    - ただし，その文字列がテンプレートで展開されるときには改行文字は空白文字に置換され，連続した空白文字は無視される．

    - 改行したい場合には `<br>` や `<p>` のタグを使用した上で，テンプレートにおいてこれらのタグが機能する形で文字列を展開する必要がある．

        - データモデルのクラスで `var1 = models.IntegerField(label="ここで<br>改行")` と定義した上で `{{ formfields }}` タグで入力フォームを実装したとき HTML タグ（たとえば `<br>` タグ）は機能しない．何となれば， `label` に渡した文字列が HTML に展開されるとき，HTMLセーフな文字列（ `ここで<br>改行` は `ここで&lt;br&gt;改行` ）に変換された上で `<label>` 要素として展開されるため．



## 2. データモデルでたくさんの変数を定義する

- [https://otree.readthedocs.io/en/latest/misc/tips_and_tricks.html#how-to-make-many-fields](https://otree.readthedocs.io/en/latest/misc/tips_and_tricks.html#how-to-make-many-fields)

- 自分の手で一つずつ定義すると以下のようにする．

  ```python
  class Player(BasePlayer):
      crt1 = models.FloatField(
          label = C.materials_crt["items"]["crt1"]["question"],
          help_text = C.materials_crt["items"]["crt1"]["unit"]
      )
      crt2 = models.FloatField(
          label = C.materials_crt["items"]["crt2"]["question"],
          help_text = C.materials_crt["items"]["crt2"]["unit"]
      )
      crt3 = models.FloatField(
          label = C.materials_crt["items"]["crt3"]["question"],
          help_text = C.materials_crt["items"]["crt3"]["unit"]
      )
      gentrust1 = models.IntegerField(
          label = C.materials_gentrust["items"]["gentrust1"],
          choices = C.materials_gentrust["opts"],
          widget = widgets.RadioSelectHorizontal
      )
      gentrust2 = models.IntegerField(
          label = C.materials_gentrust["items"]["gentrust2"],
          choices = C.materials_gentrust["opts"],
          widget = widgets.RadioSelectHorizontal
      )
      gentrust3 = models.IntegerField(
          label = C.materials_gentrust["items"]["gentrust3"],
          choices = C.materials_gentrust["opts"],
          widget = widgets.RadioSelectHorizontal
      )
      gentrust4 = models.IntegerField(
          label = C.materials_gentrust["items"]["gentrust4"],
          choices = C.materials_gentrust["opts"],
          widget = widgets.RadioSelectHorizontal
      )
      gentrust5 = models.IntegerField(
          label = C.materials_gentrust["items"]["gentrust5"],
          choices = C.materials_gentrust["opts"],
          widget = widgets.RadioSelectHorizontal
      )
      gentrust6 = models.IntegerField(
          label = C.materials_gentrust["items"]["gentrust6"],
          choices = C.materials_gentrust["opts"],
          widget = widgets.RadioSelectHorizontal
      )
  ```

    - 自分の手でコピペしてループするのは危険．うっかりミスをしがち．たとえ，自分は几帳面である，と思っている人であっても．


- `Player` クラスの外側（下側）で， `setattr(Player, "変数名", models.*Field())` をforループで回す．以下の記述は，上で一つずつ定義しているのと同じ．

  ```python
  class Player(BasePlayer):
      pass

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

    - Python の組み込み関数 `setattr()` は，第2引数の文字列の名前の変数に，第3引数の値を代入したものを，第1引数のオブジェクト（たとえば `Player` クラス）に追加する．以下の2つは等価である．

    ```python
    class Player(BasePlayer):
        testvar1 = models.FloatField(label = "入力してください．")
    ```
    ```python
    class Player(BasePlayer):
        pass

    setattr(
        Player,    ## 既に定義してあるクラスオブジェクト
        "testvar1",    ## 文字列
        models.FloatField(label = "入力してください．")
    )
    ```


## 3. 入力フォームの順番をランダム化する

- [https://otree.readthedocs.io/en/latest/forms.html#customizing-a-field-s-appearance](https://otree.readthedocs.io/en/latest/forms.html#customizing-a-field-s-appearance)


- テンプレートで `{{ formfields }}` タグなどを使って入力フォームを作るとき，入力フォームの順番は，ページクラスの `form_fields` に渡したリストの順番となる． `get_form_fields()` を定義した場合はその返り値のリストの順番が優先される．

    - たとえば

    ```python
    class Survey1(Page):
        form_model = "player"

        @staticmethod
        def get_form_fields(player: Player):
            return ["crt1", "crt3", "crt2"]
    ```

    なるページを定義した上で，テンプレートで `{{ formfields }}` を使ったとき， "crt1"， "crt3"， "crt2" の順番で入力フォームが生成される．

    ```html
    <div class="mb-3 _formfield">
        <label class="col-form-label" for="id_crt1">
            A bat and a ball cost $1.10 in total.
            The bat costs $1.00 more than the ball.
            How much does the ball cost?
        </label>
        <div class="controls">
            <input type="text" class="form-control" id="id_crt1" name="crt1" required value="">
        </div>
        <p>
            <small>
                <p class="form-text text-muted">cent(s)</p>
            </small>
        </p>
    </div>
    <div class="mb-3 _formfield">
        <label class="col-form-label" for="id_crt3">
            In a lake, there is a patch of lily pads.
            Every day, the patch doubles in size.
            If it takes 48 days for the patch to cover the entire lake,
            how long would it take for the patch to cover half of the lake?
        </label>
        <div class="controls">
            <input type="text" class="form-control" id="id_crt3" name="crt3" required value="">
        </div>
        <p>
            <small>
                <p class="form-text text-muted">day(s)</p>
            </small>
        </p>
    </div>
    <div class="mb-3 _formfield">
        <label class="col-form-label" for="id_crt2">
            If it takes 5 machines 5 minutes to make 5 widgets,
            how long would it take 100 machines to make 100 widgets?
        </label>
        <div class="controls">
            <input type="text" class="form-control" id="id_crt2" name="crt2" required value="">
        </div>
        <p>
            <small>
                <p class="form-text text-muted">minute(s)</p>
            </small>
        </p>
    </div>
    ```

    - `{{ for }}` ループを使って，テンプレートを

    ```html
    {{ for eachfield in form }}
        <div>
            <p>label: {{ eachfield.label }}</p>
            <p>name: {{ eachfield.name }}</p>
            <p>id: {{ eachfield.id }}</p>
            <p>description: {{ eachfield.description }}</p>
        </div>
    {{ endfor }}
    ```

    のように記述したとき，以下の HTML が生成される．

    ```html
    <div>
        <p>label:
            <label for="id_crt1">
                A bat and a ball cost $1.10 in total.
                The bat costs $1.00 more than the ball.
                How much does the ball cost?
            </label>
        </p>
        <p>name: crt1</p>
        <p>id: id_crt1</p>
        <p>description: cent(s)</p>
    </div>
    <div>
        <p>label:
            <label for="id_crt3">
                In a lake, there is a patch of lily pads.
                Every day, the patch doubles in size.
                If it takes 48 days for the patch to cover the entire lake,
                how long would it take for the patch to cover half of the lake?
            </label>
        </p>
        <p>name: crt3</p>
        <p>id: id_crt3</p>
        <p>description: day(s)</p>
    </div>
    <div>
        <p>label:
            <label for="id_crt2">
                If it takes 5 machines 5 minutes to make 5 widgets,
                how long would it take 100 machines to make 100 widgets?
            </label>
        </p>
        <p>name: crt2</p>
        <p>id: id_crt2</p>
        <p>description: minute(s)</p>
    </div>
    ```

        - 詳細は [こちら](otree_ref/templatefile.md#form-オブジェクト) ．


- `{{ formfields }}` タグなどを使って入力フォームを作るとき，ページクラスの `get_form_fields()` で， player ごとシャッフルされた変数名のリストを返せば， player ごとフォームの順番をランダム化できる．


- 乱数を引く処理を行う場合，乱数は一度だけ引き，その結果を記録しておくと良い．たとえば， `creating_session()` の中でリストを並び替え，その結果を `json.dumps()` を使ってJSON文字列に変換し， player モデルの変数（ `order_crt = models.LongStringField()` と定義してある）に記録しておく．

  ```python
  # import random
  # import json
  def creating_session(subsession: Subsession):
      list_crt = [*C.materials_crt["items"].keys()]    ## 変数名（文字列）のリスト

      ## 全ての player について乱数を引いて結果を保存しておく
      for p in subsession.get_players():
          new_list_crt = random.sample(list_crt, len(list_crt))
          p.order_crt = json.dumps(new_list_crt)
  ```

    - 「JSON文字列」とは， `'["crt1", "crt3", "crt2"]'` のように変換された文字列．

    - JSON文字列に変換せず，リストオブジェクトのままデータを記録しよう（たとえば `p.order_crt = new_list_crt` ）とするとエラーとなる．


- `get_form_fields()` において，JSON文字列として記録してある変数名のリストを `json.loads()` でパース（Pythonがリストとして扱えるように変換）してから返す．

  ```python
  # import json
  @staticmethod
  def get_form_fields(player: Player):
      return json.loads(player.order_crt)
  ```


- 入力フォームの順番をランダム化するには，直接 `get_form_fields()` の中で乱数を引いてリストの要素を並び替えたものを返しても良さそう．たとえば以下のような実装が考えられる（が，おすすめできない）．

  ```python
  # import random
  @staticmethod
  def get_form_fields(player: Player):
      list_crt = [*C.materials_crt["items"].keys()]
      new_list_crt = random.sample(list_crt, len(list_crt))

      return new_list_crt
  ```

    - `get_form_fields()` はページを読み込む度に実行されるため，↑ の実装だと，画面を更新する度にフォームの順番が並び替えられてしまう．


- セッション作成時に呼び出される `creating_session()` ではなく，どうしても `get_form_fields()` の中で乱数を引きたい場合，結果が記録されていない場合（ `player.field_maybe_none("order_crt") == None` ）にのみ乱数を引くような実装にしたほうが良い．たとえば以下のような実装をする．

  ```python
  # import random
  # import json
  @staticmethod
  def get_form_fields(player: Player):
      if player.field_maybe_none("order_crt") == None:
          list_crt = [*C.materials_crt["items"].keys()]
          new_list_crt = random.sample(list_crt, len(list_crt))
          player.order_crt = json.dumps(new_list_crt)

      return json.loads(player.order_crt)
  ```

    - 変数が `None` か否かを判定するときに直接 `player.order_crt == None` として比較すると，（本当に `None` の場合）エラーとなるので，組み込みメソッド `.field_maybe_none()` を使う．



## 4. ページの順番をランダム化する

- 複数ページ（ここでは CRT 用のページと Gen Trust 用のページ）で共通して使うテンプレートファイル（たとえば `survey_template.html` ）を作成する．

    - たとえば以下のような記述であれば，ページの内容（ CRT か Gen Trust か ）によらずテンプレートを共通化できる．

    ```html
    {{ block title }}
        アンケートにご回答ください
    {{ endblock }}

    {{ block content }}
        {{ formfields }}
        {{ next_button }}
    {{ endblock }}
    ```


- 入力フォームを `{{ formfields }}` タグ（のみ）を使って実装している場合，当該ページで表示させたい入力フォームの変数名のリストを `get_form_fields()` で返せば良い．

    - まず，ページの順番のリストを `creating_session()` の中で乱数を引いて決定する．なお，あらかじめ順番を記録しておく変数 `order_pages` と `order_crt` と `order_gentrust` を `models.LongStringField` で定義しておく．

    ```python
    # import random
    # import json
    def creating_session(subsession: Subsession):
        list_crt = [k for k in C.materials_crt["items"].keys()]
        list_gentrust = [k for k in C.materials_gentrust["items"].keys()]

        ## 全ての player について乱数を引いた結果を保存しておく
        for p in subsession.get_players():
            list_pages = ["crt", "gentrust"]
            new_list_pages = random.sample(list_pages, len(list_pages))
            p.order_pages = json.dumps(new_list_pages)
            ## ↑ 文字列 "crt" と "gentrust" が入ったリスト．
            ## 0番目の要素には，1ページ目（Survey1）で表示させたいもの，
            ## 1番目の要素には，2ページ目（Survey2）で表示させたいもの，が入っている．

            list_crt = [*C.materials_crt["items"].keys()]
            new_list_crt = random.sample(list_crt, len(list_crt))
            p.order_crt = new_list_crt

            list_gentrust = [*C.materials_crt["items"].keys()]
            new_list_gentrust = random.sample(list_gentrust, len(list_gentrust))
            p.order_gentrust = json.dumps(new_list_gentrust)
    ```

    - 各ページのクラスで定義する `get_form_fields()` を，共通化するために，ページクラスの外側で，自前の関数として以下を定義する．たとえば関数名を `my_get_form_fields` としている．

    ```python
    # import json
    def my_get_form_fields(player: Player, idx):
        order_pages = json.loads(player.order_pages)

        if order_pages[idx] == "crt":
            ## もしも order_pages の idx 番目の要素が "crt" の場合...
            return json.loads(player.order_crt)    ## ["crt1", "crt2", "crt3"] （を並び替えたもの）を返す．
        else:
            ## もしも order_pages の idx 番目の要素が "gentrust" の場合...
            return json.loads(player.order_gentrust)    ## ["gentrust1", "gentrust2", ..., "gentrust6"] （を並び替えたもの）を返す．
    ```

        - 第2引数 `idx` には，当該ページが何ページ目なのかを判断するための変数を受け取っている．

    - 各ページクラスを定義する．

    ```python
    class Survey1(Page):
        template_name = __name__ + "/survey_template.html"
        form_model = "player"

        @staticmethod
        def get_form_fields(player: Player):
            return my_get_form_fields(player, 0)

    class Survey2(Page):
        template_name = __name__ + "/survey_template.html"
        form_model = "player"

        @staticmethod
        def get_form_fields(player: Player):
            return my_get_form_fields(player, 1)
    ```

        - `template_name` にテンプレートファイルのパスを渡せば，共通のテンプレートファイルを異なるページで読み込むように設定できる．

            - `__name__` はアプリ名を呼び出している．「survey」なるアプリであれば， `__name__ + "/survey_template.html"` は `"survey/survey_template.html"` と等価．

        - 各ページのクラスで，自前の関数 `my_get_form_fields()` を組み込みメソッド `get_form_fields()` の中で呼び出す．

        - ページクラスの名前（ `"Survey1"`， `"Survey2"` ）の末尾の数字を取り出したい場合...

            - たとえば `Survey1` クラスの中で `__class__.__name__` で文字列 `"Survey1"` が取得できるので， `int(__class__.__name__[-1]) - 1` は `0` である．

            - ページクラスの外側で，ページクラスの名前を取得するには， `participant._current_page_name` を使えば良い．呼び出された段階でその participant が滞在しているページのクラスの名前が取得できる．

        - `Survey1` クラスと `Survey2` クラスでほとんど同じことを書いているので，クラスの継承を使ってもう少し簡略化した記述も可能．

        ```python
        class SurveyCommon(Page):
            template_name = __name__ + "/survey_template.html"
            form_model = "player"

            @staticmethod
            def get_form_fields(player: Player):
                idx = int(player.participant._current_page_name[-1]) - 1
                return my_get_form_fields(player, idx)
        
        class Survey1(SurveyCommon):
            pass

        class Survey2(SurveyCommon):
            pass
        ```


- ページ別にテンプレートファイルを分けている場合（たとえば CRT 用に `crt.html`， Gen Trust 用に `gentrust.html` ），共通のテンプレートにおいて読み込むテンプレートファイルを切り替える．

    - ページクラスの `vars_for_template()` メソッドで，当該ページで CRT を表示するのか Gen Trust を表示するのかの情報（テンプレートファイルのパスやフラグの文字列など）をテンプレートに渡す．

    ```python
    # import json
    def get_pagename(player: Player, idx):
        order_pages = json.loads(player.order_pages)
        page_name =  order_pages[idx]
        return page_name

    def my_get_form_fields(player: Player, idx):
        page_name = get_pagename(player, idx)
        if page_name == "crt":
            return json.loads(player.order_crt)
        else:
            return json.loads(player.order_gentrust)

    def my_vars_for_template(player: Player, idx):
        page_name = get_pagename(player, idx)
        if page_name == "crt":
            page_path = __name__ + "/crt.html"
        else:
            page_path = __name__ + "/gentrust.html"

        return {
            "page_num": idx + 1,
            "page_name": page_name,
            "page_path": page_path
        }

    class Survey1(Page):
        template_name = __name__ + "/survey_template.html"
        form_model = "player"

        @staticmethod
        def get_form_fields(player: Player):
            return my_get_form_fields(player, 0)

        @staticmethod
        def vars_for_template(player: Player):
            return my_vars_for_template(player, 0)

    class Survey2(Page):
        template_name = __name__ + "/survey_template.html"
        form_model = "player"

        @staticmethod
        def get_form_fields(player: Player):
            return my_get_form_fields(player, 1)

        @staticmethod
        def vars_for_template(player: Player):
            return my_vars_for_template(player, 1)
    ```

    - 共通で使うテンプレートファイル `survey_template.html` は，たとえば以下のように記述する．

    ```html
    {{ block title }}
        アンケートにご回答ください （ {{ page_num }} / 2 ページ ）
    {{ endblock }}

    {{ block content }}

        {{ if page_name == "crt" }}
            <p>以下の CRT の質問に回答してください．</p>
        {{ else }}
             <p>以下の Gen Trust の質問に回答してください．</p>
        {{ endif }}

        {{ include page_path }}

        {{ next_button }}

    {{ endblock }}
    ```


<p class="ytubevideo"><iframe width="560" height="315" src="https://www.youtube.com/embed/hvYpAj7eA_0?rel=0&enablejsapi=1&origin=https://yshimod.github.io/" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe></p>


- 勉強会当日の説明が不十分だったのと，しゃべりながらより賢い方法に気づいたのもあり，動画でのコードの書き方とこのページでの説明は一部異なっています．


- 要点は以下．

    - `{{ formfields }}` を使うとき，どの入力フォームがどの順番で生成されるのかは `get_form_fields()` が返すリストに従う．したがって，リストの中身を player ごと，あるいはページの順番ごと変化させることによって入力フォームの順番やページの順番をランダム化できる．

    - `{{ formfields }}` を使わずに画面の内容を player ごと，あるいはページの順番ごと，変化させたい場合，素材を `vars_for_template()` を使ってテンプレートに渡せば良い．

    - 乱数を引く場合には，一度だけ乱数を引いてその結果を記録しておく．リスト型の場合は JSON 文字列に変換してから記録すれば良い．

    - ページの順番をシャッフルしたい場合，ページクラスを定義するときにはページの内容に依存しないように記述しなければならない．したがって，テンプレートファイルは共通化したものを最低限用意して，その中でテンプレートタグを使って内容を変化させるように実装すればよい．またページクラスの組み込みメソッドは，インデックスで挙動を変えるような関数を自前であらかじめ定義しておき，それを呼び出すように実装すればよい．


- 自分が今処理しようとしているオブジェクト（リストや辞書）がどういうものか分からなくなったとき，その箇所で `print` してみると良いでしょう．


- 組み込み関数・メソッドの返り値の型（リストを返すのか，辞書を返すのか）が分からないとき，公式ドキュメントを検索するか， [`__init__.py` の書き方](otree_ref/init.html) を参照して確認してください．



## 5. 見た目をモダンにする

### CSS

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




### Bootstrap

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
