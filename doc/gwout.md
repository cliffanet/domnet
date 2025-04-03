# Установка зарубежного сервера (FreeBSD)

Скачиваем образ с сайта [freebsd](https://download.freebsd.org/ftp/releases/amd64/amd64/ISO-IMAGES/).

Например:

    FreeBSD-12.3-RELEASE-amd64-disc1.iso

Устанавливаем пакеты:

```bash
pkg install bash mc
```

Соглашаемся везде: `y` и `enter`.

Выполняем:

```bash
ln -s /usr/local/bin/perl /usr/bin/perl
ln -s /etc/rc.d /rc.d
```

У пользователя `root` правим настройки:

```bash
pw usermod root -s /usr/local/bin/bash -c root
```

Добавляем пользователя:

<pre>
[root@generic ~]# 📝 <b>adduser</b>
Username: 📝 <b>cliff</b>
Full name: 📝 <b>Cliff</b>
Uid (Leave empty for default):
Login group [cliff]: 📝 <b>wheel</b>
Login group is wheel. Invite cliff into other groups? []:
Login class [default]:
Shell (sh csh tcsh bash rbash nologin) [sh]: 📝 <b>bash</b>
Home directory [/home/cliff]:
Home directory permissions (Leave empty for default):
Use password-based authentication? [yes]:
Use an empty password? (yes/no) [no]:
Use a random password? (yes/no) [no]:
Enter password: 📝 <b>пароль</b>
Enter password again: 📝 <b>пароль</b>
Lock out the account after creation? [no]:
Username   : cliff
Password   : *****
Full Name  : Cliff
Uid        : 1002
Class      :
Groups     : wheel
Home       : /home/cliff
Home Mode  :
Shell      : /usr/local/bin/bash
Locked     : no
OK? (yes/no): 📝 <b>yes</b>
adduser: INFO: Successfully added (cliff) to the user database.
Add another user? (yes/no): 📝 <b>no</b>
Goodbye!
[root@generic ~]#
</pre>

## Способы отредактировать файлы

В тексте этого документа будут встречаться настройки, которые надо изменить в определённых файлах.

* `ee имя-файла`

    Этот редактор есть в системе по умолчанию. Он консольный.
    
    В этом редакторе, чтобы попасть в меню, надо нажать ESC и подождать. 
    Следом нажимаем два раза Enter - это пункты: `выйти` и `сохранить изменения`.
    
    Большинство необходимых комбинаций клавиш там указано в самом верху. `^` - это клавиша `ctrl`.

* `mc`

    Это Midnight Commander. Те, кто хоть раз пользовался Norton Commander, без труда разберутся.

* `cat > имя-файла`

    Этот способ удобен для ситуаций, где указан весь файл целиком.
    
    ⚠ Эта команда полностью стирает предыдущее содержимое файла!!!
    
    После ввода команды - вставляем содержимое файла и жмём `ctrl` + `D`

## Обновление безопасности ОС

Выполняем скачивание обновлений:

```bash
freebsd-update fetch
```

В конце выполнения скрипт показывает, какие файлы будут изменены.
Несколько раз нажимаем `q`. Теперь запускаем установку:

```bash 
freebsd-update install
```

Перезагружаемся:

```bash
reboot
```

## Пересборка ядра

Нужно только там, где нужен файрвол, встроенный в ядро.

```bash
cd /sys/amd64/conf
```

Делаем свой конфиг (внимание! Имя файла!)

```bash
cp GENERIC SERVER
```

```bash
ee SERVER
```

Параметр `ident` д.б. таким же, как имя файла

    ident           SERVER

    options         IPFIREWALL                      # firewall
    options         IPFIREWALL_VERBOSE              # enable logging to syslogd(8)
    options         IPFIREWALL_VERBOSE_LIMIT=100    # limit verbosity
    options         IPFIREWALL_DEFAULT_TO_ACCEPT    # allow everything by default

    #options         DUMMYNET                       # only for router
    #options         IPDIVERT                       # only for router

Выполняем:

```bash
config SERVER
```

```bash
cd ../compile/SERVER
```

Компилируем и устанавливаем:

```bash
make cleandepend && make depend && make all install clean
```

## SSH

Редактируем:

```bash
ee /etc/ssh/sshd_config
```

Правим параметры (найти в файле и изменить):

    Port 9122
    
    PrintMotd no

Выполняем:

```bash
/rc.d/sshd reload
```

## mc config

```bash
fetch -o /tmp/mc.tar http://doc.northnet.ru/northnet/freebsdbase/mc.tar
tar -xf /tmp/mc.tar -C /
rm /tmp/mc.tar
```

## TimeZone

Правим локальный timezone (если не было сделано при установке):

```bash
tzsetup
```

Отвечаем:

    Is this machine's CMOS clock set to UTC?
    
    No -> Europe -> Russian Federation -> MSK+00 - Moscow area

На вопрос:

    Does the abbreviation `MSK' look reasonable?
    
    Yes

## NTP - синхронизация времени

Либо через `/etc/crontab`:

    */30    *       *       *       *       root    /usr/sbin/ntpdate -u ntp.northnet.ru 192.168.50.100 192.168.50.50 > /dev/null

Либо через `ntpd`:

* В файле `/etc/rc.conf` (возможно, оно там уже оказалось при установке)

    ```
    ntpd_enable="YES"
    ```

* Запускаем

    ```bash
    /rc.d/ntpd start
    ```

* Через несколько минут проверяем состояние

    ```bash
    ntpq -p
    ```
    
    ```
         remote           refid      st t when poll reach   delay   offset  jitter
    ==============================================================================
     0.freebsd.pool. .POOL.          16 p    -   64    0    0.000   +0.000   0.002
    +128.0.142.251   194.190.168.1    2 u  464 1024  377    5.380   -0.891   0.732
    *185.209.85.222  62.231.6.98      2 u  467 1024  377    1.221   -0.667   0.663
    +ground.corbina. 194.58.204.148   2 u  686 1024  377    2.683   -1.067   0.925
    +ns1.ooonet.ru   89.109.251.22    2 u  721 1024  377   28.561   -0.586   1.007
    -time100.stupi.s .PPS.            1 u  244 1024  377   40.280  +10.429   0.812
    ```
    
    Должна быть строка, которая начинается с `*`.

Либо через `cron` + `ntpdate`:

* В файле `/etc/crontab`

    ```
    */20    *       *       *       *       root    /usr/sbin/ntpdate ntp.northnet.ru 192.168.50.100 192.168.50.50 2>&1 >> /var/log/ntpdate.log
    ```

## Файрвол



## /etc/rc.conf

Пример файла:


    # Set dumpdev to "AUTO" to enable crash dumps, "NO" to disable
    dumpdev="AUTO"
    hostname="doc.northnet.ru"
    keymap="ru.kbd"

    defaultrouter="192.168.50.1"
    ifconfig_igb0="inet 192.168.50.1000/24"
    
    tcp_extensions="NO"
    tcp_drop_synfin="YES"
    icmp_drop_redirect="YES"
    icmp_log_redirect="YES"
    sshd_enable="YES"
    ntpd_enable="YES"
    sendmail_enable="NONE"
    sendmail_submit_enable="NO"
    sendmail_outbound_enable="NO"
    sendmail_msp_queue_enable="NO"

    firewall_enable="YES"
    firewall_type="Server"

## /etc/sysctl.conf

Некоторые дополнительные параметры безопасности.

Добавляем в конец:

    net.inet.tcp.blackhole=2        # ядро убивает tcp пакеты, приходящие в систему на непрослушиваемые порты
    net.inet.udp.blackhole=0        # как и выше, только не убивает ибо traceroute пакеты не покажут этот хоп
    net.inet.icmp.drop_redirect=1   # не обращаем внимания на icmp redirect
    net.inet.icmp.log_redirect=0    # и не логируем их
    net.inet.ip.redirect=0          # не реагируем на icmp redirect
