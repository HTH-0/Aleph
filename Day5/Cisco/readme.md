
### Cisco

```markdown
# Day 05 - Lab: OSPF, ACL & Dynamic NAT 구성 실습

## 1. IP 및 인터페이스 할당 정보

| 장비명 | 인터페이스 | IP 주소 / Subnet Mask | 비고 |
| :--- | :--- | :--- | :--- |
| **Router_Left** | GigabitEthernet0/0 | 172.16.100.1 /24 | 내부 LAN 게이트웨이 |
| | Serial0/3/0 | 1.1.1.2 /24 | 중앙 라우터 연결 포트 (`NAT Outside`) |
| | Serial0/3/1 | 2.2.2.1 /24 | 이종 라우터 연결 포트 |
| | Loopback0 | 5.5.5.5 /32 | Router ID 테스트용 인터페이스 |
| **Server** | GigabitEthernet0/0.30 | 192.168.200.34 /28 | Web / FTP Server |

---

## 2. 단계별 CLI 설정 명령어

### 2.1 OSPF 및 EIGRP 라우팅 재분배 설정

```cisco
configure terminal

! EIGRP 100 프로세스에서 OSPF 1 경로 재분배
router eigrp 100
 redistribute ospf 1 metric 10000 100 255 1 1500
 exit

! OSPF 1 프로세스에서 EIGRP 100 경로 재분배 (subnets 키워드 필수)
router ospf 1
 network 1.1.1.0 0.0.0.255 area 0
 network 2.2.2.0 0.0.0.255 area 0
 network 172.16.100.0 0.0.0.255 area 0
 redistribute eigrp 100 subnets
 end

```

### 2.2 Named Extended ACL 구성 및 적용

조건:

1. `10.10.10.3` -> `192.168.200.34`: FTP 접속 허용
2. `10.10.10.0/24` -> `192.168.200.34`: FTP 접속 차단
3. `172.16.100.3` -> `192.168.200.34`: WEB(HTTP) 접속 차단
4. 그 외 모든 트래픽 허용

```cisco
configure terminal

! Extended ACL 'pra' 정의
ip access-list extended pra
 10 permit tcp host 10.10.10.3 host 192.168.200.34 eq ftp
 20 deny tcp 10.10.10.0 0.0.0.255 host 192.168.200.34 eq ftp
 30 deny tcp host 172.16.100.3 host 192.168.200.34 eq www
 40 permit ip any any
 exit

! 라우터 입구 인터페이스에 ACL 적용
interface Serial0/3/0
 ip access-group pra in
 end

```

### 2.3 Dynamic NAT 구성

```cisco
configure terminal

! 1. NAT에 사용할 공인 IP Pool 생성
ip nat pool dnat 1.1.1.10 1.1.1.254 netmask 255.255.255.0

! 2. 변환 대상 사설 IP 지정 ACL 작성
access-list 10 permit 172.16.100.0 0.0.0.255

! 3. ACL과 NAT Pool 결합
ip nat inside source list 10 pool dnat

! 4. 내부/외부 인터페이스 지정
interface GigabitEthernet0/0
 ip nat inside
 exit

interface Serial0/3/0
 ip nat outside
 end

```

---

## 3. 상태 확인 및 검증 (Troubleshooting)

### 3.1 OSPF 동작 상태 검증

```cisco
! OSPF 이웃 상태 확인 (FULL 상태인지 확인)
show ip ospf neighbor

! OSPF 데이터베이스 정보 확인
show ip ospf database

! OSPF 프로세스 ID, Router ID, 인터페이스 Cost 확인
show ip ospf interface

```

### 3.2 ACL 적용 상태 및 매칭 카운터 확인

```cisco
! ACL 설정 및 트래픽 매칭 카운트(matches) 확인
show access-lists pra

! 인터페이스에 적용된 Inbound/Outbound ACL 확인
show ip interface Serial0/3/0

```

### 3.3 NAT 변환 테이블 및 통계 확인

```cisco
! 현재 활성화된 NAT 변환 세션 리스트 확인
show ip nat translations

! NAT 변환 통계 및 Inside/Outside 인터페이스 지정 확인
show ip nat statistics

```

### 3.4 확장 핑(Extended Ping)을 통한 Return Route 검증

출발지 IP를 Loopback0으로 지정하여 양방향 통신 검증 수행.

```cisco
ping 192.168.200.34 source 5.5.5.5

```

```

```