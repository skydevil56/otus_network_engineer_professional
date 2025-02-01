# Протокол BGP. Фильтрация

## Цели

1. Настроить фильтрацию для офиса Москва
1. Настроить фильтрацию для офиса Санкт-Петербург

## Задание

1. Настроить фильтрацию при помощи (As-path filtering) в офисе Москва так, чтобы не появилось транзитного трафика.
1. Настроить фильтрацию при помощи (Prefix-list filtering) в офисе Санкт-Петербург так, чтобы не появилось транзитного трафика.
1. Настроить провайдера Киторн так, чтобы в офис Москва отдавался только маршрут по умолчанию.
1. Настроить провайдера Ламас так, чтобы в офис Москва отдавался только маршрут по умолчанию и подсети офиса Санкт-Петербург.
1. Все сети в лабораторной работе должны иметь IP связность.

## Топология

![a](media/lab11_1.PNG)

## Схема для импорта в PNETlab

[Схема для импорта в PNETlab](media/otus_cource_lab11_eBGP(filtering)_pnetlab_export-20250201-165622.zip)

## Версии ПО

- PNETlab - 5.3.11
- Роутеры - Cisco IOS Software, Linux Software (I86BI_LINUX-ADVENTERPRISEK9-M), Version 15.4(2)T4
- Коммутаторы - Cisco IOS Software, Linux Software (I86BI_LINUXL2-ADVENTERPRISEK9-M), Version 15.2(CML_NIGHTLY_20150703)
- ПК - VPC

## Решение

1. Настроить фильтрацию при помощи (As-path filtering) в офисе Москва так, чтобы не появилось транзитного трафика.

      Нужно в сторону провайдеров аноносировать только подсети из своей автономной системы (^$ - пустой список AS).

      На R13:

      ```

      ip as-path access-list 1 permit ^$

      router bgp 1001
       ...
       neighbor 172.16.1.21 filter-list 1 out
       ...

      ```

      На R14:

      ```

      ip as-path access-list 1 permit ^$

      router bgp 1001
       ...
       neighbor 172.16.2.31 filter-list 1 out
       ...

      ```

1. Настроить фильтрацию при помощи (Prefix-list filtering) в офисе Санкт-Петербург так, чтобы не появилось транзитного трафика.

      Нужно в сторону провайдера аноносировать только подсети из своей автономной системы (подсеть 10.50.0.0/16 Санкт-Петербурга).

      ```
      ip prefix-list MSK_NET seq 5 permit 10.50.0.0/16 le 32

      router bgp 2042
       ...
       neighbor 172.16.6.44 prefix-list MSK_NET out
       ...
       neighbor 172.16.7.43 prefix-list MSK_NET out

      ```

1. Настроить провайдера Киторн так, чтобы в офис Москва отдавался только маршрут по умолчанию.

      Нужно в сторону офиса Москва проанонсировать default и разрешить при помощи route-map только маршрут 0.0.0.0/0

      ```

      router bgp 101
       ...
       neighbor 172.16.1.13 default-originate
       neighbor 172.16.1.13 route-map EBGP-R13-OUT out
       ...
      
      ip prefix-list DEFAULT_ONLY seq 5 permit 0.0.0.0/0
      
      route-map EBGP-R13-OUT permit 10
       match ip address prefix-list DEFAULT_ONLY
      !

      ```      

1. Настроить провайдера Ламас так, чтобы в офис Москва отдавался только маршрут по умолчанию и подсети офиса Санкт-Петербург.

      Нужно в сторону офиса Москва проанонсировать default и разрешить (при помощи As-path filtering) только маршруты из автономной системы офиса Санкт-Петербург (2042)

      ```

      router bgp 301
       ...
       neighbor 172.16.2.14 default-originate
       neighbor 172.16.2.14 filter-list 1 out
       ...
      
      ip as-path access-list 1 permit _2042$

      ```      

1. Все сети в лабораторной работе должны иметь IP связность.

      Специальных настроек не требуется.


## Проверка работоспособности

Проверка будет осуществляться с устройста VPC11 в Москве.

1. Выполним проверку IP адресов на loopback интерфейсах роутеров R21, R31, R41-44, R52, R61, R71:

```
VPC11> ping 10.20.0.21

84 bytes from 10.20.0.21 icmp_seq=1 ttl=253 time=0.998 ms
84 bytes from 10.20.0.21 icmp_seq=2 ttl=253 time=1.121 ms
84 bytes from 10.20.0.21 icmp_seq=3 ttl=253 time=1.038 ms
84 bytes from 10.20.0.21 icmp_seq=4 ttl=253 time=1.252 ms
84 bytes from 10.20.0.21 icmp_seq=5 ttl=253 time=1.469 ms

VPC11> ping 10.30.0.31

84 bytes from 10.30.0.31 icmp_seq=1 ttl=253 time=1.296 ms
84 bytes from 10.30.0.31 icmp_seq=2 ttl=253 time=0.981 ms
84 bytes from 10.30.0.31 icmp_seq=3 ttl=253 time=1.038 ms
84 bytes from 10.30.0.31 icmp_seq=4 ttl=253 time=1.004 ms
84 bytes from 10.30.0.31 icmp_seq=5 ttl=253 time=1.092 ms

VPC11> ping 10.40.0.41

84 bytes from 10.40.0.41 icmp_seq=1 ttl=252 time=1.215 ms
84 bytes from 10.40.0.41 icmp_seq=2 ttl=252 time=1.475 ms
84 bytes from 10.40.0.41 icmp_seq=3 ttl=252 time=1.309 ms
84 bytes from 10.40.0.41 icmp_seq=4 ttl=252 time=1.372 ms
84 bytes from 10.40.0.41 icmp_seq=5 ttl=252 time=1.343 ms

VPC11> ping 10.40.0.42

84 bytes from 10.40.0.42 icmp_seq=1 ttl=251 time=1.478 ms
84 bytes from 10.40.0.42 icmp_seq=2 ttl=251 time=1.478 ms
84 bytes from 10.40.0.42 icmp_seq=3 ttl=251 time=1.556 ms
84 bytes from 10.40.0.42 icmp_seq=4 ttl=251 time=1.668 ms
84 bytes from 10.40.0.42 icmp_seq=5 ttl=251 time=1.517 ms

VPC11> ping 10.40.0.43

84 bytes from 10.40.0.43 icmp_seq=1 ttl=250 time=1.621 ms
84 bytes from 10.40.0.43 icmp_seq=2 ttl=250 time=1.775 ms
84 bytes from 10.40.0.43 icmp_seq=3 ttl=250 time=1.491 ms
84 bytes from 10.40.0.43 icmp_seq=4 ttl=250 time=1.665 ms
84 bytes from 10.40.0.43 icmp_seq=5 ttl=250 time=1.624 ms

VPC11> ping 10.40.0.44

84 bytes from 10.40.0.44 icmp_seq=1 ttl=252 time=1.552 ms
84 bytes from 10.40.0.44 icmp_seq=2 ttl=252 time=1.561 ms
84 bytes from 10.40.0.44 icmp_seq=3 ttl=252 time=1.507 ms
84 bytes from 10.40.0.44 icmp_seq=4 ttl=252 time=1.689 ms
84 bytes from 10.40.0.44 icmp_seq=5 ttl=252 time=1.707 ms

VPC11> ping 10.50.0.52

84 bytes from 10.50.0.52 icmp_seq=1 ttl=251 time=1.424 ms
84 bytes from 10.50.0.52 icmp_seq=2 ttl=251 time=1.414 ms
84 bytes from 10.50.0.52 icmp_seq=3 ttl=251 time=1.455 ms
84 bytes from 10.50.0.52 icmp_seq=4 ttl=251 time=1.487 ms
84 bytes from 10.50.0.52 icmp_seq=5 ttl=251 time=1.333 ms

VPC11> ping 10.60.0.61

84 bytes from 10.60.0.61 icmp_seq=1 ttl=250 time=1.633 ms
84 bytes from 10.60.0.61 icmp_seq=2 ttl=250 time=1.662 ms
84 bytes from 10.60.0.61 icmp_seq=3 ttl=250 time=1.822 ms
84 bytes from 10.60.0.61 icmp_seq=4 ttl=250 time=1.541 ms
84 bytes from 10.60.0.61 icmp_seq=5 ttl=250 time=1.659 ms

VPC11> ping 10.70.0.71

84 bytes from 10.70.0.71 icmp_seq=1 ttl=250 time=1.814 ms
84 bytes from 10.70.0.71 icmp_seq=2 ttl=250 time=1.658 ms
84 bytes from 10.70.0.71 icmp_seq=3 ttl=250 time=1.868 ms
84 bytes from 10.70.0.71 icmp_seq=4 ttl=250 time=1.589 ms
84 bytes from 10.70.0.71 icmp_seq=5 ttl=250 time=1.537 ms

```

2. Выполним проверку IP адресов устройств VPC51, VPC52, VPC71, VPC72:

