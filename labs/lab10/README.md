# Протокол BGP (iBGP)

## Цели

1. Настроить iBGP в офисе Москва
1. Настроить iBGP в сети провайдера Триада
1. Настроить полную IP связанность всех сетей

## Задание

1. Настроить iBGP в офисе Москва между маршрутизаторами R13 и R14, приоритетным провайдером должен быть Ламас.
1. Настроить iBGP в провайдере Триада, с использованием Route Reflector.
1. Настроить офис Санкт-Петербург так, чтобы трафик до любого офиса распределялся по двум линкам одновременно.
1. Настроить полную IP связность всех сетей.

## Топология

![a](media/lab10_1.PNG)

## Схема для импорта в PNETlab

[Схема для импорта в PNETlab](media/otus_cource_lab10_iBGP_pnetlab_export-20250125-185611.zip)

## Версии ПО

- PNETlab - 5.3.11
- Роутеры - Cisco IOS Software, Linux Software (I86BI_LINUX-ADVENTERPRISEK9-M), Version 15.4(2)T4
- Коммутаторы - Cisco IOS Software, Linux Software (I86BI_LINUXL2-ADVENTERPRISEK9-M), Version 15.2(CML_NIGHTLY_20150703)
- ПК - VPC

## Решение

1. Настроить iBGP в офисе Москва между маршрутизаторами R13 и R14, приоритетным провайдером должен быть Ламас.

      Пример настройки iBGP между маршрутизаторами R13 и R14 приведен в разделе "Конфигурации устройств".  
      Приоритетность провайдера Ламас обеспечивается на роутере R13 путем выставления на полученных от R21 (AS 101) маршрутов local preference 50, что делает данные маршруты менее приоритетными чем, маршруты полученные от R14, который в свою очередь получает маршруты от R31 (AS 301).


1.  Настроить iBGP в провайдере Триада, с использованием Route Reflector.

      Пример настройки iBGP в провайдере Триада приведен в разделе "Конфигурации устройств".  
      В качестве Route Reflector'а выбран роутер R41.  
      Настроен listen range для диапазона 10.40.0.0/24 (подсеть для loopback адресов в Триада), все кто попадает в этот диапазон объявляются Route Reflector клиентами.  
      Используется peer-группа.

1. Настроить офис Санкт-Петербург так, чтобы трафик до любого офиса распределялся по двум линкам одновременно.

      Пример настройки роутера R52 приведен в разделе "Конфигурации устройств".  
      Распределение трафика (load sharing) на R52 по двум линкам к провайдеру Триада обеспечивается командой maximum-paths 2 в настройках роутера BGP, по умолчанию устанавливается только один маршрут до подсети.

1. Настроить полную IP связность всех сетей.

      Пример настроек приведен в разделе "Конфигурации устройств".  
      Для обеспечения полной IP связности нужно настроить редистрибуцию маршрутов. 


## Проверка работоспособности

Проверка будет осуществляться с маршрутизатора R52 в Санкт-Петербурге.

1. Проверим, что в таблице маршрутизации есть BGP маршруты до подсетей 172.16.1.0/24 (роутер R13 в Москве) и 172.16.2.0/24 (роутер R14 в Москве)

```
R52#show ip bgp
BGP table version is 10, local router ID is 10.50.0.52
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter,
              x best-external, a additional-path, c RIB-compressed,
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 *>  172.16.1.0/24    172.16.6.44                            0 520 301 101 i
 *>  172.16.2.0/24    172.16.6.44                            0 520 301 i
 *>  172.16.3.0/24    172.16.6.44                            0 520 301 i
 *>  172.16.4.0/24    172.16.6.44              0             0 520 i
 *>  172.16.5.0/24    172.16.6.44                            0 520 301 101 i
 *>  172.16.6.0/24    0.0.0.0                  0         32768 i
 *                    172.16.6.44              0             0 520 i
 *   172.16.7.0/24    172.16.7.43              0             0 520 i
 *>                   0.0.0.0                  0         32768 i
 *>  172.16.8.0/24    172.16.7.43              0             0 520 i


R52#sho ip route bgp
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
       a - application route
       + - replicated route, % - next hop override

Gateway of last resort is not set

      172.16.0.0/16 is variably subnetted, 10 subnets, 2 masks
B        172.16.1.0/24 [20/0] via 172.16.6.44, 00:18:09
B        172.16.2.0/24 [20/0] via 172.16.6.44, 00:18:09
B        172.16.3.0/24 [20/0] via 172.16.6.44, 00:18:09
B        172.16.4.0/24 [20/0] via 172.16.6.44, 00:18:09
B        172.16.5.0/24 [20/0] via 172.16.6.44, 00:18:09
B        172.16.8.0/24 [20/0] via 172.16.7.43, 00:15:52
```
Видим, что маршруты до подсетей 172.16.1.0/24 (роутер R13 в Москве) и 172.16.2.0/24 (роутер R14 в Москве) присутствуют на R52. Маршрут до 172.16.1.0/24 пролегает через автономные системы 520 301 101, а маршрут 172.16.2.0/24 через 520 301. 

1. Проверим IP связность между R52 и R12 (172.16.1.21), R52 и R13 (ping 172.16.2.14).

```
R52#ping 172.16.1.21
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 172.16.1.21, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/1 ms

R52#ping 172.16.2.14
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 172.16.2.14, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/1 ms
```
Видим, что IP связность есть, цель задания достигнута.

## Конфигурации устройств

### R13

<details>
  <summary>Конфигурация</summary>

