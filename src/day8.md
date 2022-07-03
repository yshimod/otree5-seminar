{% raw %}
【第8回】 2022年7月14日


- 今後の予定
    - 第8回: 連続時間ダブルオークション（ライブページとExtraModel）
    - 第回: 補遺






# ExtraModel ・ JavaScript ・ ライブページ


## ExtraModel

## JavaScript

- 画面の表示内容を動的に書き換えたり，入力フォームの値の検証を自前で実装したりする場合には JavaScript を使う．


- JavaScript を使うには，以下の3通り．

    - テンプレートファイルの HTML タグにたとえば `onclick` 属性を加える．

    ```html
    <input type="button" onclick="alert('ボタンが押されました！');">
    ```

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

    - player モデルで `switching_point` なる変数を ` models.IntegerField()` で定義しておく．MPLの1行ずつデータを取りたい場合は各行に相当する変数を定義する．

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



## ライブページ

- [https://otree.readthedocs.io/en/latest/live.html](https://otree.readthedocs.io/en/latest/live.html)
- Udemyの講座 [https://www.udemy.com/course/learn-otree/](https://www.udemy.com/course/learn-otree/) のライブページに関するレクチャー（「Advanced oTree Features」の「Real Time Interaction between Participants - oTree LivePages」）は無料で見れる．




## 三上さん特別レクチャー



<script async src="https://cpwebassets.codepen.io/assets/embed/ei.js"></script>
{% endraw %}