```

VPC11> ping 10.50.1.51

84 bytes from 10.50.1.51 icmp_seq=1 ttl=58 time=3.008 ms
84 bytes from 10.50.1.51 icmp_seq=2 ttl=58 time=1.844 ms
84 bytes from 10.50.1.51 icmp_seq=3 ttl=58 time=2.305 ms
84 bytes from 10.50.1.51 icmp_seq=4 ttl=58 time=1.862 ms
84 bytes from 10.50.1.51 icmp_seq=5 ttl=58 time=1.979 ms

VPC11> ping 10.50.2.52

84 bytes from 10.50.2.52 icmp_seq=1 ttl=58 time=2.895 ms
84 bytes from 10.50.2.52 icmp_seq=2 ttl=58 time=4.064 ms
84 bytes from 10.50.2.52 icmp_seq=3 ttl=58 time=1.918 ms
84 bytes from 10.50.2.52 icmp_seq=4 ttl=58 time=1.724 ms
84 bytes from 10.50.2.52 icmp_seq=5 ttl=58 time=1.838 ms

VPC11> ping 10.70.1.71

84 bytes from 10.70.1.71 icmp_seq=1 ttl=57 time=3.253 ms
84 bytes from 10.70.1.71 icmp_seq=2 ttl=57 time=2.293 ms
84 bytes from 10.70.1.71 icmp_seq=3 ttl=57 time=2.591 ms
84 bytes from 10.70.1.71 icmp_seq=4 ttl=57 time=2.011 ms
84 bytes from 10.70.1.71 icmp_seq=5 ttl=57 time=2.100 ms

VPC11> ping 10.70.1.72

84 bytes from 10.70.1.72 icmp_seq=1 ttl=57 time=2.998 ms
84 bytes from 10.70.1.72 icmp_seq=2 ttl=57 time=1.798 ms
84 bytes from 10.70.1.72 icmp_seq=3 ttl=57 time=2.325 ms
84 bytes from 10.70.1.72 icmp_seq=4 ttl=57 time=1.965 ms
84 bytes from 10.70.1.72 icmp_seq=5 ttl=57 time=1.899 ms

```

Видим, что IP связность между всеми подсетями присутствует, цель задания достигнута.

## Конфигурации устройств

### R13

<details>
  <summary>Конфигурация</summary>

```

R13#terminal length 0
R13#sh run
Building configuration...

Current configuration : 1780 bytes
!
! Last configuration change at 15:02:10 UTC Sun Jan 26 2025
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
 redistribute bgp 1001 subnets
 default-information originate
!
router bgp 1001
 bgp log-neighbor-changes
 network 172.16.1.0 mask 255.255.255.0
 redistribute ospf 1
 neighbor 10.10.0.14 remote-as 1001
 neighbor 10.10.0.14 update-source Loopback1
 neighbor 172.16.1.21 remote-as 101
 neighbor 172.16.1.21 route-map EBGP-IN in
 neighbor 172.16.1.21 filter-list 1 out
!
ip forward-protocol nd
!
ip as-path access-list 1 permit ^$
!
no ip http server
no ip http secure-server
!
!
ip prefix-list OSPF-FILTER seq 10 deny 10.10.0.0/16 le 32
ip prefix-list OSPF-FILTER seq 20 permit 0.0.0.0/0 le 32
!
route-map EBGP-IN permit 0
 match ip address 1
 set local-preference 50
!
!
access-list 1 permit any
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
Ethernet0/0                10.10.3.13      YES NVRAM  up                    up
Ethernet0/1                10.10.7.13      YES NVRAM  up                    up
Ethernet0/2                172.16.1.13     YES NVRAM  up                    up
Ethernet0/3                10.10.4.13      YES NVRAM  up                    up
Loopback1                  10.10.0.13      YES NVRAM  up                    up
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

Gateway of last resort is 10.10.7.12 to network 0.0.0.0

O*E2  0.0.0.0/0 [110/1] via 10.10.7.12, 5d19h, Ethernet0/1
                [110/1] via 10.10.3.11, 5d19h, Ethernet0/0
      10.0.0.0/8 is variably subnetted, 30 subnets, 3 masks
O IA     10.10.0.11/32 [110/11] via 10.10.3.11, 1w0d, Ethernet0/0
O IA     10.10.0.12/32 [110/11] via 10.10.7.12, 1w0d, Ethernet0/1
C        10.10.0.13/32 is directly connected, Loopback1
O        10.10.0.14/32 [110/21] via 10.10.7.12, 1w0d, Ethernet0/1
                       [110/21] via 10.10.3.11, 1w0d, Ethernet0/0
O        10.10.0.15/32 [110/11] via 10.10.4.15, 1w0d, Ethernet0/3
O IA     10.10.0.16/32 [110/31] via 10.10.7.12, 1w0d, Ethernet0/1
                       [110/31] via 10.10.3.11, 1w0d, Ethernet0/0
O IA     10.10.1.0/24 [110/20] via 10.10.7.12, 1w0d, Ethernet0/1
                      [110/20] via 10.10.3.11, 1w0d, Ethernet0/0
O IA     10.10.2.0/24 [110/20] via 10.10.7.12, 1w0d, Ethernet0/1
                      [110/20] via 10.10.3.11, 1w0d, Ethernet0/0
C        10.10.3.0/24 is directly connected, Ethernet0/0
L        10.10.3.13/32 is directly connected, Ethernet0/0
C        10.10.4.0/24 is directly connected, Ethernet0/3
L        10.10.4.13/32 is directly connected, Ethernet0/3
O        10.10.5.0/24 [110/20] via 10.10.7.12, 1w0d, Ethernet0/1
O IA     10.10.6.0/24 [110/30] via 10.10.7.12, 1w0d, Ethernet0/1
                      [110/30] via 10.10.3.11, 1w0d, Ethernet0/0
C        10.10.7.0/24 is directly connected, Ethernet0/1
L        10.10.7.13/32 is directly connected, Ethernet0/1
O        10.10.8.0/24 [110/20] via 10.10.3.11, 1w0d, Ethernet0/0
O IA     10.10.9.0/24 [110/20] via 10.10.7.12, 1w0d, Ethernet0/1
                      [110/20] via 10.10.3.11, 1w0d, Ethernet0/0
O IA     10.10.255.0/24 [110/20] via 10.10.7.12, 1w0d, Ethernet0/1
                        [110/20] via 10.10.3.11, 1w0d, Ethernet0/0
O E2     10.50.0.51/32 [110/1] via 10.10.7.12, 5d19h, Ethernet0/1
                       [110/1] via 10.10.3.11, 5d19h, Ethernet0/0
O E2     10.50.0.52/32 [110/1] via 10.10.7.12, 5d19h, Ethernet0/1
                       [110/1] via 10.10.3.11, 5d19h, Ethernet0/0
O E2     10.50.0.53/32 [110/1] via 10.10.7.12, 5d19h, Ethernet0/1
                       [110/1] via 10.10.3.11, 5d19h, Ethernet0/0
O E2     10.50.0.54/32 [110/1] via 10.10.7.12, 5d19h, Ethernet0/1
                       [110/1] via 10.10.3.11, 5d19h, Ethernet0/0
O E2     10.50.1.0/24 [110/1] via 10.10.7.12, 5d19h, Ethernet0/1
                      [110/1] via 10.10.3.11, 5d19h, Ethernet0/0
O E2     10.50.2.0/24 [110/1] via 10.10.7.12, 5d19h, Ethernet0/1
                      [110/1] via 10.10.3.11, 5d19h, Ethernet0/0
O E2     10.50.3.0/24 [110/1] via 10.10.7.12, 5d19h, Ethernet0/1
                      [110/1] via 10.10.3.11, 5d19h, Ethernet0/0
O E2     10.50.4.0/24 [110/1] via 10.10.7.12, 5d19h, Ethernet0/1
                      [110/1] via 10.10.3.11, 5d19h, Ethernet0/0
O E2     10.50.5.0/24 [110/1] via 10.10.7.12, 5d19h, Ethernet0/1
                      [110/1] via 10.10.3.11, 5d19h, Ethernet0/0
O E2     10.50.6.0/24 [110/1] via 10.10.7.12, 5d19h, Ethernet0/1
                      [110/1] via 10.10.3.11, 5d19h, Ethernet0/0
O E2     10.50.255.0/24 [110/1] via 10.10.7.12, 5d19h, Ethernet0/1
                        [110/1] via 10.10.3.11, 5d19h, Ethernet0/0
      172.16.0.0/16 is variably subnetted, 3 subnets, 2 masks
C        172.16.1.0/24 is directly connected, Ethernet0/2
L        172.16.1.13/32 is directly connected, Ethernet0/2
O E2     172.16.2.0/24 [110/1] via 10.10.7.12, 1w0d, Ethernet0/1
                       [110/1] via 10.10.3.11, 1w0d, Ethernet0/0
R13#
R13#
R13#
R13#
R13#show ip bgp
BGP table version is 410, local router ID is 10.10.0.13
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter,
              x best-external, a additional-path, c RIB-compressed,
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 r>i 0.0.0.0          172.16.2.31              0    100      0 301 i
 r                    172.16.1.21                    50      0 101 i
 * i 10.10.0.11/32    10.10.8.11              11    100      0 ?
 *>                   10.10.3.11              11         32768 ?
 * i 10.10.0.12/32    10.10.5.12              11    100      0 ?
 *>                   10.10.7.12              11         32768 ?
 * i 10.10.0.13/32    10.10.5.12              21    100      0 ?
 *>                   0.0.0.0                  0         32768 ?
 * i 10.10.0.14/32    10.10.0.14               0    100      0 ?
 *>                   10.10.3.11              21         32768 ?
 * i 10.10.0.15/32    10.10.5.12              31    100      0 ?
 *>                   10.10.4.15              11         32768 ?
 * i 10.10.0.16/32    10.10.6.16              11    100      0 ?
 *>                   10.10.3.11              31         32768 ?
 * i 10.10.1.0/24     10.10.5.12              20    100      0 ?
 *>                   10.10.3.11              20         32768 ?
 * i 10.10.2.0/24     10.10.5.12              20    100      0 ?
 *>                   10.10.3.11              20         32768 ?
 * i 10.10.3.0/24     10.10.8.11              20    100      0 ?
 *>                   0.0.0.0                  0         32768 ?
 * i 10.10.4.0/24     10.10.5.12              30    100      0 ?
 *>                   0.0.0.0                  0         32768 ?
 * i 10.10.5.0/24     10.10.0.14               0    100      0 ?
 *>                   10.10.7.12              20         32768 ?
 * i 10.10.6.0/24     10.10.0.14               0    100      0 ?
 *>                   10.10.3.11              30         32768 ?
 * i 10.10.7.0/24     10.10.5.12              20    100      0 ?
 *>                   0.0.0.0                  0         32768 ?
 * i 10.10.8.0/24     10.10.0.14               0    100      0 ?
 *>                   10.10.3.11              20         32768 ?
 * i 10.10.9.0/24     10.10.5.12              20    100      0 ?
 *>                   10.10.3.11              20         32768 ?
 * i 10.10.255.0/24   10.10.5.12              20    100      0 ?
 *>                   10.10.3.11              20         32768 ?
 r>i 10.50.0.51/32    172.16.2.31              0    100      0 301 520 2042 ?
 r>i 10.50.0.52/32    172.16.2.31              0    100      0 301 520 2042 ?
 r>i 10.50.0.53/32    172.16.2.31              0    100      0 301 520 2042 ?
 r>i 10.50.0.54/32    172.16.2.31              0    100      0 301 520 2042 ?
 r>i 10.50.1.0/24     172.16.2.31              0    100      0 301 520 2042 ?
 r>i 10.50.2.0/24     172.16.2.31              0    100      0 301 520 2042 ?
 r>i 10.50.3.0/24     172.16.2.31              0    100      0 301 520 2042 ?
 r>i 10.50.4.0/24     172.16.2.31              0    100      0 301 520 2042 ?
 r>i 10.50.5.0/24     172.16.2.31              0    100      0 301 520 2042 ?
 r>i 10.50.6.0/24     172.16.2.31              0    100      0 301 520 2042 ?
 r>i 10.50.255.0/24   172.16.2.31              0    100      0 301 520 2042 ?
 *>  172.16.1.0/24    0.0.0.0                  0         32768 i
 r>i 172.16.2.0/24    10.10.0.14               0    100      0 i
R13#
R13#
R13#
R13#
R13#show ip bgp summary
BGP router identifier 10.10.0.13, local AS number 1001
BGP table version is 410, main routing table version 410
30 network entries using 4200 bytes of memory
47 path entries using 3760 bytes of memory
18/10 BGP path/bestpath attribute entries using 2592 bytes of memory
3 BGP AS-PATH entries using 72 bytes of memory
0 BGP route-map cache entries using 0 bytes of memory
0 BGP filter-list cache entries using 0 bytes of memory
BGP using 10624 total bytes of memory
BGP activity 149/119 prefixes, 596/549 paths, scan interval 60 secs

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
10.10.0.14      4         1001    9605    9624      410    0    0 6d00h          29
172.16.1.21     4          101    9249    9258      410    0    0 5d20h           1


```
</details>