```

R13# terminal length 0
R13#sh run
Building configuration...

Current configuration : 1430 bytes
!
! Last configuration change at 18:15:20 UTC Sat Jan 18 2025
!
version 15.4
service timestamps debug datetime msec
service timestamps log datetime msec
no service password-encryption
!
hostname R13
!
boot-start-marker
boot-end-marker
!
!
!
no aaa new-model
mmi polling-interval 60
no mmi auto-configure
no mmi pvc
mmi snmp-timeout 180
!
!
!
!
!
!
!
!


!
!
!
!
no ip domain lookup
ip cef
no ipv6 cef
!
multilink bundle-name authenticated
!
!
!
!
!
!
!
!
!
redundancy
!
!
!
!
!
!
!
!
!
!
!
!
!
!
!
interface Loopback1
 ip address 10.10.0.13 255.255.255.255
 ip ospf 1 area 0
!
interface Ethernet0/0
 ip address 10.10.3.13 255.255.255.0
 ip ospf 1 area 0
!
interface Ethernet0/1
 ip address 10.10.7.13 255.255.255.0
 ip ospf 1 area 0
!
interface Ethernet0/2
 ip address 172.16.1.13 255.255.255.0
!
interface Ethernet0/3
 ip address 10.10.4.13 255.255.255.0
 ip ospf 1 area 101
!
router ospf 1
 router-id 10.10.0.13
 area 101 filter-list prefix OSPF-FILTER in
 default-information originate
!
router bgp 1001
 bgp log-neighbor-changes
 network 172.16.1.0 mask 255.255.255.0
 neighbor 172.16.1.21 remote-as 101
!
ip forward-protocol nd
!
!
no ip http server
no ip http secure-server
!
!
ip prefix-list OSPF-FILTER seq 10 deny 10.10.0.0/16 le 32
ip prefix-list OSPF-FILTER seq 20 permit 0.0.0.0/0 le 32
!
!
!
control-plane
!
!
!
!
!
!
!
!
line con 0
 logging synchronous
line aux 0
line vty 0 4
 login
 transport input none
!
!
end

R13#
R13#
R13#
R13#show ip interface brief
Interface                  IP-Address      OK? Method Status                Protocol
Ethernet0/0                10.10.3.13      YES TFTP   up                    up
Ethernet0/1                10.10.7.13      YES TFTP   up                    up
Ethernet0/2                172.16.1.13     YES TFTP   up                    up
Ethernet0/3                10.10.4.13      YES TFTP   up                    up
Loopback1                  10.10.0.13      YES TFTP   up                    up
R13#
R13#
R13#
R13#
R13#show ip route
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
       a - application route
       + - replicated route, % - next hop override

Gateway of last resort is not set

      10.0.0.0/8 is variably subnetted, 19 subnets, 2 masks
O IA     10.10.0.11/32 [110/11] via 10.10.3.11, 3w6d, Ethernet0/0
O IA     10.10.0.12/32 [110/11] via 10.10.7.12, 3w6d, Ethernet0/1
C        10.10.0.13/32 is directly connected, Loopback1
O        10.10.0.14/32 [110/21] via 10.10.7.12, 3w6d, Ethernet0/1
                       [110/21] via 10.10.3.11, 3w6d, Ethernet0/0
O        10.10.0.15/32 [110/11] via 10.10.4.15, 3w6d, Ethernet0/3
O IA     10.10.0.16/32 [110/31] via 10.10.7.12, 3w6d, Ethernet0/1
                       [110/31] via 10.10.3.11, 3w6d, Ethernet0/0
O IA     10.10.1.0/24 [110/20] via 10.10.7.12, 3w6d, Ethernet0/1
                      [110/20] via 10.10.3.11, 3w6d, Ethernet0/0
O IA     10.10.2.0/24 [110/20] via 10.10.7.12, 3w6d, Ethernet0/1
                      [110/20] via 10.10.3.11, 3w6d, Ethernet0/0
C        10.10.3.0/24 is directly connected, Ethernet0/0
L        10.10.3.13/32 is directly connected, Ethernet0/0
C        10.10.4.0/24 is directly connected, Ethernet0/3
L        10.10.4.13/32 is directly connected, Ethernet0/3
O        10.10.5.0/24 [110/20] via 10.10.7.12, 3w6d, Ethernet0/1
O IA     10.10.6.0/24 [110/30] via 10.10.7.12, 3w6d, Ethernet0/1
                      [110/30] via 10.10.3.11, 3w6d, Ethernet0/0
C        10.10.7.0/24 is directly connected, Ethernet0/1
L        10.10.7.13/32 is directly connected, Ethernet0/1
O        10.10.8.0/24 [110/20] via 10.10.3.11, 3w6d, Ethernet0/0
O IA     10.10.9.0/24 [110/20] via 10.10.7.12, 3w6d, Ethernet0/1
                      [110/20] via 10.10.3.11, 3w6d, Ethernet0/0
O IA     10.10.255.0/24 [110/20] via 10.10.7.12, 3w6d, Ethernet0/1
                        [110/20] via 10.10.3.11, 3w6d, Ethernet0/0
      172.16.0.0/16 is variably subnetted, 8 subnets, 2 masks
C        172.16.1.0/24 is directly connected, Ethernet0/2
L        172.16.1.13/32 is directly connected, Ethernet0/2
B        172.16.2.0/24 [20/0] via 172.16.1.21, 00:51:04
B        172.16.3.0/24 [20/0] via 172.16.1.21, 00:52:42
B        172.16.4.0/24 [20/0] via 172.16.1.21, 00:51:04
B        172.16.5.0/24 [20/0] via 172.16.1.21, 00:49:46
B        172.16.6.0/24 [20/0] via 172.16.1.21, 00:38:53
B        172.16.7.0/24 [20/0] via 172.16.1.21, 00:33:35
R13#
R13#
R13#
R13#
R13#show ip bgp
BGP table version is 8, local router ID is 10.10.0.13
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter,
              x best-external, a additional-path, c RIB-compressed,
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 *   172.16.1.0/24    172.16.1.21              0             0 101 i
 *>                   0.0.0.0                  0         32768 i
 *>  172.16.2.0/24    172.16.1.21                            0 101 301 i
 *>  172.16.3.0/24    172.16.1.21              0             0 101 i
 *>  172.16.4.0/24    172.16.1.21                            0 101 301 i
 *>  172.16.5.0/24    172.16.1.21              0             0 101 i
 *>  172.16.6.0/24    172.16.1.21                            0 101 301 520 i
 *>  172.16.7.0/24    172.16.1.21                            0 101 301 520 2042 i
R13#
R13#
R13#
R13#
R13#show ip bgp summary
BGP router identifier 10.10.0.13, local AS number 1001
BGP table version is 8, main routing table version 8
7 network entries using 980 bytes of memory
8 path entries using 640 bytes of memory
5/5 BGP path/bestpath attribute entries using 720 bytes of memory
4 BGP AS-PATH entries using 112 bytes of memory
0 BGP route-map cache entries using 0 bytes of memory
0 BGP filter-list cache entries using 0 bytes of memory
BGP using 2452 total bytes of memory
BGP activity 7/0 prefixes, 8/0 paths, scan interval 60 secs

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
172.16.1.21     4          101      68      65        8    0    0 00:55:07        7


```
</details>

### R14

<details>
  <summary>Конфигурация</summary>

