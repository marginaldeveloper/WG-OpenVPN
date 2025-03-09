Гайд по установке OpenVPN и WireGuard на сервере
=====================================================

Часть 1: Установка OpenVPN
Обновление системы
Обновите список пакетов и установите последние обновления:


sudo apt update && sudo apt upgrade -y
Добавление репозитория OpenVPN
Добавьте репозиторий для OpenVPN и ключи для подписи:


echo "deb [signed-by=/etc/apt/keyrings/openvpn-as.gpg.key] http://as-repository.openvpn.net/as/debian $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/openvpn-as.list
wget --quiet -O - https://as-repository.openvpn.net/as-repo-public.gpg | sudo tee /etc/apt/keyrings/openvpn-as.gpg.key
Установка зависимостей
Установите необходимые зависимости для OpenVPN:


sudo apt install apt-transport-https ca-certificates -y
sudo apt update
Установка OpenVPN
Установите OpenVPN Access Server:


sudo apt install -y openvpn-as
Доступ к веб-интерфейсу
После установки OpenVPN Access Server появятся две ссылки для доступа:

Админ-панель: http://your-server-ip/admin
Клиентский интерфейс: http://your-server-ip Войдите с учетной записью openvpn и паролем, который был сгенерирован во время установки.
Настройка пользователей в Admin UI
Перейдите в User Management > User Permissions.
Добавьте или измените пользователей.
Нажмите More Settings, чтобы установить пароли для пользователей.

Скачивание клиента VPN
Перейдите в Client UI.
Войдите в интерфейс и скачайте конфигурацию клиента VPN.

Часть 2: Установка WireGuard
Генерация ключей для сервера
Создайте приватный и публичный ключи для сервера:


wg genkey | tee privatekey | wg pubkey > publickey
Сохраните приватный ключ в безопасном месте!

Проверка сетевых интерфейсов
Убедитесь, что ваш интерфейс соответствует нужному:


ip link
Если ваш адаптер не eth0, замените его в конфигурации WireGuard.

Создание конфигурации WireGuard
Откройте конфигурационный файл WireGuard:


sudo nano /etc/wireguard/wg0.conf
Вставьте следующее содержимое:


[Interface]
PrivateKey = (ваш приватный ключ)
Address = 10.0.0.1/8
SaveConfig = true
PostUp = iptables -A FORWARD -i wg0 -j ACCEPT; iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
PostDown = iptables -D FORWARD -i wg0 -j ACCEPT; iptables -t nat -D POSTROUTING -o eth0 -j MASQUERADE
ListenPort = 51820

Запуск WireGuard
wg-quick start (вроде)
Активируйте интерфейс WireGuard:


wg-quick up wg0
wg-quick status wg0
Часть 3: Настройка клиента для WireGuard
Генерация ключей для клиента
Создайте ключи для клиента:


wg genkey | tee Client_privatekey | wg pubkey > Client_publickey
Просмотр приватного ключа клиента
Отобразите приватный ключ клиента:


cat Client_privatekey
Создание конфигурации клиента
На клиенте создайте файл конфигурации:


[Interface]
PrivateKey = (приватный ключ клиента)
ListenPort = 54021
Address = 10.0.0.2/8
DNS = 192.168.31.1, 192.168.31.1

[Peer]
PublicKey = (публичный ключ сервера)
AllowedIPs = 0.0.0.0/0
Endpoint = $SERVER_IP:51820
PersistentKeepalive = 30
Добавление клиента на сервер
Добавьте клиента на сервер WireGuard:



sudo wg set wg0 peer (публичный ключ клиента) allowed-ips 10.0.0.2/32

Включение переадресации IP
Включите IP-форвардинг:

sudo sysctl -w net.ipv4.ip_forward=1
Проверка соединения
Проверьте соединение между клиентом и сервером:


ping 10.0.0.2
ping 8.8.8.8
Если все настроено правильно, вы должны получить ответ от клиента.



Документация OpenVPN: https://openvpn.net/community-resources/
Документация WireGuard: https://www.wireguard.com/documentation/
Форумы и сообщества: https://www.reddit.com/r/openvpn/ и https://www.reddit.com/r/WireGuard/