### R14

<details>
  <summary>Конфигурация</summary>

```

R14#terminal length 0
R14#sh run
Building configuration...

Current configuration : 1632 bytes
!
! Last configuration change at 15:04:50 UTC Sun Jan 26 2025
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
 redistribute bgp 1001 subnets
 default-information originate
!
router bgp 1001
 bgp log-neighbor-changes
 network 172.16.2.0 mask 255.255.255.0
 redistribute ospf 1
 neighbor 10.10.0.13 remote-as 1001
 neighbor 10.10.0.13 update-source Loopback1
 neighbor 172.16.2.31 remote-as 301
 neighbor 172.16.2.31 filter-list 1 out
!
ip forward-protocol nd
!
ip as-path access-list 1 permit ^$
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
Ethernet0/0                10.10.5.14      YES NVRAM  up                    up
Ethernet0/1                10.10.8.14      YES NVRAM  up                    up
Ethernet0/2                172.16.2.14     YES NVRAM  up                    up
Ethernet0/3                10.10.6.14      YES NVRAM  up                    up
Loopback1                  10.10.0.14      YES NVRAM  up                    up
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

Gateway of last resort is 172.16.2.31 to network 0.0.0.0

B*    0.0.0.0/0 [20/0] via 172.16.2.31, 5d19h
      10.0.0.0/8 is variably subnetted, 30 subnets, 3 masks
O IA     10.10.0.11/32 [110/11] via 10.10.8.11, 1w0d, Ethernet0/1
O IA     10.10.0.12/32 [110/11] via 10.10.5.12, 1w0d, Ethernet0/0
O        10.10.0.13/32 [110/21] via 10.10.8.11, 1w0d, Ethernet0/1
                       [110/21] via 10.10.5.12, 1w0d, Ethernet0/0
C        10.10.0.14/32 is directly connected, Loopback1
O IA     10.10.0.15/32 [110/31] via 10.10.8.11, 1w0d, Ethernet0/1
                       [110/31] via 10.10.5.12, 1w0d, Ethernet0/0
O        10.10.0.16/32 [110/11] via 10.10.6.16, 1w0d, Ethernet0/3
O IA     10.10.1.0/24 [110/20] via 10.10.8.11, 1w0d, Ethernet0/1
                      [110/20] via 10.10.5.12, 1w0d, Ethernet0/0
O IA     10.10.2.0/24 [110/20] via 10.10.8.11, 1w0d, Ethernet0/1
                      [110/20] via 10.10.5.12, 1w0d, Ethernet0/0
O        10.10.3.0/24 [110/20] via 10.10.8.11, 1w0d, Ethernet0/1
O IA     10.10.4.0/24 [110/30] via 10.10.8.11, 1w0d, Ethernet0/1
                      [110/30] via 10.10.5.12, 1w0d, Ethernet0/0
C        10.10.5.0/24 is directly connected, Ethernet0/0
L        10.10.5.14/32 is directly connected, Ethernet0/0
C        10.10.6.0/24 is directly connected, Ethernet0/3
L        10.10.6.14/32 is directly connected, Ethernet0/3
O        10.10.7.0/24 [110/20] via 10.10.5.12, 1w0d, Ethernet0/0
C        10.10.8.0/24 is directly connected, Ethernet0/1
L        10.10.8.14/32 is directly connected, Ethernet0/1
O IA     10.10.9.0/24 [110/20] via 10.10.8.11, 1w0d, Ethernet0/1
                      [110/20] via 10.10.5.12, 1w0d, Ethernet0/0
O IA     10.10.255.0/24 [110/20] via 10.10.8.11, 1w0d, Ethernet0/1
                        [110/20] via 10.10.5.12, 1w0d, Ethernet0/0
B        10.50.0.51/32 [20/0] via 172.16.2.31, 5d19h
B        10.50.0.52/32 [20/0] via 172.16.2.31, 5d19h
B        10.50.0.53/32 [20/0] via 172.16.2.31, 5d19h
B        10.50.0.54/32 [20/0] via 172.16.2.31, 5d19h
B        10.50.1.0/24 [20/0] via 172.16.2.31, 5d19h
B        10.50.2.0/24 [20/0] via 172.16.2.31, 5d19h
B        10.50.3.0/24 [20/0] via 172.16.2.31, 5d19h
B        10.50.4.0/24 [20/0] via 172.16.2.31, 5d19h
B        10.50.5.0/24 [20/0] via 172.16.2.31, 5d19h
B        10.50.6.0/24 [20/0] via 172.16.2.31, 5d19h
B        10.50.255.0/24 [20/0] via 172.16.2.31, 5d19h
      172.16.0.0/16 is variably subnetted, 3 subnets, 2 masks
O E2     172.16.1.0/24 [110/1] via 10.10.8.11, 1w0d, Ethernet0/1
                       [110/1] via 10.10.5.12, 1w0d, Ethernet0/0
C        172.16.2.0/24 is directly connected, Ethernet0/2
L        172.16.2.14/32 is directly connected, Ethernet0/2
R14#
R14#
R14#
R14#
R14#show ip bgp
BGP table version is 695, local router ID is 10.10.0.14
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter,
              x best-external, a additional-path, c RIB-compressed,
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 *>  0.0.0.0          172.16.2.31                            0 301 i
 * i 10.10.0.11/32    10.10.3.11              11    100      0 ?
 *>                   10.10.8.11              11         32768 ?
 * i 10.10.0.12/32    10.10.7.12              11    100      0 ?
 *>                   10.10.5.12              11         32768 ?
 * i 10.10.0.13/32    10.10.0.13               0    100      0 ?
 *>                   10.10.5.12              21         32768 ?
 * i 10.10.0.14/32    10.10.3.11              21    100      0 ?
 *>                   0.0.0.0                  0         32768 ?
 * i 10.10.0.15/32    10.10.4.15              11    100      0 ?
 *>                   10.10.5.12              31         32768 ?
 * i 10.10.0.16/32    10.10.3.11              31    100      0 ?
 *>                   10.10.6.16              11         32768 ?
 * i 10.10.1.0/24     10.10.3.11              20    100      0 ?
 *>                   10.10.5.12              20         32768 ?
 * i 10.10.2.0/24     10.10.3.11              20    100      0 ?
 *>                   10.10.5.12              20         32768 ?
 * i 10.10.3.0/24     10.10.0.13               0    100      0 ?
 *>                   10.10.8.11              20         32768 ?
 * i 10.10.4.0/24     10.10.0.13               0    100      0 ?
 *>                   10.10.5.12              30         32768 ?
 * i 10.10.5.0/24     10.10.7.12              20    100      0 ?
 *>                   0.0.0.0                  0         32768 ?
 * i 10.10.6.0/24     10.10.3.11              30    100      0 ?
 *>                   0.0.0.0                  0         32768 ?
 * i 10.10.7.0/24     10.10.0.13               0    100      0 ?
 *>                   10.10.5.12              20         32768 ?
 * i 10.10.8.0/24     10.10.3.11              20    100      0 ?
 *>                   0.0.0.0                  0         32768 ?
 * i 10.10.9.0/24     10.10.3.11              20    100      0 ?
 *>                   10.10.5.12              20         32768 ?
 * i 10.10.255.0/24   10.10.3.11              20    100      0 ?
 *>                   10.10.5.12              20         32768 ?
 *>  10.50.0.51/32    172.16.2.31                            0 301 520 2042 ?
 *>  10.50.0.52/32    172.16.2.31                            0 301 520 2042 ?
 *>  10.50.0.53/32    172.16.2.31                            0 301 520 2042 ?
 *>  10.50.0.54/32    172.16.2.31                            0 301 520 2042 ?
 *>  10.50.1.0/24     172.16.2.31                            0 301 520 2042 ?
 *>  10.50.2.0/24     172.16.2.31                            0 301 520 2042 ?
 *>  10.50.3.0/24     172.16.2.31                            0 301 520 2042 ?
 *>  10.50.4.0/24     172.16.2.31                            0 301 520 2042 ?
 *>  10.50.5.0/24     172.16.2.31                            0 301 520 2042 ?
 *>  10.50.6.0/24     172.16.2.31                            0 301 520 2042 ?
 *>  10.50.255.0/24   172.16.2.31                            0 301 520 2042 ?
 r>i 172.16.1.0/24    10.10.0.13               0    100      0 i
 *>  172.16.2.0/24    0.0.0.0                  0         32768 i
R14#
R14#
R14#
R14#
R14#show ip bgp summary
BGP router identifier 10.10.0.14, local AS number 1001
BGP table version is 695, main routing table version 695
30 network entries using 4200 bytes of memory
46 path entries using 3680 bytes of memory
16/10 BGP path/bestpath attribute entries using 2304 bytes of memory
2 BGP AS-PATH entries using 48 bytes of memory
0 BGP route-map cache entries using 0 bytes of memory
0 BGP filter-list cache entries using 0 bytes of memory
BGP using 10232 total bytes of memory
BGP activity 62/32 prefixes, 452/406 paths, scan interval 60 secs

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
10.10.0.13      4         1001    9625    9606      695    0    0 6d00h          17
172.16.2.31     4          301    9242    9253      695    0    0 5d19h          12


```
</details>

