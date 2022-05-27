【第4回】 2022年6月2日

<h1>oTreeプログラミングの概要</h1>

- [1. `__init__.py` の構成](#1-__init__py-の構成)
- [2. oTreeで実験プログラムを開発する前に考えておくこと](#2-otreeで実験プログラムを開発する前に考えておくこと)


## 1. `__init__.py` の構成
- チュートリアル（公共財ゲーム）を参照しながら  
    [https://otree.readthedocs.io/ja/latest/tutorial/part2.html](https://otree.readthedocs.io/ja/latest/tutorial/part2.html)



## 2. oTreeで実験プログラムを開発する前に考えておくこと
- ラボでの集団実験か？Zoomでの集団実験か？アンケートなど個人の意思決定課題か？
    - 参加者の端末は何？（ラボのPC？各参加者の私物PC？スマホに対応する？）
    - インストラクションは紙で配布する？画面を読ませる？
    - 確認クイズはどのような形式？（どうしても確認クイズに正答できない場合は？）
    - 実験中の参加者の行動を監視できるか？
    - 途中で脱落する参加者への対応は？
    - 報酬の支払い方法は？
- 実験の流れは？画面の構成は？
    - 課題の順番？（一つ一つの課題をアプリとして実装する．）  
        たとえば...
        1. インストラクション・確認クイズ
        2. 意思決定課題
        3. アンケート
        4. 報酬額フィードバック
    - トリートメントの条件分岐をどう実装する？
    - 各課題（アプリ）で表示するページの順番？（絵コンテを作ってみる．）
    - 意思決定のインターフェイスは？
- どんなデータを取る？
    - 得られる意思決定データの形式は？（インターフェイスのデザインとも関係．）
    - 意思決定データ以外に記録しておくデータは？