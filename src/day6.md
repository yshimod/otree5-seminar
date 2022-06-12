{% raw %}
【第6回】 2022年6月16日


- 今後の予定
    - 第6回（今日）: 
        - 逐次手番ゲーム（参加者ごと表示させる画面を変える）
        - 繰り返しゲーム（プレイヤーのシャッフル，タイムアウト）
    - 第回: ダブルオークション（JavaScriptとライブページ，ExtraModel）
    - 第回: 質問紙調査（画面のデザイン，CSS，Bootstrap）
    - 第回: 補遺


この勉強会を始める時点で，どのレベルの知識を知っているかという前提と，何ができるようになるかという目標を明確にしていなかったため，初心者向けを装いながら，実際のところオタク向けの内容になっています．

[第4回](day4.md)で，どのようにコードを書いていくのかの手順を説明したつもりではあるのですが，やはり初心者には難解ではないかと思われます．
文字資料だけでは何をどうしたらよいか分からないという方は動画を見て勉強するのが良いでしょう．
たとえば，[Jonas Freyさん](https://sites.google.com/site/jonasfreysite/)によるoTreeチュートリアルの動画は有用です．
- [Part 1: introduction](https://www.youtube.com/watch?v=OzkFvVhoHr0&list=PLBL9eqPcwzGPli11Yighw5LWwzIifEFd_&index=1)
- [Part 2: Install oTree with Pycharm](https://www.youtube.com/watch?v=criiiSiEtUw&list=PLBL9eqPcwzGPli11Yighw5LWwzIifEFd_&index=2)
- [Part 3: Overview over oTree](https://www.youtube.com/watch?v=7f5HgW4vgvU&list=PLBL9eqPcwzGPli11Yighw5LWwzIifEFd_&index=3)
- [Part 4: A first Simple Game](https://www.youtube.com/watch?v=eozwgQ21Kg0&list=PLBL9eqPcwzGPli11Yighw5LWwzIifEFd_&index=4)
- [Part 5: Real Effort Task with Multiple Rounds](https://www.youtube.com/watch?v=js2hsUwOFR0&list=PLBL9eqPcwzGPli11Yighw5LWwzIifEFd_&index=5)
- [Part 6: Multiplayer Games - Prisoners Dilemma](https://www.youtube.com/watch?v=ewCDAk4DxWo&list=PLBL9eqPcwzGPli11Yighw5LWwzIifEFd_&index=6)
- [Part 7: Deploying oTree Experiments on Heroku](https://www.youtube.com/watch?v=VrPdBEghYEM&list=PLBL9eqPcwzGPli11Yighw5LWwzIifEFd_&index=7)

第4回では，ゼロからコードを書いていく方法を解説しています．
しかし初心者には，すでに完成しているコードについて，少しいじっては動作を確認し，という試行錯誤を繰り返しながら自分が作りたいものに変えていく方法で開発しながら勉強するのが良いでしょう．

今回は完成したコードを読み下していきながら，特に oTree の組み込み関数の使い方を勉強していきます．



# oTree の組み込み関数の使い方

- oTree の組み込み関数は [ `__init__.py` の書き方](otree_ref/init.md) で網羅しているつもりです．
- 組み込み関数の使い方については，公式ドキュメントで関数名を検索してください．
    - たとえば，[`is_displayed` で検索](https://otree.readthedocs.io/en/latest/search.html?q=is_displayed)．
- [oTree Hub の Example code](https://www.otreehub.com/code/) で（ブラウザの検索機能を使って）検索して使用例を見るのも有用です．



## 逐次手番ゲーム（信頼ゲーム）

- `otree startproject` コマンド実行で手に入るサンプルゲームの「trust」アプリ．
- [公式ドキュメントのチュートリアル「信頼ゲーム」](https://otree.readthedocs.io/ja/latest/tutorial/part3.html) を参照．
- ソースコード [https://github.com/oTree-org/oTree/tree/lite/trust](https://github.com/oTree-org/oTree/tree/lite/trust)
- デモページ [https://otree-demo.herokuapp.com/demo/trust](https://otree-demo.herokuapp.com/demo/trust)


### `is_displayed()` を使ったページの制御

[https://otree.readthedocs.io/en/latest/pages.html#is-displayed](https://otree.readthedocs.io/en/latest/pages.html#is-displayed)

- ページクラスの中に `is_displayed()` を定義する．引数は `player` オブジェクト．返り値が `True` のとき，そのプレイヤーに表示される．



## 有限回繰り返しゲーム（マッチングペニー）

- `otree startproject` コマンド実行で手に入るサンプルゲームの「matching_pennies」アプリ．
- ソースコード [https://github.com/oTree-org/oTree/tree/lite/matching_pennies](https://github.com/oTree-org/oTree/tree/lite/matching_pennies)
- デモページ [https://otree-demo.herokuapp.com/demo/matching_pennies](https://otree-demo.herokuapp.com/demo/matching_pennies)


### `creating_session()` を使ったプレイヤーのシャッフル

[https://otree.readthedocs.io/en/latest/multiplayer/groups.html#group-matching](https://otree.readthedocs.io/en/latest/multiplayer/groups.html#group-matching)



## 無限回繰り返しゲーム（繰り返しPD）

- [https://github.com/snunnari/otree_repeated_prisoner](https://github.com/snunnari/otree_repeated_prisoner)
- ↑ oTree のバージョンが古い．
- 「無限回」の実装として微妙かも．ライブページを使う実装のほうが良さそう． 
    -[ https://www.otreehub.com/projects/otree-more-demos/](https://www.otreehub.com/projects/otree-more-demos/) の「supergames_indefinite」．





{% endraw %}