```

R14#terminal length 0
R14#sh run
Building configuration...

Current configuration : 1424 bytes
!
! Last configuration change at 18:09:16 UTC Sat Jan 18 2025
!
version 15.4
service timestamps debug datetime msec
service timestamps log datetime msec
no service password-encryption
!
hostname R14
!
boot-start-marker
boot-end-marker
!
!
!
no aaa new-model
mmi polling-interval 60
no mmi auto-configure
no mmi pvc
mmi snmp-timeout 180
!
!
!
!
!
!
!
!


!
!
!
!
no ip domain lookup
ip cef
no ipv6 cef
!
multilink bundle-name authenticated
!
!
!
!
!
!
!
!
!
redundancy
!
!
!
!
!
!
!
!
!
!
!
!
!
!
!
interface Loopback1
 ip address 10.10.0.14 255.255.255.255
 ip ospf 1 area 0
!
interface Ethernet0/0
 ip address 10.10.5.14 255.255.255.0
 ip ospf 1 area 0
!
interface Ethernet0/1
 ip address 10.10.8.14 255.255.255.0
 ip ospf 1 area 0
!
interface Ethernet0/2
 ip address 172.16.2.14 255.255.255.0
!
interface Ethernet0/3
 ip address 10.10.6.14 255.255.255.0
 ip ospf 1 area 102
!
router ospf 1
 router-id 10.10.0.14
 area 102 filter-list prefix OSPF-FILTER in
 default-information originate
!
router bgp 1001
 bgp log-neighbor-changes
 network 172.16.2.0 mask 255.255.255.0
 neighbor 172.16.2.31 remote-as 301
!
ip forward-protocol nd
!
!
no ip http server
no ip http secure-server
!
!
ip prefix-list OSPF-FILTER seq 10 deny 10.10.4.0/24
ip prefix-list OSPF-FILTER seq 20 permit 0.0.0.0/0 le 32
!
!
!
control-plane
!
!
!
!
!
!
!
!
line con 0
 logging synchronous
line aux 0
line vty 0 4
 login
 transport input none
!
!
end

R14#
R14#
R14#
R14#show ip interface brief
Interface                  IP-Address      OK? Method Status                Protocol
Ethernet0/0                10.10.5.14      YES TFTP   up                    up
Ethernet0/1                10.10.8.14      YES TFTP   up                    up
Ethernet0/2                172.16.2.14     YES TFTP   up                    up
Ethernet0/3                10.10.6.14      YES TFTP   up                    up
Loopback1                  10.10.0.14      YES TFTP   up                    up
R14#
R14#
R14#
R14#
R14#show ip route
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
       a - application route
       + - replicated route, % - next hop override

Gateway of last resort is not set

      10.0.0.0/8 is variably subnetted, 19 subnets, 2 masks
O IA     10.10.0.11/32 [110/11] via 10.10.8.11, 3w6d, Ethernet0/1
O IA     10.10.0.12/32 [110/11] via 10.10.5.12, 3w6d, Ethernet0/0
O        10.10.0.13/32 [110/21] via 10.10.8.11, 3w6d, Ethernet0/1
                       [110/21] via 10.10.5.12, 3w6d, Ethernet0/0
C        10.10.0.14/32 is directly connected, Loopback1
O IA     10.10.0.15/32 [110/31] via 10.10.8.11, 3w6d, Ethernet0/1
                       [110/31] via 10.10.5.12, 3w6d, Ethernet0/0
O        10.10.0.16/32 [110/11] via 10.10.6.16, 3w6d, Ethernet0/3
O IA     10.10.1.0/24 [110/20] via 10.10.8.11, 3w6d, Ethernet0/1
                      [110/20] via 10.10.5.12, 3w6d, Ethernet0/0
O IA     10.10.2.0/24 [110/20] via 10.10.8.11, 3w6d, Ethernet0/1
                      [110/20] via 10.10.5.12, 3w6d, Ethernet0/0
O        10.10.3.0/24 [110/20] via 10.10.8.11, 3w6d, Ethernet0/1
O IA     10.10.4.0/24 [110/30] via 10.10.8.11, 3w6d, Ethernet0/1
                      [110/30] via 10.10.5.12, 3w6d, Ethernet0/0
C        10.10.5.0/24 is directly connected, Ethernet0/0
L        10.10.5.14/32 is directly connected, Ethernet0/0
C        10.10.6.0/24 is directly connected, Ethernet0/3
L        10.10.6.14/32 is directly connected, Ethernet0/3
O        10.10.7.0/24 [110/20] via 10.10.5.12, 3w6d, Ethernet0/0
C        10.10.8.0/24 is directly connected, Ethernet0/1
L        10.10.8.14/32 is directly connected, Ethernet0/1
O IA     10.10.9.0/24 [110/20] via 10.10.8.11, 3w6d, Ethernet0/1
                      [110/20] via 10.10.5.12, 3w6d, Ethernet0/0
O IA     10.10.255.0/24 [110/20] via 10.10.8.11, 3w6d, Ethernet0/1
                        [110/20] via 10.10.5.12, 3w6d, Ethernet0/0
      172.16.0.0/16 is variably subnetted, 8 subnets, 2 masks
B        172.16.1.0/24 [20/0] via 172.16.2.31, 00:43:41
C        172.16.2.0/24 is directly connected, Ethernet0/2
L        172.16.2.14/32 is directly connected, Ethernet0/2
B        172.16.3.0/24 [20/0] via 172.16.2.31, 00:43:41
B        172.16.4.0/24 [20/0] via 172.16.2.31, 00:43:41
B        172.16.5.0/24 [20/0] via 172.16.2.31, 00:43:41
B        172.16.6.0/24 [20/0] via 172.16.2.31, 00:38:20
B        172.16.7.0/24 [20/0] via 172.16.2.31, 00:33:02
R14#
R14#
R14#
R14#
R14#show ip bgp
BGP table version is 8, local router ID is 10.10.0.14
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter,
              x best-external, a additional-path, c RIB-compressed,
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 *>  172.16.1.0/24    172.16.2.31                            0 301 101 i
 *   172.16.2.0/24    172.16.2.31              0             0 301 i
 *>                   0.0.0.0                  0         32768 i
 *>  172.16.3.0/24    172.16.2.31              0             0 301 i
 *>  172.16.4.0/24    172.16.2.31              0             0 301 i
 *>  172.16.5.0/24    172.16.2.31                            0 301 101 i
 *>  172.16.6.0/24    172.16.2.31                            0 301 520 i
 *>  172.16.7.0/24    172.16.2.31                            0 301 520 2042 i
R14#
R14#
R14#
R14#
R14#show ip bgp summary
BGP router identifier 10.10.0.14, local AS number 1001
BGP table version is 8, main routing table version 8
7 network entries using 980 bytes of memory
8 path entries using 640 bytes of memory
5/5 BGP path/bestpath attribute entries using 720 bytes of memory
4 BGP AS-PATH entries using 96 bytes of memory
0 BGP route-map cache entries using 0 bytes of memory
0 BGP filter-list cache entries using 0 bytes of memory
BGP using 2436 total bytes of memory
BGP activity 7/0 prefixes, 8/0 paths, scan interval 60 secs

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
172.16.2.31     4          301      55      52        8    0    0 00:43:41        7


```
</details>

### R21

<details>
  <summary>Конфигурация</summary>

