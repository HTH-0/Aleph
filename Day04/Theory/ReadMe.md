# [Theory] EIGRP 라우팅 프로토콜 및 VLAN·DHCP 이론

---

### 1. EIGRP (Enhanced Interior Gateway Routing Protocol)

**기본 특징 및 동작 원리**
* **AS (Autonomous System) 번호:** 동일한 AS 번호를 공유하는 라우터끼리만 라우팅 정보를 교환 (AS 번호 불일치 시 Neighbor 수립 불가)
* **와일드카드 마스크 (Wildcard Mask):** 서브넷 마스크의 반전 형태 (`255.255.255.255` - `Subnet Mask`)
  * 예: `255.255.255.224` (/27) ➔ 와일드카드 마스크 `0.0.0.31`
* **`no auto-summary`:** 클래스풀(Classful) 경계에서 실행되는 자동 축약을 비활성화하여 서브네팅 정보를 보존
* **AD(Administrative Distance) 값:** `90` (내부 경로 기준, RIP의 `120` 대비 높은 신뢰도) / 라우팅 테이블 표기 코드: `D`

**EIGRP 복합 메트릭 (Composite Metric) 및 K-Value**
* 기본 메트릭 산정 공식: Bandwidth (K1) + Delay (K3)
* K-Value 가중치 요소 (기본값: K1=1, K2=0, K3=1, K4=0, K5=0)
  * **K1 (Bandwidth / 대역폭):** 경로 상 가장 낮은 대역폭 (단위: kbps)
  * **K2 (Load / 부하량):** 회선의 송수신 트래픽 부하 (`txload`, `rxload`)
  * **K3 (Delay / 지연 시간):** 경로 상 지연 시간의 총합 (`usec`)
  * **K4 (Reliability / 신뢰도):** 링크의 데이터 전송 성공률 (`255/255`가 최상)
  * **K5 (MTU):** 최대 전송 단위 (기본 `1500 bytes`)

**EIGRP 경로 제어 및 패킷 동작**
* **Successor (최적 경로) vs Feasible Successor (백업 경로):** DUAL 알고리즘을 통해 계산된 즉시 절체 가능 예비 경로
* **Query (질의 패킷):** 최적 경로가 다운되고 백업 경로가 없을 때 인접 이웃 라우터에게 대체 경로를 요청
* **Reply (응답 패킷):** Query 패킷에 대한 응답
* **Maximum Hop Count:** 기본 `100` (최대 255 확장 가능, RIP의 15홉 제한 극복)

**라우팅 재분배 (Redistribution)**
* 서로 다른 AS 번호 또는 서로 다른 라우팅 프로토콜 영역 간 통신을 위해 경계 라우터(ASBR)에서 라우팅 정보를 양방향으로 상호 교환해 주는 기술

---

### 2. VLAN (Virtual LAN) & Inter-VLAN

**VLAN 분할 방식 및 특징**
* 물리적 단일 스위치를 논리적인 독립 네트워크로 분할하여 브로드캐스트 도메인을 격리하고 보안성을 향상
* **Port-Based (포트 기반):** 스위치의 특정 물리 포트에 VLAN 번호를 지정하는 방식 (표준 방식)
* **IP-Based / MAC-Based:** 단말의 IP 또는 MAC 주소를 기반으로 VLAN을 동적 할당

**스위치 포트 모드**
* **Access Port:** 단일 VLAN 트래픽만 전송 (PC, 서버 등 End-Device 연결용)
* **Trunk Port:** 복수의 VLAN 트래픽을 단일 링크로 전송 (스위치-스위치, 스위치-라우터 간 연결)
* **IEEE 802.1Q:** 트렁크 링크를 통과하는 이더넷 프레임에 VLAN 식별 태그(Tag)를 삽입하는 표준 프로토콜
* **Native VLAN:** 태그가 붙지 않은(Untagged) 프레임을 처리하는 기본 VLAN (기본값: VLAN 1)

**Router-on-a-Stick (Inter-VLAN 라우팅)**
* VLAN 간 통신은 서로 다른 네트워크 간 통신이므로 L3 장비가 필수
* 라우터의 물리 포트 하나를 논리적 서브인터페이스(Sub-Interface)로 분할하여 각 VLAN의 기본 게이트웨이로 활용

---

### 3. DHCP (Dynamic Host Configuration Protocol)

**라우터 기반 DHCP 서비스**
* 단말 PC에 IP 주소, 서브넷 마스크, Default Gateway, DNS 서버 주소를 자동으로 대여(Lease)
* **IP 제외(Excluded-address):** 게이트웨이 및 서버 등 고정 IP가 중복 할당되지 않도록 사전 배제 처리 필수

**DHCP Relay Agent (`ip helper-address`)**
* DHCP 요청(Discover)은 브로드캐스트 패킷이므로 기본적으로 라우터를 통과하지 못함
* DHCP 서버가 다른 원격 네트워크에 위치할 경우, 라우터가 해당 브로드캐스트를 유니캐스트로 변환하여 원격 서버로 중계해 주는 기능

```
