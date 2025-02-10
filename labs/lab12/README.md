# Основные протоколы сети интернет

## Цели

1. Настроить DHCP в офисе Москва
1. Настроить синхронизацию времени в офисе Москва при помощи протокола NTP
1. Настроить NAT в офисе Москва, Санкт-Петербург и Чокурдак


## Задание

1. Настроить NAT (PAT) на R13 и R14 в офисе Москва. Трансляция должна осуществляться в адрес автономной системы AS1001.
1. Настроить NAT так, чтобы R15 в офисе Москва был доступен с любого узла для удаленного управления.
1. Настроить статический NAT для R16 в офисе Москва.
1. Настроить NAT (PAT) на R52 в офисе Санкт-Петербург. Трансляция должна осуществляться в пул из 5 адресов автономной системы AS2042.
1. настроить статический NAT (PAT) для офиса Чокурдак.
1. Настроить IPv4 DHCP сервер в офисе Москва на маршрутизаторах R11 и R12. VPC11 и VPC12 должны получать сетевые настройки по DHCP.
1. Настроить NTP сервер на R11 и R12. Все устройства в офисе Москва должны синхронизировать время с R11 и R12.

## Топология

![a](media/lab12_1.PNG)

## Схема для импорта в PNETlab

[Схема для импорта в PNETlab](media/otus_cource_lab12_NAT_NTP_DHCP_pnetlab_export-20250210-195658.zip)

## Версии ПО

- PNETlab - 5.3.11
- Роутеры - Cisco IOS Software, Linux Software (I86BI_LINUX-ADVENTERPRISEK9-M), Version 15.4(2)T4
- Коммутаторы - Cisco IOS Software, Linux Software (I86BI_LINUXL2-ADVENTERPRISEK9-M), Version 15.2(CML_NIGHTLY_20150703)
- ПК - VPC

## Решение

Будет использоваться NAT Virtual Interface.

1. Настроить NAT(PAT) на R13 и R14 в офисе Москва. Трансляция должна осуществляться в адрес автономной системы AS1001.
      
      Для трансляции IP адресов в Москве выберем подсеть 10.10.13.0/24 на R13 и подсеть 10.10.14.0/24 на R14. Данные подсети нужно задавать на внешнем сетевом интерфейсе (e0/2) как secondary, чтобы проанонсировать данную посеть в eBGP (маршрут через Null0 не подойдет в силу работы NAT Virtual Interface, ответные пакеты будут отброшены)
      Для PAT на R13 выберм пул из 10 адресов - 10.10.13.1 - 10.10.13.10, для PAT на R14 выберем пул из 10 адресов - 10.10.14.1 - 10.10.14.10.  

      Пример настроек на R13:

      ```
      interface Ethernet0/0
       ...
       ip nat enable
       ...

      interface Ethernet0/1
       ...
       ip nat enable
       ...
      
      interface Ethernet0/2
       ...
       ip address 10.10.13.254 255.255.255.0 secondary
       ip nat enable
       ... 

      interface Ethernet0/3
       ...
       ip nat enable
       ...
      
      router bgp 1001
       ...
       network 10.10.13.0 mask 255.255.255.0
       ....
      
      ip nat pool PUBLIC_POOL 10.10.13.1 10.10.13.10 netmask 255.255.255.0
      ip nat source route-map NAT_FOR_G0/2 pool PUBLIC_POOL overload

      route-map NAT_FOR_G0/2 permit 10
       match ip address 2
       match interface Ethernet0/2

      access-list 2 permit 10.10.0.0 0.0.255.255
      ```

      Пример настроек на R14:

      ```
      interface Ethernet0/0
       ...
       ip nat enable
       ...

      interface Ethernet0/1
       ...
       ip nat enable
       ...
      
      interface Ethernet0/2
       ...
       ip address 10.10.14.254 255.255.255.0 secondary
       ip nat enable
       ... 

      interface Ethernet0/3
       ...
       ip nat enable
       ...
      
      router bgp 1001
       ...
       network 10.10.14.0 mask 255.255.255.0
       ....
      
      ip nat pool PUBLIC_POOL 10.10.14.1 10.10.14.10 netmask 255.255.255.0
      ip nat source route-map NAT_FOR_G0/2 pool PUBLIC_POOL overload

      route-map NAT_FOR_G0/2 permit 10
       match ip address 2
       match interface Ethernet0/2

      access-list 2 permit 10.10.0.0 0.0.255.255
      ```      

1. Настроить NAT так, чтобы R15 в офисе Москва был доступен с любого узла для удаленного управления.

      Например можно настроить статический NAT на R13 и избавиться от асимметрии (чтобы входящие пакеты, пришедшие со стороны Киторна не маршрутизировались R13 в сторону Ламаса). Чтобы избавиться от асиметрии было решено на R13 и R14 получать только маршрут по умолчанию от R21 и R31. Пример настройки статичского NAT (для TCP пакетов, пришедших на 22 порт на адрес 10.10.13.11 выполняетя изменение DST адерса на 10.10.0.15):
      
      ```

      ip nat source static tcp 10.10.0.15 22 10.10.13.11 22 extendable

      ```

1. Настроить статический NAT для R16 в офисе Москва.

      ```

      ip nat source static 10.10.0.16 10.10.14.11

      ```

1. Настроить NAT (PAT) на R52 в офисе Санкт-Петербурге. Трансляция должна осуществляться в пул из 5 адресов автономной системы AS2042.

      Для трансляции IP адресов в Санкт-Петербурге выберем 2 подсети 10.50.52.0/25 (пул из 5 адресов - 10.50.52.1 - 10.50.52.5 для интерфейса e0/2), 10.50.52.128/25 (пул из 5 адресов - 10.50.52.129 - 10.50.52.134 для интерфейса e0/3). Такое разделение необходимо для симметричности прохождения пакетов (чтобы обратные пакеты попадали на тот же маршрутизатор, где был выполнен PAT)
      Данные подсети нужно задать на loopback интерфейсах (lo2 и lo3), чтобы проанонсировать данные подсети в eBGP (маршрут через Null0 не подойдет в силу работы NAT Virtual Interface, ответные пакеты будут отброшены)

      ```
      interface Loopback2
       ip address 10.50.52.126 255.255.255.128
       ip nat enable

      interface Loopback3
       ip address 10.50.52.254 255.255.255.128
       ip nat enable

      interface Ethernet0/0
       ip address 10.50.4.52 255.255.255.0
       ip nat enable

      interface Ethernet0/1
       ip address 10.50.3.52 255.255.255.0
       ip nat enable

      interface Ethernet0/2
       ip address 172.16.6.52 255.255.255.0
       ip nat enable

      interface Ethernet0/3
       ip address 172.16.7.52 255.255.255.0
       ip nat enable

      router bgp 2042
       ...
       network 10.50.52.0 mask 255.255.255.128
       network 10.50.52.128 mask 255.255.255.128
       ...
       neighbor 172.16.6.44 remote-as 520
       neighbor 172.16.6.44 prefix-list MSK_NET_WO_NAT1 out
       neighbor 172.16.7.43 remote-as 520
       neighbor 172.16.7.43 prefix-list MSK_NET_WO_NAT2 out

      ip nat pool PUBLIC_POOL_G0/2 10.50.52.1 10.50.52.5 netmask 255.255.255.128
      ip nat pool PUBLIC_POOL_G0/3 10.50.52.129 10.50.52.133 netmask 255.255.255.128
      ip nat source route-map NAT_FOR_G0/2 pool PUBLIC_POOL_G0/2 overload
      ip nat source route-map NAT_FOR_G0/3 pool PUBLIC_POOL_G0/3 overload

      ip prefix-list MSK_NET_WO_NAT1 seq 4 deny 10.50.52.128/25
      ip prefix-list MSK_NET_WO_NAT1 seq 5 permit 10.50.0.0/16 le 32

      ip prefix-list MSK_NET_WO_NAT2 seq 4 deny 10.50.52.0/25
      ip prefix-list MSK_NET_WO_NAT2 seq 5 permit 10.50.0.0/16 le 32

      route-map NAT_FOR_G0/2 permit 10
       match ip address 2
       match interface Ethernet0/2

      route-map NAT_FOR_G0/3 permit 10
       match ip address 2
       match interface Ethernet0/3

      access-list 2 permit 10.50.0.0 0.0.255.255

    ```

