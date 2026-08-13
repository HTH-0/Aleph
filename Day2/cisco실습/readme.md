# 시스코(Cisco) 네트워크 기초 및 CLI 명령어 정리

## 1. 네트워크 기초 개념

### 토폴로지 (Topology)

* **Physical Topology:** 물리적인 장비 배치 및 케이블 연결 형태
* **Logical Topology:** 네트워크 내에서 실제 데이터(패킷)가 이동하는 논리적 경로

### 케이블 유형 및 연결 규칙

* **Direct-Through Cable (이종 장비 간 연결):**
* 서로 다른 계층/역할의 장비를 연결 (예: PC $\leftrightarrow$ Switch, Router $\leftrightarrow$ Switch)
* **Cross-Over Cable (동종 장비 간 연결):**
* 같은 계층/역할의 장비를 연결 (예: PC $\leftrightarrow$ PC, PC $\leftrightarrow$ Router, Switch $\leftrightarrow$ Switch)

### 네트워크 범위 구분

* **LAN (Local Area Network):** L2(Data Link) 계층 이하의 장비(스위치, MAC 주소)로 구성된 단일 근거리 네트워크
* **WAN (Wide Area Network):** L3(Router) 장비를 통해 멀리 떨어진 서로 다른 LAN과 LAN을 연결한 광역 네트워크

---

## 2. L3 네트워크 및 IP 주소 체계

### IP 대역 및 서브넷팅 (`192.168.16.0/24`)

* **네트워크 주소 (`192.168.16.0`):** 해당 네트워크 대역(아파트 단지)을 대표하는 주소 (단말 할당 불가)
* **브로드캐스트 주소 (`192.168.16.255`):** 전체 단말로 패킷을 전송하기 위한 주소 (단말 할당 불가)
* **유효 호스트 IP 범위:** `192.168.16.1` ~ `192.168.16.254` (총 254개 할당 가능)

### Prefix 표기법 (`/24`)

* `/24`는 서브넷 마스크 `255.255.255.0`을 의미함
* 2진수로 변환 시 비트값 `1`이 연속되는 개수($8 \times 3 = 24\text{ bit}$)를 표현한 것

```text
11111111.11111111.11111111.00000000 (/24)

```

### Default Gateway (디폴트 게이트웨이)

* **역할:** PC가 자신이 속한 LAN(동일 서브넷)을 벗어나 외부 네트워크(WAN)로 나갈 때 통과하는 라우터 포트의 IP
* **설정 이유:** 게이트웨이 설정이 없으면 외부 네트워크로 전달되는 패킷의 출입구를 찾지 못함

### 패킷 및 링크 동작

* **TTL (Time To Live):** 패킷 무한 루프(Looping) 방지용 수명값. 라우터를 1회 통과(1 Hop)할 때마다 1씩 감소하며, `0`이 되면 패킷 파기
* **포트 상태 LED 표시:**
* 🟢 **초록색:** 링크 정상 연결 및 통신 가능
* 🔴 **빨간색:** 포트 비활성화(`shutdown`) 또는 케이블 미연결
* 🟠 **주황색:** STP(Spanning Tree) 계산 및 링크 시도 중 (약 30초 소요)

---

## 3. Cisco CLI 명령어 모드 구조

CLI 모드는 아래와 같은 하향식 계층 구조를 갖습니다.

```text
[User Mode] 
   │   Router>
   └─ (enable) ──> [Privileged EXEC Mode] 
                        │   Router#
                        └─ (configure terminal) ──> [Global Configuration Mode]
                                                         │   Router(config)#
                                                         ├─ (line console 0) ──> [Line Config Mode]
                                                         │                          Router(config-line)#
                                                         └─ (interface gi0/0) ──> [Interface Config Mode]
                                                                                    Router(config-if)#

```

---

## 4. 제어 통로(Line)의 개념 및 동작 원리

장비에는 실제 데이터가 통과하는 도로(`Interface`)와 관리자가 장비를 제어하기 위한 통로(`Line`)가 분리되어 있습니다.

