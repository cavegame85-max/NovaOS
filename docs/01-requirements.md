# Требования и подготовка

## Аппаратные требования

### Минимальные (проверено на Core 2 Duo T9300)

| Компонент | Значение |
|---|---|
| CPU | 64-битный, 2 ядра, x86_64 |
| RAM | 2 ГБ (при 4+ лучше) |
| Диск | 10 ГБ свободного места |
| GPU | NVIDIA (nouveau), Intel (i915), AMD (radeon) |
| Wi-Fi | Atheros, Intel, Broadcom (с firmware) |
| Ethernet | любая |
| BIOS | Legacy BIOS ИЛИ UEFI |

### Проверка на текущей системе

**CPU:**
lscpu | grep -E "Model name|Architecture|CPUss"
Должно быть x86_64.

**RAM:**
free -h
Минимум 2 ГБ.

**Диск:**
lsblk
df -h /
Минимум 10 ГБ свободно.

**GPU:**
lspci -nnk | grep -A3 VGA
Модель и текущий драйвер.

**Wi-Fi:**
lspci -nnk | grep -A3 -i network
Модель и драйвер (ath9k, iwlwifi, brcmfmac и т.д.).

**BIOS или UEFI:**
[ -d /sys/firmware/efi ] && echo "UEFI" || echo "BIOS"
Запомни результат — от этого зависит способ установки GRUB.

## Программные требования

На основной системе (Linux Mint/Ubuntu/Debian для сборки):
- Любой работающий Linux с интернетом.
- Терминал.
- Права sudo.
- ~15 ГБ свободного места.
- ~2 часа времени.

Пакеты, которые понадобятся:
sudo apt install -y debootstrap binutils gcc make
parted gdisk rsync wget curl nano git

## Что нужно решить до начала

1. **Куда ставим NovaOS?**
   - Отдельная флешка (минимум 8 ГБ, рекомендуется 32+).
   - Отдельный раздел на HDD.
   - Отдельный диск.

2. **Разметка диска.**

   Для Legacy BIOS + GPT:
   - /dev/sdX1 — BIOS boot (1 МБ)
   - /dev/sdX2 — EFI (512 МБ, vfat)
   - /dev/sdX3 — корень (весь остаток, ext4)

   Для UEFI (без BIOS boot):
   - /dev/sdX1 — EFI (512 МБ, vfat)
   - /dev/sdX2 — корень (весь остаток, ext4)

3. **UUID раздела.**

   После разметки получить:
sudo blkid /dev/sdX1 /dev/sdX2 /dev/sdX3
UUID понадобятся для fstab и GRUB.

## Резервное копирование

Обязательно до начала:
- Документы, фото, видео — на внешний носитель.
- Список пакетов: dpkg --get-selections > ~/installed.txt.
- Конфиги: cp -r ~/.config ~/.bashrc ~/.profile ~/backup/.
- Ключи SSH/GPG: cp -r ~/.ssh ~/.gnupg ~/backup/.

Правило: если данные есть только на одном носителе — это не бэкап.

## Время на сборку

| Фаза | Время |
|---|---|
| Разметка диска | 5 минут |
| debootstrap | 20-30 минут |
| Установка ядра, systemd | 15 минут |
| Установка прошивок | 10 минут |
| Установка X.org + XFCE | 30-40 минут |
| Установка Wine | 20-30 минут |
| Настройка внешнего вида | 20 минут |
| Установка GRUB | 10 минут |
| **Итого** | **2-3 часа** |

Плюс установка дополнительных программ (Chrome, VS Code и т.д.) — ещё 1-2 часа.
