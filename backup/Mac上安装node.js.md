官网的安装教程 https://nodejs.org/en/download/package-manager ：
```bash
# installs nvm (Node Version Manager)
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.0/install.sh | bash

# download and install Node.js (you may need to restart the terminal)
nvm install 20

# verifies the right Node.js version is in the environment
node -v # should print `v20.17.0`

# verifies the right npm version is in the environment
npm -v # should print `10.8.2`
```

但是执行第一条命令就报错：`Failed to connect to raw.githubusercontent.com port 443 after 8 ms: Couldn't connect to server` 

**解决办法：**


1、https://www.ipaddress.com 查询 `raw.githubusercontent.com` 域名的 ip 地址。查出四条 ip 地址：
```txt
raw  IN  A  185.199.108.133
raw  IN  A  185.199.109.133
raw  IN  A  185.199.110.133
raw  IN  A  185.199.111.133
```


2、下载SwitchHosts软件 

https://github.com/oldj/SwitchHosts/releases

https://switchhosts.vercel.app 

3、SwitchHosts中加入以下host

```bash
185.199.108.133 raw.githubusercontent.com
185.199.109.133 raw.githubusercontent.com
185.199.110.133 raw.githubusercontent.com
185.199.111.133 raw.githubusercontent.com
```
记得打开开关。

最后：`ping raw.githubusercontent.com` 成功，然后执行第一条命令即可。

---

执行完之后，发现nvm并没有生效，于是手动配置nvm：配置nvm：https://juejin.cn/post/7232499180660768829

```bash
1、配置环境变量
    vim ~/.bash_profile
    
2、然后将下面的配置信息输入保存
    export NVM_DIR="$HOME/.nvm"
    [ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh" # This loads nvm
    [ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion" # This loads nvm bash_completion
    
3、刷新环境变量
    source ~/.bash_profile
```

`nvm -v` 成功！

继续根据node官网步骤往下安装就行。


