# Шаг 7: Wine — запуск .exe-программ

## Что такое Wine

Wine — это **слой совместимости**, который позволяет запускать Windows-программы (.exe) в Linux **без эмуляции** и **без самой Windows**.

**Как это работает:**
- Wine переводит **системные вызовы Windows** в **системные вызовы Linux**.
- Реализует **Windows API** (kernel32.dll, user32.dll, gdi32.dll и т.д.) на Linux.
- Создаёт **префикс** — папку `~/.wine`, которая выглядит как `C:\` для программ.

**Что Wine делает хорошо:**
- Notepad++, WinRAR, 7-Zip.
- Старые игры (до 2015).
- Мелкие утилиты.

**Что Wine делает плохо:**
- Electron-приложения (VS Code, Discord) — тормозят или не работают.
- Новые игры с античитом.
- Photoshop, MS Office.

**Правило:** если у программы **есть нативная Linux-версия** — используй её. Wine — только для того, чего на Linux нет.

## Установка Wine (в chroot)

Войди в chroot:

    sudo mount --bind /dev /mnt/novaos/dev
    sudo mount --bind /dev/pts /mnt/novaos/dev/pts
    sudo mount --bind /proc /mnt/novaos/proc
    sudo mount --bind /sys /mnt/novaos/sys
    sudo mount --bind /run /mnt/novaos/run
    sudo cp /etc/resolv.conf /mnt/novaos/etc/resolv.conf
    sudo chroot /mnt/novaos /bin/bash
    export TERM=xterm

Добавь поддержку 32-битных пакетов:

    dpkg --add-architecture i386
    apt update

**Зачем:** большинство .exe — **32-битные**. Без i386 Wine не сможет их запустить.

Установи Wine:

    apt install -y wine wine32 wine64 winetricks binfmt-support

**Разбор:**
- `wine` — метапакет.
- `wine32` — 32-битные программы.
- `wine64` — 64-битные.
- `winetricks` — установка зависимостей (VC++ Runtime, .NET, шрифты).
- `binfmt-support` — ядро узнаёт, что .exe запускать через Wine.

**Размер:** ~1.5 ГБ. **Время:** 15–25 минут.

**Проверка:**

    wine --version

Должно быть `wine-8.0 (Debian 8.0~repack-4)` или похоже.

## Настройка binfmt_misc (автозапуск .exe)

**Что это:** модуль ядра, который **автоматически** запускает .exe через Wine при **двойном клике**.

### 1. Загрузить модуль

    modprobe binfmt_misc

Проверка:

    lsmod | grep binfmt

Должно быть `binfmt_misc`.

### 2. Смонтировать

    mount binfmt_misc -t binfmt_misc /proc/sys/fs/binfmt_misc

Проверка:

    mount | grep binfmt

Должна быть строка.

### 3. Создать постоянную регистрацию

Создай `/etc/binfmt.d/wine.conf`:

    sudo nano /etc/binfmt.d/wine.conf

Впиши **одну строку**:

    :DOSWin:M::MZ::/usr/bin/wine:

**Что это значит:**
- `DOSWin` — имя регистрации.
- `M::MZ::` — файлы, начинающиеся с байтов **MZ** (сигнатура .exe).
- `/usr/bin/wine` — чем запускать.

Сохрани: Ctrl+O, Enter, Ctrl+X.

Проверка:

    systemctl restart systemd-binfmt
    ls /proc/sys/fs/binfmt_misc/ | grep -i dos

Должно быть `DOSWin`.

## Создание wine.desktop

Чтобы **двойной клик** в файловом менеджере открывал .exe:

    sudo nano /usr/share/applications/wine.desktop

Впиши:

    [Desktop Entry]
    Name=Wine Windows Program Loader
    Exec=wine %f
    Type=Application
    Terminal=false
    MimeType=application/x-ms-dos-executable;application/x-msdownload;application/x-msdos-program;
    Icon=wine
    Categories=System;

Сохрани: Ctrl+O, Enter, Ctrl+X.

**Регистрация MIME-типа:**

    sudo update-mime-database /usr/share/mime
    xdg-mime default wine.desktop application/x-ms-dos-executable
    xdg-mime query default application/x-ms-dos-executable

Последняя команда должна вернуть `wine.desktop`.

## Первая инициализация Wine (после загрузки)

**Эти команды — после первой графической загрузки.**

Создаём префикс Wine:

    WINEARCH=win32 WINEPREFIX=~/.wine wineboot -i

**Разбор:**
- `WINEARCH=win32` — 32-битный префикс (для большинства .exe).
- `WINEPREFIX=~/.wine` — где создать.
- `wineboot -i` — инициализация.

**Возможные ошибки при инициализации:**
- `err:ole:...` — не критично, Wine ищет необязательные компоненты.
- `err:winebus:...` — USB-сервисы, не нужны для базовой работы.
- Главное — в конце должно быть: `wine: configuration in L"~/.wine" has been updated.`

**Проверка:**

    wine cmd /c "echo Wine works"

Должно вывести `Wine works`.

## Установка Windows 10 в префикс

Wine по умолчанию представляется как **Windows 7**. Современные программы требуют Windows 10.

    winetricks -q win10

**Займёт 1–2 минуты.**

Проверка:

    winecfg &

Откроется окно. Вкладка **Applications** → внизу должна быть **Windows 10**. Закрой.

## Установка библиотек для Electron-приложений

**Если хочешь запускать программы на Electron (VS Code, Discord):**

    winetricks -q corefonts vcrun2019

**Разбор:**
- `corefonts` — стандартные шрифты (Arial, Times, Verdana).
- `vcrun2019` — Visual C++ Runtime 2019.

**Размер:** ~50 МБ. **Время:** 3–5 минут.

**Возможная проблема:** `winetricks` качает файлы с зеркал, некоторые могут быть **недоступны** из России. **Решение** — использовать `web.archive.org` (winetricks делает это сам).

## Проверка работы Wine

### Тест 1: Notepad

    wine notepad &

Откроется **стандартный Windows Notepad**. Если работает — Wine настроен.

### Тест 2: Простой .exe

Скачай **7-Zip** (консольная версия):

    cd ~/Загрузки
    wget -O 7zr.exe https://www.7-zip.org/a/7zr.exe
    wine 7zr.exe

Должна вывестись справка 7-Zip. **.exe работает.**

### Тест 3: Двойной клик

В **Thunar** (файловый менеджер) **дважды кликни** по `.exe` — должно открыться окно Wine.

## Ограничения Wine

| Что | Работает? |
|---|---|
| Notepad++, 7-Zip, WinRAR | ✅ Отлично |
| Старые игры (2000–2015) | ✅ Хорошо |
| VS Code, Discord (Electron) | ⚠️ Тормозит |
| Новые игры с античитом | ❌ Не работает |
| Photoshop, MS Office 2016+ | ❌ Не работает |
| .NET 6+ приложения | ❌ Не работает |

**Правило:** если есть **нативная Linux-версия** — используй её. Wine — только для **Windows-only** программ.

## Скрипт для ярлыков .exe

Если хочешь создать **ярлык** для .exe-программы на рабочем столе:

Создай `~/.local/share/applications/myapp.desktop`:

    [Desktop Entry]
    Name=My Windows App
    Exec=wine /home/USERNAME/.wine/drive_c/Program\\ Files/MyApp/app.exe
    Type=Application
    Icon=wine
    Terminal=false
    Categories=Wine;

**Замени** путь на свой.

## Что получилось

- Установлен **Wine 8.0**.
- Поддержка **32-битных** .exe.
- **binfmt_misc** — двойной клик по .exe работает.
- **wine.desktop** — ассоциация файлов.
- Префикс **Windows 10**.
- **corefonts** и **VC++ Runtime** для Electron.

## Следующий шаг

→ `08-grub.md` — загрузчик GRUB.
