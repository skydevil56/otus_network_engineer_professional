# GRE,  VPN,  DMVPN

## Цели

1. Настроить GRE между офисами Москва и Санкт-Петербург
1. Настроить DMVPN (без шифрования) между офисами Москва и Чокурдак, Лабытнанги

## Задание

1. Настроить GRE между офисами Москва и Санкт-Петербург.
1. Настроить DMVPN (без шифрования) между офисами Москва и Чокурдак, Лабытнанги.
1. Все узлы в офисах лабораторной работы должны иметь IP связность.

## Топология

![a](media/lab13_1.PNG)

## Схема для импорта в PNETlab

[Схема для импорта в PNETlab](media/otus_cource_lab13_GRE_DMVPN_pnetlab_export-20250223-163533.zip)

## Версии ПО

- PNETlab - 5.3.11
- Роутеры - Cisco IOS Software, Linux Software (I86BI_LINUX-ADVENTERPRISEK9-M), Version 15.4(2)T4
- Коммутаторы - Cisco IOS Software, Linux Software (I86BI_LINUXL2-ADVENTERPRISEK9-M), Version 15.2(CML_NIGHTLY_20150703)
- ПК - VPC

## Решение

1. Настроить GRE между офисами Москва и Санкт-Петербург.

      Так как все внутренние подсети (10.X.X.X/16) филиалов Москва, Санкт-Петербург, Чокурдак, Лабытнанги должны быть доступны через GRE, соответственно, эти подсети не должны быть анонсированы в сторону провайдеров Интерннет (за исключанием пула адресов, используемых для NAT, а именно: 10.10.13.0/24 и 10.10.14.0/24 в Москве, 10.50.52.0/25 и 10.50.52.128/25 в Санкт-Петербурге). На устройствах R42 и R43 нужно убрать статические маршруты до внутренних подсетей офисов Лабытнанги и Чокурдак.
      Между офисами должна быть связность по внешним адресам (172.16.X.X/24, 10.10.13.0/24 и 10.10.14.0/24, 10.50.52.0/25 и 10.50.52.128/25).

      Настройка GRE на R13:

      ```

      interface Tunnel0
       description to SPB ISP1
       ip address 192.168.3.1 255.255.255.0
       tunnel source 10.10.13.254
       tunnel destination 10.50.52.126
       tunnel key 2
      !
      interface Tunnel1
       description to SPB ISP2
       ip address 192.168.4.1 255.255.255.0
       tunnel source 10.10.13.254
       tunnel destination 10.50.52.254
       tunnel key 2

      ```

      Настройка BGP на R13:

      ```

      router bgp 1001
       bgp log-neighbor-changes
       network 10.10.13.0 mask 255.255.255.0
       timers bgp 3 9
       redistribute ospf 1
       neighbor 172.16.1.21 remote-as 101
       neighbor 172.16.1.21 prefix-list EBGP-ISP-FILTER out
      
       neighbor 192.168.3.2 remote-as 2042
       neighbor 192.168.3.2 ebgp-multihop 255
       neighbor 192.168.3.2 update-source Tunnel0
       neighbor 192.168.3.2 prefix-list EBGP-GRE-FILTER out
       neighbor 192.168.3.2 route-map EBGP-GRE-LP in
       neighbor 192.168.3.2 filter-list 1 out
      
       neighbor 192.168.4.2 remote-as 2042
       neighbor 192.168.4.2 ebgp-multihop 255
       neighbor 192.168.4.2 update-source Tunnel1
       neighbor 192.168.4.2 prefix-list EBGP-GRE-FILTER out
       neighbor 192.168.4.2 filter-list 1 out
      !
      ip as-path access-list 1 permit ^$
      !
      ip prefix-list ALL seq 5 permit 0.0.0.0/0 le 32
      !
      ip prefix-list EBGP-GRE-FILTER seq 10 deny 10.10.13.0/24
      ip prefix-list EBGP-GRE-FILTER seq 20 deny 10.10.14.0/24
      ip prefix-list EBGP-GRE-FILTER seq 25 deny 172.16.0.0/16 ge 24
      ip prefix-list EBGP-GRE-FILTER seq 30 permit 0.0.0.0/0 le 32
      !
      ip prefix-list EBGP-ISP-FILTER seq 10 permit 10.10.13.0/24
      !
      route-map EBGP-GRE-LP permit 10
       match ip address prefix-list ALL
       set local-preference 150
      
      ```

      Настройка GRE на R14:

      ```

      interface Tunnel0
       description to SPB ISP1
       ip address 192.168.1.1 255.255.255.0
       tunnel source 10.10.14.254
       tunnel destination 10.50.52.126
       tunnel key 1
      !
       interface Tunnel1
       description to SPB ISP2
       ip address 192.168.2.1 255.255.255.0
       tunnel source 10.10.14.254
       tunnel destination 10.50.52.254
       tunnel key 1
      
      ```

      Настройка BGP на R14:

      ```

      router bgp 1001
       bgp log-neighbor-changes
       network 10.10.14.0 mask 255.255.255.0
       timers bgp 3 9
       redistribute ospf 1
       neighbor 172.16.2.31 remote-as 301
       neighbor 172.16.2.31 prefix-list EBGP-ISP-FILTER out
      
       neighbor 192.168.1.2 remote-as 2042
       neighbor 192.168.1.2 ebgp-multihop 255
       neighbor 192.168.1.2 update-source Tunnel0
       neighbor 192.168.1.2 prefix-list EBGP-GRE-FILTER out
       neighbor 192.168.1.2 route-map EBGP-GRE-LP in
       neighbor 192.168.1.2 filter-list 1 out
      
       neighbor 192.168.2.2 remote-as 2042
       neighbor 192.168.2.2 ebgp-multihop 255
       neighbor 192.168.2.2 update-source Tunnel1
       neighbor 192.168.2.2 prefix-list EBGP-GRE-FILTER out
       neighbor 192.168.2.2 filter-list 1 out
      !
      ip as-path access-list 1 permit ^$
      !
      ip prefix-list ALL seq 5 permit 0.0.0.0/0 le 32
      !
      ip prefix-list EBGP-GRE-FILTER seq 10 deny 10.10.13.0/24
      ip prefix-list EBGP-GRE-FILTER seq 20 deny 10.10.14.0/24
      ip prefix-list EBGP-GRE-FILTER seq 25 deny 172.16.0.0/16 ge 24
      ip prefix-list EBGP-GRE-FILTER seq 30 permit 0.0.0.0/0 le 32
      !
      ip prefix-list EBGP-ISP-FILTER seq 10 permit 10.10.14.0/24
      !
      route-map EBGP-GRE-LP permit 10
       match ip address prefix-list ALL
       set local-preference 150
      
      ```

     Настройка GRE на R52:

      ```

      interface Tunnel0
       description to MSK ISP1
       ip address 192.168.1.2 255.255.255.0
       tunnel source 10.50.52.126
       tunnel destination 10.10.14.254
       tunnel key 1
      !
       interface Tunnel1
       description to MSK ISP1
       ip address 192.168.2.2 255.255.255.0
       tunnel source 10.50.52.254
       tunnel destination 10.10.14.254
       tunnel key 1
      !
       interface Tunnel2
       description to MSK ISP2
       ip address 192.168.3.2 255.255.255.0
       tunnel source 10.50.52.126
       tunnel destination 10.10.13.254
       tunnel key 2
      !
       interface Tunnel3
       description to MSK ISP2
       ip address 192.168.4.2 255.255.255.0
       tunnel source 10.50.52.254
       tunnel destination 10.10.13.254
       tunnel key 2
      
      ```

      Настройка BGP на R52:

      ```

      router bgp 2042
       bgp log-neighbor-changes
       network 10.50.52.0 mask 255.255.255.128
       network 10.50.52.128 mask 255.255.255.128
       timers bgp 3 9
       redistribute eigrp 1
       neighbor 172.16.6.44 remote-as 520
       neighbor 172.16.6.44 prefix-list EBGP-ISP1-FILTER out
      
       neighbor 172.16.7.43 remote-as 520
       neighbor 172.16.7.43 prefix-list EBGP-ISP2-FILTER out
      
       neighbor 192.168.1.1 remote-as 1001
       neighbor 192.168.1.1 ebgp-multihop 255
       neighbor 192.168.1.1 update-source Tunnel0
       neighbor 192.168.1.1 prefix-list EBGP-GRE-FILTER out
       neighbor 192.168.1.1 route-map EBGP-GRE-LP in
       neighbor 192.168.1.1 filter-list 1 out
      
       neighbor 192.168.2.1 remote-as 1001
       neighbor 192.168.2.1 ebgp-multihop 255
       neighbor 192.168.2.1 update-source Tunnel1
       neighbor 192.168.2.1 prefix-list EBGP-GRE-FILTER out
       neighbor 192.168.2.1 filter-list 1 out
      
       neighbor 192.168.3.1 remote-as 1001
       neighbor 192.168.3.1 ebgp-multihop 255
       neighbor 192.168.3.1 update-source Tunnel2
       neighbor 192.168.3.1 prefix-list EBGP-GRE-FILTER out
       neighbor 192.168.3.1 filter-list 1 out
      
       neighbor 192.168.4.1 remote-as 1001
       neighbor 192.168.4.1 ebgp-multihop 255
       neighbor 192.168.4.1 update-source Tunnel3
       neighbor 192.168.4.1 prefix-list EBGP-GRE-FILTER out
       neighbor 192.168.4.1 filter-list 1 out
       maximum-paths 2
      !
      ip as-path access-list 1 permit ^$
      !
      ip prefix-list ALL seq 5 permit 0.0.0.0/0 le 32
      !
      ip prefix-list EBGP-GRE-FILTER seq 10 deny 10.50.52.0/25
      ip prefix-list EBGP-GRE-FILTER seq 20 deny 10.50.52.128/25
      ip prefix-list EBGP-GRE-FILTER seq 25 deny 172.16.0.0/16 ge 24
      ip prefix-list EBGP-GRE-FILTER seq 30 permit 0.0.0.0/0 le 32
      !
      ip prefix-list EBGP-ISP1-FILTER seq 10 permit 10.50.52.0/25
      !
      ip prefix-list EBGP-ISP2-FILTER seq 10 permit 10.50.52.128/25
      !
      route-map EBGP-GRE-LP permit 10
       match ip address prefix-list ALL
       set local-preference 150
      
      ```


1. Настроить DMVPN (без шифрования) между офисами Москва и Чокурдак, Лабытнанги.

      Архитектура - Dual Cloud Dual Hub.  
      В качестве основного хаба - R14 (mGRE\DMVPN облако - 10.100.0.0/24), в качестве резервного - R13 (mGRE\DMVPN облако - 10.101.0.0/24).  
      NBMA адрес на R14 - 10.10.14.254, mGRE - 10.100.0.1, NBMA адрес на R13 - 10.10.13.254, mGRE - 10.101.0.1.
      В качестве протокола маршрутизации iBGP (ASN 1001). R14 и R13 являются роут рефлектор серверами (каждый хаб в своем облаке).

      Настройки на R14 (основной Хаб):

      ```

      interface Tunnel10
       description MGRE to ISP1 (DMVPN)
       ip address 10.100.0.1 255.255.255.0
       no ip redirects
       ip nhrp authentication CISCO
       ip nhrp network-id 100
       ip nhrp holdtime 90
       tunnel source 10.10.14.254
       tunnel mode gre multipoint
       tunnel key 10
       tunnel route-via Ethernet0/2 preferred
      !
      router bgp 1001
       ...
       bgp listen range 10.100.0.0/24 peer-group DMVPN-CLOUD1
       ...
       neighbor DMVPN-CLOUD1 peer-group
       neighbor DMVPN-CLOUD1 remote-as 1001
       neighbor DMVPN-CLOUD1 route-reflector-client
       neighbor DMVPN-CLOUD1 next-hop-self
       neighbor DMVPN-CLOUD1 prefix-list IBGP-DMVPN-GRE-FILTER out
      !
      ip prefix-list IBGP-DMVPN-GRE-FILTER seq 10 deny 10.10.13.0/24
      ip prefix-list IBGP-DMVPN-GRE-FILTER seq 20 deny 10.10.14.0/24
      ip prefix-list IBGP-DMVPN-GRE-FILTER seq 25 deny 172.16.0.0/16 ge 24
      ip prefix-list IBGP-DMVPN-GRE-FILTER seq 30 permit 0.0.0.0/0 le 32

      ```

      Настройки на R13 (резервный Хаб):

      ```

      interface Tunnel10
       description MGRE to ISP2 (DMVPN)
       ip address 10.101.0.1 255.255.255.0
       no ip redirects
       ip nhrp authentication CISCO
       ip nhrp network-id 101
       ip nhrp holdtime 90
       tunnel source 10.10.13.254
       tunnel mode gre multipoint
       tunnel key 11
       tunnel route-via Ethernet0/2 preferred
      !
      router bgp 1001
       ...
       bgp listen range 10.101.0.0/24 peer-group DMVPN-CLOUD2
       ...
       neighbor DMVPN-CLOUD2 peer-group
       neighbor DMVPN-CLOUD2 remote-as 1001
       neighbor DMVPN-CLOUD2 route-reflector-client
       neighbor DMVPN-CLOUD2 next-hop-self
       neighbor DMVPN-CLOUD2 prefix-list IBGP-DMVPN-GRE-FILTER out
      !
      ip prefix-list IBGP-DMVPN-GRE-FILTER seq 10 deny 10.10.13.0/24
      ip prefix-list IBGP-DMVPN-GRE-FILTER seq 20 deny 10.10.14.0/24
      ip prefix-list IBGP-DMVPN-GRE-FILTER seq 25 deny 172.16.0.0/16 ge 24
      ip prefix-list IBGP-DMVPN-GRE-FILTER seq 30 permit 0.0.0.0/0 le 32

      ```

      Настройки на R61 (спок в Лабытнанги):

      ```

      interface Tunnel10
       description MGRE to ISP (DMVPN)
       ip address 10.100.0.2 255.255.255.0
       no ip redirects
       ip nhrp authentication CISCO
       ip nhrp network-id 100
       ip nhrp holdtime 90
       ip nhrp nhs 10.100.0.1 nbma 10.10.14.254
       tunnel source Ethernet0/0
       tunnel mode gre multipoint
       tunnel key 10
       tunnel route-via Ethernet0/0 preferred
      !
      interface Tunnel20
       description MGRE to ISP (DMVPN)
       ip address 10.101.0.2 255.255.255.0
       no ip redirects
       ip nhrp authentication CISCO
       ip nhrp network-id 101
       ip nhrp holdtime 90
       ip nhrp nhs 10.101.0.1 nbma 10.10.13.254
       tunnel source Ethernet0/0
       tunnel mode gre multipoint
       tunnel key 11
       tunnel route-via Ethernet0/0 preferred
      !
      router bgp 1001
       bgp log-neighbor-changes
       network 10.60.0.61 mask 255.255.255.255
       timers bgp 3 9
       neighbor 10.100.0.1 remote-as 1001
       neighbor 10.100.0.1 route-map IBGP-DMVPN-CLOUD1-LP in
       neighbor 10.101.0.1 remote-as 1001
      !
      ip prefix-list ALL seq 5 permit 0.0.0.0/0 le 32
      !
      route-map IBGP-DMVPN-CLOUD1-LP permit 10
       match ip address prefix-list ALL
       set local-preference 150

      ```

      Настройки на R71 (спок в Чокурдак):

      ```

      interface Tunnel10
       description MGRE to ISP1 (DMVPN)
       ip address 10.100.0.3 255.255.255.0
       no ip redirects
       ip nhrp authentication CISCO
       ip nhrp network-id 100
       ip nhrp holdtime 90
       ip nhrp nhs 10.100.0.1 nbma 10.10.14.254
       tunnel source Ethernet0/0
       tunnel mode gre multipoint
       tunnel key 10
       tunnel route-via Ethernet0/0 preferred
      !
      interface Tunnel20
       description MGRE to ISP2 (DMVPN)
       ip address 10.101.0.3 255.255.255.0
       no ip redirects
       ip nhrp authentication CISCO
       ip nhrp network-id 101
       ip nhrp holdtime 90
       ip nhrp nhs 10.101.0.1 nbma 10.10.13.254
       tunnel source Ethernet0/1
       tunnel mode gre multipoint
       tunnel key 11
       tunnel route-via Ethernet0/1 preferred
      !
      router bgp 1001
       bgp log-neighbor-changes
       network 10.70.0.71 mask 255.255.255.255
       network 10.70.1.0 mask 255.255.255.0
       network 10.70.255.0 mask 255.255.255.0
       timers bgp 3 9
       neighbor 10.100.0.1 remote-as 1001
       neighbor 10.100.0.1 route-map IBGP-DMVPN-CLOUD1-LP in
       neighbor 10.101.0.1 remote-as 1001
      !
      route-map IBGP-DMVPN-CLOUD1-LP permit 10
       match ip address prefix-list ALL
       set local-preference 150

      ```

