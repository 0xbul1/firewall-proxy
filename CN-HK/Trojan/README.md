# 準備工作
你需要擁有一個自己的**功能變數名稱**，並**已經將功能變數名稱解析至你的伺服器**    
# 配置環境
硬體 : 記憶體 ≧ 512M 儲存 ≧ 5G | 64位系統         

軟體 : Debian 9/10 && Ubuntu 16/18/20
# 開始部署
- 安裝基本工具
```bash
apt update && apt -y install socat wget vim     
cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
```
- 安裝腳本
```bash
wget -qO- get.acme.sh | bash 
source ~/.bashrc
```
- 申請證書 （請將 **yourdomain.com** 改為你的功能變數名稱）
```bash
acme.sh --issue --standalone -d yourdomain.com -k ec-256
mkdir /etc/trojan
acme.sh --installcert -d yourdomain.com --fullchain-file /etc/trojan/server.pem --key-file /etc/trojan/server.key --ecc
```
- 安裝 Docker && Nginx && Trojan
```bash
wget -qO- get.docker.com | bash
docker pull nginx
docker pull teddysun/trojan
docker pull containrrr/watchtower
```
- 修改 Trojan 配置
```bash
vim /etc/trojan/config.json
```
- 將以下內容粘貼 
```bash
{
    "run_type": "server",
    "local_addr": "0.0.0.0",
    "local_port": 443,
    "remote_addr": "127.0.0.1",
    "remote_port": 80,
    "password": [
        "password0"   #改為你的密碼
    ],
    "log_level": 1,
    "ssl": {
        "cert": "/etc/trojan/server.pem",
        "key": "/etc/trojan/server.key",
        "key_password": "",
        "cipher": "ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384",
        "cipher_tls13": "TLS_AES_128_GCM_SHA256:TLS_CHACHA20_POLY1305_SHA256:TLS_AES_256_GCM_SHA384",
        "prefer_server_cipher": true,
        "alpn": [
            "http/1.1"
        ],
        "reuse_session": true,
        "session_ticket": false,
        "session_timeout": 600,
        "plain_http_response": "",
        "curves": "",
        "dhparam": ""
    },
    "tcp": {
        "prefer_ipv4": false,
        "no_delay": true,
        "keep_alive": true,
        "reuse_port": false,
        "fast_open": false,
        "fast_open_qlen": 20
    },
    "mysql": {
        "enabled": false,
        "server_addr": "127.0.0.1",
        "server_port": 3306,
        "database": "trojan",
        "username": "trojan",
        "password": ""
    }
}
```
- 修改 Nginx 配置
```bash
mkdir /etc/nginx && mkdir /etc/nginx/conf.d
vim /etc/nginx/conf.d/default.conf
```
- 將以下內容粘貼
```bash
server {
    listen 127.0.0.1:80 default_server;
    server_name yourdomain.com;    #修改為你的功能變數名稱
    location / {
        proxy_pass proxy.com;         #修改為你想偽裝的網站功能變數名稱，例如 https://unsplash.com/
        proxy_redirect     off;
        proxy_buffer_size          64k; 
        proxy_buffers              32 32k; 
        proxy_busy_buffers_size    128k; 
    }

}
server {
    listen 127.0.0.1:80;
    server_name ip.ip.ip.ip;      #修改為你伺服器的 IP地址
    return 301 https://yourdomain.com$request_uri;   #修改為你的功能變數名稱
}
server {
    listen 0.0.0.0:80;
    listen [::]:80;
    server_name _;
    return 301 https://$host$request_uri;
}
```
- 啟動服務
```bash
docker run --network host --name nginx -v /etc/nginx/conf.d:/etc/nginx/conf.d --restart=always -d nginx
docker run --network host --name trojan -v /etc/trojan:/etc/trojan --restart=always -d teddysun/trojan
docker run --name watchtower -v /var/run/docker.sock:/var/run/docker.sock --restart unless-stopped -d containrrr/watchtower --cleanup
```
- 啟動BBR加速
```bash
sudo bash -c 'echo "net.core.default_qdisc=fq" >> /etc/sysctl.conf'
sudo bash -c 'echo "net.ipv4.tcp_congestion_control=bbr" >> /etc/sysctl.conf'
sudo sysctl -p
```
# 更新軟體
使用這種配置方式後，**watchtower**會自動監測並更新軟體，你無需手動更新

# 客戶端
安卓系統 ：[點擊下載](https://github.com/trojan-gfw/igniter/releases)          
> 配置如下： **地址**填你的功能變數名稱，**端口**填 443 ，**密碼**填你剛才設置的密碼，其他選項無需更改        

Windows && Linux && MacOS : [Qv2ray 下載](https://github.com/Qv2ray/Qv2ray/releases)          

支持 Trojan 的插件 : [插件下載](https://github.com/Qv2ray/QvPlugin-Trojan/releases)      

插件的用法 : [文檔](https://qv2ray.net/plugins/usage.html) 