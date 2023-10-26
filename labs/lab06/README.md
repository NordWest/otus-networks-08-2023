#  PBR

### Цель:

 - Настроить политику маршрутизации в офисе Чокурдах
 - Распределить трафик между 2 линками

###  Задание:

 1. Настроить политику маршрутизации для сетей офиса.
 1. Распределить трафик между двумя линками с провайдером.
 1. Настроить отслеживание линка через технологию IP SLA.(только для IPv4)
 1. Настройть для офиса Лабытнанги маршрут по-умолчанию.


### 1 Разработаете и задокументируете адресное пространство для лабораторного стенда.

#### 1.1 Схема сети.

![](img/lab05.png)

#### 1.2 Таблица адресации.

 | Device        | Interface     | IP address      | Subnet mask     | Default gateway |
 | ------------- | ------------- | --------------- | --------------- | --------------- |
 | VPC1          | e0/0          | 192.168.100.10  | 255.255.255.128 | 192.168.100.1   |
 | SW2           | VLAN40        | 172.16.100.10   | 255.255.255.0   | 172.16.100.1    |
 | SW3           | VLAN40        | 172.16.100.11   | 255.255.255.0   | 172.16.100.1    |
 | SW4           | VLAN40        | 172.16.100.12   | 255.255.255.0   | 172.16.100.1    |
 | SW5           | VLAN40        | 172.16.100.13   | 255.255.255.0   | 172.16.100.1    |
 | VPC7          | e0/0          | 192.168.100.139 | 255.255.255.128 | 192.168.100.129 |
 | VPC8          | e0/0          | 192.168.101.10  | 255.255.255.128 | 192.168.101.1   |
 | SW9           | VLAN40        | 172.16.101.10   | 255.255.255.0   | 172.16.101.1    |
 | SW10          | VLAN40        | 172.16.101.11   | 255.255.255.0   | 172.16.101.1    |
 | VPC11         | e0/0          | 192.168.101.139 | 255.255.255.128 | 192.168.101.129 |
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
 | R15           | e0/0          | 10.0.0.5        | 255.255.255.254 | N/A             |
 |               | e0/1          | 10.0.0.3        | 255.255.255.254 | N/A             |
 |               | e0/2          | 10.0.0.12       | 255.255.255.254 | N/A             |
 |               | e0/3          | 10.0.0.14       | 255.255.255.254 | N/A             |
 | R16           | vlan10        | 192.168.101.2   | 255.255.255.128 | N/A             |
 |               | vlan20        | 192.168.101.130 | 255.255.255.128 | N/A             |
 |               | vlan40        | 172.16.101.2    | 255.255.255.0   | N/A             |
 |               | e0/1          | 10.0.0.16       | 255.255.255.254 | N/A             |
 |               | e0/3          | 10.0.0.18       | 255.255.255.254 | N/A             |
 | R17           | vlan10        | 192.168.101.3   | 255.255.255.128 | N/A             |
 |               | vlan20        | 192.168.101.131 | 255.255.255.128 | N/A             |
 |               | vlan40        | 172.16.101.3    | 255.255.255.0   | N/A             |
 |               | e0/1          | 10.0.0.20       | 255.255.255.254 | N/A             |
 | R18           | e0/0          | 10.0.0.17       | 255.255.255.254 | N/A             |
 |               | e0/1          | 10.0.0.21       | 255.255.255.254 | N/A             |
 |               | e0/2          | 10.0.0.22       | 255.255.255.254 | N/A             |
 |               | e0/3          | 10.0.0.24       | 255.255.255.254 | N/A             |
 | R19           | e0/0          | 10.0.0.11       | 255.255.255.254 | N/A             |
 | R20           | e0/0          | 10.0.0.15       | 255.255.255.254 | N/A             |
 | R21           | e0/0          | 10.0.0.13       | 255.255.255.254 | N/A             |
 |               | e0/1          | 10.0.0.26       | 255.255.255.254 | N/A             |
 |               | e0/2          | 10.0.0.28       | 255.255.255.254 | N/A             |
 | R22           | e0/0          | 10.0.0.9        | 255.255.255.254 | N/A             |
 |               | e0/1          | 10.0.0.27       | 255.255.255.254 | N/A             |
 |               | e0/2          | 10.0.0.30       | 255.255.255.254 | N/A             |
 | R23           | e0/0          | 10.0.0.31       | 255.255.255.254 | N/A             |
 |               | e0/1          | 10.0.0.32       | 255.255.255.254 | N/A             |
 |               | e0/2          | 10.0.0.34       | 255.255.255.254 | N/A             |
 | R24           | e0/0          | 10.0.0.29       | 255.255.255.254 | N/A             |
 |               | e0/1          | 10.0.0.36       | 255.255.255.254 | N/A             |
 |               | e0/2          | 10.0.0.35       | 255.255.255.254 | N/A             |
 |               | e0/3          | 10.0.0.23       | 255.255.255.254 | N/A             |
 | R25           | e0/0          | 10.0.0.33       | 255.255.255.254 | N/A             |
 |               | e0/1          | 10.0.0.38       | 255.255.255.254 | N/A             |
 |               | e0/2          | 10.0.0.40       | 255.255.255.254 | N/A             |
 |               | e0/3          | 10.0.0.42       | 255.255.255.254 | N/A             |
 | R26           | e0/0          | 10.0.0.37       | 255.255.255.254 | N/A             |
 |               | e0/1          | 10.0.0.44       | 255.255.255.254 | N/A             |
 |               | e0/2          | 10.0.0.41       | 255.255.255.254 | N/A             |
 |               | e0/3          | 10.0.0.25       | 255.255.255.254 | N/A             |
 | R27           | e0/0          | 10.0.0.39       | 255.255.255.254 | N/A             |
 | R28           | e0/0          | 10.0.0.45       | 255.255.255.254 | N/A             |
 |               | e0/1          | 10.0.0.43       | 255.255.255.254 | N/A             |
 |               | e0/2.10       | 192.168.102.1   | 255.255.255.128 | N/A             |
 |               | e0/2.20       | 192.168.102.129 | 255.255.255.128 | N/A             |
 |               | e0/2.40       | 172.16.102.1    | 255.255.255.0   | N/A             |
 | SW29          | VLAN40        | 172.16.102.10   | 255.255.255.0   | 172.16.102.1    |
 | VPC30         | e0/0          | 192.168.102.10  | 255.255.255.128 | 192.168.102.1   |
 | VPC31         | e0/0          | 192.168.102.139 | 255.255.255.128 | 192.168.102.129 |
 | R32           | e0/0          | 10.0.0.19       | 255.255.255.254 | N/A             |

