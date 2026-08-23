# Day 6: GRE over IPsec VPN, FHRP(HSRP), IPv6 라우팅 및 L2/L3 스위칭

## 1. GRE over IPsec VPN 터널링 구축

Cisco 2811 라우터(WIC-2T 모듈 장착)를 기반으로 지점 간 안전한 통신을 위한 VPN 터널을 구축합니다.

### 1) 가상 터널 인터페이스 설정 (GRE Tunnel)

* **본사 라우터 (HQ / Left):**
  ```text
  interface Tunnel100
   ip address 10.10.10.1 255.255.255.0
   tunnel mode gre ip
   tunnel source Serial0/3/0
   tunnel destination 2.2.2.2
   exit

  ip route 172.16.0.0 255.255.255.0 10.10.10.2

```

* **지사 라우터 (Branch / Right):**
```text
interface Tunnel100
 ip address 10.10.10.2 255.255.255.0
 tunnel mode gre ip
 tunnel source Serial0/3/1
 tunnel destination 1.1.1.1
 exit

ip route 192.168.100.0 255.255.255.0 10.10.10.1

```


* **터널 캡슐화 모드 비교:**
* `tunnel mode gre ip`: GRE over IPv4 모드로 다양한 프로토콜(OSPF 등 멀티캐스트 포함) 캡슐화 지원
* `tunnel mode ipv6ip`: 순수 IPv6 패킷만을 IPv4 헤더로 감싸는 전용 터널 모드



---

### 2) IPsec 암호화 정책 및 적용 (Security Association)

* **본사 라우터 (HQ):**
```text
! 1. 암호화 대상 트래픽(GRE) 정의
access-list 100 permit gre host 1.1.1.1 host 2.2.2.2
access-list 100 permit gre 192.168.100.0 0.0.0.255 host 2.2.2.2

! 2. IKE Phase 1 정책 (ISAKMP Policy)
crypto isakmp policy 10
 authentication pre-share
 encryption 3des
 hash md5
 exit
crypto isakmp key 1234 address 2.2.2.2

! 3. IKE Phase 2 정책 (Transform-Set)
crypto ipsec transform-set test esp-3des esp-md5-hmac

! 4. Crypto Map 생성 및 조합
crypto map vpn-map 10 ipsec-isakmp
 match address 100
 set peer 2.2.2.2
 set transform-set test
 exit

! 5. 물리 인터페이스에 Crypto Map 적용
interface Serial0/3/0
 crypto map vpn-map
 exit

```


* **지사 라우터 (Branch):**
```text
access-list 100 permit gre host 2.2.2.2 host 1.1.1.1
access-list 100 permit gre 172.16.0.0 0.0.0.255 host 1.1.1.1

crypto isakmp policy 10
 authentication pre-share
 encryption 3des
 hash md5
 exit
crypto isakmp key 1234 address 1.1.1.1

crypto ipsec transform-set test esp-3des esp-md5-hmac

crypto map vpn-map 10 ipsec-isakmp
 match address 100
 set peer 1.1.1.1
 set transform-set test
 exit

interface Serial0/3/1
 crypto map vpn-map
 exit

```


* **상태 검증:**
* `show crypto isakmp sa`: IKE Phase 1 협상 상태 확인 (`QM_IDLE` 확인)
* 경로 추적 명령어: `traceroute [목적지IP]` (Cisco/Linux) / `tracert [목적지IP]` (Windows CMD)



---

## 2. 게이트웨이 이중화 (FHRP - HSRP)

네트워크 게이트웨이 단일 장애점(SPOF)을 제거하고 고가용성(HA)을 확보하기 위한 이중화 기술입니다.

### FHRP 프로토콜 종류 및 비교

* **HSRP (Hot Standby Router Protocol):** Cisco 독자 프로토콜 (Active / Standby 구조)
* **VRRP (Virtual Router Redundancy Protocol):** IEEE 표준 프로토콜 (Master / Backup 구조)
* **GLBP (Gateway Load Balancing Protocol):** Cisco 독자 프로토콜 (AVG / AVF 구조, 다중 게이트웨이 부하 분산 지원)

