# NovaOS

**NovaOS** — лёгкая операционная система на базе Debian 12 (bookworm),
собранная вручную для старого железа (Intel Core 2 Duo, NVIDIA GeForce 310M).

## О проекте

NovaOS — это эксперимент по созданию собственного дистрибутива Linux
с нуля: от debootstrap до готового рабочего стола с Wine.

**Автор:** [@cavegame85-max](https://github.com/cavegame85-max)
**Видео о сборке:** TikTok

## Характеристики

| Компонент | Что используется |
|---|---|
| Ядро | Linux 6.1.0-53-amd64 |
| База | Debian 12 (bookworm) |
| Инициализация | systemd |
| Рабочий стол | XFCE 4.18 |
| Док | Plank |
| Окна | Arc-Dark |
| Иконки | Papirus-Dark |
| Шрифт | Inter / JetBrains Mono |
| Экран входа | LightDM + GTK Greeter |
| Wine | 8.0 + binfmt_misc (двойной клик по .exe) |

## Структура репозитория
NovaOS/
├── README.md — этот файл
├── configs/ — конфигурационные файлы
│ ├── fstab.example
│ ├── grub.example
│ ├── lightdm-gtk-greeter.conf
│ ├── wine-binfmt.conf
│ ├── wine.desktop
│ └── fastfetch-config.jsonc
├── docs/ — документация
├── logo/ — логотип NovaOS
├── packages/ — список установленных пакетов
└── scripts/ — скрипты
└── novaos-backup

## Как собрать

**Требования:**
- Свежая установка Debian 12
- Интернет
- ~15 ГБ свободного места
- 2+ часа времени

**Шаги (кратко):**
1. Установить `debootstrap` и базовые инструменты.
2. Собрать базу Debian: `debootstrap --variant=minbase bookworm /target http://deb.debian.org/debian`.
3. Настроить chroot: DNS, apt, locale, hostname.
4. Установить ядро: `apt install linux-image-amd64 linux-headers-amd64`.
5. Установить прошивки: `firmware-linux-nonfree`, `firmware-atheros`.
6. Установить графику: `xorg`, `xfce4`, `lightdm`, `network-manager`.
7. Настроить LightDM, плашку, темы — см. `configs/`.
8. Установить Wine: `wine wine32 winetricks` + binfmt_misc.
9. Установить GRUB: `grub-install --target=i386-pc`.
10. Готово.

**Подробная инструкция — в `docs/`** (в разработке).

## Статус проекта

- ✅ Базовая система (Debian 12)
- ✅ Ядро, systemd, GRUB
- ✅ Графика (XFCE, X.org, LightDM)
- ✅ Оформление (Arc-Dark, Papirus, Inter)
- ✅ Plank + часы + меню
- ✅ Wine + binfmt_misc
- ⏳ Создание ISO, скоро появится в файлах проекта

## Лицензия

Код и конфиги — под лицензией MIT.
Логотип — авторский.

## Ссылки

- YouTube/TikTok — tiktok.com/@cavegame47
- GitHub Issues — баги и предложения
