---

# [실습 01] Cisco 실습
### VLAN · DHCP · EIGRP 상호 재분배 네트워크 구성 개요

---

### 1. 기본 네트워크 설계 및 사전 계획

**IP 서브넷 설계 (왼쪽 VLAN 구역)**

* 네트워크 대역: `192.168.200.0/26` (서브넷 마스크: `255.255.255.192`)
* VLAN 10 (Marketing): `192.168.200.0 ~ 63` / GW: `192.168.200.50`
* VLAN 20 (Management): `192.168.200.64 ~ 127` / GW: `192.168.200.100`
* VLAN 30 (Information): `192.168.200.128 ~ 191` / GW: `192.168.200.150`
* 예비 대역: `192.168.200.192 ~ 255` (미사용)

**구역별 구성 요약**

* **왼쪽 구역**: Router-on-a-Stick 기반 다중 VLAN 구성 (고정 IP)
* **중앙 구역**: `172.16.100.0/24` 단일 네트워크 (고정 IP)
* **오른쪽 구역**: `10.10.10.0/24` 단일 네트워크 (라우터 기반 DHCP 자동 할당)
* **라우팅 영역**: 왼쪽/중앙 = EIGRP AS 100, 오른쪽 = EIGRP AS 200 (오른쪽 라우터를 경계로 상호 재분배)

---

### 2. 왼쪽 구역 설정 (VLAN & Router-on-a-Stick)

#### 1) 스위치 설정

* **VLAN 생성 (연결된 모든 스위치 공통 작업)**

```cisco
Switch(config)# vlan 10
Switch(config-vlan)# name Marketing
Switch(config-vlan)# vlan 20
Switch(config-vlan)# name Management
Switch(config-vlan)# vlan 30
Switch(config-vlan)# name Information

```

* **PC 연결 포트 (Access 모드)**

```cisco
Switch(config)# interface FastEthernet0/2
Switch(config-if)# switchport mode access
Switch(config-if)# switchport access vlan 10

```

* **스위치 간 및 라우터 연결 포트 (Trunk 모드)**

```cisco
Switch(config)# interface FastEthernet0/4
Switch(config-if)# switchport mode trunk

```

#### 2) 라우터 서브인터페이스 설정

```cisco
Router(config)# interface GigabitEthernet0/0
Router(config-if)# no shutdown

Router(config)# interface GigabitEthernet0/0.10
Router(config-subif)# encapsulation dot1Q 10
Router(config-subif)# ip address 192.168.200.50 255.255.255.192

Router(config)# interface GigabitEthernet0/0.20
Router(config-subif)# encapsulation dot1Q 20
Router(config-subif)# ip address 192.168.200.100 255.255.255.192

Router(config)# interface GigabitEthernet0/0.30
Router(config-subif)# encapsulation dot1Q 30
Router(config-subif)# ip address 192.168.200.150 255.255.255.192

```

---

### 3. 오른쪽 구역 설정 (DHCP 서버)

```cisco
Router(config)# interface GigabitEthernet0/0
Router(config-if)# ip address 10.10.10.1 255.255.255.0
Router(config-if)# no shutdown

! 게이트웨이 IP 중복 할당 방지
Router(config)# ip dhcp excluded-address 10.10.10.1

! DHCP 풀 생성
Router(config)# ip dhcp pool LAN_POOL
Router(dhcp-config)# network 10.10.10.0 255.255.255.0
Router(dhcp-config)# default-router 10.10.10.1
Router(dhcp-config)# dns-server 8.8.8.8

```

---

### 4. 라우팅 프로토콜 설정 (EIGRP & 재분배)

#### 1) 왼쪽 라우터 (AS 100)

```cisco
Router(config)# router eigrp 100
Router(config-router)# network 1.1.1.0 0.0.0.255
Router(config-router)# network 192.168.200.0 0.0.0.63
Router(config-router)# network 192.168.200.64 0.0.0.63
Router(config-router)# network 192.168.200.128 0.0.0.63
Router(config-router)# no auto-summary

```

#### 2) 중앙 라우터 (AS 100)

```cisco
Router(config)# router eigrp 100
Router(config-router)# network 1.1.1.0 0.0.0.255
Router(config-router)# network 172.16.100.0 0.0.0.255
Router(config-router)# network 2.2.2.0 0.0.0.255
Router(config-router)# no auto-summary

```

*(DCE 시리얼 포트인 경우 `clock rate 64000` 입력 필수)*

#### 3) 오른쪽 경계 라우터 (ASBR: AS 100 & AS 200 상호 재분배)

```cisco
! AS 100 (중앙 라우터 구간)
Router(config)# router eigrp 100
Router(config-router)# network 2.2.2.0 0.0.0.255
Router(config-router)# redistribute eigrp 200
Router(config-router)# no auto-summary

! AS 200 (오른쪽 LAN 구간)
Router(config)# router eigrp 200
Router(config-router)# network 10.10.10.0 0.0.0.255
Router(config-router)# redistribute eigrp 100
Router(config-router)# no auto-summary

```

---

### 5. 트러블슈팅 및 검증 체크포인트

* **스위치 포트 점검**
  * `show vlan brief`: Access 포트가 지정된 VLAN 번호에 정상 매핑되어 있는지 확인
  * `show interfaces trunk`: 스위치 간 연결선 및 라우터 업링크 포트(`Fa0/5` 등)가 Trunk 목록에 누락되지 않았는지 확인


  * **라우팅 및 재분배 점검**
  * `show ip route`: EIGRP 내부 경로(`D`)와 재분배된 외부 경로(`D EX`)가 라우팅 테이블에 모두 올라와 있는지 확인


  * **End-to-End 검증**
  * 왼쪽 VLAN PC(`192.168.200.10`) -> 오른쪽 DHCP PC(`10.10.10.2`) Ping 및 `tracert 10.10.10.2` 실행으로 홉 경로 일치 여부 확인
