# Настройка Router-on-a-Stick Inter-VLAN Routing

## Задание

[Ссылка на документ](media/lab01.docx)

## Топология

![a](media/lab01_1.PNG)

[Схема для инпорта в PNETlab](media/otus_cource_lab1_vlan_exports_pnetlab_export-20241013-155626.zip)

## Версии ПО

- Роутеры - Cisco IOS Software [Dublin], Linux Software (X86_64BI_LINUX-ADVENTERPRISEK9-M), Version  17.12.1, RELEASE SOFTWARE (fc5)
- Коммутаторы - Cisco IOS Software [Dublin], Linux Software (X86_64BI_LINUX_L2-ADVENTERPRISEK9-M), Version 17.12.1, RELEASE SOFTWARE (fc5)

## Таблица №1. Адресация

Device | Interface | IP Address | Subnet Mask | Default Gateway
--- | --- | --- | --- | ---
R1 | E0/1.3 | 192.168.3.1 | 255.255.255.0 | N/A
R1 | E0/1.4 | 192.168.4.1 | 255.255.255.0 | N/A
R1 | E0/1.8 | N/A | N/A | N/A
S1 | VLAN 3 | 192.168.3.11 | 255.255.255.0 | 192.168.3.1
S2 | VLAN 3 | 192.168.3.12 | 255.255.255.0 | 192.168.3.1
PC-A | eth0 | 192.168.3.3 | 255.255.255.0 | 192.168.3.1
PC-B | eth0 | 192.168.4.3 | 255.255.255.0 | 192.168.4.1


## Таблица №2. VLAN

VLAN | Name	| Interface Assigned
--- | --- | ---
3	| Management	| S1: VLAN 3; S2: VLAN 3; S1: E0/2
4	| Operations	| S2: E0/1
7	| Parking | S1: E0/3; S2: E0/2, E0/3.
8	| Native | N/A


## Проверка работоспособности стенда

### Проверим, что с устройства PC-A доступны default-gateway, PC-B, S2:

```
PC-A> show

NAME   IP/MASK              GATEWAY                             GATEWAY
PC-A   192.168.3.3/24       192.168.3.1
       fe80::250:79ff:fe66:6824/64

PC-A> 
PC-A> 
PC-A> ping 192.168.3.1

84 bytes from 192.168.3.1 icmp_seq=1 ttl=255 time=0.608 ms
84 bytes from 192.168.3.1 icmp_seq=2 ttl=255 time=0.796 ms
84 bytes from 192.168.3.1 icmp_seq=3 ttl=255 time=0.819 ms
84 bytes from 192.168.3.1 icmp_seq=4 ttl=255 time=0.745 ms
84 bytes from 192.168.3.1 icmp_seq=5 ttl=255 time=0.783 ms

PC-A> 
PC-A> 
PC-A> ping 192.168.4.3

84 bytes from 192.168.4.3 icmp_seq=1 ttl=63 time=2.406 ms
84 bytes from 192.168.4.3 icmp_seq=2 ttl=63 time=1.424 ms
84 bytes from 192.168.4.3 icmp_seq=3 ttl=63 time=1.358 ms
84 bytes from 192.168.4.3 icmp_seq=4 ttl=63 time=1.431 ms
84 bytes from 192.168.4.3 icmp_seq=5 ttl=63 time=1.302 ms

PC-A> 
PC-A> ping 192.168.3.12

84 bytes from 192.168.3.12 icmp_seq=1 ttl=255 time=0.767 ms
84 bytes from 192.168.3.12 icmp_seq=2 ttl=255 time=0.762 ms
84 bytes from 192.168.3.12 icmp_seq=3 ttl=255 time=0.925 ms
84 bytes from 192.168.3.12 icmp_seq=4 ttl=255 time=0.933 ms
84 bytes from 192.168.3.12 icmp_seq=5 ttl=255 time=0.822 ms
```

### Проверим, что с устройства PC-B доступно устройство PC-A через R1 (192.168.4.1):

```
PC-B> show

NAME   IP/MASK              GATEWAY                             GATEWAY
PC-B   192.168.4.3/24       192.168.4.1
       fe80::250:79ff:fe66:6825/64

PC-B> 
PC-B> 
PC-B> ping 192.168.3.3

84 bytes from 192.168.3.3 icmp_seq=1 ttl=63 time=1.358 ms
84 bytes from 192.168.3.3 icmp_seq=2 ttl=63 time=1.520 ms
84 bytes from 192.168.3.3 icmp_seq=3 ttl=63 time=1.477 ms
84 bytes from 192.168.3.3 icmp_seq=4 ttl=63 time=1.578 ms
84 bytes from 192.168.3.3 icmp_seq=5 ttl=63 time=1.416 ms

PC-B> 
PC-B> 
PC-B> tracer 192.168.3.3
trace to 192.168.3.3, 8 hops max, press Ctrl+C to stop
 1   192.168.4.1   1.161 ms  0.847 ms  0.788 ms
 2   *192.168.3.3   1.162 ms (ICMP type:3, code:3, Destination port unreachable)
```

