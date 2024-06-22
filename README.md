# mtg-dist
Oneline distbute and install script for mtg ( https://github.com/9seconds/mtg ).

## Note

mtg 2.0+ supports FakeTLS mode only.
Re-run this script to upgrade.

# Install
```
bash <(wget -qO- https://git.io/mtg.sh)
```

## Offline Install

For machines have difficulties visiting github, download the following files:

- `mtg-dist.bin` from [release](https://github.com/cutelua/mtg-dist/releases)
- `mtg-2.x.x-linux-amd64.tar.gz` from [mtg/release](https://github.com/9seconds/mtg/releases)

then run: 
```
sh mtg-dist.bin mtg-2.0.1-linux-amd64.tar.gz
```

# Uninstall

```
systemctl disable mtg 
systemctl stop mtg 
rm -f /usr/local/bin/mtg /etc/systemd/system/mtg.service /etc/mtg.toml   
```

## for version 0.0.6 and prior
```
systemctl disable mtg 
systemctl stop mtg 
rm -f /usr/local/bin/mtg /lib/systemd/system/mtg.service /etc/mtg.conf    
```

# Customize

The configure file is at `/etc/mtg.toml`, refer to the offcial document for whatever you want to custom.

- https://github.com/9seconds/mtg 
- https://github.com/9seconds/mtg/blob/master/example.config.toml

After edited just restart the mtg service:

```
systemctl restart mtg 
```

# Compile
This project uses `makeself` to generate a `.bin` file.

```
./makebin.sh
```


该镜像集成了nginx、mtproxy+tls 实现对流量的伪装，并采用白名单模式来应对防火墙的检测。
一：你必须先安装 docker，如果你没有请参考 https://docs.docker.com/engine/install/binaries/（或者直接输入以下命令安装：Centos7.6）⁠

直接输入命令：
```
systemctl stop postfix && systemctl enable postfix && yum install -y docker && systemctl start docker && systemctl enable docker
```
二：安装xxd工具。 xxd命令是用于将二进制数据转换为十六进制表示的工具。

如果您使用的是类Unix操作系统（如Linux或 macOS），可以尝试通过以下命令安装xxd:

在Debian/Ubuntu上运行：sudo apt-get install xxd 在CentOS/Fedora上运行：sudo yum install vim-common 在macOS上运行：brew install vim 如果您使用的是Windows系统，请确保已正确安装并配置了 Git Bash 或类似的终端模拟器，以便访问常用的命令行工具。

安装成功后，您应该能够在命令行中执行xxd命令，从而获得镜像的ID和其他有用的信息。

三：拉取镜像：
```
docker pull ellermister/nginx-mtproxy:latest
```
四：可通过 -p 指定端口映射，连接均为外部端口。（把端口映射为443）
```
docker run --name nginx-mtproxy -d -p 80:80 -p 443:443 180121/nginx-mtproxy:v1
```
然后它会出来一个摘要：也就是镜像ID，一串字符（例如：c1876e2ee632cf3525f7b88cd5a7fda84c56256c39278d9f5b617dd3639a）

查询镜像ID：
```
docker ps -a
```
第一个参数就是镜像ID

4a92deedc64c

五：停止这个镜像ID进程：（从stop后面开始的这串字符就是镜像ID）
```
/usr/bin/docker-current stop 4a92deedc64c
```
（例如：前面3e9843539a2742f99b04d313316fa9cba3a66927255d7f2d64a7a2c0a6a26d）

六：删除这个进程：（从rm后面开始的这串字符就是镜像ID）
```
/usr/bin/docker-current rm 4a92deedc64c
```
（例如：89f2135ce3f3aa546a844a39816f9f6e8c7ae0b8c25c6d69c2b0b600aaa636后面）

七：创建指定的 secret（打印镜像ID）、tag（设置版本号）、 domain（设置伪装域名，比如百度）：
```
secret=$(head -c 16 /dev/urandom | xxd -ps) tag="随便取一个，最好是数字" domain="伪装的域名不需要https://或者http://⁠" docker run --name nginx-mtproxy -d -e tag="$tag" -e secret="$secret" -e domain="$domain" -p 80:80 -p 443:443 180121/nginx-mtproxy:v1
```
八：创建完毕后，查看访问链接：
```
docker logs nginx-mtproxy
```
注意：请注意修改端口为你的 docker 映射的端口。（一般都是443）**

把访问链接里面的8443端口改为你映射的端口就可以了，比如443（然后先放着，不急，看下面）

该镜像采用白名单模式，来应对爬虫和防火墙探测。默认所有访客都不被允许连接，只有当访客尝试访问了下面的地址，才会将访客IP加入到白名单中（建议保留，因为这样不会被检测）：

访问白名单链接：

http://你机器的ip/add.php⁠

镜像默认开启了 IP 段白名单，如果你不需要可以不用白名单模式，直接用下面的命令启动就可以了：
```
docker run --name nginx-mtproxy -d -e secret="$secret" -e domain="$domain" -e ip_white_list="OFF" -p 8080:80 -p 8443:443 180121/nginx-mtproxy:v1 ip_white_list 的参数可以修改为以下几种方式：
```
IP：允许单个 IP 访问

IPSEG： 允许 IP 段访问

OFF：允许所有 IP 访问

MEGA: 允许IPV6 访问 例如：
```
VOW-3eR8MgLqt26iF1SMSQ
```
命令集：

停止服务
```
docker stop nginx-mtproxy
```
启动服务
```
docker start nginx-mtproxy
```
重启服务
```
docker restart nginx-mtproxy
```
删除服务
```
docker rm nginx-mtproxy
```
开机自启
```
docker update --restart=always nginx-mtproxy
```
