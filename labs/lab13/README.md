# GRE,  VPN,  DMVPN

## Цели

1. Настроить GRE между офисами Москва и Санкт-Петербург
1. Настроить DMVPN между офисами Москва и Чокурдак, Лабытнанги

## Задание

1. Настроить GRE между офисами Москва и Санкт-Петербург.
1. Настроить DMVPN между офисами Москва и Чокурдак, Лабытнанги.
1. Все узлы в офисах лабораторной работы должны иметь IP связность.

## Топология

![a](media/lab13_1.PNG)

## Схема для импорта в PNETlab

[Схема для импорта в PNETlab](media/otus_cource_lab12_NAT_NTP_DHCP_pnetlab_export-20250210-195658.zip)

## Версии ПО

- PNETlab - 5.3.11
- Роутеры - Cisco IOS Software, Linux Software (I86BI_LINUX-ADVENTERPRISEK9-M), Version 15.4(2)T4
- Коммутаторы - Cisco IOS Software, Linux Software (I86BI_LINUXL2-ADVENTERPRISEK9-M), Version 15.2(CML_NIGHTLY_20150703)
- ПК - VPC

## Решение
1. Настроить GRE между офисами Москва и Санкт-Петербург.

      Так как все внутренние подсети (10.X.X.X/16) филиалов Москва, Санкт-Петербург, Чокурдак, Лабытнанги должны быть доступны через GRE, соответственно, эти подсети не должны быть анонсированы в сторону провайдеров Интерннет (за исключанием пула адресов, используемые для NAT, 10.10.13.0/24 и 10.10.14.0/24 в Москве, 10.50.52.0/25 и 10.50.52.128/25 в Санкт-Петербурге). На устройствах R42 и R43 нужно убрать статические маршруты до внутренних подсетей офисов Лабытнанги и Чокурдак.
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


1. Настроить DMVPN между офисами Москва и Чокурдак, Лабытнанги.



## Конфигурации устройств

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

### R42

<details>
  <summary>Конфигурация</summary>

```



```
</details>

### R43

<details>
  <summary>Конфигурация</summary>

```



```
</details>

### R52

<details>
  <summary>Конфигурация</summary>

```



```
</details>


### R61

<details>
  <summary>Конфигурация</summary>

```



```
</details>


### R71

<details>
  <summary>Конфигурация</summary>

```



```
</details>