## Конфигурации устройств

### R1

<details>
  <summary>Конфигурация</summary>
  
```
! Last configuration change at 12:43:49 UTC Sun Oct 13 2024
!
version 17.12
service timestamps debug datetime msec
service timestamps log datetime msec
service password-encryption
!
hostname R1
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
!
multilink bundle-name authenticated
!
!
crypto pki trustpoint TP-self-signed-67180548
 enrollment selfsigned
 subject-name cn=IOS-Self-Signed-Certificate-67180548
 revocation-check none
 rsakeypair TP-self-signed-67180548
 hash sha256
!
!
crypto pki certificate chain TP-self-signed-67180548
 certificate self-signed 01
  3082032C 30820214 A0030201 02020101 300D0609 2A864886 F70D0101 0B050030 
  2F312D30 2B060355 04030C24 494F532D 53656C66 2D536967 6E65642D 43657274 
  69666963 6174652D 36373138 30353438 301E170D 32343130 31333130 30323030 
  5A170D33 34313031 33313030 3230305A 302F312D 302B0603 5504030C 24494F53 
  2D53656C 662D5369 676E6564 2D436572 74696669 63617465 2D363731 38303534 
  38308201 22300D06 092A8648 86F70D01 01010500 0382010F 00308201 0A028201 
  0100A9EF B0615B34 05A30F5E BC748DF4 7AE22665 CB8DB4E1 FC039D79 55405EA9 
  31A96B26 37D4F4A5 D35B3F67 0BB5C593 F968C721 E929BE58 9369A82E B07A3430 
  7D09DFB3 BF015FBA 052E25F6 A114290A 078B7F16 BD0C5A4B 8B65E249 08F82426 
  6E6FD4D2 D8ED23F6 162F91B3 90541479 43445094 6C01EDD8 28C1C455 FC0FE3F9 
  07EDC2B6 5C9D8CEF 77D8BBB9 BDB8A6A2 55B11187 E1FCCADE 77DCE4A5 00625AE1 
  45C96164 7BAF49EE C1CD53FC 558DC636 B91034CF 341BDF25 BAF72C07 D48B8F9B 
  126846BF 8ECC4B97 4C0C0C01 8CE56237 8CF62048 33C0B030 209702C9 5EE187D9 
  4E3AACF8 0B2C52B0 DC955212 D46B6B5C 59A1FB2A 39E2B310 A3FA38CB 3AA31844 
  C3FD0203 010001A3 53305130 1D060355 1D0E0416 04143CDC 9E05AECC 02B6DF79 
  B950D66E 5E327CB3 DCE2301F 0603551D 23041830 1680143C DC9E05AE CC02B6DF 
  79B950D6 6E5E327C B3DCE230 0F060355 1D130101 FF040530 030101FF 300D0609 
  2A864886 F70D0101 0B050003 82010100 7A720D83 1DBD9F23 23B769D7 ECC9CBC1 
  1988EB13 02012D47 0A7DBD1B 654D2EBF B8DABDC1 2EF5FF9E 46298FF2 BBA1E151 
  6A55201C 17FD055B 143246F7 46214B48 A1AE0F7A F172CA08 2CBF6D10 6501655A 
  599F3781 29010C27 4C60F1FA A4EC5E34 895BF216 19F66C61 F77F6DBF B8CC4645 
  1B2CD9B4 E0EB36CA 13582A11 3E26C92F 3B1CD87F 67993FAC F1A338C9 071DAD4E 
  075E9F56 503D9E72 146B5744 87EBDCD2 B0EE84D7 BCE164D7 BACF0984 0F349C23 
  D482988B 484EA65D 159F6716 4C6E4AC3 0387931F B6AC21F2 B0AB5281 7054FBC3 
  5574076D 47658AA9 0FB3C87A E63EEF04 472FA17C DFA4D30D E61F8A52 4A11CFA3 
  77473347 F8E97ABB AAB63F48 AEA528A2
  quit
!
!
memory free low-watermark processor 81225
!
!
spanning-tree mode rapid-pvst
!
enable secret 9 $9$YA6Japa1c0K3ZU$0iGzn7Gm098aiplFvNCzIi4tzCU24ONDFXBZF.NQAOc
!
!         
!
!
!
no cdp log mismatch duplex
!
! 
!
!
!
!
!
!
!
!
!
!
!
!
!
!
!
interface Tunnel0
 no ip address
!
interface Ethernet0/0
 no ip address
!
interface Ethernet0/0.3
 description Management VLAN
 encapsulation dot1Q 3
 ip address 192.168.3.1 255.255.255.0
!
interface Ethernet0/0.4
 description Operations VLAN
 encapsulation dot1Q 4
 ip address 192.168.4.1 255.255.255.0
!
interface Ethernet0/0.8
 encapsulation dot1Q 8 native
!
interface Ethernet0/1
 no ip address
!
interface Ethernet0/2
 no ip address
!         
interface Ethernet0/3
 no ip address
!
ip forward-protocol nd
!
ip tcp synwait-time 5
!
ip http server
ip http secure-server
ip ssh bulk-mode 131072
!
!
!
!
!
control-plane
!
!
banner motd ^C 
Unauthorized access is strictly prohibited and prosecuted to the full extent of the law.
^C
!         
line con 0
 exec-timeout 0 0
 privilege level 15
 password 7 094F471A1A0A
 logging synchronous
line aux 0
 exec-timeout 0 0
 privilege level 15
 logging synchronous
line vty 0 4
 password 7 060506324F41
 login
 transport input ssh
!
!
!
!
end
```
  