### R21

<details>
  <summary>Конфигурация</summary>

```

R21#terminal length 0
R21#sh run
Building configuration...

Current configuration : 1739 bytes
!
! Last configuration change at 15:53:59 UTC Sat Feb 1 2025
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
 redistribute static
 neighbor 172.16.1.13 remote-as 1001
 neighbor 172.16.1.13 default-originate
 neighbor 172.16.1.13 route-map EBGP-R13-OUT out
 neighbor 172.16.3.31 remote-as 301
 neighbor 172.16.5.41 remote-as 520
!
ip forward-protocol nd
!
!
no ip http server
no ip http secure-server
ip route 10.20.0.0 255.255.0.0 Null0
!
!
ip prefix-list DEFAULT_ONLY seq 5 permit 0.0.0.0/0
!
route-map EBGP-R13-OUT permit 10
 match ip address prefix-list DEFAULT_ONLY
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
Ethernet0/0                172.16.1.21     YES NVRAM  up                    up
Ethernet0/1                172.16.3.21     YES NVRAM  up                    up
Ethernet0/2                172.16.5.21     YES NVRAM  up                    up
Ethernet0/3                unassigned      YES NVRAM  administratively down down
Ethernet1/0                unassigned      YES NVRAM  administratively down down
Ethernet1/1                unassigned      YES NVRAM  administratively down down
Ethernet1/2                unassigned      YES NVRAM  administratively down down
Ethernet1/3                unassigned      YES NVRAM  administratively down down
Loopback1                  10.20.0.21      YES NVRAM  up                    up
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

      10.0.0.0/8 is variably subnetted, 40 subnets, 3 masks
B        10.10.0.11/32 [20/11] via 172.16.1.13, 5d20h
B        10.10.0.12/32 [20/11] via 172.16.1.13, 5d20h
B        10.10.0.13/32 [20/0] via 172.16.1.13, 5d20h
B        10.10.0.14/32 [20/21] via 172.16.1.13, 5d20h
B        10.10.0.15/32 [20/11] via 172.16.1.13, 5d20h
B        10.10.0.16/32 [20/31] via 172.16.1.13, 5d20h
B        10.10.1.0/24 [20/20] via 172.16.1.13, 5d20h
B        10.10.2.0/24 [20/20] via 172.16.1.13, 5d20h
B        10.10.3.0/24 [20/0] via 172.16.1.13, 5d20h
B        10.10.4.0/24 [20/0] via 172.16.1.13, 5d20h
B        10.10.5.0/24 [20/20] via 172.16.1.13, 5d20h
B        10.10.6.0/24 [20/30] via 172.16.1.13, 5d20h
B        10.10.7.0/24 [20/0] via 172.16.1.13, 5d20h
B        10.10.8.0/24 [20/20] via 172.16.1.13, 5d20h
B        10.10.9.0/24 [20/20] via 172.16.1.13, 5d20h
B        10.10.255.0/24 [20/20] via 172.16.1.13, 5d20h
S        10.20.0.0/16 is directly connected, Null0
C        10.20.0.21/32 is directly connected, Loopback1
B        10.30.0.0/16 [20/0] via 172.16.3.31, 5d20h
B        10.40.0.41/32 [20/0] via 172.16.5.41, 5d20h
B        10.40.0.42/32 [20/20] via 172.16.5.41, 5d20h
B        10.40.0.43/32 [20/30] via 172.16.5.41, 5d20h
B        10.40.0.44/32 [20/20] via 172.16.5.41, 5d20h
B        10.40.1.0/24 [20/0] via 172.16.5.41, 5d20h
B        10.40.2.0/24 [20/0] via 172.16.5.41, 5d20h
B        10.40.3.0/24 [20/20] via 172.16.5.41, 5d20h
B        10.40.4.0/24 [20/20] via 172.16.5.41, 5d20h
B        10.50.0.51/32 [20/0] via 172.16.5.41, 5d20h
B        10.50.0.52/32 [20/0] via 172.16.5.41, 5d20h
B        10.50.0.53/32 [20/0] via 172.16.5.41, 5d20h
B        10.50.0.54/32 [20/0] via 172.16.5.41, 5d20h
B        10.50.1.0/24 [20/0] via 172.16.5.41, 5d20h
B        10.50.2.0/24 [20/0] via 172.16.5.41, 5d20h
B        10.50.3.0/24 [20/0] via 172.16.5.41, 5d20h
B        10.50.4.0/24 [20/0] via 172.16.5.41, 5d20h
B        10.50.5.0/24 [20/0] via 172.16.5.41, 5d20h
B        10.50.6.0/24 [20/0] via 172.16.5.41, 5d20h
B        10.50.255.0/24 [20/0] via 172.16.5.41, 5d20h
B        10.60.0.0/16 [20/0] via 172.16.5.41, 5d20h
B        10.70.0.0/16 [20/0] via 172.16.5.41, 5d20h
      172.16.0.0/16 is variably subnetted, 13 subnets, 2 masks
C        172.16.1.0/24 is directly connected, Ethernet0/0
L        172.16.1.21/32 is directly connected, Ethernet0/0
B        172.16.2.0/24 [20/0] via 172.16.3.31, 5d20h
C        172.16.3.0/24 is directly connected, Ethernet0/1
L        172.16.3.21/32 is directly connected, Ethernet0/1
B        172.16.4.0/24 [20/0] via 172.16.3.31, 5d20h
C        172.16.5.0/24 is directly connected, Ethernet0/2
L        172.16.5.21/32 is directly connected, Ethernet0/2
B        172.16.6.0/24 [20/0] via 172.16.5.41, 5d20h
B        172.16.7.0/24 [20/0] via 172.16.5.41, 5d20h
B        172.16.8.0/24 [20/0] via 172.16.5.41, 5d20h
B        172.16.9.0/24 [20/0] via 172.16.5.41, 5d20h
B        172.16.10.0/24 [20/0] via 172.16.5.41, 5d20h
R21#
R21#
R21#
R21#
R21#show ip bgp
BGP table version is 181, local router ID is 10.20.0.21
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter,
              x best-external, a additional-path, c RIB-compressed,
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
     0.0.0.0          0.0.0.0                                0 i
 *   10.10.0.11/32    172.16.3.31                            0 301 1001 ?
 *>                   172.16.1.13             11             0 1001 ?
 *   10.10.0.12/32    172.16.3.31                            0 301 1001 ?
 *>                   172.16.1.13             11             0 1001 ?
 *   10.10.0.13/32    172.16.3.31                            0 301 1001 ?
 *>                   172.16.1.13              0             0 1001 ?
 *   10.10.0.14/32    172.16.3.31                            0 301 1001 ?
 *>                   172.16.1.13             21             0 1001 ?
 *   10.10.0.15/32    172.16.3.31                            0 301 1001 ?
 *>                   172.16.1.13             11             0 1001 ?
 *   10.10.0.16/32    172.16.3.31                            0 301 1001 ?
 *>                   172.16.1.13             31             0 1001 ?
 *   10.10.1.0/24     172.16.3.31                            0 301 1001 ?
 *>                   172.16.1.13             20             0 1001 ?
 *   10.10.2.0/24     172.16.3.31                            0 301 1001 ?
 *>                   172.16.1.13             20             0 1001 ?
 *   10.10.3.0/24     172.16.3.31                            0 301 1001 ?
 *>                   172.16.1.13              0             0 1001 ?
 *   10.10.4.0/24     172.16.3.31                            0 301 1001 ?
 *>                   172.16.1.13              0             0 1001 ?
 *   10.10.5.0/24     172.16.3.31                            0 301 1001 ?
 *>                   172.16.1.13             20             0 1001 ?
 *   10.10.6.0/24     172.16.3.31                            0 301 1001 ?
 *>                   172.16.1.13             30             0 1001 ?
 *   10.10.7.0/24     172.16.3.31                            0 301 1001 ?
 *>                   172.16.1.13              0             0 1001 ?
 *   10.10.8.0/24     172.16.3.31                            0 301 1001 ?
 *>                   172.16.1.13             20             0 1001 ?
 *   10.10.9.0/24     172.16.3.31                            0 301 1001 ?
 *>                   172.16.1.13             20             0 1001 ?
 *   10.10.255.0/24   172.16.3.31                            0 301 1001 ?
 *>                   172.16.1.13             20             0 1001 ?
 *>  10.20.0.0/16     0.0.0.0                  0         32768 ?
 *   10.30.0.0/16     172.16.5.41                            0 520 301 ?
 *>                   172.16.3.31              0             0 301 ?
 *>  10.40.0.41/32    172.16.5.41                            0 520 ?
 *                    172.16.3.31                            0 301 520 ?
 *>  10.40.0.42/32    172.16.5.41             20             0 520 ?
 *                    172.16.3.31                            0 301 520 ?
 *>  10.40.0.43/32    172.16.5.41             30             0 520 ?
 *                    172.16.3.31                            0 301 520 ?
 *>  10.40.0.44/32    172.16.5.41             20             0 520 ?
 *>  10.40.1.0/24     172.16.5.41                            0 520 ?
 *>  10.40.2.0/24     172.16.5.41                            0 520 ?
 *                    172.16.3.31                            0 301 520 ?
 *>  10.40.3.0/24     172.16.5.41             20             0 520 ?
 *                    172.16.3.31                            0 301 520 ?
 *>  10.40.4.0/24     172.16.5.41             20             0 520 ?
 *>  10.50.0.51/32    172.16.5.41                            0 520 2042 ?
 *                    172.16.3.31                            0 301 520 2042 ?
 *>  10.50.0.52/32    172.16.5.41                            0 520 2042 ?
 *                    172.16.3.31                            0 301 520 2042 ?
 *>  10.50.0.53/32    172.16.5.41                            0 520 2042 ?
 *                    172.16.3.31                            0 301 520 2042 ?
 *>  10.50.0.54/32    172.16.5.41                            0 520 2042 ?
 *                    172.16.3.31                            0 301 520 2042 ?
 *>  10.50.1.0/24     172.16.5.41                            0 520 2042 ?
 *                    172.16.3.31                            0 301 520 2042 ?
 *>  10.50.2.0/24     172.16.5.41                            0 520 2042 ?
 *                    172.16.3.31                            0 301 520 2042 ?
 *>  10.50.3.0/24     172.16.5.41                            0 520 2042 ?
 *                    172.16.3.31                            0 301 520 2042 ?
 *>  10.50.4.0/24     172.16.5.41                            0 520 2042 ?
 *                    172.16.3.31                            0 301 520 2042 ?
 *>  10.50.5.0/24     172.16.5.41                            0 520 2042 ?
 *                    172.16.3.31                            0 301 520 2042 ?
 *>  10.50.6.0/24     172.16.5.41                            0 520 2042 ?
 *                    172.16.3.31                            0 301 520 2042 ?
 *>  10.50.255.0/24   172.16.5.41                            0 520 2042 ?
 *                    172.16.3.31                            0 301 520 2042 ?
 *>  10.60.0.0/16     172.16.5.41                            0 520 ?
 *                    172.16.3.31                            0 301 520 ?
 *>  10.70.0.0/16     172.16.5.41                            0 520 ?
 *                    172.16.3.31                            0 301 520 ?
 *   172.16.1.0/24    172.16.1.13              0             0 1001 i
 *>                   0.0.0.0                  0         32768 i
 *   172.16.2.0/24    172.16.1.13                            0 1001 i
 *                    172.16.5.41                            0 520 301 i
 *>                   172.16.3.31              0             0 301 i
 *>  172.16.3.0/24    0.0.0.0                  0         32768 i
 *                    172.16.3.31              0             0 301 i
 *   172.16.4.0/24    172.16.5.41                            0 520 i
 *>                   172.16.3.31              0             0 301 i
 *>  172.16.5.0/24    0.0.0.0                  0         32768 i
 *                    172.16.5.41              0             0 520 i
 *                    172.16.3.31                            0 301 520 i
 *>  172.16.6.0/24    172.16.5.41                            0 520 i
 *                    172.16.3.31                            0 301 520 i
 *>  172.16.7.0/24    172.16.5.41                            0 520 i
 *                    172.16.3.31                            0 301 520 i
 *>  172.16.8.0/24    172.16.5.41                            0 520 i
 *                    172.16.3.31                            0 301 520 i
 *>  172.16.9.0/24    172.16.5.41                            0 520 i
 *                    172.16.3.31                            0 301 520 i
 *>  172.16.10.0/24   172.16.5.41                            0 520 i
 *                    172.16.3.31                            0 301 520 i
R21#
R21#
R21#
R21#
R21#show ip bgp summary
BGP router identifier 10.20.0.21, local AS number 101
BGP table version is 181, main routing table version 181
50 network entries using 7000 bytes of memory
97 path entries using 7760 bytes of memory
25/15 BGP path/bestpath attribute entries using 3600 bytes of memory
8 BGP AS-PATH entries using 192 bytes of memory
0 BGP route-map cache entries using 0 bytes of memory
0 BGP filter-list cache entries using 0 bytes of memory
BGP using 18552 total bytes of memory
BGP activity 227/177 prefixes, 807/710 paths, scan interval 60 secs

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
172.16.1.13     4         1001    9259    9251      181    0    0 5d20h          18
172.16.3.31     4          301    9292    9297      181    0    0 5d20h          44
172.16.5.41     4          520    9285    9303      181    0    0 5d20h          30

```
</details>

