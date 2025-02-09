# Основные протоколы сети интернет

## Цели

1. Настроить DHCP в офисе Москва
1. Настроить синхронизацию времени в офисе Москва при помощи протокола NTP
1. Настроить NAT в офисе Москва, Санкт-Петербург и Чокурдак


## Задание

1. Настроить NAT(PAT) на R13 и R14 в офисе Москва. Трансляция должна осуществляться в адрес автономной системы AS1001.
1. Настроить NAT так, чтобы R15 в офисе Москва был доступен с любого узла для удаленного управления.
1. Настроить статический NAT для R16 в офисе Москва.
1. Настроить NAT(PAT) на R52 в офисе Санкт-Петербург. Трансляция должна осуществляться в пул из 5 адресов автономной системы AS2042.
1. настроить статический NAT(PAT) для офиса Чокурдак.
1. Настроить IPv4 DHCP сервер в офисе Москва на маршрутизаторах R11 и R12. VPC11 и VPC12 должны получать сетевые настройки по DHCP.
1. Настроить NTP сервер на R11 и R12. Все устройства в офисе Москва должны синхронизировать время с R11 и R12.

## Топология

![a](media/lab12_1.PNG)

## Схема для импорта в PNETlab

[Схема для импорта в PNETlab](media/otus_cource_lab11_eBGP(filtering)_pnetlab_export-20250201-165622.zip)

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

1. Настроить NAT(PAT) на R52 в офисе Санкт-Петербурге. Трансляция должна осуществляться в пул из 5 адресов автономной системы AS2042.
      
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


## Проверка работоспособности



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

### R52

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