### HSRP 세부 설정 및 인터페이스 트래킹

```text
interface FastEthernet0/0
 ip address 192.168.100.1 255.255.255.0

 ! 가상 게이트웨이 VIP 할당
 standby 1 ip 192.168.100.254

 ! 우선순위(Priority) 지정 (기본값: 100)
 standby 1 priority 105

 ! 우선순위 강제 점유(Preemption) 활성화 (장애 복구 시 다시 Active로 승격)
 standby 1 preempt

 ! 외부 회선 장애 감시 (Serial 포트 다운 시 자동으로 Priority 감소)
 standby 1 track Serial0/3/0 10
 exit

```

* **상태 확인:** `show standby brief`

---

## 3. IPv6 주소 체계 및 OSPFv3 라우팅

IPv6는 128비트 길이의 16진수 기반 주소 체계를 사용합니다.

### 주소 표기 규칙

1. 각 16비트 블록 앞자리의 연속된 `0`은 생략 가능 (예: `0DB8` $\rightarrow$ `DB8`)
2. 연속되는 `0000` 블록은 전체 주소에서 **단 한 번만** `::`로 축약 가능
* 예: `2001:0DB8:0000:0000:0000:0000:0000:0000` $\rightarrow$ `2001:DB8::`



### 주요 IPv6 주소 범위

* **글로벌 유니캐스트 (Global Unicast):** `2001::/16`, `2002::/16` 등 (공인 인터넷 통신 가능)
* **링크 로컬 (Link-Local):** `FE80::/10` (동일 서브넷/로컬 링크 내에서만 유효)
* **듀얼 스택 (Dual Stack):** 동일 인터페이스 및 장비에 IPv4와 IPv6를 동시에 구동하는 방식

### IPv6 라우팅 및 OSPFv3 활성화

```text
! IPv6 유니캐스트 라우팅 전역 활성화 (필수)
ipv6 unicast-routing

! 인터페이스 IPv6 주소 할당 및 OSPF 프로세스 영역 할당
interface Serial0/2/0
 ipv6 address 2001:2:2:2::1/64
 ipv6 ospf 1 area 0
 no shutdown
 exit

! OSPFv3 라우팅 프로세스 및 Router-ID 지정 (32비트 IPv4 형태 ID 사용)
ipv6 router ospf 1
 router-id 11.11.11.11
 exit

```

---

## 4. L2 스위치 vs L3 스위치 동작 및 설정

### L2 vs L3 비교 요약

| 구분 | L2 스위치 | L3 스위치 |
| --- | --- | --- |
| **물리 포트 IP 할당** | 불가 (L2 스위칭 전용) | 가능 (`no switchport` 사용) |
| **관리용 IP (SVI)** | `interface vlan 1` 등에 부여 | VLAN 인터페이스 및 물리 포트 모두 가능 |
| **VLAN 활성화 조건** | 해당 VLAN에 속한 물리 포트에 링크가 활성화되어야 UP | 포트 링크 활성화 시 UP |
| **패킷 라우팅 기능** | 미지원 | 전역 `ip routing` 활성화 시 지원 |

### L3 스위치 포트 전환 및 라우팅 설정

```text
! 1. 물리 포트를 L3 라우팅 포트로 변경 후 IP 할당
Switch(config)# interface FastEthernet0/1
Switch(config-if)# no switchport
Switch(config-if)# ip address 5.5.5.5 255.255.255.0
Switch(config-if)# no shutdown
Switch(config-if)# exit

! 2. L3 스위치 전역 라우팅 엔진 활성화
Switch(config)# ip routing

! 3. IPv4 OSPF 프로세스 활성화
Switch(config)# router ospf 1
Switch(config-router)#

```

```

```