1. Настроить статический NAT (PAT) для офиса Чокурдак

    Настроим статический NAT для доступа по SSH на коммутатор SW71 по IP адерсу (порт 22) порта Ethernet0/0 роутера R71

    ```

    interface Ethernet0/0
     ...
     ip nat enable

    interface Ethernet0/2.1000
     ...
     ip nat enable
 
    ip nat source static tcp 10.70.255.71 22 interface Ethernet0/0 22

    ```

1. Настроить IPv4 DHCP сервер в офисе Москва на маршрутизаторах R11 и R12. VPC11 и VPC12 должны получать сетевые настройки по DHCP

    Настроим Failover DHCP для подсетей 10.10.1.0/24 и 10.10.2.0/24 с разделением пула на две части, первая половина на R11, вторая на R12.

    Настройка на R11:

    ```

    ip dhcp pool LAN_POOL_1
     network 10.10.1.0 255.255.255.0
     default-router 10.10.1.1
    ip dhcp excluded-address 10.10.1.1
    ip dhcp excluded-address 10.10.1.128 10.10.1.254


    ip dhcp pool LAN_POOL_2
     network 10.10.2.0 255.255.255.0
     default-router 10.10.2.1
    ip dhcp excluded-address 10.10.2.1
    ip dhcp excluded-address 10.10.2.128 10.10.2.254

    ```
    
    Настройка на R12:

    ```

    ip dhcp pool LAN_POOL_1
     network 10.10.1.0 255.255.255.0
     default-router 10.10.1.1
    ip dhcp excluded-address 10.10.1.254
    ip dhcp excluded-address 10.10.1.1 10.10.1.127

    ip dhcp pool LAN_POOL_2
     network 10.10.2.0 255.255.255.0
     default-router 10.10.2.2
    ip dhcp excluded-address 10.10.2.254
    ip dhcp excluded-address 10.10.1.2 10.10.2.127

    ```   

1. Настроить NTP сервер на R11 и R12. Все устройства в офисе Москва должны синхронизировать время с R11 и R12

    Настрйоки на R11 и R12:

    ```

    clock timezone MSK 4
    ntp master 2
    ntp server ru.pool.ntp.org prefer
    ntp server 0.ru.pool.ntp.org
    ntp server 1.ru.pool.ntp.org
    ntp server 2.ru.pool.ntp.org
    ntp server 3.ru.pool.ntp.org

    ```

    На остальных устройствах прописать:

    ```

    ntp server 10.10.1.1 prefer
    ntp server 10.10.2.1

    ```

## Конфигурации устройств

### R11

<details>
  <summary>Конфигурация</summary>

```

R11#terminal length 0
R11#sh run
Building configuration...

Current configuration : 2532 bytes
!
! Last configuration change at 22:41:10 MSK Mon Feb 10 2025
! NVRAM config last updated at 22:43:55 MSK Mon Feb 10 2025
!
version 15.4
service timestamps debug datetime msec
service timestamps log datetime msec
no service password-encryption
!
hostname R11
!
boot-start-marker
boot-end-marker
!
!
!
no aaa new-model
clock timezone MSK 4 0
mmi polling-interval 60
no mmi auto-configure
no mmi pvc
mmi snmp-timeout 180
no ip icmp rate-limit unreachable
!
!
!
!
!
!
!
!


!
ip dhcp excluded-address 10.10.1.1
ip dhcp excluded-address 10.10.1.128 10.10.1.254
ip dhcp excluded-address 10.10.2.1
ip dhcp excluded-address 10.10.2.128 10.10.2.254
!
ip dhcp pool LAN_POOL_1
 network 10.10.1.0 255.255.255.0
 default-router 10.10.1.1
!
ip dhcp pool LAN_POOL_2
 network 10.10.2.0 255.255.255.0
 default-router 10.10.2.1
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
no cdp log mismatch duplex
!
ip tcp synwait-time 5
!
!
!
!
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
 ip address 10.10.0.11 255.255.255.255
 ip ospf 1 area 10
!
interface Ethernet0/0
 no ip address
!
interface Ethernet0/0.10
 encapsulation dot1Q 10
 ip address 10.10.1.253 255.255.255.0
 ip ospf 1 area 10
 vrrp 1 ip 10.10.1.1
 vrrp 1 priority 150
!
interface Ethernet0/0.20
 encapsulation dot1Q 20
 ip address 10.10.2.253 255.255.255.0
 ip ospf 1 area 10
 vrrp 3 ip 10.10.2.1
 vrrp 3 priority 150
!
interface Ethernet0/0.1000
 encapsulation dot1Q 1000
 ip address 10.10.255.253 255.255.255.0
 ip ospf 1 area 10
 vrrp 2 ip 10.10.255.1
 vrrp 2 priority 150
!
interface Ethernet0/1
 ip address 10.10.9.11 255.255.255.0
 ip ospf 1 area 10
!
interface Ethernet0/2
 ip address 10.10.3.11 255.255.255.0
 ip ospf 1 area 0
!
interface Ethernet0/3
 ip address 10.10.8.11 255.255.255.0
 ip ospf 1 area 0
 ip ospf cost 1
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
router ospf 1
 router-id 10.10.0.11
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
 exec-timeout 0 0
 privilege level 15
 logging synchronous
line aux 0
 exec-timeout 0 0
 privilege level 15
 logging synchronous
line vty 0 4
 login
 transport input none
!
ntp master 2
ntp server ru.pool.ntp.org prefer
ntp server 0.ru.pool.ntp.org
ntp server 1.ru.pool.ntp.org
ntp server 2.ru.pool.ntp.org
ntp server 3.ru.pool.ntp.org
!
end

R11#
R11#
R11#
R11#show ip interface brief
Interface                  IP-Address      OK? Method Status                Protocol
Ethernet0/0                unassigned      YES NVRAM  up                    up
Ethernet0/0.10             10.10.1.253     YES NVRAM  up                    up
Ethernet0/0.20             10.10.2.253     YES NVRAM  up                    up
Ethernet0/0.1000           10.10.255.253   YES NVRAM  up                    up
Ethernet0/1                10.10.9.11      YES NVRAM  up                    up
Ethernet0/2                10.10.3.11      YES NVRAM  up                    up
Ethernet0/3                10.10.8.11      YES NVRAM  up                    up
Ethernet1/0                unassigned      YES NVRAM  administratively down down
Ethernet1/1                unassigned      YES NVRAM  administratively down down
Ethernet1/2                unassigned      YES NVRAM  administratively down down
Ethernet1/3                unassigned      YES NVRAM  administratively down down
Loopback1                  10.10.0.11      YES NVRAM  up                    up
R11#
R11#
R11#
R11#
R11#show ip route
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
       a - application route
       + - replicated route, % - next hop override

Gateway of last resort is 10.10.8.14 to network 0.0.0.0

O*E2  0.0.0.0/0 [110/1] via 10.10.8.14, 1d02h, Ethernet0/3
      10.0.0.0/8 is variably subnetted, 23 subnets, 2 masks
C        10.10.0.11/32 is directly connected, Loopback1
O        10.10.0.12/32 [110/11] via 10.10.255.254, 2w2d, Ethernet0/0.1000
                       [110/11] via 10.10.9.12, 2w2d, Ethernet0/1
                       [110/11] via 10.10.2.254, 2w2d, Ethernet0/0.20
                       [110/11] via 10.10.1.254, 2w2d, Ethernet0/0.10
O        10.10.0.13/32 [110/11] via 10.10.3.13, 1d05h, Ethernet0/2
O        10.10.0.14/32 [110/2] via 10.10.8.14, 1d02h, Ethernet0/3
O IA     10.10.0.15/32 [110/21] via 10.10.3.13, 1d05h, Ethernet0/2
O IA     10.10.0.16/32 [110/12] via 10.10.8.14, 1d02h, Ethernet0/3
C        10.10.1.0/24 is directly connected, Ethernet0/0.10
L        10.10.1.253/32 is directly connected, Ethernet0/0.10
C        10.10.2.0/24 is directly connected, Ethernet0/0.20
L        10.10.2.253/32 is directly connected, Ethernet0/0.20
C        10.10.3.0/24 is directly connected, Ethernet0/2
L        10.10.3.11/32 is directly connected, Ethernet0/2
O IA     10.10.4.0/24 [110/20] via 10.10.3.13, 1d05h, Ethernet0/2
O        10.10.5.0/24 [110/11] via 10.10.8.14, 1d02h, Ethernet0/3
O IA     10.10.6.0/24 [110/11] via 10.10.8.14, 1d02h, Ethernet0/3
O        10.10.7.0/24 [110/20] via 10.10.3.13, 1d05h, Ethernet0/2
C        10.10.8.0/24 is directly connected, Ethernet0/3
L        10.10.8.11/32 is directly connected, Ethernet0/3
C        10.10.9.0/24 is directly connected, Ethernet0/1
L        10.10.9.11/32 is directly connected, Ethernet0/1
O E2     10.10.13.0/24 [110/1] via 10.10.3.13, 1d05h, Ethernet0/2
C        10.10.255.0/24 is directly connected, Ethernet0/0.1000
L        10.10.255.253/32 is directly connected, Ethernet0/0.1000
      172.16.0.0/24 is subnetted, 1 subnets
O E2     172.16.1.0 [110/1] via 10.10.3.13, 1d05h, Ethernet0/2


```
</details>

