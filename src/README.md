# oTree5 勉強会

## 目標
- oTree（バージョン5以降）を使い，実験プログラムの作成から，実験の実施までを一人でできるようになる．


## 2022年5月12日

- oTreeの動かし方
    - オンプレミス（e.g., 社研サーバー），IaaS（e.g., Amazon EC2, Google Compute Engine），VPS（e.g., ConoHa）などのLinuxサーバーで動かす  
    [Server setup](server_setup/README.md)
    - PaaS（e.g., Heroku）で動かす（oTree公式のおすすめ）
    - Windowsで動かす（たとえば，ラボでzTreeを動かしていたようなマシンで）


### 事前準備
- Heroku のアカウントを作成 [https://signup.heroku.com](https://signup.heroku.com)
- GitHub のアカウントを作成 [https://github.com/signup](https://github.com/signup) （後々のために）

##### 実験実施でLinuxサーバーを使う予定が無い場合，以下は不要
- Ubuntu環境を用意する  
    - Windowsユーザーは以下のいずれかの方法で，仮想環境としてUbuntuを使えるようにしておきましょう．
        - [WSL2 を使う](ubuntu/wsl2.md) （おすすめ）
        - [VirtualBox + Vagrant を使う](ubuntu/vagrant.md) （Intel Macユーザーも同様の方法で構築可能）
    - MacはLinuxと（ほとんど）同様に使うことができるので，あえてMac上に仮想環境を構築する必要はありません．