```

R21#terminal length 0
R21#sh run
Building configuration...

Current configuration : 1462 bytes
!
! Last configuration change at 18:30:40 UTC Sat Jan 18 2025
!
version 15.4
service timestamps debug datetime msec
service timestamps log datetime msec
no service password-encryption
!
hostname R21
!
boot-start-marker
boot-end-marker
!
!
!
no aaa new-model
mmi polling-interval 60
no mmi auto-configure
no mmi pvc
mmi snmp-timeout 180
!
!
!
!
!
!
!
!


!
!
!
!
no ip domain lookup
ip cef
no ipv6 cef
!
multilink bundle-name authenticated
!
!
!
!
!
!
!
!
!
redundancy
!
!
!
!
!
!
!
!
!
!
!
!
!
!
!
interface Loopback1
 ip address 10.20.0.21 255.255.255.255
!
interface Ethernet0/0
 ip address 172.16.1.21 255.255.255.0
!
interface Ethernet0/1
 ip address 172.16.3.21 255.255.255.0
!
interface Ethernet0/2
 ip address 172.16.5.21 255.255.255.0
!
interface Ethernet0/3
 no ip address
 shutdown
!
interface Ethernet1/0
 no ip address
 shutdown
!
interface Ethernet1/1
 no ip address
 shutdown
!
interface Ethernet1/2
 no ip address
 shutdown
!
interface Ethernet1/3
 no ip address
 shutdown
!
router bgp 101
 bgp log-neighbor-changes
 network 172.16.1.0 mask 255.255.255.0
 network 172.16.3.0 mask 255.255.255.0
 network 172.16.5.0 mask 255.255.255.0
 neighbor 172.16.1.13 remote-as 1001
 neighbor 172.16.3.31 remote-as 301
 neighbor 172.16.5.41 remote-as 520
!
ip forward-protocol nd
!
!
no ip http server
no ip http secure-server
!
!
!
!
control-plane
!
!
!
!
!
!
!
!
line con 0
 logging synchronous
line aux 0
line vty 0 4
 login
 transport input none
!
!
end

R21#
R21#
R21#
R21#show ip interface brief
Interface                  IP-Address      OK? Method Status                Protocol
Ethernet0/0                172.16.1.21     YES TFTP   up                    up
Ethernet0/1                172.16.3.21     YES TFTP   up                    up
Ethernet0/2                172.16.5.21     YES manual up                    up
Ethernet0/3                unassigned      YES TFTP   administratively down down
Ethernet1/0                unassigned      YES TFTP   administratively down down
Ethernet1/1                unassigned      YES TFTP   administratively down down
Ethernet1/2                unassigned      YES TFTP   administratively down down
Ethernet1/3                unassigned      YES TFTP   administratively down down
Loopback1                  10.20.0.21      YES TFTP   up                    up
R21#
R21#
R21#
R21#
R21#show ip route
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
       a - application route
       + - replicated route, % - next hop override

Gateway of last resort is not set

      10.0.0.0/32 is subnetted, 1 subnets
C        10.20.0.21 is directly connected, Loopback1
      172.16.0.0/16 is variably subnetted, 10 subnets, 2 masks
C        172.16.1.0/24 is directly connected, Ethernet0/0
L        172.16.1.21/32 is directly connected, Ethernet0/0
B        172.16.2.0/24 [20/0] via 172.16.3.31, 00:49:43
C        172.16.3.0/24 is directly connected, Ethernet0/1
L        172.16.3.21/32 is directly connected, Ethernet0/1
B        172.16.4.0/24 [20/0] via 172.16.3.31, 00:49:43
C        172.16.5.0/24 is directly connected, Ethernet0/2
L        172.16.5.21/32 is directly connected, Ethernet0/2
B        172.16.6.0/24 [20/0] via 172.16.3.31, 00:37:32
B        172.16.7.0/24 [20/0] via 172.16.3.31, 00:32:14
R21#
R21#
R21#
R21#
R21#show ip bgp
BGP table version is 9, local router ID is 10.20.0.21
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter,
              x best-external, a additional-path, c RIB-compressed,
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 *>  172.16.1.0/24    0.0.0.0                  0         32768 i
 *                    172.16.1.13              0             0 1001 i
 *>  172.16.2.0/24    172.16.3.31              0             0 301 i
 *   172.16.3.0/24    172.16.3.31              0             0 301 i
 *>                   0.0.0.0                  0         32768 i
 *>  172.16.4.0/24    172.16.3.31              0             0 301 i
 *   172.16.5.0/24    172.16.5.41              0             0 520 i
 *>                   0.0.0.0                  0         32768 i
 *>  172.16.6.0/24    172.16.3.31                            0 301 520 i
 *>  172.16.7.0/24    172.16.3.31                            0 301 520 2042 i
R21#
R21#
R21#
R21#
R21#show ip bgp summary
BGP router identifier 10.20.0.21, local AS number 101
BGP table version is 9, main routing table version 9
7 network entries using 980 bytes of memory
10 path entries using 800 bytes of memory
6/4 BGP path/bestpath attribute entries using 864 bytes of memory
5 BGP AS-PATH entries using 120 bytes of memory
0 BGP route-map cache entries using 0 bytes of memory
0 BGP filter-list cache entries using 0 bytes of memory
BGP using 2764 total bytes of memory
BGP activity 7/0 prefixes, 10/0 paths, scan interval 60 secs

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
172.16.1.13     4         1001      63      67        9    0    0 00:53:45        1
172.16.3.31     4          301      60      62        9    0    0 00:50:12        5
172.16.5.41     4          520      33      37        9    0    0 00:26:33        1


```
</details>

### R31

<details>
  <summary>Конфигурация</summary>

