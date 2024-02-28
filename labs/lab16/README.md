#  IPSec over DmVPN


## Цель

- Настроить GRE поверх IPSec между офисами Москва и С.-Петербург
- Настроить DMVPN поверх IPSec между офисами Москва и Чокурдах, Лабытнанги

##  Задание:

1. Настроите GRE поверх IPSec между офисами Москва и С.-Петербург.
2. Настроите DMVPN поверх IPSec между Москва и Чокурдах, Лабытнанги.

- Все узлы в офисах в лабораторной работе должны иметь IP связность.
- План работы и изменения зафиксированы в документации.

### 1. Общие положения.

#### 1.1 Схема сети

![](img/lab16.png)

#### 1.2 Таблица адресации


| Device        | Interface     | IP address      | Subnet mask     |
| ------------- | ------------- | --------------- | --------------- |
| R13           | VLAN10        | 192.168.100.3   | 255.255.255.128 |
|               | VLAN20        | 192.168.100.131 | 255.255.255.128 |
|               | VLAN40        | 172.16.100.3    | 255.255.255.0   |
|               | e0/2          | 10.0.0.4        | 255.255.255.254 |
|               | e0/3          | 10.0.0.6        | 255.255.255.254 |
|               | lo1           | 10.1.0.13       | 255.255.255.255 |
| R16           | vlan10        | 192.168.101.2   | 255.255.255.128 |
|               | vlan20        | 192.168.101.130 | 255.255.255.128 |
|               | vlan40        | 172.16.101.2    | 255.255.255.0   |
|               | e0/1          | 10.0.0.16       | 255.255.255.254 |
|               | e0/3          | 10.0.0.18       | 255.255.255.254 |
|               | lo1           | 10.1.0.16       | 255.255.255.255 |
| R27           | e0/0          | 10.0.0.39       | 255.255.255.254 |
| R28           | e0/0          | 10.0.0.45       | 255.255.255.254 |
|               | e0/1          | 10.0.0.43       | 255.255.255.254 |
|               | e0/2.10       | 192.168.102.1   | 255.255.255.128 |
|               | e0/2.20       | 192.168.102.129 | 255.255.255.128 |
|               | e0/2.40       | 172.16.102.1    | 255.255.255.0   |
|               | lo1           | 10.1.0.28       | 255.255.255.255 |

#### 1.3 GRE Tunnel IP.

| Interface     | IP address     | Tunnel source   | Tunnel destination |
|---------------|----------------|-----------------|--------------------|
| tunnel0@R13   | 10.21.0.1/24   | 10.1.0.13       | 10.1.0.16          |
| tunnel0@R16   | 10.21.0.2/24   | 10.1.0.16       | 10.1.0.13          |

#### 1.3 DMVMN Tunnel IP.

| Interface     | IP address     | Tunnel source   | Tunnel destination |
|---------------|----------------|-----------------|--------------------|
| tunnel1@R13   | 10.21.1.1/24   | e0/2            | 10.0.0.22          |
| tunnel1@R28   | 10.21.1.2/24   | 10.0.0.45       | 10.10.0.12         |
| tunnel1@R27   | 10.21.1.3/24   | 10.0.0.39       | 10.10.0.12         |


### 2. Настройка.
#### 2.1 Настроите GRE поверх IPSec между офисами Москва и С.-Петербург.
- Туннель будет между R13 и R16. Предварительно настрою loopback-интерфейсы и передам маршруты о них для обеспечения связности конечных точек.
```
R13(config)#interface loopback 1                
R13(config-if)#ip address 10.1.0.13 255.255.255.255
R13(config-if)#no shutdown
R13(config-if)#ip ospf 1 area 10

R15(config)#router bgp 1001
R15(config-router)#network 10.1.0.13 mask 255.255.255.255

R16(config)#interface loopback 1
R16(config-if)#ip address 10.1.0.16 255.255.255.255


R18(config)#router bgp 2042
R18(config-router)#network 10.1.0.16 mask 255.255.255.255

R18(config)#ip prefix-list NO-TRANSIT seq 50 permit 10.1.0.16/32

R13>ping 10.1.0.16
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.1.0.16, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 2/2/4 ms


R16>ping 10.1.0.13
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.1.0.13, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 2/2/4 ms

```
- Создаю Gre-туннель между R13 и R16 на Loopback-интерфейсах. (это всё настраивалось далеко не с лёту, поэтому дальше буду приводить вставки конфигов)

