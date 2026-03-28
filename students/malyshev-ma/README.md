# Часть 2

## Контейнеры

Буквами A,B,C будут назначены три узла

B - центральный

1-ый контейнер (A)

```sh
root@A:~# ip a
...
2: eth0@if166: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
    link/ether 6e:67:c1:eb:23:a9 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 192.168.208.2/20 brd 192.168.223.255 scope global eth0
       valid_lft forever preferred_lft forever
```

2-ой контейнер (B)

```sh
root@B:~# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: eth0@if168: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
    link/ether e2:62:40:4e:d4:cb brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 192.168.208.3/20 brd 192.168.223.255 scope global eth0
       valid_lft forever preferred_lft forever
```

3-ий контейнер (C)

```sh
root@C:~# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: eth0@if171: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
    link/ether 16:d1:70:fb:c4:5c brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 192.168.208.4/20 brd 192.168.223.255 scope global eth0
       valid_lft forever preferred_lft forever
```

## Топология

Физическая (`docker`): A, B и C в одной `bridge`-сети `docker`

Логическая (будет строиться в дальнейших пунктах этой части): 
- A: `192.168.1.1`
- B: `192.168.1.2` + `192.168.2.1`
- C: `192.168.2.2`

`A-B-C`

## Взаимный пинг

```sh
root@A:~# ping -c3 192.168.208.3
PING 192.168.208.3 (192.168.208.3) 56(84) bytes of data.
64 bytes from 192.168.208.3: icmp_seq=1 ttl=64 time=0.129 ms
64 bytes from 192.168.208.3: icmp_seq=2 ttl=64 time=0.089 ms
64 bytes from 192.168.208.3: icmp_seq=3 ttl=64 time=0.089 ms

--- 192.168.208.3 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2054ms
rtt min/avg/max/mdev = 0.089/0.102/0.129/0.018 ms
root@A:~# ping -c3 192.168.208.4
PING 192.168.208.4 (192.168.208.4) 56(84) bytes of data.
64 bytes from 192.168.208.4: icmp_seq=1 ttl=64 time=0.088 ms
64 bytes from 192.168.208.4: icmp_seq=2 ttl=64 time=0.091 ms
64 bytes from 192.168.208.4: icmp_seq=3 ttl=64 time=0.093 ms

--- 192.168.208.4 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2030ms
rtt min/avg/max/mdev = 0.088/0.090/0.093/0.002 ms
```

```sh
root@B:~# ping -c3 192.168.208.2
PING 192.168.208.2 (192.168.208.2) 56(84) bytes of data.
64 bytes from 192.168.208.2: icmp_seq=1 ttl=64 time=0.049 ms
64 bytes from 192.168.208.2: icmp_seq=2 ttl=64 time=0.087 ms
64 bytes from 192.168.208.2: icmp_seq=3 ttl=64 time=0.075 ms

--- 192.168.208.2 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2028ms
rtt min/avg/max/mdev = 0.049/0.070/0.087/0.015 ms
root@B:~# ping -c3 192.168.208.4
PING 192.168.208.4 (192.168.208.4) 56(84) bytes of data.
64 bytes from 192.168.208.4: icmp_seq=1 ttl=64 time=0.156 ms
64 bytes from 192.168.208.4: icmp_seq=2 ttl=64 time=0.094 ms
64 bytes from 192.168.208.4: icmp_seq=3 ttl=64 time=0.199 ms

--- 192.168.208.4 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2044ms
rtt min/avg/max/mdev = 0.094/0.149/0.199/0.043 ms
```

```sh
root@C:~# ping -c3 192.168.208.2
PING 192.168.208.2 (192.168.208.2) 56(84) bytes of data.
64 bytes from 192.168.208.2: icmp_seq=1 ttl=64 time=0.068 ms
64 bytes from 192.168.208.2: icmp_seq=2 ttl=64 time=0.096 ms
64 bytes from 192.168.208.2: icmp_seq=3 ttl=64 time=0.093 ms

--- 192.168.208.2 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2039ms
rtt min/avg/max/mdev = 0.068/0.085/0.096/0.012 ms
root@C:~# ping -c3 192.168.208.3
PING 192.168.208.3 (192.168.208.3) 56(84) bytes of data.
64 bytes from 192.168.208.3: icmp_seq=1 ttl=64 time=0.076 ms
64 bytes from 192.168.208.3: icmp_seq=2 ttl=64 time=0.073 ms
64 bytes from 192.168.208.3: icmp_seq=3 ttl=64 time=0.087 ms

--- 192.168.208.3 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2046ms
rtt min/avg/max/mdev = 0.073/0.078/0.087/0.006 ms
```

## Назначение новых IP-адресов

A:

