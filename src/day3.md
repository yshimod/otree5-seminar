【第3回】 2022年5月26日

# VS Code の導入 ・ oTree プログラミングの概要


## VS Code の導入

- VS Code (Visual Studio Code) [https://code.visualstudio.com/](https://code.visualstudio.com/)
    - Visual Studio（統合開発環境... 単なるテキストエディタではなくコンパイラ等もついている）とは異なる！

- WSL2やVagrantで入れたUbuntuを使っている場合でも，**Windowsに** VS Codeを入れる．

- WSL 拡張機能 [https://docs.microsoft.com/ja-jp/windows/wsl/tutorials/wsl-vscode](https://docs.microsoft.com/ja-jp/windows/wsl/tutorials/wsl-vscode)

- Python 拡張機能 [https://marketplace.visualstudio.com/items?itemName=ms-python.python](https://marketplace.visualstudio.com/items?itemName=ms-python.python)

- VSCodeのチュートリアル（ドットインストール） [https://dotinstall.com/lessons/basic_vscode](https://dotinstall.com/lessons/basic_vscode)


<p class="ytubevideo"><iframe width="560" height="315" src="https://www.youtube.com/embed/p4NOI6n9A4s?rel=0&enablejsapi=1&origin=https://yshimod.github.io/" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe></p>


- 勉強会当日にVS Codeの拡張機能のインストールをやってみましたが，少々グダついたので，該当部分を撮り直して差し替えています．

- 「VSCodeのターミナルは通常のターミナルと同じことができるのか？」という質問をいただきました．
    - 同じシェル（たとえばWSLのUbuntuのbash）に接続している限り，VSCode内蔵のターミナルはOSのターミナルと同じことができる．
    - 接続しているシェル（WindowsならPowerShellかコマンドプロンプトかWSLのbash，Macならzshかbashか...）が異なれば，操作，挙動も変わる．
    - 自分で設定しない限り，WSLのbashにVSCode内蔵のターミナルで接続したときに `~/.bash_profile` や `~/.profile` が読み込まれない（ `~/.bashrc` は読み込まれる）．そのため，環境変数などの設定を `~/.bashrc` ではなく `~/.bash_profile` にしか書いていない場合，その設定が無いままシェルを動かしていることになるため挙動が変わる． oTree の環境変数の設定に注意．
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
        - PyCharmはPython特化なので oTree プログラミングにおいては有用？

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



## シェルで使用する oTree サブコマンド

#### サブコマンド

- [シェルで使用する oTree サブコマンド](otree_ref/cmd.md)


#### `startproject` と `startapp` コマンドで生成されるもの

以下のディレクトリ・ファイルは oTree を動かすために最低限必要なもの．ただし， `Procfile` と `requirements.txt` は Heroku にデプロイするために必要なものであり， Heroku を使わないのであれば不要．


- `settings.py`
    - セッションの設定を記述する．
    - [`settings.py`の書き方](otree_ref/settings.md)

- `requirements.txt`
    - pipで導入するパッケージを列挙する．
    - Herokuを使うときに必要なファイル．
    - デフォルト（ oTree v5.8.4）の記述内容:
      ```python
      # oTree-may-overwrite-this-file
      # IF YOU MODIFY THIS FILE, remove these comments.
      # otherwise, oTree will automatically overwrite it.
      otree==5.8.4
      psycopg2>=2.8.4
      sentry-sdk>=0.7.9
      ```
    - ↑ コメントアウト中にも書かれているとおり，「oTree-may-overwrite-this-file」の文字列があると， oTree が勝手に `requirements.txt` を書き換えるため，自分でライブラリのバージョンを固定したり別のライブラリを追加したりする場合は注意する．
    - 他人から oTree プロジェクトのファイルをもらったときに，その人と同じパッケージを入れるには，以下のようにpipを使う．
      ```bash
      pip install -r requirements.txt
      ```

- `Procfile`
    - Heroku用の設定ファイル．
    - 拡張子はないが単なるテキストファイル．
    - デフォルト（ oTree v5.8.4 ）の記述内容:
      ```
      web: otree prodserver1of2
      worker: otree prodserver2of2
      ```
    - ↑ 2行目のworker dynoのコマンドは不要だと思われる．

- `_static` ディレクトリ
    - 画像ファイルやCSSファイル，JSファイルを置いておく．

- `_templates` ディレクトリ
    - 複数アプリ使い回すHTMLテンプレートを置いておく．

- 各アプリのディレクトリ
    - `__init__.py`
    - `*.html`


<p class="ytubevideo"><iframe width="560" height="315" src="https://www.youtube.com/embed/BeNZQ3DkEfg?rel=0&enablejsapi=1&origin=https://yshimod.github.io/" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe></p>



## oTree のデータモデル

<p class="ytubevideo"><iframe width="560" height="315" src="https://www.youtube.com/embed/wKQI2L8wdJQ?rel=0&enablejsapi=1&origin=https://yshimod.github.io/" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe></p>


- 詳しくは [https://otree.readthedocs.io/ja/latest/conceptual_overview.html](https://otree.readthedocs.io/ja/latest/conceptual_overview.html)


## 管理者画面

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


<p class="ytubevideo"><iframe width="560" height="315" src="https://www.youtube.com/embed/ltIQU9trdtg?rel=0&enablejsapi=1&origin=https://yshimod.github.io/" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe></p>
