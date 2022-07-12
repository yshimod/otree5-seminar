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


- 辞書オブジェクトから値を取り出すときは `辞書["キー"]` とする．たとえば，コード例の `crt1` の質問文を取り出すには， `C.materials_crt["items"]["crt1"]["question"]` とする．


- 辞書オブジェクトからキーの一覧を取得するには， `.keys()` メソッドを使う．たとえばキーを一つずつ `print` するには，

  ```python
  for k in C.materials_crt["items"].keys():
      print(k)
  # crt1
  # crt2
  # crt3
  ```

    - `辞書.keys()` はリストではない（ dict_keys オブジェクトである）ため， `辞書.keys()[0]` として0番目のキーを取り出す，ということはできない．

    - リストに変換するためには， `辞書.keys()` をアンパックすれば良い．つまり `[*辞書.keys()]` とすればキーの文字列が並んだリストが得られる．

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



## 2. データモデルでたくさんのフィールドを定義する

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


- `Player` クラスの外側（下側）で， `setattr(Player, "フィールド名", models.*Field())` をforループで回す．以下の記述は，上で一つずつ定義しているのと同じ．

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


- `{{ formfields }}` タグなどを使って入力フォームを作るとき，ページクラスの `get_form_fields()` で， player ごとシャッフルされたフィールド名のリストを返せば， player ごと入力フォームの順番をランダム化できる．


- 乱数を引く処理を行う場合，乱数は一度だけ引き，その結果を記録しておくと良い．たとえば， `creating_session()` の中でリストを並び替え，その結果を `json.dumps()` を使って文字列に変換し， player モデルのフィールド（ `order_crt = models.LongStringField()` と定義してある）に記録しておく．

  ```python
  # import random
  # import json
  def creating_session(subsession: Subsession):
      list_crt = [*C.materials_crt["items"].keys()]    ## フィールド名（文字列）のリスト

      ## 全ての player について乱数を引いて結果を保存しておく
      for p in subsession.get_players():
          new_list_crt = random.sample(list_crt, len(list_crt))
          p.order_crt = json.dumps(new_list_crt)
  ```

    - `json.dumps()` は，辞書型やリスト型のオブジェクトを（JSON）文字列にエンコード（変換）する関数．その逆で，（JSONの）文字列を Python が 辞書型やリスト型のオブジェクトとして使えるようにデコードする関数は `json.loads()`．以下の操作を確認せよ．

    ```python
    # import json
    list_crt = ["crt1", "crt2", "crt3"]
    print(type(list_crt))
    # <class 'list'>
    print(list_crt[0])
    # crt1

    encoded_list_crt = json.dumps(list_crt)
    print(type(encoded_list_crt))
    # <class 'str'>
    print(encoded_list_crt)
    # ["crt1", "crt2", "crt3"]
    print(encoded_list_crt[0])
    # [

    decoded_list_crt = json.loads(encoded_list_crt)
    print(type(decoded_list_crt))
    # <class 'list'>
    print(list_crt == decoded_list_crt)
    # True
    ```

    ```python
    testdict = dict(
        testsublist = ["a", "b", "c"],
        testsubdict = dict(
            k1 = 1
        )
    )
    print(type(testdict))
    # <class 'dict'>

    encoded_testdict = json.dumps(testdict)
    print(type(encoded_testdict))
    # <class 'str'>
    print(encoded_testdict)
    # {"testsublist": ["a", "b", "c"], "testsubdict": {"k1": 1}}

    decoded_testdict = json.loads(encoded_testdict)
    print(type(decoded_testdict))
    # <class 'dict'>
    print(testdict == decoded_testdict)
    # True
    ```

    - 文字列に変換せず，リストオブジェクトのままデータを記録しようとする（たとえば `p.order_crt = new_list_crt` ）とエラーとなる．


- `get_form_fields()` において，文字列として記録してあるフィールド名のリストを `json.loads()` でパース（Pythonがリストとして扱えるように変換）してから返す．

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

    - `get_form_fields()` はページを読み込む度に実行されるため，↑ の実装だと，画面を更新する度に入力フォームの順番が並び替えられてしまう．


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

    - フィールドの中身が `None` か否かを判定するときに直接 `player.order_crt == None` として比較すると，（本当に `None` の場合）エラーとなるので，組み込みメソッド `.field_maybe_none()` を使う．



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


