
---

---

### 3. 시스코 실습 전용 문서 (`Day04/Cisco/README.md`)

```markdown
# [Cisco] EIGRP 라우팅 및 Router-on-a-Stick·DHCP 실습

---

### 1. 스위치 VLAN 및 포트 모드 설정

**VLAN 생성 및 명명**
```cisco
Switch> enable
Switch# configure terminal

Switch(config)# vlan 10
Switch(config-vlan)# name Sales
Switch(config-vlan)# vlan 20
Switch(config-vlan)# name Tech
Switch(config-vlan)# exit

```

**Access 포트 할당 (PC 연결 포트)**

```cisco
Switch(config)# interface FastEthernet0/1
Switch(config-if)# switchport mode access
Switch(config-if)# switchport access vlan 10
Switch(config-if)# exit

```

**Trunk 포트 설정 (스위치 ↔ 라우터/스위치 업링크)**

```cisco
Switch(config)# interface FastEthernet0/3
Switch(config-if)# switchport mode trunk
Switch(config-if)# exit

```

**스위치 구성 검증**

```cisco
Switch# show vlan brief        ! 생성된 VLAN 및 포트 매핑 확인
Switch# show interfaces trunk  ! 활성화된 트렁크 포트 및 캡슐화 확인

```

---

### 2. 라우터 Inter-VLAN 구성 (Router-on-a-Stick)

```cisco
Router> enable
Router# configure terminal

! 1. 물리 인터페이스 활성화
Router(config)# interface GigabitEthernet0/0
Router(config-if)# no shutdown
Router(config-if)# exit

! 2. VLAN 10 가상 서브인터페이스
Router(config)# interface GigabitEthernet0/0.10
Router(config-subif)# encapsulation dot1Q 10
Router(config-subif)# ip address 192.168.200.50 255.255.255.192
Router(config-subif)# exit

! 3. VLAN 20 가상 서브인터페이스
Router(config)# interface GigabitEthernet0/0.20
Router(config-subif)# encapsulation dot1Q 20
Router(config-subif)# ip address 192.168.200.100 255.255.255.192
Router(config-subif)# exit

```

---

### 3. DHCP 서버 및 Relay Agent 설정

**라우터 로컬 DHCP Server 구성**

```cisco
! 1. 게이트웨이 및 서버용 고정 IP 할당 제외
Router(config)# ip dhcp excluded-address 192.168.200.1 192.168.200.10
Router(config)# ip dhcp excluded-address 192.168.200.50

! 2. DHCP Pool 생성 및 네트워크 정보 정의
Router(config)# ip dhcp pool MY_POOL
Router(dhcp-config)# network 192.168.200.0 255.255.255.192
Router(dhcp-config)# default-router 192.168.200.50
Router(dhcp-config)# dns-server 8.8.8.8
Router(dhcp-config)# exit

```

**DHCP Relay Agent 구성 (원격지 DHCP 서버 중계)**

```cisco
Router(config)# interface GigabitEthernet0/0.10
Router(config-subif)# ip helper-address 10.10.10.254
Router(config-subif)# exit

```

---

### 4. EIGRP 라우팅 및 상호 재분배 구성

**기본 EIGRP 프로세스 활성화**

```cisco
Router(config)# router eigrp 10
Router(config-router)# network 192.168.10.0 0.0.0.255
Router(config-router)# network 192.168.200.0 0.0.0.63
Router(config-router)# no auto-summary
Router(config-router)# exit

```

**다중 AS 상호 재분배 (경계 라우터 ASBR)**

```cisco
! AS 10 영역으로 AS 20 정보 주입
Router(config)# router eigrp 10
Router(config-router)# redistribute eigrp 20 metric 10000 100 255 1 1500
Router(config-router)# no auto-summary
Router(config-router)# exit

! AS 20 영역으로 AS 10 정보 주입
Router(config)# router eigrp 20
Router(config-router)# redistribute eigrp 10 metric 10000 100 255 1 1500
Router(config-router)# no auto-summary
Router(config-router)# exit

```

---

### 5. EIGRP 상태 점검 및 트래픽 검증

```cisco
Router# show ip route               ! 'D'(EIGRP), 'D EX'(재분배 외부경로) 라우팅 확인
Router# show ip eigrp neighbors     ! 수립된 Neighbor 라우터 목록 확인
Router# show ip eigrp topology      ! Feasible Successor 및 토폴로지 맵 확인
Router# show ip eigrp traffic       ! Hello, Update, Query, Reply 패킷 통계 확인
Router# show ip eigrp interfaces    ! 활성화 인터페이스 및 SRTT 수치 확인
Router# show ip interface se0/3/0   ! K-Value 가중치 확인
Router# show interface se0/3/0      ! 실제 Bandwidth(BW), Delay(DLY), Reliability 확인

```

```

```