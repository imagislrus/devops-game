# NanoPi R2S — Инструкция по развёртыванию на новое устройство

## Обзор

Данная инструкция описывает два способа развёртывания настроенной конфигурации NanoPi R2S на новое устройство.

Конфигурация включает:
- **FriendlyWrt (OpenWrt)** — операционная система роутера
- **PassWall2 + Xray** — VPN-клиент с VLESS Reality
- **Tailscale** — P2P удалённый доступ (независим от VPN)
- **Раздельная маршрутизация** — российские сайты напрямую, остальные через VPN
- **Блокировка BitTorrent** — порты и DNS-трекеры
- **3 VPN-узла с разными SNI** — Microsoft, Apple, Google

---

## Что вам понадобится

- Новое устройство NanoPi R2S
- microSD карта (16–128 ГБ, Class 10 или выше)
- USB-кардридер
- Компьютер (Windows или Mac)
- Ethernet-кабель
- Файл образа: `nanopi-backup.img.gz` (416 МБ)
- Файл конфигов: `nanopi-config-backup.tar.gz` (30 КБ)

---

## Сравнение способов

| | Способ 1: Полный образ | Способ 2: Чистая прошивка + конфиги |
|---|---|---|
| **Файл** | `nanopi-backup.img.gz` | `nanopi-config-backup.tar.gz` + свежая прошивка |
| **Сложность** | Легко | Средне |
| **Плюсы** | Полная копия, всё работает сразу | Свежая версия ОС, меньший размер файла |
| **Минусы** | Большой файл (416 МБ) | Больше шагов, нужен интернет на роутере |

---

## Способ 1: Полный образ (рекомендуется)

Этот способ записывает полную копию настроенной системы на новую SD-карту. После записи нужно только переавторизовать Tailscale.

### Windows

#### Вариант A: Rufus (с GUI)

1. Скачайте **Rufus**: https://rufus.ie
2. Вставьте microSD-карту в кардридер → подключите к ПК
3. Запустите Rufus
4. В поле **Device** выберите вашу SD-карту
5. Нажмите **SELECT** → выберите файл `nanopi-backup.img.gz`
6. Нажмите **START**
7. Дождитесь завершения записи

#### Вариант B: balenaEtcher

1. Скачайте **balenaEtcher**: https://etcher.balena.io
2. **Flash from file** → выберите `nanopi-backup.img.gz`
3. **Select target** → выберите SD-карту
4. **Flash!** → дождитесь завершения

> ⚠️ **Убедитесь, что выбрали правильный диск! Запись на системный диск уничтожит Windows.**

### macOS

1. Вставьте microSD-карту в кардридер → подключите к Mac
2. Откройте **Terminal** и определите номер диска:
```bash
diskutil list
```
Найдите SD-карту по размеру (например, `/dev/disk6`).

3. Отмонтируйте карту (замените `disk6` на ваш номер):
```bash
diskutil unmountDisk /dev/disk6
```

4. Запишите образ:
```bash
gunzip -c ~/nanopi-backup.img.gz | sudo dd of=/dev/rdisk6 bs=4m
```
Введите пароль Mac когда попросит. Терминал будет молчать во время записи — это нормально.

5. После завершения извлеките карту:
```bash
diskutil eject /dev/disk6
```

> ⚠️ **Mac не сможет прочитать записанную карту — это нормально! Не форматируйте её.**

### После записи образа (общее для Windows и macOS)

1. Вставьте microSD в новый NanoPi R2S (слот снизу платы)
2. Подключите Ethernet-кабель от **WAN-порта** к основному роутеру/модему
3. Подключите питание USB-C
4. Подождите **2–3 минуты** до полной загрузки

5. **Переавторизуйте Tailscale.** Подключитесь по SSH через LAN:
```bash
ssh root@192.168.2.1
```
Затем:
```bash
tailscale up --accept-routes --advertise-exit-node
```
Откройте ссылку авторизации на телефоне и подтвердите устройство.

6. Узнайте новый Tailscale IP:
```bash
tailscale ip
```
Запишите адрес вида `100.x.x.x`.

7. Проверьте работу VPN:
```bash
curl --max-time 10 https://ifconfig.me
```
Должен вернуться IP: `72.62.40.98`

8. Проверьте Tailscale:
```bash
tailscale status
```

**Готово!** Все настройки уже на месте.

---

## Способ 2: Чистая прошивка + восстановление конфигов

### Шаг 1: Записать свежую прошивку

Скачайте свежую прошивку FriendlyWrt:
- https://www.friendlyelec.com → Downloads → NanoPi R2S
- Файл вида: `rk3328-sd-friendlywrt-XX.XX-XXXXXXXX.img.gz`

**Windows:** используйте Rufus или balenaEtcher (см. Способ 1).

**macOS:**
```bash
diskutil list
diskutil unmountDisk /dev/diskN
gunzip -c ~/Downloads/rk3328-sd-friendlywrt-*.img.gz | sudo dd of=/dev/rdiskN bs=4m
diskutil eject /dev/diskN
```

### Шаг 2: Первый запуск

1. Вставьте SD-карту в NanoPi R2S
2. Подключите LAN-порт к компьютеру, WAN-порт к основному роутеру
3. Подключите питание, подождите 2–3 минуты
4. Откройте http://192.168.2.1 (логин: `root`, пароль: `password`)
5. Смените пароль: **System → Administration**

