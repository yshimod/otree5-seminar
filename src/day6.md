{% raw %}
【第6回】 2022年6月16日

- 今後の予定
    - 第4回: 同時手番ゲーム（oTreeプログラミングの基礎）
    - 第5回: テンプレートファイルの書き方
    - 第6回（今日）: 
        - 逐次手番ゲーム（参加者ごと表示させる画面を変える）
        - 繰り返しゲーム（プレイヤーのシャッフル，タイムアウト）
    - 第回: ダブルオークション（JavaScriptとライブページ，ExtraModel）
    - 第回: 質問紙調査（画面のデザイン，CSS，Bootstrap）
    - 第回: 補遺



## 参加者ごと表示させる画面を変える

- テンプレート内において変数を展開したりテンプレートタグを使って条件分岐させたりする．
- プレイヤーごと入力フォーム（変数名）が異なるとき，`__init__.py` のページクラスのメソッド `is_displayed()` で制御する．
- ライブページを使って動的に画面の中身を書き換える（サーバーとの通信あり）．
    - 後の回で説明します．
- JavaScriptを使って動的に画面の中身を書き換える（サーバーとの通信なし）．
    - 後の回で説明します．



## `is_displayed()` を使ったページの制御

[https://otree.readthedocs.io/en/latest/pages.html#is-displayed](https://otree.readthedocs.io/en/latest/pages.html#is-displayed)

- ページクラスの中に `is_displayed()` を定義する．引数は `player` オブジェクト．返り値が `True` のとき，そのプレイヤーに表示される．
- 公式ドキュメントのチュートリアル「信頼ゲーム」を参照．[https://otree.readthedocs.io/ja/latest/tutorial/part3.html](https://otree.readthedocs.io/ja/latest/tutorial/part3.html)



{% endraw %}
