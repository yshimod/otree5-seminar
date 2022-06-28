{% raw %}

# テンプレートファイルの書き方


## 基本
- ファイル名はページクラスのクラス名と一致させる．たとえば
  ```python
  class Results(Page):
      pass
  ```
    というクラスを定義していれば，テンプレートファイルの名前は `Results.html` とする．
    - ファイル名が `results.html` （大文字・小文字の違い）でもエラーが出ずに動いてしまう環境（ファイルシステムに依存）がある．しかし，大文字・小文字もちゃんと一致させておくべき．
        - Windows や Mac はファイル名の大文字と小文字を区別しない．しかし， Linux は区別する．ありがちなトラブルは，手元のPCでは正常に動くのに，Heroku（Linux）にデプロイしたときには動かない，
        - Git でプロジェクトを管理している場合，ファイル名の大文字・小文字のリネームは無視されるので注意．
    - クラス名とファイル名を一致させたくない場合はページクラスの `template_name` にパスを渡す．詳しくは[こちら](init.html#ページクラスの変数) ．
- **タイトルブロック** （ `{{ block title }}` と `{{ endblock }}` に挟まれた部分）内に，ページの冒頭で表示するタイトルを記述する．
- **コンテンツブロック** （ `{{ block content }}` と `{{ endblock }}` に挟まれた部分）内に，ページの本文を，HTMLタグも適宜使って記述する．
- CSSを使う場合は， **スタイルブロック** （ `{{ block styles }}` と `{{ endblock }}` に挟まれた部分）に `<style>` タグで直書きするか `<link>` タグを書いてCSSファイルを読み込む．．
- JavaScriptを使う場合は， **スクリプトブロック** （ `{{ block scripts }}` と `{{ endblock }}` に挟まれた部分）に `<script>` タグで直書きするかJSファイルを読み込む．

- oTree はまずはじめに oTree サーバーがテンプレートファイルを読み込んで変数の展開などを行い HTML を生成して，これをクライアント（参加者のブラウザ）に送信する．  
[https://otree.readthedocs.io/en/latest/templates.html#how-templates-work-an-example](https://otree.readthedocs.io/en/latest/templates.html#how-templates-work-an-example)

- 自分が記述したテンプレートファイルの各ブロックの内容は，以下の該当箇所に展開され，一つのHTMLが完成する．
  ```html
  <!DOCTYPE html>
  <html>
      <head>
          <title>{% block head_title %}タイトルブロックに記述した文字列{% endblock %}</title>
          {% block internal_styles %}
              <meta name="viewport" content="width=device-width, initial-scale=1.0">
              <link rel="stylesheet" href="{% static 'bootstrap5/css/bootstrap.min.css' %}">
              <link rel="stylesheet" href="{% static 'otree/css/theme.css' %}">
          {% endblock %}

          {% block global_styles %}{% endblock %}
          {% block app_styles %}{% endblock %}
          {% block styles %}スタイルブロックに記述した要素{% endblock %}
      </head>
      <body>
          {% block body_main %}
              <div class="otree-body container">
                  <h2 class="otree-title page-header" id="_otree-title">{% block title %}タイトルブロックに記述した要素{% endblock %}</h2>
                  {% if view.remaining_timeout_seconds() != None %}
                      {% with form_element_id="form" %}
                          <!-- タイムアウトの設定がしてあると，ここで残り時間が表示される． -->
                          <div class="otree-timer alert alert-warning">
                              <p>
                                  {{ timer_text }}
                                  <span style="font-weight: bold">
                                      <span class="otree-timer__time-left"></span>
                                  </span>
                              </p>
                          </div>
                      {% endwith %}
                  {% endif %}
                  {% if form.errors %}
                      <!-- oTreeのフォーム検証に引っかかると，ここでエラーメッセージが表示される． -->
                      <div class="otree-form-errors alert alert-danger">
                          {% if form.non_field_error %}
                              {{ form.non_field_error }}
                          {% else %}
                              {{ "Please fix the errors."|gettext }}
                          {% endif %}
                      </div>
                  {% endif %}
                  {% if is_defined('js_vars') and js_vars %}
                      <!-- js_varsが定義してあると，渡した辞書をここでJavaScriptのオブジェクトとして定義する． -->
                      <script>var js_vars = {{ js_vars }};</script>
                  {% endif %}

                  <form class="otree-form" method="post" role="form" id="form" autocomplete="off">
                      <script src="{% static 'otree/js/formInputs.js' %}"></script>
                      <div class="_otree-content">
                          <!-- (((((((((((((((((((((((((((((((((((((((((((((((((((((((( CONTENT BLOCK -->
                          {% block content %}
                              コンテンツブロックに記述した要素
                          {% endblock %}
                          <!-- )))))))))))))))))))))))))))))))))))))))))))))))))))))))) END OF CONTENT BLOCK -->
                      </div>
                  </form>
                  <br/>
                  {% if is_defined('view.is_debug') and view.is_debug %}
                      <!-- DEBUGモードのとき，ここにデバッグ用の情報が表示される． -->
                      <br>
                      {% include 'otree/includes/debug_info.html' %}
                  {% endif %}
              </div>
          {% endblock %}

          {% block internal_scripts %}
              <script src="{% static 'otree/js/internet-explorer.js' %}"></script>
              <script src="{% static 'otree/js/reconnecting-websocket-iife.min.js' %}"></script>
              {% block bootstrap_scripts %}
                  <script src="{% static 'bootstrap5/js/bootstrap.bundle.min.js' %}"></script>
              {% endblock %}
              <script src="{% static 'otree/js/common.js' %}"></script>
              <script id="websocket-redirect" src="{% static 'otree/js/page-websocket-redirect.js' %}" data-socket-url="{{ view.socket_url() }}" data-is-debug="{{ view.is_debug || '' }}"></script>
              {% if view.remaining_timeout_seconds() != None %}
                  {% include 'otree/includes/TimeLimit.js.html' %}
              {% endif %}
          {% endblock %}
          {% if has_live_method %}
              <form id="liveform"></form>
              <script src="{% static 'otree/js/live.js' %}" id="otree-live" data-socket-url="{{ view.live_url() }}"></script>
          {% endif %}

          {% block global_scripts %}{% endblock %}
          {% block app_scripts %}{% endblock %}
          {% block scripts %}スクリプトブロックに記述した要素{% endblock %}
      </body>
  </html>
  ```


## 変数の展開

- テンプレート内に `{{ 変数名 }}` と記述すると， oTree サーバーはその箇所に変数を展開し，参加者には具体的な変数の中身が代入されて表示される．
- たとえば，`C.ENDOWMENT` の値が `1000` であるとして，テンプレートに  
  ```html
  <p>あなたの初期保有は{{ C.ENDOWMENT }}ポイントです．</p>
  ```
と記述したとき，クライアントが oTree サーバーから受け取るHTMLデータは，
  ```html
  <p>あなたの初期保有は1000ポイントです．</p>
  ```
となる．
- 以下のインスタンスオブジェクトが使える．
    - `player`
    - （当該 player の） `group`
    - （当該 player の） `subsession`
    - （当該 player の） `participant`
    - （当該 player の） `session`
    - `C`
- 変数の演算はほとんどできない．
    - たとえば `{{ C.ENDOWMENT * 10 }}` として値を10倍して表示する，という使い方はできない．
    - データベースに保存していない変数で表示したいものは，ページクラスで定義した `vars_for_template()` の中で変数を定義してテンプレートに変数を渡す必要がある．
        - たとえば，変数を使いたい画面のページクラスに以下のように記述する．  
        ```python
        @staticmethod
        def vars_for_template(player: Player):
            return dict(
                ENDOWMENT_times_10 = C.ENDOWMENT * 10
            )
        ```
        そしてテンプレートにおいて `{{ ENDOWMENT_times_10 }}` と記述すれば， `C.ENDOWMENT` を10倍した値が画面で表示される．



## 変数に使えるフィルター

### `default`
- `{{ x | default("something") }}` と記述すると，変数 `x` の値が `None` のときに画面では「something」が表示される．
- そもそも変数 `x` が定義されていないければエラーとなる．


### `escape`
- `{{ x | escape }}` と記述すると，変数 `x` の文字列の中に含まれる `&`， `<`， `>`， `"`， `'` をHTMLセーフな文字列に変換する．
- たとえば `x = "<div style='font-weight: bold;'>AAAAA</div>"` のとき， `<p>{{ x | escape }}</p>` は `<p>&lt;div style=&#x27;font-weight: bold;&#x27;&gt;AAAAA&lt;/div&gt;</p>` と展開され，画面にはHTMLタグがそのまま表示される．フィルターを使わないと，HTMLタグがそのまま展開されるので，画面では「AAAAA」が太字で表示される．
    - 「`<script>alert('あなたは対策不足です！');</script>`」なる文字列が `{{ x }}` で展開されると， `<script>` タグ内で記述した JavaScript コードが実際に実行される（アラートが表示される）．


### `length`
- 変数が `x = [0, 1, 2]` のとき， `{{ x | length }}` と記述すると，画面では変数 `x` の長さである3が表示される．


### `c` ないし `cu`
- `{{ x | cu }}` と記述すると，変数 `x` を oTree の通貨型に変換したもの（つまり `cu(x)` ）が画面で表示される．数値が丸められるだけでなく単位（「円」や「ポイント」）も付け加えられる．


### `json`
- 変数が `x = {'key1': 10, 'key2': 20}` のような辞書型のとき， `{{ x | json }}` と記述すると，JSONにエンコードされた文字列（Pythonで `json.dumps(x)` とした返り値）が表示される．


### `to0`
- `{{ x | to0 }}` と記述すると，変数 `x` の数値が切り下げられ，整数で表示される．
- `to1` は小数第1位まで， `to2` は小数第2位まで表示される．`to3` は存在しない．



## テンプレートタグ

### `if`
- 変数の値によって条件分岐させる．
- 書き方の例:  
  ```html
  {{ if player.contribution == 0 }}
      <p>あなたは利己的な意思決定をしました．</p>
  {{ elif player.contribution < 100 }}
      <p>あなたは少しだけ貢献しました．</p>
  {{ else }}
      <p>あなたはいっぱい貢献しました．</p>
  {{ endif }}
  ```
- 使える関係演算子の一覧: `==`, `!=`, `<`, `>`, `<=`, `>=`, `in`, `not in`
- 使える論理演算子の一覧: `and`, `or`, `not`
    - `and` と `or` では `and` が優先される．
        - たとえば `{{ if expr1 or expr2 and expr3 }}` と書いたとき，はじめに `expr2 and expr3` の真偽が評価される．`expr1 = True`， `expr2 = False`， `expr3 = False`， のとき， `expr1 or expr2 and expr3` は真．
    - テンプレートタグの中で括弧 `()` は使えない．論理演算の優先順位を示す場合には `if` タグをネストさせる．
- 省略した書き方もある．
    - `{{ expr1 ?? "真" :: "偽" }}` と書いたとき， `expr1` が真であれば「真」と表示される．
    - `expr1` は真偽値そのものでないといけない．たとえば `{{ x == 1 ?? "真" :: "偽" }}` は動かない．
    - `::` 以降を省略してはいけない．偽のときに何も表示させない場合は `{{ expr1 ?? "真" :: "" }}` と書く．


### `for`
- 変数がリスト型や辞書型のとき，forループで値を一つずつ取り出して表示できる．
- 書き方の例:  
  ```html
  {{ for p in player.get_others_in_subsession() }}
      <p>プレイヤー{{ p.id_in_group }}さんの報酬は{{ p.payoff }}でした．</p>
      {{ if p.payoff > player.payoff }}
          <p>あなたの報酬よりも多いですね．</p>
      {{ endif }}
  {{ endfor }}
  ```
- 辞書型のときはキーと値を別々に取ることもできる．たとえば `mydict = {'key1': 10, 'key2': 20}` のとき，  
  ```html
  {{ for k, v in mydict.items() }}
      <p>{{ k }}: {{ v }}</p>
  {{ endfor }}
  ```
- forループの中で変数 `forloop.counter` にはカウンター番号が入っている．インデックスを0から始める場合は `forloop.counter0`．
    - たとえばMPLを作るときに，定数に `optR = [200, 250, 300, 350, 400]` を定義して，テンプレートで  
    ```html
    <table>
        <tr>
            <th>オプションL</th>
            <th></th>
            <th></th>
            <th>オプションR</th>
        </tr>
        {{ for v in C.optR }}
            <tr>
                <td>50%の確率で650円</td>
                <td><input type="radio" name="mpl_{{ forloop.counter0 }}" value="L"></td>
                <td><input type="radio" name="mpl_{{ forloop.counter0 }}" value="R"></td>
                <td>100%の確率で{{v}}円</td>
            </tr>
        {{ endfor }}
    </table>
    ```
    と記述すると，以下のような表が生成される．
    ```html
    <table>
        <tr>
            <th>オプションL</th>
            <th></th>
            <th></th>
            <th>オプションR</th>
        </tr>
        <tr>
            <td>50%の確率で650円</td>
            <td><input type="radio" name="mpl_0" value="L"></td>
            <td><input type="radio" name="mpl_0" value="R"></td>
            <td>100%の確率で200円</td>
        </tr>
        <tr>
            <td>50%の確率で650円</td>
            <td><input type="radio" name="mpl_1" value="L"></td>
            <td><input type="radio" name="mpl_1" value="R"></td>
            <td>100%の確率で250円</td>
        </tr>
        <tr>
            <td>50%の確率で650円</td>
            <td><input type="radio" name="mpl_2" value="L"></td>
            <td><input type="radio" name="mpl_2" value="R"></td>
            <td>100%の確率で300円</td>
        </tr>
        <tr>
            <td>50%の確率で650円</td>
            <td><input type="radio" name="mpl_3" value="L"></td>
            <td><input type="radio" name="mpl_3" value="R"></td>
            <td>100%の確率で350円</td>
        </tr>
        <tr>
            <td>50%の確率で650円</td>
            <td><input type="radio" name="mpl_4" value="L"></td>
            <td><input type="radio" name="mpl_4" value="R"></td>
            <td>100%の確率で400円</td>
        </tr>
    </table>
    ```
    <table>
        <tr>
            <th>オプションL</th>
            <th></th>
            <th></th>
            <th>オプションR</th>
        </tr>
        <tr>
            <td>50%の確率で650円</td>
            <td><input type="radio" name="mpl_0" value="L"></td>
            <td><input type="radio" name="mpl_0" value="R"></td>
            <td>100%の確率で200円</td>
        </tr>
        <tr>
            <td>50%の確率で650円</td>
            <td><input type="radio" name="mpl_1" value="L"></td>
            <td><input type="radio" name="mpl_1" value="R"></td>
            <td>100%の確率で250円</td>
        </tr>
        <tr>
            <td>50%の確率で650円</td>
            <td><input type="radio" name="mpl_2" value="L"></td>
            <td><input type="radio" name="mpl_2" value="R"></td>
            <td>100%の確率で300円</td>
        </tr>
        <tr>
            <td>50%の確率で650円</td>
            <td><input type="radio" name="mpl_3" value="L"></td>
            <td><input type="radio" name="mpl_3" value="R"></td>
            <td>100%の確率で350円</td>
        </tr>
        <tr>
            <td>50%の確率で650円</td>
            <td><input type="radio" name="mpl_4" value="L"></td>
            <td><input type="radio" name="mpl_4" value="R"></td>
            <td>100%の確率で400円</td>
        </tr>
    </table>
- forループをネストさせる場合， `forloop.counter` は内側のforループのカウンター番号が展開される．外側のforループのカウンター番号を展開する場合は `{{ with parentloop_counter = forloop.counter }} {{ endwith }}` で内側のforループを挟み， 外側のカウンター番号として `{{ parentloop_counter }}` を使う．たとえば  
  ```html
  {{ for i in "ABC" }}
      <p>{{ forloop.counter }}({{ i }})</p>
      {{ with parentloop_counter = forloop.counter }}
          {{ for j in "abc" }}
              <p>{{ parentloop_counter }}({{ i }}) - {{ forloop.counter }}({{ j }})</p>
          {{ endfor }}
      {{ endwith }}
  {{ endfor }}
  ```
は
  ```html
  <p>1(A)</p>
  <p>1(A) - 1(a)</p>
  <p>1(A) - 2(b)</p>
  <p>1(A) - 3(c)</p>
  <p>2(B)</p>
  <p>2(B) - 1(a)</p>
  <p>2(B) - 2(b)</p>
  <p>2(B) - 3(c)</p>
  <p>3(C)</p>
  <p>3(C) - 1(a)</p>
  <p>3(C) - 2(b)</p>
  <p>3(C) - 3(c)</p>
  ```
と展開される．
- 使うリストなどが空だった場合の例外処理は `{{ empty }}` を使って記述．
  ```html
  {{ for v in C.mylist }}
      <p>{{ forloop.counter }}番目の要素は{{v}}．</p>
  {{ empty }}
      <p>このリストは空です．</p>
  {{ endfor }}
  ```


### `include`
- テンプレート中で，パーツとなるテンプレートファイルを読み込むことができる．
- たとえば意思決定画面でもインストラクションを表示させたいときに，インストラクションの本文（タイトルブロックや「次へ」ボタンなどを除いた部分）だけを別のHTMLファイル（同じアプリ `publicgoodsgame` のディレクトリ内の `instr.html` ）として作っておく．作ったHTMLを以下のようにして読み込むと，読み込んだファイル全体が `{{ include "パス" }}` を記述した部分に展開される．
  ```html
  {{ include "publicgoodsgame/instr.html" }}
  ```
- `include` するファイルのテンプレートに変数を（1つだけ）渡すこともできる．
    - たとえば `instr.html` の中で `{{ thisyear }}` と記述しておき，  
    ```html
    {{ include "publicgoodsgame/instr.html" with thisyear=2022 }}
    ```
    と記述すると，`instr.html` の `{{ thisyear }}` に「2022」を展開した上で，`instr.html` 全体が `{{ include "パス" }}` を記述した部分に展開される．


### `static`
- プロジェクトディレクトリ直下の `_static` ディレクトリに置いたファイルについて，開発環境でのパスとブラウザが認識できるパスは異なる．ブラウザが認識できるパスを取得するために `static` タグを使う．
- [https://otree.readthedocs.io/en/latest/misc/advanced.html#static-files](https://otree.readthedocs.io/en/latest/misc/advanced.html#static-files)
- たとえば `_static/global` ディレクトリに画像ファイル `photo.png` を置いて，それを表示させるためには，テンプレートに  
  ```html
  <img src="{{ static 'global/photo.png' }}"/>
  ```
  と記述する．これを oTree は
  ```html
  <img src="/static/global/photo.png">
  ```
  と生成する．
- プロジェクトディレクトリ直下の `_static` ディレクトリは， oTree 本体が用意している `_static` （中身は以下）を上書きする．

  %accordion%oTree 本体の `_static` の中身%accordion%
  ```
  .
  ├── bootstrap5
  │   ├── css
  │   │   ├── bootstrap.min.css
  │   │   └── bootstrap.min.css.map
  │   └── js
  │       ├── bootstrap.bundle.min.js
  │       └── bootstrap.bundle.min.js.map
  ├── favicon.ico
  ├── glyphicons
  │   ├── clock.png
  │   ├── cloud.png
  │   ├── cogwheel.png
  │   ├── delete.png
  │   ├── download-alt.png
  │   ├── eye-open.png
  │   ├── folder-closed.png
  │   ├── link.png
  │   ├── list-alt.png
  │   ├── pencil.png
  │   ├── plus.png
  │   ├── pushpin.png
  │   ├── refresh.png
  │   ├── stats.png
  │   └── usd.png
  ├── otree
  │   ├── css
  │   │   ├── table.css
  │   │   └── theme.css
  │   └── js
  │       ├── common.js
  │       ├── formInputs.js
  │       ├── internet-explorer.js
  │       ├── jquery-3.2.1.min.js
  │       ├── jquery.color-2.1.2.min.js
  │       ├── jquery.countdown.min.js
  │       ├── jquery.timeago.en-short.js
  │       ├── jquery.timeago.js
  │       ├── live.js
  │       ├── monitor2.js
  │       ├── page-websocket-redirect.js
  │       ├── reconnecting-websocket-iife.min.js
  │       └── table-utils.js
  └── robots.txt
  ```
  %/accordion%
    - 画像ファイルならともかく，CSSファイル，JSファイルを上書きするのは危険かも．


### `extends`
- 親テンプレートファイル（たとえば `myapp` なるアプリのディレクトリに `mytemplate.html` なるファイル）を自分で作ったとき，（ページクラスの `template_name` で指定する）テンプレートファイルの冒頭で `{{ extends "myapp/mytemplate.html" }}` と記述しておけば，自分で作った親テンプレートファイルの記述を継承できる．
- 親テンプレートファイルを作るとき，その冒頭に `{{ extends "otree/Page.html" }}` と記述する必要がある．


### `block`
- タイトルブロック: `{{ block title }}` と `{{ endblock }}` に挟まれた部分．
    - タイトルブロック内に，ページの冒頭で表示するタイトルを記述する．
    - oTree サーバーはテンプレートを解釈して，記述した内容を `<head>` の `<title>` タグの中身（ブラウザウィンドウのタイトルに表示される）と，`<body>` の冒頭にある `<h2>` タグの中に挿入する．
    - `<h2>` タグ内にテキストが挿入されることによって，ページ内にタイトルが大きな文字で表示される．
- コンテンツブロック: `{{ block content }}` と `{{ endblock }}` に挟まれた部分．
    - コンテンツブロック内に，ページの本文を，HTMLタグも適宜使って記述する．
    - oTree サーバーはテンプレートを解釈して，記述した内容を `<body>` 内の `<form>` タグの中に挿入する．
    - `<form>` タグに `method="post"` が設定されているため，`<form>` タグ内の `<input>` 要素等のデータがPOSTメソッドでサーバーに送信される．
- テンプレートで `{{ block styles }} {{ endblock }}` のブロックを使えば，挟まれた部分が HTML の `<head>` タグ内に置かれる．
    - `<style>` タグで直接 CSS を書ける．
    - CSSファイルを `<link rel="stylesheet" href="ここにURL">` や `<link rel="stylesheet" href="{{ static 'パス' }}">` として読み込むのも良い．
- テンプレートで `{{ block scripts }} {{ endblock }}` のブロックを使えば，挟まれた部分が HTML の `<body>` タグ内の一番下に置かれる．
    - `<script type="text/javascript">` タグで直接 JavaScript を書ける．
    - JavaScriptファイルを `<script src="ここにURL"></script>` や `<script type="text/javascript" src="{{ static 'パス' }}"></script>` として読み込むのも良い．
    - （ oTree が想定した使い方ではないが，） `<form>` タグ内に記述したくない要素を記述するのにも使えそう．
- 自分で親テンプレートファイルを作りそこで独自のブロックを定義したとき，子テンプレートファイルで親テンプレートファイルを `extends` すれば独自のブロックが使える．


### `with`
- テンプレートファイルにおいて変数を定義して展開できる．
- たとえば以下のようにして使う．  
  ```html
  {{ with ritoku = player.payoff }}
      <p>あなたの利得は{{ ritoku }}です．
  {{ endwith }}
  ```
- パーツのテンプレートファイルを `include` するときに2つ以上の変数を渡すときは，以下のようにすると良い．  
  ```html
  {{ with thisyear = 2022 }}{{ with thismonth = "May" }}
      {{ include "publicgoodsgame/instr.html" }}
  {{ endwith }}{{ endwith }}
  ```


### `formfield`
- `{{ formfield "変数名" }}` とすると，当該変数の入力フォームが以下のように展開される．
  ```html
  <div class="{{ classes }}">
      {{ if is_checkbox }}
          <!-- widget=widgets.CheckboxInput としている場合 -->
          {{ fld }}
          <label class="form-check-label" for="{{ fld.id }}">
              {{ label }}
          </label>
      {{ else }}
          <label class="col-form-label" for="{{ fld.id }}">
              {{ label }}
          </label>
          <div class="controls">
              {{ fld }}
          </div>
      {{ endif }}
      {{ if fld.description }}
          <!-- help_text に何か文字列を渡した場合 -->
          <p>
              <small>
                  <p class="form-text text-muted">
                      {{ fld.description }}
                  </p>
              </small>
          </p>
      {{ endif }}
      {{ if errors }}
          <!-- 一旦ページの入力フォームが送信されて oTree の検証に引っかかった場合 -->
          <div class="form-control-errors">
              {{ for error in errors }}
                  {{ error }}<br/>
              {{ endfor }}
          </div>
      {{ endif }}
  </div>
  ```
    - ↑ で `{{ fld }}` となっている部分は以下のような HTML タグが生成される．
        - チェックボックスの場合
        ```html
        <input type="checkbox" class="form-check-input" id="id_変数名" name="変数名" required value="y">
        ```
        - 記述フォームの場合
        ```html
        <input type="text" class="form-control" id="id_変数名" name="変数名" required value="">
        ```
        - `models.LongStringField` を使っている場合
        ```html
        <textarea class="form-control" id="id_変数名" name="変数名" required value=""></textarea>
        ```
        - ドロップダウンメニューの場合
        ```html
        <select class="form-select" id="id_変数名" name="変数名" required>
            <option value="">--------</option>
            <option value="1">A</option>
            <option value="2">B</option>
            <option value="3">C</option>
        </select>
        ```
        - ラジオボタンの場合
        ```html
        <div id="id_変数名" required>
            <div class="form-check">
                <input class="form-check-input" type="radio" id="id_変数名-0" name="変数名" required value="1">
                <label for="id_変数名-0">A</label>
            </div>
            <div class="form-check">
                <input class="form-check-input" type="radio" id="id_変数名-1" name="変数名" required value="2">
                <label for="id_変数名-1">B</label>
            </div>
            <div class="form-check">
                <input class="form-check-input" type="radio" id="id_変数名-2" name="変数名" required value="3">
                <label for="id_変数名-2">C</label>
            </div>
        </div>
        ```


### `formfield_errors`
- （ブラウザでの検証に通過した上で） oTree サーバーでの検証に失敗した際にエラーメッセージを展開できる．
- エラーメッセージを表示したい場所（入力フォームの下など）に `{{ formfield_errors "変数名" }}` を記述しておく．
- `{{ formfield "変数名" }}` を使ってフォームを実装する場合は， `{{ formfield_errors "変数名" }}` を記述しなくてもエラーメッセージを入力フォームの下で表示してくれる（ oTree 3 ではページタイトルの下側にまとめて表示されていた）．
- `{{ for }}` ループを使う場合は，たとえば以下のようにする．
  ```html
  {{ for eachfield in form }}
      <div>
          {{ eachfield.label }}
          {{ formfield_errors eachfield.name }}
      </div>
  {{ endfor }}
  ```
- oTree サーバーでの検証に引っかかった場合に以下の要素が追加されたページが表示される．
  ```html
  <div class="form-control-errors">ここにエラーメッセージ</div>
  ```
- エラーメッセージは 組み込み関数 `error_message()` や データモデルクラスの組み込みメソッド `変数名_error_message()` を使ってカスタマイズできる．
- [https://otree.readthedocs.io/en/latest/forms.html#raw-html-widgets](https://otree.readthedocs.io/en/latest/forms.html#raw-html-widgets)


### `formfields`
- `{{ formfields }}` は以下を記述することと同じ．
  ```html
  {{ for eachfield in form }}
      {{ formfield eachfield.name }}
  {{ endfor }}
  ```


### `next_button`
- `{{ next_button }}` と記述すれば， oTree サーバーが以下の `<button>` タグを生成してくれる．  
  ```html
  <button class="otree-btn-next btn btn-primary">次へ</button>
  ```


### `chat`
- `{{ chat }}` と記述すれば，チャット機能が実装される．
- [https://otree.readthedocs.io/en/latest/multiplayer/chat.html](https://otree.readthedocs.io/en/latest/multiplayer/chat.html)



## `form` オブジェクト
- `form` 自体はテンプレートタグではなく，デフォルトでテンプレートに渡される変数（オブジェクト）．
- ページクラスで `form_fields` に渡した変数に対応する入力フォームがすべて含まれている．
    - `{{ form.変数名 }}` は `{{ formfield "変数名" }}` のラベルを除いた入力フォーム部分のみが展開される．
        - HTMLタグの `name` 属性には変数名が入っている．
        - HTMLタグの `id` 属性には `"id_変数名"` なる文字列が入っている．
    - `{{ form.変数名.name }}` は変数名が展開される．
    - `{{ form.変数名.id }}` は `"id_変数名"` なる文字列が展開される．
    - `{{ form.変数名.label }}` は以下のHTMLタグが展開される．
    ```html
    <label for="id_変数名">データモデルで `label` に設定した文字列</label>
    ```
    - `{{ form.変数名.description }}` はデータモデルで `help_text` に設定した文字列が展開される．
    - `{{ form.変数名.errors }}` は oTree の検証に引っかかったときのエラーメッセージ（文字列）がリストに入った状態で展開される．
        - たとえば，以下のような使い方をする．
        ```html
        {{ if form.変数名.errors }}
            <!-- エラーが発生した場合 -->
            {{ form.変数名.errors.0 }}    {# ← リストの0番目にエラーメッセージ（文字列）が入っている #}
        {{ endif }}
        ```
- たとえば `{{ for }}` ループを使って以下のような実装が可能．
  ```html
  {{ for eachfield in form }}
      <div>
          {{ eachfield.label }}
          <input id="{{ eachfield.id }}" name="{{ eachfield.name }}">
      </div>
  {{ endfor }}
  ```
- 入力フォームがドロップダウンメニューの場合， `{{ form.変数名 }}` から更にパーツを取り出すことができる．
    - たとえば
    ```html
    <select class="form-select" id="id_変数名" name="変数名" required>
        {{ for eachopt in form.変数名 }}
            {{ eachopt }}
        {{ endfor }}
    </select>
    ```
    としたとき， `{{ eachopt }}` には，たとえば以下の HTML タグが展開される．
    ```html
    <option value="1">A</option>
    ```
- 入力フォームがラジオボタンの場合， `{{ form.変数名 }}` から更にパーツを取り出すことができる．
    - たとえば
    ```html
    <select class="form-select" id="id_変数名" name="変数名" required>
        {{ for eachopt in form.変数名 }}
            <p>
                {{ eachopt.label }}: {{ eachopt }}
            </p>
        {{ endfor }}
    </select>
    ```
    としたとき， `{{ eachopt }}` には，たとえば以下の HTML タグが展開される．
    ```html
    <input class="form-check-input" type="radio" id="id_変数名-0" name="変数名" required value="1">
    ```
    `{{ eachopt.label }}` には，たとえば以下の HTML タグが展開される．
    ```html
    <label for="id_変数名-0">A</label>
    ```
    - インデックス（パーツの `id` 属性で `"id_変数名-"` のあとに入る数字）は0から始まることに注意．



## コメントアウト

- テンプレート内で `{# #}` で囲った箇所はコメントとして， oTree サーバーは無視する．
    - 行の中では  
    ```html
    <p>初期保有は{{ C.ENDOWMENT }}です．</p> {# ← ENDOWMENTが流し込まれる． #}
    ```
    - 複数行に渡るときは  
    ```html
    {#
    <p>初期保有は{{ C.ENDOWMENT }}です．</p>
    <p>意思決定してください．</p>
    #}
    ```
- HTMLのコメント `<!-- コメント -->` については， oTree サーバーは無視せず，そのままクライアントに渡す．したがって，HTMLのコメントは画面には表示されないが，検証モードなどを使えばコメントの中身が見えてしまう．



{% endraw %}