### R12

<details>
  <summary>Конфигурация</summary>

```

R12#terminal length 0
R12#sh run
Building configuration...

Current configuration : 2527 bytes
!
! Last configuration change at 22:41:21 MSK Mon Feb 10 2025
! NVRAM config last updated at 22:43:51 MSK Mon Feb 10 2025
!
version 15.4
service timestamps debug datetime msec
service timestamps log datetime msec
no service password-encryption
!
hostname R12
!
boot-start-marker
boot-end-marker
!
!
!
no aaa new-model
clock timezone MSK 4 0
mmi polling-interval 60
no mmi auto-configure
no mmi pvc
mmi snmp-timeout 180
no ip icmp rate-limit unreachable
!
!
!
!
!
!
!
!


!
ip dhcp excluded-address 10.10.1.254
ip dhcp excluded-address 10.10.1.1 10.10.1.127
ip dhcp excluded-address 10.10.2.254
ip dhcp excluded-address 10.10.1.2 10.10.2.127
!
ip dhcp pool LAN_POOL_1
 network 10.10.1.0 255.255.255.0
 default-router 10.10.1.1
!
ip dhcp pool LAN_POOL_2
 network 10.10.2.0 255.255.255.0
 default-router 10.10.2.2
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
no cdp log mismatch duplex
!
ip tcp synwait-time 5
!
!
!
!
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
 ip address 10.10.0.12 255.255.255.0
 ip ospf 1 area 10
!
interface Ethernet0/0
 no ip address
!
interface Ethernet0/0.10
 encapsulation dot1Q 10
 ip address 10.10.1.254 255.255.255.0
 ip ospf 1 area 10
 vrrp 1 ip 10.10.1.1
 vrrp 1 priority 50
!
interface Ethernet0/0.20
 encapsulation dot1Q 20
 ip address 10.10.2.254 255.255.255.0
 ip ospf 1 area 10
 vrrp 3 ip 10.10.2.1
 vrrp 3 priority 50
!
interface Ethernet0/0.1000
 encapsulation dot1Q 1000
 ip address 10.10.255.254 255.255.255.0
 ip ospf 1 area 10
 vrrp 2 ip 10.10.255.1
 vrrp 2 priority 50
!
interface Ethernet0/1
 ip address 10.10.9.12 255.255.255.0
 ip ospf 1 area 10
!
interface Ethernet0/2
 ip address 10.10.5.12 255.255.255.0
 ip ospf 1 area 0
 ip ospf cost 1
!
interface Ethernet0/3
 ip address 10.10.7.12 255.255.255.0
 ip ospf 1 area 0
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
router ospf 1
 router-id 10.10.0.12
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
 exec-timeout 0 0
 privilege level 15
 logging synchronous
line aux 0
 exec-timeout 0 0
 privilege level 15
 logging synchronous
line vty 0 4
 login
 transport input none
!
ntp master 2
ntp server ru.pool.ntp.org prefer
ntp server 0.ru.pool.ntp.org
ntp server 1.ru.pool.ntp.org
ntp server 2.ru.pool.ntp.org
ntp server 3.ru.pool.ntp.org
!
end

R12#
R12#
R12#
R12#show ip interface brief
Interface                  IP-Address      OK? Method Status                Protocol
Ethernet0/0                unassigned      YES NVRAM  up                    up
Ethernet0/0.10             10.10.1.254     YES NVRAM  up                    up
Ethernet0/0.20             10.10.2.254     YES NVRAM  up                    up
Ethernet0/0.1000           10.10.255.254   YES NVRAM  up                    up
Ethernet0/1                10.10.9.12      YES NVRAM  up                    up
Ethernet0/2                10.10.5.12      YES NVRAM  up                    up
Ethernet0/3                10.10.7.12      YES NVRAM  up                    up
Ethernet1/0                unassigned      YES NVRAM  administratively down down
Ethernet1/1                unassigned      YES NVRAM  administratively down down
Ethernet1/2                unassigned      YES NVRAM  administratively down down
Ethernet1/3                unassigned      YES NVRAM  administratively down down
Loopback1                  10.10.0.12      YES NVRAM  up                    up
R12#
R12#
R12#
R12#
R12#show ip route
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
       a - application route
       + - replicated route, % - next hop override

Gateway of last resort is 10.10.5.14 to network 0.0.0.0

O*E2  0.0.0.0/0 [110/1] via 10.10.5.14, 1d02h, Ethernet0/2
      10.0.0.0/8 is variably subnetted, 24 subnets, 2 masks
C        10.10.0.0/24 is directly connected, Loopback1
O        10.10.0.11/32 [110/11] via 10.10.255.253, 2w2d, Ethernet0/0.1000
                       [110/11] via 10.10.9.11, 2w2d, Ethernet0/1
                       [110/11] via 10.10.2.253, 2w2d, Ethernet0/0.20
                       [110/11] via 10.10.1.253, 2w2d, Ethernet0/0.10
L        10.10.0.12/32 is directly connected, Loopback1
O        10.10.0.13/32 [110/11] via 10.10.7.13, 1d05h, Ethernet0/3
O        10.10.0.14/32 [110/2] via 10.10.5.14, 1d03h, Ethernet0/2
O IA     10.10.0.15/32 [110/21] via 10.10.7.13, 1d05h, Ethernet0/3
O IA     10.10.0.16/32 [110/12] via 10.10.5.14, 1d03h, Ethernet0/2
C        10.10.1.0/24 is directly connected, Ethernet0/0.10
L        10.10.1.254/32 is directly connected, Ethernet0/0.10
C        10.10.2.0/24 is directly connected, Ethernet0/0.20
L        10.10.2.254/32 is directly connected, Ethernet0/0.20
O        10.10.3.0/24 [110/20] via 10.10.7.13, 1d05h, Ethernet0/3
O IA     10.10.4.0/24 [110/20] via 10.10.7.13, 1d05h, Ethernet0/3
C        10.10.5.0/24 is directly connected, Ethernet0/2
L        10.10.5.12/32 is directly connected, Ethernet0/2
O IA     10.10.6.0/24 [110/11] via 10.10.5.14, 1d03h, Ethernet0/2
C        10.10.7.0/24 is directly connected, Ethernet0/3
L        10.10.7.12/32 is directly connected, Ethernet0/3
O        10.10.8.0/24 [110/11] via 10.10.5.14, 1d03h, Ethernet0/2
C        10.10.9.0/24 is directly connected, Ethernet0/1
L        10.10.9.12/32 is directly connected, Ethernet0/1
O E2     10.10.13.0/24 [110/1] via 10.10.7.13, 1d05h, Ethernet0/3
C        10.10.255.0/24 is directly connected, Ethernet0/0.1000
L        10.10.255.254/32 is directly connected, Ethernet0/0.1000
      172.16.0.0/24 is subnetted, 1 subnets
O E2     172.16.1.0 [110/1] via 10.10.7.13, 1d05h, Ethernet0/3


```
</details>

### R13

<details>
  <summary>Конфигурация</summary>

