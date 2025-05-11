# 🚀 Xray Gateway Installer

**Xray Gateway Installer** — это автоматизированный установщик клиента [Xray-core](https://github.com/XTLS/Xray-core) с полной маршрутизацией всего исходящего трафика через прокси на основе `iptables` и `TProxy`.

Создан для **простого развёртывания** на Debian 12+ в роли шлюза, с максимальной автоматизацией и надёжной обработкой сетевых сценариев, включая исключения, fallback-режим и защиту от сбоев сети.

---

## 🧠 Для чего нужен?

Проект предназначен для:

* организации **прозрачного туннелирования** всего трафика через V2Ray/Xray,
* использования шлюза в **локальной сети** или на **виртуальной машине** (например, Proxmox),
* автоматического применения правил с изоляцией от ручной настройки сети и `iptables`.

---

## ✨ Основные возможности

* 📦 Установка `Xray-core` в `/opt/xray`, запуск от пользователя `xray:xray`.
* ⚙️ Автоматическая генерация systemd unit-файлов (`xray.service`, `xray-iptables.service`).
* 🌐 TProxy-маршрутизация для UDP, REDIRECT для TCP, с поддержкой `fwmark`, `ip rule`, `ip route`.
* 🔥 Полная маршрутизация трафика через Xray:

  * с автоматическим определением интерфейсов,
  * с исключениями по IP, CIDR, портам,
  * с автозащитой локальных подсетей.
* 🛡️ Fallback-цепочка `XRAY_DISABLED` (DROP), если Xray отключён.
* 🔧 Обновление GRUB, `sysctl`, включение IP Forward, модули ядра `xt_TPROXY`, `nf_tproxy_core`.
* 👤 Создание административного пользователя с SSH-ключом (опционально).
* 🧾 Цветной лог, проверка ошибок, диагностика шагов.
* 🧠 Генерация дампа текущих интерфейсов на случай потери сети.

---

## ⚡ Быстрый старт

> **Скрипт написан и протестирован в следующем окружении:**
>
> * **ОС:** Debian 12 (bookworm)
> * **Права:** root-доступ обязателен
> * **Сеть:** Один физический или виртуальный IPv4-интерфейс (`eth0`, `ens18`, и т.д.)
> * **IPv6 не используется:** только IPv4-сеть
> * **Тестовая среда:** виртуальная машина под управлением **Proxmox VE**

<details>
<summary>Конфигурация тестовой виртуальной машины</summary>

Тестирование проводилось в окружении, максимально приближённом к минимальному реальному шлюзу. Конфигурация:

* **CPU:** Intel(R) N97
* **BIOS:** QEMU pc-i440fx-9.2, эмуляция CPU @ 2.0GHz, 2 виртуальных ядра
* **RAM:** 1.9 GiB (в минимальной конфигурации Proxmox KVM)
* **Диск:** 20 GiB (`sda`, тип qcow2)
* **Сетевой интерфейс:** `eth0`, тип: **virtio\_net**, подключён к мосту Proxmox (`vmbr1`)
* **Ядро:** `6.1.0-34-amd64` (стандартное для Debian 12)
* **Консоль:** виртуальная (`/dev/pts/1`), запуск скрипта через `bash`
* **Proxmox VE:** используется версия 8.4.1, виртуализация KVM

</details>

---

### 📥 Установка

```bash
git clone https://github.com/Torotin/xray-gateway-installer.git
cd xray-gateway-installer
chmod +x ./install.sh
sudo ./install.sh
```

> 💡 В процессе скрипт автоматически:
>
> * создаст пользователя `xray`;
> * установит бинарник Xray в `/opt/xray`;
> * создаст конфиги и логи;
> * применит системные настройки (`GRUB`, `sysctl`);
> * активирует маршрутизацию через `iptables` и `TProxy`;
> * установит все необходимые зависимости.

## 🧪 Проверка работы

После установки проверь, что службы работают:

```bash
systemctl status xray
systemctl status xray-iptables
```

### 🛠 Конфиги

Все json-конфигурации Xray находятся в:

```bash
/opt/xray/configs/*.json
```

После редактирования конфигов — обязательно перезапусти службу:

```bash
sudo systemctl restart xray
```


## 🛑 Исключения из маршрутизации

Файлы для исключений:

* `xray-exclude-iptables.cidrs` — подсети
* `xray-exclude-iptables.ips` — IP-адреса
* `xray-exclude-iptables.ports` — порты

После изменения:

```bash
sudo /opt/xray/iptables/xray-iptables.sh restart
```

---

## ❗️ Изменение имени интерфейса после перезагрузки

При изменении параметров `GRUB` (например, отключении `predictable interface names`), после перезагрузки интерфейс может смениться (`ens18` → `eth0` и т.п.).

Скрипт заранее сохраняет список интерфейсов в `network-ifaces.dump`.

### Восстановление сети:

1. Открыть консоль Proxmox/VM.
2. Найти актуальное имя интерфейса (`ip a`).
3. Обновить `/etc/network/interfaces`.
4. Применить: `systemctl restart networking`.

---

## 📁 Структура проекта

```text
xray-gateway-installer/
├── install.sh                       # Главный установочный скрипт
├── lib/                             # Модули (по этапам установки)
│   ├── 01_common.sh                 # Общие функции (права, окружение)
│   ├── 02_network.sh                # Обнаружение интерфейсов, дамп сети
│   ├── 03_grub.sh                   # Обновление параметров загрузки
│   ├── 04_admin_user.sh             # Создание администратора и SSH-ключа
│   ├── 05_sysctl.sh                 # Настройка sysctl и ip_forward
│   ├── 06_xray_core.sh              # Установка Xray-core и systemd unit
│   └── 07_xray_iptables.sh          # Настройка iptables, TProxy, systemd
├── network-ifaces.dump              # Дамп текущих сетевых интерфейсов перед изменениями
└── template/
    ├── xray-iptables.template.sh    # Шаблон скрипта xray-iptables
    └── xray-dat-update.template.sh  # Шаблон скрипта xray-dat-update
```

## 🌍 Обновление GeoIP/GeoSite баз

Скрипт поддерживает **автоматическое обновление** баз:

* [`geoip.dat`](https://github.com/v2fly/geoip) — IP-диапазоны по странам
* [`geosite.dat`](https://github.com/v2fly/domain-list-community) — доменные группы (Google, Ads, Telegram и др.)

Также поддерживаются альтернативные источники (AntiFilter, zkeen и др.).

### 🛠 Автоматическая настройка

При установке создаётся и настраивается скрипт:

```bash
/opt/xray/tools/xray-dat-update.sh
```

Он:

* проверяет наличие обновлений по `ETag` и `SHA256`,
* скачивает и сохраняет `.dat`-файлы,
* валидирует и перезапускает службу `xray` при необходимости.

### 🔁 Добавление в `cron`

Для автоматического запуска используется `cron`. При первом запуске предлагается выбрать:

```bash
sudo /opt/xray/tools/xray-dat-update.sh -ci
```

Скрипт предложит:

* день недели,
* ежедневный режим,
* отключение расписания.

Можно изменить расписание позже через:

```bash
crontab -e
```

Пример строки в `cron` (ежедневно в 04:00):

```cron
0 4 * * * /opt/xray/tools/xray-dat-update.sh >> /opt/xray/logs/xray-dat-update.log 2>&1
```

### 📦 Источники поддерживаемых баз

| Название                 | Тип     | Источник                                                                                         |
| ------------------------ | ------- | ------------------------------------------------------------------------------------------------ |
| `geoip_v2fly.dat`        | geoip   | [https://github.com/v2fly/geoip](https://github.com/v2fly/geoip)                                 |
| `geosite_v2fly.dat`      | geosite | [https://github.com/v2fly/domain-list-community](https://github.com/v2fly/domain-list-community) |
| `geoip_antifilter.dat`   | geoip   | [https://github.com/Skrill0/AntiFilter-IP](https://github.com/Skrill0/AntiFilter-IP)             |
| `geosite_antifilter.dat` | geosite | [https://github.com/Skrill0/AntiFilter-Domains](https://github.com/Skrill0/AntiFilter-Domains)   |
| `geoip_zkeen.dat`        | geoip   | [https://github.com/jameszeroX/zkeen-ip](https://github.com/jameszeroX/zkeen-ip)                 |
| `geosite_zkeengeo.dat`   | geosite | [https://github.com/jameszeroX/zkeengeo](https://github.com/jameszeroX/zkeengeo)                 |


## 📖 FAQ

### ❓ Скрипт завис на определении интерфейса, что делать?

Проверь, что у системы **есть активный IPv4-интерфейс**. Если присутствует только IPv6 — автоматическое определение может завершиться неудачей.

```bash
ip -o -4 addr show scope global
```

Также проверь конфигурацию сети:

```bash
cat /etc/network/interfaces
```

Пример корректной настройки:

```ini
# The loopback network interface
auto lo
iface lo inet loopback

# The primary network interface
auto eth0
iface eth0 inet dhcp
```

Убедись, что:

* интерфейс указан верно (`auto eth0`, `iface eth0 inet dhcp`);
* интерфейс поднят (`ip link show eth0` → state UP);
* в `/etc/network/interfaces` нет конфликтующих записей.

Если интерфейс не определяется автоматически, можно временно задать его вручную перед запуском установки:

```bash
export LAN_IF="eth0"
```

Затем перезапусти `install.sh`.

---

### ❓ Как проверить, что весь трафик действительно проходит через Xray?

1. Убедитесь, что `xray-iptables` и `xray` активны:

   ```bash
   systemctl status xray xray-iptables
   ```

2. Проверьте логи Xray (например, DNS-запросы или outbound):

   ```bash
   journalctl -u xray -e
   ```

3. Используйте сайт [https://ipleak.net](https://ipleak.net) с устройства, чей трафик маршрутизируется через шлюз.

---

### ❓ Как полностью отключить маршрутизацию через Xray?

Просто остановите службу:

```bash
sudo systemctl stop xray
```

Трафик начнёт **дропаться**, если включена fallback-цепочка `XRAY_DISABLED`.

Если хотите разрешить трафик в обход Xray, временно отключите `xray-iptables`:

```bash
sudo systemctl stop xray-iptables
```

---

### ❓ Как вручную обновить базы `geoip.dat` и `geosite.dat`?

```bash
sudo /opt/xray/tools/xray-dat-update.sh
```

Если базы не изменились — они не будут перезаписаны.

---

### ❓ Как отключить автоматическое обновление GeoIP/GeoSite?

Открой `cron`:

```bash
crontab -e
```

И удалите строку, начинающуюся с `/opt/xray/tools/xray-dat-update.sh`.


## 🧬 Лицензия и поддержка

* 🔓 Лицензия: [GNU GPLv3](https://www.gnu.org/licenses/gpl-3.0.html)
* 🛠 Используется **на свой страх и риск**
* 📬 Вопросы и предложения — через [Issues на GitHub](https://github.com/Torotin/xray-gateway-installer/issues)

---

