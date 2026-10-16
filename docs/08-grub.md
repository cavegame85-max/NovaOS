# Шаг 8: GRUB — загрузчик системы

## Что такое GRUB

GRUB (GRand Unified Bootloader) — это **загрузчик**. Он:

- Запускается **BIOS/UEFI** при включении.
- Показывает **меню** с выбором операционной системы.
- Загружает **ядро Linux** и передаёт ему управление.
- Поддерживает **несколько ОС** на одном диске.

**Типы GRUB:**
- **i386-pc** — для Legacy BIOS.
- **x86_64-efi** — для UEFI.

**Как узнать свой тип:**

    [ -d /sys/firmware/efi ] && echo "UEFI" || echo "BIOS"

## Установка GRUB (в chroot)

Войди в chroot:

    sudo mount --bind /dev /mnt/novaos/dev
    sudo mount --bind /dev/pts /mnt/novaos/dev/pts
    sudo mount --bind /proc /mnt/novaos/proc
    sudo mount --bind /sys /mnt/novaos/sys
    sudo mount --bind /run /mnt/novaos/run
    sudo cp /etc/resolv.conf /mnt/novaos/etc/resolv.conf
    sudo chroot /mnt/novaos /bin/bash
    export TERM=xterm

Установи пакеты GRUB:

    apt install -y grub-common grub-pc-bin grub-efi-amd64-bin os-prober

**Разбор:**
- `grub-common` — общие файлы.
- `grub-pc-bin` — для Legacy BIOS.
- `grub-efi-amd64-bin` — для UEFI.
- `os-prober` — находит другие ОС (Windows, другие Linux).

**Размер:** ~30 МБ.

**⚠️ При установке могут спросить** «GRUB install devices» — **не выбирай ничего**, нажми Esc или Enter.

## Создание BIOS boot раздела (для Legacy BIOS + GPT)

**Если ты на BIOS + GPT** — нужен раздел **1 МБ** без файловой системы, с флагом **bios_grub**. Он создаётся **до** корневого раздела.

**Проверка (с хоста, не в chroot):**

    sudo parted /dev/sdX print

**Если раздела `bios_grub` нет** — создай:

    sudo sgdisk -n 3:34:2047 -t 3:ef02 -c 3:"BIOS boot" /dev/sdX
    sudo partprobe /dev/sdX
    sudo udevadm settle

**Разбор:**
- `-n 3:34:2047` — новый раздел № 3, с сектора 34 по 2047.
- `-t 3:ef02` — тип BIOS boot.
- `-c 3:"BIOS boot"` — имя.

## Установка GRUB для Legacy BIOS

**Из chroot:**

    grub-install --target=i386-pc --boot-directory=/boot --recheck /dev/sdX

**Разбор:**
- `--target=i386-pc` — Legacy BIOS.
- `--boot-directory=/boot` — куда положить файлы GRUB.
- `/dev/sdX` — **весь диск** (GRUB пишет свой MBR-загрузчик в первые 512 байт).

**Ожидаемый вывод:**

    Installing for i386-pc platform.
    Installation finished. No error reported.

**Проверка:**

    ls -la /boot/grub/

Должны быть: `i386-pc/`, `fonts/`, `locale/`, `grubenv`.

## Установка GRUB для UEFI

**Из chroot (нужен смонтированный EFI):**

    apt install -y efibootmgr
    grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=NovaOS --recheck

**Разбор:**
- `--target=x86_64-efi` — UEFI.
- `--efi-directory=/boot/efi` — где EFI-раздел.
- `--bootloader-id=NovaOS` — имя загрузчика.

**Проверка:**

    ls -la /boot/efi/EFI/NovaOS/

Должен быть `grubx64.efi`.

## Настройка `/etc/default/grub`

Открой:

    nano /etc/default/grub

Впиши:

    GRUB_DEFAULT=0
    GRUB_TIMEOUT=5
    GRUB_DISTRIBUTOR=NovaOS
    GRUB_CMDLINE_LINUX_DEFAULT="quiet rootdelay=10"
    GRUB_CMDLINE_LINUX=""
    GRUB_TERMINAL=console
    GRUB_DISABLE_OS_PROBER=false

**Разбор:**
- `GRUB_DEFAULT=0` — грузить первую запись.
- `GRUB_TIMEOUT=5` — 5 секунд на выбор.
- `GRUB_DISTRIBUTOR=NovaOS` — имя в меню.
- `GRUB_CMDLINE_LINUX_DEFAULT="quiet rootdelay=10"` — тихая загрузка + задержка 10 секунд для медленных USB/HDD.
- `GRUB_TERMINAL=console` — текстовый режим.
- `GRUB_DISABLE_OS_PROBER=false` — **включить** поиск других ОС.

