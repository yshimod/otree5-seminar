# HerokuでoTreeを動かす

## まずはアカウントを作る
- [https://signup.heroku.com/](https://signup.heroku.com/)
- とりあえずは無料枠で．

## GitHubのリポジトリからHerokuへデプロイする
- [https://devcenter.heroku.com/articles/github-integration](https://devcenter.heroku.com/articles/github-integration)
- 2022年にGitHubのOAuthトークンが流出し一時的に連携機能が停止．困った困った．  
    [https://blog.heroku.com/we-heard-your-feedback](https://blog.heroku.com/we-heard-your-feedback)

<iframe width="560" height="315" src="https://www.youtube.com/embed/LnrY1AKVmqQ" frameborder="0" allowfullscreen></iframe>

## HerokuのCLIを使い，直接Herokuへデプロイする
- [https://devcenter.heroku.com/articles/git](https://devcenter.heroku.com/articles/git)
- ブラウザでHerokuのappを作っておく．
- gitをインストール．  
    [https://git-scm.com/downloads](https://git-scm.com/downloads)
    - Ubuntuの場合はapt，Macの場合はbrewで入れれば良い．
- Heroku CLIをインストール．  
    [https://devcenter.heroku.com/articles/heroku-cli](https://devcenter.heroku.com/articles/heroku-cli)
    - WSL2でUbuntuを使っている場合は...
        ```bash
        curl https://cli-assets.heroku.com/install-ubuntu.sh | sh
        ```
- Heroku CLIでログインしておく．
    ```bash
    heroku login
    ```
- まずはoTreeプロジェクトを（ローカルの）gitで管理する（GitHubのリモートリポジトリを使うか否かは関係なし）．
    ```bash
    git init
    ```
- oTreeプロジェクトにHeroku用の設定ファイルが入っているか確認．
    - Procfile
    - requirements.txt （pipで入れるもののリスト）
    - Pythonのバージョンを指定する場合はruntime.txt
- Heroku CLI でHerokuのgitサーバーをリモートリポジトリ（herokuという名前）として追加．
    ```bash
    heroku git:remote -a 「APP名」
    ```
    - GitHubを併用している場合，GitHubのリモートリポジトリが「origin」，Herokuのリモートリポジトリが「heroku」という名前で登録されている．
    - **GitHubとgitを明確に区別すること！**
- Herokuのリモートリポジトリにステージングしてコミットしてプッシュ．
    ```bash
    git add .
    git commit -m "first commit"
    git push heroku 「ブランチ名」
    ```
    - ブランチ名はデフォルトなら `master`，特殊なブランチ名（例えば`dev`）なら`dev:master`．
    - つまり，Herokuのリモートリポジトリのブランチ名は`master`（ないし`main`）．
- プッシュすると自動的にデプロイされる．


## デプロイ後にすること
- Heroku Postgresなるアドオンを追加してPostgreSQLを使うように設定（ブラウザ操作）．
- 環境変数の設定（ブラウザ操作）．
- データベースをリセットするときは，「Run console」から `otree resetdb`．
- 本番では必要に応じてDyno，PostgreSQLに課金．
