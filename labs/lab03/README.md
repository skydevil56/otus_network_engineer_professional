# Настройка DHCPv4/v6 

## Задание

[Ссылка на документ #1](media/lab03#1.docx)

[Ссылка на документ #2](media/lab03#2.docx)

## Топология

![a](media/lab03_1.PNG)


[Схема для импорта в PNETlab](media/otus_cource_lab3_dhcp_pnetlab_export-20241019-154224.zip)

## Версии ПО

- PNETlab - 5.3.11
- Роутеры - Cisco IOS Software [Dublin], Linux Software (X86_64BI_LINUX-ADVENTERPRISEK9-M), Version  17.12.1, RELEASE SOFTWARE (fc5)
- Коммутаторы - Cisco IOS Software [Dublin], Linux Software (X86_64BI_LINUX_L2-ADVENTERPRISEK9-M), Version 17.12.1, RELEASE SOFTWARE (fc5)
- ПК - Debian 12 x86_64

## Таблица №1. Адресация

Device | Interface	| IP Address | Subnet Mask	| Default Gateway
--- | --- | --- | --- | ---
R1 | e0/0 | 10.0.0.1 | 255.255.255.252	| N/A
R1 | e0/1 | N/A | N/A	| N/A
R1 | e0/1.100 | 192.168.1.1 | 255.255.255.192 (/26) | N/A
R1 | e0/1.200 | 192.168.1.65| 255.255.255.224 (/27) | N/A
R1 | e0/1.1000 |N/A	| N/A | N/A
R2 | e0/0	| 10.0.0.2	| 255.255.255.252 | N/A
R2 | e0/1	| 192.168.1.97 | 255.255.255.240 (/28) | N/A
S1 | VLAN 200 |	192.168.1.66 | 255.255.255.224 (/27) | 192.168.1.65 | 
S2 | VLAN 1	| 192.168.1.98 | 255.255.255.240 (/28)| 192.168.1.97
PC-A |	NIC | DHCP | DHCP | DHCP
PC-B |	NIC | DHCP | DHCP | DHCP

## Таблица №2. VLAN

VLAN | Name	| Interface Assigned
--- | --- | ---
1	| N/A	| S2: e0/1
100	| Clients | S1: e0/1
200	| Management| S1: VLAN 200 
999	| Parking |	S1: e0/2-3
1000 | Native | N/A

## Проверка IP связности коммутаторов в рамках VLAN1

### Проверим, что с коммутатора S1 доступны S2, S3:

```
S1#sh ip interface brief 
Interface              IP-Address      OK? Method Status                Protocol
Ethernet0/0            unassigned      YES unset  administratively down down    
Ethernet0/1            unassigned      YES unset  up                    up      
Ethernet0/2            unassigned      YES unset  up                    up      
Ethernet0/3            unassigned      YES unset  administratively down down    
Ethernet1/0            unassigned      YES unset  down                  down    
Ethernet1/1            unassigned      YES unset  down                  down    
Ethernet1/2            unassigned      YES unset  down                  down    
Ethernet1/3            unassigned      YES unset  down                  down    
Vlan1                  192.168.1.1     YES manual up                    up      

S1#show vlan brief 

VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    Et0/0, Et0/3, Et1/0, Et1/1
                                                Et1/2, Et1/3
1002 fddi-default                     act/unsup 
1003 token-ring-default               act/unsup 
1004 fddinet-default                  act/unsup 
1005 trnet-default                    act/unsup 

S1#ping 192.168.1.2
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.1.2, timeout is 2 seconds:
.!!!!
Success rate is 80 percent (4/5), round-trip min/avg/max = 1/1/1 ms

S1#ping 192.168.1.3
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.1.3, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/1 ms

```

### Проверим, что с коммутатора S3 доступен S2:

```
S3#show ip interface brief 
Interface              IP-Address      OK? Method Status                Protocol
Ethernet0/0            unassigned      YES unset  down                  down    
Ethernet0/1            unassigned      YES unset  down                  down    
Ethernet0/2            unassigned      YES unset  up                    up      
Ethernet0/3            unassigned      YES unset  administratively down down    
Ethernet1/0            unassigned      YES unset  administratively down down    
Ethernet1/1            unassigned      YES unset  up                    up      
Ethernet1/2            unassigned      YES unset  down                  down    
Ethernet1/3            unassigned      YES unset  down                  down    
Vlan1                  192.168.1.3     YES manual up                    up      

S3#show vlan brief 

VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    Et0/0, Et0/1, Et0/3, Et1/0
                                                Et1/2, Et1/3
1002 fddi-default                     act/unsup 
1003 token-ring-default               act/unsup 
1004 fddinet-default                  act/unsup 
1005 trnet-default                    act/unsup

S3#ping 192.168.1.2
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.1.2, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/1 ms

```

## Выключим интерфейсы условного внутреннего контура (зеленый цвет на схеме) и выполним анализ вывода команды `show spanning-tree`

```
S1(config)#interface range ethernet 0/3, ethernet 0/0
S1(config-if-range)#shutdown 

S1#show ip interface brief 
Interface              IP-Address      OK? Method Status                Protocol
Ethernet0/0            unassigned      YES unset  administratively down down    
Ethernet0/1            unassigned      YES unset  up                    up      
Ethernet0/2            unassigned      YES unset  up                    up      
Ethernet0/3            unassigned      YES unset  administratively down down    
Ethernet1/0            unassigned      YES unset  down                  down    
Ethernet1/1            unassigned      YES unset  down                  down    
Ethernet1/2            unassigned      YES unset  down                  down    
Ethernet1/3            unassigned      YES unset  down                  down    
Vlan1                  192.168.1.1     YES manual up                    up      
```

```
S1#show spanning-tree 

VLAN0001
  Spanning tree enabled protocol rstp
  Root ID    Priority    1
             Address     aabb.cc00.0100
             This bridge is the root
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    1      (priority 0 sys-id-ext 1)
             Address     aabb.cc00.0100
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  300 sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Et0/1               Desg FWD 100       128.2    P2p 
Et0/2               Desg FWD 100       128.3    P2p 
```

```
S2(config)#interface range ethernet 0/3, ethernet 1/0
S2(config-if-range)#shutdown 

S2#show ip interface brief 
Interface              IP-Address      OK? Method Status                Protocol
Ethernet0/0            unassigned      YES unset  administratively down down    
Ethernet0/1            unassigned      YES unset  up                    up      
Ethernet0/2            unassigned      YES unset  down                  down    
Ethernet0/3            unassigned      YES unset  administratively down down    
Ethernet1/0            unassigned      YES unset  administratively down down    
Ethernet1/1            unassigned      YES unset  down                  down    
Ethernet1/2            unassigned      YES unset  administratively down down    
Ethernet1/3            unassigned      YES unset  up                    up      
Vlan1                  192.168.1.2     YES manual up                    up  
```

```
S2#show spanning-tree 

VLAN0001
  Spanning tree enabled protocol rstp
  Root ID    Priority    1
             Address     aabb.cc00.0100
             Cost        100
             Port        2 (Ethernet0/1)
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     aabb.cc00.0200
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  300 sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Et0/1               Root FWD 100       128.2    P2p 
Et1/3               Desg FWD 100       128.8    P2p 

```

```
S3(config)#interface range ethernet 0/0, ethernet 1/2
S3(config-if-range)#shutdown

S3#sh ip interface brief 
Interface              IP-Address      OK? Method Status                Protocol
Ethernet0/0            unassigned      YES unset  administratively down down    
Ethernet0/1            unassigned      YES unset  down                  down    
Ethernet0/2            unassigned      YES unset  up                    up      
Ethernet0/3            unassigned      YES unset  administratively down down    
Ethernet1/0            unassigned      YES unset  administratively down down    
Ethernet1/1            unassigned      YES unset  up                    up      
Ethernet1/2            unassigned      YES unset  administratively down down    
Ethernet1/3            unassigned      YES unset  down                  down    
Vlan1                  192.168.1.3     YES manual up                    up      
```

```
S3#show spanning-tree 

VLAN0001
  Spanning tree enabled protocol rstp
  Root ID    Priority    1
             Address     aabb.cc00.0100
             Cost        100
             Port        3 (Ethernet0/2)
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     aabb.cc00.0300
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  300 sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Et0/2               Root FWD 100       128.3    P2p 
Et1/1               Altn BLK 100       128.6    P2p 
```

Видно, что root коммутатор это `S1`, потому что при одинаковом приоритете `32769` на всех коммутаторах у него минимальное значение MAC адреса `aabb.cc00.0100`

## При помощи изменения стоимости (`cost`) до корневого коммутатора на порту `e0/1` коммутатора `S2` изменим роль порта `e1/3` с Designated на Alternative

```
S2#show spanning-tree 

VLAN0001
  Spanning tree enabled protocol rstp
  Root ID    Priority    1
             Address     aabb.cc00.0100
             Cost        101
             Port        2 (Ethernet0/1)
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     aabb.cc00.0200
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  300 sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Et0/1               Root FWD 101       128.2    P2p 
Et1/3               Altn BLK 100       128.8    P2p 

```

## Включим интерфейсы условного внутреннего контура (зеленый цвет на схеме) и выполним анализ вывода команды `show spanning-tree`

```
S1(config)#interface range e0/3,e0/0
S1(config-if-range)#no shutdown 

S1#sh spanning-tree 

VLAN0001
  Spanning tree enabled protocol rstp
  Root ID    Priority    1
             Address     aabb.cc00.0100
             This bridge is the root
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    1      (priority 0 sys-id-ext 1)
             Address     aabb.cc00.0100
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  300 sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Et0/0               Desg FWD 100       128.1    P2p 
Et0/1               Desg FWD 100       128.2    P2p 
Et0/2               Desg FWD 100       128.3    P2p 
Et0/3               Desg FWD 100       128.4    P2p 
```

```

S2(config)#interface range ethernet 0/0,e1/2
S2(config-if-range)#no shutdown 
S2(config-if-range)#end

S2#show spanning-tree 

VLAN0001
  Spanning tree enabled protocol rstp
  Root ID    Priority    1
             Address     aabb.cc00.0100
             Cost        100
             Port        1 (Ethernet0/0)
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     aabb.cc00.0200
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  300 sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Et0/0               Root FWD 100       128.1    P2p 
Et0/1               Altn BLK 100       128.2    P2p 
Et1/2               Desg FWD 100       128.7    P2p 
Et1/3               Desg FWD 100       128.8    P2p               

```

```

S3(config)#interface range ethernet 0/3,e1/0 
S3(config-if-range)#no shutdown 

S3#show spanning-tree 

VLAN0001
  Spanning tree enabled protocol rstp
  Root ID    Priority    1
             Address     aabb.cc00.0100
             Cost        100
             Port        3 (Ethernet0/2)
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     aabb.cc00.0300
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  300 sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Et0/2               Root FWD 100       128.3    P2p 
Et0/3               Altn BLK 100       128.4    P2p 
Et1/0               Altn BLK 100       128.5    P2p 
Et1/1               Altn BLK 100       128.6    P2p 

```

## Конфигурации устройств

### S1

<details>
  <summary>Конфигурация</summary>

```
version 17.12
service timestamps debug datetime msec
service timestamps log datetime msec
service password-encryption
!
hostname S1
!
boot-start-marker
boot-end-marker
!
!
no aaa new-model
!
!
!
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
!
ip audit notify log
ip audit po max-events 100
no ip domain lookup
ip cef
login on-success log
no ipv6 cef
!
!
!
!
!
!
!
vtp version 1
multilink bundle-name authenticated
!         
!
!
!
memory free low-watermark processor 80589
!
!
spanning-tree mode rapid-pvst
spanning-tree extend system-id
spanning-tree vlan 1 priority 0
enable secret 9 $9$lr6yzy7y.TXWCU$5DznBmaO7c6wL.t6VS0yYlJf.JCN4SjyeSelqdgbKU6
!
!
vlan internal allocation policy ascending
no cdp log mismatch duplex
!
!
!
!
!
interface Ethernet0/0
 switchport trunk encapsulation dot1q
 switchport trunk allowed vlan 1
 switchport mode trunk
!
interface Ethernet0/1
 switchport trunk encapsulation dot1q
 switchport trunk allowed vlan 1
 switchport mode trunk
!
interface Ethernet0/2
 switchport trunk encapsulation dot1q
 switchport trunk allowed vlan 1
 switchport mode trunk
!
interface Ethernet0/3
 switchport trunk encapsulation dot1q
 switchport trunk allowed vlan 1
 switchport mode trunk
!
interface Ethernet1/0
!
interface Ethernet1/1
!
interface Ethernet1/2
!
interface Ethernet1/3
!
interface Vlan1
 ip address 192.168.1.1 255.255.255.0
!
ip forward-protocol nd
!
!
ip tcp synwait-time 5
ip http server
ip http secure-server
ip ssh bulk-mode 131072
!
!
!
!
!
!
control-plane
!
!
banner motd ^CC 
Unauthorized access is strictly prohibited and prosecuted to the full extent of the law
.
^C        
!
line con 0
 exec-timeout 0 0
 privilege level 15
 password 7 030752180500
 logging synchronous
line aux 0
 exec-timeout 0 0
 privilege level 15
 logging synchronous
line vty 0 4
 password 7 01100F175804
 login
 transport input ssh
!
!
end

```
</details>

### S2

<details>
  <summary>Конфигурация</summary>

```

!
! Last configuration change at 18:38:29 UTC Fri Oct 18 2024
!
version 17.12
service timestamps debug datetime msec
service timestamps log datetime msec
service password-encryption
!
hostname S2
!
boot-start-marker
boot-end-marker
!
!
no aaa new-model
!
!
!
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
!
ip audit notify log
ip audit po max-events 100
no ip domain lookup
ip cef
login on-success log
no ipv6 cef
!
!
!
!
!
!
!
vtp version 1
multilink bundle-name authenticated
!
!
!         
!
memory free low-watermark processor 80589
!
!
spanning-tree mode rapid-pvst
spanning-tree extend system-id
enable secret 9 $9$qI1Hoiel1hpnq.$yOOCCZS.pkWhEM2xUvbgCzQXFq84y7kUEcXC3Y9/hNQ
!
!
vlan internal allocation policy ascending
no cdp log mismatch duplex
!
!
!
!
!
interface Ethernet0/0
 switchport trunk encapsulation dot1q
 switchport trunk allowed vlan 1
 switchport mode trunk
!
interface Ethernet0/1
 switchport trunk encapsulation dot1q
 switchport trunk allowed vlan 1
 switchport mode trunk
!
interface Ethernet0/2
!
interface Ethernet0/3
 shutdown
!
interface Ethernet1/0
 shutdown
!
interface Ethernet1/1
!
interface Ethernet1/2
 switchport trunk encapsulation dot1q
 switchport trunk allowed vlan 1
 switchport mode trunk
!
interface Ethernet1/3
 switchport trunk encapsulation dot1q
 switchport trunk allowed vlan 1
 switchport mode trunk
!         
interface Vlan1
 ip address 192.168.1.2 255.255.255.0
!
ip forward-protocol nd
!
!
ip tcp synwait-time 5
ip http server
ip http secure-server
ip ssh bulk-mode 131072
!
!
!
!
!
!
control-plane
!
!
banner motd ^CC 
Unauthorized access is strictly prohibited and prosecuted to the full extent of the law
.
^C        
!
line con 0
 exec-timeout 0 0
 privilege level 15
 password 7 14141B180F0B
 logging synchronous
line aux 0
 exec-timeout 0 0
 privilege level 15
 logging synchronous
line vty 0 4
 password 7 104D000A0618
 login
 transport input ssh
!
!
end

```
</details>

### S3

<details>
  <summary>Конфигурация</summary>

```

Current configuration : 1893 bytes
!
! Last configuration change at 18:44:48 UTC Fri Oct 18 2024
!
version 17.12
service timestamps debug datetime msec
service timestamps log datetime msec
service password-encryption
!
hostname S3
!
boot-start-marker
boot-end-marker
!
!
no aaa new-model
!
!
!
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
!
ip audit notify log
ip audit po max-events 100
no ip domain lookup
ip cef
login on-success log
no ipv6 cef
!
!
!
!
!
!
!
vtp version 1
multilink bundle-name authenticated
!         
!
!
!
memory free low-watermark processor 80589
!
!
spanning-tree mode rapid-pvst
spanning-tree extend system-id
enable secret 9 $9$RSFveGI9RsCGok$LEiUi4y0.tqtA/kJP/coVC0KmskGMnn2hTLgDyuFuVI
!
!
vlan internal allocation policy ascending
no cdp log mismatch duplex
!
!
!
!
!
interface Ethernet0/0
 shutdown
!
interface Ethernet0/1
!         
interface Ethernet0/2
 switchport trunk encapsulation dot1q
 switchport trunk allowed vlan 1
 switchport mode trunk
!
interface Ethernet0/3
 switchport trunk encapsulation dot1q
 switchport trunk allowed vlan 1
 switchport mode trunk
!
interface Ethernet1/0
 switchport trunk encapsulation dot1q
 switchport trunk allowed vlan 1
 switchport mode trunk
!
interface Ethernet1/1
 switchport trunk encapsulation dot1q
 switchport trunk allowed vlan 1
 switchport mode trunk
!
interface Ethernet1/2
 shutdown
!         
interface Ethernet1/3
!
interface Vlan1
 ip address 192.168.1.3 255.255.255.0
!
ip forward-protocol nd
!
!
ip tcp synwait-time 5
ip http server
ip http secure-server
ip ssh bulk-mode 131072
!
!
!
!
!
!
control-plane
!
!
banner motd ^CC 
Unauthorized access is strictly prohibited and prosecuted to the full extent of the law
.
^C
!
line con 0
 exec-timeout 0 0
 privilege level 15
 password 7 02050D480809
 logging synchronous
line aux 0
 exec-timeout 0 0
 privilege level 15
 logging synchronous
line vty 0 4
 password 7 1511021F0725
 login
 transport input ssh
!
!
end


```
</details>
