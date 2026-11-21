# 浏览器能上网，Ubuntu终端为什么下载失败？

作者：Eating-King-Git。根据本人操作记录与官方文档整理，含 AI 辅助写作和核查。整理日期：2026-11-21。

我在Ubuntu配置软件时遇到过一个很直观的矛盾：浏览器能打开网页，终端却提示无法解析`security.ubuntu.com`。后来显式配置终端使用的代理，一次脚本下载达到了100%；后续环境记录也确认过APT、Git、curl和pip的下载成功。

这些结果证明相应时点、相应程序的请求走通了，却不能说明所有程序永久共享了同一套网络配置。排查这类问题，要把“连上网络”“解析域名”“连接代理”“建立TLS”“下载正确内容”分开看。

本文以Ubuntu 22.04为背景。示例用于定位问题，不要求同时修改DNS、软件源和代理，也不会执行安装脚本。

## 一、为什么浏览器和终端表现不同？

浏览器、shell中的程序、通过`sudo`启动的APT，以及Git、pip等工具，可能分别使用自己的代理或环境配置。浏览器里的成功请求，只验证了浏览器当时采用的那条路径。

另外，某些代理可以代为解析目标域名。因此，即使系统本地DNS有问题，经过该代理的网页仍可能打开；反过来，DNS正常也不等于目标网站、代理认证或软件源没有问题。curl与APT都各自支持代理设置，配置入口并不是一个通用开关。[Ubuntu 22.04：curl手册](https://manpages.ubuntu.com/manpages/jammy/man1/curl.1.html)、[APT的HTTP代理配置](https://manpages.ubuntu.com/manpages/jammy/man1/apt-transport-http.1.html)

## 二、第一步应该检查什么？

先确认复制进去的是干净命令。如果混进`<br/>`、Markdown链接或代码围栏，shell可能还没有发出任何网络请求就报错。第4篇已经介绍这一类问题；网络排查要从命令确实被执行开始。

然后检查链路、IP地址和默认路由：

```bash
nmcli device status
ip -br address
ip route
```

第一条查看NetworkManager管理的网络设备状态，第二条简要显示各接口地址，第三条查看IPv4路由。关注实际使用的Wi-Fi或有线接口是否连接、是否取得地址、是否存在合理的默认出口。不要把任意一个虚拟接口的“UP”当作互联网已经连通。[NetworkManager：nmcli手册](https://networkmanager.dev/docs/api/latest/nmcli.html)、[Ubuntu 22.04：ip手册](https://manpages.ubuntu.com/manpages/jammy/man8/ip.8.html)

这些命令只查看状态。如果最基础的连接就失败，应先在桌面网络设置里恢复正常连接；暂时没有理由编辑APT源。IPv6或VPN环境可能有额外路由，`ip route`这一条也不是完整网络审计。

## 三、“无法解析域名”究竟要查哪一层？

我的实际报错指向`security.ubuntu.com`的解析失败。可从同一个域名开始检查，而不是不断换测试网站：

```bash
resolvectl status
resolvectl query security.ubuntu.com
```

`status`查看当前解析器及各接口DNS信息；`query`尝试查询目标域名。它们适用于系统使用systemd-resolved的常见Ubuntu桌面配置。若该服务未运行，要先确认当前由谁管理解析，不要把工具报错直接当成域名不存在。[Ubuntu 22.04：resolvectl／resolvconf共用手册](https://manpages.ubuntu.com/manpages/jammy/man1/resolvconf.1.html)

能返回地址，说明这次查询成功；仍不能证明HTTPS请求一定成功。查询失败时，结合刚才的连接状态、DNS服务器和网络环境判断。校园网登录、VPN、分流设置都可能需要分别核对。

不建议一上来就覆盖`/etc/resolv.conf`，更不要把它锁死后当永久方案。先弄清配置由桌面网络管理器、系统解析器还是其他软件维护；如果最终确实要调整DNS，在对应连接里记录原设置、只改一处，并保留恢复到原值或自动获取的路径。

## 四、怎样验证终端是否能用上已有代理？

先在代理客户端中确认本地监听地址、端口和协议。界面上出现一个端口，不代表它一定是HTTP代理；HTTP、SOCKS以及混合端口要按客户端实际设置使用。

下面用本机`127.0.0.1:7890`作**示例HTTP代理**，不是默认可用值，也不是需要照抄的真实环境参数。请替换为自己已经确认的地址和端口：

```bash
local_proxy='http://127.0.0.1:7890'
curl --proxy "$local_proxy" --noproxy '' --connect-timeout 10 --max-time 30 -I https://ubuntu.com/
```

`--proxy`只为这次curl指定代理，`--noproxy ''`排除已有代理绕过列表对本次测试的影响；两个时间参数避免请求无限等待；`-I`请求响应头，不下载网页正文。代理连接失败时，先核对客户端是否运行、监听协议和端口是否一致。

看到代理返回的`200 Connection established`，还不能只凭这一行宣布目标网站请求成功；继续看后续TLS和目标服务器响应。即使目标返回HTTP状态，也要区分成功、重定向、权限错误等情况。部分服务器对HEAD和实际下载请求处理不同，最终还要验收原任务。[Ubuntu 22.04：curl选项说明](https://manpages.ubuntu.com/manpages/jammy/man1/curl.1.html)

测试结束后可以清理示例变量：

```bash
unset local_proxy
```

这里没有导出环境变量，也没有写入启动文件。测试通过后，再决定哪些应用确实需要持久配置。

## 五、为什么curl通了，sudo apt仍然失败？

curl的单次参数只影响curl。普通用户shell中的配置，也不能想当然地当作管理员进程已经继承的配置。与其盲目把所有环境变量透传给`sudo`，可以先用APT自己的选项验证一次。

仍以已经核实的HTTP代理为前提，以下命令只在这次APT调用中指定代理：

```bash
local_proxy='http://127.0.0.1:7890'
sudo apt -o Acquire::http::Proxy="$local_proxy" -o Acquire::https::Proxy="$local_proxy" update
unset local_proxy
```

`apt update`会刷新本机的软件包索引，不安装或升级软件包。两个选项分别为HTTP和HTTPS软件源指定代理；HTTPS目标通过HTTP代理连接时，代理地址仍可使用`http://`，具体协议要与监听端一致。如果以前另设了针对某个源的代理或直连规则，还需核对该源的专门配置。[APT命令说明](https://manpages.ubuntu.com/manpages/jammy/man8/apt.8.html)、[APT的HTTP传输手册](https://manpages.ubuntu.com/manpages/jammy/man1/apt-transport-http.1.html)、[APT的HTTPS传输手册](https://manpages.ubuntu.com/manpages/jammy/man1/apt-transport-https.1.html)

不要只看到最后几行“完成”就忽略前面的失败源。若提示某些索引无法下载、继续使用旧索引，包列表可能仍不完整。应逐条核对原先失败的源是否已经成功更新，再重新查询所需软件包。

如果代理包含认证信息，不要把用户名密码直接写进公开命令、截图或聊天记录。本文只演示不带认证的本地代理，不展开保存凭据的配置。

## 六、DNS和代理都正常，为什么还会失败？

继续按错误所在层排查：

| 现象 | 下一项检查 | 不宜直接做什么 |
|---|---|---|
| 连接被拒绝或超时 | 端口、目标地址、路由、服务状态 | 不要先重装证书包 |
| 证书时间或信任错误 | 系统时间、错误中的域名、证书链 | 不要用关闭TLS校验来“修复” |
| HTTP 401／403 | 账户、授权、请求路径和服务规则 | 不代表DNS必然有问题 |
| APT签名或发行文件错误 | 软件源地址、发行版代号、签名配置 | 不要允许未认证软件包来绕过 |
| 某个包没有候选 | 索引是否更新、包名和版本、启用的仓库 | 不要批量添加不明第三方源 |

查看时间可用`timedatectl status`。如果需要纠正时间，先检查桌面的日期时间和自动同步设置；单纯检查本身不改变系统。APT还会校验仓库的认证信息，网络连通和软件来源验证是两件事。[Ubuntu 22.04：timedatectl手册](https://manpages.ubuntu.com/manpages/jammy/man1/timedatectl.1.html)、[APT安全认证说明](https://manpages.ubuntu.com/manpages/jammy/man8/apt-secure.8.html)

## 七、怎样证明“这次问题解决了”？

把成功限定到实际任务。例如，原问题是安全更新源无法解析，就重新执行对应APT更新并检查该源的结果；原问题是下载文件，就检查下载退出状态、文件大小，以及发布方提供的校验值或签名。一次网页请求不能代替所有工具的验证。

我的历史记录能够支持“显式代理后有下载成功，后来多个工具也完成下载”，但不能支持“从此所有网络问题都消失”。新开终端、重启代理客户端、重新登录后，原配置可能有不同表现；需要持久使用时再补一次相同任务的复验。

请Kimi协助时，可以使用这样的提问模板，而不是只说“帮我修网络”：

> 浏览器可用，但终端访问这个域名失败。这是原命令、完整错误，以及已去除私人信息的连接、DNS和代理测试结果。请指出目前已经证实哪一层可用、下一条最小检查是什么；暂时不要修改全局DNS或软件源。

## Linux命令小板块

| 命令 | 观察层次 |
|---|---|
| `nmcli device status` | NetworkManager设备连接状态 |
| `ip -br address`、`ip route` | 地址和IPv4路由 |
| `resolvectl query security.ubuntu.com` | 一次系统解析器查询 |
| `timedatectl status` | 系统时间与同步状态 |

排错记录最好写成“哪一个程序、访问哪个目标、用了什么配置、返回什么结果”。下一次换了网络，也能从已验证的那一层继续。

---

[系列目录](../README.md) · [上一篇：第 04 篇](04-desktop-input-appimage.md) · [下一篇：第 06 篇](06-kimi-linux-workflow.md)
