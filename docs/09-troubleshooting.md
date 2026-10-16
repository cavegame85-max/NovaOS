# Шаг 9: Типичные проблемы и решения

Этот документ — сборник реальных проблем, возникших при сборке NovaOS,
и их решений. Проверено на Samsung R528 (Core 2 Duo T9300, 6 ГБ RAM).

## Проблема 1: Wi-Fi не работает

**Симптомы:**
- Интерфейс `wlp2s0` есть, но `state DOWN`.
- Или интерфейса нет вообще.

**Проверка:**

    lspci -nnk | grep -A3 -i network
    ip -br a
    dmesg | grep -iE "ath9k|firmware"

**Причины и решения:**

**A. Нет прошивки**

    dmesg | grep -i "firmware"
    Если видишь: "failed to load firmware" — нужна прошивка.
    apt install firmware-atheros

**B. NetworkManager не запущен**

    systemctl status NetworkManager
    sudo systemctl enable NetworkManager
    sudo systemctl start NetworkManager

**C. rfkill блокирует**

    rfkill list
    sudo rfkill unblock all

**D. Интерфейс DOWN вручную**

    sudo ip link set wlp2s0 up

**E. Wi-Fi сеть не находится**

    nmcli device wifi list
    nmcli device wifi rescan

## Проблема 2: Чёрный экран после GRUB

**Симптомы:** После выбора NovaOS в GRUB — чёрный экран, ничего не происходит.

**Проверка:** Нажми Ctrl+Alt+F1 — переключиться на консоль.

**Причины:**

**A. X.org не запустился**

    systemctl status lightdm
    cat /var/log/Xorg.0.log | tail -40
    Ищи строки с (EE) — это ошибки.

**B. Нет драйвера видео**

    ls /usr/lib/xorg/modules/drivers/
    Если нет nouveau_drv.so:
    apt install --reinstall xserver-xorg-video-nouveau

**C. Не хватает прошивки nouveau**

    dmesg | grep -i nouveau
    Если: "Direct firmware load for nouveau/nva8_fuc084 failed"
    Установи: apt install firmware-misc-nonfree

**D. Ошибка в конфиге X.org**

    mv /etc/X11/xorg.conf /etc/X11/xorg.conf.bak
    systemctl restart lightdm

## Проблема 3: Kernel panic — unable to mount root fs

**Симптомы:** Ядро загрузилось, но не нашло корень.

    ALERT! UUID=... does not exist. Dropping to a shell!
    (initramfs)

**Причины:**

**A. Неверный UUID в fstab**

Проверить в initramfs:

    ls /dev/sd*
    blkid

Записать правильный UUID, войти в систему, поправить fstab.

**B. Медленный USB/HDD — ядро сдалось раньше**

Добавить в GRUB: `rootdelay=10`. В `/etc/default/grub`:

    GRUB_CMDLINE_LINUX_DEFAULT="quiet rootdelay=10"

**C. Неверное имя раздела**

В `/etc/fstab` иногда указывают `/dev/sda1`, а ядро видит `/dev/sdb1`.
Решение: **только UUID**, никогда не имена.

## Проблема 4: GRUB rescue>

**Симптомы:** GRUB не нашёл свой конфиг, показывает приглашение `grub rescue>`.

**Причины:**

**A. GRUB установлен на неправильный диск**

Загрузиться с live-USB и переустановить GRUB.

**B. Потерян /boot/grub**

Проверить:

    ls (hd0,gpt3)/boot/grub

Указать вручную корень:

    set root=(hd0,gpt3)
    set prefix=(hd0,gpt3)/boot/grub
    insmod normal
    normal

**Восстановить из live-USB:**

    sudo mount /dev/sdX3 /mnt
    sudo mount /dev/sdX2 /mnt/boot/efi
    for d in dev dev/pts proc sys; do sudo mount --bind /$d /mnt/$d; done
    sudo chroot /mnt
    grub-install --target=i386-pc --boot-directory=/boot --recheck /dev/sdX
    update-grub
    exit