- 入力フォームを `{{ formfields }}` タグ（のみ）を使って実装している場合，当該ページで表示させたい入力フォームのフィールド名のリストを `get_form_fields()` で返せば良い．

    - まず，ページの順番のリストを `creating_session()` の中で乱数を引いて決定する．なお，あらかじめ順番を記録しておくフィールド `order_pages`， `order_crt`， `order_gentrust` を `models.LongStringField` で定義しておく．

    ```python
    # import random
    # import json
    def creating_session(subsession: Subsession):
        list_crt = [*C.materials_crt["items"].keys()]
        list_gentrust = [*C.materials_gentrust["items"].keys()]

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
            "page_num": idx + 1,    ## 数値 1 または 2
            "page_name": page_name,    ## 文字列 "crt" または "gentrust"
            "page_path": page_path    ## 文字列 "アプリ名/crt.html" または "アプリ名/gentrust.html"
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

        {# ↓ vars_for_template() で定義した変数 page_name で条件分岐 #}
        {{ if page_name == "crt" }}
            <p>以下の CRT の質問に回答してください．</p>
        {{ else }}
             <p>以下の Gen Trust の質問に回答してください．</p>
        {{ endif }}

        {# ↓ vars_for_template() で定義した変数（ファイルパス） page_path によって crt.html または gentrust.html が展開される #}
        {{ include page_path }}

        {{ next_button }}

    {{ endblock }}
    ```


- （余談） ↑ では，`{{ include }}` する子テンプレートファイルを切り替えるために， `vars_for_template()` でテンプレートファイルのパスを親テンプレートに渡していた．ページクラスで読み込む親テンプレートファイル（ `{{ block title }}` や `{{ block content }}` が含まれるテンプレート）自体も player ごと切り替えるために，ちょうど `form_fields` に渡したリストを `get_form_fields()` で上書きしたように， `template_name` に渡したパスを `get_template_name()` みたいな関数で上書きできたら便利かもしれない．公式ドキュメントで「 get_template_name 」を検索しても出てこないのだが，実のところ，ページクラスのメソッド `get_template_name()` は存在していて，我々がページクラスを定義する際に継承している親クラス（ `Page` クラス）において以下のように定義されている．

  ```python
  def get_template_name(self):
      if self.template_name is not None:
          return self.template_name
      return '{}/{}.html'.format(
          get_app_label_from_import_path(self.__module__), self.__class__.__name__
      )
  ```

    これを，自らが定義する（ `Page` クラスを継承した）クラスにおいてオーバーライドしてしまえば，望んだ使い方ができるかもしれない．注意点は， oTree 本体ではスタティックメソッドとして定義されておらず，第1引数には `self` を受け取っている点である．したがって，自分でオーバーライドする際にも第1引数には `self` を取り， `player` オブジェクトを使いたい場合は `self.player` とすれば良い．たとえば，以下のような実装が可能かもしれない．

  ```python
  class Survey1(Page):
      form_model = "player"

      def get_template_name(self):
          player = self.player
          idx = 0
          page_name = json.loads(player.order_pages)[idx]

          if page_name == "crt":
              return __name__ + "/Crt.html"
          else:
              return __name__ + "/GenTrust.html"
  ```



<p class="ytubevideo"><iframe width="560" height="315" src="https://www.youtube.com/embed/hvYpAj7eA_0?rel=0&enablejsapi=1&origin=https://yshimod.github.io/" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe></p>


- 勉強会当日の説明が不十分だったのと，しゃべりながらより賢い方法に気づいたのもあり，動画でのコードの書き方とこのページでの説明は一部異なっています．