```

R13#terminal length 0
R13#sh run
Building configuration...

Current configuration : 2268 bytes
!
! Last configuration change at 13:52:21 UTC Sun Feb 9 2025
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
 redistribute bgp 1001 subnets
 default-information originate
!
router bgp 1001
 bgp log-neighbor-changes
 network 10.10.13.0 mask 255.255.255.0
 network 172.16.1.0 mask 255.255.255.0
 redistribute ospf 1
 neighbor 10.10.0.14 remote-as 1001
 neighbor 10.10.0.14 update-source Loopback1
 neighbor 172.16.1.21 remote-as 101
 neighbor 172.16.1.21 filter-list 1 out
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
ip prefix-list OSPF-FILTER seq 10 deny 10.10.0.0/16 le 32
ip prefix-list OSPF-FILTER seq 20 permit 0.0.0.0/0 le 32
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

B*    0.0.0.0/0 [20/0] via 172.16.1.21, 1d04h
      10.0.0.0/8 is variably subnetted, 23 subnets, 4 masks
O IA     10.10.0.11/32 [110/11] via 10.10.3.11, 1d05h, Ethernet0/0
O IA     10.10.0.12/32 [110/11] via 10.10.7.12, 1d05h, Ethernet0/1
C        10.10.0.13/32 is directly connected, Loopback1
O        10.10.0.14/32 [110/12] via 10.10.7.12, 1d03h, Ethernet0/1
                       [110/12] via 10.10.3.11, 1d03h, Ethernet0/0
O        10.10.0.15/32 [110/11] via 10.10.4.15, 1d05h, Ethernet0/3
O IA     10.10.0.16/32 [110/22] via 10.10.7.12, 1d03h, Ethernet0/1
                       [110/22] via 10.10.3.11, 1d03h, Ethernet0/0
O IA     10.10.1.0/24 [110/20] via 10.10.7.12, 1d05h, Ethernet0/1
                      [110/20] via 10.10.3.11, 1d05h, Ethernet0/0
O IA     10.10.2.0/24 [110/20] via 10.10.7.12, 1d05h, Ethernet0/1
                      [110/20] via 10.10.3.11, 1d05h, Ethernet0/0
C        10.10.3.0/24 is directly connected, Ethernet0/0
L        10.10.3.13/32 is directly connected, Ethernet0/0
C        10.10.4.0/24 is directly connected, Ethernet0/3
L        10.10.4.13/32 is directly connected, Ethernet0/3
O        10.10.5.0/24 [110/11] via 10.10.7.12, 1d03h, Ethernet0/1
O IA     10.10.6.0/24 [110/21] via 10.10.7.12, 1d03h, Ethernet0/1
                      [110/21] via 10.10.3.11, 1d03h, Ethernet0/0
C        10.10.7.0/24 is directly connected, Ethernet0/1
L        10.10.7.13/32 is directly connected, Ethernet0/1
O        10.10.8.0/24 [110/11] via 10.10.3.11, 1d03h, Ethernet0/0
O IA     10.10.9.0/24 [110/20] via 10.10.7.12, 1d05h, Ethernet0/1
                      [110/20] via 10.10.3.11, 1d05h, Ethernet0/0
C        10.10.13.0/24 is directly connected, Ethernet0/2
L        10.10.13.11/32 is directly connected, Ethernet0/2
L        10.10.13.254/32 is directly connected, Ethernet0/2
B        10.10.14.0/24 [200/0] via 10.10.0.14, 1d04h
O IA     10.10.255.0/24 [110/20] via 10.10.7.12, 1d05h, Ethernet0/1
                        [110/20] via 10.10.3.11, 1d05h, Ethernet0/0
      172.16.0.0/16 is variably subnetted, 3 subnets, 2 masks
C        172.16.1.0/24 is directly connected, Ethernet0/2
L        172.16.1.13/32 is directly connected, Ethernet0/2
B        172.16.2.0/24 [200/0] via 10.10.0.14, 1d04h
R13#
R13#
R13#
R13#
R13#show ip bgp
BGP table version is 72, local router ID is 10.10.0.13
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter,
              x best-external, a additional-path, c RIB-compressed,
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 * i 0.0.0.0          172.16.2.31              0    100      0 301 i
 *>                   172.16.1.21                            0 101 i
 *>  10.10.0.11/32    10.10.3.11              11         32768 ?
 * i                  10.10.8.11              11    100      0 ?
 *>  10.10.0.12/32    10.10.7.12              11         32768 ?
 * i                  10.10.5.12              11    100      0 ?
 *>  10.10.0.13/32    0.0.0.0                  0         32768 ?
 * i                  10.10.5.12              21    100      0 ?
 *>  10.10.0.14/32    10.10.3.11              12         32768 ?
 * i                  10.10.0.14               0    100      0 ?
 *>  10.10.0.15/32    10.10.4.15              11         32768 ?
 * i                  10.10.5.12              31    100      0 ?
 *>  10.10.0.16/32    10.10.3.11              22         32768 ?
 * i                  10.10.6.16              11    100      0 ?
 *>  10.10.1.0/24     10.10.3.11              20         32768 ?
 * i                  10.10.5.12              20    100      0 ?
 *>  10.10.2.0/24     10.10.3.11              20         32768 ?
 * i                  10.10.5.12              20    100      0 ?
 *>  10.10.3.0/24     0.0.0.0                  0         32768 ?
 * i                  10.10.8.11              20    100      0 ?
 *>  10.10.4.0/24     0.0.0.0                  0         32768 ?
 * i                  10.10.5.12              30    100      0 ?
 *>  10.10.5.0/24     10.10.7.12              11         32768 ?
 * i                  10.10.0.14               0    100      0 ?
 *>  10.10.6.0/24     10.10.3.11              21         32768 ?
 * i                  10.10.0.14               0    100      0 ?
 *>  10.10.7.0/24     0.0.0.0                  0         32768 ?
 * i                  10.10.5.12              20    100      0 ?
 *>  10.10.8.0/24     10.10.3.11              11         32768 ?
 * i                  10.10.0.14               0    100      0 ?
 *>  10.10.9.0/24     10.10.3.11              20         32768 ?
 * i                  10.10.5.12              20    100      0 ?
 *>  10.10.13.0/24    0.0.0.0                  0         32768 i
 *>i 10.10.14.0/24    10.10.0.14               0    100      0 i
 *>  10.10.255.0/24   10.10.3.11              20         32768 ?
 * i                  10.10.5.12              20    100      0 ?
 *>  172.16.1.0/24    0.0.0.0                  0         32768 i
 *>i 172.16.2.0/24    10.10.0.14               0    100      0 i
R13#
R13#
R13#
R13#
R13#show ip bgp summary
BGP router identifier 10.10.0.13, local AS number 1001
BGP table version is 72, main routing table version 72
21 network entries using 2940 bytes of memory
38 path entries using 3040 bytes of memory
16/9 BGP path/bestpath attribute entries using 2304 bytes of memory
2 BGP AS-PATH entries using 48 bytes of memory
0 BGP route-map cache entries using 0 bytes of memory
0 BGP filter-list cache entries using 0 bytes of memory
BGP using 8332 total bytes of memory
BGP activity 430/409 prefixes, 1061/1023 paths, scan interval 60 secs

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
10.10.0.14      4         1001    1926    1931       72    0    0 1d04h          19
172.16.1.21     4          101    1919    1932       72    0    0 1d04h           1


```
</details>

### R14

<details>
  <summary>Конфигурация</summary>

