# 在互联网访问本地开发环境，利用 NGINX + SSH Tunnel 把本地开发环境公布到互联网上

在调试一些接口的时候，你的网站应用要跟其它的一些外部服务进行交流，但是如果是在本地开发环境上调试，你只能发送给外部服务数据，而外部服务返回来的数据你的本地环境是收不到的。因为你的本地环境并没有一个固定的公网 IP 地址。

有时候在服务器上去调试，复杂而且不方便，使用 SSH 的 Tunnel 功能，用它作为本地电脑的代理用，在公网上的一台服务器接待请求，再把请求转到本地开发环境上

用 SSH 在本地电脑与公网服务器之间打开一个通道，配置公网服务器的 NGINX，把收到的请求转到本地电脑与公网服务器的这个通道上。

## 需求

- 一台连接到公网的服务器。
- 公网服务器上安装了 NGINX。

## 配置

先在公网服务器上添加一个 NGINX 配置：

```bash
upstream tunnel {
  server 127.0.0.1:7689;
}

server {
  listen 80;
  server_name dev.example.com;
  
  location / {
    proxy_set_header  X-Real-IP  $remote_addr;
    proxy_set_header  X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header Host $http_host;
    proxy_redirect off;
    
    proxy_pass http://tunnel;
  }
}
```

上面用 NGINX 创建了一个代理，如果有人访问 dev.example.com 会把请求转给 tunnel，这个 tunnel 指的就是这台公网服务器，端口号是 7689，一会儿我们要用到这个端口跟本地电脑进行通信。

## 通道

我们要在本地电脑与公网服务器之间，使用 SSH 打开一个通道。要执行的命令像这样：

```bash
ssh -vnNT -R 服务器端口:localhost:本地端口 服务器用户名@服务器 IP 地址
```

示例

```bash
ssh -vnNT -R 7689:localhost:3000 root@42.120.40.68
```

在上面这个例子里，7689 指的是公网服务器的端口，localhost 后面的 3000 是本地电脑用的端口。root 是登录到公网服务器的用户，42.120.40.68 是公网服务器的 IP 地址。

因为我们配置了公网服务器的 NGINX，访问 dev.example.net ，把请求转到服务器上的 7689 端口，这个端口跟我们的本地电脑上的 3000 端口是连接到一块儿的。所以，你在本地开发环境上搭建的服务器，应该使用 3000 这个端口提供服务。也就是，当有人访问 dev.ninghao.net 这个地址的时候，用户得到的响应是你的本地开发环境上的服务器提供的。
