背景：空白服务器

首先，在https://www.freqtrade.io/en/stable/docker_quickstart/根据提示，通过 docker 进行环境搭建。

运行完下面命令，会在当前目录生成一个 `docker-compose.yml` 文件。
```bash
# Download the docker-compose file from the repository
curl https://raw.githubusercontent.com/freqtrade/freqtrade/stable/docker-compose.yml -o docker-compose.yml

# Pull the freqtrade image
docker compose pull
```
但在 `docker compose pull` 时报错，提示要安装 docker：

```bash
root@lavm-rm2dz6mpkw:ft_userdata# docker compose pull
Command 'docker' not found, but can be installed with:
apt install docker.io      # version 24.0.7-0ubuntu2~22.04.1, or
apt install podman-docker  # version 3.4.4+ds1-1ubuntu1.22.04.2
```

看了下 ubuntu 系统，安装了第一个docker ：
```bash
root@lavm-rm2dz6mpkw:ft_userdata# lsb_release -a
No LSB modules are available.
Distributor ID: Ubuntu
Description:    Ubuntu 22.04.4 LTS
Release:        22.04
Codename:       jammy
root@lavm-rm2dz6mpkw:ft_userdata# apt install docker.io
```

但是，执行 `docker compose pull` 时，还是报错：
```bash
root@lavm-rm2dz6mpkw:ft_userdata# docker compose pull
docker: 'compose' is not a docker command.
See 'docker --help'
```

尝试 `apt-get install docker-compose-plugin` ，

```bash
root@lavm-rm2dz6mpkw:ft_userdata# apt-get install docker-compose-plugin
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
E: Unable to locate package docker-compose-plugin
```
不行，于是尝试 `apt-get install docker-compose` ，安装成功，但是 `docker compose pull` 还是不行。

仔细看了文档：

![image](https://github.com/user-attachments/assets/3951fd4e-cf71-45aa-b33c-ca80c7262c38)


原来改成 `docker-compose pull` 就可以了。

`vim /etc/docker/daemon.json` 添加 docker 镜像：

```json
{
"registry-mirrors": [
   "https://docker.registry.cyou",
   "https://docker-cf.registry.cyou",
   "https://dockercf.jsdelivr.fyi",
   "https://docker.jsdelivr.fyi",
   "https://dockertest.jsdelivr.fyi",
   "https://mirror.aliyuncs.com",
   "https://dockerproxy.com",
   "https://mirror.baidubce.com",
   "https://docker.m.daocloud.io",
   "https://docker.nju.edu.cn",
   "https://docker.mirrors.sjtug.sjtu.edu.cn",
   "https://docker.mirrors.ustc.edu.cn",
   "https://mirror.iscas.ac.cn",
   "https://docker.rainbond.cc"
 ]
}
```

> vim 删除整个文件里面的内容：`:%d`

再 `service docker restsart` 就好了：

```bash
root@lavm-rm2dz6mpkw:ft_userdata# docker-compose pull
Pulling freqtrade ... done
```

运行 `docker-compose run --rm freqtrade create-userdir --userdir user_data` 创建了一个 `user_data` 目录。

运行 `docker compose run --rm freqtrade new-config --config user_data/config.json` ：

![image](https://github.com/user-attachments/assets/5de1b8fb-5d73-4646-960c-122f93bf3abd)





<!-- ##{"script":"<script src='https://blog.meekdai.com/Gmeek/plugins/GmeekVercount.js'></script>"}## -->