## Проверка работоспособности стенда

1. Проверка GRE между офисами Москва и Санкт-Петербург.

      Выполним проверку доступности устройств VPC51 и VPC52 офиса Санкт-Петербург из офиса Москва.  
      Провека будет выполняться с устройства VPC11. 

      ```

      VPC11> ping 10.50.1.51

      84 bytes from 10.50.1.51 icmp_seq=1 ttl=60 time=3.125 ms
      84 bytes from 10.50.1.51 icmp_seq=2 ttl=60 time=2.092 ms
      84 bytes from 10.50.1.51 icmp_seq=3 ttl=60 time=1.776 ms
      84 bytes from 10.50.1.51 icmp_seq=4 ttl=60 time=2.223 ms
      84 bytes from 10.50.1.51 icmp_seq=5 ttl=60 time=2.000 ms

      VPC11> ping 10.50.2.52

      84 bytes from 10.50.2.52 icmp_seq=1 ttl=60 time=2.897 ms
      84 bytes from 10.50.2.52 icmp_seq=2 ttl=60 time=2.361 ms
      84 bytes from 10.50.2.52 icmp_seq=3 ttl=60 time=2.121 ms
      84 bytes from 10.50.2.52 icmp_seq=4 ttl=60 time=1.979 ms
      84 bytes from 10.50.2.52 icmp_seq=5 ttl=60 time=7.448 ms

      ```

      Видим, что проверка доступности выполнена успешно.  
      Дополнительная информация о работе протокола BGP приведена в главе "Конфигурации устройств".


1. Проверка DMVPN (без шифрования) между офисами Москва и Чокурдак, Лабытнанги.

      Выполним проверку доступности устройств VPC71 и VPC72 офиса Чокурдак и роутера R61 офиса Лабытнанги из офиса Москва.  
      Провека будет выполняться с устройства VPC11.

      ```

      VPC11> ping 10.70.1.71

      84 bytes from 10.70.1.71 icmp_seq=1 ttl=61 time=2.657 ms
      84 bytes from 10.70.1.71 icmp_seq=2 ttl=61 time=1.972 ms
      84 bytes from 10.70.1.71 icmp_seq=3 ttl=61 time=1.825 ms
      84 bytes from 10.70.1.71 icmp_seq=4 ttl=61 time=2.002 ms
      84 bytes from 10.70.1.71 icmp_seq=5 ttl=61 time=1.977 ms

      VPC11> ping 10.70.1.72

      84 bytes from 10.70.1.72 icmp_seq=1 ttl=61 time=2.908 ms
      84 bytes from 10.70.1.72 icmp_seq=2 ttl=61 time=1.861 ms
      84 bytes from 10.70.1.72 icmp_seq=3 ttl=61 time=1.793 ms
      84 bytes from 10.70.1.72 icmp_seq=4 ttl=61 time=1.994 ms
      84 bytes from 10.70.1.72 icmp_seq=5 ttl=61 time=1.725 ms

      VPC11> ping 10.60.0.61

      84 bytes from 10.60.0.61 icmp_seq=1 ttl=253 time=2.221 ms
      84 bytes from 10.60.0.61 icmp_seq=2 ttl=253 time=1.915 ms
      84 bytes from 10.60.0.61 icmp_seq=3 ttl=253 time=1.840 ms
      84 bytes from 10.60.0.61 icmp_seq=4 ttl=253 time=1.833 ms
      84 bytes from 10.60.0.61 icmp_seq=5 ttl=253 time=2.093 ms

      ```
      Видим, что проверка доступности выполнена успешно.  
      Дополнительная информация о работе DMVPN приведена в главе "Конфигурации устройств".

## Конфигурации устройств

### R13

<details>
  <summary>Конфигурация</summary>