```

R14#terminal length 0
R14#sh run
Building configuration...

Current configuration : 2076 bytes
!
! Last configuration change at 16:42:42 UTC Sun Feb 9 2025
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
 network 10.10.14.0 mask 255.255.255.0
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
ip nat pool PUBLIC_POOL 10.10.14.1 10.10.14.10 netmask 255.255.255.0
ip nat source route-map NAT_FOR_G0/2 pool PUBLIC_POOL overload
ip nat source static 10.10.0.16 10.10.14.11
!
!
ip prefix-list OSPF-FILTER seq 10 deny 10.10.4.0/24
ip prefix-list OSPF-FILTER seq 20 permit 0.0.0.0/0 le 32
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

B*    0.0.0.0/0 [20/0] via 172.16.2.31, 1d02h
      10.0.0.0/8 is variably subnetted, 23 subnets, 3 masks
O IA     10.10.0.11/32 [110/11] via 10.10.8.11, 1d04h, Ethernet0/1
O IA     10.10.0.12/32 [110/11] via 10.10.5.12, 1d04h, Ethernet0/0
O        10.10.0.13/32 [110/21] via 10.10.8.11, 1d04h, Ethernet0/1
                       [110/21] via 10.10.5.12, 1d04h, Ethernet0/0
C        10.10.0.14/32 is directly connected, Loopback1
O IA     10.10.0.15/32 [110/31] via 10.10.8.11, 1d04h, Ethernet0/1
                       [110/31] via 10.10.5.12, 1d04h, Ethernet0/0
O        10.10.0.16/32 [110/11] via 10.10.6.16, 1d04h, Ethernet0/3
O IA     10.10.1.0/24 [110/20] via 10.10.8.11, 1d04h, Ethernet0/1
                      [110/20] via 10.10.5.12, 1d04h, Ethernet0/0
O IA     10.10.2.0/24 [110/20] via 10.10.8.11, 1d04h, Ethernet0/1
                      [110/20] via 10.10.5.12, 1d04h, Ethernet0/0
O        10.10.3.0/24 [110/20] via 10.10.8.11, 1d04h, Ethernet0/1
O IA     10.10.4.0/24 [110/30] via 10.10.8.11, 1d04h, Ethernet0/1
                      [110/30] via 10.10.5.12, 1d04h, Ethernet0/0
C        10.10.5.0/24 is directly connected, Ethernet0/0
L        10.10.5.14/32 is directly connected, Ethernet0/0
C        10.10.6.0/24 is directly connected, Ethernet0/3
L        10.10.6.14/32 is directly connected, Ethernet0/3
O        10.10.7.0/24 [110/20] via 10.10.5.12, 1d04h, Ethernet0/0
C        10.10.8.0/24 is directly connected, Ethernet0/1
L        10.10.8.14/32 is directly connected, Ethernet0/1
O IA     10.10.9.0/24 [110/20] via 10.10.8.11, 1d04h, Ethernet0/1
                      [110/20] via 10.10.5.12, 1d04h, Ethernet0/0
O E2     10.10.13.0/24 [110/1] via 10.10.8.11, 1d04h, Ethernet0/1
                       [110/1] via 10.10.5.12, 1d04h, Ethernet0/0
C        10.10.14.0/24 is directly connected, Ethernet0/2
L        10.10.14.11/32 is directly connected, Ethernet0/2
L        10.10.14.254/32 is directly connected, Ethernet0/2
O IA     10.10.255.0/24 [110/20] via 10.10.8.11, 1d04h, Ethernet0/1
                        [110/20] via 10.10.5.12, 1d04h, Ethernet0/0
      172.16.0.0/16 is variably subnetted, 3 subnets, 2 masks
O E2     172.16.1.0/24 [110/1] via 10.10.8.11, 1d04h, Ethernet0/1
                       [110/1] via 10.10.5.12, 1d04h, Ethernet0/0
C        172.16.2.0/24 is directly connected, Ethernet0/2
L        172.16.2.14/32 is directly connected, Ethernet0/2
R14#
R14#
R14#
R14#
R14#show ip bgp
BGP table version is 87, local router ID is 10.10.0.14
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter,
              x best-external, a additional-path, c RIB-compressed,
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 *>  0.0.0.0          172.16.2.31                            0 301 i
 * i                  172.16.1.21              0    100      0 101 i
 * i 10.10.0.11/32    10.10.3.11              11    100      0 ?
 *>                   10.10.8.11              11         32768 ?
 * i 10.10.0.12/32    10.10.7.12              11    100      0 ?
 *>                   10.10.5.12              11         32768 ?
 * i 10.10.0.13/32    10.10.0.13               0    100      0 ?
 *>                   10.10.5.12              21         32768 ?
 * i 10.10.0.14/32    10.10.3.11              12    100      0 ?
 *>                   0.0.0.0                  0         32768 ?
 * i 10.10.0.15/32    10.10.4.15              11    100      0 ?
 *>                   10.10.5.12              31         32768 ?
 * i 10.10.0.16/32    10.10.3.11              22    100      0 ?
 *>                   10.10.6.16              11         32768 ?
 * i 10.10.1.0/24     10.10.3.11              20    100      0 ?
 *>                   10.10.5.12              20         32768 ?
 * i 10.10.2.0/24     10.10.3.11              20    100      0 ?
 *>                   10.10.5.12              20         32768 ?
 * i 10.10.3.0/24     10.10.0.13               0    100      0 ?
 *>                   10.10.8.11              20         32768 ?
 * i 10.10.4.0/24     10.10.0.13               0    100      0 ?
 *>                   10.10.5.12              30         32768 ?
 * i 10.10.5.0/24     10.10.7.12              11    100      0 ?
 *>                   0.0.0.0                  0         32768 ?
 * i 10.10.6.0/24     10.10.3.11              21    100      0 ?
 *>                   0.0.0.0                  0         32768 ?
 * i 10.10.7.0/24     10.10.0.13               0    100      0 ?
 *>                   10.10.5.12              20         32768 ?
 * i 10.10.8.0/24     10.10.3.11              11    100      0 ?
 *>                   0.0.0.0                  0         32768 ?
 * i 10.10.9.0/24     10.10.3.11              20    100      0 ?
 *>                   10.10.5.12              20         32768 ?
 r>i 10.10.13.0/24    10.10.0.13               0    100      0 i
 *>  10.10.14.0/24    0.0.0.0                  0         32768 i
 * i 10.10.255.0/24   10.10.3.11              20    100      0 ?
 *>                   10.10.5.12              20         32768 ?
 r>i 172.16.1.0/24    10.10.0.13               0    100      0 i
 *>  172.16.2.0/24    0.0.0.0                  0         32768 i
R14#
R14#
R14#
R14#
R14#show ip bgp summary
BGP router identifier 10.10.0.14, local AS number 1001
BGP table version is 87, main routing table version 87
21 network entries using 2940 bytes of memory
38 path entries using 3040 bytes of memory
16/9 BGP path/bestpath attribute entries using 2304 bytes of memory
2 BGP AS-PATH entries using 48 bytes of memory
0 BGP route-map cache entries using 0 bytes of memory
0 BGP filter-list cache entries using 0 bytes of memory
BGP using 8332 total bytes of memory
BGP activity 36/15 prefixes, 71/33 paths, scan interval 60 secs

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
10.10.0.13      4         1001    1932    1927       87    0    0 1d04h          19
172.16.2.31     4          301    1744    1757       87    0    0 1d02h           1

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

      10.0.0.0/8 is variably subnetted, 44 subnets, 4 masks
B        10.10.0.11/32 [20/11] via 172.16.1.13, 1d04h
B        10.10.0.12/32 [20/11] via 172.16.1.13, 1d04h
B        10.10.0.13/32 [20/0] via 172.16.1.13, 1d04h
B        10.10.0.14/32 [20/12] via 172.16.1.13, 1d03h
B        10.10.0.15/32 [20/11] via 172.16.1.13, 1d04h
B        10.10.0.16/32 [20/22] via 172.16.1.13, 1d03h
B        10.10.1.0/24 [20/20] via 172.16.1.13, 1d04h
B        10.10.2.0/24 [20/20] via 172.16.1.13, 1d04h
B        10.10.3.0/24 [20/0] via 172.16.1.13, 1d04h
B        10.10.4.0/24 [20/0] via 172.16.1.13, 1d04h
B        10.10.5.0/24 [20/11] via 172.16.1.13, 1d03h
B        10.10.6.0/24 [20/21] via 172.16.1.13, 1d03h
B        10.10.7.0/24 [20/0] via 172.16.1.13, 1d04h
B        10.10.8.0/24 [20/11] via 172.16.1.13, 1d03h
B        10.10.9.0/24 [20/20] via 172.16.1.13, 1d04h
B        10.10.13.0/24 [20/0] via 172.16.1.13, 1d04h
B        10.10.14.0/24 [20/0] via 172.16.1.13, 1d04h
B        10.10.255.0/24 [20/20] via 172.16.1.13, 1d04h
S        10.20.0.0/16 is directly connected, Null0
C        10.20.0.21/32 is directly connected, Loopback1
B        10.30.0.0/16 [20/0] via 172.16.3.31, 2w0d
B        10.40.0.41/32 [20/0] via 172.16.5.41, 2w0d
B        10.40.0.42/32 [20/20] via 172.16.5.41, 2w0d
B        10.40.0.43/32 [20/30] via 172.16.5.41, 2w0d
B        10.40.0.44/32 [20/20] via 172.16.5.41, 2w0d
B        10.40.1.0/24 [20/0] via 172.16.5.41, 2w0d
B        10.40.2.0/24 [20/0] via 172.16.5.41, 2w0d
B        10.40.3.0/24 [20/20] via 172.16.5.41, 2w0d
B        10.40.4.0/24 [20/20] via 172.16.5.41, 2w0d
B        10.50.0.51/32 [20/0] via 172.16.5.41, 2w0d
B        10.50.0.52/32 [20/0] via 172.16.5.41, 2w0d
B        10.50.0.53/32 [20/0] via 172.16.5.41, 2w0d
B        10.50.0.54/32 [20/0] via 172.16.5.41, 2w0d
B        10.50.1.0/24 [20/0] via 172.16.5.41, 2w0d
B        10.50.2.0/24 [20/0] via 172.16.5.41, 2w0d
B        10.50.3.0/24 [20/0] via 172.16.5.41, 2w0d
B        10.50.4.0/24 [20/0] via 172.16.5.41, 2w0d
B        10.50.5.0/24 [20/0] via 172.16.5.41, 2w0d
B        10.50.6.0/24 [20/0] via 172.16.5.41, 2w0d
B        10.50.52.0/25 [20/0] via 172.16.5.41, 1w1d
B        10.50.52.128/25 [20/0] via 172.16.5.41, 5d00h
B        10.50.255.0/24 [20/0] via 172.16.5.41, 2w0d
B        10.60.0.0/16 [20/0] via 172.16.5.41, 2w0d
B        10.70.0.0/16 [20/0] via 172.16.5.41, 2w0d
      172.16.0.0/16 is variably subnetted, 13 subnets, 2 masks
C        172.16.1.0/24 is directly connected, Ethernet0/0
L        172.16.1.21/32 is directly connected, Ethernet0/0
B        172.16.2.0/24 [20/0] via 172.16.3.31, 2w0d
C        172.16.3.0/24 is directly connected, Ethernet0/1
L        172.16.3.21/32 is directly connected, Ethernet0/1
B        172.16.4.0/24 [20/0] via 172.16.3.31, 2w0d
C        172.16.5.0/24 is directly connected, Ethernet0/2
L        172.16.5.21/32 is directly connected, Ethernet0/2
B        172.16.6.0/24 [20/0] via 172.16.5.41, 2w0d
B        172.16.7.0/24 [20/0] via 172.16.5.41, 2w0d
B        172.16.8.0/24 [20/0] via 172.16.5.41, 2w0d
B        172.16.9.0/24 [20/0] via 172.16.5.41, 2w0d
B        172.16.10.0/24 [20/0] via 172.16.5.41, 2w0d
R21#
R21#
R21#
R21#
R21#show ip bgp
BGP table version is 657, local router ID is 10.20.0.21
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
 *>                   172.16.1.13             12             0 1001 ?
 *   10.10.0.15/32    172.16.3.31                            0 301 1001 ?
 *>                   172.16.1.13             11             0 1001 ?
 *   10.10.0.16/32    172.16.3.31                            0 301 1001 ?
 *>                   172.16.1.13             22             0 1001 ?
 *   10.10.1.0/24     172.16.3.31                            0 301 1001 ?
 *>                   172.16.1.13             20             0 1001 ?
 *   10.10.2.0/24     172.16.3.31                            0 301 1001 ?
 *>                   172.16.1.13             20             0 1001 ?
 *   10.10.3.0/24     172.16.3.31                            0 301 1001 ?
 *>                   172.16.1.13              0             0 1001 ?
 *   10.10.4.0/24     172.16.3.31                            0 301 1001 ?
 *>                   172.16.1.13              0             0 1001 ?
 *   10.10.5.0/24     172.16.3.31                            0 301 1001 ?
 *>                   172.16.1.13             11             0 1001 ?
 *   10.10.6.0/24     172.16.3.31                            0 301 1001 ?
 *>                   172.16.1.13             21             0 1001 ?
 *   10.10.7.0/24     172.16.3.31                            0 301 1001 ?
 *>                   172.16.1.13              0             0 1001 ?
 *   10.10.8.0/24     172.16.3.31                            0 301 1001 ?
 *>                   172.16.1.13             11             0 1001 ?
 *   10.10.9.0/24     172.16.3.31                            0 301 1001 ?
 *>                   172.16.1.13             20             0 1001 ?
 *   10.10.13.0/24    172.16.3.31                            0 301 1001 i
 *>                   172.16.1.13              0             0 1001 i
 *   10.10.14.0/24    172.16.3.31                            0 301 1001 i
 *>                   172.16.1.13                            0 1001 i
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
 *   10.50.0.51/32    172.16.3.31                            0 301 520 2042 ?
 *>                   172.16.5.41                            0 520 2042 ?
 *   10.50.0.52/32    172.16.3.31                            0 301 520 2042 ?
 *>                   172.16.5.41                            0 520 2042 ?
 *   10.50.0.53/32    172.16.3.31                            0 301 520 2042 ?
 *>                   172.16.5.41                            0 520 2042 ?
 *   10.50.0.54/32    172.16.3.31                            0 301 520 2042 ?
 *>                   172.16.5.41                            0 520 2042 ?
 *   10.50.1.0/24     172.16.3.31                            0 301 520 2042 ?
 *>                   172.16.5.41                            0 520 2042 ?
 *   10.50.2.0/24     172.16.3.31                            0 301 520 2042 ?
 *>                   172.16.5.41                            0 520 2042 ?
 *   10.50.3.0/24     172.16.3.31                            0 301 520 2042 ?
 *>                   172.16.5.41                            0 520 2042 ?
 *   10.50.4.0/24     172.16.3.31                            0 301 520 2042 ?
 *>                   172.16.5.41                            0 520 2042 ?
 *   10.50.5.0/24     172.16.3.31                            0 301 520 2042 ?
 *>                   172.16.5.41                            0 520 2042 ?
 *   10.50.6.0/24     172.16.3.31                            0 301 520 2042 ?
 *>                   172.16.5.41                            0 520 2042 ?
 *   10.50.52.0/25    172.16.3.31                            0 301 520 2042 i
 *>                   172.16.5.41                            0 520 2042 i
 *   10.50.52.128/25  172.16.3.31                            0 301 520 2042 i
 *>                   172.16.5.41                            0 520 2042 i
 *   10.50.255.0/24   172.16.3.31                            0 301 520 2042 ?
 *>                   172.16.5.41                            0 520 2042 ?
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
BGP table version is 657, main routing table version 657
54 network entries using 7560 bytes of memory
105 path entries using 8400 bytes of memory
28/18 BGP path/bestpath attribute entries using 4032 bytes of memory
8 BGP AS-PATH entries using 192 bytes of memory
0 BGP route-map cache entries using 0 bytes of memory
0 BGP filter-list cache entries using 0 bytes of memory
BGP using 20184 total bytes of memory
BGP activity 236/182 prefixes, 1188/1083 paths, scan interval 60 secs

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
172.16.1.13     4         1001    1934    1921      657    0    0 1d04h          20
172.16.3.31     4          301   23819   23798      657    0    0 2w0d           48
172.16.5.41     4          520   23789   23808      657    0    0 2w0d           32


```
</details>

