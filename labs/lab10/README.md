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


1. Настроить iBGP в провайдере Триада, с использованием Route Reflector.

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

Current configuration : 1705 bytes
!
! Last configuration change at 15:34:33 UTC Sat Jan 25 2025
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

Gateway of last resort is not set

      10.0.0.0/8 is variably subnetted, 42 subnets, 3 masks
O IA     10.10.0.11/32 [110/11] via 10.10.3.11, 1d02h, Ethernet0/0
O IA     10.10.0.12/32 [110/11] via 10.10.7.12, 1d02h, Ethernet0/1
C        10.10.0.13/32 is directly connected, Loopback1
O        10.10.0.14/32 [110/21] via 10.10.7.12, 1d02h, Ethernet0/1
                       [110/21] via 10.10.3.11, 1d02h, Ethernet0/0
O        10.10.0.15/32 [110/11] via 10.10.4.15, 1d02h, Ethernet0/3
O IA     10.10.0.16/32 [110/31] via 10.10.7.12, 1d02h, Ethernet0/1
                       [110/31] via 10.10.3.11, 1d02h, Ethernet0/0
O IA     10.10.1.0/24 [110/20] via 10.10.7.12, 1d02h, Ethernet0/1
                      [110/20] via 10.10.3.11, 1d02h, Ethernet0/0
O IA     10.10.2.0/24 [110/20] via 10.10.7.12, 1d02h, Ethernet0/1
                      [110/20] via 10.10.3.11, 1d02h, Ethernet0/0
C        10.10.3.0/24 is directly connected, Ethernet0/0
L        10.10.3.13/32 is directly connected, Ethernet0/0
C        10.10.4.0/24 is directly connected, Ethernet0/3
L        10.10.4.13/32 is directly connected, Ethernet0/3
O        10.10.5.0/24 [110/20] via 10.10.7.12, 1d02h, Ethernet0/1
O IA     10.10.6.0/24 [110/30] via 10.10.7.12, 1d02h, Ethernet0/1
                      [110/30] via 10.10.3.11, 1d02h, Ethernet0/0
C        10.10.7.0/24 is directly connected, Ethernet0/1
L        10.10.7.13/32 is directly connected, Ethernet0/1
O        10.10.8.0/24 [110/20] via 10.10.3.11, 1d02h, Ethernet0/0
O IA     10.10.9.0/24 [110/20] via 10.10.7.12, 1d02h, Ethernet0/1
                      [110/20] via 10.10.3.11, 1d02h, Ethernet0/0
O IA     10.10.255.0/24 [110/20] via 10.10.7.12, 1d02h, Ethernet0/1
                        [110/20] via 10.10.3.11, 1d02h, Ethernet0/0
O E2     10.20.0.0/16 [110/1] via 10.10.7.12, 22:18:22, Ethernet0/1
                      [110/1] via 10.10.3.11, 22:18:22, Ethernet0/0
O E2     10.30.0.0/16 [110/1] via 10.10.7.12, 22:16:11, Ethernet0/1
                      [110/1] via 10.10.3.11, 22:16:11, Ethernet0/0
O E2     10.40.0.41/32 [110/1] via 10.10.7.12, 22:27:36, Ethernet0/1
                       [110/1] via 10.10.3.11, 22:27:36, Ethernet0/0
O E2     10.40.0.42/32 [110/1] via 10.10.7.12, 22:27:36, Ethernet0/1
                       [110/1] via 10.10.3.11, 22:27:36, Ethernet0/0
O E2     10.40.0.43/32 [110/1] via 10.10.7.12, 22:27:36, Ethernet0/1
                       [110/1] via 10.10.3.11, 22:27:36, Ethernet0/0
O E2     10.40.0.44/32 [110/1] via 10.10.7.12, 22:27:36, Ethernet0/1
                       [110/1] via 10.10.3.11, 22:27:36, Ethernet0/0
O E2     10.40.1.0/24 [110/1] via 10.10.7.12, 22:27:36, Ethernet0/1
                      [110/1] via 10.10.3.11, 22:27:36, Ethernet0/0
O E2     10.40.2.0/24 [110/1] via 10.10.7.12, 22:27:36, Ethernet0/1
                      [110/1] via 10.10.3.11, 22:27:36, Ethernet0/0
O E2     10.40.3.0/24 [110/1] via 10.10.7.12, 22:27:36, Ethernet0/1
                      [110/1] via 10.10.3.11, 22:27:36, Ethernet0/0
O E2     10.40.4.0/24 [110/1] via 10.10.7.12, 22:27:36, Ethernet0/1
                      [110/1] via 10.10.3.11, 22:27:36, Ethernet0/0
O E2     10.50.0.51/32 [110/1] via 10.10.7.12, 22:27:36, Ethernet0/1
                       [110/1] via 10.10.3.11, 22:27:36, Ethernet0/0
O E2     10.50.0.52/32 [110/1] via 10.10.7.12, 22:27:36, Ethernet0/1
                       [110/1] via 10.10.3.11, 22:27:36, Ethernet0/0
O E2     10.50.0.53/32 [110/1] via 10.10.7.12, 22:27:36, Ethernet0/1
                       [110/1] via 10.10.3.11, 22:27:36, Ethernet0/0
O E2     10.50.0.54/32 [110/1] via 10.10.7.12, 22:27:36, Ethernet0/1
                       [110/1] via 10.10.3.11, 22:27:36, Ethernet0/0
O E2     10.50.1.0/24 [110/1] via 10.10.7.12, 22:27:36, Ethernet0/1
                      [110/1] via 10.10.3.11, 22:27:36, Ethernet0/0
O E2     10.50.2.0/24 [110/1] via 10.10.7.12, 22:27:36, Ethernet0/1
                      [110/1] via 10.10.3.11, 22:27:36, Ethernet0/0
O E2     10.50.3.0/24 [110/1] via 10.10.7.12, 22:27:36, Ethernet0/1
                      [110/1] via 10.10.3.11, 22:27:36, Ethernet0/0
O E2     10.50.4.0/24 [110/1] via 10.10.7.12, 22:27:36, Ethernet0/1
                      [110/1] via 10.10.3.11, 22:27:36, Ethernet0/0
O E2     10.50.5.0/24 [110/1] via 10.10.7.12, 22:27:36, Ethernet0/1
                      [110/1] via 10.10.3.11, 22:27:36, Ethernet0/0
O E2     10.50.6.0/24 [110/1] via 10.10.7.12, 22:27:36, Ethernet0/1
                      [110/1] via 10.10.3.11, 22:27:36, Ethernet0/0
O E2     10.50.255.0/24 [110/1] via 10.10.7.12, 22:27:36, Ethernet0/1
                        [110/1] via 10.10.3.11, 22:27:36, Ethernet0/0
O E2     10.60.0.0/16 [110/1] via 10.10.7.12, 22:22:03, Ethernet0/1
                      [110/1] via 10.10.3.11, 22:22:03, Ethernet0/0
O E2     10.70.0.0/16 [110/1] via 10.10.7.12, 22:22:03, Ethernet0/1
                      [110/1] via 10.10.3.11, 22:22:03, Ethernet0/0
      172.16.0.0/16 is variably subnetted, 11 subnets, 2 masks
C        172.16.1.0/24 is directly connected, Ethernet0/2
L        172.16.1.13/32 is directly connected, Ethernet0/2
O E2     172.16.2.0/24 [110/1] via 10.10.7.12, 22:27:44, Ethernet0/1
                       [110/1] via 10.10.3.11, 22:27:44, Ethernet0/0
O E2     172.16.3.0/24 [110/1] via 10.10.7.12, 22:27:36, Ethernet0/1
                       [110/1] via 10.10.3.11, 22:27:36, Ethernet0/0
O E2     172.16.4.0/24 [110/1] via 10.10.7.12, 22:27:36, Ethernet0/1
                       [110/1] via 10.10.3.11, 22:27:36, Ethernet0/0
O E2     172.16.5.0/24 [110/1] via 10.10.7.12, 22:27:36, Ethernet0/1
                       [110/1] via 10.10.3.11, 22:27:36, Ethernet0/0
O E2     172.16.6.0/24 [110/1] via 10.10.7.12, 22:27:36, Ethernet0/1
                       [110/1] via 10.10.3.11, 22:27:36, Ethernet0/0
O E2     172.16.7.0/24 [110/1] via 10.10.7.12, 22:27:36, Ethernet0/1
                       [110/1] via 10.10.3.11, 22:27:36, Ethernet0/0
O E2     172.16.8.0/24 [110/1] via 10.10.7.12, 22:27:36, Ethernet0/1
                       [110/1] via 10.10.3.11, 22:27:36, Ethernet0/0
O E2     172.16.9.0/24 [110/1] via 10.10.7.12, 22:27:36, Ethernet0/1
                       [110/1] via 10.10.3.11, 22:27:36, Ethernet0/0
O E2     172.16.10.0/24 [110/1] via 10.10.7.12, 22:27:36, Ethernet0/1
                        [110/1] via 10.10.3.11, 22:27:36, Ethernet0/0
R13#
R13#
R13#
R13#
R13#show ip bgp
BGP table version is 249, local router ID is 10.10.0.13
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter,
              x best-external, a additional-path, c RIB-compressed,
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
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
 r>i 10.20.0.0/16     172.16.2.31              0    100      0 301 101 ?
 r                    172.16.1.21              0     50      0 101 ?
 r>i 10.30.0.0/16     172.16.2.31              0    100      0 301 ?
 r                    172.16.1.21                    50      0 101 301 ?
 r>i 10.40.0.41/32    172.16.2.31              0    100      0 301 520 ?
 r                    172.16.1.21                    50      0 101 520 ?
 r>i 10.40.0.42/32    172.16.2.31              0    100      0 301 520 ?
 r                    172.16.1.21                    50      0 101 520 ?
 r>i 10.40.0.43/32    172.16.2.31              0    100      0 301 520 ?
 r                    172.16.1.21                    50      0 101 520 ?
 r>i 10.40.0.44/32    172.16.2.31              0    100      0 301 101 520 ?
 r                    172.16.1.21                    50      0 101 520 ?
 r>i 10.40.1.0/24     172.16.2.31              0    100      0 301 101 520 ?
 r                    172.16.1.21                    50      0 101 520 ?
 r>i 10.40.2.0/24     172.16.2.31              0    100      0 301 520 ?
 r                    172.16.1.21                    50      0 101 520 ?
 r>i 10.40.3.0/24     172.16.2.31              0    100      0 301 520 ?
 r                    172.16.1.21                    50      0 101 520 ?
 r>i 10.40.4.0/24     172.16.2.31              0    100      0 301 101 520 ?
 r                    172.16.1.21                    50      0 101 520 ?
 r>i 10.50.0.51/32    172.16.2.31              0    100      0 301 520 2042 ?
 r                    172.16.1.21                    50      0 101 520 2042 ?
 r>i 10.50.0.52/32    172.16.2.31              0    100      0 301 520 2042 ?
 r                    172.16.1.21                    50      0 101 520 2042 ?
 r>i 10.50.0.53/32    172.16.2.31              0    100      0 301 520 2042 ?
 r                    172.16.1.21                    50      0 101 520 2042 ?
 r>i 10.50.0.54/32    172.16.2.31              0    100      0 301 520 2042 ?
 r                    172.16.1.21                    50      0 101 520 2042 ?
 r>i 10.50.1.0/24     172.16.2.31              0    100      0 301 520 2042 ?
 r                    172.16.1.21                    50      0 101 520 2042 ?
 r>i 10.50.2.0/24     172.16.2.31              0    100      0 301 520 2042 ?
 r                    172.16.1.21                    50      0 101 520 2042 ?
 r>i 10.50.3.0/24     172.16.2.31              0    100      0 301 520 2042 ?
 r                    172.16.1.21                    50      0 101 520 2042 ?
 r>i 10.50.4.0/24     172.16.2.31              0    100      0 301 520 2042 ?
 r                    172.16.1.21                    50      0 101 520 2042 ?
 r>i 10.50.5.0/24     172.16.2.31              0    100      0 301 520 2042 ?
 r                    172.16.1.21                    50      0 101 520 2042 ?
 r>i 10.50.6.0/24     172.16.2.31              0    100      0 301 520 2042 ?
 r                    172.16.1.21                    50      0 101 520 2042 ?
 r>i 10.50.255.0/24   172.16.2.31              0    100      0 301 520 2042 ?
 r                    172.16.1.21                    50      0 101 520 2042 ?
 r>i 10.60.0.0/16     172.16.2.31              0    100      0 301 520 ?
 r                    172.16.1.21                    50      0 101 520 ?
 r>i 10.70.0.0/16     172.16.2.31              0    100      0 301 520 ?
 r                    172.16.1.21                    50      0 101 520 ?
 *>  172.16.1.0/24    0.0.0.0                  0         32768 i
 *                    172.16.1.21              0     50      0 101 i
 r>i 172.16.2.0/24    10.10.0.14               0    100      0 i
 r                    172.16.1.21                    50      0 101 301 i
 r>i 172.16.3.0/24    172.16.2.31              0    100      0 301 i
 r                    172.16.1.21              0     50      0 101 i
 r>i 172.16.4.0/24    172.16.2.31              0    100      0 301 i
 r                    172.16.1.21                    50      0 101 301 i
 r>i 172.16.5.0/24    172.16.2.31              0    100      0 301 101 i
 r                    172.16.1.21              0     50      0 101 i
 r>i 172.16.6.0/24    172.16.2.31              0    100      0 301 520 i
 r                    172.16.1.21                    50      0 101 520 i
 r>i 172.16.7.0/24    172.16.2.31              0    100      0 301 520 i
 r                    172.16.1.21                    50      0 101 520 i
 r>i 172.16.8.0/24    172.16.2.31              0    100      0 301 520 i
 r                    172.16.1.21                    50      0 101 520 i
 r>i 172.16.9.0/24    172.16.2.31              0    100      0 301 520 i
 r                    172.16.1.21                    50      0 101 520 i
 r>i 172.16.10.0/24   172.16.2.31              0    100      0 301 520 i
 r                    172.16.1.21                    50      0 101 520 i
