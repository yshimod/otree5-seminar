---
showToc: false
---


# oTree 5 勉強会

2022年5月12日から7月14日までの全10回にわたって，行動実験フレームワーク oTree [(Chen et al., 2015)](https://www.sciencedirect.com/science/article/pii/S2214635016000101) （バージョン5以降）の勉強会を，阪大社研の実験関係者を中心に実施しました．
この Web ページは，勉強会の内容を記録するものです．
勉強会は終了しましたが，内容を随時更新していくかもしれません．
更新履歴は[リポジトリ](https://github.com/yshimod/otree5-seminar)のコミット履歴を参照してください．
（下平勇太）


## 質問・コメント

質問・コメントがある場合は，（直接，下平に連絡していただいても構いませんが，） [https://github.com/yshimod/otree5-seminar/issues](https://github.com/yshimod/otree5-seminar/issues) でスレッドを立ててください．


## 目次

- [Ubuntuで oTree 本番環境を構築する](day1.md)
    - Python の導入に関しては[こちら](server_setup/README.md#python-を-pyenv--virtualenv-で導入)
    - oTree のインストールに関しては[こちら](server_setup/README.md#pipで-otree-のインストール)
    - PostgreSQL のインストールに関しては[こちら](server_setup/README.md#データベースサーバーソフトウェアの-postgresql-を導入)
    - 環境変数の設定に関しては[こちら](server_setup/README.md#環境変数の設定)


- [GitとGitHubの使い方 ・ Herokuの使い方](day2.md)
    - [GitとGitHubの使い方](day2.md#2-gitとgithubの使い方)
    - Herokuの使い方に関しては[こちら](heroku/README.md)


- [VS Code の導入 ・ oTree プログラミングの概要](day3.md)
    - [VS Code の導入](day3.md#vs-code-の導入)
    - [シェルで使用する oTree サブコマンド](day3.md#シェルで使用する-otree-サブコマンド)
    - [oTree のデータモデル](day3.md#otree-のデータモデル)
    - [管理者画面](day3.md#管理者画面)


- [oTree プログラミングの進め方](day4.md) （どこからコードを書いていけばよいか分からない場合はこちら）
    - [入力フォームの検証](day4.md#a1-入力フォームの検証)
    - [自作の関数の引数は何？](day4.md#a2-自作の関数の引数は何？)
    - [oTree の通貨型](day4.md#a3-otree-の通貨型)


- [テンプレートにおけるプログラミング](day5.md)
    - 機能の一覧は[こちら](otree_ref/templatefile.md)


- [oTree の組み込み関数の使い方](day6.md)
    - 機能の一覧は[こちら](otree_ref/init.md)
    - [意思決定データを Group に記録する](day6.md#意思決定データを-group-に記録する)
    - [ページ表示をスキップする](day6.md#ページ表示をスキップする)
    - [`フィールド名_max()` で入力フォーム検証の条件を動的に設定する](day6.md#フィールド名max-で入力フォーム検証の条件を動的に設定する)
    - [`creating_session()` を使ったプレイヤーのシャッフル](day6.md#creatingsession-を使ったプレイヤーのシャッフル)
    - [関数内でインポートしてよいか？](day6.md#【tips】-関数内でインポートしてよいか？)
    - [意思決定画面で時間制限を設定してみる](day6.md#意思決定画面で時間制限を設定してみる)
    - [「 `@staticmethod` 」は何か？](day6.md#【tips】-「-staticmethod-」は何か？)
    - [確率的に繰り返しを終了するとき `NUM_ROUNDS` は最大数を設定する](day6.md#確率的に繰り返しを終了するとき-numrounds-は最大数を設定する)


- [（Qualtricsのような）質問紙調査を作る方法](day7.md)
    - [データモデルでたくさんのフィールドを定義する](day7.md#2-データモデルでたくさんのフィールドを定義する)
    - [入力フォームの順番をランダム化する](day7.md#3-入力フォームの順番をランダム化する)
    - [ページの順番をランダム化する](day7.md#4-ページの順番をランダム化する)
    - [CSS](day7.md#css)
    - [Bootstrap](day7.md#bootstrap)


- [ExtraModel ・ JavaScript ・ Live page](day8.md)
    - [ExtraModel](day8.md#extramodel)
    - [JavaScript](day8.md#javascript)
    - [Live page](day8.md#live-page)
    - [三上さん特別レクチャー（ダブルオークションの実装）](day8.md#三上さん特別レクチャー（ダブルオークションの実装）)


- [演習 （少し特殊な最後通牒ゲームを作る）](exercise.md)


## 資料

- 【開発編】
    - [シェルで使用するoTreeサブコマンド](otree_ref/cmd.md)
    - [settings.py の書き方](otree_ref/settings.md)
    - [\_\_init\_\_.py の書き方](otree_ref/init.md)
    - [テンプレートファイルの書き方](otree_ref/templatefile.md)


- 【デプロイ編】
    - [Ubuntuで oTree を動かす](server_setup/README.md)
    - [Herokuで oTree を動かす](heroku/README.md)


- 【Ubuntuの仮想環境の構築】
    - [WSL2](ubuntu/wsl2.md)
    - [VirtualBox + Vagrant](ubuntu/vagrant.md)
