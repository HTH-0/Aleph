# [Cisco] 시스코 장비 기초 설정 및 CLI 명령어 실습

---

### 1. 물리 환경 및 케이블 연결 규칙

* **토폴로지:** 물리 토폴로지(배치 형태) vs 논리 토폴로지(데이터 흐름 경로)
* **케이블 구분:**
  * **Direct-Through (이종 장비):** PC ↔ Switch, Switch ↔ Router
  * **Cross-Over (동종 장비):** PC ↔ PC, Switch ↔ Switch, Router ↔ Router
* **포트 LED 상태:**
  * 🟢 **초록:** 정상 통신 (Link Up)
  * 🟠 **주황:** STP 계산 및 협상 중 (약 30초 소요)
  * 🔴 **빨강:** 링크 단선 또는 비활성화 (`shutdown`)

---

### 2. Cisco CLI 계층 구조 및 모드 전환

```text
[User Mode] 
   │   Router>
   └─ (enable / en) ──> [Privileged Mode] 
                             │   Router#
                             └─ (configure terminal / conf t) ──> [Global Config Mode]
                                                                        │   Router(config)#
                                                                        ├─ (line console 0) ──> [Line Config]
                                                                        │                         Router(config-line)#
                                                                        └─ (interface gi0/0) ──> [Interface Config]
                                                                                                  Router(config-if)#