### R31

<details>
  <summary>Конфигурация</summary>

```

R31#terminal length 0
R31#sh run
Building configuration...

Current configuration : 1818 bytes
!
! Last configuration change at 16:23:38 UTC Sun Feb 9 2025
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
 neighbor 172.16.2.14 route-map EBGP-R14-OUT out
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
ip prefix-list DEFAULT_ONLY seq 5 permit 0.0.0.0/0
!
route-map EBGP-R14-OUT permit 10
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

      10.0.0.0/8 is variably subnetted, 44 subnets, 4 masks
B        10.10.0.11/32 [20/11] via 172.16.2.14, 1d02h
B        10.10.0.12/32 [20/11] via 172.16.2.14, 1d02h
B        10.10.0.13/32 [20/21] via 172.16.2.14, 1d02h
B        10.10.0.14/32 [20/0] via 172.16.2.14, 1d02h
B        10.10.0.15/32 [20/31] via 172.16.2.14, 1d02h
B        10.10.0.16/32 [20/11] via 172.16.2.14, 1d02h
B        10.10.1.0/24 [20/20] via 172.16.2.14, 1d02h
B        10.10.2.0/24 [20/20] via 172.16.2.14, 1d02h
B        10.10.3.0/24 [20/20] via 172.16.2.14, 1d02h
B        10.10.4.0/24 [20/30] via 172.16.2.14, 1d02h
B        10.10.5.0/24 [20/0] via 172.16.2.14, 1d02h
B        10.10.6.0/24 [20/0] via 172.16.2.14, 1d02h
B        10.10.7.0/24 [20/20] via 172.16.2.14, 1d02h
B        10.10.8.0/24 [20/0] via 172.16.2.14, 1d02h
B        10.10.9.0/24 [20/20] via 172.16.2.14, 1d02h
B        10.10.13.0/24 [20/0] via 172.16.2.14, 1d02h
B        10.10.14.0/24 [20/0] via 172.16.2.14, 1d02h
B        10.10.255.0/24 [20/20] via 172.16.2.14, 1d02h
B        10.20.0.0/16 [20/0] via 172.16.3.21, 2w0d
S        10.30.0.0/16 is directly connected, Null0
C        10.30.0.31/32 is directly connected, Loopback1
B        10.40.0.41/32 [20/20] via 172.16.4.44, 2w1d
B        10.40.0.42/32 [20/30] via 172.16.4.44, 2w1d
B        10.40.0.43/32 [20/20] via 172.16.4.44, 2w1d
B        10.40.0.44/32 [20/0] via 172.16.3.21, 2w0d
B        10.40.1.0/24 [20/0] via 172.16.3.21, 2w0d
B        10.40.2.0/24 [20/20] via 172.16.4.44, 2w1d
B        10.40.3.0/24 [20/20] via 172.16.4.44, 2w1d
B        10.40.4.0/24 [20/0] via 172.16.3.21, 2w0d
B        10.50.0.51/32 [20/0] via 172.16.4.44, 5d00h
B        10.50.0.52/32 [20/0] via 172.16.4.44, 5d00h
B        10.50.0.53/32 [20/0] via 172.16.4.44, 5d00h
B        10.50.0.54/32 [20/0] via 172.16.4.44, 5d00h
B        10.50.1.0/24 [20/0] via 172.16.4.44, 5d00h
B        10.50.2.0/24 [20/0] via 172.16.4.44, 5d00h
B        10.50.3.0/24 [20/0] via 172.16.4.44, 5d00h
B        10.50.4.0/24 [20/0] via 172.16.4.44, 5d00h
B        10.50.5.0/24 [20/0] via 172.16.4.44, 5d00h
B        10.50.6.0/24 [20/0] via 172.16.4.44, 5d00h
B        10.50.52.0/25 [20/0] via 172.16.4.44, 5d00h
B        10.50.52.128/25 [20/0] via 172.16.4.44, 5d00h
B        10.50.255.0/24 [20/0] via 172.16.4.44, 5d00h
B        10.60.0.0/16 [20/0] via 172.16.4.44, 2w1d
B        10.70.0.0/16 [20/0] via 172.16.4.44, 2w1d
      172.16.0.0/16 is variably subnetted, 13 subnets, 2 masks
B        172.16.1.0/24 [20/0] via 172.16.3.21, 2w0d
C        172.16.2.0/24 is directly connected, Ethernet0/0
L        172.16.2.31/32 is directly connected, Ethernet0/0
C        172.16.3.0/24 is directly connected, Ethernet0/1
L        172.16.3.31/32 is directly connected, Ethernet0/1
C        172.16.4.0/24 is directly connected, Ethernet0/2
L        172.16.4.31/32 is directly connected, Ethernet0/2
B        172.16.5.0/24 [20/0] via 172.16.4.44, 2w0d
B        172.16.6.0/24 [20/0] via 172.16.4.44, 2w1d
B        172.16.7.0/24 [20/0] via 172.16.4.44, 2w1d
B        172.16.8.0/24 [20/0] via 172.16.4.44, 2w1d
B        172.16.9.0/24 [20/0] via 172.16.4.44, 2w1d
B        172.16.10.0/24 [20/0] via 172.16.4.44, 2w1d
R31#
R31#
R31#
R31#
R31#show ip bgp
BGP table version is 740, local router ID is 10.30.0.31
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
 *>  10.10.13.0/24    172.16.2.14                            0 1001 i
 *                    172.16.3.21                            0 101 1001 i
 *>  10.10.14.0/24    172.16.2.14              0             0 1001 i
 *                    172.16.3.21                            0 101 1001 i
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
 *>  10.50.0.51/32    172.16.4.44                            0 520 2042 ?
 *                    172.16.3.21                            0 101 520 2042 ?
 *>  10.50.0.52/32    172.16.4.44                            0 520 2042 ?
 *                    172.16.3.21                            0 101 520 2042 ?
 *>  10.50.0.53/32    172.16.4.44                            0 520 2042 ?
 *                    172.16.3.21                            0 101 520 2042 ?
 *>  10.50.0.54/32    172.16.4.44                            0 520 2042 ?
 *                    172.16.3.21                            0 101 520 2042 ?
 *>  10.50.1.0/24     172.16.4.44                            0 520 2042 ?
 *                    172.16.3.21                            0 101 520 2042 ?
 *>  10.50.2.0/24     172.16.4.44                            0 520 2042 ?
 *                    172.16.3.21                            0 101 520 2042 ?
 *>  10.50.3.0/24     172.16.4.44                            0 520 2042 ?
 *                    172.16.3.21                            0 101 520 2042 ?
 *>  10.50.4.0/24     172.16.4.44                            0 520 2042 ?
 *                    172.16.3.21                            0 101 520 2042 ?
 *>  10.50.5.0/24     172.16.4.44                            0 520 2042 ?
 *                    172.16.3.21                            0 101 520 2042 ?
 *>  10.50.6.0/24     172.16.4.44                            0 520 2042 ?
 *                    172.16.3.21                            0 101 520 2042 ?
 *>  10.50.52.0/25    172.16.4.44                            0 520 2042 i
 *                    172.16.3.21                            0 101 520 2042 i
 *>  10.50.52.128/25  172.16.4.44                            0 520 2042 i
 *                    172.16.3.21                            0 101 520 2042 i
 *>  10.50.255.0/24   172.16.4.44                            0 520 2042 ?
 *                    172.16.3.21                            0 101 520 2042 ?
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
BGP table version is 740, main routing table version 740
54 network entries using 7560 bytes of memory
104 path entries using 8320 bytes of memory
28/20 BGP path/bestpath attribute entries using 4032 bytes of memory
8 BGP AS-PATH entries using 192 bytes of memory
0 BGP route-map cache entries using 0 bytes of memory
0 BGP filter-list cache entries using 0 bytes of memory
BGP using 20104 total bytes of memory
BGP activity 102/48 prefixes, 943/839 paths, scan interval 60 secs

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
172.16.2.14     4         1001    1759    1746      740    0    0 1d02h          20
172.16.3.21     4          101   23799   23820      740    0    0 2w0d           50
172.16.4.44     4          520   24081   24099      740    0    0 2w1d           29


```
</details>

