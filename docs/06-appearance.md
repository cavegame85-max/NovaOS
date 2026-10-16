# Шаг 6: Оформление — темы, иконки, шрифты, Plank

## Философия оформления NovaOS

Цель — сделать систему **лёгкой**, но **красивой**. Не копировать Windows или macOS, а взять лучшее:

- **Win11 (70%)** — панель снизу, иконки в доке, стиль.
- **macOS (30%)** — всплывающие иконки при наведении (через Plank).

**Палитра:**
- Тёмная тема (**Arc-Dark**) — основной.
- Иконки **Papirus-Dark** — современные, не «детские».
- Шрифт **Inter** — интерфейс.
- Шрифт **JetBrains Mono** — терминал и код.

## Установка темы и иконок

Войди в chroot:

    sudo mount --bind /dev /mnt/novaos/dev
    sudo mount --bind /dev/pts /mnt/novaos/dev/pts
    sudo mount --bind /proc /mnt/novaos/proc
    sudo mount --bind /sys /mnt/novaos/sys
    sudo mount --bind /run /mnt/novaos/run
    sudo cp /etc/resolv.conf /mnt/novaos/etc/resolv.conf
    sudo chroot /mnt/novaos /bin/bash
    export TERM=xterm

Установи тему и иконки:

    apt install -y arc-theme papirus-icon-theme gnome-themes-extra gtk2-engines-murrine

**Разбор:**
- `arc-theme` — тёмная и светлая темы.
- `papirus-icon-theme` — 25+ тысяч иконок.
- `gnome-themes-extra` — дополнительные темы.
- `gtk2-engines-murrine` — движок для GTK2-тем.

**Размер:** ~50 МБ.

## Установка шрифтов

    apt install -y fonts-inter fonts-jetbrains-mono fonts-firacode fonts-liberation fonts-noto

**Разбор:**
- `fonts-inter` — современный sans-serif.
- `fonts-jetbrains-mono` — моноширинный для кода.
- `fonts-firacode` — альтернативный моноширинный.
- `fonts-liberation` — замена Arial/Times.
- `fonts-noto` — универсальный, поддержка Unicode.

**Размер:** ~300 МБ.

**Проверка:**

    fc-list | grep -iE "inter|jetbrains" | head -5

Должны быть файлы шрифтов.

## Установка Plank (док)

    apt install -y plank

**Разбор:** Plank — минималистичный док с эффектом macOS.

**Размер:** ~5 МБ.

## Применение настроек (внутри NovaOS, после загрузки)

**Эти команды — после первой графической загрузки.**

### Тема, иконки, шрифт

    xfconf-query -c xsettings -p /Net/ThemeName -s "Arc-Dark"
    xfconf-query -c xsettings -p /Net/IconThemeName -s "Papirus-Dark"
    xfconf-query -c xfwm4 -p /general/theme -s "Arc-Dark"
    xfconf-query -c xsettings -p /Gtk/FontName -s "Inter 10"
    xfconf-query -c xsettings -p /Gtk/MonospaceFontName -s "JetBrains Mono 10"

**Разбор:**
- `/Net/ThemeName` — тема GTK.
- `/Net/IconThemeName` — иконки.
- `/general/theme` — тема оконного менеджера (рамки окон).
- `/Gtk/FontName` — шрифт интерфейса.
- `/Gtk/MonospaceFontName` — моноширинный шрифт.

### Настройка терминала

Создай конфиг:

    mkdir -p ~/.config/xfce4/terminal
    cat > ~/.config/xfce4/terminal/terminalrc <<'EOF'
    [Configuration]
    FontName=JetBrains Mono 11
    ColorForeground=#c5c8c6
    ColorBackground=#1d1f21
    MiscBackgroundDarkness=0.90
    MiscCursorShape=TERMINAL_CURSOR_SHAPE_BLOCK
    EOF

**Разбор:**
- Шрифт **JetBrains Mono 11**.
- Цвет фона **#1d1f21** (тёмный).
- Текст **#c5c8c6** (светло-серый).
- Прозрачность **0.90**.
- Курсор — блок.

### Настройка Plank

Запусти Plank:

    plank &

**Правый клик по доку** → **Настройки Plank** (или `plank --preferences`).

**Настройки:**
- **Тема:** Matte (тёмная) или Transparent.
- **Позиция:** снизу.
- **Выравнивание:** по центру.
- **Размер иконок:** 48.
- **Скрывать при перекрытии:** выкл (чтобы Plank не прятался).
- **Увеличивать при наведении:** вкл.

### Добавление docklets в Plank

В **Настройки Plank** → вкладка **Docklets** → **`+`**:

- **Applications** — меню «Пуск».
- **Clock** — часы и дата.
- **Battery** — батарея.
- **Trash** — корзина.