```

R13#terminal length 0
R13#sh run
Building configuration...

Current configuration : 4217 bytes
!
! Last configuration change at 16:26:12 UTC Sat Feb 22 2025
!
version 15.4
service timestamps debug datetime msec
service timestamps log datetime msec
service password-encryption
!
hostname R13
!
boot-start-marker
boot-end-marker
!
!
enable password 7 060506324F41
!
aaa new-model
!
!
!
!
!
!
!
aaa session-id common
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
ip domain name R13.local
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
username cisco privilege 15 password 7 02050D480809
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
interface Tunnel0
 description to SPB ISP1
 ip address 192.168.3.1 255.255.255.0
 tunnel source 10.10.13.254
 tunnel destination 10.50.52.126
 tunnel key 2
!
interface Tunnel1
 description to SPB ISP2
 ip address 192.168.4.1 255.255.255.0
 tunnel source 10.10.13.254
 tunnel destination 10.50.52.254
 tunnel key 2
!
interface Tunnel10
 description MGRE to ISP2 (DMVPN)
 ip address 10.101.0.1 255.255.255.0
 no ip redirects
 ip nhrp authentication CISCO
 ip nhrp network-id 101
 ip nhrp holdtime 90
 tunnel source 10.10.13.254
 tunnel mode gre multipoint
 tunnel key 11
 tunnel route-via Ethernet0/2 preferred
!
interface Ethernet0/0
 ip address 10.10.3.13 255.255.255.0
 ip nat enable
 ip ospf 1 area 0
!
interface Ethernet0/1
 ip address 10.10.7.13 255.255.255.0
 ip nat enable
 ip ospf 1 area 0
!
interface Ethernet0/2
 ip address 10.10.13.254 255.255.255.0 secondary
 ip address 172.16.1.13 255.255.255.0
 ip nat enable
!
interface Ethernet0/3
 ip address 10.10.4.13 255.255.255.0
 ip nat enable
 ip ospf 1 area 101
!
router ospf 1
 router-id 10.10.0.13
 area 101 filter-list prefix OSPF-FILTER in
 default-information originate
!
router bgp 1001
 bgp log-neighbor-changes
 bgp listen range 10.101.0.0/24 peer-group DMVPN-CLOUD2
 network 10.10.13.0 mask 255.255.255.0
 timers bgp 3 9
 redistribute ospf 1
 neighbor DMVPN-CLOUD2 peer-group
 neighbor DMVPN-CLOUD2 remote-as 1001
 neighbor DMVPN-CLOUD2 route-reflector-client
 neighbor DMVPN-CLOUD2 next-hop-self
 neighbor DMVPN-CLOUD2 prefix-list IBGP-DMVPN-GRE-FILTER out
 neighbor 172.16.1.21 remote-as 101
 neighbor 172.16.1.21 prefix-list EBGP-ISP-FILTER out
 neighbor 192.168.3.2 remote-as 2042
 neighbor 192.168.3.2 ebgp-multihop 255
 neighbor 192.168.3.2 update-source Tunnel0
 neighbor 192.168.3.2 prefix-list EBGP-GRE-FILTER out
 neighbor 192.168.3.2 route-map EBGP-GRE-LP in
 neighbor 192.168.3.2 filter-list 1 out
 neighbor 192.168.4.2 remote-as 2042
 neighbor 192.168.4.2 ebgp-multihop 255
 neighbor 192.168.4.2 update-source Tunnel1
 neighbor 192.168.4.2 prefix-list EBGP-GRE-FILTER out
 neighbor 192.168.4.2 filter-list 1 out
!
ip forward-protocol nd
!
ip as-path access-list 1 permit ^$
!
no ip http server
no ip http secure-server
ip nat pool PUBLIC_POOL 10.10.13.1 10.10.13.10 netmask 255.255.255.0
ip nat source route-map NAT_FOR_G0/2 pool PUBLIC_POOL overload
ip nat source static tcp 10.10.0.15 22 10.10.13.11 22 extendable
!
!
ip prefix-list ALL seq 5 permit 0.0.0.0/0 le 32
!
ip prefix-list EBGP-GRE-FILTER seq 10 deny 10.10.13.0/24
ip prefix-list EBGP-GRE-FILTER seq 20 deny 10.10.14.0/24
ip prefix-list EBGP-GRE-FILTER seq 25 deny 172.16.0.0/16 ge 24
ip prefix-list EBGP-GRE-FILTER seq 30 permit 0.0.0.0/0 le 32
!
ip prefix-list EBGP-ISP-FILTER seq 10 permit 10.10.13.0/24
!
ip prefix-list IBGP-DMVPN-GRE-FILTER seq 10 deny 10.10.13.0/24
ip prefix-list IBGP-DMVPN-GRE-FILTER seq 20 deny 10.10.14.0/24
ip prefix-list IBGP-DMVPN-GRE-FILTER seq 25 deny 172.16.0.0/16 ge 24
ip prefix-list IBGP-DMVPN-GRE-FILTER seq 30 permit 0.0.0.0/0 le 32
!
ip prefix-list OSPF-FILTER seq 10 deny 10.10.0.0/16 le 32
ip prefix-list OSPF-FILTER seq 20 permit 0.0.0.0/0 le 32
!
route-map EBGP-GRE-LP permit 10
 match ip address prefix-list ALL
 set local-preference 150
!
route-map NAT_FOR_G0/2 permit 10
 match ip address 2
 match interface Ethernet0/2
!
!
access-list 1 permit any
access-list 2 permit 10.10.0.0 0.0.255.255
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
 transport input ssh
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
Ethernet0/2                172.16.1.13     YES manual up                    up
Ethernet0/3                10.10.4.13      YES NVRAM  up                    up
Loopback1                  10.10.0.13      YES NVRAM  up                    up
NVI0                       10.10.3.13      YES unset  up                    up
Tunnel0                    192.168.3.1     YES manual up                    up
Tunnel1                    192.168.4.1     YES manual up                    up
Tunnel10                   10.101.0.1      YES manual up                    up
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

Gateway of last resort is 172.16.1.21 to network 0.0.0.0

B*    0.0.0.0/0 [20/0] via 172.16.1.21, 6d19h
      10.0.0.0/8 is variably subnetted, 39 subnets, 4 masks
O IA     10.10.0.11/32 [110/11] via 10.10.3.11, 2w0d, Ethernet0/0
O IA     10.10.0.12/32 [110/11] via 10.10.7.12, 2w0d, Ethernet0/1
C        10.10.0.13/32 is directly connected, Loopback1
O        10.10.0.14/32 [110/12] via 10.10.7.12, 6d19h, Ethernet0/1
                       [110/12] via 10.10.3.11, 6d19h, Ethernet0/0
O        10.10.0.15/32 [110/11] via 10.10.4.15, 2w0d, Ethernet0/3
O IA     10.10.0.16/32 [110/22] via 10.10.7.12, 6d19h, Ethernet0/1
                       [110/22] via 10.10.3.11, 6d19h, Ethernet0/0
O IA     10.10.1.0/24 [110/20] via 10.10.7.12, 2w0d, Ethernet0/1
                      [110/20] via 10.10.3.11, 2w0d, Ethernet0/0
O IA     10.10.2.0/24 [110/20] via 10.10.7.12, 2w0d, Ethernet0/1
                      [110/20] via 10.10.3.11, 2w0d, Ethernet0/0
C        10.10.3.0/24 is directly connected, Ethernet0/0
L        10.10.3.13/32 is directly connected, Ethernet0/0
C        10.10.4.0/24 is directly connected, Ethernet0/3
L        10.10.4.13/32 is directly connected, Ethernet0/3
O        10.10.5.0/24 [110/11] via 10.10.7.12, 6d20h, Ethernet0/1
O IA     10.10.6.0/24 [110/21] via 10.10.7.12, 6d19h, Ethernet0/1
                      [110/21] via 10.10.3.11, 6d19h, Ethernet0/0
C        10.10.7.0/24 is directly connected, Ethernet0/1
L        10.10.7.13/32 is directly connected, Ethernet0/1
O        10.10.8.0/24 [110/11] via 10.10.3.11, 6d19h, Ethernet0/0
O IA     10.10.9.0/24 [110/20] via 10.10.7.12, 2w0d, Ethernet0/1
                      [110/20] via 10.10.3.11, 2w0d, Ethernet0/0
C        10.10.13.0/24 is directly connected, Ethernet0/2
L        10.10.13.11/32 is directly connected, Ethernet0/2
L        10.10.13.254/32 is directly connected, Ethernet0/2
O IA     10.10.255.0/24 [110/20] via 10.10.7.12, 2w0d, Ethernet0/1
                        [110/20] via 10.10.3.11, 2w0d, Ethernet0/0
B        10.50.0.51/32 [20/1024640] via 192.168.3.2, 6d19h
B        10.50.0.52/32 [20/0] via 192.168.3.2, 6d19h
B        10.50.0.53/32 [20/1024640] via 192.168.3.2, 6d19h
B        10.50.0.54/32 [20/1536640] via 192.168.3.2, 6d19h
B        10.50.1.0/24 [20/1536000] via 192.168.3.2, 6d19h
B        10.50.2.0/24 [20/1536000] via 192.168.3.2, 6d19h
B        10.50.3.0/24 [20/0] via 192.168.3.2, 6d19h
B        10.50.4.0/24 [20/0] via 192.168.3.2, 6d19h
B        10.50.5.0/24 [20/1536000] via 192.168.3.2, 6d19h
B        10.50.6.0/24 [20/1536000] via 192.168.3.2, 6d19h
B        10.50.255.0/24 [20/1536000] via 192.168.3.2, 6d19h
B        10.60.0.61/32 [200/0] via 10.101.0.2, 22:01:27
B        10.70.0.71/32 [200/0] via 10.101.0.3, 21:48:50
B        10.70.1.0/24 [200/0] via 10.101.0.3, 21:42:55
B        10.70.255.0/24 [200/0] via 10.101.0.3, 21:43:44
C        10.101.0.0/24 is directly connected, Tunnel10
L        10.101.0.1/32 is directly connected, Tunnel10
      172.16.0.0/16 is variably subnetted, 2 subnets, 2 masks
C        172.16.1.0/24 is directly connected, Ethernet0/2
L        172.16.1.13/32 is directly connected, Ethernet0/2
      192.168.3.0/24 is variably subnetted, 2 subnets, 2 masks
C        192.168.3.0/24 is directly connected, Tunnel0
L        192.168.3.1/32 is directly connected, Tunnel0
      192.168.4.0/24 is variably subnetted, 2 subnets, 2 masks
C        192.168.4.0/24 is directly connected, Tunnel1
L        192.168.4.1/32 is directly connected, Tunnel1
R13#
R13#
R13#
R13#
R13#show ip bgp
BGP table version is 139, local router ID is 10.10.0.13
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter,
              x best-external, a additional-path, c RIB-compressed,
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 *>  0.0.0.0          172.16.1.21                            0 101 i
 *>  10.10.0.11/32    10.10.3.11              11         32768 ?
 *>  10.10.0.12/32    10.10.7.12              11         32768 ?
 *>  10.10.0.13/32    0.0.0.0                  0         32768 ?
 *>  10.10.0.14/32    10.10.3.11              12         32768 ?
 *>  10.10.0.15/32    10.10.4.15              11         32768 ?
 *>  10.10.0.16/32    10.10.3.11              22         32768 ?
 *>  10.10.1.0/24     10.10.3.11              20         32768 ?
 *>  10.10.2.0/24     10.10.3.11              20         32768 ?
 *>  10.10.3.0/24     0.0.0.0                  0         32768 ?
 *>  10.10.4.0/24     0.0.0.0                  0         32768 ?
 *>  10.10.5.0/24     10.10.7.12              11         32768 ?
 *>  10.10.6.0/24     10.10.3.11              21         32768 ?
 *>  10.10.7.0/24     0.0.0.0                  0         32768 ?
 *>  10.10.8.0/24     10.10.3.11              11         32768 ?
 *>  10.10.9.0/24     10.10.3.11              20         32768 ?
 *>  10.10.13.0/24    0.0.0.0                  0         32768 i
 *>  10.10.255.0/24   10.10.3.11              20         32768 ?
 *>  10.50.0.51/32    192.168.3.2        1024640    150      0 2042 ?
 *                    192.168.4.2        1024640             0 2042 ?
 *>  10.50.0.52/32    192.168.3.2              0    150      0 2042 ?
 *                    192.168.4.2              0             0 2042 ?
 *>  10.50.0.53/32    192.168.3.2        1024640    150      0 2042 ?
 *                    192.168.4.2        1024640             0 2042 ?
 *>  10.50.0.54/32    192.168.3.2        1536640    150      0 2042 ?
 *                    192.168.4.2        1536640             0 2042 ?
 *>  10.50.1.0/24     192.168.3.2        1536000    150      0 2042 ?
 *                    192.168.4.2        1536000             0 2042 ?
 *>  10.50.2.0/24     192.168.3.2        1536000    150      0 2042 ?
 *                    192.168.4.2        1536000             0 2042 ?
 *>  10.50.3.0/24     192.168.3.2              0    150      0 2042 ?
 *                    192.168.4.2              0             0 2042 ?
 *>  10.50.4.0/24     192.168.3.2              0    150      0 2042 ?
 *                    192.168.4.2              0             0 2042 ?
 *>  10.50.5.0/24     192.168.3.2        1536000    150      0 2042 ?
 *                    192.168.4.2        1536000             0 2042 ?
 *>  10.50.6.0/24     192.168.3.2        1536000    150      0 2042 ?
 *                    192.168.4.2        1536000             0 2042 ?
 *>  10.50.255.0/24   192.168.3.2        1536000    150      0 2042 ?
 *                    192.168.4.2        1536000             0 2042 ?
 *>i 10.60.0.61/32    10.101.0.2               0    100      0 i
 *>i 10.70.0.71/32    10.101.0.3               0    100      0 i
 *>i 10.70.1.0/24     10.101.0.3               0    100      0 i
 *>i 10.70.255.0/24   10.101.0.3               0    100      0 i
R13#
R13#
R13#
R13#
R13#show ip bgp summary
BGP router identifier 10.10.0.13, local AS number 1001
BGP table version is 139, main routing table version 139
33 network entries using 4620 bytes of memory
44 path entries using 3520 bytes of memory
17/13 BGP path/bestpath attribute entries using 2448 bytes of memory
2 BGP AS-PATH entries using 48 bytes of memory
0 BGP route-map cache entries using 0 bytes of memory
0 BGP filter-list cache entries using 0 bytes of memory
BGP using 10636 total bytes of memory
BGP activity 567/534 prefixes, 1340/1296 paths, scan interval 60 secs

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
*10.101.0.2     4         1001   25819   25857      139    0    0 22:01:44        1
*10.101.0.3     4         1001   25573   25583      139    0    0 21:48:50        3
172.16.1.21     4          101   10831   10829      139    0    0 6d19h           1
192.168.3.2     4         2042  191458  191468      139    0    0 6d19h          11
192.168.4.2     4         2042   10819   10842      139    0    0 6d19h          11
* Dynamically created based on a listen range command
Dynamically created neighbors: 2, Subnet ranges: 1

BGP peergroup DMVPN-CLOUD2 listen range group members:
  10.101.0.0/24

Total dynamically created neighbors: 2/(100 max), Subnet ranges: 1

R13#
R13#
R13#
R13#
R13#show ip nhrp traffic
Tunnel10: Max-send limit:100Pkts/10Sec, Usage:0%
   Sent: Total 5412
         3 Resolution Request  0 Resolution Reply  0 Registration Request
         5406 Registration Reply  2 Purge Request  1 Purge Reply
         0 Error Indication  0 Traffic Indication  0 Redirect Suppress
   Rcvd: Total 5412
         3 Resolution Request  0 Resolution Reply  5406 Registration Request
         0 Registration Reply  2 Purge Request  1 Purge Reply
         0 Error Indication  0 Traffic Indication  0 Redirect Suppress
R13#
R13#
R13#
R13#
R13#show ip nhrp detail
10.101.0.2/32 via 10.101.0.2
   Tunnel10 created 22:36:19, expire 00:01:11
   Type: dynamic, Flags: unique registered used nhop
   NBMA address: 172.16.9.61
10.101.0.3/32 via 10.101.0.3
   Tunnel10 created 21:50:30, expire 00:01:03
   Type: dynamic, Flags: unique registered used nhop
   NBMA address: 172.16.10.71
R13#
R13#
R13#
R13#
R13#show dmvpn
Legend: Attrb --> S - Static, D - Dynamic, I - Incomplete
        N - NATed, L - Local, X - No Socket
        # Ent --> Number of NHRP entries with same NBMA peer
        NHS Status: E --> Expecting Replies, R --> Responding, W --> Waiting
        UpDn Time --> Up or Down Time for a Tunnel
==========================================================================

Interface: Tunnel10, IPv4 NHRP Details
Type:Hub, NHRP Peers:2,

 # Ent  Peer NBMA Addr Peer Tunnel Add State  UpDn Tm Attrb
 ----- --------------- --------------- ----- -------- -----
     1 172.16.9.61          10.101.0.2    UP 22:36:19     D
     1 172.16.10.71         10.101.0.3    UP 21:48:57     D



```
</details>

### R14

<details>
  <summary>Конфигурация</summary>

