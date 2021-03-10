# Сетевые пакеты. VLAN'ы. LACP
Строим бонды и вланы
в Office1 в тестовой подсети появляется сервера с доп интерфесами и адресами
в internal сети testLAN
- testClient1 - 10.10.10.254
- testClient2 - 10.10.10.254
- testServer1- 10.10.10.1
- testServer2- 10.10.10.1

равести вланами
testClient1 <-> testServer1
testClient2 <-> testServer2

между centralRouter и inetRouter
"пробросить" 2 линка (общая inernal сеть) и объединить их в бонд
проверить работу c отключением интерфейсов

Проверим наши VLAN'ы 
# testServer1
```ruby
[vagrant@testServer1 ~]$ ip --brief link show
lo               UNKNOWN        00:00:00:00:00:00 <LOOPBACK,UP,LOWER_UP>
eth0             UP             52:54:00:4d:77:d3 <BROADCAST,MULTICAST,UP,LOWER_UP>
eth1             UP             08:00:27:30:db:f6 <BROADCAST,MULTICAST,UP,LOWER_UP>
eth2             UP             08:00:27:e7:b5:0c <BROADCAST,MULTICAST,UP,LOWER_UP>
eth1.10@eth1     UP             08:00:27:30:db:f6 <BROADCAST,MULTICAST,UP,LOWER_UP>
[vagrant@testServer1 ~]$ ip --brief addr show
lo               UNKNOWN        127.0.0.1/8 ::1/128
eth0             UP             10.0.2.15/24 fe80::5054:ff:fe4d:77d3/64
eth1             UP             192.168.5.2/24 fe80::a00:27ff:fe30:dbf6/64
eth2             UP             10.10.11.3/24 fe80::a00:27ff:fee7:b50c/64
eth1.10@eth1     UP             10.10.10.1/24 fe80::a00:27ff:fe30:dbf6/64
[vagrant@testServer1 ~]$ ping 10.10.10.254
PING 10.10.10.254 (10.10.10.254) 56(84) bytes of data.
64 bytes from 10.10.10.254: icmp_seq=1 ttl=64 time=0.654 ms
64 bytes from 10.10.10.254: icmp_seq=2 ttl=64 time=0.390 ms
^C
--- 10.10.10.254 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 999ms
rtt min/avg/max/mdev = 0.390/0.522/0.654/0.132 ms
```
# testClient1
```ruby
[vagrant@testClient1 ~]$ ip --brief link show
lo               UNKNOWN        00:00:00:00:00:00 <LOOPBACK,UP,LOWER_UP>
eth0             UP             52:54:00:4d:77:d3 <BROADCAST,MULTICAST,UP,LOWER_UP>
eth1             UP             08:00:27:a2:8d:41 <BROADCAST,MULTICAST,UP,LOWER_UP>
eth2             UP             08:00:27:b5:64:45 <BROADCAST,MULTICAST,UP,LOWER_UP>
eth1.10@eth1     UP             08:00:27:a2:8d:41 <BROADCAST,MULTICAST,UP,LOWER_UP>
[vagrant@testClient1 ~]$ ip --brief addr show
lo               UNKNOWN        127.0.0.1/8 ::1/128
eth0             UP             10.0.2.15/24 fe80::5054:ff:fe4d:77d3/64
eth1             UP             192.168.5.3/24 fe80::a00:27ff:fea2:8d41/64
eth2             UP             10.10.11.4/24 fe80::a00:27ff:feb5:6445/64
eth1.10@eth1     UP             10.10.10.254/24 fe80::a00:27ff:fea2:8d41/64
[vagrant@testClient1 ~]$ ping 10.10.10.1
PING 10.10.10.1 (10.10.10.1) 56(84) bytes of data.
64 bytes from 10.10.10.1: icmp_seq=1 ttl=64 time=0.429 ms
64 bytes from 10.10.10.1: icmp_seq=2 ttl=64 time=0.345 ms
^C
--- 10.10.10.1 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 999ms
rtt min/avg/max/mdev = 0.345/0.387/0.429/0.042 ms
```
# testServer2
```ruby
[vagrant@testServer2 ~]$ ip --brief link show
lo               UNKNOWN        00:00:00:00:00:00 <LOOPBACK,UP,LOWER_UP>
eth0             UP             52:54:00:4d:77:d3 <BROADCAST,MULTICAST,UP,LOWER_UP>
eth1             UP             08:00:27:dd:b8:ca <BROADCAST,MULTICAST,UP,LOWER_UP>
eth2             UP             08:00:27:05:36:1a <BROADCAST,MULTICAST,UP,LOWER_UP>
eth1.20@eth1     UP             08:00:27:dd:b8:ca <BROADCAST,MULTICAST,UP,LOWER_UP>
[vagrant@testServer2 ~]$ ip --brief addr show
lo               UNKNOWN        127.0.0.1/8 ::1/128
eth0             UP             10.0.2.15/24 fe80::5054:ff:fe4d:77d3/64
eth1             UP             192.168.5.4/24 fe80::a00:27ff:fedd:b8ca/64
eth2             UP             10.10.11.5/24 fe80::a00:27ff:fe05:361a/64
eth1.20@eth1     UP             10.10.10.1/24 fe80::a00:27ff:fedd:b8ca/64
[vagrant@testServer2 ~]$ ping 10.10.10.254
PING 10.10.10.254 (10.10.10.254) 56(84) bytes of data.
64 bytes from 10.10.10.254: icmp_seq=1 ttl=64 time=0.722 ms
64 bytes from 10.10.10.254: icmp_seq=2 ttl=64 time=0.459 ms
^C
--- 10.10.10.254 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1000ms
rtt min/avg/max/mdev = 0.459/0.590/0.722/0.133 ms
```
# testClient2
```ruby
[vagrant@testClient2 ~]$ ip --brief link show
lo               UNKNOWN        00:00:00:00:00:00 <LOOPBACK,UP,LOWER_UP>
eth0             UP             52:54:00:4d:77:d3 <BROADCAST,MULTICAST,UP,LOWER_UP>
eth1             UP             08:00:27:77:ff:ce <BROADCAST,MULTICAST,UP,LOWER_UP>
eth2             UP             08:00:27:d3:16:29 <BROADCAST,MULTICAST,UP,LOWER_UP>
eth1.20@eth1     UP             08:00:27:77:ff:ce <BROADCAST,MULTICAST,UP,LOWER_UP>
[vagrant@testClient2 ~]$ ip --brief addr show
lo               UNKNOWN        127.0.0.1/8 ::1/128
eth0             UP             10.0.2.15/24
eth1             UP             192.168.5.5/24 fe80::a00:27ff:fe77:ffce/64
eth2             UP             10.10.11.6/24 fe80::a00:27ff:fed3:1629/64
eth1.20@eth1     UP             10.10.10.254/24 fe80::a00:27ff:fe77:ffce/64
[vagrant@testClient2 ~]$ ping 10.10.10.1
PING 10.10.10.1 (10.10.10.1) 56(84) bytes of data.
64 bytes from 10.10.10.1: icmp_seq=1 ttl=64 time=0.403 ms
64 bytes from 10.10.10.1: icmp_seq=2 ttl=64 time=0.468 ms
^C
--- 10.10.10.1 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1000ms
rtt min/avg/max/mdev = 0.403/0.435/0.468/0.038 ms
```
# Проверим bonding между роутерами 
inetRouter
```ruby
[root@inetRouter vagrant]# cat /proc/net/bonding/bond0
Ethernet Channel Bonding Driver: v3.7.1 (April 27, 2011)

Bonding Mode: fault-tolerance (active-backup) (fail_over_mac active)
Primary Slave: None
Currently Active Slave: eth1
MII Status: up
MII Polling Interval (ms): 100
Up Delay (ms): 0
Down Delay (ms): 0

Slave Interface: eth1
MII Status: up
Speed: 1000 Mbps
Duplex: full
Link Failure Count: 0
Permanent HW addr: 08:00:27:7e:8f:a5
Slave queue ID: 0

Slave Interface: eth2
MII Status: up
Speed: 1000 Mbps
Duplex: full
Link Failure Count: 0
Permanent HW addr: 08:00:27:eb:e6:d6
Slave queue ID: 0
[root@inetRouter vagrant]# cat /etc/sysconfig/network-scripts/ifcfg-bond0
DEVICE=bond0
NAME=bond0
TYPE=Bond
BONDING_MASTER=yes
IPADDR=10.10.1.1
NETMASK=255.255.255.252
ONBOOT=yes
USERCTL=no
BOOTPROTO=static
BONDING_OPTS="mode=1 miimon=100 fail_over_mac=1"
ZONE=internal
```
centralRouter
```ruby
[root@centralRouter vagrant]# cat /proc/net/bonding/bond0
Ethernet Channel Bonding Driver: v3.7.1 (April 27, 2011)

Bonding Mode: fault-tolerance (active-backup) (fail_over_mac active)
Primary Slave: None
Currently Active Slave: eth1
MII Status: up
MII Polling Interval (ms): 100
Up Delay (ms): 0
Down Delay (ms): 0

Slave Interface: eth1
MII Status: up
Speed: 1000 Mbps
Duplex: full
Link Failure Count: 0
Permanent HW addr: 08:00:27:e9:ed:d3
Slave queue ID: 0

Slave Interface: eth2
MII Status: up
Speed: 1000 Mbps
Duplex: full
Link Failure Count: 0
Permanent HW addr: 08:00:27:26:c9:db
Slave queue ID: 0
[root@centralRouter vagrant]# cat /etc/sysconfig/network-scripts/ifcfg-bond0
DEVICE=bond0
NAME=bond0
TYPE=Bond
BONDING_MASTER=yes
IPADDR=10.10.1.2
NETMASK=255.255.255.252
ONBOOT=yes
USERCTL=no
BOOTPROTO=static
BONDING_OPTS="mode=1 miimon=100 fail_over_mac=1"
GATEWAY=10.10.1.1
```
как видно, что активным интерфесом на обоих роутерах выбран eth1, для удобства монитринга запустим ping в текстовый файлик. И выключим eth1 на inetRouter.
```ruby
[root@inetRouter vagrant]# ping 10.10.1.2 >ping_test.txt
```
выключим eth1 на inetRouter
```ruby
[root@inetRouter vagrant]# ip link set eth1 down
[root@inetRouter vagrant]# cat /proc/net/bonding/bond0
Ethernet Channel Bonding Driver: v3.7.1 (April 27, 2011)

Bonding Mode: fault-tolerance (active-backup) (fail_over_mac active)
Primary Slave: None
Currently Active Slave: eth2
MII Status: up
MII Polling Interval (ms): 100
Up Delay (ms): 0
Down Delay (ms): 0

Slave Interface: eth1
MII Status: down
Speed: 1000 Mbps
Duplex: full
Link Failure Count: 1
Permanent HW addr: 08:00:27:7e:8f:a5
Slave queue ID: 0

Slave Interface: eth2
MII Status: up
Speed: 1000 Mbps
Duplex: full
Link Failure Count: 0
Permanent HW addr: 08:00:27:eb:e6:d6
Slave queue ID: 0
```
проверим наш файлик

