<div align="right">
  <a title="简体中文" href="README.md"><img src="https://img.shields.io/badge/-%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87-A31F34?style=for-the-badge" alt="简体中文" /></a>
  <a title="English" href="README_EN.md"><img src="https://img.shields.io/badge/-English-545759?style=for-the-badge" alt="English"></a>
</div>

# mtproxy

这是一个一键安装 MTProxy 代理的自动化脚本, 用于 Telegram 客户端进行连接。脚本默认支持 Fake TLS 和 AdTag 配置。


在此基础上，提供了 Nginx 作为前端转发，MTProxy 作为后端代理的方式以实现安全的伪装。并且在 Nginx 转发层进行配置了 IP 白名单，只有通过白名单认证过的 IP 才可以进行访问。

> 此功能提供了 Docker 镜像以便开箱即用。

## 交流群组

Telegram 群组：<https://t.me/EllerHK>

## 安装方式

提供了两种安装方式可供选择：

- 使用脚本 (建议 Debian/Ubuntu)

  选择该方式一般是你在宿主机中进行直接安装或者编译，会或多或少需要安装一些系统基础依赖库。

- 使用 Docker (任意支持docker的系统均可)

  **小白建议使用 Docker!** 不会对宿主机造成污染，如果你需要修改一些配置文件，需要你稍微学习一些基础 Docker 使用技术。

### 使用脚本

> 如果你反复遇到错误或者其他未知问题, 建议更换为 Debian 9+ 以上的系统或采用 Docker 方式运行。

执行如下代码进行安装

```bash
rm -rf /home/mtproxy && mkdir /home/mtproxy && cd /home/mtproxy
curl -fsSL -o mtproxy.sh https://github.com/ellermister/mtproxy/raw/master/mtproxy.sh
bash mtproxy.sh
```

 ![mtproxy.sh](https://raw.githubusercontent.com/ellermister/mtproxy/master/preview.jpg)

### 使用 Docker | 白名单 MTProxy Docker 镜像

该镜像集成了 nginx、mtproxy+tls 实现对流量的伪装，并采用**白名单**模式来应对防火墙的检测。

若使用该 Docker 镜像, 就不需要用脚本了，二者二选一，不要搞混了。

**如果没有安装Docker**，一键安装方式如下：

```bash
curl -fsSL https://get.docker.com -o get-docker.sh
sh get-docker.sh
```

**创建白名单镜像：**

 ```bash
docker run -d \
--name mtproxy \
--restart=always \
-e domain="cloudflare.com" \
-p 8080:80 \
-p 8443:443 \
ellermister/mtproxy
 ```

**镜像默认开启了 IP 段白名单**  
如果你不需要可以配置 `ip_white_list="OFF"` 取消：

```bash
docker run -d \
--name mtproxy \
--restart=always \
-e domain="cloudflare.com" \
-e secret="548593a9c0688f4f7d9d57377897d964" \
-e ip_white_list="OFF" \
-p 8080:80 \
-p 8443:443 \
ellermister/mtproxy
```

`ip_white_list` 选项:

- **OFF** 关闭白名单
- **IP** 开启 IP 白名单
- **IPSEG** 开启 IP 段白名单

`secret`指定密钥：如果你想创建已知的密钥，格式为：32位十六进制字符。

**在日志中查看链接的参数配置**：

```bash
docker logs -f mtproxy
```

连接端口记得修改为你映射后的外部端口，如上文例子中都是`8443`，在连接时修改端口。

更多使用请参考： <https://hub.docker.com/r/ellermister/mtproxy>

## 使用方式

配置文件 `config`，如果你想手动修改密钥或者参数请注意格式。
```bash
vim /home/mtproxy/config
```

## 开机启动

> 先创建系统服务
```bash
cat > /etc/systemd/system/mtproxy.service <<EOF
[Unit]
Description=MTProto Proxy Service
After=network.target

[Service]
Type=forking
WorkingDirectory=/home/mtproxy
ExecStart=bash mtproxy.sh start
ExecStop=bash mtproxy.sh stop
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```

>启用服务（开机自启）
```bash
systemctl daemon-reload
systemctl enable mtproxy
systemctl start mtproxy
```

## 全局命令&计划任务守护

创建mtproxy 全局命令
```bash
cat > /usr/local/bin/MTProxy <<EOF
#!/bin/bash
cd /home/mtproxy && bash mtproxy.sh "\$@"
EOF
```

>添加进程守护（防崩溃、防卡死）
```bash
(crontab -l 2>/dev/null; echo "* * * * * cd /home/mtproxy && bash mtproxy.sh start > /dev/null 2>&1 &") | crontab -
```

>命令功能
```bash
MTProxy start       # 启动
MTProxy stop        # 停止
MTProxy restart     # 重启
MTProxy debug       # 调试
systemctl status mtproxy  # 看运行状态
```


## 卸载安装
停止服务 + 关闭开机自启 + 删除守护 + 删除整个目录
```bash
MTProxy stop
systemctl stop mtproxy
systemctl disable mtproxy
rm -f /etc/systemd/system/mtproxy.service
rm -f /usr/local/bin/MTProxy
crontab -l | grep -v mtproxy | crontab -
rm -rf /home/mtproxy```



## MTProxy Admin Bot

<https://t.me/MTProxybot>
> Sorry, an error has occurred during your request. Please try again later.(Code xxxxxx)

如果你在申请绑定代理推广时遇到了此类错误，官方没有给出明确的原因。根据网友反馈，此类问题多出现于账号注册不足与 2~3 年。  
**建议使用 3 年以上的账号以及未被 banned 的账号。**

## 引用项目

- <https://github.com/TelegramMessenger/MTProxy>
- <https://github.com/9seconds/mtg>
- <https://github.com/alexbers/mtprotoproxy>