### R52

<details>
  <summary>Конфигурация</summary>

```

R52#terminal length 0
R52#sh run
Building configuration...

Current configuration : 2694 bytes
!
! Last configuration change at 18:04:24 UTC Wed Feb 5 2025
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
 ip address 10.50.52.126 255.255.255.128
 ip nat enable
!
interface Loopback3
 ip address 10.50.52.254 255.255.255.128
 ip nat enable
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
 network 172.16.6.0 mask 255.255.255.0
 network 172.16.7.0 mask 255.255.255.0
 redistribute eigrp 1
 neighbor 172.16.6.44 remote-as 520
 neighbor 172.16.6.44 prefix-list MSK_NET_WO_NAT1 out
 neighbor 172.16.7.43 remote-as 520
 neighbor 172.16.7.43 prefix-list MSK_NET_WO_NAT2 out
 maximum-paths 2
!
ip forward-protocol nd
!
!
no ip http server
no ip http secure-server
ip nat pool PUBLIC_POOL_G0/2 10.50.52.1 10.50.52.5 netmask 255.255.255.128
ip nat pool PUBLIC_POOL_G0/3 10.50.52.129 10.50.52.133 netmask 255.255.255.128
ip nat source route-map NAT_FOR_G0/2 pool PUBLIC_POOL_G0/2 overload
ip nat source route-map NAT_FOR_G0/3 pool PUBLIC_POOL_G0/3 overload
!
!
ip prefix-list MSK_NET_WO_NAT1 seq 4 deny 10.50.52.128/25
ip prefix-list MSK_NET_WO_NAT1 seq 5 permit 10.50.0.0/16 le 32
!
ip prefix-list MSK_NET_WO_NAT2 seq 4 deny 10.50.52.0/25
ip prefix-list MSK_NET_WO_NAT2 seq 5 permit 10.50.0.0/16 le 32
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

      10.0.0.0/8 is variably subnetted, 47 subnets, 4 masks
B        10.10.0.11/32 [20/0] via 172.16.7.43, 1d02h
B        10.10.0.12/32 [20/0] via 172.16.7.43, 1d02h
B        10.10.0.13/32 [20/0] via 172.16.7.43, 1d02h
B        10.10.0.14/32 [20/0] via 172.16.7.43, 1d02h
B        10.10.0.15/32 [20/0] via 172.16.7.43, 1d02h
B        10.10.0.16/32 [20/0] via 172.16.7.43, 1d02h
B        10.10.1.0/24 [20/0] via 172.16.7.43, 1d02h
B        10.10.2.0/24 [20/0] via 172.16.7.43, 1d02h
B        10.10.3.0/24 [20/0] via 172.16.7.43, 1d02h
B        10.10.4.0/24 [20/0] via 172.16.7.43, 1d02h
B        10.10.5.0/24 [20/0] via 172.16.7.43, 1d02h
B        10.10.6.0/24 [20/0] via 172.16.7.43, 1d02h
B        10.10.7.0/24 [20/0] via 172.16.7.43, 1d02h
B        10.10.8.0/24 [20/0] via 172.16.7.43, 1d02h
B        10.10.9.0/24 [20/0] via 172.16.7.43, 1d02h
B        10.10.13.0/24 [20/0] via 172.16.7.43, 1d02h
B        10.10.14.0/24 [20/0] via 172.16.6.44, 1d02h
B        10.10.255.0/24 [20/0] via 172.16.7.43, 1d02h
B        10.20.0.0/16 [20/0] via 172.16.7.43, 5d00h
                      [20/0] via 172.16.6.44, 5d00h
B        10.30.0.0/16 [20/0] via 172.16.7.43, 5d00h
                      [20/0] via 172.16.6.44, 5d00h
B        10.40.0.41/32 [20/20] via 172.16.6.44, 5d00h
B        10.40.0.42/32 [20/20] via 172.16.7.43, 5d00h
B        10.40.0.43/32 [20/0] via 172.16.7.43, 5d00h
B        10.40.0.44/32 [20/20] via 172.16.7.43, 5d00h
B        10.40.1.0/24 [20/20] via 172.16.7.43, 5d00h
B        10.40.2.0/24 [20/20] via 172.16.7.43, 5d00h
                      [20/20] via 172.16.6.44, 5d00h
B        10.40.3.0/24 [20/0] via 172.16.7.43, 5d00h
B        10.40.4.0/24 [20/0] via 172.16.7.43, 5d00h
D        10.50.0.51/32 [90/1024640] via 10.50.3.51, 2w2d, Ethernet0/1
C        10.50.0.52/32 is directly connected, Loopback1
D        10.50.0.53/32 [90/1024640] via 10.50.4.53, 2w2d, Ethernet0/0
D        10.50.0.54/32 [90/1536640] via 10.50.4.53, 2w2d, Ethernet0/0
D        10.50.1.0/24 [90/1536000] via 10.50.4.53, 2w2d, Ethernet0/0
                      [90/1536000] via 10.50.3.51, 2w2d, Ethernet0/1
D        10.50.2.0/24 [90/1536000] via 10.50.4.53, 2w2d, Ethernet0/0
                      [90/1536000] via 10.50.3.51, 2w2d, Ethernet0/1
C        10.50.3.0/24 is directly connected, Ethernet0/1
L        10.50.3.52/32 is directly connected, Ethernet0/1
C        10.50.4.0/24 is directly connected, Ethernet0/0
L        10.50.4.52/32 is directly connected, Ethernet0/0
D        10.50.5.0/24 [90/1536000] via 10.50.4.53, 2w2d, Ethernet0/0
D        10.50.6.0/24 [90/1536000] via 10.50.4.53, 2w2d, Ethernet0/0
                      [90/1536000] via 10.50.3.51, 2w2d, Ethernet0/1
C        10.50.52.0/25 is directly connected, Loopback2
L        10.50.52.126/32 is directly connected, Loopback2
C        10.50.52.128/25 is directly connected, Loopback3
L        10.50.52.254/32 is directly connected, Loopback3
D        10.50.255.0/24 [90/1536000] via 10.50.4.53, 2w2d, Ethernet0/0
                        [90/1536000] via 10.50.3.51, 2w2d, Ethernet0/1
B        10.60.0.0/16 [20/0] via 172.16.7.43, 5d00h
                      [20/0] via 172.16.6.44, 5d00h
B        10.70.0.0/16 [20/0] via 172.16.7.43, 5d00h
                      [20/0] via 172.16.6.44, 5d00h
      172.16.0.0/16 is variably subnetted, 12 subnets, 2 masks
B        172.16.1.0/24 [20/0] via 172.16.7.43, 5d00h
                       [20/0] via 172.16.6.44, 5d00h
B        172.16.2.0/24 [20/0] via 172.16.7.43, 5d00h
                       [20/0] via 172.16.6.44, 5d00h
B        172.16.3.0/24 [20/0] via 172.16.7.43, 5d00h
B        172.16.4.0/24 [20/0] via 172.16.7.43, 5d00h
                       [20/0] via 172.16.6.44, 5d00h
B        172.16.5.0/24 [20/0] via 172.16.7.43, 5d00h
                       [20/0] via 172.16.6.44, 5d00h
C        172.16.6.0/24 is directly connected, Ethernet0/2
L        172.16.6.52/32 is directly connected, Ethernet0/2
C        172.16.7.0/24 is directly connected, Ethernet0/3
L        172.16.7.52/32 is directly connected, Ethernet0/3
B        172.16.8.0/24 [20/0] via 172.16.7.43, 5d00h
                       [20/0] via 172.16.6.44, 5d00h
B        172.16.9.0/24 [20/0] via 172.16.7.43, 5d00h
                       [20/0] via 172.16.6.44, 5d00h
B        172.16.10.0/24 [20/0] via 172.16.7.43, 5d00h
                        [20/0] via 172.16.6.44, 5d00h
R52#
R52#
R52#
R52#
R52#show ip bgp
BGP table version is 377, local router ID is 10.50.52.254
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
 *   10.10.13.0/24    172.16.6.44                            0 520 301 1001 i
 *>                   172.16.7.43                            0 520 101 1001 i
 *   10.10.14.0/24    172.16.7.43                            0 520 101 1001 i
 *>                   172.16.6.44                            0 520 301 1001 i
 *   10.10.255.0/24   172.16.6.44                            0 520 301 1001 ?
 *>                   172.16.7.43                            0 520 101 1001 ?
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
 *m  10.60.0.0/16     172.16.6.44                            0 520 ?
 *>                   172.16.7.43                            0 520 ?
 *m  10.70.0.0/16     172.16.6.44                            0 520 ?
 *>                   172.16.7.43              0             0 520 ?
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
 *>  172.16.6.0/24    0.0.0.0                  0         32768 i
 *                    172.16.6.44              0             0 520 i
 *                    172.16.7.43                            0 520 i
 *>  172.16.7.0/24    0.0.0.0                  0         32768 i
 *                    172.16.6.44                            0 520 i
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
BGP router identifier 10.50.52.254, local AS number 2042
BGP table version is 377, main routing table version 377
53 network entries using 7420 bytes of memory
92 path entries using 7360 bytes of memory
12 multipath network entries and 24 multipath paths
19/17 BGP path/bestpath attribute entries using 2736 bytes of memory
5 BGP AS-PATH entries using 120 bytes of memory
0 BGP route-map cache entries using 0 bytes of memory
0 BGP filter-list cache entries using 0 bytes of memory
BGP using 17636 total bytes of memory
BGP activity 208/155 prefixes, 568/476 paths, scan interval 60 secs

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
172.16.6.44     4          520    8022    7983      377    0    0 5d00h          37
172.16.7.43     4          520    8009    7988      377    0    0 5d00h          40


```
</details>