```ruby
[root@inetRouter vagrant]# cat ping_test.txt
PING 10.10.1.2 (10.10.1.2) 56(84) bytes of data.
64 bytes from 10.10.1.2: icmp_seq=1 ttl=64 time=0.436 ms
64 bytes from 10.10.1.2: icmp_seq=2 ttl=64 time=0.455 ms
64 bytes from 10.10.1.2: icmp_seq=3 ttl=64 time=0.457 ms
64 bytes from 10.10.1.2: icmp_seq=4 ttl=64 time=0.475 ms
64 bytes from 10.10.1.2: icmp_seq=5 ttl=64 time=0.438 ms
64 bytes from 10.10.1.2: icmp_seq=6 ttl=64 time=0.431 ms
64 bytes from 10.10.1.2: icmp_seq=7 ttl=64 time=0.431 ms
64 bytes from 10.10.1.2: icmp_seq=8 ttl=64 time=0.435 ms
64 bytes from 10.10.1.2: icmp_seq=9 ttl=64 time=0.460 ms
64 bytes from 10.10.1.2: icmp_seq=10 ttl=64 time=0.454 ms

--- 10.10.1.2 ping statistics ---
10 packets transmitted, 10 received, 0% packet loss, time 9003ms
rtt min/avg/max/mdev = 0.431/0.447/0.475/0.019 ms
```
как видно, что после выключения eth1 активным стал eth2 и пинги при этом не прекратились


