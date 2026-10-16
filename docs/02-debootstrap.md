# Шаг 2: Создание базы Debian через debootstrap

## Что такое debootstrap

debootstrap — утилита, которая создаёт минимальную систему Debian в указанной папке или на указанном разделе. Это не установщик, а сборщик: он скачивает .deb-пакеты и распаковывает их.

## Установка debootstrap

На текущей системе:
sudo apt update
sudo apt install -y debootstrap
debootstrap --version

## Разметка диска (если ещё не сделано)

Для Legacy BIOS + GPT:
sudo parted /dev/sdX mklabel gpt
sudo parted /dev/sdX mkpart primary 1MiB 2MiB
sudo parted /dev/sdX set 1 bios_grub on
sudo parted /dev/sdX mkpart primary fat32 2MiB 514MiB
sudo parted /dev/sdX set 2 esp on
sudo parted /dev/sdX mkpart primary ext4 514MiB 100%

Форматирование:
sudo mkfs.vfat -F32 -n NOVAEFI /dev/sdX2
sudo mkfs.ext4 -L NOVAROOT /dev/sdX3

Монтирование:
sudo mkdir -p /mnt/novaos
sudo mount /dev/sdX3 /mnt/novaos
sudo mkdir -p /mnt/novaos/boot/efi
sudo mount /dev/sdX2 /mnt/novaos/boot/efi

## Запуск debootstrap

sudo debootstrap --arch=amd64 --variant=minbase bookworm /mnt/novaos http://deb.debian.org/debian

Разбор флагов:
--arch=amd64 — 64-битная архитектура.
--variant=minbase — минимум пакетов.
bookworm — версия Debian (12).
/mnt/novaos — куда распаковывать.
http://deb.debian.org/debian — откуда брать.

Что произойдёт:
Скачается ~120 МБ пакетов.
Распакуется в /mnt/novaos.
Создастся структура Linux: /bin, /etc, /usr, /var.

Займёт 15-30 минут (зависит от скорости интернета).

Критерий успеха:
I: Base system installed successfully.

## Проверка

ls /mnt/novaos/

Должны быть папки: bin, boot, dev, etc, home, lib, lib64, media, mnt, opt, proc, root, run, sbin, srv, sys, tmp, usr, var.

sudo du -sh /mnt/novaos/
Должно быть ~264 МБ.

## Дальше — вход в chroot

Смонтировать системные папки:
sudo mount --bind /dev /mnt/novaos/dev
sudo mount --bind /dev/pts /mnt/novaos/dev/pts
sudo mount --bind /proc /mnt/novaos/proc
sudo mount --bind /sys /mnt/novaos/sys
sudo mount --bind /run /mnt/novaos/run

DNS (для интернета в chroot):
sudo cp /etc/resolv.conf /mnt/novaos/etc/resolv.conf

Вход:
sudo chroot /mnt/novaos /bin/bash
export TERM=xterm

Приглашение изменится на root@hostname:/#. Ты внутри NovaOS.

## Настройка базовой системы (в chroot)

echo "novaos" > /etc/hostname

cat > /etc/hosts <<EOF
127.0.0.1   localhost
127.0.1.1   novaos
::1         localhost ip6-localhost ip6-loopback
EOF

apt update
apt install -y sudo curl wget nano bash-completion locales tzdata apt-utils

passwd

dpkg-reconfigure tzdata
dpkg-reconfigure locales

Разбор:
Имя хоста: novaos.
Установка базовых утилит.
Пароль root.
Настройка часового пояса и локали.

## Создание пользователя

useradd -m -s /bin/bash -G sudo,audio,video,plugdev,cdrom,floppy,users USERNAME
passwd USERNAME

Проверка:
id USERNAME

## Выход и размонтирование

exit
sudo umount /mnt/novaos/run
sudo umount /mnt/novaos/sys
sudo umount /mnt/novaos/proc
sudo umount /mnt/novaos/dev/pts
sudo umount /mnt/novaos/dev

Проверка:
mount | grep novaos
Должно быть пусто.

## Что получилось

Минимальная Debian 12 — ~264 МБ.
Работает в chroot.
Есть apt, bash, sudo.
Есть пользователь USERNAME с паролем.
Настроены локаль и часовой пояс.

## Следующий шаг

03-kernel.md — установка ядра Linux и systemd.
