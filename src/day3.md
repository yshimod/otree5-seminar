【第3回】 2022年5月26日

<h1>oTreeプログラミングの概要</h1>

- [始める前にやっておいたほうが良いこと](#始める前にやっておいたほうが良いこと)
- [1. 開発環境の準備](#1-開発環境の準備)
- [2. シェルで使用するoTreeサブコマンド](#2-シェルで使用するotreeサブコマンド)
- [3. oTreeプロジェクトファイルの構成と `settings.py`](#3-otreeプロジェクトファイルの構成と-settingspy)
- [4. 管理者画面](#4-管理者画面)
- [5. `__init__.py` の構成](#5-__init__py-の構成)
- [6. oTreeで実験プログラムを開発する前に考えておくこと](#6-otreeで実験プログラムを開発する前に考えておくこと)


## 始める前にやっておいたほうが良いこと

- VS Code（Visual Studio Code）のインストール
    - [https://code.visualstudio.com/](https://code.visualstudio.com/) からインストーラーをダウンロードしてインストール．
    - WSL2やVagrantで入れたUbuntuを使っている場合でも，**Windowsに** VS Codeを入れる．


## 1. 開発環境の準備
- VS Codeの導入
    - （VS CodeのWSL用アドオン [https://docs.microsoft.com/ja-jp/windows/wsl/tutorials/wsl-vscode](https://docs.microsoft.com/ja-jp/windows/wsl/tutorials/wsl-vscode)）
    - VS CodeのPython用アドオン
    - VS Codeでターミナルは使えるか？使いたいシェルに接続できているか？
    - Gitは使えるか？
- Chromeデベロッパーツール（検証モード）


## 2. シェルで使用するoTreeサブコマンド

## 3. oTreeプロジェクトファイルの構成と `settings.py`
- ファイル構成
    - `settings.py`
    - `requirements.txt`
    - `Procfile`
    - `_static` ディレクトリ
    - `_templates` ディレクトリ
    - 各アプリのディレクトリ
        - `__init__.py`
        - `*.html`
- 重要なoTree用語 [https://otree.readthedocs.io/ja/latest/conceptual_overview.html](https://otree.readthedocs.io/ja/latest/conceptual_overview.html)
- `settings.py` の書き方

## 4. 管理者画面
- Demo
- Sessions
- Rooms
- Data
- Server Check
- 一つの実験セッションの中で
    - Links
    - Monitor
    - Data
    - Payments
    - Description
    - Edit

## 5. `__init__.py` の構成
- チュートリアル（公共財ゲーム）を参照しながら  
    [https://otree.readthedocs.io/ja/latest/tutorial/part2.html](https://otree.readthedocs.io/ja/latest/tutorial/part2.html)



## 6. oTreeで実験プログラムを開発する前に考えておくこと
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
