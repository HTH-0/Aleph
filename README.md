# 🌐 네트워크 & IT 인프라 기초 정리

IT 하부구조, 정보 보안, OSI 7 계층 및 TCP/IP 프로토콜의 핵심 개념을 정리한 노트입니다.

---

## 💻 1. 기본 개념 (Client & Server)

* **서버 (Server):** 정보를 제공하는 역할
* **클라이언트 (Client):** 정보를 요청하는 역할

### IT 하부구조 구성요소
> **하드웨어** | **소프트웨어** | **데이터센터** | **클라우드 서비스** | **네트워크**

* **클라우드 서비스:** 물리적 하드웨어를 직접 소유·관리할 필요 없이, 전문 기업의 데이터 센터 자원을 인터넷으로 필요한 만큼 빌려 쓰는 기술
* **SaaS (Software as a Service):** 인터넷을 통해 웹 브라우저로 접속해 이용하는 소프트웨어 월세/구독 서비스

---

## 🛡️ 2. 정보 보안 (Information Security)

### 정보 보안의 3대 요소 (CIA Triad)
1. **기밀성 (Confidentiality):** 인가된 사용자만 정보에 접근 가능
2. **무결성 (Integrity):** 정보가 인위적·우연히 변조되지 않고 정확함
3. **가용성 (Availability):** 인가된 사용자가 필요할 때 언제든 정보에 접근 가능

### 정보 보안의 종류
* **관리적 보안:** 보안 정책, 절차, 교육 등
* **물리적 보안:** 출입 통제, 센터 보안, 재해 대책 등
* **기술적 보안 (★ 핵심):**
  * **네트워크 보안:** 스위칭/라우팅/접근제어정책, VPN, NAT, IDS, NMS, Firewall, DDoS 대응 등
  * **시스템 보안:** 다양한 서버 구축 및 관리 설정, Buffer Overflow(BoF), Log 관리, 쉘 스크립트 등
  * **웹 보안:** 웹 공격 탐지 및 로그 관리, 모의해킹, CTF, 워게임, OWASP 대응 등

---

## 📐 3. OSI 7 계층 모델 (OSI 7 Layer)

데이터를 주고받을 때 송신 측은 **캡슐화(Encapsulation)** 과정을 거치고, 수신 측은 **역캡슐화(De-encapsulation)** 과정을 거칩니다.

| 계층 | 이름 | 데이터 단위 | 핵심 역할 및 특징 |
| :---: | :--- | :---: | :--- |
| **L7** | **응용 (Application)** | Data | 사용자 인터페이스 제공 (인터넷 서비스 지원) |
| **L6** | **표현 (Presentation)** | Data | 데이터의 의미와 표현 방법 (암호화, 압축, 변환) |
| **L5** | **세션 (Session)** | Data | 통신 연결 관리 (토큰 제어, 동기화) |
| **L4** | **전송 (Transport)** | Segment | 단대단(End-to-End) 통신, 실제 데이터 전송 및 흐름 제어 (TCP/UDP) |
| **L3** | **네트워크 (Network)** | Packet | 라우팅(경로 설정), 논리적 주소(IP) 지정, L3 장비 |
| **L2** | **데이터링크 (Data Link)** | Frame | 물리적 전송 오류 해결 및 흐름 제어, 물리적 주소(MAC), L2 장비 |
| **L1** | **물리 (Physical)** | Bit | 전송 매체의 물리적 인터페이스, 전송 속도, 클록 동기화, L1 장비 |

> ⬆️ **보내는 쪽 (송신):** 응용 계층 $\rightarrow$ 물리 계층 (**캡슐화**)  
> ⬇️ **받는 쪽 (수신):** 물리 계층 $\rightarrow$ 응용 계층 (**역캡슐화**)

---

## 🔄 4. OSI vs TCP/IP 모델 및 프로토콜 매핑

| OSI 7 Layer | Data 단위 | TCP/IP Model | Protocol |
| :--- | :---: | :--- | :--- |
| **Application** | <br>Data<br><br> | **Application** | FTP, TELNET, SMTP, HTTP, SNMP 등 |
| **Presentation** | | |
| **Session** | | |
| **Transport** | **Segment** | **Transport** | TCP, UDP |
| **Network** | **Packet** | **Internet** | IP, ICMP, IGMP |
| **Data Link** | **Frame** | **Network Access**<br>*(Network Interface)* | Ethernet, Wi-Fi 등 |
| **Physical** | **Bit** | |

---

## ⇄ 5. 전송 프로토콜 & 주소 변환

### TCP vs UDP
* **TCP (Transmission Control Protocol):**
  * Handshaking 과정을 거침 (연결 지향적)
  * **특징:** 신뢰성 높음, 전송 속도 상대적으로 느림
* **UDP (User Datagram Protocol):**
  * Handshaking 없음 (비연결 지향적)
  * **특징:** 신뢰성 낮음, 전송 속도 빠름 (실시간 스트리밍, 온라인 게임 등에 활용)

### 주소 체계 (Address)
계층마다 사용하는 주소가 다릅니다.
* **L2 계층:** 물리적 주소 $\rightarrow$ **MAC 주소** (2계층 장비가 인지)
* **L3 계층:** 논리적 주소 $\rightarrow$ **IP 주소** (전송 과정에서 위치/주소 역할)
* **L4 계층:** 프로세스 식별 주소 $\rightarrow$ **포트(Port) 번호**

### 주요 제어 프로토콜
* **ARP (Address Resolution Protocol):** IP 주소를 MAC 주소로 변환
* **RARP (Reverse ARP):** MAC 주소를 IP 주소로 변환
* **ICMP (Internet Control Message Protocol):** 네트워크 오류 보고 및 메세지 전송 프로토콜 (e.g., `ping`)
* **IGMP (Internet Group Management Protocol):** 멀티캐스트 그룹 관리 메세지 프로토콜
