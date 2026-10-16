# Шаг 4: Сеть, прошивки и драйверы

## Что такое прошивки (firmware)

Прошивка — это **микрокод**, который загружается в устройство (Wi-Fi-адаптер, видеокарту, звуковую карту) для его работы. Без прошивки устройство **не запустится**, даже если драйвер есть.

**Различие:**
- **Драйвер** — код в ядре Linux, который **общается** с устройством.
- **Прошивка** — бинарный файл, который **загружается в само устройство**.

**Проблема:** большинство прошивок — **проприетарные**. Они не входят в стандартный Debian. Нужно подключать репозиторий **non-free-firmware**.

## Подключение non-free-firmware (в chroot)

Войди в chroot:

    sudo mount --bind /dev /mnt/novaos/dev
    sudo mount --bind /dev/pts /mnt/novaos/dev/pts
    sudo mount --bind /proc /mnt/novaos/proc
    sudo mount --bind /sys /mnt/novaos/sys
    sudo mount --bind /run /mnt/novaos/run
    sudo cp /etc/resolv.conf /mnt/novaos/etc/resolv.conf
    sudo chroot /mnt/novaos /bin/bash
    export TERM=xterm

Открой файл источников:

    nano /etc/apt/sources.list

Замени содержимое на **три строки**:

    deb http://deb.debian.org/debian bookworm main contrib non-free non-free-firmware
    deb http://deb.debian.org/debian bookworm-updates main contrib non-free non-free-firmware
    deb http://security.debian.org/debian-security bookworm-security main contrib non-free non-free-firmware

Сохрани: Ctrl+O, Enter, Ctrl+X.

Проверь:

    cat /etc/apt/sources.list

Должно быть **три строки**. Убедись, что написано `debian`, а не `debiam` (типичная опечатка).

Обнови:

    apt update

**Возможная ошибка 404** — если в URL опечатка. Проверь каждую букву.

## Установка прошивок

    apt install -y firmware-linux-nonfree firmware-atheros firmware-realtek

**Разбор:**
- `firmware-linux-nonfree` — прошивки для GPU (NVIDIA, AMD), Wi-Fi, звука.
- `firmware-atheros` — для Wi-Fi Atheros (AR9285).
- `firmware-realtek` — для Realtek-сетевух.

**Размер:** ~200 МБ. **Время:** 3–5 минут.

**Дополнительно могут установиться:**
- `intel-microcode` — микрокод для процессора Intel.
- `amd64-microcode` — микрокод для AMD.
- `firmware-amd-graphics` — для AMD GPU.
- `firmware-misc-nonfree` — разные устройства.

**Проверка:**

    dpkg -l | grep firmware

Должны быть: `firmware-linux-nonfree`, `firmware-atheros`, `firmware-realtek`, `intel-microcode`.

    ls /lib/firmware/ | head -20

Должны быть файлы прошивок (.bin, .fw, .ucode).

## Установка NetworkManager

NetworkManager управляет **Wi-Fi, Ethernet, VPN, мобильной связью**. Без него не будет интернета после загрузки.

    apt install -y network-manager network-manager-gnome

**Разбор:**
- `network-manager` — сам менеджер.
- `network-manager-gnome` — апплет для панели (иконка Wi-Fi).

**Размер:** ~50 МБ.

**Включение автозапуска:**

    systemctl enable NetworkManager

**Возможный ответ:** «Running in chroot, ignoring command 'enable'» — **это нормально**. Симлинк создастся.

**Проверка:**

    ls /etc/systemd/system/multi-user.target.wants/ | grep -i network

Должен быть `NetworkManager.service`.

## Проверка драйверов (после загрузки)

**Эти команды — уже после того, как загрузишься в NovaOS.**

Wi-Fi:

    lspci -nnk | grep -A3 -i network

Должны быть:
- **Модель**: Qualcomm Atheros AR9285 (для Samsung R528).
- **Kernel driver in use**: ath9k.
- **Kernel modules**: ath9k.

Ethernet:

    lspci -nnk | grep -A3 -i ethernet

Должны быть:
- **Модель**: Marvell 88E8040.
- **Kernel driver in use**: sky2.

Видеокарта:

    lspci -nnk | grep -A3 -i vga

Должны быть:
- **Модель**: NVIDIA GeForce 310M.
- **Kernel driver in use**: nouveau.

Звук:

    lspci -nnk | grep -A3 -i audio

Должны быть:
- **Модель**: Intel ICH9 HD Audio.
- **Kernel driver in use**: snd_hda_intel.

## Проверка сетевых интерфейсов

    ip -br a

Должны быть:
- `lo` — loopback (127.0.0.1).
- `wlp2s0` или `wlan0` — Wi-Fi.
- `enp4s0` или `eth0` — Ethernet.

**Если Wi-Fi интерфейс есть, но `state DOWN`:**

    sudo ip link set wlp2s0 up

**Подключение к Wi-Fi (после загрузки):**

    nmcli device wifi list
    nmcli device wifi connect "ИМЯ_СЕТИ" password "ПАРОЛЬ"

Или через **GUI** — иконка в панели → выбрать сеть → ввести пароль.

## Проверка интернета

    ping -c 3 ya.ru
    ping -c 3 8.8.8.8

**Если работает** — всё ок.

**Если не работает:**
- Проверь DNS: `cat /etc/resolv.conf` (должен быть `nameserver`).
- Проверь маршрут: `ip route`.
- Проверь Wi-Fi: `nmcli device status`.

## Микрокод Intel

**Что это:** обновление «внутренней прошивки» процессора Intel. Исправляет баги, уязвимости, иногда улучшает стабильность.

**Проверка после загрузки:**

    dmesg | grep -i microcode

Должна быть строка:
    microcode: updated early to revision 0x...

**Если применился** — процессор работает с исправлениями.

## Что получилось

- Подключён репозиторий **non-free-firmware**.
- Установлены **прошивки** для Wi-Fi, GPU, звука.
- Установлен **NetworkManager**.
- **Wi-Fi, Ethernet, GPU, звук** будут работать после загрузки.
- Микрокод Intel применится при первой загрузке.

## Следующий шаг

→ `05-graphics.md` — графика X.org + XFCE + LightDM.
