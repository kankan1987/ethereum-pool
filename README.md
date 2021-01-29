# ethereum-pool 以太坊矿池





### 构建平台 Linux （ubuntu）

依赖环境:

    apt install software-properties-common unzip nginx tor pcscd pcsc-tools git curl htop mc unrar screen

**以下步骤是在aws g2实例上完成的**


安装以太坊节点:

    sudo su
    mkdir /pool && cd /pool
    wget https://github.com/multi-geth/multi-geth/releases/download/v1.9.7/multi-geth-linux.zip
    unzip multi-geth-linux.zip
    ln -s /pool/geth /usr/bin/geth
    chmod ugo+x /usr/bin/geth

    geth --classic account new
    geth --classic account list

生成的账户信息如下:

Public address of the key: 0xa9c96bff41FFAFb6bE8c2990c0CF4F3b220aaEaa
Path of the secret key file: /root/.ethereum/classic/keystore/UTC--2020-03-11T22-03-07.512690005Z--a9c96bff41ffafb6be8c2990c0cf4f3b220aaeaa

创建启动脚本:

    vim /pool/start_geth.sh

    #!/bin/bash
    screen -S server geth --classic --rpc --maxpeers 75 --syncmode "fast" --rpcapi "eth,net,web3,personal" --etherbase "YOUR_WALLET" --cache=12288 --mine --unlock "YOUR_WALLET" --allow-insecure-unlock --password /pool/pwd

创建账户密码文件并给启动文件赋权限：

    nano /pool/pwd

    chmod +x /pool/start_geth.sh

启动节点并开始同步节点:

    ./start_geth.sh

安装以太坊矿池:

    sudo su
    cd /pool
    git config --global http.https://gopkg.in.followRedirects true
    git clone https://github.com/kankan1987/ethereum-pool.git
    cd open-ethereum-pool
    chmod +x ./build/env.sh
    add-apt-repository ppa:longsleep/golang-backports
    apt update
    apt install golang-go
    make
    add-apt-repository ppa:chris-lea/redis-server
    apt update
    apt install redis-server
    systemctl enable redis-server.service && systemctl stop redis-server.service && systemctl start redis-server.service

    curl -sL https://deb.nodesource.com/setup_13.x | bash -
    curl -sL https://dl.yarnpkg.com/debian/pubkey.gpg | apt-key add -
    echo "deb https://dl.yarnpkg.com/debian/ stable main" | tee /etc/apt/sources.list.d/yarn.list

    apt update
    apt install nodejs yarn


### 编译前端代码

    cd www/

修改文件中默认IP 192.168.0.200 为你自己的机器IP  :

    vim config/environment.js

编译命令如下:

    sudo npm install -g ember-cli@2.9.1
    sudo npm install -g bower
    sudo npm install fsevents@latest -f --save-optional
    sudo npm install intl-format-cache@4.2.22
    sudo npm install format-number
    sudo npm install ember-cli-accounting
    sudo npm install core-js@3.6.4
    sudo npm install jquery@3.4.0
    sudo npm install @babel/core@^7.0.0-beta.42
    sudo npm install babel-plugin-debug-macros@0.2.0
    sudo npm install ember-intl@4.3.0
    sudo npm install minimatch@3.0.2
    sudo npm install ember-cli-babel@7.18.0
    sudo npm install ember-resolver@7.0.0
    sudo npm install
    sudo npm audit fix
    sudo npm audit fix --force
    bower install --allow-root
    wget  https://files.gitter.im/sammy007/open-ethereum-pool/IBJl/intl-format-cache.rar
    unrar x intl-format-cache.rar node_modules/intl-format-cache/ -Y
    chmod +x build.sh
    ./build.sh
    cd ../
    cp misc/nginx-default.conf /etc/nginx/sites-available/default
    systemctl enable nginx.service && systemctl stop nginx.service && systemctl start nginx.service
    screen -S pool ./build/bin/ethash-mining-pool config_api.json
