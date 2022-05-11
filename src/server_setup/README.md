# Server setup

公式ドキュメント [https://otree.readthedocs.io/en/latest/server/ubuntu.html](https://otree.readthedocs.io/en/latest/server/ubuntu.html)

## 目的

- oTreeを導入するための環境を準備する．
- Windowsではなく，LinuxやmacOSなどのUNIX系OSでoTreeを動かしたい（理由は後述）．
- ↑ の簡単な方法はMacを買うか，Windowsを消してUbuntuなどのLinux OSを入れ直すこと．
- ↑ そうはいかない人が多いことも理解できるので，Windows上にLinuxの仮想環境を構築する．


## なぜWindowsではダメか？

ダメではないが......

#### Linuxの使い方に慣れましょう
- サーバ用のOSはLinuxを採用することが多い．
- 社研ラボのoTree用サーバのOSはUbuntu（Linux OSの中で一番有名？）．
- AWSのEC2やGCPのCompute EngineのOSもたいていLinux．

#### 屋上屋を架する愚は避ける
- oTreeの使い方を解説する（初級者向け）ドキュメントはたいていWindowsを使っている．
- Windowsに直接Python・oTreeを入れる方法は公式ドキュメント [https://otree.readthedocs.io/en/latest/install-windows.html#install-windows](https://otree.readthedocs.io/en/latest/install-windows.html#install-windows) を参照してください．
- UNIXな環境を前提としたドキュメントは少ない．公式ドキュメントでも説明が雑（MacやLinuxが使える人に詳しい説明は不要と思われている？その推論はいくらか正しいとしても......）．


## マシンを用意してLinuxを入れる

- 実機に
- [Ubuntuの仮想環境の構築](../ubuntu/README.md) （勉強用？）
    - 以下ではWSL2でUbuntuを動かします


## Python を pyenv (+ virtualenv) で導入

<iframe width="560" height="315" src="https://www.youtube.com/embed/xOPHDOUsg0c" frameborder="0" allowfullscreen></iframe>

- Pythonのビルド環境 [https://github.com/pyenv/pyenv/wiki](https://github.com/pyenv/pyenv/wiki)
- pyenv [https://github.com/pyenv/pyenv](https://github.com/pyenv/pyenv)
- pyenv-virtualenv [https://github.com/pyenv/pyenv-virtualenv](https://github.com/pyenv/pyenv-virtualenv)
- psycopg2 [https://www.psycopg.org/docs/](https://www.psycopg.org/docs/)


## PostgreSQL を導入

<iframe width="560" height="315" src="https://www.youtube.com/embed/uYNYo1IICvA" frameborder="0" allowfullscreen></iframe>
