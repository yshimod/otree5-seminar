{% raw %}
【第5回】 2022年6月9日

- 今後の予定
    - 第4回: 同時手番ゲーム（oTreeプログラミングの基礎）
    - 第5回（今日）: 逐次手番ゲーム（参加者ごと表示させる画面を変える）
    - 第回: 繰り返しゲーム（プレイヤーのシャッフル，タイムアウト）
    - 第回: ダブルオークション（JavaScriptとライブページ，ExtraModel）
    - 第回: 質問紙調査（画面のデザイン，CSS，Bootstrap）
    - 第回: 補遺



# 参加者ごと表示させる画面を変える

** 大切なこと: 自分で同じことを繰り返し書かない！（DRY原則） **


- テンプレート内において変数を展開したりテンプレートタグを使って条件分岐させたりする．
- プレイヤーごと入力フォーム（変数名）が異なるとき，`__init__.py` のページクラスのメソッド `is_displayed()` で制御する．
- ライブページを使って動的に画面の中身を書き換える（サーバーとの通信あり）．
    - 後の回で説明します．
- JavaScriptを使って動的に画面の中身を書き換える（サーバーとの通信なし）．
    - 後の回で説明します．




## テンプレートにおけるプログラミング

### 変数の展開

- テンプレート内に `{{ 変数名 }}` と記述すると，oTreeサーバーはその箇所に変数を展開し，参加者には具体的な変数の中身が代入されて表示される．
- たとえば，`C.ENDOWMENT` の値が `1000` であるとして，テンプレートに  
    ```html
    <p>あなたの初期保有は{{ C.ENDOWMENT }}ポイントです．</p>
    ```
    と記述したとき，クライアントがoTreeサーバーから受け取るhtmlデータは，
    ```html
    <p>あなたの初期保有は1000ポイントです．</p>
    ```
    となる．
- 以下のインスタンスオブジェクトが使える．
    - `player`: the player currently viewing the page
    - `group`: the group the current player belongs to
    - `subsession`: the subsession the current player belongs to
    - `participant`: the participant the current player belongs to
    - `session`: the current session
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
            そしてテンプレートにおいて `{{ ENDOWMENT_times_10 }}` と記述すれば，`C.ENDOWMENT` を10倍した値が画面で表示される．


### 変数に使えるフィルター

#### `default`
- `{{ x | default("something") }}` と記述すると，変数 `x` の値が `None` のときに画面では「something」が表示される．
- そもそも変数 `x` が定義されていないければエラーとなる．

#### `escape`
- `{{ x | escape }}` と記述すると，変数 `x` の文字列の中に含まれる `&`， `<`， `>`， `"`， `'` をHTMLセーフな文字列に変換する．
- たとえば `x = "<div style='font-weight: bold;'>AAAAA</div>"` のとき， `<p>{{ x | escape }}</p>` は `<p>&lt;div style=&#x27;font-weight: bold;&#x27;&gt;AAAAA&lt;/div&gt;</p>` と展開され，画面にはHTMLタグがそのまま表示される．フィルターを使わないと，HTMLタグがそのまま展開されるので，画面では「AAAAA」が太字で表示される．
- （公式ドキュメントの説明） If you want to escape content (e.g. displaying an untrusted string to a different player), you should use the `|escape` filter.

#### `length`
- 変数が `x = [0, 1, 2]` のとき， `{{ x | length }}` と記述すると，画面では変数 `x` の長さである3が表示される．

#### `c` ないし `cu`
- `{{ x | cu }}` と記述すると，変数 `x` をoTreeの通貨型に変換したもの（つまり `cu(x)` ）が画面で表示される．数値が丸められるだけでなく単位（「円」や「ポイント」）も付け加えられる．

#### `json`
- 変数が `x = {'key1': 10, 'key2': 20}` のような辞書型のとき， `{{ x | json }}` と記述すると，JSONにエンコードされた文字列（Pythonで `json.dumps(x)` とした返り値）が表示される．

