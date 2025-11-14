# Cài Termux và Debian
1. Chạy trong termux
```cmd
pkg update && pkg upgrade -y
pkg install proot-distro wget tar -y
proot-distro install debian
proot-distro login debian
```
2. Cài Node.js + npm
```cmd
apt update && apt upgrade -y
apt install curl git build-essential -y

# Cách nhanh nhất (dùng Node bản stable)
apt install nodejs npm -y

# Kiểm tra
node -v
npm -v
```

3. Cài Cloudflared   
```cmd
mkdir -p /tmp
chmod 1777 /tmp
cd /tmp
wget https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-arm64.deb
apt install ./cloudflared-linux-arm64.deb -y
```

##### Cài cloudflared qua script (không dùng file .deb)
Cloudflare có installer tự động hỗ trợ mọi kiến trúc, kể cả armhf.

Chạy:
```cmd
curl -L https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-arm -o /usr/local/bin/cloudflared
chmod +x /usr/local/bin/cloudflared
```

4. Cài nvm (Node Version Manager)
- Cài đặt:
```cmd
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/master/install.sh | bash

# Nạp nvm vào terminal hiện tại
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"
```
- Cài Node.js 23 và npm 11:
```cmd
nvm install 23
nvm use 23
npm install -g npm@11

node -v   # kiểm tra Node
npm -v    # kiểm tra npm
```

#### Bật yêu cầu pass
```cmd
nano /etc/ssh/sshd_config
```
- Dán:
```cmd
PasswordAuthentication yes
PermitRootLogin yes
```


# ssh debian phone

#### setup ssh trên server:
```cmd
apt install openssh-server -y
```
- Kiểm tra đường dẫn cấu hình SSH:
```cmd
cat /etc/ssh/sshd_config | grep Port
```
- Mặc định là Port 22. Bạn có thể đổi để tránh xung đột:
```cmd
sed -i 's/#Port 22/Port 8022/' /etc/ssh/sshd_config
```
- Tạo host keys (nếu chưa có):
```cmd
ssh-keygen -A
```

### Chạy SSH Server
```cmd
service ssh start
```
#### Lỗi thì dùng:
1. Dừng service cũ:
```cmd
pkill sshd
```
2. Tạo thư mục cần thiết (proot không tự tạo /var/run/sshd):
```cmd
mkdir -p /var/run/sshd
chmod 0755 /var/run/sshd
```
3. Chạy sshd thủ công:
```cmd
/usr/sbin/sshd -p 2022
```
4. Kiểm tra lại:
```cmd
ps aux | grep sshd
```
+ và thử:
```cmd
ssh localhost -p 2022
```

#### Đổi mật khẩu
```cmd
passwd <pass>
```


### ssh
#### Với cloudflared server:
- Tạo file `start-cloudflared.sh`:
```cmd
#!/bin/bash
exec cloudflared tunnel run --token eyJhIjoiYjYyOWQwZWVmMGY5YzkyZTNjYWUzNzg5OTJkZDg0NzAiLCJ0IjoiNDk0Y2Q5OWYtNTQwMi00OTcwLTkzMmEtYjMxZDY0ZjNlZjAxIiwicyI6Ik1ETTNZakE0WmpNdFpqTTJOQzAwTW1OaUxXSmhaR0V0TlRSbE5Ea3pZMlJpWlRrNCJ9
```
- Đảm bảo executable:
```cmd
chmod +x start-cloudflared.sh
```

- Chạy lần đầu:
```cmd
ssh-keygen -R ssh.dh74.io.vn
```

#### Chạy trên client
```cmd
ssh -o 'ProxyCommand=cloudflared access ssh --hostname %h' root@ssh.dh74.io.vn
```

#### Đẩy folder lên server
```sh
tar -cf - -C <Nguồn> <Đích> | ssh -o "ProxyCommand=cloudflared access ssh --hostname %h" root@ssh.dh74.io.vn "tar -xf - -C ~/"
```
