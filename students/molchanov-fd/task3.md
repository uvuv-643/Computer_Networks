Молчанов Фёдор Денисович P3313 413015
# task3
1. В моем понимании, топология на исходном этапе, это три компьютера, подключенные к одной локальной сети 192.168.112.0/20:
	1. 192.168.112.3/20
	2. 192.168.112.2/20
	3. 192.168.112.4/20
	При этом, есть отдельный свич, который раутит запросы, так как иначе из за одного интерфейса невозможно было бы пересылать запросы от каждого к каждому.

2. Проверка доступности пингом:
	```bash
	root@cf795886e41a:~#ping 192.168.112.4
	PING 192.168.112.4 (192.168.112.4) 56(84) bytes of data.
	64 bytes from 192.168.112.4: icmp_seq=1 ttl=64 time=0.244 ms
	64 bytes from 192.168.112.4: icmp_seq=2 ttl=64 time=0.071 ms
	^C
	--- 192.168.112.4 ping statistics ---
	2 packets transmitted, 2 received, 0% packet loss, time 1024ms
	rtt min/avg/max/mdev = 0.071/0.157/0.244/0.086 ms
	```
   Устройства в сети доступны (пингую с первого сервера третий. Адрес первого 192.168.112.3)

3. использую команду `ip addr add <new_ip_address>/<prefix_length> dev eth0` для добавления ip на eth0 для каждого из серверов,
	на первом: 192.168.1.1
	на втором: 192.168.1.2, 192.168.2.2
	на третьем: 192.168.2.3
	(везде маска /24, очевидно)
	На данный моментмогу пинговать: 1→2, 3→2, 2→оба оставшихся.

4. Добавил запись в первый комп и проверил, что могу пинговать 3 сервер:
	```bash
	root@cf795886e41a:~# ip route add 192.168.2.0/24 via 192.168.1.2
	root@cf795886e41a:~# ip route
	default via 192.168.112.1 dev eth0
	192.168.1.0/24 dev eth0 proto kernel scope link src 192.168.1.1
	192.168.2.0/24 via 192.168.1.2 dev eth0
	192.168.112.0/20 dev eth0 proto kernel scope link src 192.168.112.3
	root@cf795886e41a:~# ping 192.168.2.3
	PING 192.168.2.3 (192.168.2.3) 56(84) bytes of data.
	From 192.168.1.2 icmp_seq=2 Redirect Host(New nexthop: 192.168.2.3)
	From 192.168.1.2 icmp_seq=3 Redirect Host(New nexthop: 192.168.2.3)
	From 192.168.1.2 icmp_seq=4 Redirect Host(New nexthop: 192.168.2.3)
	From 192.168.1.2 icmp_seq=5 Redirect Host(New nexthop: 192.168.2.3)
	^C
	--- 192.168.2.3 ping statistics ---
	5 packets transmitted, 0 received, +4 errors, 100% packet loss, time 4102ms
	
	```
	то же самое сделал с 3 сервером:
	```bash
	root@0e956318da51:~# ip route add 192.168.1.0/24 via 192.168.2.2
	root@0e956318da51:~# ping 192.168.1.1
	PING 192.168.1.1 (192.168.1.1) 56(84) bytes of data.
	64 bytes from 192.168.1.1: icmp_seq=1 ttl=63 time=0.098 ms
	From 192.168.2.2 icmp_seq=2 Redirect Host(New nexthop: 192.168.1.1)
	64 bytes from 192.168.1.1: icmp_seq=2 ttl=63 time=0.102 ms
	From 192.168.2.2 icmp_seq=3 Redirect Host(New nexthop: 192.168.1.1)
	64 bytes from 192.168.1.1: icmp_seq=3 ttl=63 time=0.113 ms
	^C
	--- 192.168.1.1 ping statistics ---
	3 packets transmitted, 3 received, +2 errors, 0% packet loss, time 2050ms
	rtt min/avg/max/mdev = 0.098/0.104/0.113/0.006 ms
	```