- 要点は以下．

    - `{{ formfields }}` を使うとき，どの入力フォームがどの順番で生成されるのかは `get_form_fields()` が返すリストに従う．したがって，リストの中身を player ごと，あるいはページの順番ごと変化させることによって入力フォームの順番やページの順番をランダム化できる．

    - `{{ formfields }}` を使わずに画面の内容を player ごと，あるいはページの順番ごと，変化させたい場合，素材を `vars_for_template()` を使ってテンプレートに渡せば良い．

    - 乱数を引く場合には，一度だけ乱数を引いてその結果を記録しておく．リスト型の場合は JSON 文字列に変換してから記録すれば良い．

    - ページの順番をシャッフルしたい場合，ページクラスを定義するときにはページの内容に依存しないように記述しなければならない．したがって，テンプレートファイルは共通化したものを最低限用意して，その中でテンプレートタグを使って内容を変化させるように実装すれば良い．またページクラスの組み込みメソッドは，インデックスで挙動を変えるような関数を自前であらかじめ定義しておき，それを呼び出すように実装すれば良い．


- 自分が今処理しようとしているオブジェクト（リストや辞書）がどういうものか分からなくなったとき，その箇所で `print` してみると良いでしょう．


- 組み込み関数・メソッドの返り値の型（リストを返すのか，辞書を返すのか）が分からないとき，公式ドキュメントを検索するか， [`__init__.py` の書き方](otree_ref/init.html) を参照して確認してください．




## 5. 見た目をかっこよくする

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


- ブラウザで HTML の要素を検証し， oTree 組み込みの CSS セレクタを見つけ，それをカスタマイズすれば良い．

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