### Шаг 3: Установить Tailscale

```bash
ssh root@192.168.2.1
```

```bash
opkg update
opkg install tailscale
/etc/init.d/tailscale start
/etc/init.d/tailscale enable
tailscale up --accept-routes --advertise-exit-node
```
Откройте ссылку авторизации и подтвердите устройство.

### Шаг 4: Загрузить бэкап конфигов на роутер

**Windows:**
1. Скачайте **WinSCP**: https://winscp.net
2. Подключитесь к `192.168.2.1` (или Tailscale IP), логин `root`
3. Перетащите файл `nanopi-config-backup.tar.gz` в папку `/tmp/`

**macOS:**
```bash
scp ~/nanopi-config-backup.tar.gz root@192.168.2.1:/tmp/
```

### Шаг 5: Восстановить конфигурацию

```bash
sysupgrade -r /tmp/nanopi-config-backup.tar.gz
reboot
```

### Шаг 6: Установить недостающие пакеты

После перезагрузки подключитесь снова:
```bash
ssh root@100.x.x.x
```

Добавьте репозиторий PassWall:
```bash
wget -O /tmp/passwall.pub https://master.dl.sourceforge.net/project/openwrt-passwall-build/passwall.pub
opkg-key add /tmp/passwall.pub

read release arch << EOF
$(. /etc/openwrt_release ; echo ${DISTRIB_RELEASE%.*} $DISTRIB_ARCH)
EOF

for feed in passwall_packages passwall2; do
  echo "src/gz $feed https://master.dl.sourceforge.net/project/openwrt-passwall-build/releases/packages-$release/$arch/$feed" >> /etc/opkg/customfeeds.conf
done
```

Установите пакеты:
```bash
opkg update
opkg install xray-core
opkg install luci-app-passwall2
opkg install kmod-nft-socket kmod-nft-tproxy kmod-nft-nat
```

Скачайте российские GeoIP/GeoSite файлы:
```bash
wget -O /usr/share/v2ray/geosite.dat https://github.com/runetfreedom/russia-v2ray-rules-dat/releases/latest/download/geosite.dat
wget -O /usr/share/v2ray/geoip.dat https://github.com/runetfreedom/russia-v2ray-rules-dat/releases/latest/download/geoip.dat
```

Перезагрузите:
```bash
reboot
```

После перезагрузки все настройки PassWall2 должны восстановиться из бэкапа.

---

## Справочная информация

### Доступ к устройству

| Действие | Команда / Адрес |
|----------|----------------|
| Веб-интерфейс (локально) | `http://192.168.2.1` |
| Веб-интерфейс (удалённо) | `http://100.x.x.x` (Tailscale IP) |
| SSH (локально) | `ssh root@192.168.2.1` |
| SSH (удалённо) | `ssh root@100.x.x.x` |
| Управление VPN | LuCI → Services → PassWall2 |
| Перезапуск VPN | `/etc/init.d/passwall2 restart` |
| Статус Tailscale | `tailscale status` |
| Проверка VPN IP | `curl --max-time 10 https://ifconfig.me` |
| Подключённые устройства | `cat /tmp/dhcp.leases` |
| Перезагрузка | `reboot` |

### Архитектура сети

```
Устройство (телефон/ПК + Tailscale)
    │
    ├── P2P (Tailscale) ──→ NanoPi R2S ──→ SSH / LuCI
    │                         │
    │                         ├── PassWall2 (VLESS Reality)
    │                         │       │
    │                         │       ▼
    │                         │   3X-UI сервер → Интернет
    │                         │
    │                         └── Tailscale (независим от VPN)
    │
    └── Даже если VPN упал — доступ через Tailscale сохраняется
```

### Раздельная маршрутизация

- **Российские сайты** → напрямую (без VPN)
- **Все остальные сайты** → через VPN
- **Tailscale** → всегда напрямую (в обход VPN)

### VPN-узлы (3 SNI)

| Узел | SNI | Порт | Используется для |
|------|-----|------|-----------------|
| MyVPNServer | `www.microsoft.com` | 443 | OpenAI, ProxyGame, По умолчанию |
| SNI Apple | `www.apple.com` | 8443 | Netflix, Proxy |
| SNI Google | `www.google.com` | 2443 | GooglePlay, GFW, Russia_Block |

---

## Возможные проблемы и решения

**NanoPi R2S не загружается:**
Перезапишите SD-карту. Попробуйте другую microSD-карту.

**Mac не может прочитать SD-карту после записи:**
Это нормально — Mac не читает файловую систему Linux (ext4). Не форматируйте карту!

**Tailscale не подключается:**
Выполните `tailscale up` для повторной авторизации. Убедитесь, что Tailscale не идёт через VPN.

**VPN не работает (curl показывает не тот IP):**
Проверьте логи: `cat /tmp/log/passwall2.log`. Перезапустите: `/etc/init.d/passwall2 restart`.

**VLESS Reality не подключается:**
Перепроверьте параметры (UUID, Public Key, SNI, Short ID). Убедитесь, что на сервере включён Reality. Проверьте логи: LuCI → Services → PassWall2 → Log.

**Tailscale работает через VPN и падает вместе с ним:**
Добавьте домены Tailscale в Direct List в PassWall2 → Rule Manage → Direct:
```
login.tailscale.com
controlplane.tailscale.com
derp1.tailscale.com
derp2.tailscale.com
tailscale.com
```