R13#
R13#show ip bgp summary
BGP router identifier 10.10.0.13, local AS number 1001
BGP table version is 249, main routing table version 249
49 network entries using 6860 bytes of memory
98 path entries using 7840 bytes of memory
36/16 BGP path/bestpath attribute entries using 5184 bytes of memory
9 BGP AS-PATH entries using 216 bytes of memory
0 BGP route-map cache entries using 0 bytes of memory
0 BGP filter-list cache entries using 0 bytes of memory
BGP using 20100 total bytes of memory
BGP activity 78/29 prefixes, 257/159 paths, scan interval 60 secs

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
10.10.0.14      4         1001    1587    1602      249    0    0 22:52:50       48
172.16.1.21     4          101    1539    1540      249    0    0 22:52:50       33


```
</details>

### R14

<details>
  <summary>Конфигурация</summary>

```

R14#terminal length 0
R14#sh run
Building configuration...

Current configuration : 1557 bytes
!
! Last configuration change at 15:51:49 UTC Sat Jan 25 2025
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

Gateway of last resort is not set

      10.0.0.0/8 is variably subnetted, 42 subnets, 3 masks
O IA     10.10.0.11/32 [110/11] via 10.10.8.11, 1d02h, Ethernet0/1
O IA     10.10.0.12/32 [110/11] via 10.10.5.12, 1d02h, Ethernet0/0
O        10.10.0.13/32 [110/21] via 10.10.8.11, 1d02h, Ethernet0/1
                       [110/21] via 10.10.5.12, 1d02h, Ethernet0/0
C        10.10.0.14/32 is directly connected, Loopback1
O IA     10.10.0.15/32 [110/31] via 10.10.8.11, 1d02h, Ethernet0/1
                       [110/31] via 10.10.5.12, 1d02h, Ethernet0/0
O        10.10.0.16/32 [110/11] via 10.10.6.16, 1d02h, Ethernet0/3
O IA     10.10.1.0/24 [110/20] via 10.10.8.11, 1d02h, Ethernet0/1
                      [110/20] via 10.10.5.12, 1d02h, Ethernet0/0
O IA     10.10.2.0/24 [110/20] via 10.10.8.11, 1d02h, Ethernet0/1
                      [110/20] via 10.10.5.12, 1d02h, Ethernet0/0
O        10.10.3.0/24 [110/20] via 10.10.8.11, 1d02h, Ethernet0/1
O IA     10.10.4.0/24 [110/30] via 10.10.8.11, 1d02h, Ethernet0/1
                      [110/30] via 10.10.5.12, 1d02h, Ethernet0/0
C        10.10.5.0/24 is directly connected, Ethernet0/0
L        10.10.5.14/32 is directly connected, Ethernet0/0
C        10.10.6.0/24 is directly connected, Ethernet0/3
L        10.10.6.14/32 is directly connected, Ethernet0/3
O        10.10.7.0/24 [110/20] via 10.10.5.12, 1d02h, Ethernet0/0
C        10.10.8.0/24 is directly connected, Ethernet0/1
L        10.10.8.14/32 is directly connected, Ethernet0/1
O IA     10.10.9.0/24 [110/20] via 10.10.8.11, 1d02h, Ethernet0/1
                      [110/20] via 10.10.5.12, 1d02h, Ethernet0/0
O IA     10.10.255.0/24 [110/20] via 10.10.8.11, 1d02h, Ethernet0/1
                        [110/20] via 10.10.5.12, 1d02h, Ethernet0/0
B        10.20.0.0/16 [20/0] via 172.16.2.31, 22:19:13
B        10.30.0.0/16 [20/0] via 172.16.2.31, 22:17:03
B        10.40.0.41/32 [20/0] via 172.16.2.31, 22:28:28
B        10.40.0.42/32 [20/0] via 172.16.2.31, 22:28:28
B        10.40.0.43/32 [20/0] via 172.16.2.31, 22:28:28
B        10.40.0.44/32 [20/0] via 172.16.2.31, 22:28:28
B        10.40.1.0/24 [20/0] via 172.16.2.31, 22:28:28
B        10.40.2.0/24 [20/0] via 172.16.2.31, 22:28:28
B        10.40.3.0/24 [20/0] via 172.16.2.31, 22:28:28
B        10.40.4.0/24 [20/0] via 172.16.2.31, 22:28:28
B        10.50.0.51/32 [20/0] via 172.16.2.31, 22:28:28
B        10.50.0.52/32 [20/0] via 172.16.2.31, 22:28:28
B        10.50.0.53/32 [20/0] via 172.16.2.31, 22:28:28
B        10.50.0.54/32 [20/0] via 172.16.2.31, 22:28:28
B        10.50.1.0/24 [20/0] via 172.16.2.31, 22:28:28
B        10.50.2.0/24 [20/0] via 172.16.2.31, 22:28:28
B        10.50.3.0/24 [20/0] via 172.16.2.31, 22:28:28
B        10.50.4.0/24 [20/0] via 172.16.2.31, 22:28:28
B        10.50.5.0/24 [20/0] via 172.16.2.31, 22:28:28
B        10.50.6.0/24 [20/0] via 172.16.2.31, 22:28:28
B        10.50.255.0/24 [20/0] via 172.16.2.31, 22:28:28
B        10.60.0.0/16 [20/0] via 172.16.2.31, 22:22:55
B        10.70.0.0/16 [20/0] via 172.16.2.31, 22:22:55
      172.16.0.0/16 is variably subnetted, 11 subnets, 2 masks
O E2     172.16.1.0/24 [110/1] via 10.10.8.11, 22:35:40, Ethernet0/1
                       [110/1] via 10.10.5.12, 22:35:40, Ethernet0/0
C        172.16.2.0/24 is directly connected, Ethernet0/2
L        172.16.2.14/32 is directly connected, Ethernet0/2
B        172.16.3.0/24 [20/0] via 172.16.2.31, 22:28:28
B        172.16.4.0/24 [20/0] via 172.16.2.31, 22:28:28
B        172.16.5.0/24 [20/0] via 172.16.2.31, 22:28:28
B        172.16.6.0/24 [20/0] via 172.16.2.31, 22:28:28
B        172.16.7.0/24 [20/0] via 172.16.2.31, 22:28:28
B        172.16.8.0/24 [20/0] via 172.16.2.31, 22:28:28
B        172.16.9.0/24 [20/0] via 172.16.2.31, 22:28:28
B        172.16.10.0/24 [20/0] via 172.16.2.31, 22:28:28
R14#
R14#
R14#
R14#
R14#show ip bgp
BGP table version is 328, local router ID is 10.10.0.14
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter,
              x best-external, a additional-path, c RIB-compressed,
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 *>  10.10.0.11/32    10.10.8.11              11         32768 ?
 * i                  10.10.3.11              11    100      0 ?
 *>  10.10.0.12/32    10.10.5.12              11         32768 ?
 * i                  10.10.7.12              11    100      0 ?
 *>  10.10.0.13/32    10.10.5.12              21         32768 ?
 * i                  10.10.0.13               0    100      0 ?
 *>  10.10.0.14/32    0.0.0.0                  0         32768 ?
 * i                  10.10.3.11              21    100      0 ?
 *>  10.10.0.15/32    10.10.5.12              31         32768 ?
 * i                  10.10.4.15              11    100      0 ?
 *>  10.10.0.16/32    10.10.6.16              11         32768 ?
 * i                  10.10.3.11              31    100      0 ?
 *>  10.10.1.0/24     10.10.5.12              20         32768 ?
 * i                  10.10.3.11              20    100      0 ?
 *>  10.10.2.0/24     10.10.5.12              20         32768 ?
 * i                  10.10.3.11              20    100      0 ?
 *>  10.10.3.0/24     10.10.8.11              20         32768 ?
 * i                  10.10.0.13               0    100      0 ?
 *>  10.10.4.0/24     10.10.5.12              30         32768 ?
 * i                  10.10.0.13               0    100      0 ?
 *>  10.10.5.0/24     0.0.0.0                  0         32768 ?
 * i                  10.10.7.12              20    100      0 ?
 *>  10.10.6.0/24     0.0.0.0                  0         32768 ?
 * i                  10.10.3.11              30    100      0 ?
 *>  10.10.7.0/24     10.10.5.12              20         32768 ?
 * i                  10.10.0.13               0    100      0 ?
 *>  10.10.8.0/24     0.0.0.0                  0         32768 ?
 * i                  10.10.3.11              20    100      0 ?
 *>  10.10.9.0/24     10.10.5.12              20         32768 ?
 * i                  10.10.3.11              20    100      0 ?
 *>  10.10.255.0/24   10.10.5.12              20         32768 ?
 * i                  10.10.3.11              20    100      0 ?
 *>  10.20.0.0/16     172.16.2.31                            0 301 101 ?
 *>  10.30.0.0/16     172.16.2.31              0             0 301 ?
 *>  10.40.0.41/32    172.16.2.31                            0 301 520 ?
 *>  10.40.0.42/32    172.16.2.31                            0 301 520 ?
 *>  10.40.0.43/32    172.16.2.31                            0 301 520 ?
 *>  10.40.0.44/32    172.16.2.31                            0 301 101 520 ?
 *>  10.40.1.0/24     172.16.2.31                            0 301 101 520 ?
 *>  10.40.2.0/24     172.16.2.31                            0 301 520 ?
 *>  10.40.3.0/24     172.16.2.31                            0 301 520 ?
 *>  10.40.4.0/24     172.16.2.31                            0 301 101 520 ?
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
 *>  10.60.0.0/16     172.16.2.31                            0 301 520 ?
 *>  10.70.0.0/16     172.16.2.31                            0 301 520 ?
 r   172.16.1.0/24    172.16.2.31                            0 301 101 i
 r>i                  10.10.0.13               0    100      0 i
 *   172.16.2.0/24    172.16.2.31              0             0 301 i
 *>                   0.0.0.0                  0         32768 i
 *>  172.16.3.0/24    172.16.2.31              0             0 301 i
 *>  172.16.4.0/24    172.16.2.31              0             0 301 i
 *>  172.16.5.0/24    172.16.2.31                            0 301 101 i
 *>  172.16.6.0/24    172.16.2.31                            0 301 520 i
 *>  172.16.7.0/24    172.16.2.31                            0 301 520 i
 *>  172.16.8.0/24    172.16.2.31                            0 301 520 i
 *>  172.16.9.0/24    172.16.2.31                            0 301 520 i
 *>  172.16.10.0/24   172.16.2.31                            0 301 520 i
R14#
R14#
R14#
R14#
R14#show ip bgp summary
BGP router identifier 10.10.0.14, local AS number 1001
BGP table version is 328, main routing table version 328
49 network entries using 6860 bytes of memory
67 path entries using 5360 bytes of memory
22/16 BGP path/bestpath attribute entries using 3168 bytes of memory
5 BGP AS-PATH entries using 120 bytes of memory
0 BGP route-map cache entries using 0 bytes of memory
0 BGP filter-list cache entries using 0 bytes of memory
BGP using 15508 total bytes of memory
BGP activity 49/0 prefixes, 276/209 paths, scan interval 60 secs

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
10.10.0.13      4         1001    1601    1587      328    0    0 22:52:20       17
172.16.2.31     4          301    1506    1503      328    0    0 22:28:32       33


```
</details>

### R21

<details>
  <summary>Конфигурация</summary>

```

R21#terminal length 0
R21#sh run
Building configuration...