```

R14#terminal length 0
R14#sh run
Building configuration...

Current configuration : 4056 bytes
!
! Last configuration change at 16:26:57 UTC Sat Feb 22 2025
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
interface Tunnel0
 description to SPB ISP1
 ip address 192.168.1.1 255.255.255.0
 tunnel source 10.10.14.254
 tunnel destination 10.50.52.126
 tunnel key 1
!
interface Tunnel1
 description to SPB ISP2
 ip address 192.168.2.1 255.255.255.0
 tunnel source 10.10.14.254
 tunnel destination 10.50.52.254
 tunnel key 1
!
interface Tunnel10
 description MGRE to ISP1 (DMVPN)
 ip address 10.100.0.1 255.255.255.0
 no ip redirects
 ip nhrp authentication CISCO
 ip nhrp network-id 100
 ip nhrp holdtime 90
 tunnel source 10.10.14.254
 tunnel mode gre multipoint
 tunnel key 10
 tunnel route-via Ethernet0/2 preferred
!
interface Ethernet0/0
 ip address 10.10.5.14 255.255.255.0
 ip nat enable
 ip ospf 1 area 0
!
interface Ethernet0/1
 ip address 10.10.8.14 255.255.255.0
 ip nat enable
 ip ospf 1 area 0
!
interface Ethernet0/2
 ip address 10.10.14.254 255.255.255.0 secondary
 ip address 172.16.2.14 255.255.255.0
 ip nat enable
!
interface Ethernet0/3
 ip address 10.10.6.14 255.255.255.0
 ip nat enable
 ip ospf 1 area 102
!
router ospf 1
 router-id 10.10.0.14
 area 102 filter-list prefix OSPF-FILTER in
 default-information originate
!
router bgp 1001
 bgp log-neighbor-changes
 bgp listen range 10.100.0.0/24 peer-group DMVPN-CLOUD1
 network 10.10.14.0 mask 255.255.255.0
 timers bgp 3 9
 redistribute ospf 1
 neighbor DMVPN-CLOUD1 peer-group
 neighbor DMVPN-CLOUD1 remote-as 1001
 neighbor DMVPN-CLOUD1 route-reflector-client
 neighbor DMVPN-CLOUD1 next-hop-self
 neighbor DMVPN-CLOUD1 prefix-list IBGP-DMVPN-GRE-FILTER out
 neighbor 172.16.2.31 remote-as 301
 neighbor 172.16.2.31 prefix-list EBGP-ISP-FILTER out
 neighbor 192.168.1.2 remote-as 2042
 neighbor 192.168.1.2 ebgp-multihop 255
 neighbor 192.168.1.2 update-source Tunnel0
 neighbor 192.168.1.2 prefix-list EBGP-GRE-FILTER out
 neighbor 192.168.1.2 route-map EBGP-GRE-LP in
 neighbor 192.168.1.2 filter-list 1 out
 neighbor 192.168.2.2 remote-as 2042
 neighbor 192.168.2.2 ebgp-multihop 255
 neighbor 192.168.2.2 update-source Tunnel1
 neighbor 192.168.2.2 prefix-list EBGP-GRE-FILTER out
 neighbor 192.168.2.2 filter-list 1 out
!
ip forward-protocol nd
!
ip as-path access-list 1 permit ^$
!
no ip http server
no ip http secure-server
ip nat pool PUBLIC_POOL 10.10.14.1 10.10.14.10 netmask 255.255.255.0
ip nat source route-map NAT_FOR_G0/2 pool PUBLIC_POOL overload
ip nat source static 10.10.0.16 10.10.14.11
!
!
ip prefix-list ALL seq 5 permit 0.0.0.0/0 le 32
!
ip prefix-list EBGP-GRE-FILTER seq 10 deny 10.10.13.0/24
ip prefix-list EBGP-GRE-FILTER seq 20 deny 10.10.14.0/24
ip prefix-list EBGP-GRE-FILTER seq 25 deny 172.16.0.0/16 ge 24
ip prefix-list EBGP-GRE-FILTER seq 30 permit 0.0.0.0/0 le 32
!
ip prefix-list EBGP-ISP-FILTER seq 10 permit 10.10.14.0/24
!
ip prefix-list IBGP-DMVPN-GRE-FILTER seq 10 deny 10.10.13.0/24
ip prefix-list IBGP-DMVPN-GRE-FILTER seq 20 deny 10.10.14.0/24
ip prefix-list IBGP-DMVPN-GRE-FILTER seq 25 deny 172.16.0.0/16 ge 24
ip prefix-list IBGP-DMVPN-GRE-FILTER seq 30 permit 0.0.0.0/0 le 32
!
ip prefix-list OSPF-FILTER seq 10 deny 10.10.4.0/24
ip prefix-list OSPF-FILTER seq 20 permit 0.0.0.0/0 le 32
!
route-map EBGP-GRE-LP permit 10
 match ip address prefix-list ALL
 set local-preference 150
!
route-map NAT_FOR_G0/2 permit 10
 match ip address 2
 match interface Ethernet0/2
!
!
access-list 1 permit any
access-list 2 permit 10.10.0.0 0.0.255.255
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
NVI0                       10.10.5.14      YES unset  up                    up
Tunnel0                    192.168.1.1     YES NVRAM  up                    up
Tunnel1                    192.168.2.1     YES NVRAM  up                    up
Tunnel10                   10.100.0.1      YES manual up                    up
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

B*    0.0.0.0/0 [20/0] via 172.16.2.31, 6d19h
      10.0.0.0/8 is variably subnetted, 39 subnets, 2 masks
O IA     10.10.0.11/32 [110/11] via 10.10.8.11, 6d19h, Ethernet0/1
O IA     10.10.0.12/32 [110/11] via 10.10.5.12, 6d19h, Ethernet0/0
O        10.10.0.13/32 [110/21] via 10.10.8.11, 6d19h, Ethernet0/1
                       [110/21] via 10.10.5.12, 6d19h, Ethernet0/0
C        10.10.0.14/32 is directly connected, Loopback1
O IA     10.10.0.15/32 [110/31] via 10.10.8.11, 6d19h, Ethernet0/1
                       [110/31] via 10.10.5.12, 6d19h, Ethernet0/0
O        10.10.0.16/32 [110/11] via 10.10.6.16, 6d19h, Ethernet0/3
O IA     10.10.1.0/24 [110/20] via 10.10.8.11, 6d19h, Ethernet0/1
                      [110/20] via 10.10.5.12, 6d19h, Ethernet0/0
O IA     10.10.2.0/24 [110/20] via 10.10.8.11, 6d19h, Ethernet0/1
                      [110/20] via 10.10.5.12, 6d19h, Ethernet0/0
O        10.10.3.0/24 [110/20] via 10.10.8.11, 6d19h, Ethernet0/1
O IA     10.10.4.0/24 [110/30] via 10.10.8.11, 6d19h, Ethernet0/1
                      [110/30] via 10.10.5.12, 6d19h, Ethernet0/0
C        10.10.5.0/24 is directly connected, Ethernet0/0
L        10.10.5.14/32 is directly connected, Ethernet0/0
C        10.10.6.0/24 is directly connected, Ethernet0/3
L        10.10.6.14/32 is directly connected, Ethernet0/3
O        10.10.7.0/24 [110/20] via 10.10.5.12, 6d19h, Ethernet0/0
C        10.10.8.0/24 is directly connected, Ethernet0/1
L        10.10.8.14/32 is directly connected, Ethernet0/1
O IA     10.10.9.0/24 [110/20] via 10.10.8.11, 6d19h, Ethernet0/1
                      [110/20] via 10.10.5.12, 6d19h, Ethernet0/0
C        10.10.14.0/24 is directly connected, Ethernet0/2
L        10.10.14.11/32 is directly connected, Ethernet0/2
L        10.10.14.254/32 is directly connected, Ethernet0/2
O IA     10.10.255.0/24 [110/20] via 10.10.8.11, 6d19h, Ethernet0/1
                        [110/20] via 10.10.5.12, 6d19h, Ethernet0/0
B        10.50.0.51/32 [20/1024640] via 192.168.1.2, 6d19h
B        10.50.0.52/32 [20/0] via 192.168.1.2, 6d19h
B        10.50.0.53/32 [20/1024640] via 192.168.1.2, 6d19h
B        10.50.0.54/32 [20/1536640] via 192.168.1.2, 6d19h
B        10.50.1.0/24 [20/1536000] via 192.168.1.2, 6d19h
B        10.50.2.0/24 [20/1536000] via 192.168.1.2, 6d19h
B        10.50.3.0/24 [20/0] via 192.168.1.2, 6d19h
B        10.50.4.0/24 [20/0] via 192.168.1.2, 6d19h
B        10.50.5.0/24 [20/1536000] via 192.168.1.2, 6d19h
B        10.50.6.0/24 [20/1536000] via 192.168.1.2, 6d19h
B        10.50.255.0/24 [20/1536000] via 192.168.1.2, 6d19h
B        10.60.0.61/32 [200/0] via 10.100.0.2, 22:03:28
B        10.70.0.71/32 [200/0] via 10.100.0.3, 00:11:01
B        10.70.1.0/24 [200/0] via 10.100.0.3, 00:11:01
B        10.70.255.0/24 [200/0] via 10.100.0.3, 00:11:01
C        10.100.0.0/24 is directly connected, Tunnel10
L        10.100.0.1/32 is directly connected, Tunnel10
      172.16.0.0/16 is variably subnetted, 2 subnets, 2 masks
C        172.16.2.0/24 is directly connected, Ethernet0/2
L        172.16.2.14/32 is directly connected, Ethernet0/2
      192.168.1.0/24 is variably subnetted, 2 subnets, 2 masks
C        192.168.1.0/24 is directly connected, Tunnel0
L        192.168.1.1/32 is directly connected, Tunnel0
      192.168.2.0/24 is variably subnetted, 2 subnets, 2 masks
C        192.168.2.0/24 is directly connected, Tunnel1
L        192.168.2.1/32 is directly connected, Tunnel1
R14#
R14#
R14#
R14#
R14#show ip bgp
BGP table version is 123, local router ID is 10.10.0.14
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter,
              x best-external, a additional-path, c RIB-compressed,
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 *>  0.0.0.0          172.16.2.31                            0 301 i
 *>  10.10.0.11/32    10.10.8.11              11         32768 ?
 *>  10.10.0.12/32    10.10.5.12              11         32768 ?
 *>  10.10.0.13/32    10.10.5.12              21         32768 ?
 *>  10.10.0.14/32    0.0.0.0                  0         32768 ?
 *>  10.10.0.15/32    10.10.5.12              31         32768 ?
 *>  10.10.0.16/32    10.10.6.16              11         32768 ?
 *>  10.10.1.0/24     10.10.5.12              20         32768 ?
 *>  10.10.2.0/24     10.10.5.12              20         32768 ?
 *>  10.10.3.0/24     10.10.8.11              20         32768 ?
 *>  10.10.4.0/24     10.10.5.12              30         32768 ?
 *>  10.10.5.0/24     0.0.0.0                  0         32768 ?
 *>  10.10.6.0/24     0.0.0.0                  0         32768 ?
 *>  10.10.7.0/24     10.10.5.12              20         32768 ?
 *>  10.10.8.0/24     0.0.0.0                  0         32768 ?
 *>  10.10.9.0/24     10.10.5.12              20         32768 ?
 *>  10.10.14.0/24    0.0.0.0                  0         32768 i
 *>  10.10.255.0/24   10.10.5.12              20         32768 ?
 *>  10.50.0.51/32    192.168.1.2        1024640    150      0 2042 ?
 *                    192.168.2.2        1024640             0 2042 ?
 *>  10.50.0.52/32    192.168.1.2              0    150      0 2042 ?
 *                    192.168.2.2              0             0 2042 ?
 *>  10.50.0.53/32    192.168.1.2        1024640    150      0 2042 ?
 *                    192.168.2.2        1024640             0 2042 ?
 *>  10.50.0.54/32    192.168.1.2        1536640    150      0 2042 ?
 *                    192.168.2.2        1536640             0 2042 ?
 *>  10.50.1.0/24     192.168.1.2        1536000    150      0 2042 ?
 *                    192.168.2.2        1536000             0 2042 ?
 *>  10.50.2.0/24     192.168.1.2        1536000    150      0 2042 ?
 *                    192.168.2.2        1536000             0 2042 ?
 *>  10.50.3.0/24     192.168.1.2              0    150      0 2042 ?
 *                    192.168.2.2              0             0 2042 ?
 *>  10.50.4.0/24     192.168.1.2              0    150      0 2042 ?
 *                    192.168.2.2              0             0 2042 ?
 *>  10.50.5.0/24     192.168.1.2        1536000    150      0 2042 ?
 *                    192.168.2.2        1536000             0 2042 ?
 *>  10.50.6.0/24     192.168.1.2        1536000    150      0 2042 ?
 *                    192.168.2.2        1536000             0 2042 ?
 *>  10.50.255.0/24   192.168.1.2        1536000    150      0 2042 ?
 *                    192.168.2.2        1536000             0 2042 ?
 *>i 10.60.0.61/32    10.100.0.2               0    100      0 i
 *>i 10.70.0.71/32    10.100.0.3               0    100      0 i
 *>i 10.70.1.0/24     10.100.0.3               0    100      0 i
 *>i 10.70.255.0/24   10.100.0.3               0    100      0 i
R14#
R14#
R14#
R14#
R14#show ip bgp summary
BGP router identifier 10.10.0.14, local AS number 1001
BGP table version is 123, main routing table version 123
33 network entries using 4620 bytes of memory
44 path entries using 3520 bytes of memory
17/13 BGP path/bestpath attribute entries using 2448 bytes of memory
2 BGP AS-PATH entries using 48 bytes of memory
0 BGP route-map cache entries using 0 bytes of memory
0 BGP filter-list cache entries using 0 bytes of memory
BGP using 10636 total bytes of memory
BGP activity 37/4 prefixes, 83/39 paths, scan interval 60 secs

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
*10.100.0.2     4         1001   25859   25888      123    0    0 22:03:45        1
*10.100.0.3     4         1001     222     234      123    0    0 00:11:00        3
172.16.2.31     4          301   10809   10799      123    0    0 6d19h           1
192.168.1.2     4         2042  191498  191507      123    0    0 6d19h          11
192.168.2.2     4         2042   10802   10793      123    0    0 6d19h          11
* Dynamically created based on a listen range command
Dynamically created neighbors: 2, Subnet ranges: 1

BGP peergroup DMVPN-CLOUD1 listen range group members:
  10.100.0.0/24

Total dynamically created neighbors: 2/(100 max), Subnet ranges: 1

R14#
R14#
R14#
R14#
R14#show ip nhrp traffic
Tunnel10: Max-send limit:100Pkts/10Sec, Usage:1%
   Sent: Total 5448
         6 Resolution Request  0 Resolution Reply  0 Registration Request
         5435 Registration Reply  3 Purge Request  3 Purge Reply
         1 Error Indication  0 Traffic Indication  0 Redirect Suppress
   Rcvd: Total 5448
         6 Resolution Request  0 Resolution Reply  5435 Registration Request
         0 Registration Reply  4 Purge Request  3 Purge Reply
         0 Error Indication  0 Traffic Indication  0 Redirect Suppress
R14#
R14#
R14#
R14#
R14#show ip nhrp detail
10.100.0.2/32 via 10.100.0.2
   Tunnel10 created 22:45:44, expire 00:01:16
   Type: dynamic, Flags: unique registered used nhop
   NBMA address: 172.16.9.61
10.100.0.3/32 via 10.100.0.3
   Tunnel10 created 00:11:10, expire 00:01:28
   Type: dynamic, Flags: unique registered used nhop
   NBMA address: 172.16.8.71
R14#
R14#
R14#
R14#
R14#show dmvpn
Legend: Attrb --> S - Static, D - Dynamic, I - Incomplete
        N - NATed, L - Local, X - No Socket
        # Ent --> Number of NHRP entries with same NBMA peer
        NHS Status: E --> Expecting Replies, R --> Responding, W --> Waiting
        UpDn Time --> Up or Down Time for a Tunnel
==========================================================================

Interface: Tunnel10, IPv4 NHRP Details
Type:Hub, NHRP Peers:2,

 # Ent  Peer NBMA Addr Peer Tunnel Add State  UpDn Tm Attrb
 ----- --------------- --------------- ----- -------- -----
     1 172.16.9.61          10.100.0.2    UP 22:45:44     D
     1 172.16.8.71          10.100.0.3    UP 00:11:01     D


```
</details>

### R42

<details>
  <summary>Конфигурация</summary>

```

R42#terminal length 0
R42#sh run
Building configuration...

Current configuration : 1553 bytes
!
! Last configuration change at 12:43:28 UTC Sun Feb 16 2025
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

      10.0.0.0/8 is variably subnetted, 16 subnets, 4 masks
B        10.10.13.0/24 [200/0] via 172.16.5.21, 6d19h
B        10.10.14.0/24 [200/0] via 172.16.4.31, 6d19h
B        10.20.0.0/16 [200/0] via 172.16.5.21, 3w6d
B        10.30.0.0/16 [200/0] via 172.16.4.31, 3w6d
i L1     10.40.0.41/32 [115/20] via 10.40.2.41, 4w1d, Ethernet0/0
C        10.40.0.42/32 is directly connected, Loopback1
i L2     10.40.0.43/32 [115/20] via 10.40.3.43, 4w1d, Ethernet0/2
i L2     10.40.0.44/32 [115/30] via 10.40.3.43, 4w1d, Ethernet0/2
                       [115/30] via 10.40.2.41, 4w1d, Ethernet0/0
i L1     10.40.1.0/24 [115/20] via 10.40.2.41, 4w1d, Ethernet0/0
C        10.40.2.0/24 is directly connected, Ethernet0/0
L        10.40.2.42/32 is directly connected, Ethernet0/0
C        10.40.3.0/24 is directly connected, Ethernet0/2
L        10.40.3.42/32 is directly connected, Ethernet0/2
i L2     10.40.4.0/24 [115/20] via 10.40.3.43, 4w1d, Ethernet0/2
B        10.50.52.0/25 [200/0] via 172.16.6.52, 6d19h
B        10.50.52.128/25 [200/0] via 172.16.7.52, 6d20h
      172.16.0.0/16 is variably subnetted, 12 subnets, 2 masks
B        172.16.1.0/24 [200/0] via 172.16.5.21, 3w6d
B        172.16.2.0/24 [200/0] via 172.16.4.31, 3w6d
B        172.16.3.0/24 [200/0] via 172.16.5.21, 3w6d
B        172.16.4.0/24 [200/0] via 10.40.0.44, 3w6d
B        172.16.5.0/24 [200/0] via 10.40.0.41, 4w1d
B        172.16.6.0/24 [200/0] via 10.40.0.44, 3w6d
B        172.16.7.0/24 [200/0] via 10.40.0.43, 4w1d
B        172.16.8.0/24 [200/0] via 10.40.0.43, 4w1d
C        172.16.9.0/24 is directly connected, Ethernet0/1
L        172.16.9.42/32 is directly connected, Ethernet0/1
C        172.16.10.0/24 is directly connected, Ethernet0/3
L        172.16.10.42/32 is directly connected, Ethernet0/3
R42#
R42#
R42#
R42#
R42#show ip bgp
BGP table version is 883, local router ID is 10.40.0.42
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter,
              x best-external, a additional-path, c RIB-compressed,
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 *>i 10.10.13.0/24    172.16.5.21              0    100      0 101 1001 i
 *>i 10.10.14.0/24    172.16.4.31              0    100      0 301 1001 i
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
 *>i 10.50.52.0/25    172.16.6.52              0    100      0 2042 i
 *>i 10.50.52.128/25  172.16.7.52              0    100      0 2042 i
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
BGP table version is 883, main routing table version 883
21 network entries using 2940 bytes of memory
25 path entries using 2000 bytes of memory
13/11 BGP path/bestpath attribute entries using 1872 bytes of memory
2 BGP rrinfo entries using 48 bytes of memory
5 BGP AS-PATH entries using 120 bytes of memory
0 BGP route-map cache entries using 0 bytes of memory
0 BGP filter-list cache entries using 0 bytes of memory
BGP using 6980 total bytes of memory
BGP activity 142/121 prefixes, 173/148 paths, scan interval 60 secs

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
10.40.0.41      4          520   46207   46013      883    0    0 4w1d           18
R42#


```
</details>

