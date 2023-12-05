#  BGP

## Цель

 - Настроить BGP между автономными системами
 - Организовать доступность между офисами Москва и С.-Петербург

##  Задание:

 1. Настроите eBGP между офисом Москва и двумя провайдерами - Киторн и Ламас.
 2. Настроите eBGP между провайдерами Киторн и Ламас.
 3. Настроите eBGP между Ламас и Триада.
 4. Настроите eBGP между офисом С.-Петербург и провайдером Триада.
 5. Организуете IP доступность между пограничным роутерами офисами Москва и С.-Петербург.

### 1. Общие положения.

#### 1.1 Схема сети

![](img/lab10.png)

#### 1.2 Таблица адресации


| Device        | Interface     | IP address      | Subnet mask     | Default gateway |
| ------------- | ------------- | --------------- | --------------- | --------------- |
| R14           | e0/0          | 10.0.0.1        | 255.255.255.254 | N/A             |
|               | e0/1          | 10.0.0.7        | 255.255.255.254 | N/A             |
|               | e0/2          | 10.0.0.8        | 255.255.255.254 | N/A             |
|               | e0/3          | 10.0.0.10       | 255.255.255.254 | N/A             |
| R15           | e0/0          | 10.0.0.5        | 255.255.255.254 | N/A             |
|               | e0/1          | 10.0.0.3        | 255.255.255.254 | N/A             |
|               | e0/2          | 10.0.0.12       | 255.255.255.254 | N/A             |
|               | e0/3          | 10.0.0.14       | 255.255.255.254 | N/A             |
| R18           | e0/0          | 10.0.0.17       | 255.255.255.254 | N/A             |
|               | e0/1          | 10.0.0.21       | 255.255.255.254 | N/A             |
|               | e0/2          | 10.0.0.22       | 255.255.255.254 | N/A             |
|               | e0/3          | 10.0.0.24       | 255.255.255.254 | N/A             |
| R21           | e0/0          | 10.0.0.13       | 255.255.255.254 | N/A             |
|               | e0/1          | 10.0.0.26       | 255.255.255.254 | N/A             |
|               | e0/2          | 10.0.0.28       | 255.255.255.254 | N/A             |
| R22           | e0/0          | 10.0.0.9        | 255.255.255.254 | N/A             |
|               | e0/1          | 10.0.0.27       | 255.255.255.254 | N/A             |
|               | e0/2          | 10.0.0.30       | 255.255.255.254 | N/A             |
| R24           | e0/0          | 10.0.0.29       | 255.255.255.254 | N/A             |
|               | e0/1          | 10.0.0.36       | 255.255.255.254 | N/A             |
|               | e0/2          | 10.0.0.35       | 255.255.255.254 | N/A             |
|               | e0/3          | 10.0.0.23       | 255.255.255.254 | N/A             |

### 2. Настройка BGP.
#### 2.1 Настройка eBGP между офисом Москва и двумя провайдерами - Киторн и Ламас.

 - Настройка R14.
```
R14(config)#router bgp 1001
R14(config-router)# neighbor 10.0.0.9 remote-as 101
```
 - Настройка R22
```
R22(config)#router bgp 101
R22(config-router)# neighbor 10.0.0.8 remote-as 1001
```
 - Проверка, сосед виден.
 ```
 R14(config)#do show ip bgp summary
BGP router identifier 10.0.0.10, local AS number 1001
BGP table version is 1, main routing table version 1

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
10.0.0.9        4          101      33      36        1    0    0 00:29:28        0
 ```
 - Настройка R15
 ```
 R15(config)#router bgp 1001
 R15(config-router)# neighbor 10.0.0.13 remote-as 301
 ```
 - Настройка R21
```
R22(config)#router bgp 301
R22(config-router)# neighbor 10.0.0.12 remote-as 1001
```
- Проверка, сосед виден.
```
R15#show ip bgp summary
BGP router identifier 10.0.0.14, local AS number 1001
BGP table version is 1, main routing table version 1

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
10.0.0.13       4          301      16      19        1    0    0 00:13:42        0
```
#### 2.2 Настроика eBGP между провайдерами Киторн и Ламас.
- Настройка R21.
```
R21(config)#router bgp 301
R21(config-router)# neighbor 10.0.0.27 remote-as 101
```
- Настройка R22
```
R22(config)#router bgp 101
R22(config-router)# neighbor 10.0.0.26 remote-as 301
```
- Проверка, сосед виден.
```
R21#show ip bgp summary
BGP router identifier 10.0.0.28, local AS number 301
BGP table version is 1, main routing table version 1

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
10.0.0.12       4         1001      22      20        1    0    0 00:16:47        0
10.0.0.27       4          101    1548    1546        1    0    0 23:19:46        0
```

