# VirtualBox + Vagrant

## イントロ


- Ubuntuのセットアップの後にやるべきこと（Cのコンパイラ導入など）をスクリプトに書いておけば，自動化できるし同じ環境を再現できる．
- Intel Macでも同じことができる（Apple Silicon は未対応... [Multipassが有用？](https://b.skmoto3.com/posts/20220126-multipass-first/)）．
- マシンの性能（とくにメモリ容量）が弱いと辛いかも．


## 手順

<p class="ytubevideo"><iframe width="560" height="315" src="https://www.youtube-nocookie.com/embed/8GJrRopkoTc?rel=0&origin=https://yshimod.github.io/" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe></p>

1. [VirtualBox](https://www.virtualbox.org/) のインストール
2. [Vagrant](https://www.vagrantup.com/) のインストール
4. Vagrant用の作業用ディレクトリを作る `mkdir myVagrant`
5. PowerShellで作業用ディレクトリに入る `cd myVagrant`
6. `vagrant init ubuntu/focal64` [https://app.vagrantup.com/ubuntu/boxes/focal64](https://app.vagrantup.com/ubuntu/boxes/focal64)
7. Vagrantfileの編集
8. `vagrant up`  
    → インストール完了
9. `vagrant ssh` でシェルに入ってみる


## Vagrantfileの中身

```ruby
# -*- mode: ruby -*-
# vi: set ft=ruby :

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
Vagrant.configure("2") do |config|
  # The most common configuration options are documented and commented below.
  # For a complete reference, please see the online documentation at
  # https://docs.vagrantup.com.

  # Every Vagrant development environment requires a box. You can search for
  # boxes at https://vagrantcloud.com/search.
  config.vm.box = "ubuntu/focal64"

  # Disable automatic box update checking. If you disable this, then
  # boxes will only be checked for updates when the user runs
  # `vagrant box outdated`. This is not recommended.
  # config.vm.box_check_update = false

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine. In the example below,
  # accessing "localhost:8080" will access port 80 on the guest machine.
  # NOTE: This will enable public access to the opened port
  # config.vm.network "forwarded_port", guest: 80, host: 8080

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine and only allow access
  # via 127.0.0.1 to disable public access
  # config.vm.network "forwarded_port", guest: 80, host: 8080, host_ip: "127.0.0.1"

  # Create a private network, which allows host-only access to the machine
  # using a specific IP.
  # config.vm.network "private_network", ip: "192.168.33.10"

  # Create a public network, which generally matched to bridged network.
  # Bridged networks make the machine appear as another physical device on
  # your network.
  config.vm.network "public_network"

  # Share an additional folder to the guest VM. The first argument is
  # the path on the host to the actual folder. The second argument is
  # the path on the guest to mount the folder. And the optional third
  # argument is a set of non-required options.
  # config.vm.synced_folder "../data", "/vagrant_data"

  # Provider-specific configuration so you can fine-tune various
  # backing providers for Vagrant. These expose provider-specific options.
  # Example for VirtualBox:
  #
  config.vm.provider "virtualbox" do |vb|
  #   # Display the VirtualBox GUI when booting the machine
  #   vb.gui = true
  #
    # Customize the amount of memory on the VM:
    vb.memory = "8192"
  end
  #
  # View the documentation for the provider you are using for more
  # information on available options.

  # Enable provisioning with a shell script. Additional provisioners such as
  # Ansible, Chef, Docker, Puppet and Salt are also available. Please see the
  # documentation for more information about their specific syntax and use.
  config.vm.provision "shell", inline: <<-SHELL
    export DEBIAN_FRONTEND=noninteractive
    apt-get update
    apt-get upgrade -y

    ## Python build dependencies
    apt-get install -y make build-essential libssl-dev zlib1g-dev libbz2-dev libreadline-dev libsqlite3-dev wget curl llvm libncursesw5-dev xz-utils tk-dev libxml2-dev libxmlsec1-dev libffi-dev liblzma-dev

    ## PostgreSQL installation
    # Create the file repository configuration:
    sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list'
    # Import the repository signing key:
    wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -
    # Update the package lists:
    apt-get update
    # Install the latest version of PostgreSQL.
    # If you want a specific version, use 'postgresql-12' or similar instead of 'postgresql':
    apt-get install -y postgresql

    ## For psycopg2
    apt-get install -y libpq-dev
  SHELL
end
```

##### プロビジョニング部分の中身
（プロビジョニング: ソフトウェアのインストールなどを自動化すること．）

- Python導入の事前準備 （[https://github.com/pyenv/pyenv/wiki](https://github.com/pyenv/pyenv/wiki) 参考）
- Postgres導入 （[https://www.postgresql.org/download/linux/ubuntu/](https://www.postgresql.org/download/linux/ubuntu/) 参考）
