# Centos 7.x 搭建```shadowsocks```笔记

### 1，安装```shadowsocks```
#### 1.1，首先安装```epel```扩展源
```powershell
yum -y install epel-release
```
#### 1.2，安装完成后，安装```pip```
```powershell
yum -y install python-pip
```
#### 1.3，安装完成之后，清除```cache```
```powershell
yum clean all
```
#### 1.4，将```pip```升级到最新版本
```powershell
pip install --upgrade pip
```
#### 1.5，```pip```安装```python```版的```shadowsocks```
```powershell
pip install shadowsocks
```

### 2，```shadowsocks```的配置
#### 2.1，创建配置文件
```powershell
vim /etc/shadowsocks/shadowsocks.json
```
配置类容如下：
```json
{
    "server":"xxx",//这里是你的服务器地址[注意点1]
    "server_port":8388,//这里是你的代理端口
    "local_address": "127.0.0.1",//无需改动，
    "local_port":1080,//无需改动
    "password":"123456",//代理密码
    "timeout":300,//超时时间，无需改动
    "method":"aes-256-cfb",//加密算法，需要保证客户端的配置和此处一致
    "fast_open": false,//快速启动
    "workers": 1
}
```
[注意点1]：当使用阿里云服务器时，此处需要填分配给你的内网IP地址，填写外网地址无效，原因推测分配的外网ip地址是公用的。
#### 2.2，设置开机自启动
Centos 7.x开机自己动做了变更，此处我们新建一个```shadowsocks.service```文件来实现自启动。
为了保证服务器重启后，没有用户登录的情况下，服务也能启动，我们需要将自启动配置放到系统服务目录下。
新建配置文件如下：
```powershell
vim /lib/systems/system/shadowsocks.service
```
类容如下：
```
[Unit]
Description=shadowsocks
After=network.target

[Service]
Type=forking
TimeoutStartSec=0
ExecStart=/usr/bin/ssserver -c /etc/shadowsocks/shadowsocks.json

[Install]
WantedBy=multi-user.target
```
### 2.3，启动服务
```powershell
systemctl enable shadowsocks
systemctl start shadowsocks
```
然后查看启动是否成功
```powershell
systemctl status shadowsocks -l
```
如果显示如下日志，则表示启动成功
```prolog
shadowsocks.service - shadowsocks
   Loaded: loaded (/usr/lib/systemd/system/shadowsocks.service; enabled; vendor preset: disabled)
   Active: activating (start) since 六 2019-10-05 20:31:23 CST; 26min ago
  Control: 771 (ssserver)
   CGroup: /system.slice/shadowsocks.service
           └─771 /usr/bin/python /usr/bin/ssserver -c /etc/shadowsocks/shadowsocks.json
```
### 2.4，配置防火墙
启动shadowsocks后，需要配置防火墙，开放配置中指定的端口。否则客户端软件无法连接服务器。具体配置请参见Centos 7.x防火墙配置。
### 3.5，配置安全组
如果使用的是阿里云服务器，则好需要配置安全组，开放指定的端口。如果觉得麻烦，可以将服务器防火墙关闭，只使用阿里云的安全组配置。