### R71

<details>
  <summary>Конфигурация</summary>

```

R71#terminal length 0
R71#sh run
Building configuration...

Current configuration : 1942 bytes
!
! Last configuration change at 17:52:50 UTC Mon Feb 10 2025
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
 permit icmp host 10.70.0.71 host 10.40.0.43
!
ip sla 1
 icmp-echo 10.40.0.43 source-interface Loopback1
 tos 46
 threshold 1000
 timeout 1000
 frequency 1
ip sla schedule 1 life forever start-time now
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
      10.0.0.0/8 is variably subnetted, 5 subnets, 2 masks
C        10.70.0.71/32 is directly connected, Loopback1
C        10.70.1.0/24 is directly connected, Ethernet0/2.10
L        10.70.1.1/32 is directly connected, Ethernet0/2.10
C        10.70.255.0/24 is directly connected, Ethernet0/2.1000
L        10.70.255.1/32 is directly connected, Ethernet0/2.1000
      172.16.0.0/16 is variably subnetted, 4 subnets, 2 masks
C        172.16.8.0/24 is directly connected, Ethernet0/0
L        172.16.8.71/32 is directly connected, Ethernet0/0
C        172.16.10.0/24 is directly connected, Ethernet0/1
L        172.16.10.71/32 is directly connected, Ethernet0/1
R71#
R71#
R71#
R71#

```
</details>