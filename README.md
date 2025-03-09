# 🔧 Гайд по установке OpenVPN и WireGuard на сервер.
Взял из этих видео индусов: 
OpenVPN: https://www.youtube.com/watch?v=aViCCFztoAE
WG: https://www.youtube.com/watch?v=bVKNSf1p1d0

## ✅ Часть 1: Установка OpenVPN

### 🌐 Обновление системы
Выполните обновление списка пакетов и установите последние обновления:
```bash
sudo apt update && sudo apt upgrade -y
```

### ➕ Добавление репозитория OpenVPN
Добавьте репозиторий OpenVPN и ключи для подписи:
```bash
echo "deb [signed-by=/etc/apt/keyrings/openvpn-as.gpg.key] http://as-repository.openvpn.net/as/debian $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/openvpn-as.list
wget --quiet -O - https://as-repository.openvpn.net/as-repo-public.gpg | sudo tee /etc/apt/keyrings/openvpn-as.gpg.key
```

### ⚖️ Установка зависимостей
Установите необходимые зависимости:
```bash
sudo apt install apt-transport-https ca-certificates -y
sudo apt update
```

### 🏠 Установка OpenVPN
Установите OpenVPN Access Server:
```bash
sudo apt install -y openvpn-as
```

### 🔓 Доступ к веб-интерфейсу
После установки доступны следующие интерфейсы:
- **Админ-панель**: `http://your-server-ip/admin`
- **Клиентский интерфейс**: `http://your-server-ip`

Войдите с учетной записью `openvpn` и паролем, который был сгенерирован во время установки.

### ⚙️ Настройка пользователей в Admin UI
1. Перейдите в **User Management > User Permissions**.
2. Добавьте или измените пользователей.
3. Нажмите **More Settings**, чтобы установить пароли для пользователей.

### 💾 Скачивание клиента VPN
1. Перейдите в **Client UI**.
2. Войдите в интерфейс и скачайте конфигурацию клиента VPN.

---

## ✅ Часть 2: Установка WireGuard

### 🔐 Генерация ключей для сервера
Создайте приватный и публичный ключи:
```bash
wg genkey | tee privatekey | wg pubkey > publickey
```
**Важно!** Сохраните приватный ключ в безопасном месте.

### 📊 Проверка сетевых интерфейсов
Убедитесь, что интерфейс соответствует нужному:
```bash
ip link
```
> Если ваш адаптер не `eth0`, замените его в конфигурации WireGuard.

### 🗒️ Создание конфигурации WireGuard
Откройте конфигурационный файл:
```bash
sudo nano /etc/wireguard/wg0.conf
```
Пример содержимого файла:
```ini
[Interface]
PrivateKey = <ваш приватный ключ>
Address = 10.0.0.1/8
SaveConfig = true
PostUp = iptables -A FORWARD -i wg0 -j ACCEPT; iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
PostDown = iptables -D FORWARD -i wg0 -j ACCEPT; iptables -t nat -D POSTROUTING -o eth0 -j MASQUERADE
ListenPort = 51820
```

### 🚀 Запуск WireGuard
Активируйте интерфейс WireGuard:
```bash
sudo wg-quick up wg0
sudo wg-quick status wg0
```

---

## ✅ Часть 3: Настройка клиента для WireGuard

### 🔐 Генерация ключей для клиента
Создайте ключи:
```bash
wg genkey | tee Client_privatekey | wg pubkey > Client_publickey
```

### 🔍 Просмотр приватного ключа клиента
```bash
cat Client_privatekey
```

### 🗒️ Создание конфигурации клиента
Создайте конфигурационный файл на клиенте:
```ini
[Interface]
PrivateKey = <приватный ключ клиента>
ListenPort = 54021
Address = 10.0.0.2/8
DNS = 192.168.31.1, 192.168.31.1

[Peer]
PublicKey = <публичный ключ сервера>
AllowedIPs = 0.0.0.0/0
Endpoint = <SERVER_IP>:51820
PersistentKeepalive = 30
```

### ➕ Добавление клиента на сервер
```bash
sudo wg set wg0 peer <публичный ключ клиента> allowed-ips 10.0.0.2/32
```

### 🔄 Включение переадресации IP
```bash
sudo sysctl -w net.ipv4.ip_forward=1
```

### 📝 Проверка соединения
```bash
ping 10.0.0.2
ping 8.8.8.8
```
> Если все настроено правильно, вы получите ответы от клиента.

---

## 🖊️ Полезные ресурсы
- [📖 Документация OpenVPN](https://openvpn.net/community-resources/)
- [📖 Документация WireGuard](https://www.wireguard.com/documentation/)
- [🌐 Форум OpenVPN](https://www.reddit.com/r/openvpn/)
- [🌐 Форум WireGuard](https://www.reddit.com/r/WireGuard/)

---

Теперь у вас есть полный гайд для установки и настройки OpenVPN и WireGuard!

