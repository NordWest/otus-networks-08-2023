#  Основные протоколы сети интернет


## Цель

- Настроить DHCP в офисе Москва
- Настроить синхронизацию времени в офисе Москва
- Настроить NAT в офисе Москва, C.-Перетбруг и Чокурдах

##  Задание:

1. Настроить NAT(PAT) на R14 и R15. Трансляция должна осуществляться в адрес автономной системы AS1001.
2. Настроить NAT(PAT) на R18. Трансляция должна осуществляться в пул из 5 адресов автономной системы AS2042.
3. Настроить статический NAT для R20.
4. Настроить NAT так, чтобы R19 был доступен с любого узла для удаленного управления.
5. Настроить статический NAT(PAT) для офиса Чокурдах.
6. Настроить для IPv4 DHCP сервер в офисе Москва на маршрутизаторах R12 и R13. VPC1 и VPC7 должны получать сетевые настройки по DHCP.
7. Настроить NTP сервер на R12 и R13. Все устройства в офисе Москва должны синхронизировать время с R12 и R13.


### 1. Общие положения.

#### 1.1 Схема сети

![](img/lab14.png)

#### 1.2 Таблица адресации


| Device        | Interface     | IP address      | Subnet mask     | Default gateway |
| ------------- | ------------- | --------------- | --------------- | --------------- |
| R12           | VLAN10        | 192.168.100.2   | 255.255.255.128 | N/A             |
|               | VLAN20        | 192.168.100.130 | 255.255.255.128 | N/A             |
|               | VLAN40        | 172.16.100.2    | 255.255.255.0   | N/A             |
|               | e0/2          | 10.0.0.0        | 255.255.255.254 | N/A             |
|               | e0/3          | 10.0.0.2        | 255.255.255.254 | N/A             |
| R13           | VLAN10        | 192.168.100.3   | 255.255.255.128 | N/A             |
|               | VLAN20        | 192.168.100.131 | 255.255.255.128 | N/A             |
|               | VLAN40        | 172.16.100.3    | 255.255.255.0   | N/A             |
|               | e0/2          | 10.0.0.4        | 255.255.255.254 | N/A             |
|               | e0/3          | 10.0.0.6        | 255.255.255.254 | N/A             |
| R14           | e0/0          | 10.0.0.1        | 255.255.255.254 | N/A             |
|               | e0/1          | 10.0.0.7        | 255.255.255.254 | N/A             |
|               | e0/2          | 10.0.0.8        | 255.255.255.254 | N/A             |
|               | e0/3          | 10.0.0.10       | 255.255.255.254 | N/A             |
|               | e1/0          | 10.0.0.46       | 255.255.255.254 | N/A             |
| R15           | e0/0          | 10.0.0.5        | 255.255.255.254 | N/A             |
|               | e0/1          | 10.0.0.3        | 255.255.255.254 | N/A             |
|               | e0/2          | 10.0.0.12       | 255.255.255.254 | N/A             |
|               | e0/3          | 10.0.0.14       | 255.255.255.254 | N/A             |
|               | e1/0          | 10.0.0.47       | 255.255.255.254 | N/A             |
|               |               | 10.10.0.16      | 255.255.255.255 | N/A             |
| R18           | e0/0          | 10.0.0.17       | 255.255.255.254 | N/A             |
|               | e0/1          | 10.0.0.21       | 255.255.255.254 | N/A             |
|               | e0/2          | 10.0.0.22       | 255.255.255.254 | N/A             |
|               | e0/3          | 10.0.0.24       | 255.255.255.254 | N/A             |
| R19           | e0/0          | 10.0.0.11       | 255.255.255.254 | N/A             |
|               | lo1           | 10.1.0.19       | 255.255.255.255 | N/A             |
| R20           | e0/0          | 10.0.0.15       | 255.255.255.254 | N/A             |

#### 1.3 Public IP.