#### 2.3 Настроика eBGP между Ламас и Триада.
- Настройка R21.
```
R21(config)#router bgp 301
R21(config-router)# neighbor 10.0.0.29 remote-as 520
```
- Настройка R24
```
R24(config)#router bgp 520
R24(config-router)# neighbor 10.0.0.28 remote-as 301
```
- Проверка, сосед виден.
```
R21(config)#do show ip bgp su
BGP router identifier 10.0.0.28, local AS number 301
BGP table version is 1, main routing table version 1

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
10.0.0.12       4         1001      33      31        1    0    0 00:26:41        0
10.0.0.27       4          101    1559    1557        1    0    0 23:29:40        0
10.0.0.29       4          520       4       2        1    0    0 00:00:21        0

```

#### 2.4 Настроика eBGP между офисом С.-Петербург и провайдером Триада.

- Настройка R18.
```
R18(config)#router bgp 2042
R18(config-router)# neighbor 10.0.0.23 remote-as 520
```
- Настройка R24
```
R24(config)#router bgp 520
R24(config-router)# neighbor 10.0.0.22 remote-as 2042
```
- Проверка, сосед виден.
```
R18(config)#do show ip bgp summary
BGP router identifier 10.0.0.24, local AS number 2042
BGP table version is 1, main routing table version 1

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
10.0.0.23       4          520       2       2        1    0    0 00:00:14        0
```
#### 2.5 Организация IP доступности между пограничным роутерами офисами Москва и С.-Петербург.
- Смотрим таблицу маршрутизации на R14 и сети, известные bgp.
```
R14#show ip route
...
      10.0.0.0/8 is variably subnetted, 8 subnets, 2 masks
C        10.0.0.0/31 is directly connected, Ethernet0/0
L        10.0.0.1/32 is directly connected, Ethernet0/0
C        10.0.0.6/31 is directly connected, Ethernet0/1
L        10.0.0.7/32 is directly connected, Ethernet0/1
C        10.0.0.8/31 is directly connected, Ethernet0/2
L        10.0.0.8/32 is directly connected, Ethernet0/2
C        10.0.0.10/31 is directly connected, Ethernet0/3
L        10.0.0.10/32 is directly connected, Ethernet0/3

R14#show ip bgp
R14#
```
- Только local и connected.
- На R18 аналогично.
```
R18#show ip route


      10.0.0.0/8 is variably subnetted, 8 subnets, 2 masks
C        10.0.0.16/31 is directly connected, Ethernet0/0
L        10.0.0.17/32 is directly connected, Ethernet0/0
C        10.0.0.20/31 is directly connected, Ethernet0/1
L        10.0.0.21/32 is directly connected, Ethernet0/1
C        10.0.0.22/31 is directly connected, Ethernet0/2
L        10.0.0.22/32 is directly connected, Ethernet0/2
C        10.0.0.24/31 is directly connected, Ethernet0/3
L        10.0.0.24/32 is directly connected, Ethernet0/3

```
- Добавлю 10.0.0.8/31 в bgp.
```
R14(config)#router bgp 1001
R14(config-router)#network 10.0.0.8 mask 255.255.255.254
R14(config-router)#exit
R14(config)#do show ip bgp  
BGP table version is 2, local router ID is 10.0.0.10
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter,
              x best-external, a additional-path, c RIB-compressed,
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 *>  10.0.0.8/31      0.0.0.0                  0         32768 i
R14(config)#

```
- На R18.
```
R18#show ip route

      10.0.0.0/8 is variably subnetted, 9 subnets, 2 masks
B        10.0.0.8/31 [20/0] via 10.0.0.23, 00:01:08
C        10.0.0.16/31 is directly connected, Ethernet0/0
L        10.0.0.17/32 is directly connected, Ethernet0/0
C        10.0.0.20/31 is directly connected, Ethernet0/1
L        10.0.0.21/32 is directly connected, Ethernet0/1
C        10.0.0.22/31 is directly connected, Ethernet0/2
L        10.0.0.22/32 is directly connected, Ethernet0/2
C        10.0.0.24/31 is directly connected, Ethernet0/3
L        10.0.0.24/32 is directly connected, Ethernet0/3
R18#show ip bgp
BGP table version is 2, local router ID is 10.0.0.24
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter,
              x best-external, a additional-path, c RIB-compressed,
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 *>  10.0.0.8/31      10.0.0.23                              0 520 301 101 1001 i

```
- Объявляю сеть на R18 и пингую R14.
```
R18(config)#router bgp 2042
R18(config-router)#network 10.0.0.22 mask
R18(config-router)#network 10.0.0.22 mask 255.255.255.254

R18#ping 10.0.0.8
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.0.0.8, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/3 ms

```
- Пингую R18 с R14
```
R14#ping 10.0.0.22
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.0.0.22, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/2 ms

```
- Теперь R15. В принципе, можно на R21 сделать redistribute connected.
```
R21(config)#router bgp 301
R21(config-router)#redistribute connected
```
- Пинги пошли.
```
R15#ping 10.0.0.22
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.0.0.22, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/2/6 ms

R18#ping 10.0.0.12
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.0.0.12, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/2 ms

```