Current configuration : 1520 bytes
!
! Last configuration change at 15:51:02 UTC Sat Jan 25 2025
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
B        10.10.0.11/32 [20/11] via 172.16.1.13, 22:46:26
B        10.10.0.12/32 [20/11] via 172.16.1.13, 22:46:26
B        10.10.0.13/32 [20/0] via 172.16.1.13, 22:46:26
B        10.10.0.14/32 [20/21] via 172.16.1.13, 22:46:26
B        10.10.0.15/32 [20/11] via 172.16.1.13, 22:46:26
B        10.10.0.16/32 [20/31] via 172.16.1.13, 22:46:26
B        10.10.1.0/24 [20/20] via 172.16.1.13, 22:46:26
B        10.10.2.0/24 [20/20] via 172.16.1.13, 22:46:26
B        10.10.3.0/24 [20/0] via 172.16.1.13, 22:46:26
B        10.10.4.0/24 [20/0] via 172.16.1.13, 22:46:26
B        10.10.5.0/24 [20/20] via 172.16.1.13, 22:46:26
B        10.10.6.0/24 [20/30] via 172.16.1.13, 22:46:26
B        10.10.7.0/24 [20/0] via 172.16.1.13, 22:46:26
B        10.10.8.0/24 [20/20] via 172.16.1.13, 22:46:26
B        10.10.9.0/24 [20/20] via 172.16.1.13, 22:46:26
B        10.10.255.0/24 [20/20] via 172.16.1.13, 22:46:26
S        10.20.0.0/16 is directly connected, Null0
C        10.20.0.21/32 is directly connected, Loopback1
B        10.30.0.0/16 [20/0] via 172.16.3.31, 22:18:47
B        10.40.0.41/32 [20/0] via 172.16.5.41, 1d00h
B        10.40.0.42/32 [20/20] via 172.16.5.41, 1d00h
B        10.40.0.43/32 [20/30] via 172.16.5.41, 1d00h
B        10.40.0.44/32 [20/20] via 172.16.5.41, 1d00h
B        10.40.1.0/24 [20/0] via 172.16.5.41, 1d00h
B        10.40.2.0/24 [20/0] via 172.16.5.41, 1d00h
B        10.40.3.0/24 [20/20] via 172.16.5.41, 1d00h
B        10.40.4.0/24 [20/20] via 172.16.5.41, 1d00h
B        10.50.0.51/32 [20/0] via 172.16.5.41, 1d00h
B        10.50.0.52/32 [20/0] via 172.16.5.41, 1d00h
B        10.50.0.53/32 [20/0] via 172.16.5.41, 1d00h
B        10.50.0.54/32 [20/0] via 172.16.5.41, 1d00h
B        10.50.1.0/24 [20/0] via 172.16.5.41, 1d00h
B        10.50.2.0/24 [20/0] via 172.16.5.41, 1d00h
B        10.50.3.0/24 [20/0] via 172.16.5.41, 1d00h
B        10.50.4.0/24 [20/0] via 172.16.5.41, 1d00h
B        10.50.5.0/24 [20/0] via 172.16.5.41, 1d00h
B        10.50.6.0/24 [20/0] via 172.16.5.41, 1d00h
B        10.50.255.0/24 [20/0] via 172.16.5.41, 1d00h
B        10.60.0.0/16 [20/0] via 172.16.5.41, 22:24:39
B        10.70.0.0/16 [20/0] via 172.16.5.41, 22:24:39
      172.16.0.0/16 is variably subnetted, 13 subnets, 2 masks
C        172.16.1.0/24 is directly connected, Ethernet0/0
L        172.16.1.21/32 is directly connected, Ethernet0/0
B        172.16.2.0/24 [20/0] via 172.16.3.31, 1d02h
C        172.16.3.0/24 is directly connected, Ethernet0/1
L        172.16.3.21/32 is directly connected, Ethernet0/1
B        172.16.4.0/24 [20/0] via 172.16.3.31, 1d02h
C        172.16.5.0/24 is directly connected, Ethernet0/2
L        172.16.5.21/32 is directly connected, Ethernet0/2
B        172.16.6.0/24 [20/0] via 172.16.5.41, 1d00h
B        172.16.7.0/24 [20/0] via 172.16.5.41, 1d00h
B        172.16.8.0/24 [20/0] via 172.16.5.41, 1d00h
B        172.16.9.0/24 [20/0] via 172.16.5.41, 1d00h
B        172.16.10.0/24 [20/0] via 172.16.5.41, 1d00h
R21#
R21#
R21#
R21#
R21#show ip bgp
BGP table version is 78, local router ID is 10.20.0.21
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter,
              x best-external, a additional-path, c RIB-compressed,
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
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
 *   10.30.0.0/16     172.16.1.13                            0 1001 301 ?
 *                    172.16.5.41                            0 520 301 ?
 *>                   172.16.3.31              0             0 301 ?
 *   10.40.0.41/32    172.16.1.13                            0 1001 301 520 ?
 *                    172.16.3.31                            0 301 520 ?
 *>                   172.16.5.41                            0 520 ?
 *   10.40.0.42/32    172.16.1.13                            0 1001 301 520 ?
 *                    172.16.3.31                            0 301 520 ?
 *>                   172.16.5.41             20             0 520 ?
 *   10.40.0.43/32    172.16.1.13                            0 1001 301 520 ?
 *                    172.16.3.31                            0 301 520 ?
 *>                   172.16.5.41             30             0 520 ?
 *>  10.40.0.44/32    172.16.5.41             20             0 520 ?
 *>  10.40.1.0/24     172.16.5.41                            0 520 ?
 *   10.40.2.0/24     172.16.1.13                            0 1001 301 520 ?
 *                    172.16.3.31                            0 301 520 ?
 *>                   172.16.5.41                            0 520 ?
 *   10.40.3.0/24     172.16.1.13                            0 1001 301 520 ?
 *                    172.16.3.31                            0 301 520 ?
 *>                   172.16.5.41             20             0 520 ?
 *>  10.40.4.0/24     172.16.5.41             20             0 520 ?
 *   10.50.0.51/32    172.16.1.13                            0 1001 301 520 2042 ?
 *>                   172.16.5.41                            0 520 2042 ?
 *                    172.16.3.31                            0 301 520 2042 ?
 *   10.50.0.52/32    172.16.1.13                            0 1001 301 520 2042 ?
 *>                   172.16.5.41                            0 520 2042 ?
 *                    172.16.3.31                            0 301 520 2042 ?
 *   10.50.0.53/32    172.16.1.13                            0 1001 301 520 2042 ?
 *>                   172.16.5.41                            0 520 2042 ?
 *                    172.16.3.31                            0 301 520 2042 ?
 *   10.50.0.54/32    172.16.1.13                            0 1001 301 520 2042 ?
 *>                   172.16.5.41                            0 520 2042 ?
 *                    172.16.3.31                            0 301 520 2042 ?
 *   10.50.1.0/24     172.16.1.13                            0 1001 301 520 2042 ?
 *>                   172.16.5.41                            0 520 2042 ?
 *                    172.16.3.31                            0 301 520 2042 ?
 *   10.50.2.0/24     172.16.1.13                            0 1001 301 520 2042 ?
 *>                   172.16.5.41                            0 520 2042 ?
 *                    172.16.3.31                            0 301 520 2042 ?
 *   10.50.3.0/24     172.16.1.13                            0 1001 301 520 2042 ?
 *>                   172.16.5.41                            0 520 2042 ?
 *                    172.16.3.31                            0 301 520 2042 ?
 *   10.50.4.0/24     172.16.1.13                            0 1001 301 520 2042 ?
 *>                   172.16.5.41                            0 520 2042 ?
 *                    172.16.3.31                            0 301 520 2042 ?
 *   10.50.5.0/24     172.16.1.13                            0 1001 301 520 2042 ?
 *>                   172.16.5.41                            0 520 2042 ?
 *                    172.16.3.31                            0 301 520 2042 ?
 *   10.50.6.0/24     172.16.1.13                            0 1001 301 520 2042 ?
 *>                   172.16.5.41                            0 520 2042 ?
 *                    172.16.3.31                            0 301 520 2042 ?
 *   10.50.255.0/24   172.16.1.13                            0 1001 301 520 2042 ?
 *>                   172.16.5.41                            0 520 2042 ?
 *                    172.16.3.31                            0 301 520 2042 ?
 *   10.60.0.0/16     172.16.1.13                            0 1001 301 520 ?
 *                    172.16.3.31                            0 301 520 ?
 *>                   172.16.5.41                            0 520 ?
 *   10.70.0.0/16     172.16.1.13                            0 1001 301 520 ?
 *                    172.16.3.31                            0 301 520 ?
 *>                   172.16.5.41                            0 520 ?
 *   172.16.1.0/24    172.16.1.13              0             0 1001 i
 *>                   0.0.0.0                  0         32768 i
 *   172.16.2.0/24    172.16.1.13                            0 1001 i
 *                    172.16.5.41                            0 520 301 i
 *>                   172.16.3.31              0             0 301 i
 *   172.16.3.0/24    172.16.1.13                            0 1001 301 i
 *                    172.16.3.31              0             0 301 i
 *>                   0.0.0.0                  0         32768 i
 *   172.16.4.0/24    172.16.1.13                            0 1001 301 i
 *                    172.16.5.41                            0 520 i
 *>                   172.16.3.31              0             0 301 i
 *   172.16.5.0/24    172.16.5.41              0             0 520 i
 *>                   0.0.0.0                  0         32768 i
 *   172.16.6.0/24    172.16.1.13                            0 1001 301 520 i
 *>                   172.16.5.41                            0 520 i
 *                    172.16.3.31                            0 301 520 i
 *   172.16.7.0/24    172.16.1.13                            0 1001 301 520 i
 *>                   172.16.5.41                            0 520 i
 *                    172.16.3.31                            0 301 520 i
 *   172.16.8.0/24    172.16.1.13                            0 1001 301 520 i
 *                    172.16.3.31                            0 301 520 i
 *>                   172.16.5.41                            0 520 i
 *   172.16.9.0/24    172.16.1.13                            0 1001 301 520 i
 *                    172.16.3.31                            0 301 520 i
 *>                   172.16.5.41                            0 520 i
 *   172.16.10.0/24   172.16.1.13                            0 1001 301 520 i
 *                    172.16.3.31                            0 301 520 i
 *>                   172.16.5.41                            0 520 i
R21#
R21#
R21#
R21#
R21#show ip bgp summary
BGP router identifier 10.20.0.21, local AS number 101
BGP table version is 78, main routing table version 78
49 network entries using 6860 bytes of memory
121 path entries using 9680 bytes of memory
29/15 BGP path/bestpath attribute entries using 4176 bytes of memory
11 BGP AS-PATH entries using 280 bytes of memory
0 BGP route-map cache entries using 0 bytes of memory
0 BGP filter-list cache entries using 0 bytes of memory
BGP using 20996 total bytes of memory
BGP activity 52/3 prefixes, 216/95 paths, scan interval 60 secs

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
172.16.1.13     4         1001    1541    1541       78    0    0 22:54:00       44
172.16.3.31     4          301    1744    1752       78    0    0 1d02h          43
172.16.5.41     4          520    1745    1749       78    0    0 1d02h          30


```
</details>

### R31

<details>
  <summary>Конфигурация</summary>

```

R31#terminal length 0
R31#sh run
Building configuration...

Current configuration : 1520 bytes
!
! Last configuration change at 15:53:12 UTC Sat Jan 25 2025
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
 neighbor 172.16.3.21 remote-as 101
 neighbor 172.16.4.44 remote-as 520
!
ip forward-protocol nd
!
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
B        10.10.0.11/32 [20/11] via 172.16.2.14, 22:30:56
B        10.10.0.12/32 [20/11] via 172.16.2.14, 22:30:56
B        10.10.0.13/32 [20/21] via 172.16.2.14, 22:30:56
B        10.10.0.14/32 [20/0] via 172.16.2.14, 22:30:56
B        10.10.0.15/32 [20/31] via 172.16.2.14, 22:30:56
B        10.10.0.16/32 [20/11] via 172.16.2.14, 22:30:56
B        10.10.1.0/24 [20/20] via 172.16.2.14, 22:30:56
B        10.10.2.0/24 [20/20] via 172.16.2.14, 22:30:56
B        10.10.3.0/24 [20/20] via 172.16.2.14, 22:30:56
B        10.10.4.0/24 [20/30] via 172.16.2.14, 22:30:56
B        10.10.5.0/24 [20/0] via 172.16.2.14, 22:30:56
B        10.10.6.0/24 [20/0] via 172.16.2.14, 22:30:56
B        10.10.7.0/24 [20/20] via 172.16.2.14, 22:30:56
B        10.10.8.0/24 [20/0] via 172.16.2.14, 22:30:56
B        10.10.9.0/24 [20/20] via 172.16.2.14, 22:30:56
B        10.10.255.0/24 [20/20] via 172.16.2.14, 22:30:56
B        10.20.0.0/16 [20/0] via 172.16.3.21, 22:21:41
S        10.30.0.0/16 is directly connected, Null0
C        10.30.0.31/32 is directly connected, Loopback1
B        10.40.0.41/32 [20/20] via 172.16.4.44, 1d00h
B        10.40.0.42/32 [20/30] via 172.16.4.44, 1d00h
B        10.40.0.43/32 [20/20] via 172.16.4.44, 1d00h
B        10.40.0.44/32 [20/0] via 172.16.3.21, 1d00h
B        10.40.1.0/24 [20/0] via 172.16.3.21, 1d00h
B        10.40.2.0/24 [20/20] via 172.16.4.44, 1d00h
B        10.40.3.0/24 [20/20] via 172.16.4.44, 1d00h
B        10.40.4.0/24 [20/0] via 172.16.3.21, 1d00h
B        10.50.0.51/32 [20/0] via 172.16.4.44, 1d00h
B        10.50.0.52/32 [20/0] via 172.16.4.44, 1d00h
B        10.50.0.53/32 [20/0] via 172.16.4.44, 1d00h
B        10.50.0.54/32 [20/0] via 172.16.4.44, 1d00h
B        10.50.1.0/24 [20/0] via 172.16.4.44, 1d00h
B        10.50.2.0/24 [20/0] via 172.16.4.44, 1d00h
B        10.50.3.0/24 [20/0] via 172.16.4.44, 1d00h
B        10.50.4.0/24 [20/0] via 172.16.4.44, 1d00h
B        10.50.5.0/24 [20/0] via 172.16.4.44, 1d00h
B        10.50.6.0/24 [20/0] via 172.16.4.44, 1d00h
B        10.50.255.0/24 [20/0] via 172.16.4.44, 1d00h
B        10.60.0.0/16 [20/0] via 172.16.4.44, 22:25:22
B        10.70.0.0/16 [20/0] via 172.16.4.44, 22:25:22
      172.16.0.0/16 is variably subnetted, 13 subnets, 2 masks
