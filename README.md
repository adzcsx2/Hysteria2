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
#### 搭建伪装网站和注册证书
step1:
```

wget -N --no-check-certificate "https://raw.githubusercontent.com/adzcsx2/Trojansh/master/trojan1.sh" && chmod +x trojan1.sh && ./trojan1.sh
```

step2:
```
wget -N --no-check-certificate "https://raw.githubusercontent.com/adzcsx2/Trojansh/master/trojan2.sh" && chmod +x trojan2.sh && ./trojan2.sh
```
step3: （如果需要搭建trojan协议，不需要可以省略,trojan和hysteria同443端口不冲突）
```

wget -N --no-check-certificate "https://raw.githubusercontent.com/adzcsx2/Trojansh/master/trojan3.sh" && chmod +x trojan3.sh && ./trojan3.sh
```
## 搭建Hysteria2

#### 安装
```
bash <(curl -fsSL https://get.hy2.sh/)
```

####  服务器配置文件
```
cat << EOF > /etc/hysteria/config.yaml
listen: :443 #监听端口

#使用CA证书
acme:
  domains:
    - a.com #你的域名，需要先解析到服务器ip
  email: test@sharklasers.com

#使用自签证书
#tls:
#  cert: /etc/hysteria/server.crt
#  key: /etc/hysteria/server.key

auth:
  type: password
  password: 123456 #设置认证密码
  
masquerade:
  type: proxy
  proxy:
    url: https://bing.com #伪装网址
    rewriteHost: true
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

#### 生成自签证书(前面注册了证书可以省略)

```
openssl req -x509 -nodes -newkey ec:<(openssl ecparam -name prime256v1) -keyout /etc/hysteria/server.key -out /etc/hysteria/server.crt -subj "/CN=bing.com" -days 36500 && sudo chown hysteria /etc/hysteria/server.key && sudo chown hysteria /etc/hysteria/server.crt
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

#### 开放端口
```
#开启TCP流量端口
firewall-cmd --add-port=1-65535/tcp --permanent
#开启UDP流量端口
firewall-cmd --add-port=1-65535/udp --permanent
```

#### 开启UDP流量转发
```
firewall-cmd --add-forward-port=port=20000-50000:proto=udp:toport=443 --permanent
```

#### 重载应用配置（配置完毕后必须进行这一步）
```
firewall-cmd --reload
```


## 其他命令
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
