【第2回】 2022年5月19日

# GitとGitHubの使い方 ・ Herokuの使い方


## 0. 準備

- 結局どうにかして（PostgreSQLうんぬんは無視してでも） oTree をインストールする．
    - WindowsにPythonを直接インストールして，pipで oTree を入れる．  
        [https://otree.readthedocs.io/en/latest/install-windows.html#install-windows](https://otree.readthedocs.io/en/latest/install-windows.html#install-windows)  
        - 【フリーライド】立命館竹内研長瀬さんのチュートリアル動画が参考になります．  
            「#2 oTree Windowsでのインストール」
            （動画リンクは竹内研Slack内を検索してください．）
    - 頑張ってWSL2を使い，Ubuntu環境を用意してから...  
        1. [WSL2のインストール](ubuntu/wsl2.md)
        1. [Ubuntuで oTree を動かす](server_setup/README.md)
    - Macの場合はまずhomebrewをインストールしてから...
        - 【フリーライド】立命館竹内研長瀬さんのチュートリアル動画が参考になります．  
            「#2 oTree Macでのインストール」
            （動画リンクは竹内研Slack内を検索してください．）


- Herokuのアカウントを作成． [https://signup.heroku.com/login](https://signup.heroku.com/login)
- GitHubのアカウントを作成． [https://github.com/signup](https://github.com/signup)



## 1. GitとGitHubを使うメリット

<p class="ytubevideo"><iframe width="560" height="315" src="https://www.youtube.com/embed/mqX7XHzyoXs?rel=0&enablejsapi=1&origin=https://yshimod.github.io/" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe></p>


- Gitはローカル環境においてバージョン管理を行うソフトウェア．
    - ファイルを上書き保存していくと開発の過程が記録されない．
    - ファイル名に枝番をつける的な手動バージョン管理ではしっちゃかめっちゃかに．
    - Gitでシステマティックにバージョン管理をしよう！
- GitHub（クラウド）にローカルのGitで管理しているリポジトリをアップロードできる．
    - クラウドを介して複数人での開発が効率化できる．
    - 複数のマシンで1つのプロジェクトを管理できる．
    - GitHubはホスティングサービスとしても有用．
    - GitHubとHerokuとの連携も便利 ~~（2022年5月現在使えないけど）~~ ．
    - （GitHubのほかにGitLabもある．）
- 研究活動でのGitHubの活用例
    - 国里愛彦ラボ [https://kunisatolab.github.io/main/how-to-github.html](https://kunisatolab.github.io/main/how-to-github.html)


## 2. GitとGitHubの使い方

#### 参考資料
  - 公式の教科書 [https://git-scm.com/book/en/v2](https://git-scm.com/book/en/v2)
  - 名城大理工学部大原研 [https://www1.meijo-u.ac.jp/~kohara/cms/technicalreport](https://www1.meijo-u.ac.jp/~kohara/cms/technicalreport)
  - ドットインストール git入門 [https://dotinstall.com/lessons/basic_git](https://dotinstall.com/lessons/basic_git) （無料で視聴可能）
  - Webでも書籍でも，Gitに関するドキュメントは大量に存在する．


#### 初期設定

<p class="ytubevideo"><iframe width="560" height="315" src="https://www.youtube.com/embed/xnznSzfV_6g?rel=0&enablejsapi=1&origin=https://yshimod.github.io/" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe></p>

1. Gitのインストール
    - Windowsに直接Pythonを入れて oTree を動かす場合，Gitのインストーラーを [https://gitforwindows.org/](https://gitforwindows.org/) からダウンロードしてインストールする．
        - 参考 [https://www.curict.com/item/60/60bfe0e.html](https://www.curict.com/item/60/60bfe0e.html)

    - Ubuntuを使う場合，以下のコマンドでインストール．

    ```bash
    sudo apt update
    sudo apt install git
    ```

    - Macを使う場合，Xcode Command Line Toolsが導入済みならばすでに `git` コマンドは使えるはず．あるいはhomebrewで入れる．

1. GitHubのアカウントを作る． [https://github.com/signup](https://github.com/signup)

1. GitHubで登録しているユーザー名とメールアドレスをローカルで設定（コミット毎にユーザー名とメールアドレスが記録される）．

  ```bash
  git config --global user.email "xxxxxxxx@example.com"
  git config --global user.name "namaenamae"
  ```

1. GitHubへSSHで接続できるようにする．  
    [https://docs.github.com/en/authentication/connecting-to-github-with-ssh/about-ssh](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/about-ssh)

    1. SSHの鍵を作る．オプション`-C`以下でGitHubで登録したメールアドレスを陽に指定する．

    ```bash
    ssh-keygen -t ed25519 -C "xxxxxxxx@example.com"
    ```

        - 鍵の場所は `~/.ssh` ．

        - 鍵にパスフレーズは不要（設定するなら鍵をssh-agentに登録する必要あり）．

    1. 公開鍵（`*.pub`）をGitHubに登録．  
        [https://docs.github.com/en/authentication/connecting-to-github-with-ssh/adding-a-new-ssh-key-to-your-github-account](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/adding-a-new-ssh-key-to-your-github-account)

        - `cat ~/.ssh/鍵ファイル名.pub` で公開鍵の中身を表示し，コピーしておく．

        - GitHub [https://github.com/settings/keys](https://github.com/settings/keys) をブラウザで開き，「New SSH key」をクリックし，コピーした公開鍵の中身をペーストする．

    1. `~/.ssh/config` にSSH鍵関係の情報を追記．

    ```
    Host github.com
            IdentityFile ~/.ssh/鍵ファイル名
            User git
    ```

    1. アクセスできるか確認．

    ```bash
    ssh -T git@github.com
    ```

    1. すでにGitHubに（プライベート）リポジトリがあればクローンしてみる．

    ```bash
    git clone git@github.com:namaenamae/rrrrrrrrrr.git
    ```


#### 基本的な手順

[https://learnxinyminutes.com/docs/git/](https://learnxinyminutes.com/docs/git/)


<p class="ytubevideo"><iframe width="560" height="315" src="https://www.youtube.com/embed/LK7CFRvSTlc?rel=0&enablejsapi=1&origin=https://yshimod.github.io/" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe></p>


1. （初回のみ）初期化... 作業ディレクトリに移動して，そのディレクトリをGitリポジトリとして設定（`.git`ディレクトリができる）．

  ```bash
  git init
  ```

1. （初回のみ）ブラウザでGitHubを開き，作業ディレクトリと同名のリポジトリを作成．リモートリポジトリを設定．

  ```bash
  git remote add origin git@github.com:namaenamae/rrrrrrrrrr.git
  ```

1. 作成・編集したファイルをステージングエリアに追加する．たとえばディレクトリ全体なら...

  ```bash
  git add .
  ```

1. 「キリのよいところで」コミットする．

  ```bash
  git commit -m "first commit"
  ```

    - オプション `-m` 以下を指定しない場合はVimかnanoが起動するので，それでコメントを書く．

1. ある程度コミットが溜まったら，GitHubへプッシュ（アップロード）．

  ```bash
  git push -u origin master
  ```
    - オプション `-u` は2回目以降では不要．

    - ブランチ名はデフォルトで `master`になっているはずだが要確認．  
        GitHubはポリコレ的にデフォルトブランチ名を `main` に変えることを勧めている．

1. 明くる日作業を再開するときには，それまでの（他人の）変更分をダウンロードする．

  ```bash
  git pull origin master
  ```

1. 1日の作業の流れ: `pull` -> ( 編集 -> `add` -> `commit` ) の繰り返し -> `push`


#### ブランチの操作

- ブランチを切る（たとえば開発用に`namaenamae/dev`なるブランチを作ってそこに切り替える）．

  ```bash
  git checkout -b namaenamae/dev
  ```

- ブランチを切り替える（たとえば`master`に戻る）．

  ```bash
  git checkout master
  ```

- `master`ブランチに`namaenamae/dev`ブランチの変更分をマージするには，`git checkout master`した後，

  ```bash
  git merge namaenamae/dev
  ```

- ↑ のように `merge` でマージできるが，節目にはGitHubでプルリクエストしてマージ分を誰かにレビューしてもらった方が良い．


## 3. Herokuの使い方

- [詳しくはこちら](heroku/README.md)


#### デプロイの方法

<p class="ytubevideo"><iframe width="560" height="315" src="https://www.youtube.com/embed/bfrCwKshc74?rel=0&enablejsapi=1&origin=https://yshimod.github.io/" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe></p>


#### アドオンと環境変数の設定

<p class="ytubevideo"><iframe width="560" height="315" src="https://www.youtube.com/embed/8rkwfkv6sTg?rel=0&enablejsapi=1&origin=https://yshimod.github.io/" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe></p>