### R31

<details>
  <summary>Конфигурация</summary>

```

R31#terminal length 0
R31#sh run
Building configuration...

Current configuration : 1639 bytes
!
! Last configuration change at 20:00:27 UTC Sun Jan 26 2025
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
 redistribute static
 neighbor 172.16.2.14 remote-as 1001
 neighbor 172.16.2.14 default-originate
 neighbor 172.16.2.14 filter-list 1 out
 neighbor 172.16.3.21 remote-as 101
 neighbor 172.16.4.44 remote-as 520
!
ip forward-protocol nd
!
ip as-path access-list 1 permit _2042$
!
no ip http server
no ip http secure-server
ip route 10.30.0.0 255.255.0.0 Null0
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
Ethernet0/0                172.16.2.31     YES NVRAM  up                    up
Ethernet0/1                172.16.3.31     YES NVRAM  up                    up
Ethernet0/2                172.16.4.31     YES NVRAM  up                    up
Ethernet0/3                unassigned      YES NVRAM  administratively down down
Ethernet1/0                unassigned      YES NVRAM  administratively down down
Ethernet1/1                unassigned      YES NVRAM  administratively down down
Ethernet1/2                unassigned      YES NVRAM  administratively down down
Ethernet1/3                unassigned      YES NVRAM  administratively down down
Loopback1                  10.30.0.31      YES NVRAM  up                    up
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

      10.0.0.0/8 is variably subnetted, 40 subnets, 3 masks
B        10.10.0.11/32 [20/11] via 172.16.2.14, 5d19h
B        10.10.0.12/32 [20/11] via 172.16.2.14, 5d19h
B        10.10.0.13/32 [20/21] via 172.16.2.14, 5d19h
B        10.10.0.14/32 [20/0] via 172.16.2.14, 5d19h
B        10.10.0.15/32 [20/31] via 172.16.2.14, 5d19h
B        10.10.0.16/32 [20/11] via 172.16.2.14, 5d19h
B        10.10.1.0/24 [20/20] via 172.16.2.14, 5d19h
B        10.10.2.0/24 [20/20] via 172.16.2.14, 5d19h
B        10.10.3.0/24 [20/20] via 172.16.2.14, 5d19h
B        10.10.4.0/24 [20/30] via 172.16.2.14, 5d19h
B        10.10.5.0/24 [20/0] via 172.16.2.14, 5d19h
B        10.10.6.0/24 [20/0] via 172.16.2.14, 5d19h
B        10.10.7.0/24 [20/20] via 172.16.2.14, 5d19h
B        10.10.8.0/24 [20/0] via 172.16.2.14, 5d19h
B        10.10.9.0/24 [20/20] via 172.16.2.14, 5d19h
B        10.10.255.0/24 [20/20] via 172.16.2.14, 5d19h
B        10.20.0.0/16 [20/0] via 172.16.3.21, 5d20h
S        10.30.0.0/16 is directly connected, Null0
C        10.30.0.31/32 is directly connected, Loopback1
B        10.40.0.41/32 [20/20] via 172.16.4.44, 6d00h
B        10.40.0.42/32 [20/30] via 172.16.4.44, 6d00h
B        10.40.0.43/32 [20/20] via 172.16.4.44, 6d00h
B        10.40.0.44/32 [20/0] via 172.16.3.21, 5d20h
B        10.40.1.0/24 [20/0] via 172.16.3.21, 5d20h
B        10.40.2.0/24 [20/20] via 172.16.4.44, 6d00h
B        10.40.3.0/24 [20/20] via 172.16.4.44, 6d00h
B        10.40.4.0/24 [20/0] via 172.16.3.21, 5d20h
B        10.50.0.51/32 [20/0] via 172.16.4.44, 6d00h
B        10.50.0.52/32 [20/0] via 172.16.4.44, 6d00h
B        10.50.0.53/32 [20/0] via 172.16.4.44, 6d00h
B        10.50.0.54/32 [20/0] via 172.16.4.44, 6d00h
B        10.50.1.0/24 [20/0] via 172.16.4.44, 6d00h
B        10.50.2.0/24 [20/0] via 172.16.4.44, 6d00h
B        10.50.3.0/24 [20/0] via 172.16.4.44, 6d00h
B        10.50.4.0/24 [20/0] via 172.16.4.44, 6d00h
B        10.50.5.0/24 [20/0] via 172.16.4.44, 6d00h
B        10.50.6.0/24 [20/0] via 172.16.4.44, 6d00h
B        10.50.255.0/24 [20/0] via 172.16.4.44, 6d00h
B        10.60.0.0/16 [20/0] via 172.16.4.44, 6d00h
B        10.70.0.0/16 [20/0] via 172.16.4.44, 6d00h
      172.16.0.0/16 is variably subnetted, 13 subnets, 2 masks
B        172.16.1.0/24 [20/0] via 172.16.3.21, 5d19h
C        172.16.2.0/24 is directly connected, Ethernet0/0
L        172.16.2.31/32 is directly connected, Ethernet0/0
C        172.16.3.0/24 is directly connected, Ethernet0/1
L        172.16.3.31/32 is directly connected, Ethernet0/1
C        172.16.4.0/24 is directly connected, Ethernet0/2
L        172.16.4.31/32 is directly connected, Ethernet0/2
B        172.16.5.0/24 [20/0] via 172.16.4.44, 5d20h
B        172.16.6.0/24 [20/0] via 172.16.4.44, 6d00h
B        172.16.7.0/24 [20/0] via 172.16.4.44, 6d00h
B        172.16.8.0/24 [20/0] via 172.16.4.44, 6d00h
B        172.16.9.0/24 [20/0] via 172.16.4.44, 6d00h
B        172.16.10.0/24 [20/0] via 172.16.4.44, 6d00h
R31#
R31#
R31#
R31#
R31#show ip bgp
BGP table version is 381, local router ID is 10.30.0.31
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter,
              x best-external, a additional-path, c RIB-compressed,
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
     0.0.0.0          0.0.0.0                                0 i
 *>  10.10.0.11/32    172.16.2.14             11             0 1001 ?
 *                    172.16.3.21                            0 101 1001 ?
 *>  10.10.0.12/32    172.16.2.14             11             0 1001 ?
 *                    172.16.3.21                            0 101 1001 ?
 *>  10.10.0.13/32    172.16.2.14             21             0 1001 ?
 *                    172.16.3.21                            0 101 1001 ?
 *>  10.10.0.14/32    172.16.2.14              0             0 1001 ?
 *                    172.16.3.21                            0 101 1001 ?
 *>  10.10.0.15/32    172.16.2.14             31             0 1001 ?
 *                    172.16.3.21                            0 101 1001 ?
 *>  10.10.0.16/32    172.16.2.14             11             0 1001 ?
 *                    172.16.3.21                            0 101 1001 ?
 *>  10.10.1.0/24     172.16.2.14             20             0 1001 ?
 *                    172.16.3.21                            0 101 1001 ?
 *>  10.10.2.0/24     172.16.2.14             20             0 1001 ?
 *                    172.16.3.21                            0 101 1001 ?
 *>  10.10.3.0/24     172.16.2.14             20             0 1001 ?
 *                    172.16.3.21                            0 101 1001 ?
 *>  10.10.4.0/24     172.16.2.14             30             0 1001 ?
 *                    172.16.3.21                            0 101 1001 ?
 *>  10.10.5.0/24     172.16.2.14              0             0 1001 ?
 *                    172.16.3.21                            0 101 1001 ?
 *>  10.10.6.0/24     172.16.2.14              0             0 1001 ?
 *                    172.16.3.21                            0 101 1001 ?
 *>  10.10.7.0/24     172.16.2.14             20             0 1001 ?
 *                    172.16.3.21                            0 101 1001 ?
 *>  10.10.8.0/24     172.16.2.14              0             0 1001 ?
 *                    172.16.3.21                            0 101 1001 ?
 *>  10.10.9.0/24     172.16.2.14             20             0 1001 ?
 *                    172.16.3.21                            0 101 1001 ?
 *>  10.10.255.0/24   172.16.2.14             20             0 1001 ?
 *                    172.16.3.21                            0 101 1001 ?
 *   10.20.0.0/16     172.16.4.44                            0 520 101 ?
 *>                   172.16.3.21              0             0 101 ?
 *>  10.30.0.0/16     0.0.0.0                  0         32768 ?
 *   10.40.0.41/32    172.16.3.21                            0 101 520 ?
 *>                   172.16.4.44             20             0 520 ?
 *   10.40.0.42/32    172.16.3.21                            0 101 520 ?
 *>                   172.16.4.44             30             0 520 ?
 *   10.40.0.43/32    172.16.3.21                            0 101 520 ?
 *>                   172.16.4.44             20             0 520 ?
 *>  10.40.0.44/32    172.16.3.21                            0 101 520 ?
 *>  10.40.1.0/24     172.16.3.21                            0 101 520 ?
 *   10.40.2.0/24     172.16.3.21                            0 101 520 ?
 *>                   172.16.4.44             20             0 520 ?
 *   10.40.3.0/24     172.16.3.21                            0 101 520 ?
 *>                   172.16.4.44             20             0 520 ?
 *>  10.40.4.0/24     172.16.3.21                            0 101 520 ?
 *   10.50.0.51/32    172.16.3.21                            0 101 520 2042 ?
 *>                   172.16.4.44                            0 520 2042 ?
 *   10.50.0.52/32    172.16.3.21                            0 101 520 2042 ?
 *>                   172.16.4.44                            0 520 2042 ?
 *   10.50.0.53/32    172.16.3.21                            0 101 520 2042 ?
 *>                   172.16.4.44                            0 520 2042 ?
 *   10.50.0.54/32    172.16.3.21                            0 101 520 2042 ?
 *>                   172.16.4.44                            0 520 2042 ?
 *   10.50.1.0/24     172.16.3.21                            0 101 520 2042 ?
 *>                   172.16.4.44                            0 520 2042 ?
 *   10.50.2.0/24     172.16.3.21                            0 101 520 2042 ?
 *>                   172.16.4.44                            0 520 2042 ?
 *   10.50.3.0/24     172.16.3.21                            0 101 520 2042 ?
 *>                   172.16.4.44                            0 520 2042 ?
 *   10.50.4.0/24     172.16.3.21                            0 101 520 2042 ?
 *>                   172.16.4.44                            0 520 2042 ?
 *   10.50.5.0/24     172.16.3.21                            0 101 520 2042 ?
 *>                   172.16.4.44                            0 520 2042 ?
 *   10.50.6.0/24     172.16.3.21                            0 101 520 2042 ?
 *>                   172.16.4.44                            0 520 2042 ?
 *   10.50.255.0/24   172.16.3.21                            0 101 520 2042 ?
 *>                   172.16.4.44                            0 520 2042 ?
 *   10.60.0.0/16     172.16.3.21                            0 101 520 ?
 *>                   172.16.4.44                            0 520 ?
 *   10.70.0.0/16     172.16.3.21                            0 101 520 ?
 *>                   172.16.4.44                            0 520 ?
 *   172.16.1.0/24    172.16.2.14                            0 1001 i
 *                    172.16.4.44                            0 520 101 i
 *>                   172.16.3.21              0             0 101 i
 *   172.16.2.0/24    172.16.2.14              0             0 1001 i
 *>                   0.0.0.0                  0         32768 i
 *   172.16.3.0/24    172.16.3.21              0             0 101 i
 *>                   0.0.0.0                  0         32768 i
 *   172.16.4.0/24    172.16.4.44              0             0 520 i
 *>                   0.0.0.0                  0         32768 i
 *   172.16.5.0/24    172.16.3.21              0             0 101 i
 *>                   172.16.4.44                            0 520 i
 *   172.16.6.0/24    172.16.3.21                            0 101 520 i
 *>                   172.16.4.44              0             0 520 i
 *   172.16.7.0/24    172.16.3.21                            0 101 520 i
 *>                   172.16.4.44                            0 520 i
 *   172.16.8.0/24    172.16.3.21                            0 101 520 i
 *>                   172.16.4.44                            0 520 i
 *   172.16.9.0/24    172.16.3.21                            0 101 520 i
 *>                   172.16.4.44                            0 520 i
 *   172.16.10.0/24   172.16.3.21                            0 101 520 i
 *>                   172.16.4.44                            0 520 i
R31#
R31#
R31#
R31#
R31#show ip bgp summary
BGP router identifier 10.30.0.31, local AS number 301
BGP table version is 381, main routing table version 381
50 network entries using 7000 bytes of memory
96 path entries using 7680 bytes of memory
25/17 BGP path/bestpath attribute entries using 3600 bytes of memory
8 BGP AS-PATH entries using 192 bytes of memory
0 BGP route-map cache entries using 0 bytes of memory
0 BGP filter-list cache entries using 0 bytes of memory
BGP using 18472 total bytes of memory
BGP activity 93/43 prefixes, 572/476 paths, scan interval 60 secs

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
172.16.2.14     4         1001    9254    9243      381    0    0 5d19h          18
172.16.3.21     4          101    9297    9293      381    0    0 5d20h          46
172.16.4.44     4          520    9579    9580      381    0    0 6d00h          27


```
</details>

