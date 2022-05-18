# WSL2

## イントロ

- 導入自体は簡単．
- oTreeを動かす段階になって，（主にネットワーク関連で）難易度が少々上がる．
- ドキュメント [https://docs.microsoft.com/ja-jp/windows/wsl/](https://docs.microsoft.com/ja-jp/windows/wsl/)


## 手順

[https://docs.microsoft.com/ja-jp/windows/wsl/setup/environment](https://docs.microsoft.com/ja-jp/windows/wsl/setup/environment)

<iframe width="560" height="315" src="https://www.youtube.com/embed/G3WAFlfOoYM" frameborder="0" allowfullscreen></iframe>


1. `wsl --install`
2. マシンを再起動
3. Ubuntuのユーザ名・パスワードを設定


## 注意

- 使うPCの設定次第では，WSL2のインストールが失敗する可能性があります．Microsoftのドキュメント [https://docs.microsoft.com/ja-jp/windows/wsl/troubleshooting#installation-issues](https://docs.microsoft.com/ja-jp/windows/wsl/troubleshooting#installation-issues) が参考になります．
    - エラー `0x80070003` または エラー `0x80370102` と表示された場合... BIOS（ないしはUEFI）の設定を変更して仮想化支援技術を有効にする必要があります．
        - CPUがIntel製の場合はVT-x，AMD製の場合はAMD-Vを有効にしてください．
        - BIOSのメニューは，使っているPC（のマザーボード）のメーカーによって異なるので，詳しくは検索してください．
        - BIOS画面に入るには，PCの電源を入れた後すぐにDeleteキーやF2キーなどを押し続けます．どのキーを押すのかもメーカーによって異なります．PC起動直後でメーカーのロゴが表示されている最中に，画面のどこかにどのキーを押すべきかが表示されていることが多いです．