#### 1.3 Таблица VLAN.

##### 1.3.1 Москва

| VLAN         | Name        | Interface assigned             |
| ------------ | ----------- | ------------------------------ |
| 1            | N/A         | N/A                            |
| 10           | VPC1        | SW3: e0/2                      |
| 20           | VPC7        | SW2: e0/2                      |
| 40           | Management  | SW2-5: vlan40, R12-13: vlan40  |
| 999          | Native      | N/A                            |

##### 1.3.2 Санкт-Петербург

| VLAN         | Name        | Interface assigned             |
| ------------ | ----------- | ------------------------------ |
| 1            | N/A         | N/A                            |
| 10           | VPC8        | SW9: e0/2                      |
| 20           | VPC11       | SW10: e0/2                     |
| 40           | Management  | SW9-10: vlan40, R16-17: vlan40 |
| 999          | Native      | N/A                            |

##### 1.3.1 Чокурдах

| VLAN         | Name        | Interface assigned             |
| ------------ | ----------- | ------------------------------ |
| 1            | N/A         | N/A                            |
| 10           | VPC30       | SW29: e0/0                     |
| 20           | VPC31       | SW29: e0/1                     |
| 40           | Management  | SW29: vlan40, R28: vlan40      |
| 999          | Native      | N/A                            |

 ### 2. Настроите ip адреса на каждом активном порту

 - Например R23:

 ```
  R23(config)#interface e0/0
  R23(config-if)#ip address 10.0.0.31 255.255.255.254
  R23(config-if)#no shutdown
  R23(config-if)#
  R23(config-if)#exit
  R23(config)#interface e0/1
  R23(config-if)#ip address 10.0.0.32 255.255.255.254
  R23(config-if)#no shutdown
  R23(config)#interface e0/2
  R23(config-if)#ip address 10.0.0.34 255.255.255.254
  R23(config-if)#no shutdown
 ```
 - На роутерах R12,R13,R16 и R17 необходимо сменить образ на L2, чтобы иметь возможность создавать интерфейсы vlan.


### 3. Настроите каждый VPC в каждом офисе в своем VLAN.

- Настраиваю статику на VPC.

```
VPCS> ip 192.168.100.10 255.255.255.128 192.168.100.1
Checking for duplicate address...
VPCS : 192.168.100.10 255.255.255.128 gateway 192.168.100.1

VPCS> ip 192.168.100.139 255.255.255.128 192.168.100.129
Checking for duplicate address...
VPCS : 192.168.100.139 255.255.255.128 gateway 192.168.100.129

VPCS> ip 192.168.101.10 255.255.255.0 192.168.101.1
Checking for duplicate address...
VPCS : 192.168.101.10 255.255.255.0 gateway 192.168.101.1

VPCS> ip 192.168.101.139 255.255.255.128 192.168.101.129
Checking for duplicate address...
VPCS : 192.168.101.139 255.255.255.128 gateway 192.168.101.129
```

 - Настваиваю access и trunk интерфейсы на соответствующих коммутаторах.

