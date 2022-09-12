# Herokuで oTree を動かす

- 2022年11月28日より Heroku の無料プランが廃止されます．  
[https://blog.heroku.com/next-chapter](https://blog.heroku.com/next-chapter)
- Students and Nonprofit Program も用意されるようですが，実際のところどうなるでしょうか？
- Heroku の代替として [Render](../rendercom/README.md) が選択肢の一つかもしれません．

## まずはアカウントを作る
- [https://signup.heroku.com/](https://signup.heroku.com/)


## (方法1) GitHubのリポジトリからHerokuへデプロイする

<p class="ytubevideo"><iframe width="560" height="315" src="https://www.youtube.com/embed/LnrY1AKVmqQ?rel=0&enablejsapi=1&origin=https://yshimod.github.io/" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe></p>


[https://devcenter.heroku.com/articles/github-integration](https://devcenter.heroku.com/articles/github-integration)


1. ブラウザでHerokuのappを作っておく．
2. Heroku Dashboard​ で GitHub Integration の有効化．
3. リポジトリを指定し，「Deploy Branch」を押してデプロイ開始．


- ~~*2022年4月にGitHubのOAuthトークンが流出し一時的に連携機能が停止．困った困った．*~~  
    ~~[https://blog.heroku.com/we-heard-your-feedback](https://blog.heroku.com/we-heard-your-feedback)~~
- 2022年5月25日に機能が再開したようです．  
    [https://status.heroku.com/incidents/2413](https://status.heroku.com/incidents/2413)


## (方法2) HerokuのCLIを使い直接Herokuへデプロイする

[https://devcenter.heroku.com/articles/git](https://devcenter.heroku.com/articles/git)

1. ブラウザでHerokuのappを作っておく．

2. Gitをインストール．  
    [https://git-scm.com/downloads](https://git-scm.com/downloads)
    - Ubuntuの場合は `apt`，Macの場合は `brew` で入れれば良い．

3. Heroku CLIをインストール．  
    [https://devcenter.heroku.com/articles/heroku-cli](https://devcenter.heroku.com/articles/heroku-cli)
    - WSL2でUbuntuを使っている場合は...
    ```bash
    curl https://cli-assets.heroku.com/install-ubuntu.sh | sh
    ```

4. Heroku CLIでログインしておく．
  ```bash
  heroku login
  ```

5. まずは oTree プロジェクトを（ローカルの）Gitで管理する（初期化する）．
  ```bash
  git init
  ```
    - **すでにGitHubをリモートリポジトリとして使っていてる場合，ここで初期化してはいけない**．

6. oTree プロジェクトにHeroku用の設定ファイルが入っているか確認．
    - `Procfile`
    - `requirements.txt` （pipで入れるもののリスト）
    - Pythonのバージョンを指定する場合は `runtime.txt`

7. Heroku CLI でHerokuのGitサーバーをリモートリポジトリ（herokuという名前）として追加．「APP名」はHeroku Dashboardで確認する．
  ```bash
  heroku git:remote -a 「APP名」
  ```

8. Herokuのリモートリポジトリにステージングしてコミットしてプッシュ．
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

9. プッシュすると自動的にデプロイされる．


## デプロイ後にすること

- Heroku Postgresなるアドオンを追加してPostgreSQLを使うように設定（ブラウザ操作）．
- ログのためにPapertrailなるアドオンを追加してもよし．
- 環境変数の設定（ブラウザ操作）．
  - [環境変数として設定するもの](../server_setup/README.md#envvar)
- データベースをリセットするときは，「Run console」から `otree resetdb`．
- 本番では必要に応じてDyno，PostgreSQLに課金．


## 課金メニューはどれを選ぶ？

- Dyno
    - Webサービス用のDynoが1つだけあれば良い（oTree はマルチタスクに非対応なので2つ以上用意しても無駄）．
    - Free
        - 無料
        - RAM: 512MB
        - 30分間アイドル状態が続くとスリープになる．スリープ時にアクセスするとページが表示されるまでに時間がかかる．
    - Hobby
        - $7/月（2022年9月時点）
        - RAM: 512MB
        - 常時稼動
    - Standard-1X
        - $25/月（2022年9月時点）
        - RAM: 512MB
        - 常時稼動
        - Preboot（必要性は謎）
    - Standard-2X
        - $50/月（2022年9月時点）
        - RAM: 1GB
        - 常時稼動
        - Preboot（必要性は謎）

- Postgres
    - Postgres アドオンを使わなくても，SQLiteで oTree を動かすことは可能だが，Dynoが自動的に再起動するたびにSQLiteのデータは消えてしまう点に注意．動作確認で無い限りPostgresアドオンは使っておいたほうが良い．
    - レコード行数制限に注目すべき．oTree は，1アプリ1参加者あたり，少なくとも3レコード（Player，Group，Subsession の各テーブル）が必要．ラウンドやアプリが複数あればその分レコード数は増えるし，ExtraModelを使う場合は更に（爆発的に）レコード数が増えるかもしれない．
        - こまめに自分でデータをダンプして，こまめに `otree resetdb` でリセットする，という運用も可能．
    - 同時接続数の制限は気にしなくても良いはず．なんとなればすなわち，oTree はシングルプロセスなのでPostgresへの接続数は高々1だから．外部からデータベースに直接接続する場合には注意．
    - Hobby Dev
        - 無料
        - レコード行数制限: 1万件
        - ストレージ: 1GB
        - インメモリキャッシュなし（データへのアクセスに時間がかかる）
    - Hobby Basic
        - $9/月（2022年9月時点）
        - レコード行数制限: 1000万件
        - ストレージ: 10GB
        - インメモリキャッシュなし（データへのアクセスに時間がかかる）
    - Standard 0
        - $50/月（2022年9月時点）
        - レコード行数制限: 無制限
        - ストレージ: 64GB
        - RAM: 4GB


- 実践例:
    - Dyno: Standard-1X （$25/月）（選んだ理由: なんとなく，大は小を兼ねると思って．）
    - Postgres: Standard 0 （$50/月）（選んだ理由: なんとなく，大は小を兼ねると思って．）
    - 使用状況: 同時に最大16人が接続
    - Memory Usage: 最大 123MB
    - Response Time: 最大 1秒
    - Throughput: 
        - 2XXレスポンス: 最大 0.3rps
        - 3XXレスポンス: 最大 0.2rps
    - Dyno Load: 最大 0.8 (/1)
    - レコード行数: 1万件を超過
    - 感想: Dyno は Hobby， Postgres は Hobby Basic で十分だったかも．Postgres は使い方次第では Standard 0 まで必要になることがあるかもしれない．
