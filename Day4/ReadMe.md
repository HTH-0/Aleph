# 📡 네트워크 실습 및 이론 정리: EIGRP & VLAN

EIGRP 라우팅 프로토콜의 작동 원리, 메트릭 요소, 재분배(Redistribution) 개념과 함께 스위치 VLAN 구성, Inter-VLAN 라우팅, DHCP Server/Relay 설정 내용을 정리한 문서입니다.

---

## 📌 1. EIGRP (Enhanced Interior Gateway Routing Protocol)

### 1.1 기본 개념 및 설정

* **AS (Autonomous System) 번호**: 동일한 AS 번호를 사용하는 라우터끼리만 라우팅 정보를 교환 (AS 번호가 다르면 기본 통신 불가).
* **와일드카드 마스크 (Wildcard Mask)**: 서브넷 마스크의 반전 형태 (`255.255.255.255` - `Subnet Mask`).
* 예: 서브넷 마스크가 `255.255.255.224`일 경우 와일드카드는 `0.0.0.31`


* **`no auto-summary`**: 클래스풀(Classful) 경계에서 수행되는 자동 축약을 비활성화하여 서브네팅된 주소 정보를 그대로 유지.

```text
Router(config)# router eigrp 10
Router(config-router)# network 192.168.10.0 0.0.0.255
Router(config-router)# no auto-summary

```

* **라우팅 테이블 표기**: `show ip route` 실행 시 **`D`** 코드로 표시 (AD 값: `90`).
* *참고: RIP (AD `120`) 대비 신뢰도가 높음.*



---

### 1.2 EIGRP 메트릭 (Metric) 및 K-Value

EIGRP는 5가지 요소를 기반으로 복합 메트릭(Composite Metric)을 계산합니다.

$$\text{Default Metric} = \text{Bandwidth (K1)} + \text{Delay (K3)}$$

* **K1 (Bandwidth / 대역폭)**: 회선의 대역폭 (속도가 아닌 차선 개념)
* **K2 (Load / 부하량)**: 트래픽 송수신 부하 (`txload`, `rxload`)
* **K3 (Delay / 지연 시간)**: 데이터 전송에 걸리는 시간 (`usec`)
* **K4 (Reliability / 신뢰도)**: 링크의 안정성 (`255/255`가 최상)
* **K5 (MTU)**: 최대 전송 단위 (기본 `1500 bytes`)

#### 상태 확인 명령어

```bash
show ip interface se0/3/0   # K-Value 가중치 확인 (K1=1, K2=0, K3=1, K4=0, K5=0)
show interface se0/3/0      # 대역폭, 지연, 신뢰도, MTU 실제 수치 확인

```

---

### 1.3 EIGRP 주요 점검 명령어 & 트래픽 패킷

* `show ip eigrp topology`: 백업 경로(Feasible Successor) 및 모든 경로 지도 확인
* `show ip eigrp traffic`: 송수신된 EIGRP 패킷 통계 확인
* `show ip eigrp interfaces`: EIGRP가 활성화된 포트, 이웃(Neighbor) 수, SRTT(평균 왕복 시간) 확인

#### 질의 및 응답 패킷 (Queries & Replies)

* **Query**: 최적 경로(Successor)가 끊어졌을 때 백업 경로마저 없으면 이웃 라우터에게 대체 경로를 묻는 패킷.
* **Reply**: Query 패킷을 받은 이웃 라우터가 보내는 응답 패킷.
* **Maximum Hop Count**: 기본 `100` (최대 255까지 확장 가능, RIP의 15홉 제한 극복).

---

### 1.4 재분배 (Redistribution)

AS 번호가 서로 다르거나 라우팅 프로토콜이 다른 영역 간 통신을 위해 경계 라우터(BR: Boundary Router)에서 라우팅 정보를 양방향으로 상호 교환해 주는 설정입니다.