```sh
root@A:~# ip addr add 192.168.1.1/24 dev eth0
root@A:~# ip a
...
2: eth0@if166: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
    link/ether 6e:67:c1:eb:23:a9 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 192.168.208.2/20 brd 192.168.223.255 scope global eth0
       valid_lft forever preferred_lft forever
    inet 192.168.1.1/24 scope global eth0
       valid_lft forever preferred_lft forever
```

B:

```sh
root@B:~# ip addr add 192.168.1.2/24 dev eth0
ip addr add 192.168.2.1/24 dev eth0
root@B:~# ip a
...
2: eth0@if168: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
    link/ether e2:62:40:4e:d4:cb brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 192.168.208.3/20 brd 192.168.223.255 scope global eth0
       valid_lft forever preferred_lft forever
    inet 192.168.1.2/24 scope global eth0
       valid_lft forever preferred_lft forever
    inet 192.168.2.1/24 scope global eth0
       valid_lft forever preferred_lft forever
```

C:

```sh
root@C:~# ip addr add 192.168.2.2/24 dev eth0
root@C:~# ip a
...
2: eth0@if171: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
    link/ether 16:d1:70:fb:c4:5c brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 192.168.208.4/20 brd 192.168.223.255 scope global eth0
       valid_lft forever preferred_lft forever
    inet 192.168.2.2/24 scope global eth0
       valid_lft forever preferred_lft forever
```

Адреса назначены

## Настройка IP-forwarding

B:

```sh
root@B:~# echo 1 > /proc/sys/net/ipv4/ip_forward
-bash: /proc/sys/net/ipv4/ip_forward: Read-only file system
root@B:~# cat /proc/sys/net/ipv4/ip_forward
1
```

## Настройка таблиц маршрутизации

Попробуем с A пропинговать B через сеть `192.168.2.0/24`:

```sh
root@A:~# ping -c3 192.168.2.1
PING 192.168.2.1 (192.168.2.1) 56(84) bytes of data.
^C
--- 192.168.2.1 ping statistics ---
3 packets transmitted, 0 received, 100% packet loss, time 2046ms
```

Почему не выходит? Рассмотрим таблицу маршрутизации:

```sh
root@A:~# ip r
default via 192.168.208.1 dev eth0
192.168.1.0/24 dev eth0 proto kernel scope link src 192.168.1.1
192.168.208.0/20 dev eth0 proto kernel scope link src 192.168.208.2
```

Пакет уйдёт на `default gateway`: `192.168.208.1` - `docker gateway`, у которого нет информации о сети `192.168.2.0/24`

Поэтому нужно настроить стандартный маршрут для `192.168.2.0/24` так, чтобы он шёл через узел `192.168.1.2` (адрес B в сети `192.168.1.0/24`):

```sh
root@A:~# ip route add 192.168.2.0/24 via 192.168.1.2
root@A:~# ip r
default via 192.168.208.1 dev eth0
192.168.1.0/24 dev eth0 proto kernel scope link src 192.168.1.1
192.168.2.0/24 via 192.168.1.2 dev eth0
192.168.208.0/20 dev eth0 proto kernel scope link src 192.168.208.2
```

Попытаемся пропинговать теперь:

```sh
root@A:~# ping -c3 192.168.2.1
PING 192.168.2.1 (192.168.2.1) 56(84) bytes of data.
64 bytes from 192.168.2.1: icmp_seq=1 ttl=64 time=0.087 ms
64 bytes from 192.168.2.1: icmp_seq=2 ttl=64 time=0.087 ms
64 bytes from 192.168.2.1: icmp_seq=3 ttl=64 time=0.090 ms

--- 192.168.2.1 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2034ms
rtt min/avg/max/mdev = 0.087/0.088/0.090/0.001 ms
root@A:~# ping -c3 192.168.2.2
PING 192.168.2.2 (192.168.2.2) 56(84) bytes of data.
From 192.168.1.2 icmp_seq=2 Redirect Host(New nexthop: 192.168.2.2)
From 192.168.1.2 icmp_seq=3 Redirect Host(New nexthop: 192.168.2.2)

--- 192.168.2.2 ping statistics ---
3 packets transmitted, 0 received, +2 errors, 100% packet loss, time 2033ms
```

B отвечает, C - нет. Почему? В таблице маршрутизации C нет информации о том, куда отправить пакет из сети `192.168.1.0/24`:

```sh
root@C:~# ip r
default via 192.168.208.1 dev eth0
192.168.2.0/24 dev eth0 proto kernel scope link src 192.168.2.2
192.168.208.0/20 dev eth0 proto kernel scope link src 192.168.208.4
```

Это надо также добавить:

```sh
root@C:~# ip route add 192.168.1.0/24 via 192.168.2.1
root@C:~# ip r
default via 192.168.208.1 dev eth0
192.168.1.0/24 via 192.168.2.1 dev eth0
192.168.2.0/24 dev eth0 proto kernel scope link src 192.168.2.2
192.168.208.0/20 dev eth0 proto kernel scope link src 192.168.208.4
```