## Проблема 5: Plank зависает наверху

**Симптомы:** Plank появляется в неправильной позиции, не внизу.

**Причина:** Plank читает конфиг **до** того, как XFCE развернёт рабочий стол.

**Решение:** перезапускать Plank через 5 секунд после старта сессии.

Создай `~/.config/autostart/restart-plank.desktop`:

    [Desktop Entry]
    Type=Application
    Name=Restart Plank
    Exec=sh -c "sleep 5 && killall plank; sleep 1 && setsid plank > /dev/null 2>&1 < /dev/null &"
    Hidden=false
    NoDisplay=true
    X-GNOME-Autostart-enabled=true

## Проблема 6: XFCE-панель возвращается после перезагрузки

**Симптомы:** Убил панель через `killall xfce4-panel`, но после перезагрузки она снова появляется.

**Причина:** XFCE-сессия сама запускает панель.

**Решение:**

Создай `~/.config/autostart/kill-panel.desktop`:

    [Desktop Entry]
    Type=Application
    Name=Kill Panel
    Exec=sh -c "sleep 2 && killall xfce4-panel"
    Hidden=false
    NoDisplay=true
    X-GNOME-Autostart-enabled=true

## Проблема 7: Флешка «умирает» при работе

**Симптомы:**
- `I/O error, dev sdb`
- Флешка монтируется в read-only.
- Файловая система «грязная».

**Причины:**
- Дешёвые флешки с китайским контроллером (UDisk, abcd:1234).
- Перегрев при длительной записи.

**Решение:**

Проверить ФС:

    sudo fsck.ext4 -f -y /dev/sdX3

Проверить на бэдблоки:

    sudo badblocks -sv /dev/sdX3

**Правило:** если флешка **уже дважды** упала — не используй её
для основной системы. Возьми новую или установи на HDD/SSD.

## Проблема 8: Wine падает с «kernel32.dll not found»

**Симптомы:**

    wine: could not load kernel32.dll, status c0000135

**Причины:**
- Повреждён префикс Wine.
- Неправильная архитектура префикса (win32 vs win64).

**Решение:**

Удалить префикс и создать заново:

    wineserver -k
    rm -rf ~/.wine
    WINEARCH=win32 WINEPREFIX=~/.wine wineboot -i

**Проверка:**

    wine cmd /c "echo Wine works"

## Проблема 9: .exe не запускается двойным кликом

**Симптомы:** Двойной клик по .exe ничего не делает, или открывается «Пустое окно».

**Проверка:**

    xdg-mime query default application/x-ms-dos-executable

Должно быть `wine.desktop`.

**Решение:**

    sudo update-mime-database /usr/share/mime
    xdg-mime default wine.desktop application/x-ms-dos-executable
    sudo systemctl restart systemd-binfmt

Проверить binfmt:

    cat /proc/sys/fs/binfmt_misc/DOSWin

Должно быть `enabled`.

## Проблема 10: Мало свободного места

**Симптомы:** `No space left on device` при apt install.

**Проверка:**

    df -h /
    du -sh /var/cache/apt/archives/
    du -sh /var/log/

**Решение:**

Очистить кэш apt:

    sudo apt clean
    sudo apt autoremove -y

Очистить старые журналы:

    sudo journalctl --vacuum-size=100M

Удалить ненужные языковые пакеты:

    sudo apt install localepurge
    sudo localepurge

## Проблема 11: Медленно работает система на USB 2.0

**Причины:**
- USB 2.0 даёт максимум 30 МБ/с чтения.
- Флешка «дешёвая» — медленнее в 2–3 раза.

**Решения:**

1. **Перенести систему на HDD/SSD** — самый правильный путь.
2. **Отключить лишнее:**
       systemctl disable bluetooth ModemManager avahi-daemon
3. **Уменьшить swappiness:**
       echo "vm.swappiness=10" | sudo tee /etc/sysctl.d/99-swap.conf
4. **Использовать zRAM:**
       sudo apt install zram-tools