**Настройка формата часов** (в настройках Clock):

    %H:%M  %d.%m

Даст: `22:45  05.10`

## Убираем XFCE-панель (если нужна только Plank)

**Если хочешь Plank вместо XFCE-панели:**

1. `xfce4-panel --quit` — закрыть.
2. `rm -rf ~/.config/xfce4/panel` — удалить конфиг.
3. Создать `~/.config/autostart/kill-panel.desktop`:

    [Desktop Entry]
    Type=Application
    Name=Kill Panel
    Exec=sh -c "sleep 2 && killall xfce4-panel"
    Hidden=false
    NoDisplay=true
    X-GNOME-Autostart-enabled=true

**Смысл:** при входе в сессию **через 2 секунды** убиваем панель.

## Автозапуск Plank

Создай `~/.config/autostart/restart-plank.desktop`:

    [Desktop Entry]
    Type=Application
    Name=Restart Plank
    Exec=sh -c "sleep 5 && killall plank; sleep 1 && setsid plank > /dev/null 2>&1 < /dev/null &"
    Hidden=false
    NoDisplay=true
    X-GNOME-Autostart-enabled=true

**Смысл:** через **5 секунд после старта** Plank убивается и запускается заново — уже в правильной позиции (снизу).

## Настройка экрана входа LightDM

Открой `/etc/lightdm/lightdm-gtk-greeter.conf`:

    sudo nano /etc/lightdm/lightdm-gtk-greeter.conf

Впиши:

    [greeter]
    background=/usr/share/backgrounds/novaos-default.jpg
    theme-name=Arc-Dark
    icon-theme-name=Papirus-Dark
    font-name=Inter 10
    xft-antialias=true
    xft-dpi=96
    xft-hintstyle=hintslight
    xft-rgba=rgb
    indicators=~host;~spacer;~clock;~spacer;~session;~power
    clock-format=%H:%M  %d.%m.%Y
    position=50%,center
    user-background=true

**Разбор:**
- `background=` — путь к обоям. Замени `novaos-default.jpg` на свой файл.
- `theme-name=Arc-Dark` — тёмная тема.
- `icon-theme-name=Papirus-Dark` — иконки.
- `font-name=Inter 10` — шрифт.
- `indicators=...` — верхняя панель (host, часы, сессия, питание).
- `clock-format=` — формат часов.
- `position=50%,center` — окно входа по центру.

**Обои**: положи файл в `/usr/share/backgrounds/novaos-default.jpg`:

    sudo mkdir -p /usr/share/backgrounds
    sudo cp /home/USERNAME/Pictures/wallpaper.jpg /usr/share/backgrounds/novaos-default.jpg

## Логотип fastfetch

Создай `~/.config/fastfetch/nova-logo.txt`:

    ███╗   ██╗ ██████╗ ██╗   ██╗ █████╗
    ████╗  ██║██╔═══██╗██║   ██║██╔══██╗
    ██╔██╗ ██║██║   ██║██║   ██║███████║
    ██║╚██╗██║██║   ██║╚██╗ ██╔╝██╔══██║
    ██║ ╚████║╚██████╔╝ ╚████╔╝ ██║  ██║
    ╚═╝  ╚═══╝ ╚═════╝   ╚═══╝  ╚═╝  ╚═╝
              ██████╗ ███████╗
             ██╔═══██╗██╔════╝
             ██║   ██║███████╗
             ██║   ██║╚════██║
             ╚██████╔╝███████║
              ╚═════╝ ╚══════╝

Создай `~/.config/fastfetch/config.jsonc`:

    {
        "$schema": "https://github.com/fastfetch-cli/fastfetch/raw/dev/doc/json_schema.json",
        "logo": {
            "source": "~/.config/fastfetch/nova-logo.txt",
            "type": "file",
            "padding": {
                "top": 0,
                "left": 2,
                "right": 2
            }
        },
        "display": {
            "separator": "  ",
            "color": {
                "keys": "blue"
            }
        },
        "modules": [
            "title", "separator",
            "os", "host", "kernel", "uptime", "packages", "shell",
            "display", "de", "wm", "wmtheme", "theme", "icons",
            "font", "cursor", "terminal", "terminalfont",
            "cpu", "gpu", "memory", "swap", "disk",
            "localip", "battery", "locale",
            "break", "colors"
        ]
    }

**Проверка:**

    fastfetch

Покажет логотип NovaOS + информацию о системе.

## Что получилось

- Установлены **темы, иконки, шрифты**.
- Установлен **Plank** — док в стиле macOS.
- Настроен **экран входа LightDM**.
- Настроен **fastfetch** с логотипом.
- Система выглядит **как NovaOS**, а не как Debian по умолчанию.

## Следующий шаг

→ `07-wine.md` — Wine для запуска .exe-программ.