| AS            | IP global               | IP local          |
|---------------|-------------------------|-------------------|
| 1001          | 10.10.0.1/32            | 10.0.0.0          |
|               |                         | 10.0.0.2          |
|               | 10.10.0.2/32            | 10.0.0.15         |
|               | 10.10.0.3/32            | 10.0.0.11         |
| 2042          | 10.10.1.1-10.10.1.5     | 10.0.0.20         |
| Чокурдах      | 10.10.2.1               | 172.16.102.10     |


### 2. Настройка.
#### 2.1 Настроить NAT(PAT) на R14 и R15. Трансляция должна осуществляться в адрес автономной системы AS1001.
- Сначала R15, через который идет трафик.
- Прописываю маршрут до белого адреса и объявляю его в BGP
```
R15(config)#ip route 10.10.0.1 255.255.255.255 null 0
R15(config)#router bgp 1001
R15(config-router)#network 10.10.0.1 mask 255.255.255.255
```
- Теперь настройка NAT. Пока только для одного интерфейса со стороны R12.
```
R15(config)#access-list 100 permit ip 10.0.0.2 0.0.0.0 any
R15(config)#ip nat pool POOL_NAT_AS1001 10.10.0.1 10.10.0.1 prefix 24

R15(config)#interface ethernet 0/2
R15(config-if)#ip nat outside
R15(config)#interface ethernet 0/1
R15(config-if)#ip nat inside

R15(config)#ip nat inside source list 100 pool POOL_NAT_AS1001 overload
```
- Проверяю пингом.
```
R12>ping 10.0.0.22
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.0.0.22, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/3 ms
R12>
```
- Пинг есть, в таблице трансляции виже работу NAT.
```
R15#sh ip nat translations
Pro Inside global      Inside local       Outside local      Outside global
icmp 10.10.0.1:7       10.0.0.2:7         10.0.0.22:7        10.0.0.22:7
```
- Теперь настрою на R14.
```
R14(config)#ip route 10.10.0.1 255.255.255.255 null 0
R14(config)#router bgp 1001
R14(config-router)#network 10.10.0.1 mask 255.255.255.255

R14(config)#interface e0/0
R14(config-if)#ip nat inside
R14(config-if)#exit
R14(config)#interface e0/2
R14(config-if)#ip nat outside
R14(config-if)#exit
R14(config)#access-list 100 permit ip 10.0.0.0 0.255.255.255 any
R14(config)#access-list 100 permit ip 10.0.0.2 0.255.255.255 any
R14(config)#ip nat pool POOL_NAT_AS1001 10.10.0.1 10.10.0.1 prefix 24
R14(config)#ip nat inside source list 100 pool POOL_NAT_AS1001 overload
```
- Проверяю
```
R12>ping 10.0.0.22
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.0.0.22, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 2/2/3 ms


R14(config)#do show ip nat translations                         
Pro Inside global      Inside local       Outside local      Outside global
icmp 10.10.0.1:21      10.0.0.2:21        10.0.0.22:21       10.0.0.22:21
tcp 10.10.0.1:179      10.0.0.8:179       10.0.0.9:50125     10.0.0.9:50125
tcp 10.10.0.1:15704    10.0.0.8:15704     10.0.0.9:179       10.0.0.9:179
tcp 10.10.0.1:39263    10.0.0.8:39263     10.0.0.9:179       10.0.0.9:179
```
- Непонятно только, почему транслируется 10.0.0.8.