### R52

<details>
  <summary>Конфигурация</summary>

```

R52#terminal length 0
R52#sh run
Building configuration...

Current configuration : 1691 bytes
!
! Last configuration change at 15:44:43 UTC Sun Jan 26 2025
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
   redistribute bgp 2042 metric 1000 1 255 1 1500
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
 redistribute eigrp 1
 neighbor 172.16.6.44 remote-as 520
 neighbor 172.16.6.44 prefix-list MSK_NET out
 neighbor 172.16.7.43 remote-as 520
 neighbor 172.16.7.43 prefix-list MSK_NET out
 maximum-paths 2
!
ip forward-protocol nd
!
!
no ip http server
no ip http secure-server
!
!
ip prefix-list MSK_NET seq 5 permit 10.50.0.0/16 le 32
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
Ethernet0/0                10.50.4.52      YES NVRAM  up                    up
Ethernet0/1                10.50.3.52      YES NVRAM  up                    up
Ethernet0/2                172.16.6.52     YES NVRAM  up                    up
Ethernet0/3                172.16.7.52     YES NVRAM  up                    up
Loopback1                  10.50.0.52      YES NVRAM  up                    up
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

      10.0.0.0/8 is variably subnetted, 41 subnets, 3 masks
B        10.10.0.11/32 [20/0] via 172.16.7.43, 5d19h
B        10.10.0.12/32 [20/0] via 172.16.7.43, 5d19h
B        10.10.0.13/32 [20/0] via 172.16.7.43, 5d19h
B        10.10.0.14/32 [20/0] via 172.16.7.43, 5d19h
B        10.10.0.15/32 [20/0] via 172.16.7.43, 5d19h
B        10.10.0.16/32 [20/0] via 172.16.7.43, 5d19h
B        10.10.1.0/24 [20/0] via 172.16.7.43, 5d19h
B        10.10.2.0/24 [20/0] via 172.16.7.43, 5d19h
B        10.10.3.0/24 [20/0] via 172.16.7.43, 5d19h
B        10.10.4.0/24 [20/0] via 172.16.7.43, 5d19h
B        10.10.5.0/24 [20/0] via 172.16.7.43, 5d19h
B        10.10.6.0/24 [20/0] via 172.16.7.43, 5d19h
B        10.10.7.0/24 [20/0] via 172.16.7.43, 5d19h
B        10.10.8.0/24 [20/0] via 172.16.7.43, 5d19h
B        10.10.9.0/24 [20/0] via 172.16.7.43, 5d19h
B        10.10.255.0/24 [20/0] via 172.16.7.43, 5d19h
B        10.20.0.0/16 [20/0] via 172.16.7.43, 5d20h
                      [20/0] via 172.16.6.44, 5d20h
B        10.30.0.0/16 [20/0] via 172.16.7.43, 6d00h
                      [20/0] via 172.16.6.44, 6d00h
B        10.40.0.41/32 [20/20] via 172.16.6.44, 6d00h
B        10.40.0.42/32 [20/20] via 172.16.7.43, 6d00h
B        10.40.0.43/32 [20/0] via 172.16.7.43, 6d00h
B        10.40.0.44/32 [20/20] via 172.16.7.43, 6d00h
B        10.40.1.0/24 [20/20] via 172.16.7.43, 6d00h
B        10.40.2.0/24 [20/20] via 172.16.7.43, 6d00h
                      [20/20] via 172.16.6.44, 6d00h
B        10.40.3.0/24 [20/0] via 172.16.7.43, 6d00h
B        10.40.4.0/24 [20/0] via 172.16.7.43, 6d00h
D        10.50.0.51/32 [90/1024640] via 10.50.3.51, 1w0d, Ethernet0/1
C        10.50.0.52/32 is directly connected, Loopback1
D        10.50.0.53/32 [90/1024640] via 10.50.4.53, 1w0d, Ethernet0/0
D        10.50.0.54/32 [90/1536640] via 10.50.4.53, 1w0d, Ethernet0/0
D        10.50.1.0/24 [90/1536000] via 10.50.4.53, 1w0d, Ethernet0/0
                      [90/1536000] via 10.50.3.51, 1w0d, Ethernet0/1
D        10.50.2.0/24 [90/1536000] via 10.50.4.53, 1w0d, Ethernet0/0
                      [90/1536000] via 10.50.3.51, 1w0d, Ethernet0/1
C        10.50.3.0/24 is directly connected, Ethernet0/1
L        10.50.3.52/32 is directly connected, Ethernet0/1
C        10.50.4.0/24 is directly connected, Ethernet0/0
L        10.50.4.52/32 is directly connected, Ethernet0/0
D        10.50.5.0/24 [90/1536000] via 10.50.4.53, 1w0d, Ethernet0/0
D        10.50.6.0/24 [90/1536000] via 10.50.4.53, 1w0d, Ethernet0/0
                      [90/1536000] via 10.50.3.51, 1w0d, Ethernet0/1
D        10.50.255.0/24 [90/1536000] via 10.50.4.53, 1w0d, Ethernet0/0
                        [90/1536000] via 10.50.3.51, 1w0d, Ethernet0/1
B        10.60.0.0/16 [20/0] via 172.16.7.43, 6d00h
                      [20/0] via 172.16.6.44, 6d00h
B        10.70.0.0/16 [20/0] via 172.16.7.43, 6d00h
                      [20/0] via 172.16.6.44, 6d00h
      172.16.0.0/16 is variably subnetted, 12 subnets, 2 masks
B        172.16.1.0/24 [20/0] via 172.16.7.43, 5d20h
                       [20/0] via 172.16.6.44, 5d20h
B        172.16.2.0/24 [20/0] via 172.16.7.43, 6d00h
                       [20/0] via 172.16.6.44, 6d00h
B        172.16.3.0/24 [20/0] via 172.16.7.43, 5d20h
B        172.16.4.0/24 [20/0] via 172.16.7.43, 6d00h
                       [20/0] via 172.16.6.44, 6d00h
B        172.16.5.0/24 [20/0] via 172.16.7.43, 6d00h
                       [20/0] via 172.16.6.44, 6d00h
C        172.16.6.0/24 is directly connected, Ethernet0/2
L        172.16.6.52/32 is directly connected, Ethernet0/2
C        172.16.7.0/24 is directly connected, Ethernet0/3
L        172.16.7.52/32 is directly connected, Ethernet0/3
B        172.16.8.0/24 [20/0] via 172.16.7.43, 6d00h
                       [20/0] via 172.16.6.44, 6d00h
B        172.16.9.0/24 [20/0] via 172.16.7.43, 6d00h
                       [20/0] via 172.16.6.44, 6d00h
B        172.16.10.0/24 [20/0] via 172.16.7.43, 6d00h
                        [20/0] via 172.16.6.44, 6d00h
R52#
R52#
R52#
R52#
R52#show ip bgp
BGP table version is 249, local router ID is 10.50.0.52
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter,
              x best-external, a additional-path, c RIB-compressed,
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 *   10.10.0.11/32    172.16.6.44                            0 520 301 1001 ?
 *>                   172.16.7.43                            0 520 101 1001 ?
 *   10.10.0.12/32    172.16.6.44                            0 520 301 1001 ?
 *>                   172.16.7.43                            0 520 101 1001 ?
 *   10.10.0.13/32    172.16.6.44                            0 520 301 1001 ?
 *>                   172.16.7.43                            0 520 101 1001 ?
 *   10.10.0.14/32    172.16.6.44                            0 520 301 1001 ?
 *>                   172.16.7.43                            0 520 101 1001 ?
 *   10.10.0.15/32    172.16.6.44                            0 520 301 1001 ?
 *>                   172.16.7.43                            0 520 101 1001 ?
 *   10.10.0.16/32    172.16.6.44                            0 520 301 1001 ?
 *>                   172.16.7.43                            0 520 101 1001 ?
 *   10.10.1.0/24     172.16.6.44                            0 520 301 1001 ?
 *>                   172.16.7.43                            0 520 101 1001 ?
 *   10.10.2.0/24     172.16.6.44                            0 520 301 1001 ?
 *>                   172.16.7.43                            0 520 101 1001 ?
 *   10.10.3.0/24     172.16.6.44                            0 520 301 1001 ?
 *>                   172.16.7.43                            0 520 101 1001 ?
 *   10.10.4.0/24     172.16.6.44                            0 520 301 1001 ?
 *>                   172.16.7.43                            0 520 101 1001 ?
 *   10.10.5.0/24     172.16.6.44                            0 520 301 1001 ?
 *>                   172.16.7.43                            0 520 101 1001 ?
 *   10.10.6.0/24     172.16.6.44                            0 520 301 1001 ?
 *>                   172.16.7.43                            0 520 101 1001 ?
 *   10.10.7.0/24     172.16.6.44                            0 520 301 1001 ?
 *>                   172.16.7.43                            0 520 101 1001 ?
 *   10.10.8.0/24     172.16.6.44                            0 520 301 1001 ?
 *>                   172.16.7.43                            0 520 101 1001 ?
 *   10.10.9.0/24     172.16.6.44                            0 520 301 1001 ?
 *>                   172.16.7.43                            0 520 101 1001 ?
 *   10.10.255.0/24   172.16.6.44                            0 520 301 1001 ?
 *>                   172.16.7.43                            0 520 101 1001 ?
 *m  10.20.0.0/16     172.16.7.43                            0 520 101 ?
 *>                   172.16.6.44                            0 520 101 ?
 *m  10.30.0.0/16     172.16.6.44                            0 520 301 ?
 *>                   172.16.7.43                            0 520 301 ?
 *>  10.40.0.41/32    172.16.6.44             20             0 520 ?
 *                    172.16.7.43             30             0 520 ?
 *   10.40.0.42/32    172.16.6.44             30             0 520 ?
 *>                   172.16.7.43             20             0 520 ?
 *   10.40.0.43/32    172.16.6.44             20             0 520 ?
 *>                   172.16.7.43                            0 520 ?
 *>  10.40.0.44/32    172.16.7.43             20             0 520 ?
 *>  10.40.1.0/24     172.16.7.43             20             0 520 ?
 *m  10.40.2.0/24     172.16.6.44             20             0 520 ?
 *>                   172.16.7.43             20             0 520 ?
 *   10.40.3.0/24     172.16.6.44             20             0 520 ?
 *>                   172.16.7.43                            0 520 ?
 *>  10.40.4.0/24     172.16.7.43                            0 520 ?
 *>  10.50.0.51/32    10.50.3.51         1024640         32768 ?
 *>  10.50.0.52/32    0.0.0.0                  0         32768 ?
 *>  10.50.0.53/32    10.50.4.53         1024640         32768 ?
 *>  10.50.0.54/32    10.50.4.53         1536640         32768 ?
 *>  10.50.1.0/24     10.50.3.51         1536000         32768 ?
 *>  10.50.2.0/24     10.50.3.51         1536000         32768 ?
 *>  10.50.3.0/24     0.0.0.0                  0         32768 ?
 *>  10.50.4.0/24     0.0.0.0                  0         32768 ?
 *>  10.50.5.0/24     10.50.4.53         1536000         32768 ?
 *>  10.50.6.0/24     10.50.3.51         1536000         32768 ?
 *>  10.50.255.0/24   10.50.3.51         1536000         32768 ?
 *m  10.60.0.0/16     172.16.6.44                            0 520 ?
 *>                   172.16.7.43                            0 520 ?
 *m  10.70.0.0/16     172.16.6.44                            0 520 ?
 *>                   172.16.7.43              0             0 520 ?
 *m  172.16.1.0/24    172.16.7.43                            0 520 101 i
 *>                   172.16.6.44                            0 520 101 i
 *m  172.16.2.0/24    172.16.6.44                            0 520 301 i
 *>                   172.16.7.43                            0 520 301 i
 *   172.16.3.0/24    172.16.6.44                            0 520 301 i
 *>                   172.16.7.43                            0 520 101 i
 *m  172.16.4.0/24    172.16.6.44              0             0 520 i
 *>                   172.16.7.43                            0 520 i
 *m  172.16.5.0/24    172.16.6.44                            0 520 i
 *>                   172.16.7.43                            0 520 i
 *   172.16.6.0/24    172.16.6.44              0             0 520 i
 *>                   0.0.0.0                  0         32768 i
 *                    172.16.7.43                            0 520 i
 *   172.16.7.0/24    172.16.6.44                            0 520 i
 *>                   0.0.0.0                  0         32768 i
 *                    172.16.7.43              0             0 520 i
 *m  172.16.8.0/24    172.16.6.44                            0 520 i
 *>                   172.16.7.43              0             0 520 i
 *m  172.16.9.0/24    172.16.6.44                            0 520 i
 *>                   172.16.7.43                            0 520 i
 *m  172.16.10.0/24   172.16.6.44                            0 520 i
 *>                   172.16.7.43                            0 520 i
R52#
R52#
R52#
R52#
R52#show ip bgp summary
BGP router identifier 10.50.0.52, local AS number 2042
BGP table version is 249, main routing table version 249
49 network entries using 6860 bytes of memory
86 path entries using 6880 bytes of memory
12 multipath network entries and 24 multipath paths
17/15 BGP path/bestpath attribute entries using 2448 bytes of memory
5 BGP AS-PATH entries using 120 bytes of memory
0 BGP route-map cache entries using 0 bytes of memory
0 BGP filter-list cache entries using 0 bytes of memory
BGP using 16308 total bytes of memory
BGP activity 149/100 prefixes, 387/301 paths, scan interval 60 secs

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
172.16.6.44     4          520    9540    9535      249    0    0 6d00h          35
172.16.7.43     4          520    9557    9582      249    0    0 6d00h          38

```
</details>