### R43

<details>
  <summary>Конфигурация</summary>

```

R43#terminal length 0
R43#sh run
Building configuration...

Current configuration : 1585 bytes
!
! Last configuration change at 12:43:38 UTC Sun Feb 16 2025
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

      10.0.0.0/8 is variably subnetted, 16 subnets, 4 masks
B        10.10.13.0/24 [200/0] via 172.16.5.21, 6d19h
B        10.10.14.0/24 [200/0] via 172.16.4.31, 6d19h
B        10.20.0.0/16 [200/0] via 172.16.5.21, 3w6d
B        10.30.0.0/16 [200/0] via 172.16.4.31, 3w6d
i L2     10.40.0.41/32 [115/30] via 10.40.4.44, 4w1d, Ethernet0/0
                       [115/30] via 10.40.3.42, 4w1d, Ethernet0/2
i L2     10.40.0.42/32 [115/20] via 10.40.3.42, 4w1d, Ethernet0/2
C        10.40.0.43/32 is directly connected, Loopback1
i L2     10.40.0.44/32 [115/20] via 10.40.4.44, 4w1d, Ethernet0/0
i L2     10.40.1.0/24 [115/20] via 10.40.4.44, 4w1d, Ethernet0/0
i L2     10.40.2.0/24 [115/20] via 10.40.3.42, 4w1d, Ethernet0/2
C        10.40.3.0/24 is directly connected, Ethernet0/2
L        10.40.3.43/32 is directly connected, Ethernet0/2
C        10.40.4.0/24 is directly connected, Ethernet0/0
L        10.40.4.43/32 is directly connected, Ethernet0/0
B        10.50.52.0/25 [200/0] via 172.16.6.52, 6d19h
B        10.50.52.128/25 [20/0] via 172.16.7.52, 6d20h
      172.16.0.0/16 is variably subnetted, 12 subnets, 2 masks
B        172.16.1.0/24 [200/0] via 172.16.5.21, 3w6d
B        172.16.2.0/24 [200/0] via 172.16.4.31, 3w6d
B        172.16.3.0/24 [200/0] via 172.16.5.21, 3w6d
B        172.16.4.0/24 [200/0] via 10.40.0.44, 3w6d
B        172.16.5.0/24 [200/0] via 10.40.0.41, 4w1d
B        172.16.6.0/24 [200/0] via 10.40.0.44, 3w6d
C        172.16.7.0/24 is directly connected, Ethernet0/3
L        172.16.7.43/32 is directly connected, Ethernet0/3
C        172.16.8.0/24 is directly connected, Ethernet0/1
L        172.16.8.43/32 is directly connected, Ethernet0/1
B        172.16.9.0/24 [200/0] via 10.40.0.42, 4w1d
B        172.16.10.0/24 [200/0] via 10.40.0.42, 4w1d
R43#
R43#
R43#
R43#
R43#show ip bgp
BGP table version is 823, local router ID is 10.40.0.43
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter,
              x best-external, a additional-path, c RIB-compressed,
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 *>i 10.10.13.0/24    172.16.5.21              0    100      0 101 1001 i
 *>i 10.10.14.0/24    172.16.4.31              0    100      0 301 1001 i
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
 *>i 10.50.52.0/25    172.16.6.52              0    100      0 2042 i
 *>  10.50.52.128/25  172.16.7.52              0             0 2042 i
 *>i 172.16.1.0/24    172.16.5.21              0    100      0 101 i
 *>i 172.16.2.0/24    172.16.4.31              0    100      0 301 i
 *>i 172.16.3.0/24    172.16.5.21              0    100      0 101 i
 *>i 172.16.4.0/24    10.40.0.44               0    100      0 i
 *>i 172.16.5.0/24    10.40.0.41               0    100      0 i
 *>i 172.16.6.0/24    10.40.0.44               0    100      0 i
 *>  172.16.7.0/24    0.0.0.0                  0         32768 i
 *>  172.16.8.0/24    0.0.0.0                  0         32768 i
 *>i 172.16.9.0/24    10.40.0.42               0    100      0 i
 *>i 172.16.10.0/24   10.40.0.42               0    100      0 i
R43#
R43#
R43#
R43#
R43#show ip bgp summary
BGP router identifier 10.40.0.43, local AS number 520
BGP table version is 823, main routing table version 823
24 network entries using 3360 bytes of memory
26 path entries using 2080 bytes of memory
14/14 BGP path/bestpath attribute entries using 2016 bytes of memory
2 BGP rrinfo entries using 48 bytes of memory
5 BGP AS-PATH entries using 120 bytes of memory
0 BGP route-map cache entries using 0 bytes of memory
0 BGP filter-list cache entries using 0 bytes of memory
BGP using 7624 total bytes of memory
BGP activity 118/94 prefixes, 300/274 paths, scan interval 60 secs

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
10.40.0.41      4          520   46205   46053      823    0    0 4w1d           18
172.16.7.52     4         2042   10839   10863      823    0    0 6d20h           1


```
</details>

### R52

<details>
  <summary>Конфигурация</summary>

