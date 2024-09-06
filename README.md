# initial server setup after installation

### Настрока ssh соединения

Генерируем  ключ ssh

```
ssh-keygen -t ed25519
```

#### Вход на сервер

```
ssh root@ip
```

#### Создание пользователя

##### Если пользователь не существует
```
sudo adduser adrin
sudo usermod -aG sudo adrin
sudo su adrin
```
###### Удаление пользователя с домашней папкой
```
sudo deluser --remove-home usename
```
##### Создаем папку .ssh и файл authorized_keys (копируем туда public_id)

```
mkdir ~/.ssh
echo public_id > ~/.ssh/authorized_keys
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
```

##### Обновляем сервер:
```
sudo apt update && sudo apt upgrade -y
```
##### Меняем настройки в sshd_config
```
sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config.bak
sudo sed -i 's/#Port 22/Port 22/g' /etc/ssh/sshd_config
sudo sed -i 's/#PermitRootLogin prohibit-password/PermitRootLogin no/g' /etc/ssh/sshd_config
sudo sed -i 's/#MaxSessions 10/MaxSessions 2/g' /etc/ssh/sshd_config
sudo sed -i 's/#PubkeyAuthentication yes/PubkeyAuthentication yes/g' /etc/ssh/sshd_config
sudo sed -i 's/#PasswordAuthentication yes/PasswordAuthentication no/g' /etc/ssh/sshd_config
sudo sed -i 's/#PermitEmptyPasswords no/PermitEmptyPasswords no/g' /etc/ssh/sshd_config
```
##### Перезапускаем сервис sshd и проверяем статус
```
sudo systemctl restart sshd.service
sudo systemctl status sshd.service
```

#### Установка fail2ban

##### Ubuntu

```
sudo apt install fail2ban
sudo systemctl start fail2ban
sudo systemctl enable fail2ban
sudo systemctl status fail2ban
```
###################################################

##### Debian
```
sudo nano /etc/fail2ban/jail.d/00-systemd.conf
```
```
[sshd]
enabled = true
backend = systemd
```
##################################################

```
sudo nano /etc/fail2ban/jail.local
```

```
[DEFAULT]
# Set the ban time in seconds (e.g., 3600 seconds = 1 hour)
bantime = 3h
findtime = 10m
maxretry = 3
ignoreip = 127.0.0.1 192.168.0.5

bantime.increment = true
bantime.rndtime = 30m
bantime.maxtime = 60d

[sshd]
enabled = true
port = ssh
filter = sshd
logpath = /var/log/auth.log
maxretry = 3
bantime = 3h
```

```
sudo systemctl restart fail2ban
sudo systemctl status fail2ban
```
```
sudo fail2ban-client status sshd
sudo fail2ban-client set sshd unbanip IP_ADDRESS
```

```
sudo apt install iptables
#sudo systemctl start iptables
#udo systemctl enable iptables
#sudo systemctl status iptables
```

```
sudo iptables -A INPUT -i lo -j ACCEPT
sudo iptables -A OUTPUT -o lo -j ACCEPT
sudo iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
sudo iptables -I INPUT -m state --state INVALID -j DROP
sudo iptables -A INPUT -p icmp -j ACCEPT
sudo iptables -A INPUT -p tcp -m tcp --dport 22 -j ACCEPT
sudo iptables -P OUTPUT ACCEPT
sudo iptables -P FORWARD DROP
sudo iptables -P INPUT DROP
```

```
sudo apt install iptables-persistent
```
##### В процессе установки iptables-persistent сохранить правила для IPv4 и IPv6 в /etc/iptables/
```
sudo reboot
```
##### Проверяем, восстановились ли правила 
```
sudo iptables -vnL
```
