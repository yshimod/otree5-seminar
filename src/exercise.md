{% raw %}
【演習】 2022年6月30日・7月7日

# 【演習】 少し特殊な最後通牒ゲームを作る

- 課題: 次の論文での実験を実装する  
[https://www.sciencedirect.com/science/article/pii/S0167268116301366](https://www.sciencedirect.com/science/article/pii/S0167268116301366)  
Rodriguez-Lara, I. (2016). Equity and bargaining power in ultimatum games.
*Journal of Economic Behavior & Organization*, 130, 144-165.


- [https://github.com/yshimod/RodriguezLara2016](https://github.com/yshimod/RodriguezLara2016) のリポジトリをダウンロードして（ GitHub アカウントがあればフォークしてからクローンして），それを編集していく．

- エフォートタスクはすでに1つのアプリ（ `effort` ）で実装されてい（るものとし）て，そのスコア（クイズの得点）が `participant.effort_score` に入っているので，この値を使う．

- 実装の優先順位は...
    1. Ultimatum game (Scenario 1) だけでも完成させる．
        1. 新しいアプリを追加する．
        1. プレイヤーの役割と reward level をプレイヤーごとランダムに決める．
        1. グループの joint endowment を計算して記録しておく．
        1. サンプルゲーム「trust」「dictator」などを参考に，最後通牒ゲームを実装する．
        1. 結果表示画面で最後通牒ゲームにおける意思決定と利得を表示させる．
    1. No-veto-cost game (Scenario 2) も完成させる．
        - 最終的な報酬として，2つのゲーム（ Scenario ）のどちらかをランダムに選ぶ機構を実装する．
    1. ~~`SESSION_CONFIGS` で定義する変数を使って，2つのゲームの順番を変える機構を実装する．~~ （ちょっと面倒なので後回し）
    1. ~~インストラクションを実装する．~~ （難しいことではないので後回し）
    1. 見てくれを良くする．
    1. エフォートタスクを実装する．


- 前日までにコードを提出してもらえたら，当日，Zoomで添削しながら解説します．

- 完成しなかった場合は，途中までで提出して，何が分からなかったのかをお知らせください．

- 誰も提出してくれなかった場合は，当日ゼロからコードを書いていく様子をZoomでお見せすることになってしまいます．


<p class="ytubevideo"><iframe width="560" height="315" src="https://www.youtube.com/embed/nZLnngQsWes?rel=0&enablejsapi=1&origin=https://yshimod.github.io/" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe></p>



#### 6月30日

<p class="ytubevideo"><iframe width="560" height="315" src="https://www.youtube.com/embed/ka8scVX9YzY?rel=0&enablejsapi=1&origin=https://yshimod.github.io/" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe></p>


- やったこと...
    - 実験課題用に新しいアプリを作る．
    - 必要なページのクラスをとりあえず定義し，対応するテンプレートファイルも作成する．
    - 必要な変数（フィールド）を定義する．
    - `creating_session()` で reward level をランダムに決定する．
        - まだ プレイヤーの役割はランダム化していない．
    - 最初の待機ページの `after_all_players_arrive()` で， `effort` アプリで生成した変数を受け取り， group の変数に代入する．
    - 同じく `after_all_players_arrive()` で， joint endowment も計算し group の変数に代入する．
    - 意思決定のページを該当する役割のプライヤーにのみ表示するように `is_displayed()` を定義する．
    - 入力フォームを設定する．
        - まだ，最大値・最小値の検証は実装していない．
- 本日の進捗分をコミットしておきました: [https://github.com/yshimod/RodriguezLara2016](https://github.com/yshimod/RodriguezLara2016)


- **ToDo**
    - とりあえず Scenario 1 の利得計算の実装を行う．
    - 入力フォームの最大値・最小値の検証を実装する．
    - プレイヤーの役割をランダムに決定する．
    - Scenario 2 (No-veto-cost game) 用のページを追加する．変数（フィールド）も追加する．
    - Scenario 1 と Scenario 2 のどちらを実現させるのかをランダムに決定する機構を実装する．
    - 意思決定画面の見てくれを良くする（入力フォームをスライダーにする）．



{% endraw %}