```

R31#terminal length 0
R31#sh run
Building configuration...

Current configuration : 1462 bytes
!
! Last configuration change at 18:23:56 UTC Sat Jan 18 2025
!
version 15.4
service timestamps debug datetime msec
service timestamps log datetime msec
no service password-encryption
!
hostname R31
!
boot-start-marker
boot-end-marker
!
!
!
no aaa new-model
mmi polling-interval 60
no mmi auto-configure
no mmi pvc
mmi snmp-timeout 180
!
!
!
!
!
!
!
!


!
!
!
!
no ip domain lookup
ip cef
no ipv6 cef
!
multilink bundle-name authenticated
!
!
!
!
!
!
!
!
!
redundancy
!
!
!
!
!
!
!
!
!
!
!
!
!
!
!
interface Loopback1
 ip address 10.30.0.31 255.255.255.255
!
interface Ethernet0/0
 ip address 172.16.2.31 255.255.255.0
!
interface Ethernet0/1
 ip address 172.16.3.31 255.255.255.0
!
interface Ethernet0/2
 ip address 172.16.4.31 255.255.255.0
!
interface Ethernet0/3
 no ip address
 shutdown
!
interface Ethernet1/0
 no ip address
 shutdown
!
interface Ethernet1/1
 no ip address
 shutdown
!
interface Ethernet1/2
 no ip address
 shutdown
!
interface Ethernet1/3
 no ip address
 shutdown
!
router bgp 301
 bgp log-neighbor-changes
 network 172.16.2.0 mask 255.255.255.0
 network 172.16.3.0 mask 255.255.255.0
 network 172.16.4.0 mask 255.255.255.0
 neighbor 172.16.2.14 remote-as 1001
 neighbor 172.16.3.21 remote-as 101
 neighbor 172.16.4.44 remote-as 520
!
ip forward-protocol nd
!
!
no ip http server
no ip http secure-server
!
!
!
!
control-plane
!
!
!
!
!
!
!
!
line con 0
 logging synchronous
line aux 0
line vty 0 4
 login
 transport input none
!
!
end

R31#
R31#
R31#
R31#show ip interface brief
Interface                  IP-Address      OK? Method Status                Protocol
Ethernet0/0                172.16.2.31     YES TFTP   up                    up
Ethernet0/1                172.16.3.31     YES TFTP   up                    up
Ethernet0/2                172.16.4.31     YES TFTP   up                    up
Ethernet0/3                unassigned      YES TFTP   administratively down down
Ethernet1/0                unassigned      YES TFTP   administratively down down
Ethernet1/1                unassigned      YES TFTP   administratively down down
Ethernet1/2                unassigned      YES TFTP   administratively down down
Ethernet1/3                unassigned      YES TFTP   administratively down down
Loopback1                  10.30.0.31      YES TFTP   up                    up
R31#
R31#
R31#
R31#
R31#show ip route
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
       a - application route
       + - replicated route, % - next hop override

Gateway of last resort is not set

      10.0.0.0/32 is subnetted, 1 subnets
C        10.30.0.31 is directly connected, Loopback1
      172.16.0.0/16 is variably subnetted, 10 subnets, 2 masks
B        172.16.1.0/24 [20/0] via 172.16.3.21, 00:49:35
C        172.16.2.0/24 is directly connected, Ethernet0/0
L        172.16.2.31/32 is directly connected, Ethernet0/0
C        172.16.3.0/24 is directly connected, Ethernet0/1
L        172.16.3.31/32 is directly connected, Ethernet0/1
C        172.16.4.0/24 is directly connected, Ethernet0/2
L        172.16.4.31/32 is directly connected, Ethernet0/2
B        172.16.5.0/24 [20/0] via 172.16.3.21, 00:47:48
B        172.16.6.0/24 [20/0] via 172.16.4.44, 00:36:54
B        172.16.7.0/24 [20/0] via 172.16.4.44, 00:31:37
R31#
R31#
R31#
R31#
R31#show ip bgp
BGP table version is 9, local router ID is 10.30.0.31
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter,
              x best-external, a additional-path, c RIB-compressed,
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 *>  172.16.1.0/24    172.16.3.21              0             0 101 i
 *   172.16.2.0/24    172.16.2.14              0             0 1001 i
 *>                   0.0.0.0                  0         32768 i
 *>  172.16.3.0/24    0.0.0.0                  0         32768 i
 *                    172.16.3.21              0             0 101 i
 *   172.16.4.0/24    172.16.4.44              0             0 520 i
 *>                   0.0.0.0                  0         32768 i
 *>  172.16.5.0/24    172.16.3.21              0             0 101 i
 *>  172.16.6.0/24    172.16.4.44              0             0 520 i
 *>  172.16.7.0/24    172.16.4.44                            0 520 2042 i
R31#
R31#
R31#
R31#
R31#show ip bgp summary
BGP router identifier 10.30.0.31, local AS number 301
BGP table version is 9, main routing table version 9
7 network entries using 980 bytes of memory
10 path entries using 800 bytes of memory
5/4 BGP path/bestpath attribute entries using 720 bytes of memory
4 BGP AS-PATH entries using 96 bytes of memory
0 BGP route-map cache entries using 0 bytes of memory
0 BGP filter-list cache entries using 0 bytes of memory
BGP using 2596 total bytes of memory
BGP activity 7/0 prefixes, 10/0 paths, scan interval 60 secs

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
172.16.2.14     4         1001      51      54        9    0    0 00:42:15        1
172.16.3.21     4          101      62      59        9    0    0 00:49:35        3
172.16.4.44     4          520      46      50        9    0    0 00:37:24        3


```
</details>


### R41

<details>
  <summary>Конфигурация</summary>

```

R41#terminal length 0
R41#sh run
Building configuration...

Current configuration : 1395 bytes
!
! Last configuration change at 18:30:13 UTC Sat Jan 18 2025
!
version 15.4
service timestamps debug datetime msec
service timestamps log datetime msec
no service password-encryption
!
hostname R41
!
boot-start-marker
boot-end-marker
!
!
!
no aaa new-model
mmi polling-interval 60
no mmi auto-configure
no mmi pvc
mmi snmp-timeout 180
!
!
!
!
!
!
!
!


!
!
!
!
no ip domain lookup
ip cef
no ipv6 cef
!
multilink bundle-name authenticated
!
!
!
!
!
!
!
!
!
redundancy
!
!
!
!
!
!
!
!
!
!
!
!
!
!
!
interface Loopback1
 ip address 10.40.0.41 255.255.255.255
 ip router isis
!
interface Ethernet0/0
 ip address 172.16.5.41 255.255.255.0
!
interface Ethernet0/1
 ip address 10.40.2.41 255.255.255.0
 ip router isis
!
interface Ethernet0/2
 ip address 10.40.1.41 255.255.255.0
 ip router isis
!
interface Ethernet0/3
 no ip address
!
interface Ethernet1/0
 no ip address
 shutdown
!
interface Ethernet1/1
 no ip address
 shutdown
!
interface Ethernet1/2
 no ip address
 shutdown
!
interface Ethernet1/3
 no ip address
 shutdown
!
router isis
 net 49.2222.0000.0000.0041.00
!
router bgp 520
 bgp log-neighbor-changes
 network 172.16.5.0 mask 255.255.255.0
 neighbor 172.16.5.21 remote-as 101
!
ip forward-protocol nd
!
!
no ip http server
no ip http secure-server
!
!
!
!
control-plane
!
!
!
!
!
!
!
!
line con 0
 logging synchronous
line aux 0
line vty 0 4
 login
 transport input none
!
!
end

R41#
R41#
R41#
R41#show ip interface brief
Interface                  IP-Address      OK? Method Status                Protocol
Ethernet0/0                172.16.5.41     YES manual up                    up
Ethernet0/1                10.40.2.41      YES manual up                    up
Ethernet0/2                10.40.1.41      YES manual up                    up
Ethernet0/3                unassigned      YES manual down                  down
Ethernet1/0                unassigned      YES manual administratively down down
Ethernet1/1                unassigned      YES manual administratively down down
Ethernet1/2                unassigned      YES manual administratively down down
Ethernet1/3                unassigned      YES manual administratively down down
Loopback1                  10.40.0.41      YES manual up                    up
R41#
R41#
R41#
R41#
R41#show ip route
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
       a - application route
       + - replicated route, % - next hop override

Gateway of last resort is not set

      10.0.0.0/8 is variably subnetted, 10 subnets, 2 masks
C        10.40.0.41/32 is directly connected, Loopback1
i L1     10.40.0.42/32 [115/20] via 10.40.2.42, 4w0d, Ethernet0/1
i L2     10.40.0.43/32 [115/30] via 10.40.2.42, 4w0d, Ethernet0/1
                       [115/30] via 10.40.1.44, 4w0d, Ethernet0/2
i L2     10.40.0.44/32 [115/20] via 10.40.1.44, 4w0d, Ethernet0/2
C        10.40.1.0/24 is directly connected, Ethernet0/2
L        10.40.1.41/32 is directly connected, Ethernet0/2
C        10.40.2.0/24 is directly connected, Ethernet0/1
L        10.40.2.41/32 is directly connected, Ethernet0/1
i L1     10.40.3.0/24 [115/20] via 10.40.2.42, 4w0d, Ethernet0/1
i L2     10.40.4.0/24 [115/20] via 10.40.1.44, 4w0d, Ethernet0/2
      172.16.0.0/16 is variably subnetted, 6 subnets, 2 masks
B        172.16.1.0/24 [20/0] via 172.16.5.21, 00:25:17
B        172.16.2.0/24 [20/0] via 172.16.5.21, 00:25:17
B        172.16.3.0/24 [20/0] via 172.16.5.21, 00:25:17
B        172.16.4.0/24 [20/0] via 172.16.5.21, 00:25:17
C        172.16.5.0/24 is directly connected, Ethernet0/0
L        172.16.5.41/32 is directly connected, Ethernet0/0
R41#
R41#
R41#
R41#
R41#show ip bgp
BGP table version is 6, local router ID is 10.40.0.41
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter,
              x best-external, a additional-path, c RIB-compressed,
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 *>  172.16.1.0/24    172.16.5.21              0             0 101 i
 *>  172.16.2.0/24    172.16.5.21                            0 101 301 i
 *>  172.16.3.0/24    172.16.5.21              0             0 101 i
 *>  172.16.4.0/24    172.16.5.21                            0 101 301 i
 *   172.16.5.0/24    172.16.5.21              0             0 101 i
 *>                   0.0.0.0                  0         32768 i
R41#
R41#
R41#
R41#
R41#show ip bgp summary
BGP router identifier 10.40.0.41, local AS number 520
BGP table version is 6, main routing table version 6
5 network entries using 700 bytes of memory
6 path entries using 480 bytes of memory
3/3 BGP path/bestpath attribute entries using 432 bytes of memory
2 BGP AS-PATH entries using 48 bytes of memory
0 BGP route-map cache entries using 0 bytes of memory
0 BGP filter-list cache entries using 0 bytes of memory
BGP using 1660 total bytes of memory
BGP activity 5/0 prefixes, 6/0 paths, scan interval 60 secs

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
172.16.5.21     4          101      36      32        6    0    0 00:25:16        5


```
</details>