#### `to0`
- `{{ x | to0 }}` と記述すると，変数 `x` の数値が切り下げられ，整数で表示される．
- `to1` は小数第1位まで， `to2` は小数第2位まで表示される．`to3` は存在しない．



### テンプレートタグ

#### `if`
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


#### `for`
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
- 使うリストなどが空だった場合の例外処理は `{{ empty }}` を使って記述．
    ```html
    {{ for v in C.mylist }}
        <p>{{ forloop.counter }}番目の要素は{{v}}．</p>
    {{ empty }}
        <p>このリストは空です．</p>
    {{ endfor }}
    ```


#### `include`
- テンプレート中で，別の（サブ）テンプレートファイルを読み込むことができる．
- たとえば意思決定画面でもインストラクションを表示させたいときに，インストラクションの本文（タイトルブロックや「次へ」ボタンなどを除いた部分）だけを別のHTMLファイル（同じアプリ `publicgoodsgame` のディレクトリ内の `instr.html` ）として作っておく．作ったHTMLを以下のようにして読み込むと，読み込んだファイル全体が `{{ include ... }}` を記述した部分に展開される．
    ```html
    {{ include "publicgoodsgame/instr.html" }}
    ```
- `include` するファイルのテンプレートに変数を渡すこともできる．
    - たとえば `instr.html` の中で `{{ thisyear }}` と記述しておき，  
        ```html
        {{ include "publicgoodsgame/instr.html" with thisyear=2022 }}
        ```
        と記述すると，`instr.html` の `{{ thisyear }}` に「2022」を展開した上で，`instr.html` 全体が `{{ include ... }}` を記述した部分に展開される．


#### `static`
- プロジェクトディレクトリ直下の `_static` ディレクトリに置いたものを読み込むことができる．


#### `extends`


#### `block`


#### `with`


#### `formfield`


#### `formfield_errors`


#### `formfields`


#### `next_button`


#### `chat`



### コメントアウト

- テンプレート内で `{# #}` で囲った箇所はコメントとして，oTreeサーバーは無視する．
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
- HTMLのコメント `<!-- コメント -->` については，oTreeサーバーは無視せず，そのままクライアントに渡す．したがって，HTMLのコメントは画面には表示されないが，検証モードなどを使えばコメントの中身が見えてしまう．



### oTree3との違い

- oTree3はDjangoがベースであったため，Djangoの組み込みフィルターが使えたが，oTree5では，Djangoの組み込みフィルターが（ほとんど）使えなくなった．oTree3からoTree5へ移行するときは注意．多用されていたと思われるフィルターは以下:
    - `floatformat:0`: 整数への（銀行家の）丸め．oTree5では `to0` に変更されたが，切り下げとなることに注意．
    - `add`: テンプレートで変数の足し算ができた（たとえば `{{ x|add:"1" }}` とすれば xに1を足した数値が表示されていた）．
    - `date`: 変数が `datetime` オブジェクトであれば，日付が人間が読める書式に変換されて表示されていた．
- テンプレートタグのデリミタが `{% %}` から `{{ }}` に変更され，テンプレートタグと変数の記述方法が縮退してしまった．
    - oTree5でも互換性のために `{% %}` は使える．ただし，`{%` と `{{`， `%}` と `}}` をそれぞれ区別しないという実装のようなので，`"` と `'` の関係のようにネストさせて使えるというわけではない．



## `is_displayed()` を使ったページの制御

[https://otree.readthedocs.io/en/latest/pages.html#is-displayed](https://otree.readthedocs.io/en/latest/pages.html#is-displayed)

- ページクラスの中に `is_displayed()` を定義する．引数は `player` オブジェクト．返り値が `True` のとき，そのプレイヤーに表示される．
- 公式ドキュメントのチュートリアル「信頼ゲーム」を参照．[https://otree.readthedocs.io/ja/latest/tutorial/part3.html](https://otree.readthedocs.io/ja/latest/tutorial/part3.html)



{% endraw %}
