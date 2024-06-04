# Hysteria2 服务端搭建

## 准备工作

#### 新机器
```
sudo apt update
sudo apt upgrade
apt install socat 
```

#### BBRplus加速(需要运行2次，一次下载后重启，下载过程中，选项全部选右边的。重启后再次运行选中协议)
```
wget -N --no-check-certificate "https://raw.githubusercontent.com/chiakge/Linux-NetSpeed/master/tcp.sh" && chmod +x tcp.sh && ./tcp.sh
```
#### 搭建伪装网站和注册证书，以及trojan协议
step1:
```

wget -N --no-check-certificate "https://raw.githubusercontent.com/adzcsx2/Trojansh/master/trojan1.sh" && chmod +x trojan1.sh && ./trojan1.sh
```

step2:
```
wget -N --no-check-certificate "https://raw.githubusercontent.com/adzcsx2/Trojansh/master/trojan2.sh" && chmod +x trojan2.sh && ./trojan2.sh
```
step3: （如果需要搭建trojan协议，不需要可以省略，trojan和hysteria由于一个是tcp一个是udp，所以同443端口不冲突）
```

wget -N --no-check-certificate "https://raw.githubusercontent.com/adzcsx2/Trojansh/master/trojan3.sh" && chmod +x trojan3.sh && ./trojan3.sh
```
## 搭建Hysteria2

#### 安装
```
bash <(curl -fsSL https://get.hy2.sh/)
```

### 方式一：自签名
#### 生成自签证书(前面注册了证书可以省略)

```
openssl req -x509 -nodes -newkey ec:<(openssl ecparam -name prime256v1) -keyout /etc/hysteria/server.key -out /etc/hysteria/server.crt -subj "/CN=bing.com" -days 36500 && sudo chown hysteria /etc/hysteria/server.key && sudo chown hysteria /etc/hysteria/server.crt
```

####  服务器配置文件
```
cat << EOF > /etc/hysteria/config.yaml
listen: :443 #监听端口

#使用CA证书
#acme:
#  domains:
#    - a.com #你的域名，需要先解析到服务器ip
#  email: test@sharklasers.com

#使用自签证书
tls:
  cert: /etc/hysteria/server.crt
  key: /etc/hysteria/server.key

auth:
  type: password
  password: 123456 #设置认证密码
  
masquerade:
  type: proxy
  proxy:
    url: https://my-domain.com #伪装网址
    rewriteHost: true
quic:
  initStreamReceiveWindow: 8388608 
  maxStreamReceiveWindow: 8388608 
  initConnReceiveWindow: 20971520 
  maxConnReceiveWindow: 20971520 
  maxIdleTimeout: 30s 
  keepAlivePeriod: 10s 
  disablePathMTUDiscovery: false 
EOF
```

### 方式二：使用服务器证书 (不知道为啥经常出现无法使用的情况)
####  服务器配置文件
```
cat << EOF > /etc/hysteria/config.yaml
listen: :443 #监听端口

#使用CA证书
#acme:
#  domains:
#    - a.com #你的域名，需要先解析到服务器ip
#  email: test@sharklasers.com

#使用自签证书
tls:
  cert: /etc/hysteria/server.crt
  key: /etc/hysteria/server.key

auth:
  type: password
  password: 123456 #设置认证密码
  
masquerade:
  type: proxy
  proxy:
    url: https://bing.com #伪装网址
    rewriteHost: true
obfs:
  type: salamander
  salamander:
    password: 1231231 
quic:
  initStreamReceiveWindow: 8388608 
  maxStreamReceiveWindow: 8388608 
  initConnReceiveWindow: 20971520 
  maxConnReceiveWindow: 20971520 
  maxIdleTimeout: 30s 
  keepAlivePeriod: 10s 
  disablePathMTUDiscovery: false
EOF
```


#### 设置开机自启
```
systemctl enable hysteria-server.service
```

#### 重启以生效
```
systemctl restart hysteria-server.service
```
#### 查看是否开启成功
```
systemctl status hysteria-server.service
```



## 设置端口跳跃

#### 安装filewalld
```
sudo apt-get install firewalld
```
#### 开机启动
```
systemctl enable firewalld
```

#### 开启TCP流量端口
```
firewall-cmd --add-port=1-65535/tcp --permanent
```
#### 开启UDP流量端口
```
firewall-cmd --add-port=1-65535/udp --permanent
```

#### 开启TCP流量转发 （20000到50000端口转443）
```
firewall-cmd --add-forward-port=port=20000-50000:proto=tcp:toport=443 --permanent
```
#### 开启UDP流量转发 （20000到50000端口转443）
```
firewall-cmd --add-forward-port=port=20000-50000:proto=udp:toport=443 --permanent
```

#### 重载应用配置（配置完毕后必须进行这一步）
```
firewall-cmd --reload
```


## 其他命令
### Hysteria2
```
#启动Hysteria2
systemctl start hysteria-server.service
#重启Hysteria2
systemctl restart hysteria-server.service
#查看Hysteria2状态
systemctl status hysteria-server.service
#停止Hysteria2
systemctl stop hysteria-server.service
#设置开机自启
systemctl enable hysteria-server.service
```
### firewalld
```
#启动服务
systemctl start firewalld
#关闭服务
systemctl stop firewalld
#重启服务
systemctl restart firewalld
#查看服务状态
systemctl status firewalld
#开机自启服务
systemctl enable firewalld
#开机禁用服务
systemctl disable firewalld
#查看是否开机自启
systemctl is-enable firewalld
#开启UDP流量转发 （20000到50000端口转443）
firewall-cmd --add-forward-port=port=20000-50000:proto=udp:toport=443 --permanent
#删除UDP流量转发 （20000到50000端口转443）
firewall-cmd --remove-forward-port=port=20000-50000:proto=udp:toport=443 --permanent
#重载应用配置
firewall-cmd --reload
```
### trojan
```
#启动
systemctl start trojan
#重启
systemctl restart trojan
#关闭
systemctl stop trojan
#状态查询（如果有显示绿色active(running)，就表示正常运行中）
systemctl status trojan
#错误查询
journalctl -e -u trojan.service
#开机自动启动
systemctl enable trojan
#禁止开机自动启动
systemctl disable trojan
```


### 配置文件路径
#### trojan:
```
/usr/local/etc/trojan/config.json
```
#### hysteria2
```
/etc/hysteria
```

### 客户端配置文件
```
server: 你的域名:20000-50000 
auth: 123456

bandwidth:
  up: 30 mbps
  down: 100 mbps
  
tls:
  sni: cn.bing.com
  insecure: true #使用自签时需要改成true
transport:
  udp:
    hopInterval: 30s 
socks5:
  listen: 127.0.0.1:1080
http:
  listen: 127.0.0.1:8080
obfs:
  type: salamander
  salamander:
    password: 1231231 
quic:
  initStreamReceiveWindow: 8388608 
  maxStreamReceiveWindow: 8388608 
  initConnReceiveWindow: 20971520 
  maxConnReceiveWindow: 20971520 
  maxIdleTimeout: 30s 
  keepAlivePeriod: 10s 
  disablePathMTUDiscovery: false 
lazy: true
```
