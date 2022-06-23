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
        1. プレイヤーの役割と reward level をプレイヤーごとランダムに決める．この時点でグループの joint endowment を計算して記録しておく．
        1. サンプルゲーム「trust」「dictator」などを参考に，最後通牒ゲームを実装する．
        1. 結果表示画面で最後通牒ゲームにおける意思決定と利得を表示させる．
    1. No-veto-cost game (Scenario 2) も完成させる．
        - 最終的な報酬として，2つのゲーム（ Scenario ）のどちらかをランダムに選ぶ機構を実装する．
    1. `SESSION_CONFIGS` で定義する変数を使って，2つのゲームの順番を変える機構を実装する．
    1. インストラクションを実装する．
    1. 見てくれを良くする．
    1. エフォートタスクを実装する．


- 前日までにコードを提出してもらえたら，当日，Zoomで添削しながら解説します．

- 完成しなかった場合は，途中までで提出して，何が分からなかったのかをお知らせください．

- 誰も提出してくれなかった場合は，当日ゼロからコードを書いていく様子をZoomでお見せすることになってしまいます．


<p class="ytubevideo"><iframe width="560" height="315" src="https://www.youtube.com/embed/nZLnngQsWes?rel=0&enablejsapi=1&origin=https://yshimod.github.io/" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe></p>


{% endraw %}