### R43

<details>
  <summary>Конфигурация</summary>

```

R43#sh run
Building configuration...

Current configuration : 1498 bytes
!
! Last configuration change at 18:23:34 UTC Sat Jan 18 2025
!
version 15.4
service timestamps debug datetime msec
service timestamps log datetime msec
no service password-encryption
!
hostname R43
!
boot-start-marker
boot-end-marker
!
!
!
no aaa new-model
mmi polling-interval 60
no mmi auto-configure
no mmi pvc
mmi snmp-timeout 180
!
!
!
!
!
!
!
!


!
!
!
!
no ip domain lookup
ip cef
no ipv6 cef
!
multilink bundle-name authenticated
!
!
!
!
!
!
!
!
!
redundancy
!
!
!
!
!
!
!
!
!
!
!
!
!
!
!
interface Loopback1
 ip address 10.40.0.43 255.255.255.255
 ip router isis
!
interface Ethernet0/0
 ip address 10.40.4.43 255.255.255.0
 ip router isis
!
interface Ethernet0/1
 ip address 172.16.8.43 255.255.255.0
!
interface Ethernet0/2
 ip address 10.40.3.43 255.255.255.0
 ip router isis
!
interface Ethernet0/3
 ip address 172.16.7.43 255.255.255.0
!
interface Ethernet1/0
 no ip address
 shutdown
!
interface Ethernet1/1
 no ip address
 shutdown
!
interface Ethernet1/2
 no ip address
 shutdown
!
interface Ethernet1/3
 no ip address
 shutdown
!
router isis
 net 49.2600.0000.0000.4300
!
router bgp 520
 bgp log-neighbor-changes
 network 172.16.7.0 mask 255.255.255.0
 network 172.16.8.0 mask 255.255.255.0
 neighbor 172.16.7.52 remote-as 2042
!
ip forward-protocol nd
!
!
no ip http server
no ip http secure-server
ip route 10.70.0.0 255.255.0.0 172.16.8.71
!
!
!
!
control-plane
!
!
!
!
!
!
!
!
line con 0
 logging synchronous
line aux 0
line vty 0 4
 login
 transport input none
!
!
end

R43#
R43#
R43#
R43#show ip interface brief
Interface                  IP-Address      OK? Method Status                Protocol
Ethernet0/0                10.40.4.43      YES manual up                    up
Ethernet0/1                172.16.8.43     YES TFTP   up                    up
Ethernet0/2                10.40.3.43      YES manual up                    up
Ethernet0/3                172.16.7.43     YES manual up                    up
Ethernet1/0                unassigned      YES TFTP   administratively down down
Ethernet1/1                unassigned      YES TFTP   administratively down down
Ethernet1/2                unassigned      YES TFTP   administratively down down
Ethernet1/3                unassigned      YES TFTP   administratively down down
Loopback1                  10.40.0.43      YES TFTP   up                    up
R43#
R43#
R43#
R43#
R43#show ip route
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
       a - application route
       + - replicated route, % - next hop override

Gateway of last resort is not set

      10.0.0.0/8 is variably subnetted, 11 subnets, 3 masks
i L2     10.40.0.41/32 [115/30] via 10.40.4.44, 4w0d, Ethernet0/0
                       [115/30] via 10.40.3.42, 4w0d, Ethernet0/2
i L2     10.40.0.42/32 [115/20] via 10.40.3.42, 4w0d, Ethernet0/2
C        10.40.0.43/32 is directly connected, Loopback1
i L2     10.40.0.44/32 [115/20] via 10.40.4.44, 4w0d, Ethernet0/0
i L2     10.40.1.0/24 [115/20] via 10.40.4.44, 4w0d, Ethernet0/0
i L2     10.40.2.0/24 [115/20] via 10.40.3.42, 4w0d, Ethernet0/2
C        10.40.3.0/24 is directly connected, Ethernet0/2
L        10.40.3.43/32 is directly connected, Ethernet0/2
C        10.40.4.0/24 is directly connected, Ethernet0/0
L        10.40.4.43/32 is directly connected, Ethernet0/0
S        10.70.0.0/16 [1/0] via 172.16.8.71
      172.16.0.0/16 is variably subnetted, 5 subnets, 2 masks
B        172.16.6.0/24 [20/0] via 172.16.7.52, 00:29:48
C        172.16.7.0/24 is directly connected, Ethernet0/3
L        172.16.7.43/32 is directly connected, Ethernet0/3
C        172.16.8.0/24 is directly connected, Ethernet0/1
L        172.16.8.43/32 is directly connected, Ethernet0/1
R43#
R43#
R43#
R43#
R43#show ip bgp
BGP table version is 5, local router ID is 10.40.0.43
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter,
              x best-external, a additional-path, c RIB-compressed,
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 *>  172.16.6.0/24    172.16.7.52              0             0 2042 i
 *>  172.16.7.0/24    0.0.0.0                  0         32768 i
 *                    172.16.7.52              0             0 2042 i
 *>  172.16.8.0/24    0.0.0.0                  0         32768 i
R43#
R43#
R43#
R43#
R43#show ip bgp summary
BGP router identifier 10.40.0.43, local AS number 520
BGP table version is 5, main routing table version 5
3 network entries using 420 bytes of memory
4 path entries using 320 bytes of memory
2/2 BGP path/bestpath attribute entries using 288 bytes of memory
1 BGP AS-PATH entries using 24 bytes of memory
0 BGP route-map cache entries using 0 bytes of memory
0 BGP filter-list cache entries using 0 bytes of memory
BGP using 1052 total bytes of memory
BGP activity 3/0 prefixes, 4/0 paths, scan interval 60 secs

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
172.16.7.52     4         2042      43      37        5    0    0 00:29:47        2


```
</details>

