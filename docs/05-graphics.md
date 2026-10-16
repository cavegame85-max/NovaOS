# Шаг 5: Графика — X.org, XFCE, LightDM

## Что такое графический стек Linux

Графика в Linux — это **многослойная система**:

    Приложения (Firefox, VS Code)
            ↓
    Рабочий стол (XFCE, GNOME, KDE)
            ↓
    Оконный менеджер (XFWM, Mutter, KWin)
            ↓
    Графический сервер (X.org или Wayland)
            ↓
    Драйвер видео (nouveau, nvidia, i915)
            ↓
    Видеокарта

**Для NovaOS выбрано:**
- **X.org** — старый, надёжный, хорошо работает на старом железе.
- **XFCE 4.18** — лёгкий, быстрый, минималистичный.
- **LightDM** — лёгкий менеджер входа.

**Wayland НЕ используется** — он тяжелее и хуже работает с NVIDIA GeForce 310M.

## Установка X.org (в chroot)

Войди в chroot:

    sudo mount --bind /dev /mnt/novaos/dev
    sudo mount --bind /dev/pts /mnt/novaos/dev/pts
    sudo mount --bind /proc /mnt/novaos/proc
    sudo mount --bind /sys /mnt/novaos/sys
    sudo mount --bind /run /mnt/novaos/run
    sudo cp /etc/resolv.conf /mnt/novaos/etc/resolv.conf
    sudo chroot /mnt/novaos /bin/bash
    export TERM=xterm

Установи базовую графику:

    apt install -y xorg xinit x11-utils x11-xserver-utils

**Разбор:**
- `xorg` — метапакет: X-сервер, библиотеки, утилиты.
- `xinit` — команда `startx`.
- `x11-utils`, `x11-xserver-utils` — утилиты (`xrandr`, `xset`).

**Размер:** ~150 МБ. **Время:** 3–5 минут.

**Проверка:**

    which Xorg

Должно быть `/usr/bin/Xorg`.

## Установка драйверов видео

    apt install -y xserver-xorg-video-nouveau xserver-xorg-video-fbdev xserver-xorg-video-vesa

**Разбор:**
- `nouveau` — открытый драйвер NVIDIA для старых GeForce.
- `fbdev` — фреймбуфер (аварийный).
- `vesa` — VESA (аварийный, если остальные не работают).

**Проверка:**

    ls /usr/lib/xorg/modules/drivers/

Должны быть `nouveau_drv.so`, `fbdev_drv.so`, `vesa_drv.so`.

## Установка драйверов ввода

    apt install -y xserver-xorg-input-all

Включает `evdev`, `libinput`, `synaptics` — для мыши, клавиатуры, тачпада.

## Установка XFCE

    apt install -y xfce4 xfce4-goodies xfce4-terminal thunar mousepad

**Разбор:**
- `xfce4` — метапакет рабочего стола (xfwm4, xfce4-panel, xfdesktop4, xfce4-session).
- `xfce4-goodies` — плагины: часы, батарея, сеть, звук.
- `xfce4-terminal` — терминал.
- `thunar` — файловый менеджер.
- `mousepad` — текстовый редактор.

**Размер:** ~250 МБ. **Время:** 5–8 минут.

**Что установится дополнительно:**
- `xfwm4` — оконный менеджер.
- `xfdesktop4` — рабочий стол (иконки, обои).
- `xfce4-panel` — панель.
- `xfce4-session` — менеджер сессии.
- `xfce4-settings` — настройки.

## Установка LightDM

    apt install -y lightdm lightdm-gtk-greeter

**При установке спросит:** «Default display manager» → выбери **lightdm**.

**Разбор:**
- `lightdm` — менеджер входа.
- `lightdm-gtk-greeter` — GTK-интерфейс экрана входа.

**Размер:** ~10 МБ.

**Включение автозапуска:**

    systemctl enable lightdm

**Возможный ответ:** «Running in chroot, ignoring command 'enable'» — нормально.

**Проверка:**

    ls -la /etc/systemd/system/display-manager.service

Должен быть симлинк на `/lib/systemd/system/lightdm.service`.

## Установка D-Bus и утилит

    apt install -y dbus-x11 xdg-user-dirs xdg-utils fonts-liberation

**Разбор:**
- `dbus-x11` — D-Bus для графических приложений (нужен для XFCE).
- `xdg-user-dirs` — создаёт папки `Документы`, `Загрузки`, `Изображения`.
- `xdg-utils` — `xdg-open` (открытие файлов двойным кликом).
- `fonts-liberation` — замена Microsoft-шрифтов (Arial, Times).

## Проверка установки

    dpkg -l | grep -E "xorg|xfce4|lightdm" | wc -l

Должно быть **50–80** пакетов.

    which Xorg xfce4-session lightdm thunar

Все четыре должны быть найдены.

    du -sh /usr /var

`/usr` должен быть **~1–2 ГБ**, `/var` — **~300 МБ**.

## Первый запуск графики (после загрузки)

**Порядок:**
1. Выходишь из chroot: `exit`.
2. Размонтируешь папки: `sudo umount /mnt/novaos/{run,sys,proc,dev/pts,dev}`.
3. Устанавливаешь GRUB (см. `08-grub.md`).
4. Загружаешься с флешки/HDD.
5. GRUB → NovaOS → systemd поднимает LightDM → экран входа.

**Возможные проблемы на первом запуске:**

**1. Чёрный экран, LightDM не стартует**

Проверь:
    systemctl status lightdm
    cat /var/log/Xorg.0.log | tail -40

Часто причина — нет драйвера видео или ошибка в конфиге X.org.

**2. Экран входа есть, но после входа — чёрный экран**

    cat ~/.xsession-errors

Возможные причины: XFCE не может запуститься, конфликт с настройками.

**3. «Failed to start X server»**

    ls /usr/lib/xorg/modules/drivers/

Проверь наличие `nouveau_drv.so`. Если нет — переустанови `xserver-xorg-video-nouveau`.

## Что получилось

- Установлен **X.org** — графический сервер.
- Установлен **XFCE 4.18** — рабочий стол.
- Установлен **LightDM** — экран входа.
- Установлены **драйверы видео** (nouveau, fbdev, vesa).
- Установлены **драйверы ввода** (мышь, клавиатура, тачпад).
- Система **готова к графической загрузке**.

## Следующий шаг

→ `06-appearance.md` — оформление: темы, иконки, шрифты, Plank.