Проверим пинг с A:

```sh
root@A:~# ping -c3 192.168.2.2
PING 192.168.2.2 (192.168.2.2) 56(84) bytes of data.
64 bytes from 192.168.2.2: icmp_seq=1 ttl=63 time=0.099 ms
64 bytes from 192.168.2.2: icmp_seq=2 ttl=63 time=0.077 ms
From 192.168.1.2 icmp_seq=3 Redirect Host(New nexthop: 192.168.2.2)

--- 192.168.2.2 ping statistics ---
3 packets transmitted, 2 received, +1 errors, 33.3333% packet loss, time 2052ms
rtt min/avg/max/mdev = 0.077/0.088/0.099/0.011 ms
```

Ошибка с `Redirect Host...` появляется из-за `ICMP Redirect`

Проверим пинг B и A, отправляя с C:

```sh
root@C:~# ping -c3 192.168.1.2
PING 192.168.1.2 (192.168.1.2) 56(84) bytes of data.
64 bytes from 192.168.1.2: icmp_seq=1 ttl=64 time=0.066 ms
64 bytes from 192.168.1.2: icmp_seq=2 ttl=64 time=0.069 ms
64 bytes from 192.168.1.2: icmp_seq=3 ttl=64 time=0.155 ms

--- 192.168.1.2 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2048ms
rtt min/avg/max/mdev = 0.066/0.096/0.155/0.041 ms
root@C:~# ping -c3 192.168.1.1
PING 192.168.1.1 (192.168.1.1) 56(84) bytes of data.
64 bytes from 192.168.1.1: icmp_seq=1 ttl=63 time=0.094 ms
From 192.168.2.1 icmp_seq=2 Redirect Host(New nexthop: 192.168.1.1)
64 bytes from 192.168.1.1: icmp_seq=2 ttl=63 time=0.123 ms

--- 192.168.1.1 ping statistics ---
2 packets transmitted, 2 received, +1 errors, 0% packet loss, time 1026ms
rtt min/avg/max/mdev = 0.094/0.108/0.123/0.014 ms
```

Получаем то же самое

## Анализ трафика через `tcpdump` на узле B

Слушаем на B:

```sh
root@B:~# tcpdump -i eth0 icmp
tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
listening on eth0, link-type EN10MB (Ethernet), snapshot length 262144 bytes
16:39:39.581159 IP 192.168.1.1 > 192.168.2.2: ICMP echo request, id 139, seq 1, length 64
16:39:39.581201 IP 192.168.1.1 > 192.168.2.2: ICMP echo request, id 139, seq 1, length 64
16:39:39.581221 IP 192.168.2.2 > 192.168.1.1: ICMP echo reply, id 139, seq 1, length 64
16:39:39.581227 IP 192.168.2.1 > 192.168.2.2: ICMP redirect 192.168.1.1 to host 192.168.1.1, length 92
16:39:39.581228 IP 192.168.2.2 > 192.168.1.1: ICMP echo reply, id 139, seq 1, length 64
16:39:40.609462 IP 192.168.1.1 > 192.168.2.2: ICMP echo request, id 139, seq 2, length 64
16:39:40.609541 IP 192.168.1.2 > 192.168.1.1: ICMP redirect 192.168.2.2 to host 192.168.2.2, length 92
16:39:40.609546 IP 192.168.1.1 > 192.168.2.2: ICMP echo request, id 139, seq 2, length 64
16:39:40.609575 IP 192.168.2.2 > 192.168.1.1: ICMP echo reply, id 139, seq 2, length 64
16:39:40.609582 IP 192.168.2.1 > 192.168.2.2: ICMP redirect 192.168.1.1 to host 192.168.1.1, length 92
16:39:40.609583 IP 192.168.2.2 > 192.168.1.1: ICMP echo reply, id 139, seq 2, length 64
^C
11 packets captured
11 packets received by filter
0 packets dropped by kernel
```

Отправляем с A на C:

```sh
root@A:~# ping -c3 192.168.2.2
PING 192.168.2.2 (192.168.2.2) 56(84) bytes of data.
64 bytes from 192.168.2.2: icmp_seq=1 ttl=63 time=0.106 ms
From 192.168.1.2 icmp_seq=2 Redirect Host(New nexthop: 192.168.2.2)
64 bytes from 192.168.2.2: icmp_seq=2 ttl=63 time=0.175 ms

--- 192.168.2.2 ping statistics ---
2 packets transmitted, 2 received, +1 errors, 0% packet loss, time 1028ms
rtt min/avg/max/mdev = 0.106/0.140/0.175/0.034 ms
```
