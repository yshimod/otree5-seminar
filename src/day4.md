{% raw %}
【第4回】 2022年6月2日 （2022年6月9日に補足）

# oTree プログラミングの進め方


## 1. oTree で実験プログラムを開発する前に考えておくこと

- 実験セッションの流れは？
    - たとえば...
        1. インストラクション・確認クイズ
        2. 意思決定課題
        3. アンケート
        4. 報酬額フィードバック
    - 実験課題ごとにアプリを作るとして，途中でアプリを分割・結合するのは面倒くさい．
    - 課題の順番をランダム化するには，アプリを分割しないほうが良い．

- 表示するページの構成は？
    - 絵コンテを作ってみる．
    - はじめにページクラスの定義（と空テンプレートファイルの作成）だけやっておく．

- 意思決定のインターフェイスは？
    - テキストフォーム，ドロップダウンリスト，ラジオボタン，......

- トリートメントの条件分岐をどう実装する？
    - 条件別にアプリを作る？ SESSION_CONFIGSで条件分岐？ セッション内で乱数を引いて割り付ける？
    - → 途中で実装を変えるのは面倒くさい．

- どんなデータを取る？
    - 得られる意思決定データの形式は？（インターフェイスのデザインとも関係．）
    - 意思決定データ以外に記録しておくデータは？

- ラボでの集団実験か？Zoomでの集団実験か？アンケートなど個人の意思決定課題か？
    - 参加者の端末は何？（ラボのPCを想定してハードコーディングする？ 各参加者の私物PCやスマホに対応する？）
    - インストラクションは紙で配布する？画面を読ませる？
    - 確認クイズはどのような形式？（どうしても確認クイズに正答できない場合は？）
    - 実験中の参加者の行動を監視できるか？
    - 途中で脱落する参加者への対応は？
    - 報酬の支払い方法は？


<p class="ytubevideo"><iframe width="560" height="315" src="https://www.youtube.com/embed/tlWHJFvWkv4?rel=0&enablejsapi=1&origin=https://yshimod.github.io/" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe></p>


- プログラム開発を開始する前に細かく計画しておいた方が良い．
    - 一度作り始めてから仕様を変更するとき，場合によってはゼロスタートと変わらない工数を要することも．

- 勉強会では様々な事態に対応できるように機能を網羅しようとしますが，取捨選択してください．



## 2. 計画

<p class="ytubevideo"><iframe width="560" height="315" src="https://www.youtube.com/embed/XBL7DyYMtIA?rel=0&enablejsapi=1&origin=https://yshimod.github.io/" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe></p>