#### 2.2 Настроить NAT(PAT) на R18. Трансляция должна осуществляться в пул из 5 адресов автономной системы AS2042.
- Настраиваю R18
```
R18(config)#ip route 10.10.1.1 255.255.255.255 null 0
R18(config)#ip route 10.10.1.2 255.255.255.255 null 0
R18(config)#ip route 10.10.1.3 255.255.255.255 null 0
R18(config)#ip route 10.10.1.4 255.255.255.255 null 0
R18(config)#ip route 10.10.1.5 255.255.255.255 null 0

R18(config)#router bgp 2042
R18(config-router)#network 10.10.1.1 mask 255.255.255.255
R18(config-router)#network 10.10.1.2 mask 255.255.255.255
R18(config-router)#network 10.10.1.3 mask 255.255.255.255
R18(config-router)#network 10.10.1.4 mask 255.255.255.255
R18(config-router)#network 10.10.1.5 mask 255.255.255.255
R18(config-router)#exit
```
- Сморю на R24 и там сетей 10.10.1.[1-5] нет. Надо их добавить ещё в prefix-list.
```
R18(config)#ip prefix-list NO-TRANSIT permit 10.10.1.1/32
R18(config)#ip prefix-list NO-TRANSIT permit 10.10.1.2/32
R18(config)#ip prefix-list NO-TRANSIT permit 10.10.1.3/32
R18(config)#ip prefix-list NO-TRANSIT permit 10.10.1.4/32
R18(config)#ip prefix-list NO-TRANSIT permit 10.10.1.5/32
```
- Адреса появились.
```
R24#show ip bgp
...
 * i 10.10.1.1/32     10.0.0.24                0    100      0 2042 i
 *>                   10.0.0.22                0             0 2042 i
 * i 10.10.1.2/32     10.0.0.24                0    100      0 2042 i
 *>                   10.0.0.22                0             0 2042 i
 * i 10.10.1.3/32     10.0.0.24                0    100      0 2042 i
 *>                   10.0.0.22                0             0 2042 i
 * i 10.10.1.4/32     10.0.0.24                0    100      0 2042 i
 *>                   10.0.0.22                0             0 2042 i
 * i 10.10.1.5/32     10.0.0.24                0    100      0 2042 i
 *>                   10.0.0.22                0             0 2042 i
 r>i 192.168.102.0    10.0.0.45                0    100      0 i
```
- Настраиваю NAT.
```
R15(config)#access-list 100 permit ip 10.0.0.20 0.0.0.0 any
R15(config)#ip nat pool POOL_NAT_AS2042 10.10.1.1 10.10.1.5 prefix 24

R18(config)#interface e0/2
R18(config-if)#ip nat outside
R18(config-if)#exit
R18(config)#interface e0/3
R18(config-if)#ip nat outside
R18(config-if)#exit
R18(config)#interface e0/0
R18(config-if)#ip nat inside
R18(config-if)#exit          
R18(config)#interface e0/1
R18(config-if)#ip nat inside

R18(config)#ip nat inside source list 100 pool POOL_NAT_AS2042 overload
```
- Проверяю
```
R17>ping 10.0.0.12
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.0.0.12, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/3 ms
R17>
R18#show ip nat translations
Pro Inside global      Inside local       Outside local      Outside global
icmp 10.10.1.1:3       10.0.0.20:3        10.0.0.12:3        10.0.0.12:3
R18#
```

#### 2.3 Настроить статический NAT для R20.
- Для начала настраиваю iBGP между R15 и R20, чтобы на последнем появились какие-то внешние маршруты.
- Назначаю роутеру трансляцию адреса 10.10.0.2
```
R15(config)#ip nat inside source static 10.0.0.15 10.10.0.2
R15(config)#ip route 10.10.0.2 255.255.255.255 Null0
R15(config)#interface e0/3
R15(config-if)#ip nat inside
R15(config)#router bgp 1001
R15(config-router)#network 10.10.0.2 mask 255.255.255.255
```
- Проверяю
```
R20#ping 10.0.0.22
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.0.0.22, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/2/6 ms

R15#show ip nat translations
Pro Inside global      Inside local       Outside local      Outside global
icmp 10.10.0.2:4       10.0.0.15:4        10.0.0.22:4        10.0.0.22:4

```
#### 2.4 Настроить NAT так, чтобы R19 был доступен с любого узла для удаленного управления.
- Настраиваю на R15.
```
R15(config)#ip route 10.10.0.3 255.255.255.255 null 0
R15(config)#router bgp 1001
R15(config-router)#network 10.10.0.3 mask 255.255.255.255
R15(config-router)#exit
R15(config)#interface e1/0
R15(config-if)#ip nat in
R15(config-if)#ip nat inside
R15(config-if)#exit

R15(config)#ip nat inside source static tcp 10.0.0.11 22 10.10.0.3 1922
```