- oTree ではデフォルトで Bootstrap が読み込まれているので，使用することができる．

    - [Bootstrap のドキュメント](https://getbootstrap.com/docs/5.0/getting-started/introduction/)

    - Bootstrap のバージョンは 5.0.1 （oTree v5.8.5 において）．


- デフォルトで読み込まれているので，自分で CDN などを読み込んではいけない．異なるバージョンを使うことはできない（？）．


- 基本的には HTML タグの `class` に Bootstrap が定義するクラス名を記述すれば良い．



#### ラジオボタンを Bootstrap の Button group で実装してみる

- [https://getbootstrap.com/docs/5.0/components/button-group/#checkbox-and-radio-button-groups](https://getbootstrap.com/docs/5.0/components/button-group/#checkbox-and-radio-button-groups)


- （前提）以下のように `__init__.py` が記述してあるとする．ここでは順番のシャッフルうんぬんについては捨象する．

  %accordion%`__init__.py`%accordion%
  ```python
  from otree.api import *

  class C(BaseConstants):
      NAME_IN_URL = 'survey'
      PLAYERS_PER_GROUP = None
      NUM_ROUNDS = 1

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

  class Subsession(BaseSubsession):
      pass

  class Group(BaseGroup):
      pass

  class Player(BasePlayer):
      pass

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

  class Survey1(Page):
      template_name = __name__ + "/GenTrust.html"
      form_model = "player"
      form_fields = [*C.materials_gentrust["items"].keys()]    ## とりあえず入力フォームの順番は固定のままにしておく．

  page_sequence = [Survey1]
  ```
  %/accordion%


- テンプレートタグ `{{ formfields }}` を使うと以下のような（ダサい）スタイルとなる．

  <p class="codepen" data-height="300" data-theme-id="dark" data-default-tab="result" data-slug-hash="xxWGjOM" data-editable="true" data-user="yshimod" style="height: 300px; box-sizing: border-box; display: flex; align-items: center; justify-content: center; border: 2px solid; margin: 1em 0; padding: 1em;">
      <span>See the Pen <a href="https://codepen.io/yshimod/pen/xxWGjOM">
      oTree day7-0</a> by yshimod (<a href="https://codepen.io/yshimod">@yshimod</a>)
      on <a href="https://codepen.io">CodePen</a>.</span>
  </p>


- まず， Bootstrap のドキュメントからコードの例をコピーして，テンプレートに貼り付ける．

  ```html
  <div class="btn-group" role="group" aria-label="Basic radio toggle button group">
      <input type="radio" class="btn-check" name="btnradio" id="btnradio1" autocomplete="off" checked>
      <label class="btn btn-outline-primary" for="btnradio1">Radio 1</label>

      <input type="radio" class="btn-check" name="btnradio" id="btnradio2" autocomplete="off">
      <label class="btn btn-outline-primary" for="btnradio2">Radio 2</label>

      <input type="radio" class="btn-check" name="btnradio" id="btnradio3" autocomplete="off">
      <label class="btn btn-outline-primary" for="btnradio3">Radio 3</label>
  </div>
  ```

  <p class="codepen" data-height="300" data-theme-id="dark" data-default-tab="result" data-slug-hash="QWmbrKM" data-editable="true" data-user="yshimod" style="height: 300px; box-sizing: border-box; display: flex; align-items: center; justify-content: center; border: 2px solid; margin: 1em 0; padding: 1em;">
      <span>See the Pen <a href="https://codepen.io/yshimod/pen/QWmbrKM">
      oTree day7-1</a> by yshimod (<a href="https://codepen.io/yshimod">@yshimod</a>)
      on <a href="https://codepen.io">CodePen</a>.</span>
  </p>


- 外側の `<div>` 要素:

    - `class` 属性の `btn-group` は Bootstrap が定義したもの．
    
        - 子要素の `class="btn"` であるものを `<div class="btn-group">` で囲む．

    - `role` 属性と `aria-*` 属性は WAI-ARIA （支援技術を利用するためのルール） のためのもの．

        - （少なくともラボで実験を実施する場合は）属性ごと消してしまっても良い．

        - [https://developer.mozilla.org/ja/docs/Learn/Accessibility/WAI-ARIA_basics](https://developer.mozilla.org/ja/docs/Learn/Accessibility/WAI-ARIA_basics)


- 内側の `<input>` 要素:

    - （HTMLの機能） `type` 属性を `radio` にすることによって， `<input>` 要素がラジオボタンとして機能する．

        - [https://developer.mozilla.org/ja/docs/Web/HTML/Element/input/radio](https://developer.mozilla.org/ja/docs/Web/HTML/Element/input/radio)

    - （HTMLの機能） `name` 属性の値はフィールド名にする．選択肢の数だけ `<input>` 要素があるが，すべて同じフィールド名にする．

        - とりあえず `name="gentrust1"` としておく．

    - （HTMLの機能） `name` 属性とは異なり， `id` 属性の値は `<input>` 要素ごと区別させなければならない．

        - `<input>` 要素の `id` は `<label>` 要素との紐付けのために必要なものなので，その値自体は，ユニークでさえあれば，何でも良い．

        - oTree の流儀（というより Django の流儀を引き継いでいるのだが）としては，フィールド名の頭に `id_` をつけ，その上で枝番を（0から）つける．ただし，このルールにしたがう必要はない．

        - とりあえず `name="id_gentrust1-0"` としておく．

    - （HTMLの機能） `checked` 属性が追加されていると，そのラジオボタンが選択されていることを表す．

        - テンプレートにおいて `checked` 属性を記述してしまうと，デフォルトでそのボタンが選択されてしまう．

        - 論理属性なので，値を渡す必要はない． `checked=""` と記述してあっても， `checked` 属性自体が存在するため，そのラジオボタンが選択されていることになる．

        - 今回は，デフォルトでいずれかのラジオボタンが選択されている必要がないため， `checked` 属性は消しておく．

    - （HTMLの機能） あるラジオボタンが選択された状態でフォームが送信されると，そのラジオボタン `value` 属性の値が送信される．

        - 陽に `value` 属性を指定しない場合は，デフォルトで `"on"` なる文字列となる．

        - コピペした Bootstrap のコードには `value` 属性が記述されていないため，自分で追加しなければならない．

        - データモデルの定義をしたときに `choices` を記述している場合，その値と一致しなければならない．

        - とりあえず，値を直書きしておく．

    - （HTMLの機能） 必須回答にする場合（データモデルの定義で `blank=True` としていない場合）， `required` 属性を追加しておく．

        - 複数ある選択肢のうちの一つにだけ `required` 属性が追加されていれば， `name` 属性が同じ値である `<input>` 要素（ラジオボタン）の1つが選択されていないとフォームの送信ができない．

        - ↑ このような仕様ではあるが，保守性のために，すべての `<input>` 要素に `required` 属性を追加しておいた方が良い．

        - 論理属性なので，値を渡す必要はない．

    - （HTMLの機能） 一般論として， `autocomplete` 属性の値は `"off"` にしておいたほうが良い．さもないと， Firefox だけはラジオボタンの選択を維持してしまう．

        - しかしながら， oTree 本体でフォーム全体（コンテンツブロック全体）に対して `autocomplete="off"` としてあるため，自分で陽に書く必要はない．

        - とは言え念の為，サンプルのまま残しておく．

    - （Bootstrapの機能） `class` 属性の `"btn-check"` は Bootstrap が定義したクラス名．

        - [https://getbootstrap.jp/docs/5.0/forms/checks-radios/#radio-toggle-buttons](https://getbootstrap.jp/docs/5.0/forms/checks-radios/#radio-toggle-buttons)

        - `<label class="btn">` 要素とセットで機能する．

        - サンプルのままにしておく．


- 内側の `<label>` 要素:

    - （HTMLの機能） `for` 属性を対応する `<input>` 要素の `id` と一致させる．

    - （HTMLの機能） ラベルとして表示するテキストを `<label>` の子要素に書く．

        - とりあえず選択肢のラベルを直書きする．

    - （Bootstrapの機能） `class` 属性の `"btn btn-outline-primary"` は Bootstrap が定義したもの．2つ目のクラス `btn-outline-primary` は modifier と呼ばれるもので，base class である `btn` を修飾する．

        - たとえば `btn-outline-primary` の代わりに `btn-outline-danger` とすると，ボタンの色が赤色になる．

        - たとえばボタンのサイズを大きくしたい場合は，更に `btn-lg` クラスも適用する．つまり， `class="btn btn-outline-primary btn-lg"` とする．

        - [https://getbootstrap.jp/docs/5.0/components/buttons/](https://getbootstrap.jp/docs/5.0/components/buttons/)


- とりあえず選択肢ごと変わる部分の記述を直書きした状態は以下の通り．

  ```html
  <div class="btn-group">
      <!-- 最初の3つだけ -->
      <input type="radio" class="btn-check" name="gentrust1" id="id_gentrust1-0" autocomplete="off" value="1" required>
      <label class="btn btn-outline-primary" for="id_gentrust1-0">Strongly Disagree</label>

      <input type="radio" class="btn-check" name="gentrust1" id="id_gentrust1-1" autocomplete="off" value="2" required>
      <label class="btn btn-outline-primary" for="id_gentrust1-1">Disagree</label>

      <input type="radio" class="btn-check" name="gentrust1" id="id_gentrust1-2" autocomplete="off" value="3" required>
      <label class="btn btn-outline-primary" for="id_gentrust1-2">Neutral</label>
  </div>
  ```


- （oTreeの機能） 選択肢の繰り返しの部分は `{{ for }}` ループを使って記述する．

    - （前提） テンプレートで展開される `{{ C.materials_gentrust.opts }}` の正体は以下のリスト．

    ```python
    [
        [1, "Strongly Disagree"],
        [2, "Disagree"],
        [3, "Neutral"],
        [4, "Agree"],
        [5, "Strongly Agree"]
    ]
    ```

    - データモデルの定義の `choices` にこのリストを渡しているため，フォームで送信するべき値（ `<input>` 要素の `value` 属性の値）は，リストの各要素の0番目の要素（つまり `1`， `2`， `3`， `4`， `5` ）．

    - たとえば `{{ C.materials_gentrust.opts.1.1 }}` は `Disagree` が展開される．

    - テンプレートで `{{ for v in C.materials_gentrust.opts }}` と書くと，ループの中で...

        - `{{ v.0 }}` は `1`， `2`， `3`， `4`， `5` が展開される．

        - `{{ v.1 }}` は `Strongly Disagree`， `Disagree`， `Neutral`， `Agree`， `Strongly Agree` が展開される．

        - `{{ forloop.counter }}` は `1`， `2`， `3`， `4`， `5` が展開される．

        - `{{ forloop.counter0 }}` は `0`， `1`， `2`， `3`， `4` が展開される．

    - ある一つの質問項目の5つの選択肢は， `{{ for }}` ループを使って以下のように記述すれば良い．

    ```html
    <div class="btn-group">
        {{ for v in C.materials_gentrust.opts }}
            <input type="radio" class="btn-check" name="gentrust1" id="id_gentrust1-{{ forloop.counter0 }}" autocomplete="off" value="{{ v.0 }}" required>
            <label class="btn btn-outline-primary" for="id_gentrust1-{{ forloop.counter0 }}">{{ v.1 }}</label>
        {{ endfor }}
    </div>
    ```

  <p class="codepen" data-height="300" data-theme-id="dark" data-default-tab="result" data-slug-hash="oNqXdzr" data-editable="true" data-user="yshimod" style="height: 300px; box-sizing: border-box; display: flex; align-items: center; justify-content: center; border: 2px solid; margin: 1em 0; padding: 1em;">
      <span>See the Pen <a href="https://codepen.io/yshimod/pen/oNqXdzr">
      oTree day7-2</a> by yshimod (<a href="https://codepen.io/yshimod">@yshimod</a>)
      on <a href="https://codepen.io">CodePen</a>.</span>
  </p>


- さらに質問項目の繰り返しの部分も `{{ for }}` ループを使って記述する．

    - （前提） `form_fields = ['gentrust1', 'gentrust2', 'gentrust3', 'gentrust4', 'gentrust5', 'gentrust6']` と指定してある．

    - （oTreeの機能） テンプレートで `{{ for eachfield in form }}` と書くと，ループの中で...

        - `{{ eachfield.label }}` は，  
        `<label for="id_gentrust1">Most people are basically honest.</label>`，  
        `<label for="id_gentrust2">Most people are trustworthy.</label>`，  
        ...，  
        `<label for="id_gentrust6">Most people will respond in kind when they are trusted by others.</label>`  
        が展開される．

            - ラジオボタンを生成する目的において `for` 属性は不要であるが，あっても問題なく動くため，このまま `<p>` タグで包んで使う．

            - `<label>` タグを生成せずに，単なるテキストだけ展開してくれれば幾分便利なのだが， oTree の著者は気が利かない．

        - `{{ eachfield.name }}` は， `gentrust1`， `gentrust2`， ...， `gentrust6` が展開される．

        - `{{ eachfield.id }}` は， `id_gentrust1`， `id_gentrust2`， ...， `id_gentrust6` が展開される．

        - `{{ eachfield }}` は，以下のように一つの質問項目のすべての選択肢が展開される（ただしデータモデルの定義で `widget = widgets.RadioSelectHorizontal` と指定していることを前提）．

        %accordion%`{{ eachfield }}`で展開される HTML%accordion%
        ```html
        <div class="form-check form-check-inline">
            <input class="form-check-input" type="radio" id="id_gentrust1-0" name="gentrust1" required value="1">
            <label for="id_gentrust1-0" class="form-check-label">Strongly Disagree</label>
        </div>
        <div class="form-check form-check-inline">
            <input class="form-check-input" type="radio" id="id_gentrust1-1" name="gentrust1" required value="2">
            <label for="id_gentrust1-1" class="form-check-label">Disagree</label>
        </div>
        <div class="form-check form-check-inline">
            <input class="form-check-input" type="radio" id="id_gentrust1-2" name="gentrust1" required value="3">
            <label for="id_gentrust1-2" class="form-check-label">Neutral</label>
        </div>
        <div class="form-check form-check-inline">
            <input class="form-check-input" type="radio" id="id_gentrust1-3" name="gentrust1" required value="4">
            <label for="id_gentrust1-3" class="form-check-label">Agree</label>
        </div>
        <div class="form-check form-check-inline">
            <input class="form-check-input" type="radio" id="id_gentrust1-4" name="gentrust1" required value="5">
            <label for="id_gentrust1-4" class="form-check-label">Strongly Agree</label>
        </div>
        ```
        %/accordion%

            - 選択肢を一つずつ取り出すことも可能． `{{ eachfield.0 }}` （ `{{ for eachopt in eachfield }}` の0番目）は

            ```html
            <input class="form-check-input" type="radio" id="id_gentrust1-0" name="gentrust1" required value="1">
            ```

            `{{ eachfield.0.label }}` は

            ```html
            <label for="id_gentrust1-0">Strongly Disagree</label>
            ```

            `{{ eachfield.0.label }}` は `id_gentrust1-0` が，それぞれ展開される．

            - ↑ を見れば分かるように，選択肢を `{{ form }}` の中から取り出して展開しても（Bootstrapを使う分には）使い勝手が悪い．

            - ゆえに，選択肢を `{{ for }}` ループで作るときには，素材を直接 `C` クラスから取り出す（あるいは `vars_for_template()` で渡す ）方が良い．

            - `{{ eachfield.0.label }}` で `<label>` タグを生成せずに，単なるテキストだけ展開してくれれば幾分便利なのだが， oTree の著者は気が利かない．

        -  詳細は [こちら](otree_ref/templatefile.md#form-オブジェクト) ．

    - （oTreeの機能） 入力フォームの検証を oTree サーバー側でも行うとき（たとえば正答の判定を行う場合など）， `{{ formfield_errors フィールド名 }}` を記述しておけば，検証に引っかかった場合にエラーメッセージが `<div class="form-control-errors">ここにエラーメッセージ</div>` として展開される．

        - `{{ for eachfield in form }}` のループの中では `{{ formfield_errors eachfield.name }}` と記述しておけば良い．

        - HTML タグの直書きで実装したい場合は，たとえば以下のように記述すれば良い．

        ```html
        {{ if eachfield.errors }}
            <!-- 検証に引っかかった場合にのみ以下の要素が展開される -->
            <p>もう一度確認してください．</p>
        {{ endif }}
        ```

    - 質問文と選択肢のみを `{{ for }}` ループで質問項目の数だけ作るには以下のように記述すれば良い．

    ```html
    {{ for eachfield in form }}
        <p>{{ eachfield.label }}</p>
        <div class="btn-group">
            {{ for v in C.materials_gentrust.opts }}
                <input type="radio" class="btn-check" name="{{ eachfield.name }}" id="{{ eachfield.id }}-{{ forloop.counter0 }}" autocomplete="off" value="{{ v.0 }}" required>
                <label class="btn btn-outline-primary" for="{{ eachfield.id }}-{{ forloop.counter0 }}">{{ v.1 }}</label>
            {{ endfor }}
        </div>
        {{ formfield_errors eachfield.name }}
    {{ endfor }}
    ```

  <p class="codepen" data-height="300" data-theme-id="dark" data-default-tab="result" data-slug-hash="qBodYqG" data-editable="true" data-user="yshimod" style="height: 300px; box-sizing: border-box; display: flex; align-items: center; justify-content: center; border: 2px solid; margin: 1em 0; padding: 1em;">
      <span>See the Pen <a href="https://codepen.io/yshimod/pen/qBodYqG">
      oTree day7-3</a> by yshimod (<a href="https://codepen.io/yshimod">@yshimod</a>)
      on <a href="https://codepen.io">CodePen</a>.</span>
  </p>


- （Bootstrapの機能） 質問項目同士の間隔を離らかすには，質問項目全体を `<div>` 要素で包み，その `class` 属性に `my-5` などを加える．

    - [https://getbootstrap.jp/docs/5.0/utilities/spacing](https://getbootstrap.jp/docs/5.0/utilities/spacing)

    - `my-5` は `m`argin の `y`方向を，size `5` の距離にすることを意味する（「size」は Bootstrap のドキュメントを参照のこと）．

    - 細かく調整したい場合は，Bootstrapを使わずに直接 `styles="margin: 10px auto;"` などと指定する．


- （Bootstrapの機能） 一つの質問項目を線で囲むには， Card を使ってみる．

    - [https://getbootstrap.jp/docs/5.0/components/card/](https://getbootstrap.jp/docs/5.0/components/card/)

    - ドキュメントのコードの例:

    ```html
    <div class="card" style="width: 18rem;">
        <img src="..." class="card-img-top" alt="...">
        <div class="card-body">
            <h5 class="card-title">Card title</h5>
            <p class="card-text">Some quick example text to build on the card title and make up the bulk of the card's content.</p>
            <a href="#" class="btn btn-primary">Go somewhere</a>
        </div>
    </div>
    ```

    - 不要な要素を消すと以下のようになる．コメントアウトしてある部分に要素を挿入する．

    ```html
    <div class="card">
        <div class="card-body">
            <h5 class="card-title"><!-- ここにタイトル --></h5>
            <p class="card-text"><!-- ここに質問文 --></p>
            <!-- ここに選択肢要素 -->
            <!-- ここにエラーメッセージ要素 -->
        </div>
    </div>
    ```

    - たとえば `<h5 class="card-title">` 要素で，何番目の質問であるか（ `{{ forloop.counter }}` ）を表示してみる．


- 結局，（ページクラスで直接読み込む）テンプレートは以下のように記述すれば良い．

  ```html
  {{ block title }}
      Survey
  {{ endblock }}
  {{ block content }}
      {{ for eachfield in form }}
          <div class="card my-5">
              <div class="card-body">
                  <h5 class="card-title">{{ forloop.counter }}</h5>
                  <p class="card-text">{{ eachfield.label }}</p>
                  <div class="btn-group">
                      {{ for v in C.materials_gentrust.opts }}
                          <input type="radio" class="btn-check" name="{{ eachfield.name }}" id="{{ eachfield.id }}-{{ forloop.counter0 }}" autocomplete="off" value="{{ v.0 }}" required>
                          <label class="btn btn-outline-primary" for="{{ eachfield.id }}-{{ forloop.counter0 }}">{{ v.1 }}</label>
                      {{ endfor }}
                  </div>
                  {{ formfield_errors eachfield.name }}
              </div>
          </div>
      {{ endfor }}
      {{ next_button }}
  {{ endblock }}
  ```

  <p class="codepen" data-height="300" data-theme-id="dark" data-default-tab="result" data-slug-hash="ZExGoLR" data-editable="true" data-user="yshimod" style="height: 300px; box-sizing: border-box; display: flex; align-items: center; justify-content: center; border: 2px solid; margin: 1em 0; padding: 1em;">
      <span>See the Pen <a href="https://codepen.io/yshimod/pen/ZExGoLR">
      oTree day7-4</a> by yshimod (<a href="https://codepen.io/yshimod">@yshimod</a>)
      on <a href="https://codepen.io">CodePen</a>.</span>
  </p>



<p class="ytubevideo"><iframe width="560" height="315" src="https://www.youtube.com/embed/deJDSzgFY68?rel=0&enablejsapi=1&origin=https://yshimod.github.io/" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe></p>


- 勉強会当日は完成したコードを見ながら説明しましたが，改めて振り返るとあまりに説明を端折りすぎていたように思えたので，このページでは， Bootstrap のドキュメントからコードをコピーしてきたところから順を追ってテンプレートを書く方法を説明しています．

- Bootstrap の使い方については様々な書籍や野良のWebページが存在しているので，そちらを参照すると良いでしょう．

    - 注意点は， oTree 5 のデフォルトで読み込んでいる Bootstrap のバージョンが 5.0.1 である点です． Bootstrap のバージョン4と5では，仕様に大きな差異があります．

    - 野良のWebページのコードの例をよく見ずにコピペして使おうとしたとき，実は古いバージョンのコードであり自分のプロジェクトでは上手く機能しない，ということはありえそうです．

    - oTree 3 （3.4.0） では Bootstrap のバージョン 4.0.0 をデフォルトで読み込んでいるため， oTree 3 で書かれたコードで Bootstrap を駆使していた場合， oTree 5 でそのまま動かそうとしても意図した挙動にならないかもしれません．  
    [https://otree.readthedocs.io/en/latest/misc/otreelite.html#bootstrap](https://otree.readthedocs.io/en/latest/misc/otreelite.html#bootstrap)




<script async src="https://cpwebassets.codepen.io/assets/embed/ei.js"></script>
{% endraw %}