B        172.16.1.0/24 [20/0] via 172.16.3.21, 1d02h
C        172.16.2.0/24 is directly connected, Ethernet0/0
L        172.16.2.31/32 is directly connected, Ethernet0/0
C        172.16.3.0/24 is directly connected, Ethernet0/1
L        172.16.3.31/32 is directly connected, Ethernet0/1
C        172.16.4.0/24 is directly connected, Ethernet0/2
L        172.16.4.31/32 is directly connected, Ethernet0/2
B        172.16.5.0/24 [20/0] via 172.16.3.21, 1d02h
B        172.16.6.0/24 [20/0] via 172.16.4.44, 1d02h
B        172.16.7.0/24 [20/0] via 172.16.4.44, 1d00h
B        172.16.8.0/24 [20/0] via 172.16.4.44, 1d00h
B        172.16.9.0/24 [20/0] via 172.16.4.44, 1d00h
B        172.16.10.0/24 [20/0] via 172.16.4.44, 1d00h
R31#
R31#
R31#
R31#
R31#show ip bgp
BGP table version is 115, local router ID is 10.30.0.31
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter,
              x best-external, a additional-path, c RIB-compressed,
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
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
 *>  10.40.0.41/32    172.16.4.44             20             0 520 ?
 *                    172.16.3.21                            0 101 520 ?
 *   10.40.0.42/32    172.16.3.21                            0 101 520 ?
 *>                   172.16.4.44             30             0 520 ?
 *>  10.40.0.43/32    172.16.4.44             20             0 520 ?
 *                    172.16.3.21                            0 101 520 ?
 *>  10.40.0.44/32    172.16.3.21                            0 101 520 ?
 *>  10.40.1.0/24     172.16.3.21                            0 101 520 ?
 *>  10.40.2.0/24     172.16.4.44             20             0 520 ?
 *                    172.16.3.21                            0 101 520 ?
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
 *   172.16.5.0/24    172.16.4.44                            0 520 i
 *>                   172.16.3.21              0             0 101 i
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
BGP table version is 115, main routing table version 115
49 network entries using 6860 bytes of memory
95 path entries using 7600 bytes of memory
24/17 BGP path/bestpath attribute entries using 3456 bytes of memory
8 BGP AS-PATH entries using 192 bytes of memory
0 BGP route-map cache entries using 0 bytes of memory
0 BGP filter-list cache entries using 0 bytes of memory
BGP using 18108 total bytes of memory
BGP activity 49/0 prefixes, 183/88 paths, scan interval 60 secs

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
172.16.2.14     4         1001    1505    1508      115    0    0 22:30:55       18
172.16.3.21     4          101    1752    1744      115    0    0 1d02h          46
172.16.4.44     4          520    1747    1744      115    0    0 1d02h          27


```
</details>

### R41

<details>
  <summary>Конфигурация</summary>

```

R41#terminal length 0
R41#sh run
Building configuration...

Current configuration : 1603 bytes
!
! Last configuration change at 13:59:35 UTC Sat Jan 25 2025
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
 bgp listen range 10.40.0.0/24 peer-group IBGP
 network 172.16.5.0 mask 255.255.255.0
 redistribute isis level-1-2
 neighbor IBGP peer-group
 neighbor IBGP remote-as 520
 neighbor IBGP update-source Loopback1
 neighbor IBGP route-reflector-client
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
Ethernet0/0                172.16.5.41     YES NVRAM  up                    up
Ethernet0/1                10.40.2.41      YES NVRAM  up                    up
Ethernet0/2                10.40.1.41      YES NVRAM  up                    up
Ethernet0/3                unassigned      YES NVRAM  down                  down
Ethernet1/0                unassigned      YES NVRAM  administratively down down
Ethernet1/1                unassigned      YES NVRAM  administratively down down
Ethernet1/2                unassigned      YES NVRAM  administratively down down
Ethernet1/3                unassigned      YES NVRAM  administratively down down
Loopback1                  10.40.0.41      YES NVRAM  up                    up
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

      10.0.0.0/8 is variably subnetted, 41 subnets, 3 masks
B        10.10.0.11/32 [20/0] via 172.16.5.21, 22:47:48
B        10.10.0.12/32 [20/0] via 172.16.5.21, 22:47:48
B        10.10.0.13/32 [20/0] via 172.16.5.21, 22:47:48
B        10.10.0.14/32 [20/0] via 172.16.5.21, 22:47:48
B        10.10.0.15/32 [20/0] via 172.16.5.21, 22:47:48
B        10.10.0.16/32 [20/0] via 172.16.5.21, 22:47:48
B        10.10.1.0/24 [20/0] via 172.16.5.21, 22:47:48
B        10.10.2.0/24 [20/0] via 172.16.5.21, 22:47:48
B        10.10.3.0/24 [20/0] via 172.16.5.21, 22:47:48
B        10.10.4.0/24 [20/0] via 172.16.5.21, 22:47:48
B        10.10.5.0/24 [20/0] via 172.16.5.21, 22:47:48
B        10.10.6.0/24 [20/0] via 172.16.5.21, 22:47:48
B        10.10.7.0/24 [20/0] via 172.16.5.21, 22:47:48
B        10.10.8.0/24 [20/0] via 172.16.5.21, 22:47:48
B        10.10.9.0/24 [20/0] via 172.16.5.21, 22:47:48
B        10.10.255.0/24 [20/0] via 172.16.5.21, 22:47:48
B        10.20.0.0/16 [20/0] via 172.16.5.21, 22:22:20
B        10.30.0.0/16 [200/0] via 172.16.4.31, 22:20:09
C        10.40.0.41/32 is directly connected, Loopback1
i L1     10.40.0.42/32 [115/20] via 10.40.2.42, 1d02h, Ethernet0/1
i L2     10.40.0.43/32 [115/30] via 10.40.2.42, 1d02h, Ethernet0/1
                       [115/30] via 10.40.1.44, 1d02h, Ethernet0/2
i L2     10.40.0.44/32 [115/20] via 10.40.1.44, 1d02h, Ethernet0/2
C        10.40.1.0/24 is directly connected, Ethernet0/2
L        10.40.1.41/32 is directly connected, Ethernet0/2
C        10.40.2.0/24 is directly connected, Ethernet0/1
L        10.40.2.41/32 is directly connected, Ethernet0/1
i L1     10.40.3.0/24 [115/20] via 10.40.2.42, 1d02h, Ethernet0/1
i L2     10.40.4.0/24 [115/20] via 10.40.1.44, 1d02h, Ethernet0/2
B        10.50.0.51/32 [200/1024640] via 172.16.6.52, 1d00h
B        10.50.0.52/32 [200/0] via 172.16.6.52, 1d00h
B        10.50.0.53/32 [200/1024640] via 172.16.6.52, 1d00h
B        10.50.0.54/32 [200/1536640] via 172.16.6.52, 1d00h
B        10.50.1.0/24 [200/1536000] via 172.16.6.52, 1d00h
B        10.50.2.0/24 [200/1536000] via 172.16.6.52, 1d00h
B        10.50.3.0/24 [200/0] via 172.16.6.52, 1d00h
B        10.50.4.0/24 [200/0] via 172.16.6.52, 1d00h
B        10.50.5.0/24 [200/1536000] via 172.16.6.52, 1d00h
B        10.50.6.0/24 [200/1536000] via 172.16.6.52, 1d00h
B        10.50.255.0/24 [200/1536000] via 172.16.6.52, 1d00h
B        10.60.0.0/16 [200/0] via 172.16.9.61, 22:26:01
B        10.70.0.0/16 [200/0] via 172.16.10.71, 22:26:01
      172.16.0.0/16 is variably subnetted, 11 subnets, 2 masks
B        172.16.1.0/24 [20/0] via 172.16.5.21, 1d02h
B        172.16.2.0/24 [200/0] via 172.16.4.31, 1d00h
B        172.16.3.0/24 [20/0] via 172.16.5.21, 1d02h
B        172.16.4.0/24 [200/0] via 10.40.0.44, 1d00h
C        172.16.5.0/24 is directly connected, Ethernet0/0
L        172.16.5.41/32 is directly connected, Ethernet0/0
B        172.16.6.0/24 [200/0] via 10.40.0.44, 1d00h
B        172.16.7.0/24 [200/0] via 10.40.0.43, 1d00h
B        172.16.8.0/24 [200/0] via 10.40.0.43, 1d00h
B        172.16.9.0/24 [200/0] via 10.40.0.42, 1d00h
B        172.16.10.0/24 [200/0] via 10.40.0.42, 1d00h
R41#
R41#
R41#
R41#
R41#show ip bgp
BGP table version is 75, local router ID is 10.40.0.41
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter,
              x best-external, a additional-path, c RIB-compressed,
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 * i 10.10.0.11/32    172.16.4.31              0    100      0 301 1001 ?
 *>                   172.16.5.21                            0 101 1001 ?
 * i 10.10.0.12/32    172.16.4.31              0    100      0 301 1001 ?
 *>                   172.16.5.21                            0 101 1001 ?
 * i 10.10.0.13/32    172.16.4.31              0    100      0 301 1001 ?
 *>                   172.16.5.21                            0 101 1001 ?
 * i 10.10.0.14/32    172.16.4.31              0    100      0 301 1001 ?
 *>                   172.16.5.21                            0 101 1001 ?
 * i 10.10.0.15/32    172.16.4.31              0    100      0 301 1001 ?
 *>                   172.16.5.21                            0 101 1001 ?
 * i 10.10.0.16/32    172.16.4.31              0    100      0 301 1001 ?
 *>                   172.16.5.21                            0 101 1001 ?
 * i 10.10.1.0/24     172.16.4.31              0    100      0 301 1001 ?
 *>                   172.16.5.21                            0 101 1001 ?
 * i 10.10.2.0/24     172.16.4.31              0    100      0 301 1001 ?
 *>                   172.16.5.21                            0 101 1001 ?
 * i 10.10.3.0/24     172.16.4.31              0    100      0 301 1001 ?
 *>                   172.16.5.21                            0 101 1001 ?
 * i 10.10.4.0/24     172.16.4.31              0    100      0 301 1001 ?
 *>                   172.16.5.21                            0 101 1001 ?
 * i 10.10.5.0/24     172.16.4.31              0    100      0 301 1001 ?
 *>                   172.16.5.21                            0 101 1001 ?
 * i 10.10.6.0/24     172.16.4.31              0    100      0 301 1001 ?
 *>                   172.16.5.21                            0 101 1001 ?
 * i 10.10.7.0/24     172.16.4.31              0    100      0 301 1001 ?
 *>                   172.16.5.21                            0 101 1001 ?
 * i 10.10.8.0/24     172.16.4.31              0    100      0 301 1001 ?
 *>                   172.16.5.21                            0 101 1001 ?
 * i 10.10.9.0/24     172.16.4.31              0    100      0 301 1001 ?
 *>                   172.16.5.21                            0 101 1001 ?
 * i 10.10.255.0/24   172.16.4.31              0    100      0 301 1001 ?
 *>                   172.16.5.21                            0 101 1001 ?
 *>  10.20.0.0/16     172.16.5.21              0             0 101 ?
 *   10.30.0.0/16     172.16.5.21                            0 101 301 ?
 *>i                  172.16.4.31              0    100      0 301 ?
 r>i 10.40.0.41/32    10.40.3.42              30    100      0 ?
 * i 10.40.0.42/32    10.40.3.42              20    100      0 ?
 *>                   10.40.2.42              20         32768 ?
 * i 10.40.0.43/32    10.40.4.43              20    100      0 ?
 * i                  10.40.3.43              20    100      0 ?
 *>                   10.40.1.44              30         32768 ?
 * i 10.40.0.44/32    10.40.4.44              20    100      0 ?
 *>                   10.40.1.44              20         32768 ?
 r>i 10.40.1.0/24     10.40.4.44              20    100      0 ?
 r>i 10.40.2.0/24     10.40.3.42              20    100      0 ?
 * i 10.40.3.0/24     10.40.4.43              20    100      0 ?
 *>                   10.40.2.42              20         32768 ?
 * i 10.40.4.0/24     10.40.3.43              20    100      0 ?
 *>                   10.40.1.44              20         32768 ?
 * i 10.50.0.51/32    172.16.7.52        1024640    100      0 2042 ?
 *>i                  172.16.6.52        1024640    100      0 2042 ?
 * i 10.50.0.52/32    172.16.7.52              0    100      0 2042 ?
 *>i                  172.16.6.52              0    100      0 2042 ?
 * i 10.50.0.53/32    172.16.7.52        1024640    100      0 2042 ?
 *>i                  172.16.6.52        1024640    100      0 2042 ?
 * i 10.50.0.54/32    172.16.7.52        1536640    100      0 2042 ?
 *>i                  172.16.6.52        1536640    100      0 2042 ?
 * i 10.50.1.0/24     172.16.7.52        1536000    100      0 2042 ?
 *>i                  172.16.6.52        1536000    100      0 2042 ?
 * i 10.50.2.0/24     172.16.7.52        1536000    100      0 2042 ?
 *>i                  172.16.6.52        1536000    100      0 2042 ?
 * i 10.50.3.0/24     172.16.7.52              0    100      0 2042 ?
 *>i                  172.16.6.52              0    100      0 2042 ?
 * i 10.50.4.0/24     172.16.7.52              0    100      0 2042 ?
 *>i                  172.16.6.52              0    100      0 2042 ?
 * i 10.50.5.0/24     172.16.7.52        1536000    100      0 2042 ?
 *>i                  172.16.6.52        1536000    100      0 2042 ?
 * i 10.50.6.0/24     172.16.7.52        1536000    100      0 2042 ?
 *>i                  172.16.6.52        1536000    100      0 2042 ?
 * i 10.50.255.0/24   172.16.7.52        1536000    100      0 2042 ?
 *>i                  172.16.6.52        1536000    100      0 2042 ?
 *>i 10.60.0.0/16     172.16.9.61              0    100      0 ?
 * i 10.70.0.0/16     172.16.8.71              0    100      0 ?
 *>i                  172.16.10.71             0    100      0 ?
 *>  172.16.1.0/24    172.16.5.21              0             0 101 i
 *>i 172.16.2.0/24    172.16.4.31              0    100      0 301 i
 *                    172.16.5.21                            0 101 301 i
 * i 172.16.3.0/24    172.16.4.31              0    100      0 301 i
 *>                   172.16.5.21              0             0 101 i
 *>i 172.16.4.0/24    10.40.0.44               0    100      0 i
 *                    172.16.5.21                            0 101 301 i
 *   172.16.5.0/24    172.16.5.21              0             0 101 i
 *>                   0.0.0.0                  0         32768 i
 *>i 172.16.6.0/24    10.40.0.44               0    100      0 i
 *>i 172.16.7.0/24    10.40.0.43               0    100      0 i
 *>i 172.16.8.0/24    10.40.0.43               0    100      0 i
 *>i 172.16.9.0/24    10.40.0.42               0    100      0 i
 *>i 172.16.10.0/24   10.40.0.42               0    100      0 i