```

R52#terminal length 0
R52#sh run
Building configuration...

Current configuration : 4523 bytes
!
! Last configuration change at 18:54:12 UTC Sun Feb 16 2025
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
interface Loopback2
 description NAT ISP 1
 ip address 10.50.52.126 255.255.255.128
 ip nat enable
!
interface Loopback3
 description NAT ISP 2
 ip address 10.50.52.254 255.255.255.128
 ip nat enable
!
interface Tunnel0
 description to MSK ISP1
 ip address 192.168.1.2 255.255.255.0
 tunnel source 10.50.52.126
 tunnel destination 10.10.14.254
 tunnel key 1
!
interface Tunnel1
 description to MSK ISP1
 ip address 192.168.2.2 255.255.255.0
 tunnel source 10.50.52.254
 tunnel destination 10.10.14.254
 tunnel key 1
!
interface Tunnel2
 description to MSK ISP2
 ip address 192.168.3.2 255.255.255.0
 tunnel source 10.50.52.126
 tunnel destination 10.10.13.254
 tunnel key 2
!
interface Tunnel3
 description to MSK ISP2
 ip address 192.168.4.2 255.255.255.0
 tunnel source 10.50.52.254
 tunnel destination 10.10.13.254
 tunnel key 2
!
interface Ethernet0/0
 ip address 10.50.4.52 255.255.255.0
 ip nat enable
!
interface Ethernet0/1
 ip address 10.50.3.52 255.255.255.0
 ip nat enable
!
interface Ethernet0/2
 ip address 172.16.6.52 255.255.255.0
 ip nat enable
!
interface Ethernet0/3
 ip address 172.16.7.52 255.255.255.0
 ip nat enable
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
 network 10.50.52.0 mask 255.255.255.128
 network 10.50.52.128 mask 255.255.255.128
 timers bgp 3 9
 redistribute eigrp 1
 neighbor 172.16.6.44 remote-as 520
 neighbor 172.16.6.44 prefix-list EBGP-ISP1-FILTER out
 neighbor 172.16.7.43 remote-as 520
 neighbor 172.16.7.43 prefix-list EBGP-ISP2-FILTER out
 neighbor 192.168.1.1 remote-as 1001
 neighbor 192.168.1.1 ebgp-multihop 255
 neighbor 192.168.1.1 update-source Tunnel0
 neighbor 192.168.1.1 prefix-list EBGP-GRE-FILTER out
 neighbor 192.168.1.1 route-map EBGP-GRE-LP in
 neighbor 192.168.1.1 filter-list 1 out
 neighbor 192.168.2.1 remote-as 1001
 neighbor 192.168.2.1 ebgp-multihop 255
 neighbor 192.168.2.1 update-source Tunnel1
 neighbor 192.168.2.1 prefix-list EBGP-GRE-FILTER out
 neighbor 192.168.2.1 filter-list 1 out
 neighbor 192.168.3.1 remote-as 1001
 neighbor 192.168.3.1 ebgp-multihop 255
 neighbor 192.168.3.1 update-source Tunnel2
 neighbor 192.168.3.1 prefix-list EBGP-GRE-FILTER out
 neighbor 192.168.3.1 filter-list 1 out
 neighbor 192.168.4.1 remote-as 1001
 neighbor 192.168.4.1 ebgp-multihop 255
 neighbor 192.168.4.1 update-source Tunnel3
 neighbor 192.168.4.1 prefix-list EBGP-GRE-FILTER out
 neighbor 192.168.4.1 filter-list 1 out
 maximum-paths 2
!
ip forward-protocol nd
!
ip as-path access-list 1 permit ^$
!
no ip http server
no ip http secure-server
ip nat pool PUBLIC_POOL_G0/2 10.50.52.1 10.50.52.5 netmask 255.255.255.128
ip nat pool PUBLIC_POOL_G0/3 10.50.52.129 10.50.52.133 netmask 255.255.255.128
ip nat source route-map NAT_FOR_G0/2 pool PUBLIC_POOL_G0/2 overload
ip nat source route-map NAT_FOR_G0/3 pool PUBLIC_POOL_G0/3 overload
!
!
ip prefix-list ALL seq 5 permit 0.0.0.0/0 le 32
!
ip prefix-list EBGP-GRE-FILTER seq 10 deny 10.50.52.0/25
ip prefix-list EBGP-GRE-FILTER seq 20 deny 10.50.52.128/25
ip prefix-list EBGP-GRE-FILTER seq 25 deny 172.16.0.0/16 ge 24
ip prefix-list EBGP-GRE-FILTER seq 30 permit 0.0.0.0/0 le 32
!
ip prefix-list EBGP-ISP1-FILTER seq 10 permit 10.50.52.0/25
!
ip prefix-list EBGP-ISP2-FILTER seq 10 permit 10.50.52.128/25
!
route-map EBGP-GRE-LP permit 10
 match ip address prefix-list ALL
 set local-preference 150
!
route-map NAT_FOR_G0/2 permit 10
 match ip address 2
 match interface Ethernet0/2
!
route-map NAT_FOR_G0/3 permit 10
 match ip address 2
 match interface Ethernet0/3
!
!
access-list 2 permit 10.50.0.0 0.0.255.255
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
Loopback2                  10.50.52.126    YES manual up                    up
Loopback3                  10.50.52.254    YES manual up                    up
NVI0                       10.50.4.52      YES unset  up                    up
Tunnel0                    192.168.1.2     YES manual up                    up
Tunnel1                    192.168.2.2     YES manual up                    up
Tunnel2                    192.168.3.2     YES manual up                    up
Tunnel3                    192.168.4.2     YES manual up                    up
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

      10.0.0.0/8 is variably subnetted, 49 subnets, 4 masks
B        10.10.0.11/32 [20/11] via 192.168.1.1, 6d19h
B        10.10.0.12/32 [20/11] via 192.168.1.1, 6d19h
B        10.10.0.13/32 [20/21] via 192.168.1.1, 6d19h
B        10.10.0.14/32 [20/0] via 192.168.1.1, 6d19h
B        10.10.0.15/32 [20/31] via 192.168.1.1, 6d19h
B        10.10.0.16/32 [20/11] via 192.168.1.1, 6d19h
B        10.10.1.0/24 [20/20] via 192.168.1.1, 6d19h
B        10.10.2.0/24 [20/20] via 192.168.1.1, 6d19h
B        10.10.3.0/24 [20/20] via 192.168.1.1, 6d19h
B        10.10.4.0/24 [20/30] via 192.168.1.1, 6d19h
B        10.10.5.0/24 [20/0] via 192.168.1.1, 6d19h
B        10.10.6.0/24 [20/0] via 192.168.1.1, 6d19h
B        10.10.7.0/24 [20/20] via 192.168.1.1, 6d19h
B        10.10.8.0/24 [20/0] via 192.168.1.1, 6d19h
B        10.10.9.0/24 [20/20] via 192.168.1.1, 6d19h
B        10.10.13.0/24 [20/0] via 172.16.7.43, 6d19h
                       [20/0] via 172.16.6.44, 6d19h
B        10.10.14.0/24 [20/0] via 172.16.7.43, 6d19h
                       [20/0] via 172.16.6.44, 6d19h
B        10.10.255.0/24 [20/20] via 192.168.1.1, 6d19h
B        10.20.0.0/16 [20/0] via 172.16.7.43, 6d19h
                      [20/0] via 172.16.6.44, 6d19h
B        10.30.0.0/16 [20/0] via 172.16.7.43, 6d19h
                      [20/0] via 172.16.6.44, 6d19h
B        10.40.0.41/32 [20/20] via 172.16.6.44, 6d19h
B        10.40.0.42/32 [20/20] via 172.16.7.43, 6d20h
B        10.40.0.43/32 [20/0] via 172.16.7.43, 6d20h
B        10.40.0.44/32 [20/20] via 172.16.7.43, 6d20h
B        10.40.1.0/24 [20/20] via 172.16.7.43, 6d20h
B        10.40.2.0/24 [20/20] via 172.16.7.43, 6d19h
                      [20/20] via 172.16.6.44, 6d19h
B        10.40.3.0/24 [20/0] via 172.16.7.43, 6d20h
B        10.40.4.0/24 [20/0] via 172.16.7.43, 6d20h
D        10.50.0.51/32 [90/1024640] via 10.50.3.51, 4w1d, Ethernet0/1
C        10.50.0.52/32 is directly connected, Loopback1
D        10.50.0.53/32 [90/1024640] via 10.50.4.53, 4w1d, Ethernet0/0
D        10.50.0.54/32 [90/1536640] via 10.50.4.53, 4w1d, Ethernet0/0
D        10.50.1.0/24 [90/1536000] via 10.50.4.53, 4w1d, Ethernet0/0
                      [90/1536000] via 10.50.3.51, 4w1d, Ethernet0/1
D        10.50.2.0/24 [90/1536000] via 10.50.4.53, 4w1d, Ethernet0/0
                      [90/1536000] via 10.50.3.51, 4w1d, Ethernet0/1
C        10.50.3.0/24 is directly connected, Ethernet0/1
L        10.50.3.52/32 is directly connected, Ethernet0/1
C        10.50.4.0/24 is directly connected, Ethernet0/0
L        10.50.4.52/32 is directly connected, Ethernet0/0
D        10.50.5.0/24 [90/1536000] via 10.50.4.53, 4w1d, Ethernet0/0
D        10.50.6.0/24 [90/1536000] via 10.50.4.53, 4w1d, Ethernet0/0
                      [90/1536000] via 10.50.3.51, 4w1d, Ethernet0/1
C        10.50.52.0/25 is directly connected, Loopback2
L        10.50.52.126/32 is directly connected, Loopback2
C        10.50.52.128/25 is directly connected, Loopback3
L        10.50.52.254/32 is directly connected, Loopback3
D        10.50.255.0/24 [90/1536000] via 10.50.4.53, 4w1d, Ethernet0/0
                        [90/1536000] via 10.50.3.51, 4w1d, Ethernet0/1
B        10.60.0.61/32 [20/0] via 192.168.1.1, 22:05:38
B        10.70.0.71/32 [20/0] via 192.168.1.1, 00:13:23
B        10.70.1.0/24 [20/0] via 192.168.1.1, 00:13:23
B        10.70.255.0/24 [20/0] via 192.168.1.1, 00:13:23
      172.16.0.0/16 is variably subnetted, 12 subnets, 2 masks
B        172.16.1.0/24 [20/0] via 172.16.7.43, 6d19h
                       [20/0] via 172.16.6.44, 6d19h
B        172.16.2.0/24 [20/0] via 172.16.7.43, 6d19h
                       [20/0] via 172.16.6.44, 6d19h
B        172.16.3.0/24 [20/0] via 172.16.7.43, 6d20h
B        172.16.4.0/24 [20/0] via 172.16.7.43, 6d19h
                       [20/0] via 172.16.6.44, 6d19h
B        172.16.5.0/24 [20/0] via 172.16.7.43, 6d19h
                       [20/0] via 172.16.6.44, 6d19h
C        172.16.6.0/24 is directly connected, Ethernet0/2
L        172.16.6.52/32 is directly connected, Ethernet0/2
C        172.16.7.0/24 is directly connected, Ethernet0/3
L        172.16.7.52/32 is directly connected, Ethernet0/3
B        172.16.8.0/24 [20/0] via 172.16.7.43, 6d19h
                       [20/0] via 172.16.6.44, 6d19h
B        172.16.9.0/24 [20/0] via 172.16.7.43, 6d19h
                       [20/0] via 172.16.6.44, 6d19h
B        172.16.10.0/24 [20/0] via 172.16.7.43, 6d19h
                        [20/0] via 172.16.6.44, 6d19h
      192.168.1.0/24 is variably subnetted, 2 subnets, 2 masks
C        192.168.1.0/24 is directly connected, Tunnel0
L        192.168.1.2/32 is directly connected, Tunnel0
      192.168.2.0/24 is variably subnetted, 2 subnets, 2 masks
C        192.168.2.0/24 is directly connected, Tunnel1
L        192.168.2.2/32 is directly connected, Tunnel1
      192.168.3.0/24 is variably subnetted, 2 subnets, 2 masks
C        192.168.3.0/24 is directly connected, Tunnel2
L        192.168.3.2/32 is directly connected, Tunnel2
      192.168.4.0/24 is variably subnetted, 2 subnets, 2 masks
C        192.168.4.0/24 is directly connected, Tunnel3
L        192.168.4.2/32 is directly connected, Tunnel3
R52#
R52#
R52#
R52#
R52#show ip bgp
BGP table version is 421, local router ID is 10.50.52.254
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter,
              x best-external, a additional-path, c RIB-compressed,
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 *   10.10.0.11/32    192.168.3.1             11             0 1001 ?
 *>                   192.168.1.1             11    150      0 1001 ?
 *                    192.168.2.1             11             0 1001 ?
 *                    192.168.4.1             11             0 1001 ?
 *   10.10.0.12/32    192.168.3.1             11             0 1001 ?
 *>                   192.168.1.1             11    150      0 1001 ?
 *                    192.168.2.1             11             0 1001 ?
 *                    192.168.4.1             11             0 1001 ?
 *   10.10.0.13/32    192.168.3.1              0             0 1001 ?
 *>                   192.168.1.1             21    150      0 1001 ?
 *                    192.168.2.1             21             0 1001 ?
 *                    192.168.4.1              0             0 1001 ?
 *   10.10.0.14/32    192.168.3.1             12             0 1001 ?
 *>                   192.168.1.1              0    150      0 1001 ?
 *                    192.168.2.1              0             0 1001 ?
 *                    192.168.4.1             12             0 1001 ?
 *   10.10.0.15/32    192.168.3.1             11             0 1001 ?
 *>                   192.168.1.1             31    150      0 1001 ?
 *                    192.168.2.1             31             0 1001 ?
 *                    192.168.4.1             11             0 1001 ?
 *   10.10.0.16/32    192.168.3.1             22             0 1001 ?
 *>                   192.168.1.1             11    150      0 1001 ?
 *                    192.168.2.1             11             0 1001 ?
 *                    192.168.4.1             22             0 1001 ?
 *   10.10.1.0/24     192.168.3.1             20             0 1001 ?
 *>                   192.168.1.1             20    150      0 1001 ?
 *                    192.168.2.1             20             0 1001 ?
 *                    192.168.4.1             20             0 1001 ?
 *   10.10.2.0/24     192.168.3.1             20             0 1001 ?
 *>                   192.168.1.1             20    150      0 1001 ?
 *                    192.168.2.1             20             0 1001 ?
 *                    192.168.4.1             20             0 1001 ?
 *   10.10.3.0/24     192.168.3.1              0             0 1001 ?
 *>                   192.168.1.1             20    150      0 1001 ?
 *                    192.168.2.1             20             0 1001 ?
 *                    192.168.4.1              0             0 1001 ?
 *   10.10.4.0/24     192.168.3.1              0             0 1001 ?
 *>                   192.168.1.1             30    150      0 1001 ?
 *                    192.168.2.1             30             0 1001 ?
 *                    192.168.4.1              0             0 1001 ?
 *   10.10.5.0/24     192.168.3.1             11             0 1001 ?
 *>                   192.168.1.1              0    150      0 1001 ?
 *                    192.168.2.1              0             0 1001 ?
 *                    192.168.4.1             11             0 1001 ?
 *   10.10.6.0/24     192.168.3.1             21             0 1001 ?
 *>                   192.168.1.1              0    150      0 1001 ?
 *                    192.168.2.1              0             0 1001 ?
 *                    192.168.4.1             21             0 1001 ?
 *   10.10.7.0/24     192.168.3.1              0             0 1001 ?
 *>                   192.168.1.1             20    150      0 1001 ?
 *                    192.168.2.1             20             0 1001 ?
 *                    192.168.4.1              0             0 1001 ?
 *   10.10.8.0/24     192.168.3.1             11             0 1001 ?
 *>                   192.168.1.1              0    150      0 1001 ?
 *                    192.168.2.1              0             0 1001 ?
 *                    192.168.4.1             11             0 1001 ?
 *   10.10.9.0/24     192.168.3.1             20             0 1001 ?
 *>                   192.168.1.1             20    150      0 1001 ?
 *                    192.168.2.1             20             0 1001 ?
 *                    192.168.4.1             20             0 1001 ?
 *m  10.10.13.0/24    172.16.6.44                            0 520 101 1001 i
 *>                   172.16.7.43                            0 520 101 1001 i
 *m  10.10.14.0/24    172.16.6.44                            0 520 301 1001 i
 *>                   172.16.7.43                            0 520 301 1001 i
 *   10.10.255.0/24   192.168.3.1             20             0 1001 ?
 *>                   192.168.1.1             20    150      0 1001 ?
 *                    192.168.2.1             20             0 1001 ?
 *                    192.168.4.1             20             0 1001 ?
 *m  10.20.0.0/16     172.16.6.44                            0 520 101 ?
 *>                   172.16.7.43                            0 520 101 ?
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
 *>  10.50.52.0/25    0.0.0.0                  0         32768 i
 *>  10.50.52.128/25  0.0.0.0                  0         32768 i
 *>  10.50.255.0/24   10.50.3.51         1536000         32768 ?
 *   10.60.0.61/32    192.168.2.1                            0 1001 i
 *>                   192.168.1.1                   150      0 1001 i
 *                    192.168.4.1                            0 1001 i
 *                    192.168.3.1                            0 1001 i
 *   10.70.0.71/32    192.168.2.1                            0 1001 i
 *>                   192.168.1.1                   150      0 1001 i
 *                    192.168.4.1                            0 1001 i
 *                    192.168.3.1                            0 1001 i
 *   10.70.1.0/24     192.168.2.1                            0 1001 i
 *>                   192.168.1.1                   150      0 1001 i
 *                    192.168.4.1                            0 1001 i
 *                    192.168.3.1                            0 1001 i
 *   10.70.255.0/24   192.168.2.1                            0 1001 i
 *>                   192.168.1.1                   150      0 1001 i
 *                    192.168.4.1                            0 1001 i
 *                    192.168.3.1                            0 1001 i
 *m  172.16.1.0/24    172.16.6.44                            0 520 101 i
 *>                   172.16.7.43                            0 520 101 i
 *m  172.16.2.0/24    172.16.6.44                            0 520 301 i
 *>                   172.16.7.43                            0 520 301 i
 *   172.16.3.0/24    172.16.6.44                            0 520 301 i
 *>                   172.16.7.43                            0 520 101 i
 *m  172.16.4.0/24    172.16.6.44              0             0 520 i
 *>                   172.16.7.43                            0 520 i
 *m  172.16.5.0/24    172.16.6.44                            0 520 i
 *>                   172.16.7.43                            0 520 i
 rm  172.16.6.0/24    172.16.6.44              0             0 520 i
 r>                   172.16.7.43                            0 520 i
 rm  172.16.7.0/24    172.16.6.44                            0 520 i
 r>                   172.16.7.43              0             0 520 i
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
BGP router identifier 10.50.52.254, local AS number 2042
BGP table version is 421, main routing table version 421
55 network entries using 7700 bytes of memory
134 path entries using 10720 bytes of memory
14 multipath network entries and 28 multipath paths
32/22 BGP path/bestpath attribute entries using 4608 bytes of memory
6 BGP AS-PATH entries using 144 bytes of memory
0 BGP route-map cache entries using 0 bytes of memory
0 BGP filter-list cache entries using 0 bytes of memory
BGP using 23172 total bytes of memory
BGP activity 314/259 prefixes, 1138/1004 paths, scan interval 60 secs

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
172.16.6.44     4          520  191559  191547      421    0    0 6d19h          19
172.16.7.43     4          520   10874   10850      421    0    0 6d20h          22
192.168.1.1     4         1001  191553  191544      421    0    0 6d19h          20
192.168.2.1     4         1001   10796   10805      421    0    0 6d19h          20
192.168.3.1     4         1001  191555  191544      421    0    0 6d19h          20
192.168.4.1     4         1001   10847   10824      

```
</details>


### R61

<details>
  <summary>Конфигурация</summary>