### R44

<details>
  <summary>Конфигурация</summary>

```

R44#terminal length 0
R44#sh run
Building configuration...

Current configuration : 1491 bytes
!
! Last configuration change at 18:24:50 UTC Sat Jan 18 2025
!
version 15.4
service timestamps debug datetime msec
service timestamps log datetime msec
no service password-encryption
!
hostname R44
!
boot-start-marker
boot-end-marker
!
!
!
no aaa new-model
mmi polling-interval 60
no mmi auto-configure
no mmi pvc
mmi snmp-timeout 180
!
!
!
!
!
!
!
!


!
!
!
!
no ip domain lookup
ip cef
no ipv6 cef
!
multilink bundle-name authenticated
!
!
!
!
!
!
!
!
!
redundancy
!
!
!
!
!
!
!
!
!
!
!
!
!
!
!
interface Loopback1
 ip address 10.40.0.44 255.255.255.255
 ip router isis
!
interface Ethernet0/0
 ip address 172.16.4.44 255.255.255.0
!
interface Ethernet0/1
 ip address 10.40.4.44 255.255.255.0
 ip router isis
!
interface Ethernet0/2
 ip address 10.40.1.44 255.255.255.0
 ip router isis
!
interface Ethernet0/3
 ip address 172.16.6.44 255.255.255.0
!
interface Ethernet1/0
 no ip address
 shutdown
!
interface Ethernet1/1
 no ip address
 shutdown
!
interface Ethernet1/2
 no ip address
 shutdown
!
interface Ethernet1/3
 no ip address
 shutdown
!
router isis
 net 49.2400.0000.0000.4400
!
router bgp 520
 bgp log-neighbor-changes
 network 172.16.4.0 mask 255.255.255.0
 network 172.16.6.0 mask 255.255.255.0
 neighbor 172.16.4.31 remote-as 301
 neighbor 172.16.6.52 remote-as 2042
!
ip forward-protocol nd
!
!
no ip http server
no ip http secure-server
!
!
!
!
control-plane
!
!
!
!
!
!
!
!
line con 0
 logging synchronous
line aux 0
line vty 0 4
 login
 transport input none
!
!
end

R44#
R44#
R44#
R44#show ip interface brief
Interface                  IP-Address      OK? Method Status                Protocol
Ethernet0/0                172.16.4.44     YES manual up                    up
Ethernet0/1                10.40.4.44      YES manual up                    up
Ethernet0/2                10.40.1.44      YES manual up                    up
Ethernet0/3                172.16.6.44     YES manual up                    up
Ethernet1/0                unassigned      YES manual administratively down down
Ethernet1/1                unassigned      YES manual administratively down down
Ethernet1/2                unassigned      YES manual administratively down down
Ethernet1/3                unassigned      YES manual administratively down down
Loopback1                  10.40.0.44      YES manual up                    up
R44#
R44#
R44#
R44#
R44#show ip route
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
       a - application route
       + - replicated route, % - next hop override

Gateway of last resort is not set

      10.0.0.0/8 is variably subnetted, 10 subnets, 2 masks
i L2     10.40.0.41/32 [115/20] via 10.40.1.41, 4w0d, Ethernet0/2
i L2     10.40.0.42/32 [115/30] via 10.40.4.43, 4w0d, Ethernet0/1
                       [115/30] via 10.40.1.41, 4w0d, Ethernet0/2
i L2     10.40.0.43/32 [115/20] via 10.40.4.43, 4w0d, Ethernet0/1
C        10.40.0.44/32 is directly connected, Loopback1
C        10.40.1.0/24 is directly connected, Ethernet0/2
L        10.40.1.44/32 is directly connected, Ethernet0/2
i L2     10.40.2.0/24 [115/20] via 10.40.1.41, 4w0d, Ethernet0/2
i L2     10.40.3.0/24 [115/20] via 10.40.4.43, 4w0d, Ethernet0/1
C        10.40.4.0/24 is directly connected, Ethernet0/1
L        10.40.4.44/32 is directly connected, Ethernet0/1
      172.16.0.0/16 is variably subnetted, 9 subnets, 2 masks
B        172.16.1.0/24 [20/0] via 172.16.4.31, 00:35:05
B        172.16.2.0/24 [20/0] via 172.16.4.31, 00:35:05
B        172.16.3.0/24 [20/0] via 172.16.4.31, 00:35:05
C        172.16.4.0/24 is directly connected, Ethernet0/0
L        172.16.4.44/32 is directly connected, Ethernet0/0
B        172.16.5.0/24 [20/0] via 172.16.4.31, 00:35:05
C        172.16.6.0/24 is directly connected, Ethernet0/3
L        172.16.6.44/32 is directly connected, Ethernet0/3
B        172.16.7.0/24 [20/0] via 172.16.6.52, 00:29:18
R44#
R44#
R44#
R44#
R44#show ip bgp
BGP table version is 9, local router ID is 10.40.0.44
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter,
              x best-external, a additional-path, c RIB-compressed,
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 *>  172.16.1.0/24    172.16.4.31                            0 301 101 i
 *>  172.16.2.0/24    172.16.4.31              0             0 301 i
 *>  172.16.3.0/24    172.16.4.31              0             0 301 i
 *>  172.16.4.0/24    0.0.0.0                  0         32768 i
 *                    172.16.4.31              0             0 301 i
 *>  172.16.5.0/24    172.16.4.31                            0 301 101 i
 *   172.16.6.0/24    172.16.6.52              0             0 2042 i
 *>                   0.0.0.0                  0         32768 i
 *>  172.16.7.0/24    172.16.6.52              0             0 2042 i
R44#
R44#
R44#
R44#
R44#show ip bgp summary
BGP router identifier 10.40.0.44, local AS number 520
BGP table version is 9, main routing table version 9
7 network entries using 980 bytes of memory
9 path entries using 720 bytes of memory
4/4 BGP path/bestpath attribute entries using 576 bytes of memory
3 BGP AS-PATH entries using 72 bytes of memory
0 BGP route-map cache entries using 0 bytes of memory
0 BGP filter-list cache entries using 0 bytes of memory
BGP using 2348 total bytes of memory
BGP activity 7/0 prefixes, 9/0 paths, scan interval 60 secs

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
172.16.4.31     4          301      47      43        9    0    0 00:35:05        5
172.16.6.52     4         2042      39      42        9    0    0 00:30:18        2

```
</details>