R41#
R41#
R41#
R41#
R41#show ip bgp summary
BGP router identifier 10.40.0.41, local AS number 520
BGP table version is 75, main routing table version 75
49 network entries using 6860 bytes of memory
88 path entries using 7040 bytes of memory
19/16 BGP path/bestpath attribute entries using 2736 bytes of memory
6 BGP AS-PATH entries using 144 bytes of memory
0 BGP route-map cache entries using 0 bytes of memory
0 BGP filter-list cache entries using 0 bytes of memory
BGP using 16780 total bytes of memory
BGP activity 51/2 prefixes, 128/40 paths, scan interval 60 secs

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
*10.40.0.42     4          520    1648    1659       75    0    0 1d00h           6
*10.40.0.43     4          520    1652    1661       75    0    0 1d00h          19
*10.40.0.44     4          520    1652    1656       75    0    0 1d00h          34
172.16.5.21     4          101    1751    1746       75    0    0 1d02h          23
* Dynamically created based on a listen range command
Dynamically created neighbors: 3, Subnet ranges: 1

BGP peergroup IBGP listen range group members:
  10.40.0.0/24

Total dynamically created neighbors: 3/(100 max), Subnet ranges: 1


```
</details>

### R42

<details>
  <summary>Конфигурация</summary>

```

R42#terminal length 0
R42#sh run
Building configuration...

Current configuration : 1640 bytes
!
! Last configuration change at 15:49:33 UTC Sat Jan 25 2025
!
version 15.4
service timestamps debug datetime msec
service timestamps log datetime msec
no service password-encryption
!
hostname R42
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
 ip address 10.40.0.42 255.255.255.255
 ip router isis
!
interface Ethernet0/0
 ip address 10.40.2.42 255.255.255.0
 ip router isis
!
interface Ethernet0/1
 ip address 172.16.9.42 255.255.255.0
!
interface Ethernet0/2
 ip address 10.40.3.42 255.255.255.0
 ip router isis
!
interface Ethernet0/3
 ip address 172.16.10.42 255.255.255.0
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
 net 49.2222.0000.0000.0042.00
!
router bgp 520
 bgp log-neighbor-changes
 network 172.16.9.0 mask 255.255.255.0
 network 172.16.10.0 mask 255.255.255.0
 redistribute static
 redistribute isis level-1-2
 neighbor 10.40.0.41 remote-as 520
 neighbor 10.40.0.41 update-source Loopback1
!
ip forward-protocol nd
!
!
no ip http server
no ip http secure-server
ip route 10.60.0.0 255.255.0.0 172.16.9.61
ip route 10.70.0.0 255.255.0.0 172.16.10.71
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

R42#
R42#
R42#
R42#show ip interface brief
Interface                  IP-Address      OK? Method Status                Protocol
Ethernet0/0                10.40.2.42      YES NVRAM  up                    up
Ethernet0/1                172.16.9.42     YES NVRAM  up                    up
Ethernet0/2                10.40.3.42      YES NVRAM  up                    up
Ethernet0/3                172.16.10.42    YES NVRAM  up                    up
Ethernet1/0                unassigned      YES NVRAM  administratively down down
Ethernet1/1                unassigned      YES NVRAM  administratively down down
Ethernet1/2                unassigned      YES NVRAM  administratively down down
Ethernet1/3                unassigned      YES NVRAM  administratively down down
Loopback1                  10.40.0.42      YES NVRAM  up                    up
R42#
R42#
R42#
R42#
R42#show ip route
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
B        10.10.0.11/32 [200/0] via 172.16.5.21, 22:48:24
B        10.10.0.12/32 [200/0] via 172.16.5.21, 22:48:24
B        10.10.0.13/32 [200/0] via 172.16.5.21, 22:48:24
B        10.10.0.14/32 [200/0] via 172.16.5.21, 22:48:24
B        10.10.0.15/32 [200/0] via 172.16.5.21, 22:48:24
B        10.10.0.16/32 [200/0] via 172.16.5.21, 22:48:24
B        10.10.1.0/24 [200/0] via 172.16.5.21, 22:48:24
B        10.10.2.0/24 [200/0] via 172.16.5.21, 22:48:24
B        10.10.3.0/24 [200/0] via 172.16.5.21, 22:48:24
B        10.10.4.0/24 [200/0] via 172.16.5.21, 22:48:24
B        10.10.5.0/24 [200/0] via 172.16.5.21, 22:48:24
B        10.10.6.0/24 [200/0] via 172.16.5.21, 22:48:24
B        10.10.7.0/24 [200/0] via 172.16.5.21, 22:48:24
B        10.10.8.0/24 [200/0] via 172.16.5.21, 22:48:24
B        10.10.9.0/24 [200/0] via 172.16.5.21, 22:48:24
B        10.10.255.0/24 [200/0] via 172.16.5.21, 22:48:24
B        10.20.0.0/16 [200/0] via 172.16.5.21, 22:22:56
B        10.30.0.0/16 [200/0] via 172.16.4.31, 22:20:45
i L1     10.40.0.41/32 [115/20] via 10.40.2.41, 1d02h, Ethernet0/0
C        10.40.0.42/32 is directly connected, Loopback1
i L2     10.40.0.43/32 [115/20] via 10.40.3.43, 1d02h, Ethernet0/2
i L2     10.40.0.44/32 [115/30] via 10.40.3.43, 1d02h, Ethernet0/2
                       [115/30] via 10.40.2.41, 1d02h, Ethernet0/0
i L1     10.40.1.0/24 [115/20] via 10.40.2.41, 1d02h, Ethernet0/0
C        10.40.2.0/24 is directly connected, Ethernet0/0
L        10.40.2.42/32 is directly connected, Ethernet0/0
C        10.40.3.0/24 is directly connected, Ethernet0/2
L        10.40.3.42/32 is directly connected, Ethernet0/2
i L2     10.40.4.0/24 [115/20] via 10.40.3.43, 1d02h, Ethernet0/2
B        10.50.0.51/32 [200/1024640] via 172.16.6.52, 1d00h
B        10.50.0.52/32 [200/0] via 172.16.6.52, 1d00h
B        10.50.0.53/32 [200/1024640] via 172.16.6.52, 1d00h
B        10.50.0.54/32 [200/1536640] via 172.16.6.52, 1d00h
B        10.50.1.0/24 [200/1536000] via 172.16.6.52, 1d00h
B        10.50.2.0/24 [200/1536000] via 172.16.6.52, 1d00h
B        10.50.3.0/24 [200/0] via 172.16.6.52, 1d00h
B        10.50.4.0/24 [200/0] via 172.16.6.52, 1d00h
B        10.50.5.0/24 [200/1536000] via 172.16.6.52, 1d00h
B        10.50.6.0/24 [200/1536000] via 172.16.6.52, 1d00h
B        10.50.255.0/24 [200/1536000] via 172.16.6.52, 1d00h
S        10.60.0.0/16 [1/0] via 172.16.9.61
S        10.70.0.0/16 [1/0] via 172.16.10.71
      172.16.0.0/16 is variably subnetted, 12 subnets, 2 masks
B        172.16.1.0/24 [200/0] via 172.16.5.21, 1d00h
B        172.16.2.0/24 [200/0] via 172.16.4.31, 1d00h
B        172.16.3.0/24 [200/0] via 172.16.5.21, 1d00h
B        172.16.4.0/24 [200/0] via 10.40.0.44, 1d00h
B        172.16.5.0/24 [200/0] via 10.40.0.41, 1d00h
B        172.16.6.0/24 [200/0] via 10.40.0.44, 1d00h
B        172.16.7.0/24 [200/0] via 10.40.0.43, 1d00h
B        172.16.8.0/24 [200/0] via 10.40.0.43, 1d00h
C        172.16.9.0/24 is directly connected, Ethernet0/1
L        172.16.9.42/32 is directly connected, Ethernet0/1
C        172.16.10.0/24 is directly connected, Ethernet0/3
L        172.16.10.42/32 is directly connected, Ethernet0/3
R42#
R42#
R42#
R42#
R42#show ip bgp
BGP table version is 56, local router ID is 10.40.0.42
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter,
              x best-external, a additional-path, c RIB-compressed,
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 *>i 10.10.0.11/32    172.16.5.21              0    100      0 101 1001 ?
 *>i 10.10.0.12/32    172.16.5.21              0    100      0 101 1001 ?
 *>i 10.10.0.13/32    172.16.5.21              0    100      0 101 1001 ?
 *>i 10.10.0.14/32    172.16.5.21              0    100      0 101 1001 ?
 *>i 10.10.0.15/32    172.16.5.21              0    100      0 101 1001 ?
 *>i 10.10.0.16/32    172.16.5.21              0    100      0 101 1001 ?
 *>i 10.10.1.0/24     172.16.5.21              0    100      0 101 1001 ?
 *>i 10.10.2.0/24     172.16.5.21              0    100      0 101 1001 ?
 *>i 10.10.3.0/24     172.16.5.21              0    100      0 101 1001 ?
 *>i 10.10.4.0/24     172.16.5.21              0    100      0 101 1001 ?
 *>i 10.10.5.0/24     172.16.5.21              0    100      0 101 1001 ?
 *>i 10.10.6.0/24     172.16.5.21              0    100      0 101 1001 ?
 *>i 10.10.7.0/24     172.16.5.21              0    100      0 101 1001 ?
 *>i 10.10.8.0/24     172.16.5.21              0    100      0 101 1001 ?
 *>i 10.10.9.0/24     172.16.5.21              0    100      0 101 1001 ?
 *>i 10.10.255.0/24   172.16.5.21              0    100      0 101 1001 ?
 *>i 10.20.0.0/16     172.16.5.21              0    100      0 101 ?
 *>i 10.30.0.0/16     172.16.4.31              0    100      0 301 ?
 *>  10.40.0.41/32    10.40.2.41              20         32768 ?
 *>  10.40.0.43/32    10.40.3.43              20         32768 ?
 * i                  10.40.1.44              30    100      0 ?
 *>  10.40.0.44/32    10.40.2.41              30         32768 ?
 * i                  10.40.1.44              20    100      0 ?
 * i 10.40.1.0/24     10.40.4.44              20    100      0 ?
 *>                   10.40.2.41              20         32768 ?
 *>  10.40.4.0/24     10.40.3.43              20         32768 ?
 * i                  10.40.1.44              20    100      0 ?
 *>i 10.50.0.51/32    172.16.6.52        1024640    100      0 2042 ?
 *>i 10.50.0.52/32    172.16.6.52              0    100      0 2042 ?
 *>i 10.50.0.53/32    172.16.6.52        1024640    100      0 2042 ?
 *>i 10.50.0.54/32    172.16.6.52        1536640    100      0 2042 ?
 *>i 10.50.1.0/24     172.16.6.52        1536000    100      0 2042 ?
 *>i 10.50.2.0/24     172.16.6.52        1536000    100      0 2042 ?
 *>i 10.50.3.0/24     172.16.6.52              0    100      0 2042 ?
 *>i 10.50.4.0/24     172.16.6.52              0    100      0 2042 ?
 *>i 10.50.5.0/24     172.16.6.52        1536000    100      0 2042 ?
 *>i 10.50.6.0/24     172.16.6.52        1536000    100      0 2042 ?
 *>i 10.50.255.0/24   172.16.6.52        1536000    100      0 2042 ?
 *>  10.60.0.0/16     172.16.9.61              0         32768 ?
 *>  10.70.0.0/16     172.16.10.71             0         32768 ?
 *>i 172.16.1.0/24    172.16.5.21              0    100      0 101 i
 *>i 172.16.2.0/24    172.16.4.31              0    100      0 301 i
 *>i 172.16.3.0/24    172.16.5.21              0    100      0 101 i
 *>i 172.16.4.0/24    10.40.0.44               0    100      0 i
 *>i 172.16.5.0/24    10.40.0.41               0    100      0 i
 *>i 172.16.6.0/24    10.40.0.44               0    100      0 i
 *>i 172.16.7.0/24    10.40.0.43               0    100      0 i
 *>i 172.16.8.0/24    10.40.0.43               0    100      0 i
 *>  172.16.9.0/24    0.0.0.0                  0         32768 i
 *>  172.16.10.0/24   0.0.0.0                  0         32768 i