5. в предыдущем пункте указано, что оставшиеся два хоста (1 и 3) теперь могут общаться друг с другом
6. не придумал ничего лучше, кроме как проверять данные по айпи (смотрел, пришли ли пакеты с третьего на первый на первом хосте):
	```bash
		root@cf795886e41a:~# tshark -Y 'ip.addr == 192.168.2.3'
	Running as user "root" and group "root". This could be dangerous.
	Capturing on 'eth0'
	 ** (tshark:816) 17:59:27.570366 [Main MESSAGE] -- Capture started.
	 ** (tshark:816) 17:59:27.570466 [Main MESSAGE] -- File: "/tmp/wireshark_eth03UN5M3.pcapng"
	    4 22.517635307  192.168.2.3 ? 192.168.1.1  ICMP 98 Echo (ping) request  id=0x0066, seq=1/256, ttl=63
	    5 22.517654128  192.168.1.1 ? 192.168.2.3  ICMP 98 Echo (ping) reply    id=0x0066, seq=1/256, ttl=64 (request in 4)
	    6 22.517664396  192.168.1.2 ? 192.168.1.1  ICMP 126 Redirect             (Redirect for host)
	   13 23.534933690  192.168.2.3 ? 192.168.1.1  ICMP 98 Echo (ping) request  id=0x0066, seq=2/512, ttl=63
	   14 23.534958864  192.168.1.1 ? 192.168.2.3  ICMP 98 Echo (ping) reply    id=0x0066, seq=2/512, ttl=64 (request in 13)
	   15 23.534967398  192.168.1.2 ? 192.168.1.1  ICMP 126 Redirect             (Redirect for host)
	   18 24.558888170  192.168.2.3 ? 192.168.1.1  ICMP 98 Echo (ping) request  id=0x0066, seq=3/768, ttl=63
	   19 24.558912800  192.168.1.1 ? 192.168.2.3  ICMP 98 Echo (ping) reply    id=0x0066, seq=3/768, ttl=64 (request in 18)
	   20 24.558921520  192.168.1.2 ? 192.168.1.1  ICMP 126 Redirect             (Redirect for host)
	   25 25.582988100  192.168.2.3 ? 192.168.1.1  ICMP 98 Echo (ping) request  id=0x0066, seq=4/1024, ttl=63
	   26 25.583017020  192.168.1.1 ? 192.168.2.3  ICMP 98 Echo (ping) reply    id=0x0066, seq=4/1024, ttl=64 (request in 25)
	   27 25.583026500  192.168.1.2 ? 192.168.1.1  ICMP 126 Redirect             (Redirect for host)
	```
	на третьем хосте мне лень было качать tshark, так что сделал ту же проверку, только через tcpdump:
	```bash
	root@0e956318da51:~# tcpdump src host 192.168.1.1
	tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
	listening on eth0, link-type EN10MB (Ethernet), snapshot length 262144 bytes
	18:06:22.697555 IP 192.168.1.1 > 192.168.2.3: ICMP echo request, id 103, seq 1, length 64
	18:06:23.713493 IP 192.168.1.1 > 192.168.2.3: ICMP echo request, id 103, seq 2, length 64
	18:06:24.737535 IP 192.168.1.1 > 192.168.2.3: ICMP echo request, id 103, seq 3, length 64
	18:06:25.761578 IP 192.168.1.1 > 192.168.2.3: ICMP echo request, id 103, seq 4, length 64
	```
# task 2
1. 1 и 3 компы с одним интерфейсом, 2 комп с двумя:
	1. 192.168.128.3/20 
	2. 192.168.128.2/20(eth0), 192.168.144.3/20(eth1)
	3. 192.168.144.2/20
2. Могу пинговать 1 <-> 2, и 3 отдельно (никого не может и его не могут, т.к. он в отдельной сети)