Сохрани: Ctrl+O, Enter, Ctrl+X.

## Правка `/etc/fstab`

**Открой `/etc/fstab`:**

    nano /etc/fstab

Замени на свои UUID:

    # <file system>                    <mount point>   <type>   <options>           <dump> <pass>
    UUID=REPLACE-WITH-YOUR-ROOT-UUID   /               ext4     errors=remount-ro   0      1
    UUID=REPLACE-WITH-YOUR-EFI-UUID    /boot/efi       vfat     umask=0077          0      1

**Где взять UUID:**

    blkid /dev/sdX1 /dev/sdX3

Скопируй UUID и вставь в файл.

**Важно:** если UUID неверные — система **не загрузится** (kernel panic).

## Генерация `grub.cfg`

    update-grub

**Ожидаемый вывод:**

    Generating grub configuration file ...
    Found linux image: /boot/vmlinuz-6.1.0-53-amd64
    Found initrd image: /boot/initrd.img-6.1.0-53-amd64
    Found linux image: /boot/vmlinuz-6.1.0-50-amd64
    Found initrd image: /boot/initrd.img-6.1.0-50-amd64
    Found Linux Mint 21.3 Virginia (21.3) on /dev/sdY3
    done

**Проверка:**

    grep "menuentry" /boot/grub/grub.cfg | head -10

Должны быть записи:
- `NovaOS GNU/Linux` (по умолчанию).
- `Advanced options for NovaOS`.
- `Linux Mint 21.3` (если установлена).

    grep "root=UUID" /boot/grub/grub.cfg | head -5

Убедись, что UUID **совпадает** с тем, что в `/etc/fstab`.

    grep "rootdelay" /boot/grub/grub.cfg | head -3

Должно быть `rootdelay=10`.

## Красивое меню GRUB

**Опционально** — можно оформить меню.

Установи тему:

    apt install -y grub2-theme-vimix

Или скачай тему вручную:

    cd /boot/grub/themes
    git clone https://github.com/vinceliuice/grub2-themes.git

**Настрой в `/etc/default/grub`:**

    GRUB_THEME="/boot/grub/themes/grub2-themes/theme.txt"
    GRUB_BACKGROUND="/boot/grub/themes/grub2-themes/background.png"

**Обнови:**

    update-grub

**Что увидишь при загрузке:**
- Красивое меню с иконками.
- Обои за меню.
- Стрелки → выбор ОС.
- Enter → загрузка.

## Выход и размонтирование

    exit

По одной:

    sudo umount /mnt/novaos/run
    sudo umount /mnt/novaos/sys
    sudo umount /mnt/novaos/proc
    sudo umount /mnt/novaos/dev/pts
    sudo umount /mnt/novaos/dev

    sudo umount /mnt/novaos/boot/efi
    sudo umount /mnt/novaos

**Проверка:**

    mount | grep novaos

Должно быть пусто.

## Sync и первая загрузка

**Синхронизация:**

    sync

**Подожди 5 секунд.**

**Выключи:**

    sudo poweroff

**Включи.** Настрой BIOS: **F10** → выбрать **USB/HDD** с NovaOS.

**Что увидишь:**

1. **GRUB menu:**
       NovaOS GNU/Linux
       Advanced options for NovaOS GNU/Linux
       Linux Mint 21.3 (on /dev/sdX)

2. Выбираешь **NovaOS** → Enter.

3. Загрузка ядра:
       Loading Linux 6.1.0-53-amd64 ...
       Loading initial ramdisk ...

4. **Пауза 10 секунд** — rootdelay.

5. **systemd** запускает сервисы.

6. **LightDM** — экран входа.

7. Логин + пароль → **XFCE рабочий стол**.

## Возможные проблемы

**Kernel panic: VFS: Unable to mount root fs**
- Неверный UUID в `/etc/fstab`.
- Решение: загрузиться с live-USB, поправить fstab.

**GRUB rescue>**
- GRUB не нашёл `/boot/grub/`.
- Решение: загрузиться с live-USB, `grub-install` заново.

**Чёрный экран после GRUB**
- Не запустился X.org.
- Решение: `Ctrl+Alt+F1` → консоль → проверить логи.

**Mint пропал из меню**
- `os-prober` не нашёл его.
- Решение: `sudo os-prober` → `sudo update-grub`.

## Что получилось

- Установлен **GRUB** — загрузчик.
- Настроен **fstab** с UUID.
- Сгенерирован **grub.cfg** с меню.
- **NovaOS** появляется в меню загрузки.
- **Mint** остаётся доступным.

## Следующий шаг

→ `09-troubleshooting.md` — типичные проблемы.
