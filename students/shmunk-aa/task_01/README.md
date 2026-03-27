
## Параметры подключения к серверам

### Для части 2

#### server_a:

- `ssh -i ~/uvuv -p 22021 root@80.249.144.74`

#### server_b:

- `ssh -i ~/uvuv -p 22022 root@80.249.144.74`

#### server_c:

- `ssh -i ~/uvuv -p 22023 root@80.249.144.74`

### Для части 3

#### server_a:

- `ssh -i ~/uvuv -p22024 root@80.249.144.74`

#### server_b:

- `ssh -i ~/uvuv -p 22025 root@80.249.144.74`

#### server_c:

- `ssh -i ~/uvuv -p 22026 root@80.249.144.74`

# Часть 2.
## Топология

Все узлы связаны между собой, физически(можем пропинговать любой с любого по ip докера) они подключены к одному "свичу", но при помощи ip адресов мы разделяем их на две сети. Узлы А и С напрямую не подключены, но узел В подключен и к А и к С. Поэтому, несмотря на то, что они все соеденины, будет выглядеть как A-B-C и необходимо настроить маршрутизациб от A к С и наоборот.

## Прописываем ip:

### server_a:

- `ip addr add 192.168.1.2/24 dev eth0`

### server_b:

- `ip addr add 192.168.1.1/24 dev eth0`
- `ip addr add 192.168.2.1/24 dev eth0`

### server_c:

- `ip addr add 192.168.2.2/24 dev eth0`

маршрутизация на каждом узле была включена по умолчанию, включать отдельно не требуется.

## Прописываем маршруты

### server_a:

- `ip route add 192.168.2.0/24 via 192.168.1.1 dev eth0`

### server_c:

- `ip route add 192.168.1.0/24 via 192.168.2.1 dev eth0`

## Дампы трафика

### server_a(192.168.1.2)

```text
root@7da58b2c31c6:~# tcpdump -i eth0 -n icmp
tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
listening on eth0, link-type EN10MB (Ethernet), snapshot length 262144 bytes
13:58:58.828968 IP 192.168.2.2 > 192.168.1.2: ICMP echo request, id 21, seq 1, length 64
13:58:58.829000 IP 192.168.1.2 > 192.168.2.2: ICMP echo reply, id 21, seq 1, length 64
13:58:58.829015 IP 192.168.1.1 > 192.168.1.2: ICMP redirect 192.168.2.2 to host 192.168.2.2, length 92
13:58:59.841540 IP 192.168.2.2 > 192.168.1.2: ICMP echo request, id 21, seq 2, length 64
13:58:59.841575 IP 192.168.1.2 > 192.168.2.2: ICMP echo reply, id 21, seq 2, length 64
13:58:59.841586 IP 192.168.1.1 > 192.168.1.2: ICMP redirect 192.168.2.2 to host 192.168.2.2, length 92
```

### server_c(192.168.2.2)

```text
root@5156e6819095:~# tcpdump -i eth0 -n icmp
tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
listening on eth0, link-type EN10MB (Ethernet), snapshot length 262144 bytes
13:59:35.921580 IP 192.168.1.2 > 192.168.2.2: ICMP echo request, id 22, seq 1, length 64
13:59:35.921601 IP 192.168.2.2 > 192.168.1.2: ICMP echo reply, id 22, seq 1, length 64
13:59:35.921612 IP 192.168.2.1 > 192.168.2.2: ICMP redirect 192.168.1.2 to host 192.168.1.2, length 92
13:59:36.929539 IP 192.168.1.2 > 192.168.2.2: ICMP echo request, id 22, seq 2, length 64
13:59:36.929564 IP 192.168.2.2 > 192.168.1.2: ICMP echo reply, id 22, seq 2, length 64
13:59:36.929598 IP 192.168.2.1 > 192.168.2.2: ICMP redirect 192.168.1.2 to host 192.168.1.2, length 92
13:59:37.953573 IP 192.168.1.2 > 192.168.2.2: ICMP echo request, id 22, seq 3, length 64
13:59:37.953598 IP 192.168.2.2 > 192.168.1.2: ICMP echo reply, id 22, seq 3, length 64
13:59:37.953607 IP 192.168.2.1 > 192.168.2.2: ICMP redirect 192.168.1.2 to host 192.168.1.2, length 92
```

# Часть 3.
## Топология

Здесь уже нельзя с любого узла пропинговать другой, уже физически выстроено, что A-B-C и необходимо аналогично прописать маршруты. У узла А и узла С по одному интерфейсу, у узла В их два.
## Прописываем ip:

### server_a:

- `ip addr add 192.168.1.2/24 dev eth0`

### server_b:

- `ip addr add 192.168.1.1/24 dev eth0`
- `ip addr add 192.168.2.1/24 dev eth1`

### server_c:

- `ip addr add 192.168.2.2/24 dev eth0`

## Прописываем маршруты

### server_a:

- `ip route add 192.168.2.0/24 via 192.168.1.1 dev eth0`

### server_c:

- `ip route add 192.168.1.0/24 via 192.168.2.1 dev eth0`

## Дампы трафика

### server_a(192.168.1.2)

```text
root@1eaf68af1567:~# tcpdump -i eth0 -n icmp
tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
listening on eth0, link-type EN10MB (Ethernet), snapshot length 262144 bytes
14:30:51.787756 IP 192.168.2.2 > 192.168.1.2: ICMP echo request, id 42, seq 1, length 64
14:30:51.787800 IP 192.168.1.2 > 192.168.2.2: ICMP echo reply, id 42, seq 1, length 64
14:30:52.801533 IP 192.168.2.2 > 192.168.1.2: ICMP echo request, id 42, seq 2, length 64
14:30:52.801552 IP 192.168.1.2 > 192.168.2.2: ICMP echo reply, id 42, seq 2, length 64
14:30:53.825520 IP 192.168.2.2 > 192.168.1.2: ICMP echo request, id 42, seq 3, length 64
14:30:53.825539 IP 192.168.1.2 > 192.168.2.2: ICMP echo reply, id 42, seq 3, length 64
14:30:54.849526 IP 192.168.2.2 > 192.168.1.2: ICMP echo request, id 42, seq 4, length 64
14:30:54.849549 IP 192.168.1.2 > 192.168.2.2: ICMP echo reply, id 42, seq 4, length 64
```

### server_c(192.168.2.2)

```text
root@aadec5ac33f6:~# tcpdump -i eth0 -n icmp
tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
listening on eth0, link-type EN10MB (Ethernet), snapshot length 262144 bytes
14:31:44.368202 IP 192.168.1.2 > 192.168.2.2: ICMP echo request, id 43, seq 1, length 64
14:31:44.368246 IP 192.168.2.2 > 192.168.1.2: ICMP echo reply, id 43, seq 1, length 64
14:31:45.377526 IP 192.168.1.2 > 192.168.2.2: ICMP echo request, id 43, seq 2, length 64
14:31:45.377545 IP 192.168.2.2 > 192.168.1.2: ICMP echo reply, id 43, seq 2, length 64
14:31:46.401504 IP 192.168.1.2 > 192.168.2.2: ICMP echo request, id 43, seq 3, length 64
14:31:46.401523 IP 192.168.2.2 > 192.168.1.2: ICMP echo reply, id 43, seq 3, length 64
```