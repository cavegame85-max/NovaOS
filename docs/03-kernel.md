# Шаг 3: Ядро Linux и systemd

## Что такое ядро Linux

Ядро — это **сердце операционной системы**. Оно:

- Работает с процессором (распределяет время между программами).
- Управляет памятью (кто что занимает).
- Работает с дисками, USB, сетью, звуком.
- Запускает программы.

**Ядро ≠ операционная система.** Это **часть** ОС. Кроме ядра, нужны:

- **init** (systemd) — запускает всё при загрузке.
- **userspace** — программы, утилиты, библиотеки.
- **драйверы** — модули для устройств.

## Что такое systemd

systemd — это **система инициализации**. Она:

- Запускается ядром **первой** (PID 1).
- Запускает все остальные службы: сеть, звук, USB, экран входа.
- Следит за ними и перезапускает при сбое.
- Управляет системой: shutdown, reboot, suspend.

**Альтернативы:** OpenRC, runit, SysVinit. Для NovaOS выбрана **systemd** — потому что она стандарт Debian и даёт много документации.

## Установка ядра (в chroot)

Войди в chroot:

    sudo mount --bind /dev /mnt/novaos/dev
    sudo mount --bind /dev/pts /mnt/novaos/dev/pts
    sudo mount --bind /proc /mnt/novaos/proc
    sudo mount --bind /sys /mnt/novaos/sys
    sudo mount --bind /run /mnt/novaos/run
    sudo cp /etc/resolv.conf /mnt/novaos/etc/resolv.conf
    sudo chroot /mnt/novaos /bin/bash
    export TERM=xterm

Проверь, что ты внутри:

    hostname
    cat /etc/debian_version

Должно быть `novaos` (или `USERNAME`-хост) и `12.x`.

Установи ядро:

    apt update
    apt install -y linux-image-amd64 linux-headers-amd64 initramfs-tools

**Разбор:**
- `linux-image-amd64` — метапакет ядра (тянет последнее доступное).
- `linux-headers-amd64` — заголовки ядра (нужны для компиляции модулей).
- `initramfs-tools` — утилиты создания initramfs (начальный образ загрузки).

**Размер:** ~150 МБ. **Время:** 3–5 минут.

**Что произойдёт:**
- Скачаются пакеты ядра.
- Распакуются в `/boot/`.
- Автоматически соберётся `initrd.img-<версия>-amd64`.

## Проверка ядра

    ls -la /boot/

Должны быть:
- `vmlinuz-<версия>-amd64` — само ядро.
- `initrd.img-<версия>-amd64` — initramfs.
- `config-<версия>-amd64` — конфиг ядра.
- `System.map-<версия>-amd64` — карта символов.

Символические ссылки:

    ls -la /vmlinuz /initrd.img

Должны указывать на файлы в `/boot/`.

## Установка systemd

    apt install -y systemd systemd-sysv

**Разбор:**
- `systemd` — сама система инициализации.
- `systemd-sysv` — символические ссылки для совместимости.

**Размер:** ~10 МБ. **Время:** 1–2 минуты.

**Предупреждение:** при установке возможно сообщение «systemd не может быть запущен в chroot» — это **нормально**. systemd запустится при **первой настоящей загрузке**.

**Проверка:**

    dpkg -l | grep systemd | head -10

Должны быть: `systemd`, `systemd-sysv`, `libsystemd0`, `libsystemd-shared`.

## Установка сетевых утилит и базовых пакетов

    apt install -y iproute2 iputils-ping net-tools dnsutils kmod udev \
      less tree htop man-db

**Разбор:**
- `iproute2` — команда `ip` (современная замена `ifconfig`).
- `iputils-ping` — команда `ping`.
- `net-tools` — классические `ifconfig`, `route`.
- `dnsutils` — `dig`, `nslookup`.
- `kmod` — модули ядра (`lsmod`, `modprobe`).
- `udev` — работа с устройствами.
- `less`, `tree`, `htop` — утилиты.
- `man-db` — система справки.

**Размер:** ~20 МБ.

## Обновление системы

    apt upgrade -y

**Что произойдёт:** обновятся пакеты, выпущенные Debian после релиза. Ядро может обновиться — тогда появится **второе** ядро. Это нормально.

**Размер:** 50–100 МБ. **Время:** 3–5 минут.

**Проверка:**

    apt list --upgradable

Должно быть **пусто** (`Listing... Done`).

## Проверка ядра внутри NovaOS

    uname -a

Покажет **текущее ядро Mint** (потому что мы в chroot, общее ядро). **Это нормально.**

После **первой настоящей загрузки** `uname -a` покажет **своё ядро**.

## Выход

    exit

**Размонтировать** (по одной команде):

    sudo umount /mnt/novaos/run
    sudo umount /mnt/novaos/sys
    sudo umount /mnt/novaos/proc
    sudo umount /mnt/novaos/dev/pts
    sudo umount /mnt/novaos/dev

**Проверка:**

    mount | grep novaos

Должно быть пусто.

## Что получилось

- Установлено **ядро Linux** (`vmlinuz-*`).
- Создан **initramfs** (`initrd.img-*`).
- Установлен **systemd**.
- Сеть, утилиты, `man`, `htop`.
- Система **готова к следующему этапу** — установке прошивок и графики.

## Следующий шаг

→ `04-network-firmware.md` — сеть, прошивки, драйверы.
