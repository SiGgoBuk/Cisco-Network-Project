# Cisco-Network-Project
# 🌐 CCIE-Based Network Configuration Project

> CCIE 기반 네트워크 설계 및 구성 — 라우터 6대 · 스위치 4대

<br>

## 개요

| 항목 | 내용 |
|------|------|
| **기간** | 2026.02 |
| **플랫폼** | Web-IOU |
| **규모** | Router 6대 (R1~R6), Switch 4대 (SW1~SW4) |
| **역할** | 조장 — 역할 분배, 자료 구성 기획, PDF 작성 |

<br>

## 구성도

### 논리적 구성도 및 물리적 구성도

image
image
image

<br>

## VLAN 구성

| VLAN ID | Name | 네트워크 |
|---------|------|---------|
| 11 | VLAN_BB1 | 150.1.YY.0/24 |
| 12 | VLAN_BB2 | 150.2.YY.0/24 |
| 13 | VLAN_BB3 | 150.3.YY.0/24 |
| 21 | VLAN_A | YY.YY.20.0/24 |
| 22 | VLAN_B | YY.YY.62.0/24 |
| 23 | VLAN_C | YY.YY.55.0/24 |
| 50 | VLAN_CUSTOMER1 | - |
| 79 | VLAN_CUSTOMER2 | - |
| 100 | VLAN_SWITCHES | YY.YY.90.0/24 |

<br>

## 설정 목차

