背景：Ubuntu 24.04，Codex CLI v0.147.0。人在国内，OpenAI/ChatGPT 的流量全部走 Xray 代理（VLESS+WebSocket 隧道，Docker 部署，v2rayA 镜像，监听 10808 端口）。

现象很典型：

- `codex login`：正常
- `codex exec`（非交互模式）：正常
- `codex`（交互式 TUI 模式）：界面能正常显示，但只要输入一条消息（比如 `test`），立刻提示 "Conversation interrupted"，随后进入 "Request timed out" 的无限重连循环

```bash
$ codex
> test
Conversation interrupted
Request timed out
# 开始不断重连
```

`codex exec` 能通、TUI 却断，说明不是账号或基础链路的问题，而是某个环节在交互模式下不稳定。开查。

## 第一阶段：怀疑代理配置

既然非交互模式正常，第一个怀疑对象就是 WebSocket 长连接 —— 交互模式的消息收发走 WebSocket，普通 HTTP 请求不依赖长连接。把代理的各条路径挨个测了一遍：

```bash
# SOCKS5 代理
curl -x socks5h://127.0.0.1:10808 https://chatgpt.com

# HTTP CONNECT 代理
curl -x http://127.0.0.1:10808 https://chatgpt.com

# WebSocket 长连接（保持 120s）
curl -x http://127.0.0.1:10808 -H "Connection: Upgrade" -H "Upgrade: websocket" \
     -H "Sec-WebSocket-Version: 13" -H "Sec-WebSocket-Key: dGhlIHNhbXBsZSBub25jZQ==" \
     https://chatgpt.com
```

结果：全部通过。Xray 日志里也能看到连接被正常接受和转发。`codex doctor` 的 WebSocket 测试则时好时坏 —— 偶尔通过，偶尔超时。代理本身看起来没问题，但正是这个"时好时坏"让我放不下。

## 第二阶段：怀疑 mux 多路复用

注意到 Xray 配置里 `mux: false`（多路复用被关闭）。多路复用和长连接的组合经常出问题，试试开启：

```bash
# 配置文件在宿主机上，容器里是只读 bind-mount
vim /tmp/xray-config/config.json
# mux: false -> true，保存

# 重启容器
docker restart v2raya
```

结果：**更糟糕**，立刻改回。注意 Xray 配置是只读挂载进容器的，必须先改宿主机的 `/tmp/xray-config/config.json` 再重启。这个方向排除。

## 第三阶段：关键发现 —— IPv6

回头仔细看 `codex doctor` 的输出，发现一行之前忽略的信息：

```text
DNS preference: first IPv6
```

DNS 优先解析 IPv6？验证一下：

```bash
getent ahosts chatgpt.com
# 返回结果第一行是 IPv6 地址，IPv4 排在后面
```

再测试强制 IPv6 / IPv4 走 HTTP 代理：

```bash
# 强制 IPv6 + HTTP 代理：立刻失败
curl -6 -x http://127.0.0.1:10808 https://chatgpt.com

# 强制 IPv4 + HTTP 代理：正常
curl -4 -x http://127.0.0.1:10808 https://chatgpt.com
```

破案了。问题不是代理，是 **IPv6**。

## 根因分析

- HTTP 代理模式下，DNS 由**客户端**解析。系统 DNS 优先返回 IPv6，Codex 就优先尝试连 IPv6
- 但 Xray 跑在 Docker 里，这个环境**没有 IPv6 出口**，IPv6 连接必然超时
- WebSocket 长连接对超时特别敏感，一超时就报 "Conversation interrupted"，然后进入重连死循环
- 而 `socks5h://` 的 DNS 由**代理端**解析，代理端拿到的是 IPv4 地址，所以 SOCKS5 一直正常
- `codex exec` 偶尔能用：它走 Happy Eyeballs（RFC 6555），IPv6 失败后会回落到 IPv4，比交互模式的 WebSocket 宽容得多

## 解决方案

### Fix 1（推荐，系统级）：让 DNS 优先 IPv4

编辑 `/etc/gai.conf`，把 IPv4（IPv4-mapped IPv6）的优先级调到比 IPv6 高：

```bash
sudo vim /etc/gai.conf
```

```ini
precedence ::ffff:0:0/96  100
```

`::ffff:0:0/96` 是 IPv4 在 IPv6 地址空间中的映射前缀，RFC 3484 默认优先级只有 10，低于 IPv6 的 40；改成 100 后系统会优先返回 IPv4 地址。改完立即生效，无需重启。

### Fix 2（用户级，无需 root）

不想动系统配置的话，可以在 `~/.bashrc` 里加：

```bash
echo 'export RES_OPTIONS="inet4"' >> ~/.bashrc
source ~/.bashrc
```

**注意两点：**

1. Codex 是静态链接（musl）的二进制，`RES_OPTIONS` 是 glibc 的环境变量，对它是否生效需要实测确认
2. 改完环境变量后，一定要杀掉残留的 app-server 进程 —— 那个 8 月 9 号就在跑的旧进程继承的是旧环境变量，不杀不会生效：

```bash
# 查看残留进程
ps aux | grep codex

# 杀掉旧进程，让它用新环境变量重新拉起
pkill -f "codex app-server"
```

验证：

```bash
codex doctor
# DNS preference 应显示 first IPv4
```

## 总结

1. **HTTP 代理和 `socks5h://` 的 DNS 解析位置不同**：HTTP 代理由客户端解析 DNS，`socks5h://` 由代理端解析。前者受客户端系统 DNS 配置影响，后者不受 —— 这就是为什么 SOCKS5 一直正常
2. **Docker/代理环境普遍没有 IPv6 出口**，但现代系统默认优先 IPv6。排查"走代理就连不上"的问题时，`curl -6` / `curl -4` 对比 + `getent ahosts` 是最快的定位手段，`/etc/gai.conf` 的 `precedence` 是标准解法
3. **静态链接二进制不认 glibc 环境变量**：musl 静态链接的程序对 `RES_OPTIONS` 这类 glibc 专属变量可能无感，改了环境变量要验证是否真正生效
4. **改环境变量后要杀常驻进程**：后台的 app-server 继承的是启动时的环境，不重启就永远不会用到新配置

最终 `codex` 交互模式恢复正常，输入消息不再报 "Conversation interrupted"。

<!-- ##{"script":"<script src='https://blog.meekdai.com/Gmeek/plugins/GmeekVercount.js'></script>"}## -->