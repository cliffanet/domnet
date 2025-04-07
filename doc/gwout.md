# Установка зарубежного сервера (FreeBSD)

Скачиваем образ с сайта [freebsd](https://download.freebsd.org/ftp/releases/amd64/amd64/ISO-IMAGES/).

Например:

    FreeBSD-12.3-RELEASE-amd64-disc1.iso

Устанавливаем пакеты:

```bash
pkg install bash mc bird2 git net-snmp p5-JSON-XS p5-LWP-Protocol-https bind-tools sqlite3
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
fetch -o /tmp/mc.tar https://github.com/cliffanet/domnet/raw/refs/heads/master/doc/mc.tar
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

Либо через `/etc/crontab`

    */30    *       *       *       *       root    /usr/sbin/ntpdate -u ntp.northnet.ru 192.168.50.100 192.168.50.50 > /dev/null

Либо через `ntpd`

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

Либо через `cron` + `ntpdate`

* В файле `/etc/crontab`

    ```
    */20    *       *       *       *       root    /usr/sbin/ntpdate ntp.northnet.ru 192.168.50.100 192.168.50.50 2>&1 >> /var/log/ntpdate.log
    ```

## Файрвол (NAT)

Файл `/etc/pf.conf`

    set block-policy drop
    set limit src-nodes 1000000
    set limit states 1000000
    set limit frags 200000
    set optimization aggressive

    scrub fragment reassemble
    nat on bce0 from !bce0 to any -> (bce0)

Имя интерфейса `bce0` заменяем на верное - основной сетевой интерфейс сервера.

## bird


Файл `/usr/local/etc/bird.conf`

    log "/var/log/bird.log" all;

    protocol device { }

    protocol kernel {
        learn;
        ipv4 {                # Connect protocol to IPv4 table by channel
            import all;       # Import to table, default is import all
            export all;       # Export to protocol. default is export none
        };
    }

    protocol static {
        ipv4;                               # Again, IPv4 channel with default options
        include "/home/routes.conf";

        # cliff
        route 192.168.3.0/24 via 172.16.1.2;
    }

    template bgp bgp_peer {
        local as 65000;
        #direct;
        ipv4 {
            import none;
            export filter {
                if proto = "static1" && ifname = "bce0" then
                    accept;
                else
                    reject;
            };
        };
    }

    protocol bgp bgp_cliff from bgp_peer { neighbor 172.16.1.2 as 65003; }


## /etc/rc.conf

    # Set dumpdev to "AUTO" to enable crash dumps, "NO" to disable
    dumpdev="AUTO"
    hostname="manager.example.com"
    keymap="ru.kbd"

    defaultrouter="XXX.XXX.XXX.XXX"
    ifconfig_bce0="inet XXX.XXX.XXX.XXX/24"
    
    tcp_extensions="NO"
    tcp_drop_synfin="YES"
    icmp_drop_redirect="YES"
    icmp_log_redirect="YES"
    sendmail_enable="NONE"
    sendmail_submit_enable="NO"
    sendmail_outbound_enable="NO"
    sendmail_msp_queue_enable="NO"

    sshd_enable="YES"
    ntpd_enable="YES"

    pf_enable="YES"
    pf_rules="/etc/pf.conf"
    pf_program="/sbin/pfctl"

    bird_enable="YES"
    snmpd_enable="YES"

## /etc/sysctl.conf

Некоторые дополнительные параметры безопасности.

Добавляем в конец:

    net.inet.tcp.blackhole=2        # ядро убивает tcp пакеты, приходящие в систему на непрослушиваемые порты
    net.inet.udp.blackhole=0        # как и выше, только не убивает ибо traceroute пакеты не покажут этот хоп
    net.inet.icmp.drop_redirect=1   # не обращаем внимания на icmp redirect
    net.inet.icmp.log_redirect=0    # и не логируем их
    net.inet.ip.redirect=0          # не реагируем на icmp redirect

    net.inet.ip.forwarding=1

## /usr/local/share/snmp/snmpd.conf

    syslocation  outdoor
    syscontact   manager
    rocommunity  ROCOMMUNITY

## Скрипт, обновляющий таблицу маршрутизации

```bash
cd /home
git clone https://github.com/cliffanet/domnet.git
rm -rf /home/domnet/.git
```


## Шифрование канала

```bash
pkg install ipsec-tools
```

Файл `/etc/rc.conf`

    ipsec_enable="YES"
    ipsec_program="/usr/local/sbin/setkey"
    ipsec_file="/usr/local/etc/racoon/setkey.conf"
    racoon_enable="YES"
    racoon_flags="-l /var/log/racoon.log"

Файл `/usr/local/etc/racoon/racoon.conf`

    path include "/usr/local/etc/racoon";
    path pre_shared_key "/usr/local/etc/racoon/psk.txt";

    listen {
        isakmp XXX.XXX.XXX.XXX [500];
        isakmp_natt XXX.XXX.XXX.XXX [4500];
    }

    timer
    {
        # To keep the NAT-mappings on your NAT gateway, there must be
        # traffic between the peers. Normally the UDP-Encap traffic
        # (i.e. the real data transported over the tunnel) would be
        # enough, but to be safe racoon will send a short
        # "Keep-alive packet" every few seconds to every peer with
        # whom it does NAT-Traversal.
        # The default is 20s. Set it to 0s to disable sending completely.
        natt_keepalive 10 sec;
    }

    remote anonymous {
        exchange_mode main;
        proposal_check obey;
        support_proxy on;
        nat_traversal on;
        #generate_policy on;
        ike_frag on;
        dpd_delay 20;

        proposal {
            encryption_algorithm aes;
            hash_algorithm sha1;
            authentication_method pre_shared_key;
            dh_group modp1024;
        }

        proposal {
            encryption_algorithm aes;
            hash_algorithm sha1;
            authentication_method pre_shared_key;
            dh_group modp2048;
        }

        proposal {
            encryption_algorithm des;
            hash_algorithm sha1;
            authentication_method pre_shared_key;
            dh_group modp1024;
        }
    }

    sainfo  anonymous {
        pfs_group                       2;
        lifetime                        time 1 hour;
        encryption_algorithm            aes;
        authentication_algorithm        hmac_sha1;
        compression_algorithm           deflate;
    }

Файл `/usr/local/etc/racoon/psk.txt`

    * mypassword

Файл `/usr/local/etc/racoon/setkey.conf`

    flush;
    spdflush;
    spdadd 0.0.0.0/0[0] 0.0.0.0/0[1701] udp -P in ipsec esp/transport//require;
    spdadd 0.0.0.0/0[1701] 0.0.0.0/0[0] udp -P out ipsec esp/transport//require;
    spdadd YY.YY.YY.YY/32 XXX.XXX.XXX.XXX/32 ipencap -P in ipsec esp/transport/YY.YY.YY.YY-XXX.XXX.XXX.XXX/require;
    spdadd XXX.XXX.XXX.XXX/32 YY.YY.YY.YY/32 ipencap -P out ipsec esp/transport/XXX.XXX.XXX.XXX-YY.YY.YY.YY/require;

    # YY.YY.YY.YY - локальный опорный роутер (peer для данного сервера)

## mpd

> Чтобы работал l2tp, необходимо настроить всё из раздела [Шифрование канала](#шифрование-канала)

```bash
pkg install mpd5
```

Файл `/etc/rc.conf`

    mpd_enable="YES"

Файл `/usr/local/etc/mpd5/mpd.conf`

    startup:
        log +PHYS2
        # configure the console
        set console self 127.0.0.1 5005
        set console open


    default:
        load l2tp_server

    l2tp_server:
        set ippool add pool1 192.168.101.101 192.168.101.150

        create bundle template B
        set iface idle 1800
        set iface enable proxy-arp
        set iface enable tcpmssfix
        set ipcp yes vjcomp
        set ipcp ranges 192.168.101.1/32 ippool pool1
        set ipcp dns 8.8.8.8

        set bundle enable compression
        set ccp yes mppc
        set mppc yes e40
        set mppc yes e128
        set mppc yes stateless

        create link template L l2tp
        set link action bundle B
        set link enable multilink
        set link yes acfcomp protocomp
        set link no pap chap eap
        set link enable chap
        set link enable chap-msv2
        set link keep-alive 10 60
        set link mtu 1460
        set l2tp self 0.0.0.0
        set l2tp disable dataseq
        set link enable incoming

Файл `/usr/local/etc/mpd5/mpd.secret`

    user    superpassword

