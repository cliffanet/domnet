# Добавление локального опорного на стороне зарубежного сервера

1. Добавить gif-интерфейс.

    Для интерфейса выделяем свободную подсеть в 172.16.1.0 с маской /30.

    - XXX.XXX.XXX.XXX - внешний IP зарубежного сервера,
    - YYY.YYY.YYY.YYY - внешний IP локального опорника.

    Файл `/etc/rc.conf`

        gif_interfaces="gif0"
        # NAME
        gifconfig_gif0="XXX.XXX.XXX.XXX YYY.YYY.YYY.YYY"
        ifconfig_gif0="inet 172.16.1.X 172.16.1.Y netmask 255.255.255.252"

2. Дописать static-route в bird на локальные маршруты.

    В файле `/usr/local/etc/bird.conf`

        protocol static {
            ipv4;                               # Again, IPv4 channel with default options
            include "/home/routes.conf";

        >>  # NAME
        >>  route 192.168.XXX.0/24 via 172.16.1.Y;
        }

3. Добавить BGP-сессию.

    В файле `/usr/local/etc/bird.conf`

        protocol bgp bgp_NAME from bgp_peer { neighbor 172.16.1.Y as 6500Y; }

4. Шифрование.

    Общий ключ - в файле /usr/local/etc/racoon/psk.txt

    В файле `/usr/local/etc/racoon/setkey.conf`

        spdadd YYY.YYY.YYY.YYY/32 XXX.XXX.XXX.XXX/32 ipencap -P in ipsec esp/transport/YYY.YYY.YYY.YYY-XXX.XXX.XXX.XXX/require;
        spdadd XXX.XXX.XXX.XXX/32 YYY.YYY.YYY.YYY/32 ipencap -P out ipsec esp/transport/XXX.XXX.XXX.XXX-YYY.YYY.YYY.YYY/require;

    - XXX.XXX.XXX.XXX - внешний IP зарубежного сервера,
    - YYY.YYY.YYY.YYY - внешний IP локального опорника.

