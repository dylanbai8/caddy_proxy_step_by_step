## 使用方法：按照空行分出的段落，在Xshell依次执行以下命令即可。

## 0.前期准备

此教程使用 caddy 直接搭建 https(h2) 代理。（不适用于无80端口的 nat vps）

```
由于使用到了底层ssl加密传输 需要有域名配合才能搭建使用

你可以使用本站提供的临时域名：
如服务器IP为 103.1.14.203 临时域名即 103-1-14-203.ip.c2ray.ml

也可以使用自有域名如：
解析域名 “youdiangan.ga” A记录到 “103.1.14.203”
或者解析域名 “youdiangan.ga” AAAA记录到 “2607:2200:0:2347:0:3353:e1:1a2b”

(AAAA记录可以配合cloudflare转换为ip4，适用于无80端口的 nat vps)
```

## 1.VPS安装Debian8 更新系统 安装必要软件

```
apt update -y

apt install curl -y
```

## 2.执行官方脚本 安装caddy

```

curl https://getcaddy.com | bash -s personal http.forwardproxy,http.proxyprotocol

```

## 3.配置caddy

添加caddy配置文件：/usr/local/bin/ [Caddyfile]

配置https(h2)代理的用户名、密码

```
touch /usr/local/bin/Caddyfile

cat <<EOF > /usr/local/bin/Caddyfile
103-1-14-203.ip.c2ray.ml:443 {
tls admin@103-1-14-203.ip.c2ray.ml
root /www
gzip
index index.html
forwardproxy {
        basicauth username password
}
}
EOF
```

## 4.设置caddy开机自启动

添加开机自启动文件：/etc/systemd/system/ [caddy.service]

```
touch /etc/systemd/system/caddy.service

cat <<EOF > /etc/systemd/system/caddy.service
[Unit]
Description=Caddy_Server
After=network.target
Wants=network.target
[Service]
Type=simple
ExecStart=/usr/local/bin/caddy -conf=/usr/local/bin/Caddyfile -agree=true -ca=https://acme-v02.api.letsencrypt.org/directory
RestartPreventExitStatus=23
Restart=always
User=root
[Install]
WantedBy=multi-user.target
EOF

systemctl enable caddy
```

## 5.添加一个骚气的伪装网站

```
rm -rf /www && mkdir /www

wget -c -r -np -k -L -p https://www.stenabulk.com

mv ./*stenabulk*/* /www
```

详细教程，参照：

https://github.com/dylanbai8/c2ray_step_by_step/blob/master/附录-加一个骚气的伪装网站.md


## 重启caddy 载入配置文件

```
systemctl daemon-reload

systemctl restart caddy
```

## 7.客户端配置

```
- Chrome:     ProxySwitchyOmega 插件，代理类型选择 https, 端口填443, 再点击右侧的小锁，输入用来验证代理的用户名和密码即可。
- Firefox:    Foxyproxy 插件，配置方式大同小异。
- IOS:        SURGE 等，代理类型选择HTTPS，配置方式大同小异。
- Android:    ProxyDroid、Postern 等，配置方式大同小异。
```

## 可能用到的命令

```
关闭apache2
systemctl stop apache2
systemctl disable apache2

基于caddy的https代理可以与c2ray项目共存，有兴趣的可以自行研究一下。
https://github.com/dylanbai8/c2ray_step_by_step/blob/master/C2ray-(WebSocket+TLS+Web)-底层传输SSL加密.md
```

## 关联项目：

https://c2ray.ml

https://github.com/dylanbai8/c2ray


