{% raw %}
【第8回】 2022年7月14日

# ExtraModel ・ JavaScript ・ Live page


## ExtraModel

- [詳細はこちら](otree_ref/init.html#extramodel)

<p class="ytubevideo"><iframe width="560" height="315" src="https://www.youtube.com/embed/dimFNK-jyOM?rel=0&enablejsapi=1&origin=https://yshimod.github.io/" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe></p>



### `custom_export()`

- [詳細はこちら](otree_ref/init.html#customexport)

<p class="ytubevideo"><iframe width="560" height="315" src="https://www.youtube.com/embed/-y3MuhCiLic?rel=0&enablejsapi=1&origin=https://yshimod.github.io/" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe></p>




## JavaScript

- 画面の表示内容を動的に書き換えたり，入力フォームの値の検証を自前で実装したりする場合には JavaScript を使う．


### JavaScript の記述方法

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


### `js_vars()`

- oTree サーバーから JavaScript に変数を渡すには，ページクラスの組み込みメソッド `js_vars()` を使う．

- たとえば以下のように設定する．

  ```python
  @staticmethod
  def js_vars(player: Player):
      return dict(
          testlist = list(range(0, 10, 2))
      )
  ```

- 返り値の辞書オブジェクトの中の値は，単なる数値，文字列，リスト，辞書オブジェクト，など， JSON 文字列化できるものに限られる．

- 渡した変数を JavaScript で使うには，自動的に定義される `js_vars` なるオブジェクトから取り出す．たとえば以下のように記述する．

  ```html
  {{ block scripts }}
      <script>
          const testlist = js_vars.testlist;    // js_vars から 自分が定義した testlist を取り出す．

          // リストの要素を一つずつ console.log する．
          testlist.forEach(el => {
              console.log(el);
          });
      </script>
  {{ endblock }}
  ```


### jQuery

- oTree ではデフォルトで jQuery が読み込まれている．

- [jQuery のドキュメント](https://api.jquery.com/)

- jQuery のバージョンは 3.2.1 （oTree v5.8.5 において）．

- JavaScript コードの中で `jQuery()` や `$()` で jQuery を呼び出す．

    - たとえば `document.getElementById("id_something")` は `$("#id_something")` と似た機能．ただし後者は jQuery のオブジェクトが返ってくる．


### （作例） JavaScript で（おせっかいな） MPL を実装する．

- MPLでスイッチングポイントをクリックしたときに，自動的にラジオボタンが切り替える JavaScript コード:

    - コード（js_samples アプリ） [https://github.com/yshimod/otree_survey](https://github.com/yshimod/otree_survey)
    - デモページ（「JavaScriptを使った作例」の1ページ目） [https://otree-seminar-survey-sample.herokuapp.com/demo](https://otree-seminar-survey-sample.herokuapp.com/demo)

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



### ライブラリの利用

- JavaScript のライブラリを自分で読み込んで使っても良い．

- ライブラリをインストールするには， CDN を使うか，ダウンロードしたライブラリのコードを `_static` ディレクトリに置いておく．

- oTree 公式ドキュメントでは [HighCharts](https://www.highcharts.com/) を使っている．  
[https://otree.readthedocs.io/en/latest/templates.html#charts](https://otree.readthedocs.io/en/latest/templates.html#charts)

- （日本語ドキュメントが豊富な） [Chart.js](https://www.chartjs.org/) も有用．
    - コード（js_samples アプリ） [https://github.com/yshimod/otree_survey](https://github.com/yshimod/otree_survey)
    - デモページ（「JavaScriptを使った作例」の2ページ目） [https://otree-seminar-survey-sample.herokuapp.com/demo](https://otree-seminar-survey-sample.herokuapp.com/demo)

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
      <script src="https://cdn.jsdelivr.net/npm/chart.js@3.8.0/dist/chart.min.js" tegrity="sha256-cHVO4dqZfamRhWD7s4iXyaXWVK10odD+qp4xidFzqTI=" crossorigin="anonymous"></script>
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

  <p class="codepen" data-height="500" data-theme-id="dark" data-default-tab="result" data-slug-hash="XWEbqRy" ta-editable="true" data-user="yshimod" style="height: 500px; box-sizing: border-box; display: flex; align-items: center; stify-content: center; border: 2px solid; margin: 1em 0; padding: 1em;">
      <span>See the Pen <a href="https://codepen.io/yshimod/pen/XWEbqRy">
      oTree day8-2</a> by yshimod (<a href="https://codepen.io/yshimod">@yshimod</a>)
      on <a href="https://codepen.io">CodePen</a>.</span>
  </p>


- [jsPsych](https://www.jspsych.org/) を使う場合は工夫が必要．  
実装例:
    - [https://github.com/yshimod/jspsych_on_otree](https://github.com/yshimod/jspsych_on_otree)
    - [https://www.otreehub.com/projects/jspsych-on-otree/](https://www.otreehub.com/projects/jspsych-on-otree/)


<p class="ytubevideo"><iframe width="560" height="315" src="https://www.youtube.com/embed/VMMWpYfzvsc?rel=0&enablejsapi=1&origin=https://yshimod.github.io/" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe></p>



## Live page

- [https://otree.readthedocs.io/en/latest/live.html](https://otree.readthedocs.io/en/latest/live.html)
- Udemyの講座 [https://www.udemy.com/course/learn-otree/](https://www.udemy.com/course/learn-otree/) の Live page に関するレクチャー（「Advanced oTree Features」の「Real Time Interaction between Participants - oTree LivePages」）は無料で見れる．


- 通常，クライアントと oTree サーバーのデータのやり取りは HTTP プロトコルによって行われる．

    - 画面を表示するとき（実験刺激を呈示するとき）には，クライアントが oTree サーバー（ Web サーバー ）に GET メソッドでリクエストを送信し，リクエストに応じてサーバーがクライアントに画面の内容（ HTML文書 ）を送信し返す．なお， HTML は Python によってテンプレートファイルをもとに動的に生成される．

    - 次のページへ進むときには，クライアントがサーバーに POST メソッドで，入力フォームの値を含めたリクエストを送信し，これを受けてサーバーがデータを受け取りつつ，次の画面の内容を送信し返す．


- HTTP プロトコルはクライアントからリクエストを送信することによって通信が始める方法であり，サーバー側から通信を始めることはできない．この問題を解決する，すなわちサーバー側から通信を開始する方法として WebSocket プロトコルが存在する．

- oTree において WebSocket による通信を利用するためには Live page と呼ばれる機能を使う．

- 「サーバー側から通信を開始する方法」があると言っても， oTree の Live page は管理者（実験者）の任意のタイミングでデータを送信することはできず，セッションの参加者の操作をきっかけとしてデータを送信することになる．

- WebSocket の通信内容を Chrome で確認するには，デベロッパーツールを開き，Networkタブで，名前が `live` から始まる要素（ Type が websocket ）の Messages を見れば良い．


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

- JavaScript で `liveSend()` を呼び出しても，（後述の） `live_method()` がページクラスで定義されていなければ実行されない．



### （Python） クライアントにデータを送信する `live_method()`

- ページクラスの組み込みメソッド `live_method()` を Live page を使うページのクラスで定義する．

- 引数は `player` と `data` ． `data` には `liveSend()` の引数に渡したものが入っている．

- 宛先をキーとして，送信するデータを格納した辞書オブジェクトを関数の返り値にすると，指定した宛先（当該 player 以外でもよい）にデータを送信できる．

- 宛先は `id_in_group` の自然数で指定する．
    - `liveSend()` を実行した player 自身にデータを送る場合は `{player.id_in_group: 値}` を返す．
    - group の全員に送信する場合のキーは `0`．
    - 他の group や subsession 全体へは送信できない．
    - キーが数値ないし変数のときには `dict()` が使えないので， `{ }` で辞書オブジェクトを定義する．


- たとえば `liveSend()` で `{"number": "3"}` なる値がサーバーに送信された後，中身の数値を10倍したものを， group の全員に送信するためには以下のようにする．

  ```python
  @staticmethod
  def live_method(player: Player, data):
      received_number = int(data["number"])    ## 数値 3 が入る．
      return_number = received_number * 10

      return {
          0: {"return_number": return_number}
      }
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
          受け取った値: <span id="pushednumber"></span>
      </div>
  {{ endblock }}

  {{ block scripts }}
      <script>
          function liveRecv(data) {
              const pushednumber = data.return_number;
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



<p class="ytubevideo"><iframe width="560" height="315" src="https://www.youtube.com/embed/6SLXuQuXvrk?rel=0&enablejsapi=1&origin=https://yshimod.github.io/" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe></p>



## 三上さん特別レクチャー

- [信州大 三上亮さん](https://sites.google.com/view/ryomikami/japanese) に Live page を解説していただきました．ありがとうございます．

- [スライド](https://drive.google.com/file/d/1tjHZmcoO_Hfdf9mo6skRx_JlAGpi5FQJ/view?usp=sharing)

- 連続時間ダブルオークションのコード:  
[https://github.com/oTree-org/more-demos](https://github.com/oTree-org/more-demos) （double_auction アプリ）
    - コメントを付けたコード [https://github.com/RyoMikami/otree5_lecture](https://github.com/RyoMikami/otree5_lecture)
    - デモページ [https://otree-more-demos.herokuapp.com/](https://otree-more-demos.herokuapp.com/)


<p class="ytubevideo"><iframe width="560" height="315" src="https://www.youtube.com/embed/iyjIbaW_yXQ?rel=0&enablejsapi=1&origin=https://yshimod.github.io/" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe></p>


- （下平注） このサンプルのコードにはいくつか注意するべき点があります． Live page の勉強用や，授業におけるデモンストレーションとして使うならまだしも，ダブルオークションの研究で使う場合は以下の点を検討した方が良さそうです．

    - [`__init__.py`](https://github.com/oTree-org/more-demos/blob/master/double_auction/__init__.py) の61行目から66行目において，売買が成立する buyer と seller のペアを探す関数を定義されていますが， `id_in_group` の昇順に一人ずつ取引が成立するか確認して，最初に見つかったペアを返す実装となっています．通常は，売りオファーの安い順・買いオファーの高い順，およびオファーの早い順，に取引を成立させていく必要があると思われます．

    - [`__init__.py`](https://github.com/oTree-org/more-demos/blob/master/double_auction/__init__.py) の53行目から58行目で Transaction なる ExtraModel を定義し，85行目から91行目において取引の内容（誰とどの価格で取引したか，など）を Transaction に記録しています．留保価格/生産費用（break_even_point），オファー（current_offer），利得（payoff）は player のフィールドが定義され，そこに記録されているのですが，価格は記録されていません．また， `custom_export()` で Transaction の中身を CSV ファイルに出力するような実装が行われていません．したがって，このままの状態では価格を直接確認することはできません．複数個の財を購入した buyer については，利得と留保価格から価格を逆算することもできません． z-Tree の contracts テーブルのような，1レコードに取引時刻，ペアのID，取引価格，を記録したテーブルは `custom_export()` を使って自分で実装しなければなりません．

        - たとえば以下のような実装が考えられます．

        ```python
        def custom_export(players: list[Player]):
            yield [
                "セッションコード",
                "groupのid_in_subsession",
                "ラウンド",
                "buyerのid_in_subsession",
                "sellerのid_in_subsession",
                "価格",
                "経過秒"
            ]

            for p in players:
                if p.is_buyer:
                    records_list: list[Transaction] = Transaction.filter(buyer = p)
                    for record in records_list:
                        yield [
                            record.group.session.code,
                            record.group.id_in_subsession,
                            record.group.round_number,
                            record.buyer.id_in_subsession,
                            record.seller.id_in_subsession,
                            record.price,
                            record.seconds
                        ]
        ```

    - [`Trading.html`](https://github.com/oTree-org/more-demos/blob/master/double_auction/Trading.html) （JavaScript部分）の99行目から101行目において， seller が財を持っていない場合にオファーを送信するボタンを押せないように変更するコードが記述されています．これ自体に問題は無いのですが， buyer が既に財を1つ手に入れている場合に対する処理は記述されておらず，結局， buyer は何個でも財を買える状態になっています．もしも buyer が買える財の数を1つだけにする場合には自分で実装する必要があります．

        - なお下平は，てっきり buyer は1つだけ買えるものと勘違いしていたため，（島田さんの授業で実施した）実験データを見て「複数個買えとる buyer おるがや．バグか！」と早とちりし，RAさんも巻き込んで検証に多大な時間をかけてしまいました．たとえば，参加者が多い場合に，ある参加者のオファーがサーバーに飛んできてマッチングを計算している最中に，他の参加者のオファーが飛んできて，おかしくなっているのではないか，などと，当てずっぽうの仮設を立て無為な時間を過ごしました（ちなみに oTree はシングルスレッドで動いているため，処理に割り込むということはありません）．三上さんにこの点を指摘していただいたおかげで自分の過ちに気づくことができました．まずはじっくりコードを読め，という教訓を得ました．




<script async src="https://cpwebassets.codepen.io/assets/embed/ei.js"></script>
{% endraw %}