R42#
R42#
R42#
R42#
R42#show ip bgp summary
BGP router identifier 10.40.0.42, local AS number 520
BGP table version is 56, main routing table version 56
46 network entries using 6440 bytes of memory
50 path entries using 4000 bytes of memory
16/14 BGP path/bestpath attribute entries using 2304 bytes of memory
2 BGP rrinfo entries using 48 bytes of memory
4 BGP AS-PATH entries using 96 bytes of memory
0 BGP route-map cache entries using 0 bytes of memory
0 BGP filter-list cache entries using 0 bytes of memory
BGP using 12888 total bytes of memory
BGP activity 46/0 prefixes, 53/3 paths, scan interval 60 secs

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
10.40.0.41      4          520    1660    1648       56    0    0 1d00h          41


```
</details>

### R43

<details>
  <summary>Конфигурация</summary>

```

R43#terminal length 0
R43#sh run
Building configuration...

Current configuration : 1628 bytes
!
! Last configuration change at 15:49:25 UTC Sat Jan 25 2025
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
 redistribute static
 redistribute isis level-1-2
 neighbor 10.40.0.41 remote-as 520
 neighbor 10.40.0.41 update-source Loopback1
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
Ethernet0/0                10.40.4.43      YES NVRAM  up                    up
Ethernet0/1                172.16.8.43     YES NVRAM  up                    up
Ethernet0/2                10.40.3.43      YES NVRAM  up                    up
Ethernet0/3                172.16.7.43     YES NVRAM  up                    up
Ethernet1/0                unassigned      YES NVRAM  administratively down down
Ethernet1/1                unassigned      YES NVRAM  administratively down down
Ethernet1/2                unassigned      YES NVRAM  administratively down down
Ethernet1/3                unassigned      YES NVRAM  administratively down down
Loopback1                  10.40.0.43      YES NVRAM  up                    up
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

      10.0.0.0/8 is variably subnetted, 41 subnets, 3 masks
B        10.10.0.11/32 [200/0] via 172.16.5.21, 22:48:59
B        10.10.0.12/32 [200/0] via 172.16.5.21, 22:48:59
B        10.10.0.13/32 [200/0] via 172.16.5.21, 22:48:59
B        10.10.0.14/32 [200/0] via 172.16.5.21, 22:48:59
B        10.10.0.15/32 [200/0] via 172.16.5.21, 22:48:59
B        10.10.0.16/32 [200/0] via 172.16.5.21, 22:48:59
B        10.10.1.0/24 [200/0] via 172.16.5.21, 22:48:59
B        10.10.2.0/24 [200/0] via 172.16.5.21, 22:48:59
B        10.10.3.0/24 [200/0] via 172.16.5.21, 22:48:59
B        10.10.4.0/24 [200/0] via 172.16.5.21, 22:48:59
B        10.10.5.0/24 [200/0] via 172.16.5.21, 22:48:59
B        10.10.6.0/24 [200/0] via 172.16.5.21, 22:48:59
B        10.10.7.0/24 [200/0] via 172.16.5.21, 22:48:59
B        10.10.8.0/24 [200/0] via 172.16.5.21, 22:48:59
B        10.10.9.0/24 [200/0] via 172.16.5.21, 22:48:59
B        10.10.255.0/24 [200/0] via 172.16.5.21, 22:48:59
B        10.20.0.0/16 [200/0] via 172.16.5.21, 22:23:31
B        10.30.0.0/16 [200/0] via 172.16.4.31, 22:21:20
i L2     10.40.0.41/32 [115/30] via 10.40.4.44, 1d02h, Ethernet0/0
                       [115/30] via 10.40.3.42, 1d02h, Ethernet0/2
i L2     10.40.0.42/32 [115/20] via 10.40.3.42, 1d02h, Ethernet0/2
C        10.40.0.43/32 is directly connected, Loopback1
i L2     10.40.0.44/32 [115/20] via 10.40.4.44, 1d02h, Ethernet0/0
i L2     10.40.1.0/24 [115/20] via 10.40.4.44, 1d02h, Ethernet0/0
i L2     10.40.2.0/24 [115/20] via 10.40.3.42, 1d02h, Ethernet0/2
C        10.40.3.0/24 is directly connected, Ethernet0/2
L        10.40.3.43/32 is directly connected, Ethernet0/2
C        10.40.4.0/24 is directly connected, Ethernet0/0
L        10.40.4.43/32 is directly connected, Ethernet0/0
B        10.50.0.51/32 [20/1024640] via 172.16.7.52, 1d00h
B        10.50.0.52/32 [20/0] via 172.16.7.52, 1d00h
B        10.50.0.53/32 [20/1024640] via 172.16.7.52, 1d00h
B        10.50.0.54/32 [20/1536640] via 172.16.7.52, 1d00h
B        10.50.1.0/24 [20/1536000] via 172.16.7.52, 1d00h
B        10.50.2.0/24 [20/1536000] via 172.16.7.52, 1d00h
B        10.50.3.0/24 [20/0] via 172.16.7.52, 1d00h
B        10.50.4.0/24 [20/0] via 172.16.7.52, 1d00h
B        10.50.5.0/24 [20/1536000] via 172.16.7.52, 1d00h
B        10.50.6.0/24 [20/1536000] via 172.16.7.52, 1d00h
B        10.50.255.0/24 [20/1536000] via 172.16.7.52, 1d00h
B        10.60.0.0/16 [200/0] via 172.16.9.61, 22:27:12
S        10.70.0.0/16 [1/0] via 172.16.8.71
      172.16.0.0/16 is variably subnetted, 12 subnets, 2 masks
B        172.16.1.0/24 [200/0] via 172.16.5.21, 1d00h
B        172.16.2.0/24 [200/0] via 172.16.4.31, 1d00h
B        172.16.3.0/24 [200/0] via 172.16.5.21, 1d00h
B        172.16.4.0/24 [200/0] via 10.40.0.44, 1d00h
B        172.16.5.0/24 [200/0] via 10.40.0.41, 1d00h
B        172.16.6.0/24 [200/0] via 10.40.0.44, 1d00h
C        172.16.7.0/24 is directly connected, Ethernet0/3
L        172.16.7.43/32 is directly connected, Ethernet0/3
C        172.16.8.0/24 is directly connected, Ethernet0/1
L        172.16.8.43/32 is directly connected, Ethernet0/1
B        172.16.9.0/24 [200/0] via 10.40.0.42, 1d00h
B        172.16.10.0/24 [200/0] via 10.40.0.42, 1d00h
R43#
R43#
R43#
R43#
R43#show ip bgp
BGP table version is 60, local router ID is 10.40.0.43
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter,
              x best-external, a additional-path, c RIB-compressed,
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 *>i 10.10.0.11/32    172.16.5.21              0    100      0 101 1001 ?
 *>i 10.10.0.12/32    172.16.5.21              0    100      0 101 1001 ?
 *>i 10.10.0.13/32    172.16.5.21              0    100      0 101 1001 ?
 *>i 10.10.0.14/32    172.16.5.21              0    100      0 101 1001 ?
 *>i 10.10.0.15/32    172.16.5.21              0    100      0 101 1001 ?
 *>i 10.10.0.16/32    172.16.5.21              0    100      0 101 1001 ?
 *>i 10.10.1.0/24     172.16.5.21              0    100      0 101 1001 ?
 *>i 10.10.2.0/24     172.16.5.21              0    100      0 101 1001 ?
 *>i 10.10.3.0/24     172.16.5.21              0    100      0 101 1001 ?
 *>i 10.10.4.0/24     172.16.5.21              0    100      0 101 1001 ?
 *>i 10.10.5.0/24     172.16.5.21              0    100      0 101 1001 ?
 *>i 10.10.6.0/24     172.16.5.21              0    100      0 101 1001 ?
 *>i 10.10.7.0/24     172.16.5.21              0    100      0 101 1001 ?
 *>i 10.10.8.0/24     172.16.5.21              0    100      0 101 1001 ?
 *>i 10.10.9.0/24     172.16.5.21              0    100      0 101 1001 ?
 *>i 10.10.255.0/24   172.16.5.21              0    100      0 101 1001 ?
 *>i 10.20.0.0/16     172.16.5.21              0    100      0 101 ?
 *>i 10.30.0.0/16     172.16.4.31              0    100      0 301 ?
 *>  10.40.0.41/32    10.40.3.42              30         32768 ?
 *>  10.40.0.42/32    10.40.3.42              20         32768 ?
 * i                  10.40.2.42              20    100      0 ?
 r>i 10.40.0.43/32    10.40.1.44              30    100      0 ?
 *>  10.40.0.44/32    10.40.4.44              20         32768 ?
 * i                  10.40.1.44              20    100      0 ?
 *>  10.40.1.0/24     10.40.4.44              20         32768 ?
 *>  10.40.2.0/24     10.40.3.42              20         32768 ?
 r>i 10.40.3.0/24     10.40.2.42              20    100      0 ?
 r>i 10.40.4.0/24     10.40.1.44              20    100      0 ?
 * i 10.50.0.51/32    172.16.6.52        1024640    100      0 2042 ?
 *>                   172.16.7.52        1024640             0 2042 ?
 * i 10.50.0.52/32    172.16.6.52              0    100      0 2042 ?
 *>                   172.16.7.52              0             0 2042 ?
 * i 10.50.0.53/32    172.16.6.52        1024640    100      0 2042 ?
 *>                   172.16.7.52        1024640             0 2042 ?
 * i 10.50.0.54/32    172.16.6.52        1536640    100      0 2042 ?
 *>                   172.16.7.52        1536640             0 2042 ?
 * i 10.50.1.0/24     172.16.6.52        1536000    100      0 2042 ?
 *>                   172.16.7.52        1536000             0 2042 ?
 * i 10.50.2.0/24     172.16.6.52        1536000    100      0 2042 ?
 *>                   172.16.7.52        1536000             0 2042 ?
 * i 10.50.3.0/24     172.16.6.52              0    100      0 2042 ?
 *>                   172.16.7.52              0             0 2042 ?
 * i 10.50.4.0/24     172.16.6.52              0    100      0 2042 ?
 *>                   172.16.7.52              0             0 2042 ?
 * i 10.50.5.0/24     172.16.6.52        1536000    100      0 2042 ?
 *>                   172.16.7.52        1536000             0 2042 ?
 * i 10.50.6.0/24     172.16.6.52        1536000    100      0 2042 ?
 *>                   172.16.7.52        1536000             0 2042 ?
 * i 10.50.255.0/24   172.16.6.52        1536000    100      0 2042 ?
 *>                   172.16.7.52        1536000             0 2042 ?
 *>i 10.60.0.0/16     172.16.9.61              0    100      0 ?
 *>  10.70.0.0/16     172.16.8.71              0         32768 ?
 * i                  172.16.10.71             0    100      0 ?
 *>i 172.16.1.0/24    172.16.5.21              0    100      0 101 i
 *>i 172.16.2.0/24    172.16.4.31              0    100      0 301 i
 *>i 172.16.3.0/24    172.16.5.21              0    100      0 101 i
 *>i 172.16.4.0/24    10.40.0.44               0    100      0 i
 *>i 172.16.5.0/24    10.40.0.41               0    100      0 i
 *>i 172.16.6.0/24    10.40.0.44               0    100      0 i
 *                    172.16.7.52              0             0 2042 i
 *   172.16.7.0/24    172.16.7.52              0             0 2042 i
 *>                   0.0.0.0                  0         32768 i
 *>  172.16.8.0/24    0.0.0.0                  0         32768 i
 *>i 172.16.9.0/24    10.40.0.42               0    100      0 i
 *>i 172.16.10.0/24   10.40.0.42               0    100      0 i
