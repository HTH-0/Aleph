

### 3. 시스코 실습 전용 문서 (`Day03/Cisco/README.md`)

```markdown
# [Cisco] 3-Router 서브넷 분할 및 RIP 라우팅 실습

---

### 1. 실습 환경 및 서브넷 IP 설계

* **기본 대역:** `192.168.100.0/24`
* **서브넷 마스크:** `255.255.255.224` (`/27`, 8개 영역 분할)

| 구역 | 서브넷 대역 | 네트워크 / 브로드캐스트 | 가용 호스트 IP 범위 | 게이트웨이 / 인터페이스 IP |
| :--- | :--- | :--- | :--- | :--- |
| **LAN A (Switch A)** | `192.168.100.0/27` | `.0` / `.31` | `.1` ~ `.30` | Router A `Gig0/0`: `.1` |
| **LAN B (Switch B)** | `192.168.100.32/27` | `.32` / `.63` | `.33` ~ `.62` | Router B `Gig0/0`: `.33` |
| **LAN C (Switch C)** | `192.168.100.64/27` | `.64` / `.95` | `.65` ~ `.94` | Router C `Gig0/0`: `.65` |
| **WAN AB (R_A - R_B)** | `192.168.100.96/27` | `.96` / `.127` | `.97` ~ `.126` | R_A `Se0/3/0`(DCE): `.97`<br>R_B `Se0/3/0`: `.98` |
| **WAN BC (R_B - R_C)** | `192.168.100.128/27`| `.128` / `.159`| `.129` ~ `.158`| R_B `Se0/3/1`: `.129`<br>R_C `Se0/3/1`(DCE): `.130`|

---

### 2. 하드웨어 구성 및 인터페이스 IP 설정

**장비 모듈 장착**
* Router A, B, C 전원 OFF ➔ `HWIC-2T` 시리얼 슬롯 장착 ➔ 전원 ON

**1) Router A 설정 (DCE 측 시리얼 포함)**
```cisco
Router> enable
Router# configure terminal
Router(config)# hostname Router_A

! LAN 인터페이스
Router_A(config)# interface GigabitEthernet0/0
Router_A(config-if)# ip address 192.168.100.1 255.255.255.224
Router_A(config-if)# no shutdown
Router_A(config-if)# exit

! WAN 인터페이스 (DCE)
Router_A(config)# interface Serial0/3/0
Router_A(config-if)# ip address 192.168.100.97 255.255.255.224
Router_A(config-if)# clock rate 64000
Router_A(config-if)# no shutdown
Router_A(config-if)# exit

```

**2) Router B 설정**

```cisco
Router> enable
Router# configure terminal
Router(config)# hostname Router_B

! LAN 인터페이스
Router_B(config)# interface GigabitEthernet0/0
Router_B(config-if)# ip address 192.168.100.33 255.255.255.224
Router_B(config-if)# no shutdown
Router_B(config-if)# exit

! WAN 인터페이스 1 (Router A 연결)
Router_B(config)# interface Serial0/3/0
Router_B(config-if)# ip address 192.168.100.98 255.255.255.224
Router_B(config-if)# no shutdown
Router_B(config-if)# exit

! WAN 인터페이스 2 (Router C 연결)
Router_B(config)# interface Serial0/3/1
Router_B(config-if)# ip address 192.168.100.129 255.255.255.224
Router_B(config-if)# no shutdown
Router_B(config-if)# exit

```

**3) Router C 설정 (DCE 측 시리얼 포함)**

```cisco
Router> enable
Router# configure terminal
Router(config)# hostname Router_C

! LAN 인터페이스
Router_C(config)# interface GigabitEthernet0/0
Router_C(config-if)# ip address 192.168.100.65 255.255.255.224
Router_C(config-if)# no shutdown
Router_C(config-if)# exit

! WAN 인터페이스 (DCE)
Router_C(config)# interface Serial0/3/1
Router_C(config-if)# ip address 192.168.100.130 255.255.255.224
Router_C(config-if)# clock rate 64000
Router_C(config-if)# no shutdown
Router_C(config-if)# exit

```

---

### 3. 동적 라우팅 프로토콜(RIP) 설정

각 라우터에 직접 연결된 네트워크 대역을 등록합니다.

```cisco
! Router A
Router_A(config)# router rip
Router_A(config-router)# network 192.168.100.0
Router_A(config-router)# exit

! Router B
Router_B(config)# router rip
Router_B(config-router)# network 192.168.100.0
Router_B(config-router)# exit

! Router C
Router_C(config)# router rip
Router_C(config-router)# network 192.168.100.0
Router_C(config-router)# exit

```

---

### 4. 검증 및 트러블슈팅

* **라우팅 테이블 확인:**

```cisco
Router_A# show ip route

```

`R` (RIP) 코드로 원격 서브넷 대역들이 `[120/1]`, `[120/2]` 형태로 정상 등록되었는지 확인.

* **프로토콜 동작 확인:**

```cisco
Router_A# show ip protocols

```

Update 타이머(30초) 및 라우팅 정보 송수신 인터페이스 확인.

* **End-to-End 통신 검증:**
* LAN A의 PC(`192.168.100.2`) ➔ LAN C의 PC(`192.168.100.66`)로 Ping 테스트 및 통신 성공 확인



```

```