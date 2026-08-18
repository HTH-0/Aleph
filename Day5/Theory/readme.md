# Day 05 - Theory: OSPF, ACL & NAT 동작 원리

## 1. OSPF (Open Shortest Path First)

### 1.1 OSPF Area 구조 및 특징
* **Backbone Area (Area 0)**: OSPF 네트워크의 중심 영역으로, 모든 다른 Area는 반드시 Area 0과 물리적 또는 논리적으로 직접 연결되어야 함.
* **Single Area**: 네트워크 전체에 단 하나의 Area만 존재하는 구조이며, 반드시 Area 번호를 `0`으로 지정함.
* **Multi Area**: 2개 이상의 Area로 구성된 네트워크 구조이며, Area 간 라우팅은 별도의 재분배 설정 없이 자동으로 수행됨.
* **Virtual Link**: 물리적으로 Area 0에 직접 연결되지 못한 Area가 존재할 경우, 논리적으로 Area 0에 연결하기 위해 사용하는 터널링 기술.
* **Auto-Summary**: OSPF는 자동 요약(Auto-Summary) 기능이 기본적으로 작동하지 않으므로 `no auto-summary` 명령어가 필요 없음.

### 1.2 Router ID 선출 우선순위
OSPF 프로세스 내에서 라우터를 식별하기 위한 32비트 번호로, 아래의 우선순위에 따라 결정됨.

1. 관리자가 `router-id` 명령어로 직접 지정한 값
2. 활성화된 논리 인터페이스(Loopback)에 할당된 IP 중 가장 높은 주소
3. 활성화된 물리 인터페이스에 할당된 IP 중 가장 높은 주소

> **Note**: Router ID 변경 시 `clear ip ospf process` 명령을 실행하거나 라우터를 재부팅(`reload`)해야 적용됨.

### 1.3 OSPF Process ID
* 라우터 내부(Local)에서 해당 OSPF 프로세스를 식별하기 위한 번호임.
* OSPF 패킷 전달 및 인웃(Neighbor) 성립 조건에 영향을 주지 않으므로, 이웃 라우터 간 Process ID가 달라도 통신이 가능함.

---

## 2. ACL (Access Control List)

네트워크 보안 및 통신 정책 설정을 위해 라우터의 입구(Inbound) 및 출구(Outbound)에서 패킷을 제어(Permit / Deny)하는 접근 제어 목록임.

### 2.1 Standard ACL vs Extended ACL 비교

| 항목 | Standard ACL | Extended ACL |
| :--- | :--- | :--- |
| **ACL 번호 범위** | 1 ~ 99, 1300 ~ 1999 | 100 ~ 199, 2000 ~ 2699 |
| **검사 조건** | 출발지 IP 주소 (Source IP) | 출발지 IP, 목적지 IP, 프로토콜, 포트 번호 |
| **권장 배치 위치** | 목적지(Destination)와 가장 가까운 라우터 | 출발지(Source)와 가장 가까운 라우터 |
| **배치 사유** | 출발지 근처 차단 시 다른 목적지 통신까지 차단됨 | 차단 대상 패킷을 입구에서 즉시 처리하여 대역폭 낭비 방지 |

### 2.2 ACL 작성 및 검사 규칙
* **순차적 검사**: ACL 항목은 상단(Line 1)부터 아래로 순차 적용되므로, 범위가 좁은 조건(특정 IP)을 상단에, 넓은 조건(네트워크 대역)을 하단에 배치함.
* **Implicit Deny Any**: 모든 ACL의 최하단에는 명시되지 않은 모든 트래픽을 차단하는 `deny ip any any` 조건이 기본으로 존재함. 차단 규칙 외 트래픽 허용을 위해 맨 마지막에 `permit` 구문이 필수임.

### 2.3 Extended ACL 연산자 식별자

| 연산자 | 구문 | 설명 |
| :--- | :--- | :--- |
| **eq** | Equal | 지정한 포트 번호와 일치 |
| **neq** | Not Equal | 지정한 포트 번호가 아닌 경우 |
| **gt** | Greater Than | 지정한 포트 번호 초과 |
| **lt** | Lower / Less Than | 지정한 포트 번호 미만 |
| **range** | Range | 지정한 두 포트 번호 범위 내 포함 |
| **established** | Established | TCP ACK/RST 플래그가 설정된 기존 세션 응답 트래픽 허용 |

---

## 3. NAT (Network Address Translation) 및 PAT

사설 IP 주소(Private IP)를 공인 IP 주소(Public IP)로 변환하여 외부 인터넷과의 통신을 가능하게 하는 기술임.

### 3.1 변환 유형 분류
* **Static NAT**: 사설 IP와 공인 IP를 1:1 고정 매핑함.
* **Dynamic NAT**: 사설 IP 그룹을 준비된 공인 IP 풀(Pool)의 주소와 1:1 동적 매핑함.
* **PAT (Port Address Translation)**: Port 번호를 활용하여 하나의 공인 IP에 여러 사설 IP를 N:1 동적 매핑함 (Cisco 명령어 키워드: `overload`).