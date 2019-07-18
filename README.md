# Install shadow socks on Debian

Install the packages

```
sudo apt update
sudo apt install shadowsocks-libev
```

For Debian 9 or 8, package can be found at [Backports](https://backports.debian.org/)

For Debian 9:

```
sudo sh -c 'printf "deb http://deb.debian.org/debian stretch-backports main" > /etc/apt/sources.list.d/stretch-backports.list'
sudo apt update
sudo apt -t stretch-backports install shadowsocks-libev
```

For Debian 8:

```
sudo sh -c 'printf "deb http://deb.debian.org/debian jessie-backports main\n" > /etc/apt/sources.list.d/jessie-backports.list'
sudo sh -c 'printf "deb http://deb.debian.org/debian jessie-backports-sloppy main" >> /etc/apt/sources.list.d/jessie-backports.list'
sudo apt update
sudo apt -t jessie-backports-sloppy install shadowsocks-libev
```

To run 

```
ssserver -s 监听地址(通常为0.0.0.0) -p 监听端口 -k 密码 -m 加密方法 -t 超时时间（秒）
```

can combined with `nohup`

```
nohup ssserver -s 0.0.0.0 -p 443 -k a29rw4pacnj2ahmf -m aes-192-cfb -t 600 &
```

If to use obfs, follow the installation instructions from [5]. Bascially if you are not using a sid debian you have to compile it. 

You can first try:

```bash
sudo apt install simple-obfs
```

If this doesn't work, try:

```bash
sudo apt-get install --no-install-recommends build-essential autoconf libtool libssl-dev libpcre3-dev libev-dev asciidoc xmlto automake
git clone https://github.com/shadowsocks/simple-obfs.git
cd simple-obfs
git submodule update --init --recursive
./autogen.sh
./configure && make
sudo make install
```

Then you have to give it permssion by:[6]

```bash
setcap cap_net_bind_service+ep /usr/local/bin/obfs-server # your installation path
```

To run as deamon, first edit configuration file:

```
emacs /etc/shadowsocks-libev/config.json
```

sample looks like this:

```
{
    "server":["::0","0.0.0.0"],
    "server_port":443,
    "password":"your_pass_word",
    "timeout":600,
    "method":"aes-256-cfb",
    "mode":"tcp_and_udp",
    "fast_open":false,
    "plugin":"obfs-server",
    "plugin_opts":"obfs=http"
}
```

Then 

```
systemctl enable shadowsocks-libev
systemctl start shadowsocks-libev
systemctl status shadowsocks-libev
```

You can verify this by:

```bash
ps -ef | grep -v grep | grep "server"
```

You will see two process runing:

```bash
nobody   27375     1  0 08:36 ?        00:00:00 /usr/bin/ss-server -c /etc/shadowsocks-libev/config.json -u
nobody   27376 27375  0 08:36 ?        00:00:00 obfs-server
```

Also you can apply [Google BBR](https://github.com/google/bbr) mod to accelate connection speed by

```bash
wget --no-check-certificate https://github.com/teddysun/across/raw/master/bbr.sh && chmod +x bbr.sh && ./bbr.sh
```

#### Reference

1. https://wiki.archlinux.org/index.php/Shadowsocks
2. https://github.com/shadowsocks/shadowsocks-libev
3. https://github.com/shadowsocks/shadowsocks-libev/blob/master/doc/ss-server.asciidoc#options
4. https://dreamcreator108.com/dreams/debian-stretch-shadowsocks-libev/ 
5. https://github.com/shadowsocks/simple-obfs
6. https://github.com/shadowsocks/shadowsocks-libev/issues/1166