```

R61#terminal length 0
R61#sh run
Building configuration...

Current configuration : 2255 bytes
!
! Last configuration change at 16:13:55 UTC Sat Feb 22 2025
!
version 15.4
service timestamps debug datetime msec
service timestamps log datetime msec
no service password-encryption
!
hostname R61
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
 ip address 10.60.0.61 255.255.255.255
!
interface Tunnel10
 description MGRE to ISP (DMVPN)
 ip address 10.100.0.2 255.255.255.0
 no ip redirects
 ip nhrp authentication CISCO
 ip nhrp network-id 100
 ip nhrp holdtime 90
 ip nhrp nhs 10.100.0.1 nbma 10.10.14.254
 tunnel source Ethernet0/0
 tunnel mode gre multipoint
 tunnel key 10
 tunnel route-via Ethernet0/0 preferred
!
interface Tunnel20
 description MGRE to ISP (DMVPN)
 ip address 10.101.0.2 255.255.255.0
 no ip redirects
 ip nhrp authentication CISCO
 ip nhrp network-id 101
 ip nhrp holdtime 90
 ip nhrp nhs 10.101.0.1 nbma 10.10.13.254
 tunnel source Ethernet0/0
 tunnel mode gre multipoint
 tunnel key 11
 tunnel route-via Ethernet0/0 preferred
!
interface Ethernet0/0
 ip address 172.16.9.61 255.255.255.0
!
interface Ethernet0/1
 no ip address
 shutdown
!
interface Ethernet0/2
 no ip address
 shutdown
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
router bgp 1001
 bgp log-neighbor-changes
 network 10.60.0.61 mask 255.255.255.255
 timers bgp 3 9
 neighbor 10.100.0.1 remote-as 1001
 neighbor 10.100.0.1 route-map IBGP-DMVPN-CLOUD1-LP in
 neighbor 10.101.0.1 remote-as 1001
!
ip forward-protocol nd
!
!
no ip http server
no ip http secure-server
ip route 0.0.0.0 0.0.0.0 172.16.9.42
!
!
ip prefix-list ALL seq 5 permit 0.0.0.0/0 le 32
!
route-map IBGP-DMVPN-CLOUD1-LP permit 10
 match ip address prefix-list ALL
 set local-preference 150
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

R61#
R61#
R61#
R61#show ip interface brief
Interface                  IP-Address      OK? Method Status                Protocol
Ethernet0/0                172.16.9.61     YES NVRAM  up                    up
Ethernet0/1                unassigned      YES NVRAM  administratively down down
Ethernet0/2                unassigned      YES NVRAM  administratively down down
Ethernet0/3                unassigned      YES NVRAM  administratively down down
Ethernet1/0                unassigned      YES NVRAM  administratively down down
Ethernet1/1                unassigned      YES NVRAM  administratively down down
Ethernet1/2                unassigned      YES NVRAM  administratively down down
Ethernet1/3                unassigned      YES NVRAM  administratively down down
Loopback1                  10.60.0.61      YES NVRAM  up                    up
Tunnel10                   10.100.0.2      YES manual up                    up
Tunnel20                   10.101.0.2      YES manual up                    up
R61#
R61#
R61#
R61#
R61#show ip route
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
       a - application route
       + - replicated route, % - next hop override

Gateway of last resort is 172.16.9.42 to network 0.0.0.0

S*    0.0.0.0/0 [1/0] via 172.16.9.42
      10.0.0.0/8 is variably subnetted, 35 subnets, 2 masks
B        10.10.0.11/32 [200/11] via 10.100.0.1, 22:12:21
B        10.10.0.12/32 [200/11] via 10.100.0.1, 22:12:21
B        10.10.0.13/32 [200/21] via 10.100.0.1, 22:12:21
B        10.10.0.14/32 [200/0] via 10.100.0.1, 22:12:21
B        10.10.0.15/32 [200/31] via 10.100.0.1, 22:12:21
B        10.10.0.16/32 [200/11] via 10.100.0.1, 22:12:21
B        10.10.1.0/24 [200/20] via 10.100.0.1, 22:12:21
B        10.10.2.0/24 [200/20] via 10.100.0.1, 22:12:21
B        10.10.3.0/24 [200/20] via 10.100.0.1, 22:12:21
B        10.10.4.0/24 [200/30] via 10.100.0.1, 22:12:21
B        10.10.5.0/24 [200/0] via 10.100.0.1, 22:12:21
B        10.10.6.0/24 [200/0] via 10.100.0.1, 22:12:21
B        10.10.7.0/24 [200/20] via 10.100.0.1, 22:12:21
B        10.10.8.0/24 [200/0] via 10.100.0.1, 22:12:21
B        10.10.9.0/24 [200/20] via 10.100.0.1, 22:12:21
B        10.10.255.0/24 [200/20] via 10.100.0.1, 22:12:21
B        10.50.0.51/32 [200/1024640] via 10.101.0.1, 22:12:21
B        10.50.0.52/32 [200/0] via 10.101.0.1, 22:12:21
B        10.50.0.53/32 [200/1024640] via 10.101.0.1, 22:12:21
B        10.50.0.54/32 [200/1536640] via 10.101.0.1, 22:12:21
B        10.50.1.0/24 [200/1536000] via 10.101.0.1, 22:12:21
B        10.50.2.0/24 [200/1536000] via 10.101.0.1, 22:12:21
B        10.50.3.0/24 [200/0] via 10.101.0.1, 22:12:21
B        10.50.4.0/24 [200/0] via 10.101.0.1, 22:12:21
B        10.50.5.0/24 [200/1536000] via 10.101.0.1, 22:12:21
B        10.50.6.0/24 [200/1536000] via 10.101.0.1, 22:12:21
B        10.50.255.0/24 [200/1536000] via 10.101.0.1, 22:12:21
C        10.60.0.61/32 is directly connected, Loopback1
B        10.70.0.71/32 [200/0] via 10.100.0.3, 00:19:38
B        10.70.1.0/24 [200/0] via 10.100.0.3, 00:19:38
B        10.70.255.0/24 [200/0] via 10.100.0.3, 00:19:38
C        10.100.0.0/24 is directly connected, Tunnel10
L        10.100.0.2/32 is directly connected, Tunnel10
C        10.101.0.0/24 is directly connected, Tunnel20
L        10.101.0.2/32 is directly connected, Tunnel20
      172.16.0.0/16 is variably subnetted, 2 subnets, 2 masks
C        172.16.9.0/24 is directly connected, Ethernet0/0
L        172.16.9.61/32 is directly connected, Ethernet0/0
R61#
R61#
R61#
R61#
R61#show ip bgp
BGP table version is 46, local router ID is 10.60.0.61
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter,
              x best-external, a additional-path, c RIB-compressed,
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 r>i 0.0.0.0          10.100.0.1               0    150      0 301 i
 r i                  10.101.0.1               0    100      0 101 i
 *>i 10.10.0.11/32    10.100.0.1              11    150      0 ?
 * i                  10.101.0.1              11    100      0 ?
 *>i 10.10.0.12/32    10.100.0.1              11    150      0 ?
 * i                  10.101.0.1              11    100      0 ?
 *>i 10.10.0.13/32    10.100.0.1              21    150      0 ?
 * i                  10.101.0.1               0    100      0 ?
 *>i 10.10.0.14/32    10.100.0.1               0    150      0 ?
 * i                  10.101.0.1              12    100      0 ?
 *>i 10.10.0.15/32    10.100.0.1              31    150      0 ?
 * i                  10.101.0.1              11    100      0 ?
 *>i 10.10.0.16/32    10.100.0.1              11    150      0 ?
 * i                  10.101.0.1              22    100      0 ?
 *>i 10.10.1.0/24     10.100.0.1              20    150      0 ?
 * i                  10.101.0.1              20    100      0 ?
 *>i 10.10.2.0/24     10.100.0.1              20    150      0 ?
 * i                  10.101.0.1              20    100      0 ?
 *>i 10.10.3.0/24     10.100.0.1              20    150      0 ?
 * i                  10.101.0.1               0    100      0 ?
 *>i 10.10.4.0/24     10.100.0.1              30    150      0 ?
 * i                  10.101.0.1               0    100      0 ?
 *>i 10.10.5.0/24     10.100.0.1               0    150      0 ?
 * i                  10.101.0.1              11    100      0 ?
 *>i 10.10.6.0/24     10.100.0.1               0    150      0 ?
 * i                  10.101.0.1              21    100      0 ?
 *>i 10.10.7.0/24     10.100.0.1              20    150      0 ?
 * i                  10.101.0.1               0    100      0 ?
 *>i 10.10.8.0/24     10.100.0.1               0    150      0 ?
 * i                  10.101.0.1              11    100      0 ?
 *>i 10.10.9.0/24     10.100.0.1              20    150      0 ?
 * i                  10.101.0.1              20    100      0 ?
 *>i 10.10.255.0/24   10.100.0.1              20    150      0 ?
 * i                  10.101.0.1              20    100      0 ?
 * i 10.50.0.51/32    10.100.0.1         1024640    150      0 2042 ?
 *>i                  10.101.0.1         1024640    150      0 2042 ?
 * i 10.50.0.52/32    10.100.0.1               0    150      0 2042 ?
 *>i                  10.101.0.1               0    150      0 2042 ?
 * i 10.50.0.53/32    10.100.0.1         1024640    150      0 2042 ?
 *>i                  10.101.0.1         1024640    150      0 2042 ?
 * i 10.50.0.54/32    10.100.0.1         1536640    150      0 2042 ?
 *>i                  10.101.0.1         1536640    150      0 2042 ?
 * i 10.50.1.0/24     10.100.0.1         1536000    150      0 2042 ?
 *>i                  10.101.0.1         1536000    150      0 2042 ?
 * i 10.50.2.0/24     10.100.0.1         1536000    150      0 2042 ?
 *>i                  10.101.0.1         1536000    150      0 2042 ?
 * i 10.50.3.0/24     10.100.0.1               0    150      0 2042 ?
 *>i                  10.101.0.1               0    150      0 2042 ?
 * i 10.50.4.0/24     10.100.0.1               0    150      0 2042 ?
 *>i                  10.101.0.1               0    150      0 2042 ?
 * i 10.50.5.0/24     10.100.0.1         1536000    150      0 2042 ?
 *>i                  10.101.0.1         1536000    150      0 2042 ?
 * i 10.50.6.0/24     10.100.0.1         1536000    150      0 2042 ?
 *>i                  10.101.0.1         1536000    150      0 2042 ?
 * i 10.50.255.0/24   10.100.0.1         1536000    150      0 2042 ?
 *>i                  10.101.0.1         1536000    150      0 2042 ?
 *>  10.60.0.61/32    0.0.0.0                  0         32768 i
 *>i 10.70.0.71/32    10.100.0.3               0    150      0 i
 * i                  10.101.0.3               0    100      0 i
 *>i 10.70.1.0/24     10.100.0.3               0    150      0 i
 * i                  10.101.0.3               0    100      0 i
 *>i 10.70.255.0/24   10.100.0.3               0    150      0 i
 * i                  10.101.0.3               0    100      0 i
R61#
R61#
R61#
R61#
R61#show ip bgp summary
BGP router identifier 10.60.0.61, local AS number 1001
BGP table version is 46, main routing table version 46
32 network entries using 4480 bytes of memory
63 path entries using 5040 bytes of memory
24/13 BGP path/bestpath attribute entries using 3456 bytes of memory
2 BGP rrinfo entries using 48 bytes of memory
3 BGP AS-PATH entries using 72 bytes of memory
0 BGP route-map cache entries using 0 bytes of memory
0 BGP filter-list cache entries using 0 bytes of memory
BGP using 13096 total bytes of memory
BGP activity 96/64 prefixes, 167/104 paths, scan interval 60 secs

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
10.100.0.1      4         1001   26056   26027       46    0    0 22:12:22       31
10.101.0.1      4         1001   26065   26027       46    0    0 22:12:22       31
R61#
R61#
R61#
R61#
R61#show ip nhrp traffic
Tunnel10: Max-send limit:100Pkts/10Sec, Usage:0%
   Sent: Total 21793
         9568 Resolution Request  9472 Resolution Reply  2749 Registration Request
         0 Registration Reply  4 Purge Request  0 Purge Reply
         0 Error Indication  0 Traffic Indication  0 Redirect Suppress
   Rcvd: Total 21797
         9576 Resolution Request  9468 Resolution Reply  0 Registration Request
         2749 Registration Reply  0 Purge Request  3 Purge Reply
         1 Error Indication  0 Traffic Indication  0 Redirect Suppress
Tunnel20: Max-send limit:100Pkts/10Sec, Usage:0%
   Sent: Total 21921
         9641 Resolution Request  9544 Resolution Reply  2734 Registration Request
         0 Registration Reply  2 Purge Request  0 Purge Reply
         0 Error Indication  0 Traffic Indication  0 Redirect Suppress
   Rcvd: Total 21913
         9635 Resolution Request  9543 Resolution Reply  0 Registration Request
         2734 Registration Reply  0 Purge Request  1 Purge Reply
         0 Error Indication  0 Traffic Indication  0 Redirect Suppress
R61#
R61#
R61#
R61#
R61#show ip nhrp detail
10.100.0.1/32 via 10.100.0.1
   Tunnel10 created 23:01:11, never expire
   Type: static, Flags: used
   NBMA address: 10.10.14.254
10.101.0.1/32 via 10.101.0.1
   Tunnel20 created 22:46:57, never expire
   Type: static, Flags: used
   NBMA address: 10.10.13.254
R61#
R61#
R61#
R61#
R61#show dmvpn
Legend: Attrb --> S - Static, D - Dynamic, I - Incomplete
        N - NATed, L - Local, X - No Socket
        # Ent --> Number of NHRP entries with same NBMA peer
        NHS Status: E --> Expecting Replies, R --> Responding, W --> Waiting
        UpDn Time --> Up or Down Time for a Tunnel
==========================================================================

Interface: Tunnel10, IPv4 NHRP Details
Type:Spoke, NHRP Peers:1,

 # Ent  Peer NBMA Addr Peer Tunnel Add State  UpDn Tm Attrb
 ----- --------------- --------------- ----- -------- -----
     1 10.10.14.254         10.100.0.1    UP 22:54:22     S

Interface: Tunnel20, IPv4 NHRP Details
Type:Spoke, NHRP Peers:1,

 # Ent  Peer NBMA Addr Peer Tunnel Add State  UpDn Tm Attrb
 ----- --------------- --------------- ----- -------- -----
     1 10.10.13.254         10.101.0.1    UP 22:46:57     S


```
</details>


### R71

<details>
  <summary>Конфигурация</summary>

