# HerokuでoTreeを動かす

- [HerokuでoTreeを動かす](#herokuでotreeを動かす)
  - [まずはアカウントを作る](#まずはアカウントを作る)
  - [(方法1) GitHubのリポジトリからHerokuへデプロイする](#方法1-githubのリポジトリからherokuへデプロイする)
  - [(方法2) HerokuのCLIを使い直接Herokuへデプロイする](#方法2-herokuのcliを使い直接herokuへデプロイする)
  - [デプロイ後にすること](#デプロイ後にすること)
  - [課金メニューはどれを選ぶ？](#課金メニューはどれを選ぶ)


## まずはアカウントを作る
- [https://signup.heroku.com/](https://signup.heroku.com/)
- とりあえずは無料枠で．

## (方法1) GitHubのリポジトリからHerokuへデプロイする

<iframe width="560" height="315" src="https://www.youtube-nocookie.com/embed/LnrY1AKVmqQ?rel=0" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>


[https://devcenter.heroku.com/articles/github-integration](https://devcenter.heroku.com/articles/github-integration)


    1. ブラウザでHerokuのappを作っておく．
    1. Heroku Dashboard​ で GitHub Integration の有効化．
    1. リポジトリを指定し，「Deploy Branch」を押してデプロイ開始．
- *2022年4月にGitHubのOAuthトークンが流出し一時的に連携機能が停止．困った困った．*  
    [https://blog.heroku.com/we-heard-your-feedback](https://blog.heroku.com/we-heard-your-feedback)


## (方法2) HerokuのCLIを使い直接Herokuへデプロイする

[https://devcenter.heroku.com/articles/git](https://devcenter.heroku.com/articles/git)

    1. ブラウザでHerokuのappを作っておく．
    1. Gitをインストール．  
        [https://git-scm.com/downloads](https://git-scm.com/downloads)
        - Ubuntuの場合はapt，Macの場合はbrewで入れれば良い．
    1. Heroku CLIをインストール．  
        [https://devcenter.heroku.com/articles/heroku-cli](https://devcenter.heroku.com/articles/heroku-cli)
        - WSL2でUbuntuを使っている場合は...
            ```bash
            curl https://cli-assets.heroku.com/install-ubuntu.sh | sh
            ```
    1. Heroku CLIでログインしておく．
        ```bash
        heroku login
        ```
    1. まずはoTreeプロジェクトを（ローカルの）Gitで管理する（初期化する）．  
        **すでにGitHubをリモートリポジトリとして使っていてる場合，ここで初期化してはいけない**．
        ```bash
        git init
        ```
    1. oTreeプロジェクトにHeroku用の設定ファイルが入っているか確認．
        - Procfile
        - requirements.txt （pipで入れるもののリスト）
        - Pythonのバージョンを指定する場合はruntime.txt
    1. Heroku CLI でHerokuのGitサーバーをリモートリポジトリ（herokuという名前）として追加．「APP名」はHeroku Dashboardで確認する．
        ```bash
        heroku git:remote -a 「APP名」
        ```
    1. Herokuのリモートリポジトリにステージングしてコミットしてプッシュ．
        ```bash
        git add .
        git commit -m "first commit"
        git push heroku 「ブランチ名」:main
        ```
        - GitHubを併用している場合，GitHubのリモートリポジトリが「origin」，Herokuのリモートリポジトリが「heroku」という名前で登録されている．
        - **GitHubとGitを明確に区別すること！**
        - Herokuのリモートリポジトリのブランチ名は`main`（ないし`master`）．
        - ローカルのブランチ名はデフォルトなら `master`．`git push heroku master:main` ないしは `git push heroku master`でプッシュできる．
        - ローカルのブランチ名が，たとえば`dev`であれば，`git push heroku dev:main`．
    1. プッシュすると自動的にデプロイされる．


## デプロイ後にすること
- Heroku Postgresなるアドオンを追加してPostgreSQLを使うように設定（ブラウザ操作）．
- ログのためにPapertrailなるアドオンを追加してもよし．
- 環境変数の設定（ブラウザ操作）．
- データベースをリセットするときは，「Run console」から `otree resetdb`．
- 本番では必要に応じてDyno，PostgreSQLに課金．


## 課金メニューはどれを選ぶ？
- 執筆中．
