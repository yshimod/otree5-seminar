{% raw %}
【第8回】 2022年7月14日


- 今後の予定
    - 第8回: 連続時間ダブルオークション（ライブページとExtraModel）
    - 第回: 補遺




# ExtraModel ・ JavaScript ・ ライブページ


## ExtraModel

- [ExtraModel](otree_ref/init.html#extramodel)
- [`custom_export()`](otree_ref/init.html#customexport)




## JavaScript

- 画面の表示内容を動的に書き換えたり，入力フォームの値の検証を自前で実装したりする場合には JavaScript を使う．


- JavaScript を使うには，以下の2通り．

    - テンプレートファイルに `{{ block scripts }} {{ endblock }}` を置き，その中に`<script>` タグで JavaScript コードを書く．

    ```html
    {{ block title }}
        タイトル
    {{ endblock }}

    {{ block content }}
        {{ formfields }}
        {{ next_button }}
    {{ endblock }}

    {{ block scripts }}
        <script>
            alert("Hello World!");
        </script>
    {{ endblock }}
    ```

    - JS ファイル（ `myscripts.js` ）を作成して， `_static/global` などに入れておき，それを読み込む．

    ```html
    {{ block title }}
        タイトル
    {{ endblock }}

    {{ block content }}
        {{ formfields }}
        {{ next_button }}
    {{ endblock }}

    {{ block scripts }}
        <script src="{{ static 'global/myscripts.js' }}"></script>
    {{ endblock }}
    ```

    JS ファイル `myscripts.js` の中身は以下．

    ```js
    alert("Hello World!");
    ```


- JavaScript で入力フォームの要素（たとえば値）を操作する場合，通常は `document.querySelector('[name=フィールド名]')` などを使えばよいが， oTree も多少便利な機能 `formInputs` を用意しているため使える．

    - たとえばフィールド名が `xyz` である入力フォームの値を取り出すには以下のようにする．
    ```JavaScript
    const el_xyz = formInputs.xyz;
    const original_value = el_xyz.value;
    console.log("取り出した値:", original_value);
    ```

    - たとえばフィールド名が `xyz` である入力フォームの値を更新するには以下のようにする．
    ```JavaScript
    const el_xyz = formInputs.xyz;
    el_xyz.value = 999;
    ```

    - なぜか公式ドキュメントからは記述が消えてしまったため，今後実装が変更となる（ないし機能が廃止になる）可能性がある．

        - [著者によるフォーラムのポスト](https://groups.google.com/g/otree/c/3fRobN1YZTU/m/kCKws1VZAQAJ)


- oTree サーバーから JavaScript に変数を渡すには，ページクラスの組み込みメソッド `js_vars()` を使う．

    - たとえば

    ```python
    @staticmethod
    def js_vars(player: Player):
        return dict(
            testlist = list(range(0, 10, 2))
        )
    ```

    としたとき，クライアントに送信される HTML （タイトルの要素の直後，コンテンツブロックの要素の直前）には自動的に

    ```html
    <script>var js_vars = {"testlist": [0, 2, 4, 6, 8]};</script>
    ```

    と展開されている．渡した変数を JavaScript で使うには，たとえば以下のようにする．

    ```html
    {{ block scripts }}
        <script>
            const testlist = js_vars.testlist;
            testlist.forEach(el => {
                console.log(el);
            });
        </script>
    {{ endblock }}
    ```


- （作例）MPLでスイッチングポイントをクリックしたときに，自動的にラジオボタンが切り替える JavaScript コード:

    - 定数に `optR = [200, 250, 300, 350, 400]` を設定しておく．

    - player モデルで `switching_point` なるフィールドを ` models.IntegerField()` で定義しておく．MPLの1行ずつデータを取りたい場合は各行に相当するフィールドを定義する．

  %accordion%テンプレート%accordion%
  ```html
  {{ block title }}
      タイトル
  {{ endblock }}
  {{ block content }}
      <table class="table table-striped table-hover">
          <thead>
              <tr>
                  <th class="text-end">オプションL</th>
                  <th></th>
                  <th></th>
                  <th class="text-start">オプションR</th>
              </tr>
          </thead>
          <tbody>
              {{ for v in C.optR }}
                  <tr>
                      <td class="text-end">
                          50%の確率で650円
                      </td>
                      <td>
                          <input class="text-center BtnChoice BtnL" type="radio" name="mpl_{{ forloop.counter0 }}" id="L_mpl_{{ forloop.counter0 }}" value="{{ forloop.counter0 }}">
                      </td>
                      <td>
                          <input class="text-center BtnChoice BtnR" type="radio" name="mpl_{{ forloop.counter0 }}" id="R_mpl_{{ forloop.counter0 }}" value="{{ forloop.counter0 }}">
                      </td>
                      <td class="text-start">
                          100%の確率で{{ v }}円
                      </td>
                  </tr>
              {{ endfor }}
          </tbody>
      </table>
      <input type="hidden" name="switching_point" id="id_switching_point" value="">
      {{ next_button }}
  {{ endblock }}
  {{ block scripts }}
      <script>
          const listlength = 5;    // 本当は js_vars から受け取ると良い．
          const btnsAll = document.querySelectorAll('.BtnChoice');
          btnsAll.forEach(el => {
              el.addEventListener('click', function() {
                  let switchingPoint;
                  const tmpCol = el.id.slice(0,1);
                  if (tmpCol == "L") {
                      switchingPoint = parseInt(el.name.slice(4), 10) + 1;
                  }
                  else if (tmpCol == "R") {
                      switchingPoint = parseInt(el.name.slice(4), 10);
                  }
                  document.getElementById("id_switching_point").value = switchingPoint;
                  let tmpRow;
                  for (var i = 0; i < listlength; i++) {
                      tmpRow = 'mpl_' + i;
                      if (i < switchingPoint) {
                          document.getElementsByName(tmpRow)[0].checked = true;
                          document.getElementsByName(tmpRow)[1].checked = false;
                      } else {
                          document.getElementsByName(tmpRow)[0].checked = false;
                          document.getElementsByName(tmpRow)[1].checked = true;
                      }
                  }
              }, false);
          });
      </script>
  {{ endblock }}
  ```
  %/accordion%

  <p class="codepen" data-height="500" data-theme-id="dark" data-default-tab="result" data-slug-hash="VwXLxbw" data-editable="true" data-user="yshimod" style="height: 500px; box-sizing: border-box; display: flex; align-items: center; justify-content: center; border: 2px solid; margin: 1em 0; padding: 1em;">
      <span>See the Pen <a href="https://codepen.io/yshimod/pen/VwXLxbw">
      oTree day8-1</a> by yshimod (<a href="https://codepen.io/yshimod">@yshimod</a>)
      on <a href="https://codepen.io">CodePen</a>.</span>
  </p>


- oTree ではデフォルトで jQuery が読み込まれている．

    - [jQuery のドキュメント](https://api.jquery.com/)

    - jQuery のバージョンは 3.2.1 （oTree v5.8.5 において）．

    - JavaScript コードの中で `jQuery()` や `$()` で jQuery を呼び出す．

        - たとえば `document.getElementById("id_something")` は `$("#id_something")` と似た機能．ただし後者は jQuery のオブジェクトが返ってくる．


- JavaScript のライブラリを自分で読み込んで使っても良い．

    - ライブラリをインストールするには， CDN を使うか，ダウンロードしたライブラリのコードを `_static` ディレクトリに置いておく．

    - oTree 公式ドキュメントでは [HighCharts](https://www.highcharts.com/) を使っている．  
    [https://otree.readthedocs.io/en/latest/templates.html#charts](https://otree.readthedocs.io/en/latest/templates.html#charts)

    - （日本語ドキュメントが豊富な） [Chart.js](https://www.chartjs.org/) も有用．

    %accordion%テンプレート%accordion%
    ```html
    {{ block title }}
        タイトル
    {{ endblock }}

    {{ block content }}
        <div class="row align-items-center">
            <div class="col-8">
                <div style="width: 480px; max-width: 100%; margin: auto;">
                    <canvas id="myChart" style="height: 300px; width: 100%;"></canvas>
                </div>
            </div>
            <div class="col-2">
                <button type="button" class="btn btn-danger btn-lg" onclick="updateFunction();">ボタン</button>
            </div>
        </div>

        {{ next_button }}
    {{ endblock }}

    {{ block scripts }}
        <script src="https://cdn.jsdelivr.net/npm/chart.js@3.8.0/dist/chart.min.js" integrity="sha256-cHVO4dqZfamRhWD7s4iXyaXWVK10odD+qp4xidFzqTI=" crossorigin="anonymous"></script>
        <script>
            const ctx = document.getElementById("myChart");
            const myChart = new Chart(ctx, {
                type: 'bar',
                data: {
                    labels: [''],
                    datasets: [
                        {
                            label: 'ド',
                            data: [20],
                            backgroundColor: "#ffcf00",
                            stack: 'stack1'
                        },
                        {
                            label: 'イ',
                            data: [30],
                            backgroundColor: "#dd0000",
                            stack: 'stack1'
                        },
                        {
                            label: 'ツ',
                            data: [50],
                            backgroundColor: "#000000",
                            stack: 'stack1'
                        }
                    ],
                },
                options: {
                    plugins: {
                        title: {
                            display: true,
                            text: 'ドイツの割合'
                        },
                        legend: {
                            position: 'bottom',
                            onClick: function () { return false }
                        },
                    },
                    responsive: true,
                    tooltip: {
                        mode: 'index'
                    },
                    hover: true,
                    scales: {
                        y: {
                            stacked: true,
                            max: 100
                        }
                    }
                }
            });

            function updateFunction() {
                const yellow = Math.random();
                const red = Math.random();
                const black = Math.random();
                const totnum = yellow + red + black;

                myChart.data.datasets[0].data[0] = 100 * yellow / totnum;
                myChart.data.datasets[1].data[0] = 100 * red / totnum;
                myChart.data.datasets[2].data[0] = 100 * black / totnum;
                myChart.update();
            }
        </script>
    {{ endblock }}
    ```
    %/accordion%

    <p class="codepen" data-height="500" data-theme-id="dark" data-default-tab="result" data-slug-hash="XWEbqRy" data-editable="true" data-user="yshimod" style="height: 500px; box-sizing: border-box; display: flex; align-items: center; justify-content: center; border: 2px solid; margin: 1em 0; padding: 1em;">
        <span>See the Pen <a href="https://codepen.io/yshimod/pen/XWEbqRy">
        oTree day8-2</a> by yshimod (<a href="https://codepen.io/yshimod">@yshimod</a>)
        on <a href="https://codepen.io">CodePen</a>.</span>
    </p>


- [jsPsych](https://www.jspsych.org/) を使う場合は工夫が必要．  
実装例:
    - [https://github.com/yshimod/jspsych_on_otree](https://github.com/yshimod/jspsych_on_otree)
    - [https://www.otreehub.com/projects/jspsych-on-otree/](https://www.otreehub.com/projects/jspsych-on-otree/)




## Live page

- [https://otree.readthedocs.io/en/latest/live.html](https://otree.readthedocs.io/en/latest/live.html)
- Udemyの講座 [https://www.udemy.com/course/learn-otree/](https://www.udemy.com/course/learn-otree/) の Live page に関するレクチャー（「Advanced oTree Features」の「Real Time Interaction between Participants - oTree LivePages」）は無料で見れる．


- 通常，クライアントと oTree サーバーのデータのやり取りは HTTP プロトコルによって行われる．

    - 画面を表示するとき（実験刺激を呈示するとき）には，クライアントが oTree サーバー（ Web サーバー ）に GET メソッドでリクエストを送信し，リクエストに応じてサーバーがクライアントに画面の内容（ HTML文書 ）を送信し返す．なお， HTML は Python によってテンプレートファイルをもとに動的に生成される．

    - 次のページへ進むときには，クライアントがサーバーに POST メソッドで，入力フォームの値を含めたリクエストを送信し，これを受けてサーバーがデータを受け取りつつ，次の画面の内容を送信し返す．


- HTTP プロトコルはクライアントからリクエストを送信することによって通信が始める方法であり，サーバー側から通信を始めることはできない．この問題を解決する，すなわちサーバー側から通信を開始する方法として WebSocket プロトコルが存在する．

- oTree において WebSocket による通信を利用するためには Live page と呼ばれる機能を使う．

- 「サーバー側から通信を開始する方法」があると言っても， oTree の Live page は管理者（実験者）の任意のタイミングでデータを送信することはできず，セッションの参加者の操作をきっかけとしてデータを送信することになる．



### （JavaScript） Live page を作動させる `liveSend()`

- テンプレートにおいて JavaScript で `liveSend()` を呼び出すと， oTree サーバーで `live_method()` が作動する．

- `liveSend()` の引数として JavaScript のオブジェクトを渡すことができる．数値，文字列だけでも良い．

- たとえばクリックのタイミングで発火させる場合には以下のように実装すれば良い．
  ```html
  {{ block content }}
      <input type="number" id="sendnumber">
      <button type="button" id="sendbutton">送信</button>
  {{ endblock }}

  {{ block scripts }}
      <script>
          function my_send_func() {
              const sendnumber_value = document.getElementById("sendnumber").value;
              const send_obj = {
                  "number": sendnumber_value
              };
              liveSend(send_obj);
          }

          window.onload = function() {
              document.getElementById("sendbutton").addEventListener("click", my_send_func, false);
          };
      </script>
  {{ endblock }}
  ```

- Chrome でどんなデータが送信されたのかを確認するには，デベロッパーツールを開き，Networkタブで，名前が `live` から始まる要素（Headers Request URL が `ws://` か `wss://` から始まっている）の Messages を見れば良い．



### （Python） クライアントにデータを送信する `live_method()`

- ページクラスの組み込みメソッド `live_method()` を Live page を使うページのクラスで定義する．

- 引数は `player` と `data` ． `data` には `liveSend()` の引数に渡したものが入っている．

- 宛先をキーとして，送信するデータを格納した辞書オブジェクトを関数の返り値にすると，指定した宛先（当該 player 以外でもよい）にデータを送信できる．

- 宛先は `id_in_group` の自然数で指定する．
    - `liveSend()` を実行した player 自身にデータを送る場合は `{player.id_in_group: 値}` を返す．
    - group の全員に送信する場合のキーは `0`．
    - 他の group や subsession 全体へは送信できない．
    - キーが数値ないし変数のときには `dict()` が使えないので， `{ }` で辞書オブジェクトを定義する．


- たとえば `liveSend()` で `{"number": "3"}` なる値がサーバーに送信された後，中身の数値を10倍したものを， group の他人 player 全員に送信するためには以下のようにする．
  ```python
  @staticmethod
  def live_method(player: Player, data):
      received_number = int(data["number"])    ## 数値 3 が入る．
      return_number = received_number * 10

      return { p.id_in_group: {"return_number": return_number} for p in player.get_others_in_group() }
      ## 3人グループで自分の id_in_group が 2 のとき， ↑ の辞書オブジェクトは
      ## { 1: {"return_number": 30}, 3: {"return_number": 30} }
      ## となっている．
  ```



### （JavaScript） サーバーから送信されたデータを受け取る `liveRecv()`

- テンプレートにおいて JavaScript で `liveRecv()` を定義しておくと， oTree サーバーで `live_method()` が作動してデータが送信されたタイミングで自動的に `liveRecv()` が発火する．

- `liveRecv()` の引数として `data` を受け取る． `live_method()` の返り値である辞書オブジェクトの値のみが入っている（宛先のキーは送信されない）．

- たとえば `{"return_number": 30}` なるデータを受け取り，このタイミングで受け取った数字を画面に表示させるためには以下のようにする．
  ```html
  {{ block content }}
      <input type="number" id="sendnumber">
      <button type="button" id="sendbutton">送信</button>
      <div>
          相手から受け取った値: <span id="pushednumber"></span>
      </div>
  {{ endblock }}

  {{ block scripts }}
      <script>
          function liveRecv(data) {
              const pushednumber = data["return_number"];
              document.getElementById("pushednumber").innerText = pushednumber;
          }

          // 以下は liveSend() のために既に記述していた部分．
          function my_send_func() {
              const sendnumber_value = document.getElementById("sendnumber").value;
              const send_obj = {
                  "number": sendnumber_value
              };
              liveSend(send_obj);
          }

          window.onload = function() {
              document.getElementById("sendbutton").addEventListener("click", my_send_func, false);
          };
      </script>
  {{ endblock }}
  ```

- Chrome でどんなデータを受け取ったのかを確認するには，デベロッパーツールを開き，Networkタブで，名前が `live` から始まる要素（Headers Request URL が `ws://` か `wss://` から始まっている）の Messages を見れば良い．




## 三上さん特別レクチャー



<script async src="https://cpwebassets.codepen.io/assets/embed/ei.js"></script>
{% endraw %}
