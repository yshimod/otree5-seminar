【第3回】 2022年5月26日

<h1>oTreeプログラミングの概要</h1>

- [1. VS Codeの導入](#1-vs-codeの導入)
    - [どのテキストエディタを使うべきか？](#どのテキストエディタを使うべきか)
- [2. シェルで使用するoTreeサブコマンド](#2-シェルで使用するotreeサブコマンド)
    - [サブコマンド](#サブコマンド)
    - [`startproject` と `startapp` コマンドで生成されるもの](#startproject-と-startapp-コマンドで生成されるもの)
- [3. 重要なoTree用語](#3-重要なotree用語)
- [4. 管理者画面](#4-管理者画面)


## 1. VS Codeの導入

- VS Code (Visual Studio Code) [https://code.visualstudio.com/](https://code.visualstudio.com/)
    - Visual Studio（統合開発環境... 単なるテキストエディタではなくコンパイラ等もついている）とは異なる！
- WSL2やVagrantで入れたUbuntuを使っている場合でも，**Windowsに** VS Codeを入れる．
- WSL 拡張機能 [https://docs.microsoft.com/ja-jp/windows/wsl/tutorials/wsl-vscode](https://docs.microsoft.com/ja-jp/windows/wsl/tutorials/wsl-vscode)
- Python 拡張機能 [https://marketplace.visualstudio.com/items?itemName=ms-python.python](https://marketplace.visualstudio.com/items?itemName=ms-python.python)
- VSCodeのチュートリアル（ドットインストール） [https://dotinstall.com/lessons/basic_vscode](https://dotinstall.com/lessons/basic_vscode)


<iframe width="560" height="315" src="https://www.youtube-nocookie.com/embed/p4NOI6n9A4s?rel=0" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>


- 勉強会当日にVS Codeの拡張機能のインストールをやってみましたが，少々グダついたので，該当部分を撮り直して差し替えています．
- 「VSCodeのターミナルは通常のターミナルと同じことができるのか？」という質問をいただきました．
    - 同じシェル（たとえばWSLのUbuntuのbash）に接続している限り，VSCode内蔵のターミナルはOSのターミナルと同じことができる．
    - 接続しているシェル（WindowsならPowerShellかコマンドプロンプトかWSLのbash，Macならzshかbashか...）が異なれば，操作，挙動も変わる．
    - 自分で設定しない限り，WSLのbashにVSCode内蔵のターミナルで接続したときに `~/.bash_profile` や `~/.profile` が読み込まれない（ `~/.bashrc` は読み込まれる）．そのため，環境変数などの設定を `~/.bashrc` ではなく `~/.bash_profile` にしか書いていない場合，その設定が無いままシェルを動かしていることになるため挙動が変わる．oTreeの環境変数の設定に注意．
    - ターミナルの機能（フォントの設定やショートカット）は，VSCode内蔵のターミナルでは（同様の設定をしない限り）使えない．
    - 「シェル」と「ターミナル」は全く異なる概念．だが（特にWindowsの場合は一体として扱われることが多いため）混同しやすい．
        - [平井重行「シェル」](https://www.cc.kyoto-su.ac.jp/~hirai/text/shell.html)
        - [@sirycity「PowerShellやWindowsTerminalの違い(というか関連性)について」](https://jsnotice.com/posts/2021-05-31/)


#### どのテキストエディタを使うべきか？

- **結論: 好きなものを使う．状況に応じて使い分ける．**
- 「VS Code と PyCharm，どちらが良いか？」という質問をいただきました．
    - 私はPyCharmを使ったことがないので分かりません．
    - Webで検索すると，賛否両論のようです．
        - VS Code の方が軽量？
        - PyCharmはPython特化なのでoTreeプログラミングにおいては有用？
- VSCodeが良いな，と思うところ... （拡張機能が豊富なおかげで）いろいろな言語での開発が便利:
    - Jupyter Notebook（ブラウザで作業するのではなく）
    - LaTeX（VSCodeで生成したPDFをプレビューできる）
    - markdown（VSCodeでプレビューできる）
- 専用のエディタを使った方が良い場合も
    - R は RStudio （VS Codeでもある程度似たことはできるが）
    - Wolfram Engine は無料で使えるようになったが，やはり課金してMathematicaを使う方が便利
- 今どきはクラウドで作業することも多い:
    - LaTeX は Overleaf
    - Python (Jupyter) は Google Colaboratory
- おせっかい機能が不要であれば，よりシンプルなエディタを使うのが良い
    - Vim
    - Emacs
    - [秀丸エディタ](https://hide.maruo.co.jp/software/hidemaru.html)
    - [Notepad++](https://notepad-plus-plus.org/)
    - [mi](https://www.mimikaki.net/)
- Windows標準の「メモ帳」を使うときは要注意！
    - （最近は改善されたようですが）文字コード，改行コードの扱いが不親切．
- 「エディタ戦争」
    - [@JJ1LIS「3分でわかるエディタ戦争物語」](https://qiita.com/JJ1LIS/items/22e406ec26ad1e5c6228)
    - [GIGAZINE「戦国時代だったテキストエディタ界をVisual Studio Codeが天下統一しつつある」](https://gigazine.net/news/20210723-the-era-of-visual-studio-code/)
    - [Quoraトピック「Visual Studio CodeはVimやEmacs原理主義者をどのくらい滅ぼしましたか？」](https://jp.quora.com/Visual-Studio-Code%E3%81%AFVim%E3%82%84Emacs%E5%8E%9F%E7%90%86%E4%B8%BB%E7%BE%A9%E8%80%85%E3%82%92%E3%81%A9%E3%81%AE%E3%81%8F%E3%82%89%E3%81%84%E6%BB%85%E3%81%BC%E3%81%97%E3%81%BE%E3%81%97%E3%81%9F%E3%81%8B)


## 2. シェルで使用するoTreeサブコマンド

#### サブコマンド
- [一覧](otree_ref/cmd.md)


#### `startproject` と `startapp` コマンドで生成されるもの

- `settings.py`
    - セッションの設定を記述する．
    - [詳細](otree_ref/settings.md)
- `requirements.txt`
    - pipで導入するパッケージを列挙する．
    - Herokuを使うときに必要なファイル．
    - デフォルト（oTree v5.8.4）の記述内容:
        ```python
        # oTree-may-overwrite-this-file
        # IF YOU MODIFY THIS FILE, remove these comments.
        # otherwise, oTree will automatically overwrite it.
        otree==5.8.4
        psycopg2>=2.8.4
        sentry-sdk>=0.7.9
        ```
    - ↑コメントアウト中にも書かれていますが，「oTree-may-overwrite-this-file」の文字列があると，oTreeが勝手に `requirements.txt` を書き換えるため，自分でライブラリのバージョンを固定したり別のライブラリを追加したりする場合は注意する．
    - 他人からoTreeプロジェクトのファイルをもらったときに，その人と同じパッケージを入れるには，以下のようにpipを使う．
        ```bash
        pip install -r requirements.txt
        ```
- `Procfile`
    - Heroku用の設定ファイル．
    - 拡張子はないが単なるテキストファイル．
    - デフォルト（oTree v5.8.4）の記述内容:
        ```
        web: otree prodserver1of2
        worker: otree prodserver2of2
        ```
    - ↑2行目のworker dynoのコマンドは不要だと思われる．
- `_static` ディレクトリ
    - 画像ファイルやcssファイル，jsファイルを置いておく．
    - ちなみにoTree本体が用意しているstaticの中身は以下:
        ```
        .
        ├── bootstrap5
        │   ├── css
        │   │   ├── bootstrap.min.css
        │   │   └── bootstrap.min.css.map
        │   └── js
        │       ├── bootstrap.bundle.min.js
        │       └── bootstrap.bundle.min.js.map
        ├── favicon.ico
        ├── glyphicons
        │   ├── clock.png
        │   ├── cloud.png
        │   ├── cogwheel.png
        │   ├── delete.png
        │   ├── download-alt.png
        │   ├── eye-open.png
        │   ├── folder-closed.png
        │   ├── link.png
        │   ├── list-alt.png
        │   ├── pencil.png
        │   ├── plus.png
        │   ├── pushpin.png
        │   ├── refresh.png
        │   ├── stats.png
        │   └── usd.png
        ├── otree
        │   ├── css
        │   │   ├── table.css
        │   │   └── theme.css
        │   └── js
        │       ├── common.js
        │       ├── formInputs.js
        │       ├── internet-explorer.js
        │       ├── jquery-3.2.1.min.js
        │       ├── jquery.color-2.1.2.min.js
        │       ├── jquery.countdown.min.js
        │       ├── jquery.timeago.en-short.js
        │       ├── jquery.timeago.js
        │       ├── live.js
        │       ├── monitor2.js
        │       ├── page-websocket-redirect.js
        │       ├── reconnecting-websocket-iife.min.js
        │       └── table-utils.js
        └── robots.txt
        ```
    - ↑これと自分が `_static` ディレクトリに置いたものでパスが重複すると，自分が置いたものが優先される？
    - 画像ファイルならともかく，cssファイル，jsファイルを上書きするのは危険かも．
- `_templates` ディレクトリ
    - 複数アプリ使い回すHTMLテンプレートを置いておく．
- 各アプリのディレクトリ
    - `__init__.py`
    - `*.html`

<iframe width="560" height="315" src="https://www.youtube-nocookie.com/embed/BeNZQ3DkEfg?rel=0" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>



## 3. 重要なoTree用語

<iframe width="560" height="315" src="https://www.youtube-nocookie.com/embed/wKQI2L8wdJQ?rel=0" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

- 詳しくは [https://otree.readthedocs.io/ja/latest/conceptual_overview.html](https://otree.readthedocs.io/ja/latest/conceptual_overview.html)


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

<iframe width="560" height="315" src="https://www.youtube-nocookie.com/embed/ltIQU9trdtg?rel=0" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>



