# Планирование адресного пространства 

## Задание

В офисе Москва нужно настроить:
1. Сетевую IPv4 адресацию на всех устройствах.
1. На коммутаторах SW13 и SW14 между соответствующими интерфейсами протокол LACP.
1. На коммутаторах SW11-SW14 протокол PVRSTP.
1. На маршрутизаторах R13 и R14 маршрут по умолчанию через R21 и R31 соответственно.
1. На маршрутизаторах R11 и R12 протокол VRRP на саб интерфейсах для VLAN 10, 20, 1000.
1. Протокол OSPF:
- Маршрутизаторы R13-R14 должны находиться в зоне 0 - backbone.
- Маршрутизаторы R11-R12 должны находиться в зоне 10. Дополнительно к маршрутам должны получать маршрут по умолчанию.
- Маршрутизатор R15 должен находиться в зоне 101 и получать только маршрут по умолчанию.
- Маршрутизатор R16 должн находиться в зоне 102 и получать все маршруты, кроме маршрутов до сетей зоны 101.
- Настройки для IPv6 (если используется) повторяет логику IPv4.

## Топология

![a](media/lab06_1.PNG)

[Схема для импорта в PNETlab](media/otus_cource_lab4_net_planning_pnetlab_export-20241116-183936)

## Версии ПО

- PNETlab - 5.3.11
- Роутеры - Cisco IOS Software, Linux Software (I86BI_LINUX-ADVENTERPRISEK9-M), Version 15.4(2)T4
- Коммутаторы - Cisco IOS Software, Linux Software (I86BI_LINUXL2-ADVENTERPRISEK9-M), Version 15.2(CML_NIGHTLY_20150703)
- ПК - VPC