```

R71#terminal length 0
R71#sh run
Building configuration...

Current configuration : 3076 bytes
!
! Last configuration change at 14:08:02 UTC Sun Feb 23 2025
!
version 15.4
service timestamps debug datetime msec
service timestamps log datetime msec
no service password-encryption
!
hostname R71
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
track 1 ip sla 1 reachability
 delay down 10 up 10
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
 ip address 10.70.0.71 255.255.255.255
!
interface Tunnel10
 description MGRE to ISP1 (DMVPN)
 ip address 10.100.0.3 255.255.255.0
 no ip redirects
 ip nhrp authentication CISCO
 ip nhrp network-id 100
 ip nhrp holdtime 90
 ip nhrp nhs 10.100.0.1 nbma 10.10.14.254
 tunnel source Ethernet0/0
 tunnel mode gre multipoint
 tunnel key 10
 tunnel route-via Ethernet0/0 preferred
!
interface Tunnel20
 description MGRE to ISP2 (DMVPN)
 ip address 10.101.0.3 255.255.255.0
 no ip redirects
 ip nhrp authentication CISCO
 ip nhrp network-id 101
 ip nhrp holdtime 90
 ip nhrp nhs 10.101.0.1 nbma 10.10.13.254
 tunnel source Ethernet0/1
 tunnel mode gre multipoint
 tunnel key 11
 tunnel route-via Ethernet0/1 preferred
!
interface Ethernet0/0
 ip address 172.16.8.71 255.255.255.0
 ip nat enable
!
interface Ethernet0/1
 ip address 172.16.10.71 255.255.255.0
!
interface Ethernet0/2
 no ip address
!
interface Ethernet0/2.10
 encapsulation dot1Q 10
 ip address 10.70.1.1 255.255.255.0
!
interface Ethernet0/2.1000
 encapsulation dot1Q 1000
 ip address 10.70.255.1 255.255.255.0
 ip nat enable
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
router bgp 1001
 bgp log-neighbor-changes
 network 10.70.0.71 mask 255.255.255.255
 network 10.70.1.0 mask 255.255.255.0
 network 10.70.255.0 mask 255.255.255.0
 timers bgp 3 9
 neighbor 10.100.0.1 remote-as 1001
 neighbor 10.100.0.1 route-map IBGP-DMVPN-CLOUD1-LP in
 neighbor 10.101.0.1 remote-as 1001
!
ip local policy route-map SLA
ip forward-protocol nd
!
!
no ip http server
no ip http secure-server
ip nat source static tcp 10.70.255.71 22 interface Ethernet0/0 22
ip route 0.0.0.0 0.0.0.0 172.16.8.43 track 1
ip route 0.0.0.0 0.0.0.0 172.16.10.42 100
!
ip access-list extended SLA
 permit icmp host 172.16.8.71 host 10.40.0.43
!
!
ip prefix-list ALL seq 5 permit 0.0.0.0/0 le 32
ip sla 1
 icmp-echo 10.40.0.43 source-interface Ethernet0/0
 tos 46
 threshold 1000
 timeout 1000
 frequency 1
ip sla schedule 1 life forever start-time now
!
route-map IBGP-DMVPN-CLOUD1-LP permit 10
 match ip address prefix-list ALL
 set local-preference 150
!
route-map SLA permit 10
 description SLA
 match ip address SLA
 set ip next-hop 172.16.8.43
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

R71#
R71#
R71#
R71#show ip interface brief
Interface                  IP-Address      OK? Method Status                Protocol
Ethernet0/0                172.16.8.71     YES NVRAM  up                    up
Ethernet0/1                172.16.10.71    YES NVRAM  up                    up
Ethernet0/2                unassigned      YES NVRAM  up                    up
Ethernet0/2.10             10.70.1.1       YES NVRAM  up                    up
Ethernet0/2.1000           10.70.255.1     YES NVRAM  up                    up
Ethernet0/3                unassigned      YES NVRAM  administratively down down
Ethernet1/0                unassigned      YES NVRAM  administratively down down
Ethernet1/1                unassigned      YES NVRAM  administratively down down
Ethernet1/2                unassigned      YES NVRAM  administratively down down
Ethernet1/3                unassigned      YES NVRAM  administratively down down
Loopback1                  10.70.0.71      YES NVRAM  up                    up
NVI0                       172.16.8.71     YES unset  up                    up
Tunnel10                   10.100.0.3      YES manual up                    up
Tunnel20                   10.101.0.3      YES manual up                    up
R71#
R71#
R71#
R71#
R71#show ip route
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
       a - application route
       + - replicated route, % - next hop override

Gateway of last resort is 172.16.8.43 to network 0.0.0.0

S*    0.0.0.0/0 [1/0] via 172.16.8.43
      10.0.0.0/8 is variably subnetted, 37 subnets, 2 masks
B        10.10.0.11/32 [200/11] via 10.100.0.1, 00:20:38
B        10.10.0.12/32 [200/11] via 10.100.0.1, 00:20:38
B        10.10.0.13/32 [200/21] via 10.100.0.1, 00:20:38
B        10.10.0.14/32 [200/0] via 10.100.0.1, 00:20:38
B        10.10.0.15/32 [200/31] via 10.100.0.1, 00:20:38
B        10.10.0.16/32 [200/11] via 10.100.0.1, 00:20:38
B        10.10.1.0/24 [200/20] via 10.100.0.1, 00:20:38
B        10.10.2.0/24 [200/20] via 10.100.0.1, 00:20:38
B        10.10.3.0/24 [200/20] via 10.100.0.1, 00:20:38
B        10.10.4.0/24 [200/30] via 10.100.0.1, 00:20:38
B        10.10.5.0/24 [200/0] via 10.100.0.1, 00:20:38
B        10.10.6.0/24 [200/0] via 10.100.0.1, 00:20:38
B        10.10.7.0/24 [200/20] via 10.100.0.1, 00:20:38
B        10.10.8.0/24 [200/0] via 10.100.0.1, 00:20:38
B        10.10.9.0/24 [200/20] via 10.100.0.1, 00:20:38
B        10.10.255.0/24 [200/20] via 10.100.0.1, 00:20:38
B        10.50.0.51/32 [200/1024640] via 10.101.0.1, 22:00:29
B        10.50.0.52/32 [200/0] via 10.101.0.1, 22:00:29
B        10.50.0.53/32 [200/1024640] via 10.101.0.1, 22:00:29
B        10.50.0.54/32 [200/1536640] via 10.101.0.1, 22:00:29
B        10.50.1.0/24 [200/1536000] via 10.101.0.1, 22:00:29
B        10.50.2.0/24 [200/1536000] via 10.101.0.1, 22:00:29
B        10.50.3.0/24 [200/0] via 10.101.0.1, 22:00:29
B        10.50.4.0/24 [200/0] via 10.101.0.1, 22:00:29
B        10.50.5.0/24 [200/1536000] via 10.101.0.1, 22:00:29
B        10.50.6.0/24 [200/1536000] via 10.101.0.1, 22:00:29
B        10.50.255.0/24 [200/1536000] via 10.101.0.1, 22:00:29
B        10.60.0.61/32 [200/0] via 10.100.0.2, 00:20:38
C        10.70.0.71/32 is directly connected, Loopback1
C        10.70.1.0/24 is directly connected, Ethernet0/2.10
L        10.70.1.1/32 is directly connected, Ethernet0/2.10
C        10.70.255.0/24 is directly connected, Ethernet0/2.1000
L        10.70.255.1/32 is directly connected, Ethernet0/2.1000
C        10.100.0.0/24 is directly connected, Tunnel10
L        10.100.0.3/32 is directly connected, Tunnel10
C        10.101.0.0/24 is directly connected, Tunnel20
L        10.101.0.3/32 is directly connected, Tunnel20
      172.16.0.0/16 is variably subnetted, 4 subnets, 2 masks
C        172.16.8.0/24 is directly connected, Ethernet0/0
L        172.16.8.71/32 is directly connected, Ethernet0/0
C        172.16.10.0/24 is directly connected, Ethernet0/1
L        172.16.10.71/32 is directly connected, Ethernet0/1
R71#
R71#
R71#
R71#
R71#show ip bgp
BGP table version is 371, local router ID is 10.70.0.71
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter,
              x best-external, a additional-path, c RIB-compressed,
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 r>i 0.0.0.0          10.100.0.1               0    150      0 301 i
 r i                  10.101.0.1               0    100      0 101 i
 *>i 10.10.0.11/32    10.100.0.1              11    150      0 ?
 * i                  10.101.0.1              11    100      0 ?
 *>i 10.10.0.12/32    10.100.0.1              11    150      0 ?
 * i                  10.101.0.1              11    100      0 ?
 *>i 10.10.0.13/32    10.100.0.1              21    150      0 ?
 * i                  10.101.0.1               0    100      0 ?
 *>i 10.10.0.14/32    10.100.0.1               0    150      0 ?
 * i                  10.101.0.1              12    100      0 ?
 *>i 10.10.0.15/32    10.100.0.1              31    150      0 ?
 * i                  10.101.0.1              11    100      0 ?
 *>i 10.10.0.16/32    10.100.0.1              11    150      0 ?
 * i                  10.101.0.1              22    100      0 ?
 *>i 10.10.1.0/24     10.100.0.1              20    150      0 ?
 * i                  10.101.0.1              20    100      0 ?
 *>i 10.10.2.0/24     10.100.0.1              20    150      0 ?
 * i                  10.101.0.1              20    100      0 ?
 *>i 10.10.3.0/24     10.100.0.1              20    150      0 ?
 * i                  10.101.0.1               0    100      0 ?
 *>i 10.10.4.0/24     10.100.0.1              30    150      0 ?
 * i                  10.101.0.1               0    100      0 ?
 *>i 10.10.5.0/24     10.100.0.1               0    150      0 ?
 * i                  10.101.0.1              11    100      0 ?
 *>i 10.10.6.0/24     10.100.0.1               0    150      0 ?
 * i                  10.101.0.1              21    100      0 ?
 *>i 10.10.7.0/24     10.100.0.1              20    150      0 ?
 * i                  10.101.0.1               0    100      0 ?
 *>i 10.10.8.0/24     10.100.0.1               0    150      0 ?
 * i                  10.101.0.1              11    100      0 ?
 *>i 10.10.9.0/24     10.100.0.1              20    150      0 ?
 * i                  10.101.0.1              20    100      0 ?
 *>i 10.10.255.0/24   10.100.0.1              20    150      0 ?
 * i                  10.101.0.1              20    100      0 ?
 * i 10.50.0.51/32    10.100.0.1         1024640    150      0 2042 ?
 *>i                  10.101.0.1         1024640    150      0 2042 ?
 * i 10.50.0.52/32    10.100.0.1               0    150      0 2042 ?
 *>i                  10.101.0.1               0    150      0 2042 ?
 * i 10.50.0.53/32    10.100.0.1         1024640    150      0 2042 ?
 *>i                  10.101.0.1         1024640    150      0 2042 ?
 * i 10.50.0.54/32    10.100.0.1         1536640    150      0 2042 ?
 *>i                  10.101.0.1         1536640    150      0 2042 ?
 * i 10.50.1.0/24     10.100.0.1         1536000    150      0 2042 ?
 *>i                  10.101.0.1         1536000    150      0 2042 ?
 * i 10.50.2.0/24     10.100.0.1         1536000    150      0 2042 ?
 *>i                  10.101.0.1         1536000    150      0 2042 ?
 * i 10.50.3.0/24     10.100.0.1               0    150      0 2042 ?
 *>i                  10.101.0.1               0    150      0 2042 ?
 * i 10.50.4.0/24     10.100.0.1               0    150      0 2042 ?
 *>i                  10.101.0.1               0    150      0 2042 ?
 * i 10.50.5.0/24     10.100.0.1         1536000    150      0 2042 ?
 *>i                  10.101.0.1         1536000    150      0 2042 ?
 * i 10.50.6.0/24     10.100.0.1         1536000    150      0 2042 ?
 *>i                  10.101.0.1         1536000    150      0 2042 ?
 * i 10.50.255.0/24   10.100.0.1         1536000    150      0 2042 ?
 *>i                  10.101.0.1         1536000    150      0 2042 ?
 *>i 10.60.0.61/32    10.100.0.2               0    150      0 i
 * i                  10.101.0.2               0    100      0 i
 *>  10.70.0.71/32    0.0.0.0                  0         32768 i
 *>  10.70.1.0/24     0.0.0.0                  0         32768 i
 *>  10.70.255.0/24   0.0.0.0                  0         32768 i
R71#
R71#
R71#
R71#
R71#show ip bgp summary
BGP router identifier 10.70.0.71, local AS number 1001
BGP table version is 371, main routing table version 371
32 network entries using 4480 bytes of memory
61 path entries using 4880 bytes of memory
24/13 BGP path/bestpath attribute entries using 3456 bytes of memory
2 BGP rrinfo entries using 48 bytes of memory
3 BGP AS-PATH entries using 72 bytes of memory
0 BGP route-map cache entries using 0 bytes of memory
0 BGP filter-list cache entries using 0 bytes of memory
BGP using 12936 total bytes of memory
BGP activity 67/35 prefixes, 421/360 paths, scan interval 60 secs

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
10.100.0.1      4         1001     422     410      371    0    0 00:20:37       29
10.101.0.1      4         1001   25811   25801      371    0    0 22:00:28       29
R71#
R71#
R71#
R71#
R71#show ip nhrp traffic
Tunnel10: Max-send limit:100Pkts/10Sec, Usage:0%
   Sent: Total 21770
         9576 Resolution Request  9468 Resolution Reply  2723 Registration Request
         0 Registration Reply  0 Purge Request  3 Purge Reply
         0 Error Indication  0 Traffic Indication  0 Redirect Suppress
   Rcvd: Total 21752
         9556 Resolution Request  9471 Resolution Reply  0 Registration Request
         2722 Registration Reply  3 Purge Request  0 Purge Reply
         0 Error Indication  0 Traffic Indication  0 Redirect Suppress
Tunnel20: Max-send limit:100Pkts/10Sec, Usage:0%
   Sent: Total 21904
         9635 Resolution Request  9543 Resolution Reply  2725 Registration Request
         0 Registration Reply  0 Purge Request  1 Purge Reply
         0 Error Indication  0 Traffic Indication  0 Redirect Suppress
   Rcvd: Total 21900
         9638 Resolution Request  9544 Resolution Reply  0 Registration Request
         2717 Registration Reply  1 Purge Request  0 Purge Reply
         0 Error Indication  0 Traffic Indication  0 Redirect Suppress
R71#
R71#
R71#
R71#
R71#show ip nhrp detail
10.100.0.1/32 via 10.100.0.1
   Tunnel10 created 22:07:09, never expire
   Type: static, Flags: used
   NBMA address: 10.10.14.254
10.101.0.1/32 via 10.101.0.1
   Tunnel20 created 22:00:36, never expire
   Type: static, Flags: used
   NBMA address: 10.10.13.254
R71#
R71#
R71#
R71#
R71#show dmvpn
Legend: Attrb --> S - Static, D - Dynamic, I - Incomplete
        N - NATed, L - Local, X - No Socket
        # Ent --> Number of NHRP entries with same NBMA peer
        NHS Status: E --> Expecting Replies, R --> Responding, W --> Waiting
        UpDn Time --> Up or Down Time for a Tunnel
==========================================================================

Interface: Tunnel10, IPv4 NHRP Details
Type:Spoke, NHRP Peers:1,

 # Ent  Peer NBMA Addr Peer Tunnel Add State  UpDn Tm Attrb
 ----- --------------- --------------- ----- -------- -----
     1 10.10.14.254         10.100.0.1    UP 00:20:39     S

Interface: Tunnel20, IPv4 NHRP Details
Type:Spoke, NHRP Peers:1,

 # Ent  Peer NBMA Addr Peer Tunnel Add State  UpDn Tm Attrb
 ----- --------------- --------------- ----- -------- -----
     1 10.10.13.254         10.101.0.1    UP 22:00:35     S


```
</details>