R43#
R43#
R43#
R43#
R43#show ip bgp summary
BGP router identifier 10.40.0.43, local AS number 520
BGP table version is 60, main routing table version 60
49 network entries using 6860 bytes of memory
65 path entries using 5200 bytes of memory
22/17 BGP path/bestpath attribute entries using 3168 bytes of memory
2 BGP rrinfo entries using 48 bytes of memory
4 BGP AS-PATH entries using 96 bytes of memory
0 BGP route-map cache entries using 0 bytes of memory
0 BGP filter-list cache entries using 0 bytes of memory
BGP using 15372 total bytes of memory
BGP activity 49/0 prefixes, 68/3 paths, scan interval 60 secs

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
10.40.0.41      4          520    1662    1653       60    0    0 1d00h          44
172.16.7.52     4         2042    1749    1743       60    0    0 1d02h          13


```
</details>

### R44

<details>
  <summary>Конфигурация</summary>

```

R44#terminal length 0
R44#sh run
Building configuration...

Current configuration : 1600 bytes
!
! Last configuration change at 14:11:05 UTC Sat Jan 25 2025
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
 redistribute isis level-1-2
 neighbor 10.40.0.41 remote-as 520
 neighbor 10.40.0.41 update-source Loopback1
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
Ethernet0/0                172.16.4.44     YES NVRAM  up                    up
Ethernet0/1                10.40.4.44      YES NVRAM  up                    up
Ethernet0/2                10.40.1.44      YES NVRAM  up                    up
Ethernet0/3                172.16.6.44     YES NVRAM  up                    up
Ethernet1/0                unassigned      YES NVRAM  administratively down down
Ethernet1/1                unassigned      YES NVRAM  administratively down down
Ethernet1/2                unassigned      YES NVRAM  administratively down down
Ethernet1/3                unassigned      YES NVRAM  administratively down down
Loopback1                  10.40.0.44      YES NVRAM  up                    up
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

      10.0.0.0/8 is variably subnetted, 41 subnets, 3 masks
B        10.10.0.11/32 [20/0] via 172.16.4.31, 22:33:22
B        10.10.0.12/32 [20/0] via 172.16.4.31, 22:33:22
B        10.10.0.13/32 [20/0] via 172.16.4.31, 22:33:22
B        10.10.0.14/32 [20/0] via 172.16.4.31, 22:33:22
B        10.10.0.15/32 [20/0] via 172.16.4.31, 22:33:22
B        10.10.0.16/32 [20/0] via 172.16.4.31, 22:33:22
B        10.10.1.0/24 [20/0] via 172.16.4.31, 22:33:22
B        10.10.2.0/24 [20/0] via 172.16.4.31, 22:33:22
B        10.10.3.0/24 [20/0] via 172.16.4.31, 22:33:22
B        10.10.4.0/24 [20/0] via 172.16.4.31, 22:33:22
B        10.10.5.0/24 [20/0] via 172.16.4.31, 22:33:22
B        10.10.6.0/24 [20/0] via 172.16.4.31, 22:33:22
B        10.10.7.0/24 [20/0] via 172.16.4.31, 22:33:22
B        10.10.8.0/24 [20/0] via 172.16.4.31, 22:33:22
B        10.10.9.0/24 [20/0] via 172.16.4.31, 22:33:22
B        10.10.255.0/24 [20/0] via 172.16.4.31, 22:33:22
B        10.20.0.0/16 [200/0] via 172.16.5.21, 22:24:07
B        10.30.0.0/16 [20/0] via 172.16.4.31, 22:21:56
i L2     10.40.0.41/32 [115/20] via 10.40.1.41, 1d02h, Ethernet0/2
i L2     10.40.0.42/32 [115/30] via 10.40.4.43, 1d02h, Ethernet0/1
                       [115/30] via 10.40.1.41, 1d02h, Ethernet0/2
i L2     10.40.0.43/32 [115/20] via 10.40.4.43, 1d02h, Ethernet0/1
C        10.40.0.44/32 is directly connected, Loopback1
C        10.40.1.0/24 is directly connected, Ethernet0/2
L        10.40.1.44/32 is directly connected, Ethernet0/2
i L2     10.40.2.0/24 [115/20] via 10.40.1.41, 1d02h, Ethernet0/2
i L2     10.40.3.0/24 [115/20] via 10.40.4.43, 1d02h, Ethernet0/1
C        10.40.4.0/24 is directly connected, Ethernet0/1
L        10.40.4.44/32 is directly connected, Ethernet0/1
B        10.50.0.51/32 [20/1024640] via 172.16.6.52, 1d00h
B        10.50.0.52/32 [20/0] via 172.16.6.52, 1d00h
B        10.50.0.53/32 [20/1024640] via 172.16.6.52, 1d00h
B        10.50.0.54/32 [20/1536640] via 172.16.6.52, 1d00h
B        10.50.1.0/24 [20/1536000] via 172.16.6.52, 1d00h
B        10.50.2.0/24 [20/1536000] via 172.16.6.52, 1d00h
B        10.50.3.0/24 [20/0] via 172.16.6.52, 1d00h
B        10.50.4.0/24 [20/0] via 172.16.6.52, 1d00h
B        10.50.5.0/24 [20/1536000] via 172.16.6.52, 1d00h
B        10.50.6.0/24 [20/1536000] via 172.16.6.52, 1d00h
B        10.50.255.0/24 [20/1536000] via 172.16.6.52, 1d00h
B        10.60.0.0/16 [200/0] via 172.16.9.61, 22:27:48
B        10.70.0.0/16 [200/0] via 172.16.10.71, 22:27:48
      172.16.0.0/16 is variably subnetted, 12 subnets, 2 masks
B        172.16.1.0/24 [200/0] via 172.16.5.21, 1d00h
B        172.16.2.0/24 [20/0] via 172.16.4.31, 1d02h
B        172.16.3.0/24 [20/0] via 172.16.4.31, 1d02h
C        172.16.4.0/24 is directly connected, Ethernet0/0
L        172.16.4.44/32 is directly connected, Ethernet0/0
B        172.16.5.0/24 [200/0] via 10.40.0.41, 1d00h
C        172.16.6.0/24 is directly connected, Ethernet0/3
L        172.16.6.44/32 is directly connected, Ethernet0/3
B        172.16.7.0/24 [200/0] via 10.40.0.43, 1d00h
B        172.16.8.0/24 [200/0] via 10.40.0.43, 1d00h
B        172.16.9.0/24 [200/0] via 10.40.0.42, 1d00h
B        172.16.10.0/24 [200/0] via 10.40.0.42, 1d00h
R44#
R44#
R44#
R44#
R44#show ip bgp
BGP table version is 95, local router ID is 10.40.0.44
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter,
              x best-external, a additional-path, c RIB-compressed,
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 * i 10.10.0.11/32    172.16.5.21              0    100      0 101 1001 ?
 *>                   172.16.4.31                            0 301 1001 ?
 * i 10.10.0.12/32    172.16.5.21              0    100      0 101 1001 ?
 *>                   172.16.4.31                            0 301 1001 ?
 * i 10.10.0.13/32    172.16.5.21              0    100      0 101 1001 ?
 *>                   172.16.4.31                            0 301 1001 ?
 * i 10.10.0.14/32    172.16.5.21              0    100      0 101 1001 ?
 *>                   172.16.4.31                            0 301 1001 ?
 * i 10.10.0.15/32    172.16.5.21              0    100      0 101 1001 ?
 *>                   172.16.4.31                            0 301 1001 ?
 * i 10.10.0.16/32    172.16.5.21              0    100      0 101 1001 ?
 *>                   172.16.4.31                            0 301 1001 ?
 * i 10.10.1.0/24     172.16.5.21              0    100      0 101 1001 ?
 *>                   172.16.4.31                            0 301 1001 ?
 * i 10.10.2.0/24     172.16.5.21              0    100      0 101 1001 ?
 *>                   172.16.4.31                            0 301 1001 ?
 * i 10.10.3.0/24     172.16.5.21              0    100      0 101 1001 ?
 *>                   172.16.4.31                            0 301 1001 ?
 * i 10.10.4.0/24     172.16.5.21              0    100      0 101 1001 ?
 *>                   172.16.4.31                            0 301 1001 ?
 * i 10.10.5.0/24     172.16.5.21              0    100      0 101 1001 ?
 *>                   172.16.4.31                            0 301 1001 ?
 * i 10.10.6.0/24     172.16.5.21              0    100      0 101 1001 ?
 *>                   172.16.4.31                            0 301 1001 ?
 * i 10.10.7.0/24     172.16.5.21              0    100      0 101 1001 ?
 *>                   172.16.4.31                            0 301 1001 ?
 * i 10.10.8.0/24     172.16.5.21              0    100      0 101 1001 ?
 *>                   172.16.4.31                            0 301 1001 ?
 * i 10.10.9.0/24     172.16.5.21              0    100      0 101 1001 ?
 *>                   172.16.4.31                            0 301 1001 ?
 * i 10.10.255.0/24   172.16.5.21              0    100      0 101 1001 ?
 *>                   172.16.4.31                            0 301 1001 ?
 *>i 10.20.0.0/16     172.16.5.21              0    100      0 101 ?
 *                    172.16.4.31                            0 301 101 ?
 *>  10.30.0.0/16     172.16.4.31              0             0 301 ?
 *>  10.40.0.41/32    10.40.1.41              20         32768 ?
 * i                  10.40.3.42              30    100      0 ?
 *>  10.40.0.42/32    10.40.1.41              30         32768 ?
 * i                  10.40.2.42              20    100      0 ?
 *>  10.40.0.43/32    10.40.4.43              20         32768 ?
 *>  10.40.2.0/24     10.40.1.41              20         32768 ?
 * i                  10.40.3.42              20    100      0 ?
 *>  10.40.3.0/24     10.40.4.43              20         32768 ?
 * i                  10.40.2.42              20    100      0 ?
 *>  10.50.0.51/32    172.16.6.52        1024640             0 2042 ?
 *>  10.50.0.52/32    172.16.6.52              0             0 2042 ?
 *>  10.50.0.53/32    172.16.6.52        1024640             0 2042 ?
 *>  10.50.0.54/32    172.16.6.52        1536640             0 2042 ?
 *>  10.50.1.0/24     172.16.6.52        1536000             0 2042 ?
 *>  10.50.2.0/24     172.16.6.52        1536000             0 2042 ?
 *>  10.50.3.0/24     172.16.6.52              0             0 2042 ?
 *>  10.50.4.0/24     172.16.6.52              0             0 2042 ?
 *>  10.50.5.0/24     172.16.6.52        1536000             0 2042 ?
 *>  10.50.6.0/24     172.16.6.52        1536000             0 2042 ?
 *>  10.50.255.0/24   172.16.6.52        1536000             0 2042 ?
 *>i 10.60.0.0/16     172.16.9.61              0    100      0 ?
 *>i 10.70.0.0/16     172.16.10.71             0    100      0 ?
 *>i 172.16.1.0/24    172.16.5.21              0    100      0 101 i
 *                    172.16.4.31                            0 301 101 i
 *>  172.16.2.0/24    172.16.4.31              0             0 301 i
 * i 172.16.3.0/24    172.16.5.21              0    100      0 101 i
 *>                   172.16.4.31              0             0 301 i
 *   172.16.4.0/24    172.16.4.31              0             0 301 i
 *>                   0.0.0.0                  0         32768 i
 *>i 172.16.5.0/24    10.40.0.41               0    100      0 i
 *                    172.16.4.31                            0 301 101 i
 *   172.16.6.0/24    172.16.6.52              0             0 2042 i
 *>                   0.0.0.0                  0         32768 i
 *>i 172.16.7.0/24    10.40.0.43               0    100      0 i
 *                    172.16.6.52              0             0 2042 i
 *>i 172.16.8.0/24    10.40.0.43               0    100      0 i
 *>i 172.16.9.0/24    10.40.0.42               0    100      0 i
 *>i 172.16.10.0/24   10.40.0.42               0    100      0 i