```text
# AS 10 영역에서 AS 20의 정보를 받아올 때 (상대 영역 인터페이스에 둘 다 적용 필요)
Router(config)# router eigrp 10
Router(config-router)# redistribute eigrp 20 metric 10000 100 255 1 1500

```

---

## VLAN (Virtual LAN) & Inter-VLAN

### 2.1 VLAN 방식 및 특징

물리적 스위치 하나를 논리적으로 분할하여 브로드캐스트 도메인을 격리하고 보안성/비용 효율성을 높입니다.

* **Port-Based (포트 기반)**: 스위치 물리 포트에 VLAN을 직접 배정하는 정석적인 방식 (가장 흔히 사용).
* **IP-Based / MAC-Based**: IP 대역 또는 MAC 주소에 따라 유연하게 배정.

---

### 2.2 스위치 포트 모드 설정

#### 1) Access Port (단일 장비 연결)

PC 등 단일 엔드포인트 장비가 연결되는 포트 설정입니다.

```text
Switch(config)# vlan 10
Switch(config-vlan)# name Sales
Switch(config-vlan)# exit

Switch(config)# interface fastEthernet 0/1
Switch(config-if)# switchport mode access
Switch(config-if)# switchport access vlan 10

```

#### 2) Trunk Port (스위치 ↔ 스위치 / 스위치 ↔ 라우터)

여러 VLAN 트래픽을 식별 태그(Tagging, IEEE 802.1Q)를 붙여 하나의 포트로 통과시키는 고속 전용 통로입니다.

```text
Switch(config)# interface fastEthernet 0/3
Switch(config-if)# switchport mode trunk

```

* **Native VLAN (Default VLAN 1)**: 태그가 붙지 않은(Untagged) 일반 트래픽을 처리하는 기본 VLAN.

#### 상태 확인

```bash
show vlan brief            # 스위치 내 생성된 VLAN 및 포트 할당 현황
show interfaces trunk      # 현재 동작 중인 트렁크 포트 및 허용 VLAN 확인

```

---

### 2.3 Router-on-a-Stick (Inter-VLAN 라우팅)

VLAN이 다르면 서로 다른 네트워크이므로 L3 장비(라우터 또는 L3 스위치)가 필수입니다. 라우터의 단일 물리 포트를 서브 인터페이스로 나눕니다.

```text
# 물리 인터페이스 전원 ON
Router(config)# interface gigabitEthernet 0/0
Router(config-if)# no shutdown
Router(config-if)# exit

# VLAN 10 전용 서브 인터페이스 설정
Router(config)# interface gigabitEthernet 0/0.10
Router(config-subif)# encapsulation dot1Q 10           # VLAN ID 태깅
Router(config-subif)# ip address 192.168.200.50 255.255.255.192
Router(config-subif)# exit

```

---

## 💡 3. DHCP (Dynamic Host Configuration Protocol) 설정

### 3.1 라우터 내부 DHCP Pool 생성

네트워크에 속한 PC들에게 IP 주소, 게이트웨이, DNS 정보를 자동으로 할당합니다.

```text
# 1. 할당 제외 IP 설정 (예: 게이트웨이, 서버 IP 등)
Router(config)# ip dhcp excluded-address 192.168.200.1 192.168.200.10

# 2. DHCP Pool 생성 및 정보 설정
Router(config)# ip dhcp pool MY_POOL
Router(dhcp-config)# network 192.168.200.0 255.255.255.0
Router(dhcp-config)# default-router 192.168.200.50
Router(dhcp-config)# dns-server 8.8.8.8

```

### 3.2 DHCP Relay Agent (서버가 다른 네트워크에 있을 때)

DHCP 서버가 원격지에 존재하여 브로드캐스트 패킷이 라우터를 넘지 못할 때, 유니캐스트로 변환하여 전달해 주는 설정입니다.

```text
Router(config)# interface gigabitEthernet 0/0.10
Router(config-subif)# ip helper-address [원격 DHCP 서버 IP]

```