```
R13:
interface Tunnel1
 ip address 10.21.0.1 255.255.255.0
 ip mtu 1400
 ip tcp adjust-mss 1360
 tunnel source 10.1.0.13
 tunnel destination 10.1.0.16

R16:
interface Tunnel1
 ip address 10.21.0.2 255.255.255.0
 ip mtu 1400
 ip tcp adjust-mss 1360
 tunnel source 10.1.0.16
 tunnel destination 10.1.0.13

```
- Дальше настраивается ikev2.
```
R13:
crypto ikev2 proposal PH1
 encryption aes-cbc-128
 integrity sha256
 group 2
!
crypto ikev2 policy IK2POL
 proposal PH1
!
!
crypto ikev2 profile PROF1
 match address local 10.1.0.13
 match identity remote address 10.1.0.16 255.255.255.255
 authentication remote pre-share key PSKEY
 authentication local pre-share key PSKEY

R16:
crypto ikev2 proposal PH1
 encryption aes-cbc-128
 integrity sha256
 group 2
crypto ikev2 policy IK2POL
 proposal PH1
crypto ikev2 profile PROF1
 match address local 10.1.0.16
 match identity remote address 10.1.0.13 255.255.255.255
 authentication remote pre-share key PSKEY
 authentication local pre-share key PSKEY
```
- Затем нужно создать "ipsec transform-set" и "ipsec profile". На обоих узлах одинаково.
```
crypto ipsec transform-set IPSEC_TS esp-aes esp-sha256-hmac
 mode transport
crypto ipsec profile protect-gre
 set transform-set IPSEC_TS
 set ikev2-profile PROF1
```
- Осталось применить ipsec profile на интерфейсе Tunnel 1 обоих узлов.
```
interface Tunnel1
tunnel protection ipsec profile protect-gre
```
- Теперь проверяю.
```
R16#ping 10.21.0.1          
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.21.0.1, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 7/7/7 ms
R16#sho crypto ikev2 sa
 IPv4 Crypto IKEv2  SA

Tunnel-id Local                 Remote                fvrf/ivrf            Status
2         10.1.0.16/500         10.1.0.13/500         none/none            READY  
      Encr: AES-CBC, keysize: 128, Hash: SHA256, DH Grp:2, Auth sign: PSK, Auth verify: PSK
      Life/Active Time: 86400/5967 sec

 IPv6 Crypto IKEv2  SA

R16#sho crypto ikev2 session
 IPv4 Crypto IKEv2 Session

Session-id:1, Status:UP-ACTIVE, IKE count:1, CHILD count:1

Tunnel-id Local                 Remote                fvrf/ivrf            Status
2         10.1.0.16/500         10.1.0.13/500         none/none            READY  
      Encr: AES-CBC, keysize: 128, Hash: SHA256, DH Grp:2, Auth sign: PSK, Auth verify: PSK
      Life/Active Time: 86400/5972 sec
Child sa: local selector  10.1.0.16/0 - 10.1.0.16/65535
          remote selector 10.1.0.13/0 - 10.1.0.13/65535
          ESP spi in/out: 0x2BDAB8F7/0xFDC96FDB  

 IPv6 Crypto IKEv2 Session

R16#sh crypto ipsec sa

interface: Tunnel1
    Crypto map tag: Tunnel1-head-0, local addr 10.1.0.16

   protected vrf: (none)
   local  ident (addr/mask/prot/port): (10.1.0.16/255.255.255.255/47/0)
   remote ident (addr/mask/prot/port): (10.1.0.13/255.255.255.255/47/0)
   current_peer 10.1.0.13 port 500
     PERMIT, flags={origin_is_acl,}
    #pkts encaps: 1312, #pkts encrypt: 1312, #pkts digest: 1312
    #pkts decaps: 20, #pkts decrypt: 20, #pkts verify: 20
    #pkts compressed: 0, #pkts decompressed: 0
    #pkts not compressed: 0, #pkts compr. failed: 0
    #pkts not decompressed: 0, #pkts decompress failed: 0
    #send errors 0, #recv errors 0

     local crypto endpt.: 10.1.0.16, remote crypto endpt.: 10.1.0.13
     plaintext mtu 1458, path mtu 1500, ip mtu 1500, ip mtu idb Ethernet0/1
     current outbound spi: 0xFDC96FDB(4257837019)
     PFS (Y/N): N, DH group: none

     inbound esp sas:
      spi: 0x2BDAB8F7(735754487)
        transform: esp-aes esp-sha256-hmac ,
        in use settings ={Transport, }
        conn id: 6, flow_id: 6, sibling_flags 80000000, crypto map: Tunnel1-head-0
        sa timing: remaining key lifetime (k/sec): (4191785/1024)
        IV size: 16 bytes
        replay detection support: Y
        Status: ACTIVE(ACTIVE)

     inbound ah sas:

     inbound pcp sas:

     outbound esp sas:
      spi: 0xFDC96FDB(4257837019)
        transform: esp-aes esp-sha256-hmac ,
        in use settings ={Transport, }
        conn id: 5, flow_id: 5, sibling_flags 80000000, crypto map: Tunnel1-head-0
        sa timing: remaining key lifetime (k/sec): (4191709/1024)
        IV size: 16 bytes
        replay detection support: Y
        Status: ACTIVE(ACTIVE)

     outbound ah sas:

     outbound pcp sas:
R16#sh crypto ipsec profile
IPSEC profile default
	Security association lifetime: 4608000 kilobytes/3600 seconds
	Responder-Only (Y/N): N
	PFS (Y/N): N
	Mixed-mode : Disabled
	Transform sets={
		default:  { esp-aes esp-sha-hmac  } ,
	}

IPSEC profile protect-gre
	IKEv2 Profile: PROF1
	Security association lifetime: 4608000 kilobytes/3600 seconds
	Responder-Only (Y/N): N
	PFS (Y/N): N
	Mixed-mode : Disabled
	Transform sets={
		IPSEC_TS:  { esp-aes esp-sha256-hmac  } ,
	}


```
##### 2.2 Настроите DMVPN поверх IPSec между Москва и Чокурдах, Лабытнанги.