- При обращении по ssh на 10.10.0.3:1922 соединение будет пробрасываться на 10.0.0.11:22

#### 2.5 Настроить статический NAT(PAT) для офиса Чокурдах.
- Проверяю перед настройкой.
```
SW29#ping 10.0.0.32
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.0.0.32, timeout is 2 seconds:
.....
Success rate is 0 percent (0/5)

```
- Настраиваю.
```
R28(config)#ip nat inside source static 172.16.102.10 10.10.2.1
R28(config)#interface e0/2
R28(config-if)#ip nat inside
R28(config-if)#exit
R28(config)#interface e0/0
R28(config-if)#ip nat outside
R28(config-if)#exit
R28(config)#interface e0/1
R28(config-if)#ip nat outside
R28(config-if)#exit
```
- Проверяю.
```
SW29>ping 10.0.0.32
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.0.0.32, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/3 ms

R28(config)#do show ip nat translations
Pro Inside global      Inside local       Outside local      Outside global
icmp 10.10.2.1:15      172.16.102.10:15   10.0.0.32:15       10.0.0.32:15
--- 10.10.2.1          172.16.102.10      ---                ---

```
#### 2.6 Настроить для IPv4 DHCP сервер в офисе Москва на маршрутизаторах R12 и R13. VPC1 и VPC7 должны получать сетевые настройки по DHCP.
- Настройка R12.
```
R12(config)#ip dhcp excluded-address 192.168.100.1 192.168.100.9
R12(config)#ip dhcp excluded-address 192.168.100.129 192.168.100.137

R12(config)#ip dhcp pool pool_vpc1
R12(dhcp-config)#network 192.168.100.0 255.255.255.128
R12(dhcp-config)#default-router 192.168.100.1
R12(dhcp-config)#lease 0 0 30

```
- Проверяю на VPC1
```
VPC1> ip dhcp
DDORA IP 192.168.100.10/25 GW 192.168.100.1
```
- Теперь настройка для VPC7
```
R12(config)#ip dhcp pool pool_vpc7
R12(dhcp-config)#network 192.168.100.128 255.255.255.128
R12(dhcp-config)#default-router 192.168.100.129
R12(dhcp-config)#lease 0 0 30
```
- Проверяю
```
VPC7> ip dhcp
DDORA IP 192.168.100.138/25 GW 192.168.100.129
VPC7> ping 192.168.100.129

84 bytes from 192.168.100.129 icmp_seq=1 ttl=255 time=1.175 ms
84 bytes from 192.168.100.129 icmp_seq=2 ttl=255 time=2.290 ms
84 bytes from 192.168.100.129 icmp_seq=3 ttl=255 time=1.867 ms
84 bytes from 192.168.100.129 icmp_seq=4 ttl=255 time=1.550 ms
84 bytes from 192.168.100.129 icmp_seq=5 ttl=255 time=1.457 ms

- Для R13 настройка аналогичная.
```
#### 2.7 Настроить NTP сервер на R12 и R13. Все устройства в офисе Москва должны синхронизировать время с R12 и R13.
- Настройка R12.
```
R12(config)#ntp master 2
R12(config)#interface vlan10
R12(config-if)#ntp broadcast
R12(config-if)#exit
R12(config)#interface vlan20
R12(config-if)#ntp broadcast
R12(config-if)#exit
R12(config)#interface vlan40
R12(config-if)#ntp br
R12(config-if)#ntp broadcast

```
- Настраиваю клиент на SW3 и проверяю.
```
SW3(config)#interface vlan40
SW3(config-if)#ntp broadcast client

SW3#show ntp associations

  address         ref clock       st   when   poll reach  delay  offset   disp
* 172.16.100.2    127.127.1.1      2     18     64     1  1.000   0.500 7938.4
 * sys.peer, # selected, + candidate, - outlyer, x falseticker, ~ configured

```
- На остальных узлах настраивается аналогично.