### R52

<details>
  <summary>Конфигурация</summary>

```

R52#terminal length 0
R52#sh run
Building configuration...

Current configuration : 1453 bytes
!
! Last configuration change at 18:22:07 UTC Sat Jan 18 2025
!
version 15.4
service timestamps debug datetime msec
service timestamps log datetime msec
no service password-encryption
!
hostname R52
!
boot-start-marker
boot-end-marker
!
!
!
no aaa new-model
mmi polling-interval 60
no mmi auto-configure
no mmi pvc
mmi snmp-timeout 180
!
!
!
!
!
!
!
!


!
!
!
!
no ip domain lookup
ip cef
no ipv6 cef
!
multilink bundle-name authenticated
!
!
!
!
!
!
!
!
!
redundancy
!
!
!
!
!
!
!
!
!
!
!
!
!
!
!
interface Loopback1
 ip address 10.50.0.52 255.255.255.255
!
interface Ethernet0/0
 ip address 10.50.4.52 255.255.255.0
!
interface Ethernet0/1
 ip address 10.50.3.52 255.255.255.0
!
interface Ethernet0/2
 ip address 172.16.6.52 255.255.255.0
!
interface Ethernet0/3
 ip address 172.16.7.52 255.255.255.0
!
!
router eigrp SPB
 !
 address-family ipv4 unicast autonomous-system 1
  !
  topology base
  exit-af-topology
  network 10.50.0.52 0.0.0.0
  network 10.50.3.0 0.0.0.255
  network 10.50.4.0 0.0.0.255
  eigrp router-id 10.50.0.52
 exit-address-family
!
router bgp 2042
 bgp log-neighbor-changes
 network 172.16.6.0 mask 255.255.255.0
 network 172.16.7.0 mask 255.255.255.0
 neighbor 172.16.6.44 remote-as 520
 neighbor 172.16.7.43 remote-as 520
!
ip forward-protocol nd
!
!
no ip http server
no ip http secure-server
!
!
!
!
control-plane
!
!
!
!
!
!
!
!
line con 0
 logging synchronous
line aux 0
line vty 0 4
 login
 transport input none
!
!
end

R52#
R52#
R52#
R52#show ip interface brief
Interface                  IP-Address      OK? Method Status                Protocol
Ethernet0/0                10.50.4.52      YES manual up                    up
Ethernet0/1                10.50.3.52      YES manual up                    up
Ethernet0/2                172.16.6.52     YES manual up                    up
Ethernet0/3                172.16.7.52     YES manual up                    up
Loopback1                  10.50.0.52      YES manual up                    up
R52#
R52#
R52#
R52#
R52#show ip route
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
       a - application route
       + - replicated route, % - next hop override

Gateway of last resort is not set

      10.0.0.0/8 is variably subnetted, 13 subnets, 2 masks
D        10.50.0.51/32 [90/1024640] via 10.50.3.51, 3w6d, Ethernet0/1
C        10.50.0.52/32 is directly connected, Loopback1
D        10.50.0.53/32 [90/1024640] via 10.50.4.53, 3w6d, Ethernet0/0
D        10.50.0.54/32 [90/1536640] via 10.50.4.53, 3w6d, Ethernet0/0
D        10.50.1.0/24 [90/1536000] via 10.50.4.53, 3w6d, Ethernet0/0
                      [90/1536000] via 10.50.3.51, 3w6d, Ethernet0/1
D        10.50.2.0/24 [90/1536000] via 10.50.4.53, 3w6d, Ethernet0/0
                      [90/1536000] via 10.50.3.51, 3w6d, Ethernet0/1
C        10.50.3.0/24 is directly connected, Ethernet0/1
L        10.50.3.52/32 is directly connected, Ethernet0/1
C        10.50.4.0/24 is directly connected, Ethernet0/0
L        10.50.4.52/32 is directly connected, Ethernet0/0
D        10.50.5.0/24 [90/1536000] via 10.50.4.53, 3w6d, Ethernet0/0
D        10.50.6.0/24 [90/1536000] via 10.50.4.53, 3w6d, Ethernet0/0
                      [90/1536000] via 10.50.3.51, 3w6d, Ethernet0/1
D        10.50.255.0/24 [90/1536000] via 10.50.4.53, 3w6d, Ethernet0/0
                        [90/1536000] via 10.50.3.51, 3w6d, Ethernet0/1
      172.16.0.0/16 is variably subnetted, 10 subnets, 2 masks
B        172.16.1.0/24 [20/0] via 172.16.6.44, 00:29:26
B        172.16.2.0/24 [20/0] via 172.16.6.44, 00:29:26
B        172.16.3.0/24 [20/0] via 172.16.6.44, 00:29:26
B        172.16.4.0/24 [20/0] via 172.16.6.44, 00:29:26
B        172.16.5.0/24 [20/0] via 172.16.6.44, 00:29:26
C        172.16.6.0/24 is directly connected, Ethernet0/2
L        172.16.6.52/32 is directly connected, Ethernet0/2
C        172.16.7.0/24 is directly connected, Ethernet0/3
L        172.16.7.52/32 is directly connected, Ethernet0/3
B        172.16.8.0/24 [20/0] via 172.16.7.43, 00:27:09
R52#
R52#
R52#
R52#
R52#show ip bgp
BGP table version is 10, local router ID is 10.50.0.52
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter,
              x best-external, a additional-path, c RIB-compressed,
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 *>  172.16.1.0/24    172.16.6.44                            0 520 301 101 i
 *>  172.16.2.0/24    172.16.6.44                            0 520 301 i
 *>  172.16.3.0/24    172.16.6.44                            0 520 301 i
 *>  172.16.4.0/24    172.16.6.44              0             0 520 i
 *>  172.16.5.0/24    172.16.6.44                            0 520 301 101 i
 *>  172.16.6.0/24    0.0.0.0                  0         32768 i
 *                    172.16.6.44              0             0 520 i
 *   172.16.7.0/24    172.16.7.43              0             0 520 i
 *>                   0.0.0.0                  0         32768 i
 *>  172.16.8.0/24    172.16.7.43              0             0 520 i
R52#
R52#
R52#
R52#
R52#show ip bgp summary
BGP router identifier 10.50.0.52, local AS number 2042
BGP table version is 10, main routing table version 10
8 network entries using 1120 bytes of memory
10 path entries using 800 bytes of memory
4/4 BGP path/bestpath attribute entries using 576 bytes of memory
3 BGP AS-PATH entries using 72 bytes of memory
0 BGP route-map cache entries using 0 bytes of memory
0 BGP filter-list cache entries using 0 bytes of memory
BGP using 2568 total bytes of memory
BGP activity 8/0 prefixes, 10/0 paths, scan interval 60 secs

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
172.16.6.44     4          520      41      38       10    0    0 00:29:26        6
172.16.7.43     4          520      35      41       10    0    0 00:28:10        2


```
</details>