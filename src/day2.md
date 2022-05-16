# 【第2回】 2022年5月19日

### 始める前にやっておいたほうが良いこと
- 結局どうにかして（PostgreSQLうんぬんは無視してでも）oTreeをインストールする．
    - WindowsにPythonを直接インストールして，pipでoTreeを入れる．  
        [https://otree.readthedocs.io/en/latest/install-windows.html#install-windows](https://otree.readthedocs.io/en/latest/install-windows.html#install-windows)  
        - 【フリーライド】立命館竹内研長瀬さんのチュートリアル動画が参考になります．  
            「#2 oTree Windowsでのインストール」
            （動画リンクは竹内研Slack内を検索してください．）
    - 頑張ってWSL2を使い，Ubuntu環境を用意してから...  
        1. [WSL2のインストール](ubuntu/wsl2.md)
        1. [UbuntuでoTreeを動かす](server_setup/README.md)
    - Macの場合はまずhomebrewをインストールしてから...
        - 【フリーライド】立命館竹内研長瀬さんのチュートリアル動画が参考になります．  
            「#2 oTree Macでのインストール」
            （動画リンクは竹内研Slack内を検索してください．）

- Herokuのアカウントを作成． [https://signup.heroku.com/login](https://signup.heroku.com/login)
- GitHubのアカウントを作成． [https://github.com/signup](https://github.com/signup)

- Gitのインストール
    - Windowsに直接Pythonを入れてoTreeを動かす場合，Gitのインストーラーを [https://gitforwindows.org/](https://gitforwindows.org/) からダウンロードしてインストールする．
        - 参考 [https://www.curict.com/item/60/60bfe0e.html](https://www.curict.com/item/60/60bfe0e.html)
    - Ubuntuを使う場合，以下のコマンドでインストール．
        ```bash
        sudo apt update
        sudo apt install git
        ```
    - Macを使う場合，Xcode Command Line Toolsが導入済みならばすでに `git` コマンドは使えるはず．あるいはhomebrewで入れる．

- VS Code（Visual Studio Code）のインストール
    - [https://code.visualstudio.com/](https://code.visualstudio.com/) からインストーラーをダウンロードしてインストール．
    - WSL2やVagrantで入れたUbuntuを使っている場合でも，**Windowsに** VS Codeを入れる．


### 1. GitとGitHub
- GitHubを使うメリット
- GitとGitHubの使い方


### 2. Herokuの使い方
- デプロイ
- アドオンの設定
- 環境変数の設定


### 3. 開発環境の準備
- VS Codeの導入
- （VSCodeのWSL用アドオン）
- VSCodeのPython用アドオン


### 4. oTreeプログラミングの概要
- プロジェクトファイルの構成
- `__init__.py` の構成
- oTree3との変更点