* **Line의 개념:** 관리자가 장비에 들어가서 명령어를 치고 **제어하기 위해 사용하는 전용 내부 통로**
* **구조적 원리:** `IP 주소` (장비 찾기) $\rightarrow$ `Port 22/23` (현관문) $\rightarrow$ **`Line` (내부 세부 창구)**
* 포트로 들어온 여러 사용자가 서로 섞이지 않도록 내부에서 라인(Line) 단위로 1:1 세션을 나눠 제어함


* **Console vs VTY 차이:**
* **`line console 0`:** 물리 케이블 직접 연결. IP 없이도 작동하며 **초기 세팅 및 비상 복구용** (1개만 존재)
* **`line vty 0 4`:** 네트워크 원격 접속(Telnet/SSH). **일상 관리용**이며 여러 명 동시 접속 가능


* **VTY 범위 및 동작:**
* **`vty`:** "가상 원격 접속 통로"를 뜻하는 시스코 지정 키워드 (수정 불가)
* **`0 4`:** 0~4번 라인, 즉 **동시 접속자 5명** 허용
* **설정 덮어쓰기:** `0 4` 후 `0 7`로 재설정 시 라인이 8개로 확장되며, 중복된 0~4번 설정은 새로 입력한 내용으로 덮어씌워짐(Override)



---

## 5. 모드별 주요 CLI 명령어

### 1) 사용자 모드 (`Router>`)

* **`enable` (축약: `en`):** 특권 모드(`Router#`)로 이동

### 2) 특권 모드 (`Router#`)

* **`configure terminal` (축약: `conf t`):** 전역 설정 모드로 이동
* **`show mac-address-table`:** 스위치 학습 MAC 주소 테이블 확인
* **`show running-config`:** 현재 RAM에서 동작 중인 휘발성 설정 확인
* **`show startup-config`:** NVRAM에 저장된 비휘발성 저장 설정 확인
* **`copy running-config startup-config` (또는 `write memory` / `wr`):** 현재 RAM 설정을 NVRAM에 저장

### 3) 전역 설정 모드 (`Router(config)#`)

* **`hostname <장비명>`:** 장비 이름 변경
* **`enable password <비밀번호>`:** Privileged 모드 진입 패스워드 설정 (평문)
* **`enable secret <비밀번호>`:** Privileged 모드 진입 패스워드 설정 (암호화, `password`보다 우선 적용)
* **`service password-encryption`:** 설정 파일 내 모든 평문 비밀번호 암호화

### 4) 회선 설정 모드 (`Router(config-line)#`)

* **`line console 0`:** 콘솔 직접 접속 포트 설정 모드 진입 **(초기 세팅 및 비상 복구용)** (✨ 설명 보완)
* **`line vty 0 4`:** 원격 접속(Telnet/SSH) 포트 설정 모드 진입 **(0~4번, 동시 접속자 5명)** (✨ 설명 보완)
* **`password <비밀번호>`:** 해당 회선 접속 패스워드 지정
* **`login`:** 접속 시 패스워드 인증 절차 적용

### 5) 인터페이스 설정 모드 (`Router(config-if)#`)

* **`interface gigabitEthernet 0/0` (축약: `int gi0/0`):** Gi0/0 포트 설정 모드 진입
* **`ip address <IP주소> <서브넷마스크>`:** 포트에 IP 주소 부여 (예: `ip address 192.168.16.1 255.255.255.0`)
* **`no shutdown` (축약: `no shut`):** 포트 활성화 (라우터 포트는 기본값이 `shutdown`이므로 필수 실행)

---

## 6. CLI 유용한 단축키 및 편의 기능

* **도움말 (`?`):** 명령어 뒤에 붙여 사용 가능한 키워드 확인 (예: `show ?`, `configure ?`)
* **자동 완성 (`Tab`):** 명령어 일부 입력 후 누르면 풀 스펠링 자동 완성
* **출력 페이징 제어:**
* `Enter`: 1줄씩 보기
* `Space`: 1페이지씩 보기
* **모드 이동 및 탈출:**
* `exit`: 바로 이전 상위 모드로 이동
* `End` 또는 `Ctrl + Z`: 어느 하위 모드에 있든 즉시 특권 모드(`Router#`)로 이동
* **`do` 명령어:** 설정 모드(`config` 계층)에서 특권 모드 명령어를 즉시 실행할 때 사용 (예: `do show running-config`)

---