1. [Frame Relay 구성](#1-frame-relay-구성)
2. [Bridging & Switching](#2-bridging--switching)
3. [IP IGP 라우팅 프로토콜](#3-ip-igp-라우팅-프로토콜)
4. [Redistribution (재분배)](#4-redistribution-재분배)
5. [보안 및 기타 설정](#5-보안-및-기타-설정)

<br>

---

## 1. Frame Relay 구성

### 구성 방식

| 구간 | 방식 | DLCI |
|------|------|------|
| R1 ↔ R3 | Point-to-Point | 103 / 301 |
| R2 ↔ R4 ↔ R5 | Multipoint | 205 / 405 / 502 / 504 |
| R2 ↔ R6 | Point-to-Point | 206 / 602 |

```cisco
! R1
interface s1/0
 encapsulation frame-relay
 no frame-relay inverse-arp
 no shutdown

interface s1/0.103 point-to-point
 ip address 14.14.13.1 255.255.255.0
 frame-relay interface-dlci 103

! R2
interface s1/0
 encapsulation frame-relay
 no frame-relay inverse-arp
 no shutdown

interface s1/0.245 multipoint
 ip address 14.14.245.2 255.255.255.0
 frame-relay map ip 14.14.245.5 205 broadcast
 frame-relay map ip 14.14.245.4 205 broadcast
 frame-relay map ip 14.14.245.2 205
 ip ospf network broadcast

interface s1/0.206 point-to-point
 ip address 14.14.26.2 255.255.255.0
 frame-relay interface-dlci 206

! R3
interface s1/0
 encapsulation frame-relay
 no frame-relay inverse-arp
 no shutdown

interface s1/0.301 point-to-point
 ip address 14.14.13.3 255.255.255.0
 frame-realy interface-dlci 301

! R4
interface s1/0
 encapsulation frame-relay
 no frame-relay inverse-arp
 no shutdown

interface s1/0.245 multipoint
 ip address 14.14.245.4 255.255.255.0
 frame-relay map ip 14.14.245.5 405 broadcast
 frame-relay map ip 14.14.245.2 405 broadcast
 frame-relay map ip 14.14.245.4 405
 ip ospf network broadcast

! R5
interface s1/0
 encapsulation frame-relay
 no frame-relay inverse-arp
 no shutdown

interface s1/0.245 multipoint
 ip address 14.14.245.5 255.255.255.0
 frame-relay map ip 14.14.245.2 502 broadcast
 frame-relay map ip 14.14.245.4 504 broadcast
 frame-relay map ip 14.14.245.5 502
 ip ospf network broadcast

! R6
interface s1/0
 encapsulation frame-relay
 no frame-relay inverse-arp
 no shutdown

interface s1/0.602 point-to-point
 ip address 14.14.26.6 255.255.255.0
 frame-relay interface-dlci 602
```

<br>

---

## 2. Bridging & Switching

### 2-1. VTP 구성

SW1을 VTP Server로, SW2~SW4를 Client로 구성하여 VLAN 정보를 자동 전파

```cisco
! SW1 (Server)
vtp mode server
vtp domain history.com
vtp password history
vtp version 2

! SW2, SW3, SW4 (Client)
vtp mode client
vtp domain history.com
vtp password history
vtp version 2
```

### 2-2. Trunk Port (R6 ↔ SW2)

```cisco
! R6 — 서브인터페이스로 dot1q 트렁크 구성
interface e0/1
 no shutdown

interface e0/1.13
 encapsulation dot1q 13
 ip address 150.3.14.1 255.255.255.0

interface e0/1.22
 encapsulation dot1q 22
 ip address 14.14.62.6 255.255.255.0

! SW2
interface e1/2
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk allowed vlan 13,22
```

### 2-3, 2-4 SW 구성

스위치 간 대역폭 확장 및 이중화를 위한 EtherChannel 구성

```cisco
! SW1
interface range e3/0-1
 switchport trunk encapsulation dot1q
 switchport mode trunk
 channel-group 12 mode on    ! SW1-SW2 Po12

interface range e3/2-3
 switchport trunk encapsulation dot1q
 switchport mode trunk
 channel-group 13 mode on    ! SW1-SW3 Po13

! SW2
interface range e3/0-1
 switchport trunk encapsulation dot1q
 switchport mode trunk
 channel-group 12 mode on

interface range e3/2-3
 switchport trunk encapsulation dot1q
 switchport mode trunk
 channel-group 24 mode on

! SW3
interface range e3/0-1
 switchport trunk encapsulation dot1q
 switchport mode trunk
 channel-group 34 mode on

interface range e3/2-3
 switchport trunk encapsulation dot1q
 switchport mode trunk
 channel-group 13 mode on

! SW4
interface range e3/0-1
 switchport trunk encapsulation dot1q
 switchport mode trunk
 channel-group 34 mode on

interface range e3/2-3
 switchport trunk encapsulation dot1q
 switchport mode trunk
 channel-group 24 mode on

! SW1-SW4
interface range e2/2-3
 shutdown

! SW1-SW2 로드밸런싱
port-channel load-balance src-dst-ip
```

### 2-5. VLAN 구성

```cisco
vlan 11
 name BB1
vlan 12
 name BB2
vlan 13
 name BB3
vlan 21
 name VLAN_A
vlan 22
 name VLAN_B
vlan 23
 name VLAN_C
vlan 50
 name VLAN_CUSTOMER1
vlan 79
 name VLAN_CUSTOMER2
vlan 100
 name VLAN_SWITCHES

int e2/0
 switchport mode access
 switchport access vlan 11

int e0/0
 switchport mode access
 switchport access vlan 11

int e0/2
 switch port mode access
 switchport access vlan 22

interface e0/3
 no switchport
 ip address 14.14.33.7 255.255.255.0

interface e1/2
 no switchport
 ip address 14.14.36.7 255.255.255.0

! SW2
interface e2/0
 switchport mode access
 switchport access vlan 12

interface e1/2
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk allowed vlan 13,22

interface e0/2
 switchport mode access
 switchport access vlan 21

int e1/1
 switchport mode access
 switchport access vlan 23

! SW3
interface e2/0
 switchport mode access
 switchport access vlan 13

```

### 2-6. STP shared vlan

VLAN 그룹별 루트 스위치를 분산하여 루프 방지 및 트래픽 최적화

```cisco
! 전체 스위치 공통
spanning-tree mode mst
spanning-tree mst configuration
 name HISTORY
 revision 1
 instance 1 vlan 11,21
 instance 2 vlan 100

! SW1 — Instance 1 Root
spanning-tree mst 1 root primary

! SW4 — Instance 2 Root
spanning-tree mst 2 root primary
```

### 2-8. ﻿UDLD

```cisco
! SW3-SW4
int ran e3/0-1
 udld port aggressive
```

### 2-10. Mac address Database

```cisco
! SW3
mac address-table aging-time 500 vlan 13
```

### 2-11. Root Switch

```cisco
! SW1-SW4
spanning-tree mst configuration
 instance 3 vlan 12

! SW2
spanning-tree mst 3 root primary
```

### 2.13 Create the interfaces VLAN below VLAN 100 (VLAN_SWITCHES)

```cisco
! SW1
interface vlan 100
 no shutdown
 ip address 14.14.90.1 255.255.255.0

! SW2
interface vlan 100
 no shutdown
 ip address 14.14.90.2 255.255.255.0

! SW3
interface vlan 100
 no shutdown
 ip address 14.14.90.3 255.255.255.0

! SW4
interface vlan 100
 no shutdown
 ip address 14.14.90.4 255.255.255.0
```

<br>

---

## 3. IP IGP 라우팅 프로토콜

### 3-1. RIPv2 (R1 ↔ R3 ↔ BB1)

```cisco
! R1
router rip
 version 2
 no auto-summary
 network 150.1.0.0
 network 14.0.0.0
 passive-interface default
 neighbor 150.1.14.254
 neighbor 14.14.13.3

! R3
router rip
 version 2
 no auto-summary
 network 14.0.0.0
 passive-interface default
 neighbor 14.14.13.1
```

### 3-2. EIGRP YY (스위치 구간)

```cisco
! SW1
ip routing
router eigrp 14
 no auto-summary
 network 14.14.90.1 0.0.0.0        ! VLAN_SWITCHES SVI

! SW2
ip routing
router eigrp 14
 no auto-summary
 network 14.14.8.8 0.0.0.0         ! Loopback0
 network 14.14.90.2 0.0.0.0        ! VLAN_SWITCHES SVI

! SW3
ip routing
router eigrp 14
 no auto-summary
 network 14.14.9.9 0.0.0.0         ! Loopback0
 network 14.14.90.3 0.0.0.0        ! VLAN_SWITCHES SVI

! SW4
ip routing
router eigrp 14
 no auto-summary
 network 14.14.10.10 0.0.0.0       ! Loopback0
 network 14.14.90.4 0.0.0.0        ! VLAN_SWITCHES SVI
```

### 3-3. EIGRP 100 (R6 ↔ BB3)

```cisco
! R6
router eigrp 100
 no auto-summary
 network 150.3.14.1 0.0.0.0
 eigrp stub redistribute
```

### 3-4. OSPF Multi-Area

```cisco
! Area 0
! R2
router ospf 14
 network 14.14.245.2 0.0.0.0 area 0

int s1/0.245
 ip ospf priority 0          ! DR 선출 방지

! R6
router ospf 14
 network 14.14.36.6 0.0.0.0 area 0

! R4, R5 동일하게 priority 0 설정
! SW1
router ospf 14
 network 14.14.36.7 0.0.0.0 area 0

! Area 3
! R3
router ospf 14
 network 14.14.3.3 0.0.0.0 area 3
 network 14.14.33.3 0.0.0.0 area 3

! SW1
router ospf 14
 network 14.14.33.7 0.0.0.0 area 3
 network 14.14.7.7 0.0.0.0 area 3

! Area 4 — Totally Stub
! R2
router ospf 14
 net 14.14.2.2 0.0.0.0 area 4
 net 14.14.20.2 0.0.0.0 area 4
 area 4 stub no-summary

! Area 5 — Stub
! R5
router ospf 14
 net 14.14.55.5 0.0.0.0 area 5
 net 14.14.5.5 0.0.0.0 area 5
 area 5 stub

! Area 26 — Virtual-Link (비연속 Area 26을 Backbone에 연결)
! R2
router ospf 14
 network 14.14.62.2 0.0.0.0 area 26
 network 14.14.26.2 0.0.0.0 area 26
 area 26 virtual-link 14.14.6.6

int s1/0.206
 ip ospf network point-to-point
int e0/0
 ip ospf network point-to-point

! R6
router ospf 14
 network 14.14.62.6 0.0.0.0 area 26
 network 14.14.26.6 0.0.0.0 area 26
 area 26 virtual-link 14.14.2.2

int s1/0.602
 ip ospf network point-to-point
int e0/1.22
 ip ospf network point-to-point
```

> **Loopback /32 방지**: OSPF 라우팅 테이블에 /32가 아닌 /24로 표시되도록 설정
```cisco
! R1~R6, SW1~SW4 공통
interface lo0
 ip ospf network point-to-point
```

<br>

---

## 4. Redistribution (재분배)

복수의 라우팅 프로토콜 간 경로 정보를 상호 교환

```
RIPv2 ←→ OSPF (R3)
OSPF ←→ EIGRP YY (SW1)
OSPF ←→ EIGRP 100 (R6)
```

```cisco
! R3 — RIP ↔ OSPF 재분배
router rip
 redistribute ospf 14 metric 3

router ospf 14
 redistribute rip subnets

! SW1 — OSPF ↔ EIGRP YY 재분배
router eigrp 14
 redistribute ospf 14 metric 1 1 1 1 1

router ospf 14
 redistribute eigrp 14 subnets

! R6 — OSPF ↔ EIGRP 100 재분배
router ospf 14
 redistribute eigrp 100 subnets

router eigrp 100
 redistribute ospf 14 metric 1 1 1 1 1
 eigrp stub redistribute
```

<br>

---

## 5. 기타 설정

### 5-2. DHCP 서버 (R6)

```cisco
ip dhcp excluded-address 14.14.62.6
ip dhcp excluded-address 14.14.62.2

ip dhcp pool DHCP
 network 14.14.62.0 255.255.255.0
 dns-server 150.100.1.50 150.100.1.51
 domain-name history.com
 lease infinite
 default-router 14.14.62.6 14.14.62.2
```

### 5-3. UDP Broadcast 중계 (ip helper-address)

```cisco
! R6 — VLAN_BB3 구간의 DHCP 요청을 BB2로 중계
ip forward-protocol udp bootpc

interface e0/1.13
 ip helper-address 150.2.14.244
```

## 6. 보안

### 6-1. OSPF MD5 인증 (Area 0)

```cisco
! R2
router ospf 14
 area 0 authentication message-digest
 area 26 virtual-link 14.14.6.6 message-digest-key 1 md5 history

interface s1/0
 ip ospf message-digest-key 1 md5 history

! R4, R5, SW1, R6 동일하게 적용
```


<br>

---

## 검증 명령어

```cisco
! 라우팅 테이블 확인
show ip route

! OSPF 이웃 확인
show ip ospf neighbor

! EIGRP 이웃 확인
show ip eigrp neighbors

! VTP 상태 확인
show vtp status

! VLAN 확인
show vlan brief

! EtherChannel 확인
show etherchannel summary

! MST 확인
show spanning-tree mst configuration
show spanning-tree vlan 11
```

<br>

## 참고

- Web-IOU 기반 구성
- Rack number: 14 (YY = 14)
- 팀명: HISTORY
- 팀 구성: 조장 김동진 / 조원 장규혁, 이영훈, 정성현