</details>

### S1

<details>
  <summary>Конфигурация</summary>

```
! Last configuration change at 12:48:17 UTC Sun Oct 13 2024
!
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
!
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
enable secret 9 $9$0Crs7kAFEQSpP.$y5Ml7K4WYDyA7oC.C5G/SCaEi/NstWgb21gDhBZRwB.
!
!
vlan internal allocation policy ascending
!
!
!
!
!
interface Ethernet0/0
 switchport trunk encapsulation dot1q
 switchport trunk native vlan 8
 switchport trunk allowed vlan 3,4,8
 switchport mode trunk
!
interface Ethernet0/1
 switchport trunk encapsulation dot1q
 switchport trunk native vlan 8
 switchport trunk allowed vlan 3,4,8
 switchport mode trunk
!
interface Ethernet0/2
 switchport access vlan 3
 switchport mode access
!
interface Ethernet0/3
 switchport access vlan 7
 switchport mode access
 shutdown
!
interface Vlan3
 ip address 192.168.3.11 255.255.255.0
!
ip default-gateway 192.168.3.1
ip forward-protocol nd
!
!
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
banner motd ^C
Unauthorized access is strictly prohibited and prosecuted to the full extent of the law.
^C
!
line con 0
 password 7 00071A150754
 logging synchronous
line aux 0
line vty 0 4
 password 7 045802150C2E
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
! Last configuration change at 12:08:30 UTC Sun Oct 13 2024
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
enable secret 9 $9$jA6NSgpJrEfKyE$3uOO4yeZtDH6jSQlynxybqJA.j9DbpwS1w4mjI7P2AE
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
 switchport trunk native vlan 8
 switchport trunk allowed vlan 3,4,8
 switchport mode trunk
!
interface Ethernet0/1
 switchport access vlan 4
 switchport mode access
!
interface Ethernet0/2
 switchport access vlan 7
 switchport mode access
 shutdown
!
interface Ethernet0/3
 switchport access vlan 7
 switchport mode access
 shutdown
!
interface Vlan3
 ip address 192.168.3.12 255.255.255.0
!
ip default-gateway 192.168.3.1
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
banner motd ^C
Unauthorized access is strictly prohibited and prosecuted to the full extent of the law.
^C
!
line con 0
 exec-timeout 0 0
 privilege level 15
 password 7 05080F1C2243
 logging synchronous
line aux 0
 exec-timeout 0 0
 privilege level 15
 password 7 05080F1C2243
 logging synchronous
line vty 0 2
 password 7 05080F1C2243
 login
 transport input ssh
line vty 3 4
 login
 transport input ssh
!
!
end

S2#  
```
</details>

### PC-A

<details>
  <summary>Конфигурация</summary>

```
PC-A> show

NAME   IP/MASK              GATEWAY                             GATEWAY
PC-A   192.168.3.3/24       192.168.3.1
       fe80::250:79ff:fe66:6824/64
```

</details>



### PC-B


<details>
  <summary>Конфигурация</summary>

```
PC-B> show

NAME   IP/MASK              GATEWAY                             GATEWAY
PC-B   192.168.4.3/24       192.168.4.1
       fe80::250:79ff:fe66:6825/64
```

</details>

