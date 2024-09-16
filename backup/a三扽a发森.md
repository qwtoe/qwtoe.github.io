mac

安装nodejs时：`curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.0/install.sh | bash`。出现下面报错：

`Failed to connect to raw.githubusercontent.com port 443 after 8 ms: Couldn't connect to server` 


1、
https://www.ipaddress.com 查询 `raw.githubusercontent.com` 域名的 ip 地址。查出四条 ip 地址：
```txt
raw  IN  A  185.199.108.133
raw  IN  A  185.199.109.133
raw  IN  A  185.199.110.133
raw  IN  A  185.199.111.133
```


2、下载SwitchHosts软件
https://github.com/oldj/SwitchHosts/releases

https://switchhosts.vercel.app 

3、加入host

```bash
185.199.108.133 raw.githubusercontent.com
185.199.109.133 raw.githubusercontent.com
185.199.110.133 raw.githubusercontent.com
185.199.111.133 raw.githubusercontent.com
```
记得打开开关。

`ping raw.githubusercontent.com` 成功。


安装node：
https://nodejs.org/en/download/package-manager

配置nvm：
https://juejin.cn/post/7232499180660768829