## Проблема 12: Timeshift работает в фоне и тормозит систему

**Симптомы:**
- `pgrep -a timeshift` показывает процесс.
- `mount | grep timeshift` показывает монтирование.

**Решение:** подождать окончания (10–30 минут), или:

    sudo killall timeshift
    sudo killall rsync

**На будущее:** отключить автоматические снапшоты:

Открой Timeshift → **Настройки** → **Расписание** → **Отключить**.

## Проблема 13: GParted зависает при ужимании HDD

**Симптомы:**
- Прогресс-бар на 15–20% не двигается **часами**.
- Счётчик времени операции **не меняется**.

**Причина:** HDD 5400 rpm + большой раздел (100+ ГБ данных) =
операция физически не проходит.

**Решение:**

1. Прервать: **Прервать операцию** в GParted (риск).
2. Или **подождать 5–8 часов**.
3. **Или перейти к плану B:**
   - Сохранить данные Mint на внешний носитель.
   - Удалить sda3 полностью.
   - Создать новые разделы с нуля (sda3 = NovaOS, sda4 = данные).

**Правило:** ужимать ext4 с 400+ ГБ данных на HDD — **плохая идея**.

## Проблема 14: Неверный UUID в fstab после переустановки

**Симптомы:** После изменения разделов система не грузится.

**Проверка (в initramfs или live-USB):**

    blkid /dev/sdX3

**Решение:** поправить `/etc/fstab` и `/boot/grub/grub.cfg`.

    sudo blkid /dev/sdX3
    sudo nano /mnt/novaos/etc/fstab
    Обновить UUID.
    sudo nano /mnt/novaos/etc/default/grub
    Обновить UUID в GRUB_CMDLINE_LINUX если нужно.
    sudo chroot /mnt/novaos update-grub

## Проблема 15: «Target is busy» при umount

**Симптомы:**

    umount: /mnt/novaos/run: target is busy.

**Причины:** Timeshift, монтирование, активные процессы.

**Решение:**

    sudo umount -l /mnt/novaos/run

**Или:** найти, кто держит:

    sudo lsof +D /mnt/novaos/run

## Полезные команды диагностики

Проверка системы в целом:

    systemctl --failed
    dmesg | grep -iE "error|fail|warn" | tail -30
    df -h
    free -h
    journalctl -p err -b

Проверка железа:

    lspci -nnk
    lsusb
    lsblk -f
    lscpu

Проверка сети:

    ip -br a
    ip route
    cat /etc/resolv.conf
    ping -c 3 8.8.8.8
    ping -c 3 ya.ru

Проверка графики:

    echo $DISPLAY
    xrandr
    glxinfo | head -20

## Куда смотреть логи

| Лог | Что искать |
|---|---|
| /var/log/Xorg.0.log | Ошибки X.org |
| /var/log/lightdm/lightdm.log | Ошибки LightDM |
| /var/log/dpkg.log | История установки пакетов |
| /var/log/apt/history.log | Команды apt |
| journalctl -b | Журнал текущей загрузки |
| ~/.xsession-errors | Ошибки сессии пользователя |

## Когда всё сломалось

**Если система не грузится:**

1. **Не паникуй.**
2. Загрузиться с **live-USB** (Mint или другая система).
3. Смонтировать раздел NovaOS:
       sudo mount /dev/sdX3 /mnt
4. Проверить логи, поправить fstab / grub.
5. Размонтировать, перезагрузиться.

**Если повреждена ФС:**

    sudo fsck.ext4 -f -y /dev/sdX3

**Если потерян GRUB:**

Восстановить по разделу 8.

## Заключение

**90% проблем решаются:**
- Проверкой логов (`dmesg`, `journalctl`, `/var/log/`).
- Правильными UUID в `fstab`.
- Загрузкой с live-USB.

**Правило:** не бойся ошибиться. У тебя есть **бэкап важного**,
**live-USB** и **документация**. Восстановить можно всё.

## Следующий шаг

→ `10-faq.md` — часто задаваемые вопросы.