R44#
R44#
R44#
R44#
R44#show ip bgp summary
BGP router identifier 10.40.0.44, local AS number 520
BGP table version is 95, main routing table version 95
46 network entries using 6440 bytes of memory
73 path entries using 5840 bytes of memory
20/14 BGP path/bestpath attribute entries using 2880 bytes of memory
2 BGP rrinfo entries using 48 bytes of memory
6 BGP AS-PATH entries using 144 bytes of memory
0 BGP route-map cache entries using 0 bytes of memory
0 BGP filter-list cache entries using 0 bytes of memory
BGP using 15352 total bytes of memory
BGP activity 46/0 prefixes, 85/12 paths, scan interval 60 secs

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
10.40.0.41      4          520    1658    1654       95    0    0 1d00h          30
172.16.4.31     4          301    1747    1749       95    0    0 1d02h          23
172.16.6.52     4         2042    1751    1751       95    0    0 1d02h          13


```
</details>

### R52

<details>
  <summary>Конфигурация</summary>

```

R52#terminal length 0
R52#sh run
Building configuration...

Current configuration : 1542 bytes
!
! Last configuration change at 15:04:15 UTC Sat Jan 25 2025
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
 neighbor 172.16.7.43 remote-as 520
 maximum-paths 2
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
B        10.10.0.11/32 [20/0] via 172.16.6.44, 22:33:57
B        10.10.0.12/32 [20/0] via 172.16.6.44, 22:33:57
B        10.10.0.13/32 [20/0] via 172.16.6.44, 22:33:57
B        10.10.0.14/32 [20/0] via 172.16.6.44, 22:33:57
B        10.10.0.15/32 [20/0] via 172.16.6.44, 22:33:57
B        10.10.0.16/32 [20/0] via 172.16.6.44, 22:33:57
B        10.10.1.0/24 [20/0] via 172.16.6.44, 22:33:57
B        10.10.2.0/24 [20/0] via 172.16.6.44, 22:33:57
B        10.10.3.0/24 [20/0] via 172.16.6.44, 22:33:57
B        10.10.4.0/24 [20/0] via 172.16.6.44, 22:33:57
B        10.10.5.0/24 [20/0] via 172.16.6.44, 22:33:57
B        10.10.6.0/24 [20/0] via 172.16.6.44, 22:33:57
B        10.10.7.0/24 [20/0] via 172.16.6.44, 22:33:57
B        10.10.8.0/24 [20/0] via 172.16.6.44, 22:33:57
B        10.10.9.0/24 [20/0] via 172.16.6.44, 22:33:57
B        10.10.255.0/24 [20/0] via 172.16.6.44, 22:33:57
B        10.20.0.0/16 [20/0] via 172.16.7.43, 22:24:12
                      [20/0] via 172.16.6.44, 22:24:12
B        10.30.0.0/16 [20/0] via 172.16.7.43, 22:22:32
                      [20/0] via 172.16.6.44, 22:22:32
B        10.40.0.41/32 [20/20] via 172.16.6.44, 1d00h
B        10.40.0.42/32 [20/20] via 172.16.7.43, 1d00h
B        10.40.0.43/32 [20/0] via 172.16.7.43, 1d00h
B        10.40.0.44/32 [20/20] via 172.16.7.43, 1d00h
B        10.40.1.0/24 [20/20] via 172.16.7.43, 1d00h
B        10.40.2.0/24 [20/20] via 172.16.7.43, 23:11:28
                      [20/20] via 172.16.6.44, 23:11:28
B        10.40.3.0/24 [20/0] via 172.16.7.43, 1d00h
B        10.40.4.0/24 [20/0] via 172.16.7.43, 1d00h
D        10.50.0.51/32 [90/1024640] via 10.50.3.51, 1d02h, Ethernet0/1
C        10.50.0.52/32 is directly connected, Loopback1
D        10.50.0.53/32 [90/1024640] via 10.50.4.53, 1d02h, Ethernet0/0
D        10.50.0.54/32 [90/1536640] via 10.50.4.53, 1d02h, Ethernet0/0
D        10.50.1.0/24 [90/1536000] via 10.50.4.53, 1d02h, Ethernet0/0
                      [90/1536000] via 10.50.3.51, 1d02h, Ethernet0/1
D        10.50.2.0/24 [90/1536000] via 10.50.4.53, 1d02h, Ethernet0/0
                      [90/1536000] via 10.50.3.51, 1d02h, Ethernet0/1
C        10.50.3.0/24 is directly connected, Ethernet0/1
L        10.50.3.52/32 is directly connected, Ethernet0/1
C        10.50.4.0/24 is directly connected, Ethernet0/0
L        10.50.4.52/32 is directly connected, Ethernet0/0
D        10.50.5.0/24 [90/1536000] via 10.50.4.53, 1d02h, Ethernet0/0
D        10.50.6.0/24 [90/1536000] via 10.50.4.53, 1d02h, Ethernet0/0
                      [90/1536000] via 10.50.3.51, 1d02h, Ethernet0/1
D        10.50.255.0/24 [90/1536000] via 10.50.4.53, 1d02h, Ethernet0/0
                        [90/1536000] via 10.50.3.51, 1d02h, Ethernet0/1
B        10.60.0.0/16 [20/0] via 172.16.7.43, 22:28:24
                      [20/0] via 172.16.6.44, 22:28:24
B        10.70.0.0/16 [20/0] via 172.16.7.43, 22:27:53
                      [20/0] via 172.16.6.44, 22:27:53
      172.16.0.0/16 is variably subnetted, 12 subnets, 2 masks
B        172.16.1.0/24 [20/0] via 172.16.7.43, 23:11:28
                       [20/0] via 172.16.6.44, 23:11:28
B        172.16.2.0/24 [20/0] via 172.16.7.43, 23:11:28
                       [20/0] via 172.16.6.44, 23:11:28
B        172.16.3.0/24 [20/0] via 172.16.6.44, 1d02h
B        172.16.4.0/24 [20/0] via 172.16.7.43, 23:11:28
                       [20/0] via 172.16.6.44, 23:11:28
B        172.16.5.0/24 [20/0] via 172.16.7.43, 23:11:28
                       [20/0] via 172.16.6.44, 23:11:28
C        172.16.6.0/24 is directly connected, Ethernet0/2
L        172.16.6.52/32 is directly connected, Ethernet0/2
C        172.16.7.0/24 is directly connected, Ethernet0/3
L        172.16.7.52/32 is directly connected, Ethernet0/3
B        172.16.8.0/24 [20/0] via 172.16.7.43, 23:11:28
                       [20/0] via 172.16.6.44, 23:11:28
B        172.16.9.0/24 [20/0] via 172.16.7.43, 23:11:28
                       [20/0] via 172.16.6.44, 23:11:28
B        172.16.10.0/24 [20/0] via 172.16.7.43, 23:11:28
                        [20/0] via 172.16.6.44, 23:11:28
R52#
R52#
R52#
R52#
R52#show ip bgp
BGP table version is 115, local router ID is 10.50.0.52
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter,
              x best-external, a additional-path, c RIB-compressed,
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 *   10.10.0.11/32    172.16.7.43                            0 520 101 1001 ?
 *>                   172.16.6.44                            0 520 301 1001 ?
 *   10.10.0.12/32    172.16.7.43                            0 520 101 1001 ?
 *>                   172.16.6.44                            0 520 301 1001 ?
 *   10.10.0.13/32    172.16.7.43                            0 520 101 1001 ?
 *>                   172.16.6.44                            0 520 301 1001 ?
 *   10.10.0.14/32    172.16.7.43                            0 520 101 1001 ?
 *>                   172.16.6.44                            0 520 301 1001 ?
 *   10.10.0.15/32    172.16.7.43                            0 520 101 1001 ?
 *>                   172.16.6.44                            0 520 301 1001 ?
 *   10.10.0.16/32    172.16.7.43                            0 520 101 1001 ?
 *>                   172.16.6.44                            0 520 301 1001 ?
 *   10.10.1.0/24     172.16.7.43                            0 520 101 1001 ?
 *>                   172.16.6.44                            0 520 301 1001 ?
 *   10.10.2.0/24     172.16.7.43                            0 520 101 1001 ?
 *>                   172.16.6.44                            0 520 301 1001 ?
 *   10.10.3.0/24     172.16.7.43                            0 520 101 1001 ?
 *>                   172.16.6.44                            0 520 301 1001 ?
 *   10.10.4.0/24     172.16.7.43                            0 520 101 1001 ?
 *>                   172.16.6.44                            0 520 301 1001 ?
 *   10.10.5.0/24     172.16.7.43                            0 520 101 1001 ?
 *>                   172.16.6.44                            0 520 301 1001 ?
 *   10.10.6.0/24     172.16.7.43                            0 520 101 1001 ?
 *>                   172.16.6.44                            0 520 301 1001 ?
 *   10.10.7.0/24     172.16.7.43                            0 520 101 1001 ?
 *>                   172.16.6.44                            0 520 301 1001 ?
 *   10.10.8.0/24     172.16.7.43                            0 520 101 1001 ?
 *>                   172.16.6.44                            0 520 301 1001 ?
 *   10.10.9.0/24     172.16.7.43                            0 520 101 1001 ?
 *>                   172.16.6.44                            0 520 301 1001 ?
 *   10.10.255.0/24   172.16.7.43                            0 520 101 1001 ?
 *>                   172.16.6.44                            0 520 301 1001 ?
 *>  10.20.0.0/16     172.16.7.43                            0 520 101 ?
 *m                   172.16.6.44                            0 520 101 ?
 *m  10.30.0.0/16     172.16.7.43                            0 520 301 ?
 *>                   172.16.6.44                            0 520 301 ?
 *>  10.40.0.41/32    172.16.6.44             20             0 520 ?
 *                    172.16.7.43             30             0 520 ?
 *>  10.40.0.42/32    172.16.7.43             20             0 520 ?
 *                    172.16.6.44             30             0 520 ?
 *   10.40.0.43/32    172.16.6.44             20             0 520 ?
 *>                   172.16.7.43                            0 520 ?
 *>  10.40.0.44/32    172.16.7.43             20             0 520 ?
 *>  10.40.1.0/24     172.16.7.43             20             0 520 ?
 *>  10.40.2.0/24     172.16.6.44             20             0 520 ?
 *m                   172.16.7.43             20             0 520 ?
 *>  10.40.3.0/24     172.16.7.43                            0 520 ?
 *                    172.16.6.44             20             0 520 ?
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
 *m  10.60.0.0/16     172.16.7.43                            0 520 ?
 *>                   172.16.6.44                            0 520 ?
 *m  10.70.0.0/16     172.16.7.43              0             0 520 ?
 *>                   172.16.6.44                            0 520 ?
 *m  172.16.1.0/24    172.16.7.43                            0 520 101 i
 *>                   172.16.6.44                            0 520 101 i
 *m  172.16.2.0/24    172.16.7.43                            0 520 301 i
 *>                   172.16.6.44                            0 520 301 i
 *   172.16.3.0/24    172.16.7.43                            0 520 101 i
 *>                   172.16.6.44                            0 520 301 i
 *m  172.16.4.0/24    172.16.7.43                            0 520 i
 *>                   172.16.6.44              0             0 520 i
 *m  172.16.5.0/24    172.16.7.43                            0 520 i
 *>                   172.16.6.44                            0 520 i
 *   172.16.6.0/24    172.16.7.43                            0 520 i
 *                    172.16.6.44              0             0 520 i
 *>                   0.0.0.0                  0         32768 i
 *   172.16.7.0/24    172.16.6.44                            0 520 i
 *                    172.16.7.43              0             0 520 i
 *>                   0.0.0.0                  0         32768 i
 *m  172.16.8.0/24    172.16.6.44                            0 520 i
 *>                   172.16.7.43              0             0 520 i
 *m  172.16.9.0/24    172.16.7.43                            0 520 i
 *>                   172.16.6.44                            0 520 i
 *m  172.16.10.0/24   172.16.7.43                            0 520 i
 *>                   172.16.6.44                            0 520 i
R52#
R52#
R52#
R52#
R52#show ip bgp summary
BGP router identifier 10.50.0.52, local AS number 2042
BGP table version is 115, main routing table version 115
49 network entries using 6860 bytes of memory
86 path entries using 6880 bytes of memory
12 multipath network entries and 24 multipath paths
17/14 BGP path/bestpath attribute entries using 2448 bytes of memory
5 BGP AS-PATH entries using 120 bytes of memory
0 BGP route-map cache entries using 0 bytes of memory
0 BGP filter-list cache entries using 0 bytes of memory
BGP using 16308 total bytes of memory
BGP activity 49/0 prefixes, 89/3 paths, scan interval 60 secs

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
172.16.6.44     4          520    1752    1752      115    0    0 1d02h          35
172.16.7.43     4          520    1744    1750      115    0    0 1d02h          38

```
</details>