```
SW3(config)#vlan 10,20,40
SW3(config-vlan)#exit
SW3(config)#interface e0/2
SW3(config-if)#switchport mode access
SW3(config-if)#switchport access vlan 10
SW3(config-if)#no shutdown

SW3(config)#interface range e0/0-1
SW3(config-if-range)#switchport trunk encapsulation dot1q
SW3(config-if-range)#switchport mode trunk
SW3(config-if-range)#switchport trunk
SW3(config-if-range)#switchport trunk allowed vlan 10,20,40
SW3(config-if-range)#switchport trunk native vlan 999

```

### 4. Настроите VLAN/Loopbackup interface управления для сетевых устройств.

```
SW4(config)#interface vlan 40
SW4(config-if)#ip address 172.16.100.12 255.255.255.0
SW4(config-if)#exit
SW4(config)#ip route 0.0.0.0 0.0.0.0 172.16.100.1
```

### 5. Настроите сети офисов так, чтобы не возникало broadcast штормов, а использование линков было максимально оптимизировано.

- На коммутаторах SW4, SW5, SW9, SW10 настраиваются PortChannel.

```
SW4(config)#interface range e0/2-3
SW4(config-if-range)#switchport trunk encapsulation dot1q
SW4(config-if-range)#switchport mode trunk
SW4(config-if-range)#switchport trunk allowed vlan 10,20,40
SW4(config-if-range)#switchport trunk native vlan 999
SW4(config-if-range)#channel-group 1 mode active
```

 - Коммутаторы SW4 и SW5 назначаются root primary и root secondary.
 ```
SW4(config)#spanning-tree vlan 10,20,40 root primary

SW5(config)#spanning-tree vlan 10,20,40 root secondary
 ```

- На R12, R13 настраивается VRRP.
```
R12(config)#interface vlan 10
R12(config-if)#vrrp 1 ip 192.168.100.1
R12(config-if)#exit
R12(config)#interface vlan 20
R12(config-if)#vrrp 2 ip 192.168.100.129
R12(config-if)#exit
R12(config)#interface vlan 40
R12(config-if)#vrrp 3 ip 172.16.100.1
R12(config-if)#exit
R12(config)#do show vrrp brief
Interface          Grp Pri Time  Own Pre State   Master addr     Group addr
Vl10               1   100 3609       Y  Backup  192.168.100.3   192.168.100.1  
Vl20               2   100 3609       Y  Backup  192.168.100.131 192.168.100.129
Vl40               3   100 3609       Y  Backup  172.16.100.3    172.16.100.1

R13(config)#do show vrrp brief
Interface          Grp Pri Time  Own Pre State   Master addr     Group addr
Vl10               1   100 3609       Y  Master  192.168.100.3   192.168.100.1  
Vl20               2   100 3609       Y  Master  192.168.100.131 192.168.100.129
Vl40               3   100 3609       Y  Master  172.16.100.3    172.16.100.1
```

 - На R16, R17 настраивается HSRP.
 ```
 R12(config-if)#interface vlan10
 R12(config-if)#standby version 2
 R12(config-if)#standby 1 ip 192.168.100.1
 R12(config-if)#standby 1 preempt

 R12(config-if)#interface vlan20
 R12(config-if)#standby version 2
 R12(config-if)#standby 2 ip 192.168.100.129
 R12(config-if)#standby 2 preempt

 R12(config-if)#interface vlan40
 R12(config-if)#standby version 2
 R12(config-if)#standby 3 ip 172.16.100.1
 R12(config-if)#standby 3 preempt

 ```
 - Проверка настройки пингами между узлами.
 ```
 VPC8> ping 192.168.101.139

 84 bytes from 192.168.101.139 icmp_seq=1 ttl=63 time=2.240 ms

 VPC8> ping 192.168.101.1  

 84 bytes from 192.168.101.1 icmp_seq=1 ttl=255 time=0.778 ms

 VPC8> ping 192.168.101.129

 84 bytes from 192.168.101.129 icmp_seq=1 ttl=255 time=0.971 ms


 VPC11> ping 192.168.101.129

 84 bytes from 192.168.101.129 icmp_seq=1 ttl=255 time=1.990 ms

 ^C
 VPC11> ping 192.168.101.1  

 84 bytes from 192.168.101.1 icmp_seq=1 ttl=255 time=1.397 ms

 VPC11> ping 192.168.101.10

84 bytes from 192.168.101.10 icmp_seq=1 ttl=63 time=2.156 ms


 ```
