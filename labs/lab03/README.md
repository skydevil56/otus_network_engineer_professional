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

## Таблица №1. Адресация (IPv4)

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

## Таблица №2. Адресация (IPv6)

Device | Interface	| IPv6 Address | Default Gateway
--- | --- | --- | --- 
R1 | e0/0 | 2001:db8:acad:2::1/64, fe80::1 | N/A
R1 | e0/1 | N/A | N/A
R1 | e0/1.100 | 2001:db8:acad:1::1/64, fe80::1 | N/A
R1 | e0/1.200 | N/A | N/A 
R1 | e0/1.1000 | N/A	| N/A
R2 | e0/0	| 2001:db8:acad:2::2/64, fe80::2	| N/A
R2 | e0/1	| 2001:db8:acad:3::1 /64, fe80::1 | N/A
S1 | VLAN 200 |	N/A | N/A 
S2 | VLAN 1	| N/A | N/A
PC-A |	NIC | DHCP | DHCP
PC-B |	NIC | DHCP | DHCP

## Таблица №3. VLAN

VLAN | Name	| Interface Assigned
--- | --- | ---
1	| N/A	| S2: e0/1
100	| Clients | S1: e0/1
200	| Management| S1: VLAN 200 
999	| Parking |	S1: e0/2-3
1000 | Native | N/A

## Проверка IPv4

### Проверим получение сетевых настроек (IPv4) на устройстве `PC-A`
```
root@PC-A:~# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 
1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host noprefixroute 
       valid_lft forever preferred_lft forever
2: ens3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group defau
lt qlen 1000
    link/ether 50:34:2e:00:29:00 brd ff:ff:ff:ff:ff:ff
    altname enp0s3
    inet 192.168.1.6/26 brd 192.168.1.63 scope global dynamic ens3
       valid_lft 217544sec preferred_lft 217544sec
    inet6 fe80::5234:2eff:fe00:2900/64 scope link 
       valid_lft forever preferred_lft forever
```

```
root@PC-A:~# ip r
default via 192.168.1.1 dev ens3 
192.168.1.0/26 dev ens3 proto kernel scope link src 192.168.1.6 
```

```
root@PC-A:~# ping 192.168.1.1 -c 4
PING 192.168.1.1 (192.168.1.1) 56(84) bytes of data.
64 bytes from 192.168.1.1: icmp_seq=1 ttl=255 time=1.08 ms
64 bytes from 192.168.1.1: icmp_seq=2 ttl=255 time=1.03 ms
64 bytes from 192.168.1.1: icmp_seq=3 ttl=255 time=1.08 ms
64 bytes from 192.168.1.1: icmp_seq=4 ttl=255 time=1.09 ms

--- 192.168.1.1 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3004ms
rtt min/avg/max/mdev = 1.033/1.072/1.093/0.023 ms
```

### Проверим получение сетевых настроек (IPv4) на устройстве `PC-B`

```
root@PC-B:~# ifup ens3 
Internet Systems Consortium DHCP Client 4.4.3-P1
Copyright 2004-2022 Internet Systems Consortium.
All rights reserved.
For info, please visit https://www.isc.org/software/dhcp/

Listening on LPF/ens3/50:77:d8:00:2a:00
Sending on   LPF/ens3/50:77:d8:00:2a:00
Sending on   Socket/fallback
DHCPDISCOVER on ens3 to 255.255.255.255 port 67 interval 4
DHCPOFFER of 192.168.1.102 from 192.168.1.97
DHCPREQUEST for 192.168.1.102 on ens3 to 255.255.255.255 port 67
DHCPACK of 192.168.1.102 from 192.168.1.97
bound to 192.168.1.102 -- renewal in 84819 seconds.
```

```
root@PC-B:~# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 
1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host noprefixroute 
       valid_lft forever preferred_lft forever
2: ens3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group defau
lt qlen 1000
    link/ether 50:77:d8:00:2a:00 brd ff:ff:ff:ff:ff:ff
    altname enp0s3
    inet 192.168.1.102/28 brd 192.168.1.111 scope global dynamic ens3
       valid_lft 217721sec preferred_lft 217721sec
    inet6 fe80::5277:d8ff:fe00:2a00/64 scope link 
       valid_lft forever preferred_lft forever
```

```
root@PC-B:~# ip r
default via 192.168.1.97 dev ens3 
192.168.1.96/28 dev ens3 proto kernel scope link src 192.168.1.102 
```

```
root@PC-B:~# ping 192.168.1.97 -c 4
PING 192.168.1.97 (192.168.1.97) 56(84) bytes of data.
64 bytes from 192.168.1.97: icmp_seq=1 ttl=255 time=1.07 ms
64 bytes from 192.168.1.97: icmp_seq=2 ttl=255 time=1.03 ms
64 bytes from 192.168.1.97: icmp_seq=3 ttl=255 time=1.03 ms
64 bytes from 192.168.1.97: icmp_seq=4 ttl=255 time=0.999 ms

--- 192.168.1.97 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3005ms
rtt min/avg/max/mdev = 0.999/1.031/1.067/0.024 ms
```

### Проверим информацию о работе DHCP сервера (IPv4) на устройстве `R1`

```
R1#show ip dhcp pool    

Pool R1_Client_LAN :
 Utilization mark (high/low)    : 100 / 0
 Subnet size (first/next)       : 0 / 0 
 Total addresses                : 62
 Leased addresses               : 1
 Excluded addresses             : 5
 Pending event                  : none
 1 subnet is currently in the pool :
 Current index        IP address range                    Leased/Excluded/Total
 192.168.1.7          192.168.1.1      - 192.168.1.62      1     / 5     / 62   

Pool R2_Client_LAN :
 Utilization mark (high/low)    : 100 / 0
 Subnet size (first/next)       : 0 / 0 
 Total addresses                : 14
 Leased addresses               : 1
 Excluded addresses             : 5
 Pending event                  : none
 1 subnet is currently in the pool :
 Current index        IP address range                    Leased/Excluded/Total
 192.168.1.103        192.168.1.97     - 192.168.1.110     1     / 5     / 14   
```

```
R1#show ip dhcp binding 
Bindings from all pools not associated with VRF:
IP address      Client-ID/ Lease expiration Type       State      Interface
Hardware address/
User name
192.168.1.6     ff2e.0029.0000.0100.    Oct 22 2024 04:53 AM    Automatic  Active     Ethernet0/1.100
                012e.a699.7750.342e.
                0029.00
192.168.1.102   ffd8.002a.0000.0100.    Oct 22 2024 05:08 AM    Automatic  Active     Ethernet0/0
                012e.a69c.5050.77d8.
                002a.00
```

Видно, что оба устройства `PC-A` и `PC-B` успешно получили сетевые настройки от DHCP сервера на `R1`.

## Проверка IPv6



## Конфигурации устройств

### PC-A

<details>
  <summary>Конфигурация</summary>

```



```
</details>

### PC-B

<details>
  <summary>Конфигурация</summary>

```



```
</details>


### S1

<details>
  <summary>Конфигурация</summary>

```



```
</details>

### S2

<details>
  <summary>Конфигурация</summary>

```



```
</details>

### R1

<details>
  <summary>Конфигурация</summary>

```



```
</details>

### R2

<details>
  <summary>Конфигурация</summary>

```



```
</details>