## Решение
### Настройка сетевой IPv4 адресации на всех устройствах
Настройка сетевой IPv4 адресации на всех устройствах осуществляется в соответствии с разработанным ранее планом адресации ([см. тут](https://github.com/skydevil56/otus_network_engineer_professional/tree/main/labs/lab04)).

### Настройка на коммутаторах SW13 и SW14 между соответствующими интерфейсами протокола LACP
Протокол LACP настраивается на интерфейсах e0/2 и e0/3 (подробные настройки представлены в конфигурациях ниже).

### Настройка на коммутаторах SW11-SW14 протокола PVRSTP
В качестве корневого коммутатора (priority 0) для vlan 10,20,1000 будет назначен SW13, следующим по приоритету (priority 4096) будет назначен SW14 (подробные настройки представлены в конфигурациях ниже).

### Настройка на маршрутизаторах R13 и R14 маршрута по умолчанию через R21 и R31 соответственно
Пока между автономными системами AS1001 и AS101, AS1001 и AS201 не настроен eBGP на маршрутизаторах R13 и R14 будет задан  статический маршрут по умолчанию через R21 и R31 соответственно (подробные настройки представлены в конфигурациях ниже).

### Настройка на маршрутизаторах R11 и R12 протокола VRRP на саб интерфейсах для VLAN 10, 20, 1000
Маршрутизаторы R11 и R12 будут иметь три VRRP инстанса, под одному на каждый VLAN (саб интерфейс).
VRRP инстансы на R11 будут иметь приоритет 150 (чем больше тем выше приоритет), VRRP инстансы на R12 будут иметь приоритет 50.
VRRP IP (кластерный адрес) для VLAN 10, 20, 1000 соответственно 10.10.1.1, 10.10.2.1, 10.10.255.1 (подробные настройки представлены в конфигурациях ниже).

### Настройка протокола OSPF
1. Интерфейсы e0/1, e0/1 маршрутизаторов R13 и R14 будут в зоне 0 (backbone), интерфейсы e0/2, e0/3 маршрутизаторов R11 и R12 будут в зоне 0.
1. Интерфейсы e0/1, e0/1 маршрутизаторов R11 и R12 будут в зоне 10. Дополнительно к маршрутам от других зон будут получать маршрут по умолчанию (default-information originate на R13 и R14).
1. Интерфейс e0/0 маршрутизатора R15 будет находиться в зоне 101 и сам маршрутизатор будет получать только маршрут по умолчанию (default-information originate на R13 и настройка области 101 как stab R15 и totally stab на R13).
1. Интерфейс e0/0 маршрутизатора R16 будет находиться в зоне 102 и cам маршрутизатор будет получать маршрут по умолчанию и все маршруты других зон, кроме маршрутов до сетей зоны 101 (default-information originate и настройка фильтрации на R14).

## Конфигурации устройств

### R11

<details>
  <summary>Конфигурация</summary>

```




```
</details>

### R12

<details>
  <summary>Конфигурация</summary>

```




```
</details>

### R13

<details>
  <summary>Конфигурация</summary>

```




```
</details>

### R14

<details>
  <summary>Конфигурация</summary>

```




```
</details>

### R15

<details>
  <summary>Конфигурация</summary>

```




```
</details>

### R16

<details>
  <summary>Конфигурация</summary>

```




```
</details>

### SW11

<details>
  <summary>Конфигурация</summary>

```

SW11#sh run
Building configuration...

Current configuration : 1252 bytes
!
! Last configuration change at 10:43:54 UTC Sun Dec 1 2024
!
version 15.2
service timestamps debug datetime msec
service timestamps log datetime msec
no service password-encryption
service compress-config
!
hostname SW11
!
boot-start-marker
boot-end-marker
!
!
!
no aaa new-model
!
!
!
!
!
no ip icmp rate-limit unreachable
!
!
!
no ip domain-lookup
ip cef
no ipv6 cef
!
!
spanning-tree mode pvst
spanning-tree extend system-id
!
vlan internal allocation policy ascending
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
interface Ethernet0/0
 switchport trunk encapsulation dot1q
 switchport mode trunk
!
interface Ethernet0/1
 switchport trunk encapsulation dot1q
 switchport mode trunk
!
interface Ethernet0/2
 switchport access vlan 10
 switchport mode access
!
interface Ethernet0/3
!
interface Ethernet1/0
!
interface Ethernet1/1
!
interface Ethernet1/2
!
interface Ethernet1/3
!
interface Vlan1000
 ip address 10.10.255.11 255.255.255.0
!
ip forward-protocol nd
!
no ip http server
no ip http secure-server
!
ip route 0.0.0.0 0.0.0.0 10.10.255.1
!
!
!
!
!
control-plane
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
!
!
end

SW11#
SW11#
SW11#
SW11#show vlan

VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    Et0/3, Et1/0, Et1/1, Et1/2
                                                Et1/3
10   VLAN0010                         active    Et0/2
20   VLAN0020                         active
1000 MGMT                             active
1002 fddi-default                     act/unsup
1003 token-ring-default               act/unsup
1004 fddinet-default                  act/unsup
1005 trnet-default                    act/unsup

VLAN Type  SAID       MTU   Parent RingNo BridgeNo Stp  BrdgMode Trans1 Trans2
---- ----- ---------- ----- ------ ------ -------- ---- -------- ------ ------
1    enet  100001     1500  -      -      -        -    -        0      0
10   enet  100010     1500  -      -      -        -    -        0      0
20   enet  100020     1500  -      -      -        -    -        0      0
1000 enet  101000     1500  -      -      -        -    -        0      0
1002 fddi  101002     1500  -      -      -        -    -        0      0
1003 tr    101003     1500  -      -      -        -    -        0      0
1004 fdnet 101004     1500  -      -      -        ieee -        0      0
1005 trnet 101005     1500  -      -      -        ibm  -        0      0

Primary Secondary Type              Ports
------- --------- ----------------- ------------------------------------------

SW11#
SW11#
SW11#
SW11#show spanning-tree vlan 10,20,1000

VLAN0010
  Spanning tree enabled protocol ieee
  Root ID    Priority    10
             Address     aabb.cc00.0300
             Cost        100
             Port        1 (Ethernet0/0)
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32778  (priority 32768 sys-id-ext 10)
             Address     aabb.cc00.0200
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  300 sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Et0/0               Root FWD 100       128.1    Shr
Et0/1               Altn BLK 100       128.2    Shr
Et0/2               Desg FWD 100       128.3    Shr


VLAN0020
  Spanning tree enabled protocol ieee
  Root ID    Priority    20
             Address     aabb.cc00.0300
             Cost        100
             Port        1 (Ethernet0/0)
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32788  (priority 32768 sys-id-ext 20)
             Address     aabb.cc00.0200
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  300 sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Et0/0               Root FWD 100       128.1    Shr
Et0/1               Altn BLK 100       128.2    Shr


VLAN1000
  Spanning tree enabled protocol ieee
  Root ID    Priority    1000
             Address     aabb.cc00.0300
             Cost        100
             Port        1 (Ethernet0/0)
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    33768  (priority 32768 sys-id-ext 1000)
             Address     aabb.cc00.0200
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  300 sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Et0/0               Root FWD 100       128.1    Shr
Et0/1               Altn BLK 100       128.2    Shr

```
</details>

### SW12

<details>
  <summary>Конфигурация</summary>

```

SW12#sh run
Building configuration...

Current configuration : 1092 bytes
!
! Last configuration change at 10:43:41 UTC Sun Dec 1 2024
!
version 15.2
service timestamps debug datetime msec
service timestamps log datetime msec
no service password-encryption
service compress-config
!
hostname SW12
!
boot-start-marker
boot-end-marker
!
!
!
no aaa new-model
!
!
!
!
!
!
!
!
no ip domain-lookup
ip cef
no ipv6 cef
!
!
spanning-tree mode pvst
spanning-tree extend system-id
!
vlan internal allocation policy ascending
!
!
!
!
!
!
!
!
!
!
!
!
!
interface Ethernet0/0
 switchport trunk encapsulation dot1q
 switchport mode trunk
!
interface Ethernet0/1
 switchport trunk encapsulation dot1q
 switchport mode trunk
!
interface Ethernet0/2
 switchport access vlan 20
 switchport mode access
!
interface Ethernet0/3
!
interface Ethernet1/0
!
interface Ethernet1/1
!
interface Ethernet1/2
!
interface Ethernet1/3
!
interface Vlan1000
 ip address 10.10.255.12 255.255.255.0
!
ip forward-protocol nd
!
no ip http server
no ip http secure-server
!
ip route 0.0.0.0 0.0.0.0 10.10.255.1
!
!
!
!
!
control-plane
!
!
line con 0
 logging synchronous
line aux 0
line vty 0 4
!
!
end

SW12#
SW12#
SW12#
SW12#
SW12#
SW12#show vlan

VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    Et0/3, Et1/0, Et1/1, Et1/2
                                                Et1/3
10   VLAN0010                         active
20   VLAN0020                         active    Et0/2
1000 MGMT                             active
1002 fddi-default                     act/unsup
1003 token-ring-default               act/unsup
1004 fddinet-default                  act/unsup
1005 trnet-default                    act/unsup

VLAN Type  SAID       MTU   Parent RingNo BridgeNo Stp  BrdgMode Trans1 Trans2
---- ----- ---------- ----- ------ ------ -------- ---- -------- ------ ------
1    enet  100001     1500  -      -      -        -    -        0      0
10   enet  100010     1500  -      -      -        -    -        0      0
20   enet  100020     1500  -      -      -        -    -        0      0
1000 enet  101000     1500  -      -      -        -    -        0      0
1002 fddi  101002     1500  -      -      -        -    -        0      0
1003 tr    101003     1500  -      -      -        -    -        0      0
1004 fdnet 101004     1500  -      -      -        ieee -        0      0
1005 trnet 101005     1500  -      -      -        ibm  -        0      0

Primary Secondary Type              Ports
------- --------- ----------------- ------------------------------------------
SW12#
SW12#
SW12#
SW12#
SW12#show spanning-tree vlan 10,20,1000

VLAN0010
  Spanning tree enabled protocol ieee
  Root ID    Priority    10
             Address     aabb.cc00.0300
             Cost        100
             Port        2 (Ethernet0/1)
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32778  (priority 32768 sys-id-ext 10)
             Address     aabb.cc00.0100
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  300 sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Et0/0               Altn BLK 100       128.1    Shr
Et0/1               Root FWD 100       128.2    Shr


VLAN0020
  Spanning tree enabled protocol ieee
  Root ID    Priority    20
             Address     aabb.cc00.0300
             Cost        100
             Port        2 (Ethernet0/1)
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32788  (priority 32768 sys-id-ext 20)
             Address     aabb.cc00.0100
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  300 sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Et0/0               Altn BLK 100       128.1    Shr
Et0/1               Root FWD 100       128.2    Shr
Et0/2               Desg FWD 100       128.3    Shr


VLAN1000
  Spanning tree enabled protocol ieee
  Root ID    Priority    1000
             Address     aabb.cc00.0300
             Cost        100
             Port        2 (Ethernet0/1)
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    33768  (priority 32768 sys-id-ext 1000)
             Address     aabb.cc00.0100
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  300 sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Et0/0               Altn BLK 100       128.1    Shr
Et0/1               Root FWD 100       128.2    Shr

```
</details>

### SW13

<details>
  <summary>Конфигурация</summary>

```

SW13#sh run
Building configuration...

Current configuration : 1487 bytes
!
! Last configuration change at 15:18:42 UTC Sun Nov 24 2024
!
version 15.2
service timestamps debug datetime msec
service timestamps log datetime msec
no service password-encryption
service compress-config
!
hostname SW13
!
boot-start-marker
boot-end-marker
!
!
!
no aaa new-model
!
!
!
!
!
!
!
!
no ip domain-lookup
ip cef
no ipv6 cef
!
!
!
spanning-tree mode pvst
spanning-tree extend system-id
spanning-tree vlan 10,20,1000 priority 0
!
vlan internal allocation policy ascending
!
!
!
!
!
!
!
!
!
!
!
!
!
interface Port-channel1
 switchport trunk encapsulation dot1q
 switchport mode trunk
!
interface Ethernet0/0
 switchport trunk encapsulation dot1q
 switchport mode trunk
!
interface Ethernet0/1
 switchport trunk encapsulation dot1q
 switchport mode trunk
 duplex auto
!
interface Ethernet0/2
 switchport trunk encapsulation dot1q
 switchport mode trunk
 channel-group 1 mode active
!
interface Ethernet0/3
 switchport trunk encapsulation dot1q
 switchport mode trunk
 channel-group 1 mode active
!
interface Ethernet1/0
 switchport trunk encapsulation dot1q
 switchport mode trunk
!
interface Ethernet1/1
 switchport trunk encapsulation dot1q
 switchport mode trunk
!
interface Ethernet1/2
!
interface Ethernet1/3
!
interface Vlan1000
 ip address 10.10.255.13 255.255.255.0
!
ip forward-protocol nd
!
no ip http server
no ip http secure-server
!
ip route 0.0.0.0 0.0.0.0 10.10.255.1
!
!
!
!
!
control-plane
!
!
line con 0
 logging synchronous
line aux 0
line vty 0 4
!
!
end

SW13#
SW13#
SW13#
SW13#
SW13#
SW13#show vlan

VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    Et1/2, Et1/3
10   VLAN0010                         active
20   VLAN0020                         active
1000 MGMT                             active
1002 fddi-default                     act/unsup
1003 token-ring-default               act/unsup
1004 fddinet-default                  act/unsup
1005 trnet-default                    act/unsup

VLAN Type  SAID       MTU   Parent RingNo BridgeNo Stp  BrdgMode Trans1 Trans2
---- ----- ---------- ----- ------ ------ -------- ---- -------- ------ ------
1    enet  100001     1500  -      -      -        -    -        0      0
10   enet  100010     1500  -      -      -        -    -        0      0
20   enet  100020     1500  -      -      -        -    -        0      0
1000 enet  101000     1500  -      -      -        -    -        0      0
1002 fddi  101002     1500  -      -      -        -    -        0      0
1003 tr    101003     1500  -      -      -        -    -        0      0
1004 fdnet 101004     1500  -      -      -        ieee -        0      0
1005 trnet 101005     1500  -      -      -        ibm  -        0      0

Primary Secondary Type              Ports
------- --------- ----------------- ------------------------------------------
SW13#
SW13#
SW13#
SW13#
SW13#show spanning-tree vlan 10,20,1000

VLAN0010
  Spanning tree enabled protocol ieee
  Root ID    Priority    10
             Address     aabb.cc00.0300
             This bridge is the root
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    10     (priority 0 sys-id-ext 10)
             Address     aabb.cc00.0300
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  300 sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Et0/0               Desg FWD 100       128.1    Shr
Et0/1               Desg FWD 100       128.2    Shr
Et1/0               Desg FWD 100       128.5    Shr
Et1/1               Desg FWD 100       128.6    Shr
Po1                 Desg FWD 56        128.65   Shr


VLAN0020
  Spanning tree enabled protocol ieee
  Root ID    Priority    20
             Address     aabb.cc00.0300
             This bridge is the root
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    20     (priority 0 sys-id-ext 20)
             Address     aabb.cc00.0300
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  300 sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Et0/0               Desg FWD 100       128.1    Shr
Et0/1               Desg FWD 100       128.2    Shr
Et1/0               Desg FWD 100       128.5    Shr
Et1/1               Desg FWD 100       128.6    Shr
Po1                 Desg FWD 56        128.65   Shr


VLAN1000
  Spanning tree enabled protocol ieee
  Root ID    Priority    1000
             Address     aabb.cc00.0300
             This bridge is the root
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    1000   (priority 0 sys-id-ext 1000)
             Address     aabb.cc00.0300
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  300 sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Et0/0               Desg FWD 100       128.1    Shr
Et0/1               Desg FWD 100       128.2    Shr
Et1/0               Desg FWD 100       128.5    Shr
Et1/1               Desg FWD 100       128.6    Shr
Po1                 Desg FWD 56        128.65   Shr

```
</details>

### SW14

<details>
  <summary>Конфигурация</summary>

```

SW14#sh run
Building configuration...

Current configuration : 1457 bytes
!
! Last configuration change at 15:19:01 UTC Sun Nov 24 2024
!
version 15.2
service timestamps debug datetime msec
service timestamps log datetime msec
no service password-encryption
service compress-config
!
hostname SW14
!
boot-start-marker
boot-end-marker
!
!
!
no aaa new-model
!
!
!
!
!
!
!
!
ip cef
no ipv6 cef
!
!
!
spanning-tree mode pvst
spanning-tree extend system-id
spanning-tree vlan 10,20,1000 priority 4096
!
vlan internal allocation policy ascending
!
!
!
!
!
!
!
!
!
!
!
!
!
interface Port-channel1
 switchport trunk encapsulation dot1q
 switchport mode trunk
!
interface Ethernet0/0
 switchport trunk encapsulation dot1q
 switchport mode trunk
!
interface Ethernet0/1
 switchport trunk encapsulation dot1q
 switchport mode trunk
!
interface Ethernet0/2
 switchport trunk encapsulation dot1q
 switchport mode trunk
 channel-group 1 mode active
!
interface Ethernet0/3
 switchport trunk encapsulation dot1q
 switchport mode trunk
 channel-group 1 mode active
!
interface Ethernet1/0
 switchport trunk encapsulation dot1q
 switchport mode trunk
!
interface Ethernet1/1
 switchport trunk encapsulation dot1q
 switchport mode trunk
!
interface Ethernet1/2
!
interface Ethernet1/3
!
interface Vlan1000
 ip address 10.10.255.14 255.255.255.0
!
ip forward-protocol nd
!
no ip http server
no ip http secure-server
!
ip route 0.0.0.0 0.0.0.0 10.10.255.1
!
!
!
!
!
control-plane
!
!
line con 0
 logging synchronous
line aux 0
line vty 0 4
!
!
end

SW14#
SW14#
SW14#
SW14#show vlan

VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    Et1/2, Et1/3
10   VLAN0010                         active
20   VLAN0020                         active
1000 MGMT                             active
1002 fddi-default                     act/unsup
1003 token-ring-default               act/unsup
1004 fddinet-default                  act/unsup
1005 trnet-default                    act/unsup

VLAN Type  SAID       MTU   Parent RingNo BridgeNo Stp  BrdgMode Trans1 Trans2
---- ----- ---------- ----- ------ ------ -------- ---- -------- ------ ------
1    enet  100001     1500  -      -      -        -    -        0      0
10   enet  100010     1500  -      -      -        -    -        0      0
20   enet  100020     1500  -      -      -        -    -        0      0
1000 enet  101000     1500  -      -      -        -    -        0      0
1002 fddi  101002     1500  -      -      -        -    -        0      0
1003 tr    101003     1500  -      -      -        -    -        0      0
1004 fdnet 101004     1500  -      -      -        ieee -        0      0
1005 trnet 101005     1500  -      -      -        ibm  -        0      0

Primary Secondary Type              Ports
------- --------- ----------------- ------------------------------------------
SW14#show spanning-tree vlan 10,20,1000

VLAN0010
  Spanning tree enabled protocol ieee
  Root ID    Priority    10
             Address     aabb.cc00.0300
             Cost        56
             Port        65 (Port-channel1)
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    4106   (priority 4096 sys-id-ext 10)
             Address     aabb.cc00.0400
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  300 sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Et0/0               Desg FWD 100       128.1    Shr
Et0/1               Desg FWD 100       128.2    Shr
Et1/0               Desg FWD 100       128.5    Shr
Et1/1               Desg FWD 100       128.6    Shr
Po1                 Root FWD 56        128.65   Shr


VLAN0020
  Spanning tree enabled protocol ieee
  Root ID    Priority    20
             Address     aabb.cc00.0300
             Cost        56
             Port        65 (Port-channel1)
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    4116   (priority 4096 sys-id-ext 20)
             Address     aabb.cc00.0400
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  15  sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Et0/0               Desg FWD 100       128.1    Shr
Et0/1               Desg FWD 100       128.2    Shr
Et1/0               Desg FWD 100       128.5    Shr
Et1/1               Desg FWD 100       128.6    Shr
Po1                 Root FWD 56        128.65   Shr


VLAN1000
  Spanning tree enabled protocol ieee
  Root ID    Priority    1000
             Address     aabb.cc00.0300
             Cost        56
             Port        65 (Port-channel1)
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    5096   (priority 4096 sys-id-ext 1000)
             Address     aabb.cc00.0400
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  300 sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Et0/0               Desg FWD 100       128.1    Shr
Et0/1               Desg FWD 100       128.2    Shr
Et1/0               Desg FWD 100       128.5    Shr
Et1/1               Desg FWD 100       128.6    Shr
Po1                 Root FWD 56        128.65   Shr




```
</details>

### VPC11

<details>
  <summary>Конфигурация</summary>

```

VPC11> show

NAME   IP/MASK              GATEWAY                             GATEWAY
VPC11  10.10.1.11/24        10.10.1.1
       fe80::250:79ff:fe66:6807/64


```
</details>

### VPC12

<details>
  <summary>Конфигурация</summary>

```

VPC12> show

NAME   IP/MASK              GATEWAY                             GATEWAY
VPC12  10.10.2.12/24        10.10.2.1
       fe80::250:79ff:fe66:6808/64


```
</details>

