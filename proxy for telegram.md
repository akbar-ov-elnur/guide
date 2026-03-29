# Guide по созданию прокси для telegram с использованием MTProto #

1. Обновление системы:
```bash
apt update && apt upgrade -y
```

2. Включить файрвол и открыть порт:
```bash
ufw --force enable
```
```bash
ufw allow 22/tcp
```
```bash
ufw allow 443/tcp
```

3. Установить зависимости:
```bash
apt install -y python3 python3-pip git
```

4. Скачать прокси:

```bash
git clone https://github.com/alexbers/mtprotoproxy.git
```
```bash
cd mtprotoproxy
```

5. Сгенерировать секрет и записать конфиг:

```bash
SECRET=$(openssl rand -hex 16)
cat > /root/mtprotoproxy/config.py << EOF
PORT = 443
USERS = {
    "tg": "dd$SECRET",
}
MODES = {
    "classic": False,
    "secure": False,
    "tls": True
}
TLS_DOMAIN = "www.cloudflare.com"
EOF
```

6. Создать systemd сервис:

```bash
cat > /etc/systemd/system/mtproto.service << EOF
[Unit]
Description=MTProto Proxy
After=network.target

[Service]
WorkingDirectory=/root/mtprotoproxy
ExecStart=/usr/bin/python3 /root/mtprotoproxy/mtprotoproxy.py
Restart=always

[Install]
WantedBy=multi-user.target
EOF
```

7. Запустить:

```bash
systemctl enable --now mtproto
```

8. Получить ссылку для подключения:

```bash
sleep 3 && journalctl -u mtproto | grep "tg://" | tail -1
```

На восьмом шаге получится ссылка типа: 

```bash
tg://proxy?server=<IP_SERVER>&port=<PORT>&secret=<SECRET>
```
Если открыть эту ссылку на телефоне или ПК, то Telegram автоматически предложт подключиться к прокси.