- チュートリアル（公共財ゲーム）を参照しながら  
    [https://otree.readthedocs.io/ja/latest/tutorial/part2.html](https://otree.readthedocs.io/ja/latest/tutorial/part2.html)
    - 3人ゲーム
    - 同時手番
    - 利得: 初期保有 - 貢献額 + （グループ内合計貢献額 * 倍率 / 人数）
    - 定数
        - `PLAYERS_PER_GROUP`（1グループあたりの人数） `= 3`
        - `NUM_ROUNDS`（ラウンド数） `= 1`
        - `ENDOWMENT`（初期保有） `= 1000`
        - `MULTIPLIER`（倍率） `= 2`
    - 記録するデータ
        - `player.contribution`（各プレイヤーの貢献額）
        - `group.total_contribution`（各グループの合計貢献額）
        - `group.individual_share`（各グループにおける各プレイヤーへの配分額）
    - 3ページ構成:
        1. "Contribute"（意思決定ページ）
        2. "ResultsWaitPage"（待機ページ）
        3. "Results"（結果表示ページ）



## 3. プロジェクトの空ファイルを作成する

- 勉強のためにサンプルゲームを追加せず，ゼロの状態から始めています．
- 実際プログラムを始めるときには，サンプルゲームや，自分（あるいは友達）が以前作ったプロジェクトを書き換えていくのが多いでしょう．


#### 作業1

<p class="ytubevideo"><iframe width="560" height="315" src="https://www.youtube.com/embed/XWCOhnx6Tvw?rel=0&enablejsapi=1&origin=https://yshimod.github.io/" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe></p>


1. `otree startproject pgg` で `pgg` なるプロジェクトを作成．サンプルゲームは追加しない．

2. `cd pgg` で作成したプロジェクトディレクトリに入った後，`otree startapp publicgoodsgame` で `publicgoodsgame` なるアプリを作成．

3. Gitでプロジェクトを管理する場合，この段階で `git init` しておく．
    - GitHubも使う場合はブラウザでGitHubを開き，リポジトリを作成しておく．このとき，リポジトリ名はプロジェクト名 `publicgoodsgame` と一緒にしておく．



## 4. `settings.py` を書いておく

- プロジェクトのディレクトリ直下にある `settings.py` を編集して，作成したアプリ `publicgoodsgame` を起動するように設定する．

- 公式ドキュメントのチュートリアルでは，アプリの `__init__.py` やテンプレートファイルの編集を一通りしたあとに `settings.py` の編集をしているが，ここでは最初にやっておく．

- そのココロは，アプリの `__init__.py` やテンプレートファイルを編集している最中に `otree devserver` でサーバーを起動させておき，ブラウザで動作確認をするため．`settings.py` の `SESSION_CONFIG` が設定されていないとアプリを動かせない．


#### 作業2

<p class="ytubevideo"><iframe width="560" height="315" src="https://www.youtube.com/embed/AA-EYCAknsw?rel=0&enablejsapi=1&origin=https://yshimod.github.io/" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe></p>


1. `SESSION_CONFIG` を以下のように設定．
  ```python
  SESSION_CONFIG = [
      dict(
          name = "test_expt",
          app_sequence = ["publicgoodsgame"],
          num_demo_participants = 3
      )
  ]
  ```
    - `name` はセッション名．管理者画面で表示する名前として `display_name` を別に設定しても良い．
    - `app_sequence` には使うアプリの名前をリストで渡す．まだアプリは `publicgoodsgame` 一つしかないため，要素が一つだけのリスト `["publicgoodsgame"]` を設定する．
    - `num_demo_participants` にはデモページでの参加者数を設定する．今作ろうとしている公共財ゲームは3人ゲームなので3の倍数を設定する．

2. `LANGUAGE_CODE = 'ja'` として日本語を使うようにしておく．

3. `REAL_WORLD_CURRENCY_CODE = 'JPY'` として通貨単位を「円」にする．

4. その他， `ADMIN_*` の設定などはとりあえずいじらないでおいておく．

5. 編集が終わったら， `otree devserver` でサーバーを起動させてみる．
    - サーバーを起動させる前に，プロジェクトのディレクトリに `db.sqlite3` があれば消しておく．



## 5. テンプレートファイルを作成して `__init__.py` の最低限の設定をする

- 参加者に呈示されるページの内容は，テンプレートファイル（アプリのディレクトリにあるHTMLファイル）に記述する．
    - テンプレートファイルを編集するときには，サーバーを起動させ（ `otree devserver` ）ブラウザで画面がどのように表示されるのかを確認しながら作業した方が良い．
    - サーバーを起動させて，テンプレートファイルをページとして正しい順番で表示させるためには `__init__.py` に設定しなければならない．


1. `otree startapp` でアプリを追加した段階では `MyPage.html` と `Results.html` の2つのテンプレートファイルが生成されている．

1. テンプレートファイルを必要なだけコピーする（不要な分は削除する）．

1. `__init__.py` で，表示するページごとに `Page` クラスを継承するクラスを定義する．全プレイヤーの回答を待機するページが必要な場合は `WaitPage` クラスを継承するクラスを設定する．
    - クラスの名前はテンプレートファイルのファイル名と同じにする．
    - 待機ページはテンプレートファイルを用意しなくて良い．
    - クラス名は（Pythonの慣習として）頭文字を大文字にすると良い．
    - 最低限クラスの存在を定義するだけであれば，中身は `pass` と書いておけば良い．何も書かないとエラーになる．
    -  `otree startapp` でアプリを追加した段階では以下のように定義されている．これを適宜書き換えれば良い．
      ```python
      class MyPage(Page):
          pass

      class ResultsWaitPage(WaitPage):
          pass

      class Results(Page):
          pass
      ```

1. 表示するページの順番を `page_sequence` で設定する．クラス名（クラスオブジェクト）をリストで渡す．

1. 定数クラス `C` に必要最低限の設定をする．
    - デフォルトで `NAME_IN_URL`，`PLAYERS_PER_GROUP`，`NUM_ROUNDS` が設定されている．単にテンプレートファイルがブラウザでどう見えるかを確認するために oTree サーバーを起動することが目的であれば，とりあえずデフォルトのままでも良い．


#### 作業3

<p class="ytubevideo"><iframe width="560" height="315" src="https://www.youtube.com/embed/4h6R9dFzXKk?rel=0&enablejsapi=1&origin=https://yshimod.github.io/" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe></p>


- 必要なテンプレートファイルは `Contribute.html`（意思決定ページ）と `Results.html`（結果表示ページ）．`otree startapp` で生成された `MyPage.html` のファイル名を `Contribute.html` に変更する．`Results.html` はそのまま．

- ↑ のファイル名変更に対応するために，`__init__.py` で `MyPage` なるクラスの名前を `Contribute` に変更する．

- ↑ のクラス名変更に対応し，`page_sequence = [Contribute, ResultsWaitPage, Results]` に直す．

- この段階でアプリのディレクトリの中身は以下:
  ```
  ./publicgoodsgame/
  ├── Contribute.html
  ├── Results.html
  ├── __init__.py
  └── __pycache__
  ```

- この段階で `__init__.py` の内容は以下:
  ```python
  from otree.api import *

  doc = """
  Your app description
  """

  class C(BaseConstants):
      NAME_IN_URL = 'publicgoodsgame'
      PLAYERS_PER_GROUP = None
      NUM_ROUNDS = 1

  class Subsession(BaseSubsession):
      pass

  class Group(BaseGroup):
      pass

  class Player(BasePlayer):
      pass

  # PAGES
  class Contribute(Page):
      pass

  class ResultsWaitPage(WaitPage):
      pass

  class Results(Page):
      pass

  page_sequence = [Contribute, ResultsWaitPage, Results]
  ```



## 6. テンプレートファイルを編集する

1. タイトルブロックを編集する．
    - `{{ block title }}` と `{{ endblock }}` に挟まれた部分（デフォルトでは `Page title` と入っている部分）を「タイトルブロック」と呼ぶ．
    - タイトルブロック内に，ページの冒頭で表示するタイトルを記述する．

2. コンテンツブロックを編集する．
    - `{{ block content }}` と `{{ endblock }}` に挟まれた部分を「コンテンツブロック」と呼ぶ．
    - コンテンツブロック内に，ページの本文を，HTMLタグも適宜使って記述する．
    - 説明文などの文章は `<p>` タグや `<div>` タグで記述する．
    - 一般論として，`<br>` タグによる改行を多用するのではなく，段落ごと `<p>` タグを使うのが良い．

3. コンテンツブロック内に入力フォームを挿入する
    - たとえばフィールド名を `contribution` にするとき，以下をコンテンツブロック内に記述すれば，とりあえず入力フォームができる．
      ```html
      <input name="contribution">
      ```
    - クライアント側で行う検証も設定する．
        - 必須回答にする場合には `required` 属性を追加する．
        - 文字数の長さを `maxlength`，`minlength` 属性で指定したり，数値の範囲を `max`，`min` 属性で指定したりすることができる．
        - たとえば，必須回答で数値を入力させ，かつ，最小値を0，最大値を100にするときは以下．
          ```html
          <input type="number" name="contribution" required min="0" max="100">
          ```
    - テンプレートタグで記述すれば，↑ 以上のようにしてタグを自分で記述して入力フォームを作る作業を， oTree サーバーがやってくれる．
        - コンテンツブロック内に `{{ formfields }}` と記述すれば，`<input>` タグと，その入力フォームに対応する `<label>` タグを生成してくれる．
        - [https://otree.readthedocs.io/en/latest/forms.html](https://otree.readthedocs.io/en/latest/forms.html)

4. コンテンツブロック内に「次へ」ボタンを挿入する
    - コンテンツブロック内でフォームを送信させるボタンを作れば，それが次のページへ進ませるボタンとなる．
        - たとえば以下をコンテンツブロック内に記述すれば良い．
          ```html
          <button type="submit">次へ</button>
          ```
        - `type` 属性のデフォルトが `submit` であるため， `<button>` タグで `type="submit"` と陽に指定しなくても良い．
    - コンテンツブロック内に `{{ next_button }}` と記述すれば， oTree サーバーが以下の `<button>` タグを生成してくれる．
      ```html
      <button class="otree-btn-next btn btn-primary">次へ</button>
      ```


#### 作業4

<p class="ytubevideo"><iframe width="560" height="315" src="https://www.youtube.com/embed/udHF5mDzv94?rel=0&enablejsapi=1&origin=https://yshimod.github.io/" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe></p>


- とりあえず `Contribute.html` に説明文と入力フォームを作る．

- 見てくれはさておき，とにかく最低限のことだけを記述する．

- コンテンツブロック内でテンプレートタグを使わない場合:
  ```html
  {{ block title }}
      意思決定
  {{ endblock }}
  {{ block content }}
      <p>以下の入力欄に貢献額を入力してください．</p>
      <input name="contribution">
      <button>次へ</button>
  {{ endblock }}
  ```

- コンテンツブロック内でテンプレートタグを使う場合:
  ```html
  {{ block title }}
      意思決定
  {{ endblock }}
  {{ block content }}
      <p>以下の入力欄に貢献額を入力してください．</p>
      {{ formfields }}
      {{ next_button }}
  {{ endblock }}
  ```

- ページ内で変数を展開する（たとえば，計算した報酬額などを `Results.html` で表示する）方法は次回に後回し．



## 7. `__init__.py` を編集する

1. まずテンプレートファイルの存在を oTree に知らせる．
    - oTree 3 では，主に `pages.py` に記述していた内容．
    - 表示するページごとに `Page` クラスを継承するページクラスを設定する．
    - ページクラスの名前はテンプレートファイルのファイル名と同じにする．
    - ページクラスの名前がURLに表示される．
        - ページクラスの名前を参加者に見せたくない場合，クラス名は無味乾燥なものにしておく．
        - クラス変数 `template_name` にテンプレートファイルのパスを代入することによって，ページクラスとテンプレートファイルの関係を陽に記述すれば，クラス名とテンプレートファイルのファイル名を一致させる必要はない．
    - 待機ページのために `WaitPage` クラスを継承するクラスを設定する．
        - 同時手番ゲームで利得を計算するためには，グループ内の全メンバーの意思決定が完了したタイミングで利得を計算する関数を動かさなければならない．タイミングを合わせるために，先に意思決定を済ませた参加者に表示するための待機ページを用意すればよい．
        - 待機ページのためにテンプレートファイルを作成して通常のページとして実装することも可能ではある．しかし， oTree の機能（`WaitPage` クラス）を使うのが楽．
    - 各ページのクラスを設定した後，表示するページの順番を `page_sequence` で設定する．クラス名（クラスオブジェクト）をリストで渡す．

2. 入力フォームの変数をデータベースのどこに保存するかを設定する．
    - oTree 3 では，主に `models.py` に記述していた内容．
    - player， group， subsession の各階層で保存するべきデータのフィールド名を `Player` クラス，`Group` クラス，`Subsession` クラスのそれぞれで定義する．
        - 公共財ゲームなど，プレイヤーに役割の区別がなく対称的な場合，意思決定データは一つのフィールドで player の階層に保存しておけば良い．
        - 信頼ゲームにおける提案者・応答者など，プレイヤーに役割の区別があって，かつ，グループ内で一つの役割に複数のプレイヤーが縮退しない場合，各プレイヤーの意思決定データはグループにおいてユニークなので， player の階層に保存するよりも group の階層に保存する方が良い．
        - セッションにおいて，（グループをまたいで）ある特定の一人だけが意思決定する場合は， subsession の階層に保存する方が良い．
        - participant と session の階層へは入力フォームから直接データを保存できないため，工夫を要する．
    - デフォルトで `Player` クラス，`Group` クラス，`Subsession` クラスが `__init__.py` に記述されているので，フィールドを設定する場合には `pass` を削除して書き込めば良い．
    - `フィールド名 = models.*Field()` と記述して設定する．
        - `models.*Field()` の詳細は [こちら](otree_ref/init.md#フィールドの型) ．
        - `models.*Field()` の引数で，最大値と最小値などの検証の設定や，初期値や選択肢の設定ができる．詳細は [こちら](otree_ref/init.md#フィールドの型) ．
            - `label`: 入力フォームのラベル（デフォルトはフィールド名）
            - `min`: 最小値
            - `max`: 最大値
        - （テンプレートタグではなく）タグの直打ちで入力フォームを作りながら，`models.*Field()` の引数で `choices`，`min`，`max` を設定している場合，タグの直打ちでの実装との整合性に気をつける．
            - タグの属性で`min`や`max`などの制約を設定しない場合（クライアントでの検証をしない場合）でも， oTree サーバー側で検証は行われる．
            - たとえば `models.IntegerField()` の引数で `choices = [0, 100]` としておきながら，タグ直打ちで `<input type="number" name="contribution" required min="0" max="100">` と入力フォームを作り，参加者が「10」と回答した場合，クライアントの検証は通過するが，10が `[0, 100]` に含まれないため， oTree の検証は通過せず，エラーが出る．
    - データモデルを設定した後，どのページで入力フォームを使うのかを設定する．
        - 入力フォームを使うページのクラスの中で以下の2つを設定:
            - `form_model`: 保存したいデータのモデル（ player， group， subsession のいずれか）から一つを選んで文字列で指定する．
                - `Player` クラスで定義した `contribution` なるフィールドを使う場合は `form_model = "player"` とする．
            - `form_fields`: 保存したいデータのフィールド名をリストで指定する．
                - `Player` クラスで定義した `contribution` なるフィールドを使う場合は `form_fields = ["contribution"]` とする．
    - ページクラスの設定とデータモデルの設定が終われば，とりあえず意思決定データを収集することはできる．質問紙調査であれば，ここまでの作業で完成．

3. 定数や関数の設定をしてゲームとして成立させる．
    - oTree で収集した意思決定データからゲームの利得（報酬額）を計算することができる．
        - （その場で）フィードバックする必要が無ければ，必ずしも oTree で計算しなければならないわけではない．
        - たとえばラボでくじを引いて，その結果を使って報酬額を計算する，ということもできる．  
        [https://otree.readthedocs.io/en/latest/misc/rest_api.html#session-vars-endpoint](https://otree.readthedocs.io/en/latest/misc/rest_api.html#session-vars-endpoint)
    - `C` クラス （ oTree 3 では， `Constants` ）で定数を定義する．
        - 必ず定義しなければならないもの:
            - `NAME_IN_URL`: デフォルトではアプリ名の文字列が設定されている．任意に変えても良い．
            - `PLAYERS_PER_GROUP`: ゲーム実験の場合，各グループの人数（2以上）を設定する．グループを設定しない場合は `None` とする．
            - `NUM_ROUNDS`: アプリを繰り返す場合，繰り返す（最大）回数を設定する．繰り返さない場合は `1` とする．
        - ↑ 以上の3つの変数名については小文字（ `name_in_url`， `players_per_group`， `num_rounds` ）でも良い．バージョン互換性対応がされているため．
        - `C` クラス で定義する変数名は大文字にすると良い（？）． 自分で定義するものについては厳密に大文字小文字を区別することに注意．
        - ゲームのパラメータも `C` クラス で設定すると良い．
            - しかし，トリートメントを `C` クラス で実装するべきではない．たとえば公共財ゲームの限界収益率を変えて実験を行う場合，アプリごと複製して，複製した各アプリの `C` クラス で限界収益率の値を変える，という方法でも実装は可能である．しかし，アプリごと複製する，というのは筋が悪い（DRY原則に反する）．それよりも `settings.py` の `SESSION_CONFIGS` で定数を定義し，セッションを作成する際に値を具体的に設定する方が良い．
            - パラメータを（たとえば報酬額を計算する関数で）ハードコーディングしてしまうのも良くない．当座の実験計画では変更する予定のないパラメータであっても，将来的には値を変えて実験を実施する場面が訪れるかもしれない．ハードコーディングしてしまうと，パラメータの変更に漏れが生じるかもしれない．
    - 報酬を計算する関数などは，`__init__.py` のどこかに記述すれば良い．
        - oTree 3 では `Player` クラスなどの中でインスタンスメソッドとして定義していた．
        - 関数の引数に注意．
            - 待機ページで `after_all_players_arrive` として呼び出したい場合，引数は `group` （`Group` クラスのインスタンスオブジェクト）とする．このとき，各プレイヤーに個別の処理をするときにはforループを使う．
            - `before_next_page` や `vars_for_template` の中で呼び出したい場合，引数は `player` （`Player` クラスのインスタンスオブジェクト）とする．呼び出されるタイミングはプレイヤーごと異なるため，グループでユニークな処理をしたい場合は注意．
        - 自前で作った関数は，陽に呼び出さないと動かない．忘れずに設定する．


#### 作業5

<p class="ytubevideo"><iframe width="560" height="315" src="https://www.youtube.com/embed/eUxjPHHrrTY?rel=0&enablejsapi=1&origin=https://yshimod.github.io/" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe></p>


- まずはテンプレートに記述した入力フォームが機能するように（フォームを送信したときに oTree サーバーで認識できるように）設定する．
    - `Player` クラスに `contribution = models.FloatField()` を記述する．
        - 公式ドキュメントのチュートリアルでは `models.CurrencyField()` を使用しているが， oTree の通貨型を理解するのが面倒なので，単純な実数型を使う．
    - `Contribute` クラスに `form_model = "player"` と `form_fields = ["contribution"]` を記述する．


<p class="ytubevideo"><iframe width="560" height="315" src="https://www.youtube.com/embed/qsyWxGr8U-M?rel=0&enablejsapi=1&origin=https://yshimod.github.io/" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe></p>


- 利得を計算する際の途中の変数（グループでの貢献額の合計値と分配額）をグループモデルのフィールドに保存するために定義する．
    - `Group` クラスに `total_contribution = models.FloatField()` と `individual_share = models.FloatField()`
        - 公式ドキュメントのチュートリアルでは `models.CurrencyField()` を使用しているが， oTree の通貨型を理解するのが面倒なので，単純な実数型を使う．

- 定数を定義する．
    - 3人での公共財ゲームを作るため，`PLAYERS_PER_GROUP = 3` とする．
    - 利得を計算する際のパラメータとして， `ENDOWMENT = 1000` と `MULTIPLIER = 2` を定義する．

- 利得を計算する関数 `set_payoffs` を定義する．
    - `ResultsWaitPage` クラスにおいて `after_all_players_arrive` として呼び出すため，`ResultsWaitPage` クラスよりも上に記述する．
    - `ResultsWaitPage` クラスにおいて `after_all_players_arrive` として呼び出すため，引数は `group` とする．
        - 型アノテーションをつけて `group: Group` と記述しておくと良い．
    - 関数の中身は公式ドキュメントのチュートリアルからコピペして以下の通り:
      ```python
      def set_payoffs(group: Group):
          ## 全プレイヤーのインスタンスオブジェクトが入ったリスト．
          players = group.get_players()

          ## players から一人ずつプレイヤーオブジェクト p を取り出し，contribution の値を並べたリストを内包記法で生成．
          contributions = [p.contribution for p in players]

          ## 各プレイヤーの contribution の合計を，リスト contributions の sum として計算し，
          ## Group の total_contribution に代入（代入することによってデータベースにも書き込まれる）．
          group.total_contribution = sum(contributions)

          ## 各プレイヤーの配分額を計算し，Group の individual_share に代入．
          group.individual_share = group.total_contribution * C.MULTIPLIER / C.PLAYERS_PER_GROUP

          ## 一人ずつプレイヤーの報酬額を計算し，Player の 予め用意されている payoff フィールドに代入．
          for player in players:
              player.payoff = C.ENDOWMENT - player.contribution + group.individual_share
              ## ↑ player.payoff に値を代入すると，勝手に oTree 組み込みの通貨型に変換される．値は丸められる．
      ```
    - `ResultsWaitPage` クラスにおいて `after_all_players_arrive = "set_payoffs"` とすると，グループの全プレイヤーの意思決定が送信されたタイミングで関数 `set_payoffs` が呼び出される．
        - `ResultsWaitPage` クラスに以下のように記述しても良い．
          ```python
          @staticmethod
          def after_all_players_arrive(group: Group):
              set_payoffs(group)
          ```

- この段階で `__init__.py` の内容は以下:
  ```python
  from otree.api import *

  doc = """
  Your app description
  """

  class C(BaseConstants):
      NAME_IN_URL = 'publicgoodsgame'
      PLAYERS_PER_GROUP = 3
      NUM_ROUNDS = 1
      ENDOWMENT = 1000
      MULTIPLIER = 2

  class Subsession(BaseSubsession):
      pass

  class Group(BaseGroup):
      total_contribution = models.FloatField()
      individual_share = models.FloatField()

  class Player(BasePlayer):
      contribution = models.IntegerField()

  def set_payoffs(group: Group):
      players = group.get_players()
      contributions = [p.contribution for p in players]
      group.total_contribution = sum(contributions)
      group.individual_share = group.total_contribution * C.MULTIPLIER / C.PLAYERS_PER_GROUP
      for player in players:
          player.payoff = C.ENDOWMENT - player.contribution + group.individual_share

  # PAGES
  class Contribute(Page):
      form_model = 'player'
      form_fields = ['contribution']

  class ResultsWaitPage(WaitPage):
      after_all_players_arrive = "set_payoffs"

  class Results(Page):
      pass

  page_sequence = [Contribute, ResultsWaitPage, Results]
  ```



---

【補足】 <a id="day4suppl"></a>


## A1. 入力フォームの検証

<p class="ytubevideo"><iframe width="560" height="315" src="https://www.youtube.com/embed/mwSDVeIXyQs?rel=0&enablejsapi=1&origin=https://yshimod.github.io/" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe></p>


- [https://otree.readthedocs.io/en/latest/forms.html#simple-form-field-validation](https://otree.readthedocs.io/en/latest/forms.html#simple-form-field-validation)

- ページで入力フォームが送信されるとき（「次へ」ボタンが押されるとき）， oTree は入力された値を検証する．

- HTMLタグで入力フォームの検証を実装していれば，ページで入力フォームを送信する前にブラウザでも入力フォームに入力された値を検証する．  
[https://developer.mozilla.org/ja/docs/Learn/Forms/Form_validation](https://developer.mozilla.org/ja/docs/Learn/Forms/Form_validation)

- 任意回答であることを陽に設定しない場合，少なくとも値が入力されているかの検証が行われる．
    - 任意回答にする（空欄を許す）場合には `<input>` 要素に `required` 属性を追加しないことに加え， `__init__.py` でフィールドを定義するときに `blank=True` とする．

- （例1） データモデルのクラスで `input1 = models.IntegerField()` として，テンプレートでは `<input name="input1">` とHTMLタグを直書きしている場合...
    - 入力フォームで「a」と入力すると，ブラウザでの検証は行われず，そのままフォームの送信は行われる．
    - しかし， oTree サーバー側での検証において，整数型のフィールドに文字列を入れようとしていることを検出して，次のページへ遷移させず画面にエラーを表示する．

- （例2） データモデルのクラスで `input1 = models.IntegerField(max=100)` として，テンプレートでは `<input name="input1" type="number" max="1000">` とHTMLタグを直書きしている場合...
    - HTMLタグでは， `type="number"` で値を整数に限定し， `max="1000"` でその最大値を1000としている．
    - HTMLタグでは， `required` 属性をつけていないため，空欄であってもフォームの送信は行われる．しかし， oTree サーバー側の検証には引っかかり，次のページへ遷移させず画面にエラーを表示する．
    - 入力フォームで「1001」と入力すると，ブラウザでの検証にひっかかり，フォームの送信が行われない．
    - 入力フォームで「999」と入力すると，ブラウザでの検証は通過し，フォームの送信は行われる．しかし， oTree サーバー側の検証には引っかかり，次のページへ遷移させず画面にエラーを表示する．

- （例3） データモデルのクラスで `input1 = models.IntegerField(max=100)` として，テンプレートでは `{{ formfields }}` とテンプレートタグ使っている場合...
    - 入力フォームの部分のタグは以下のように生成されている:
      ```html
      <label class="col-form-label" for="id_input1">Input1</label>
      <div class="controls">
          <input type="number" class="form-control" id="id_input1" max="100" name="input1" required value="">
      </div>
      ```
    - HTMLタグで実装されている要件と oTree サーバーでの要件が一致しているため，ブラウザにおける検証に通過すれば， oTree サーバーでの検証も通過する．

- （例4） データモデルのクラスで `input1 = models.IntegerField(max=100, blank=True)` として，テンプレートでは `{{ formfields }}` とテンプレートタグ使っている場合...
    - 入力フォームの部分のタグは以下のように生成されている:
      ```html
      <label class="col-form-label" for="id_input1">Input1</label>
      <div class="controls">
          <input type="number" class="form-control" id="id_input1" max="100" name="input1" value="">
      </div>
      ```
    - HTMLタグでは， `required` 属性をつけられていないため，空欄であってもフォームの送信は行われる． oTree サーバー側でも空欄を許容して，次のページへの遷移が行われる．

- （例5） データモデルのクラスで `input1 = models.IntegerField(choices=[0, 100])` としている場合...
    - テンプレートで `{{ formfields }}` とテンプレートタグ使っている場合，入力フォームの部分のタグは以下のように生成される:
      ```html
      <label class="col-form-label" for="id_input1">Input1</label>
      <div class="controls">
          <select class="form-select" id="id_input1" name="input1" required>
              <option value="">--------</option>
              <option value="0">0</option>
              <option value="100">100</option>
          </select>
      </div>
      ```
    - もしもテンプレートで `<input name="input1" type="number" min="0" max="100">` とHTMLタグを直書きしている場合，「50」と入力したときにブラウザの検証には通過するが， oTree サーバー側の検証にはひっかかる．「0」か「100」と入力しない限り oTree サーバー側の検証にひっかかる．

- ブラウザでの検証において表示されるメッセージ（たとえば `required` があるときに Chrome で空欄のままフォームを送信しようとすると，「！ このフィールドを入力してください。」と表示される）はブラウザごと異なる．カスタマイズするには HTML， CSS， JavaScript それぞれでコードを書く必要があって面倒だが，Bootstrap を使うと便利．

- HTMLタグの直打ちで入力フォームを実装しているとき， oTree サーバー側での検証を経てエラーメッセージ画面に表示するには，テンプレートに `{{ formfield_errors 'フィールド名' }}` タグを入れる．

- oTree サーバー側検証でのエラーメッセージをカスタマイズするにはページクラスの組み込みメソッド `error_message()` や `フィールド名_error_message()` を使う．

- `error_message()` や `フィールド名_error_message()` を使って，最大値・最小値のような単純な検証だけではなく，より込み入った検証を実装することも可能． `error_message()` で記述した検証のアルゴリズムは HTML のタグには反映されないため，ブラウザでは検証されない．
    - たとえば，`input1 = models.IntegerField(min=0, max=100)` とした上で，
      ```python
      def input1_error_message(player, value):
          if value != 50:
          return '不正解です．'
      ```
    と定義した場合，「10」と入力フォームに入力したときにブラウザの検証には通過するが， oTree サーバー側の検証にはひっかかる．



## A2. 自作の関数の引数は何？


<p class="ytubevideo"><iframe width="560" height="315" src="https://www.youtube.com/embed/LXgnO3RR5OQ?rel=0&enablejsapi=1&origin=https://yshimod.github.io/" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe></p>


- [\_\_init\_\_.py の書き方](otree_ref/init.md) 参照．

- チュートリアルの公共財ゲームでは，自作の関数 `set_payoffs()` を定義していたが，その引数は `group` であった．自作の関数の引数は `group` でないとだめか？

- どこで呼び出す関数か？
    - 待機ページの `after_all_players_arrive` で呼び出すとき，引数は `group`． `wait_for_all_groups = True` と設定したときの引数は `subsession`．
    - フォームが送信された後に実行される `before_next_page()` の中で使う場合， `before_next_page()` が `player` を受け取るため，そのまま `player` を自作の関数に渡しても良いし， `player.group` を渡しても良い．



## A3. oTree の通貨型


<p class="ytubevideo"><iframe width="560" height="315" src="https://www.youtube.com/embed/TWdogNESdns?rel=0&enablejsapi=1&origin=https://yshimod.github.io/" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe></p>


- [https://otree.readthedocs.io/en/latest/currency.html](https://otree.readthedocs.io/en/latest/currency.html)

- `cu()` に数値を渡すと， oTree は数値を「通貨型」に変換する．

- `REAL_WORLD_CURRENCY_CODE = 'USD'` と設定したとき...
    - 数値は小数点以下が2桁に丸められる． `cu(2.7182)` は `2.72`．
    - テンプレートで表示すると「$2.72」．

- `REAL_WORLD_CURRENCY_CODE = 'JPY'` と設定したとき...
    - 数値は整数に丸められる． `cu(2.7182)` は `3`．
    - テンプレートで表示すると「3円」（「¥3」ではない）．

- `USE_POINTS = True` と設定したとき...
    - `REAL_WORLD_CURRENCY_CODE` によらず，数値は整数に丸められる．
    - （ `LANGUAGE_CODE = 'ja'` のとき）テンプレートで表示すると「3ポイント」．
    - 単位をデフォルト（「ポイント」）から（たとえば「トークン」に）変えるときは `settings.py` で `POINTS_CUSTOM_NAME = 'トークン'` とする．

- `to_real_world_currency()`: ポイントから通貨へ変換するメソッド．引数は `session`．
    - たとえば `JPY` で `USE_POINTS = True` のとき， `cu(2.7182).to_real_world_currency(player.session)` をテンプレートに渡せば「3円」と表示される．

- 組み込みのフィールド `player.payoff` は `CurrencyField` で定義されている．
    - `player.payoff` に数値を代入すると，勝手に通貨型に変換される．つまり勝手に数値が丸められる．

- 数値の丸めは decimal モジュールの [`ROUND_HALF_UP` モード](https://docs.python.org/ja/3/library/decimal.html#decimal.ROUND_HALF_UP)で実行されている．つまり，銀行家の丸めではなく「四捨五入」される．

- 日本で実験をするとき，ポイント表示であれ通貨表示であれ， oTree の通貨型を使う場合は，「数値が四捨五入で整数に丸められる」ことを意識すれば良い．
    - 端数を切り上げる場合は，自前で丸めの処理をした後，整数を `player.payoff` などに渡せば良い．

- 通貨型と，普通の整数型や浮動小数点型の数値との間で `+` を作用させると，通貨型になる．
    - Pylance などは警告を出すが，演算は可能．




{% endraw %}
