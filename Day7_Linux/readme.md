# Day 7: Rocky Linux 환경 구축 및 LAMP 스택 기초 실습

## 1. VirtualBox를 활용한 가상머신(VM) 환경 설정

### VM 생성 사양
* **OS 유형:** Red Hat (64-bit)
* **VM 이름:** rocky
* **가상 하드 디스크:** 10GB

### 시스템 및 하드웨어 세부 구성
* **마더보드:** 부팅 순서 조정 (1순위: 하드 디스크, 2순위: 광학 드라이브, 플로피 체크 해제)
* **프로세서:** `Nested VT-x/AMD-V` 활성화 여부 확인 및 적용
* **저장소:** 컨트롤러 IDE에 Rocky Linux 설치 ISO 이미지 마운트
* **장치 비활성화:** 오디오 컨트롤러 및 USB 컨트롤러 비활성화 (리소스 최적화)
* **네트워크 설정:** 
  * 연결 방식: 어댑터에 브리지 (Bridged Adapter)
  * 무차별 모드 (Promiscuous Mode): 거부
  * 케이블 연결됨 체크 확인

### OS 설치 옵션
* **단축키 설정:** Host key combo를 `Left Windows`로 변경하여 마우스 탈출 편의성 확보
* **파티션:** 자동 파티션 구성 선택
* **계정 설정:** root 암호 지정, root 계정 잠금 해제, root 계정의 SSH 비밀번호 로그인 허용

### VM 백업 및 관리
* **스냅샷:** 특정 시점의 전체 시스템 상태를 기록하는 복원 지점
* **완전한 복제 (Full Clone):** 원본과 완전히 독립된 별도의 가상 머신 생성
* **연결된 복제 (Linked Clone):** 원본 스냅샷을 읽기 전용으로 공유하고 변경된 데이터만 별도 저장하여 용량 절약

---

## 2. 원격 접속 및 리눅스 기본 환경

### PuTTY 원격 접속
* SSH 프로토콜을 통해 Rocky Linux IP 주소로 원격 터미널 접속

### 셸 프롬프트 구조
```text
[root@localhost ~]#
[사용자명@호스트명 현재작업디렉토리]권한기호(#: root, $: 일반사용자)

```

* `~` : 현재 사용자의 홈 디렉터리 (root의 경우 `/root`)
* `/` : 시스템 최상위 루트 디렉터리

---

## 3. 리눅스 파일 시스템 계층 구조 (FHS)

| 디렉터리 | 주요 역할 및 설명 |
| --- | --- |
| `/bin` | 일반 사용자 및 관리자가 사용하는 필수 명령어 (`ls`, `cp`, `cd` 등) |
| `/sbin` | 시스템 관리자 전용 관리/복구 명령어 (`fdisk`, `reboot` 등) |
| `/boot` | 부팅 핵심 파일 (리눅스 커널 이미지, GRUB 부트로더 설정) |
| `/etc` | 시스템 전체 및 각종 응용 프로그램 설정 파일 |
| `/home` | 일반 사용자들의 개인 홈 디렉터리 (`/home/user1` 등) |
| `/root` | 최고 관리자(root) 전용 홈 디렉터리 |
| `/lib`, `/lib64` | 실행 바이너리에 필요한 공유 라이브러리 (64비트는 `/lib64`) |
| `/usr` | 2차 소프트웨어 및 라이브러리 저장소 |
| `/var` | 시스템 운영 중 실시간 변경되는 가변 데이터 (로그, 메일 등) |
| `/dev` | 하드 디스크, 입출력 장치 등 하드웨어 장치 파일 |
| `/proc` | 현재 실행 중인 프로세스와 메모리 정보를 담은 가상 파일 시스템 (RAM) |
| `/sys` | 커널 및 하드웨어 드라이버 상태 정보를 제공하는 가상 디렉터리 |
| `/mnt` | 파일 시스템을 수동으로 임시 마운트하는 디렉터리 |
| `/media` | 이동식 미디어(USB, CD-ROM 등) 자동 마운트 디렉터리 |
| `/opt` | 외부 써드파티(Third-party) 상용 프로그램 설치 경로 |
| `/srv` | Web, FTP 등 시스템 제공 서비스의 실제 데이터 저장소 |
| `/tmp` | 공용 임시 파일 저장소 (주기적 또는 재부팅 시 삭제 가능) |
| `/run` | 부팅 이후 구동 중인 서비스의 런타임 상태 데이터 (RAM) |
| `/afs` | Andrew File System 분산 네트워크 마운트 포인트 |

---

## 4. 리눅스 기본 명령어 정리

### 기본 탐색 및 파일/디렉터리 조작

* `pwd`: 현재 작업 디렉터리 경로 출력 (Print Working Directory)
* `cd [경로]`: 작업 디렉터리 이동 (절대경로: `/` 기준, 상대경로: 현재 위치 기준)
* `ls`: 디렉터리 목록 조회
* `ls -al`: 숨김 파일을 포함한 상세 정보 출력
* `ls -h`: 파일 크기를 읽기 쉬운 단위(K, M, G)로 표시
* `ls --help` / `man ls`: 명령어 도움말 및 매뉴얼 출력


* `mkdir [폴더명]`: 디렉터리 생성 (`-p`: 하위 경로 일괄 생성)
* `rmdir [폴더명]`: 비어 있는 디렉터리 삭제
* `rm -rf [경로]`: 파일 및 하위 디렉터리 강제 일괄 삭제
* `touch [파일명]`: 빈 파일 생성 또는 타임스탬프 갱신
* `cat [파일명]`: 파일 내용 화면 출력

### 계정 및 권한 관리

* `useradd [계정명]`: 신규 사용자 생성
* `passwd [계정명]`: 사용자 비밀번호 설정
* `su [계정명]`: 사용자 전환 (Switch User)
* `chmod [권한] [대상]`: 파일/디렉터리 권한 변경
* 숫자 방식: `r=4`, `w=2`, `x=1` (예: `chmod 750 file`)
* 문자 방식: `u`(소유자), `g`(그룹), `o`(기타) (예: `chmod u+x file`)


* `chown -R [소유자]:[그룹] [대상]`: 소유자 및 소유 그룹 재귀적 변경

### 파일 권한 표기 읽는 법

```text
drwxr-xr-x. 2 root root 62 8월 20 10:37 libssh

```

* `d`: 파일 유형 (`d`: 디렉터리, `-`: 일반 파일, `l`: 심볼릭 링크)
* `rwxr-xr-x`: 소유자(rwx) / 그룹(r-x) / 기타 사용자(r-x) 접근 권한
* `2`: 하드 링크 수
* `root root`: 소유자 / 소유 그룹
* `62`: 파일 크기 (바이트)
* `8월 20 10:37`: 최종 수정 일시

---

## 5. vi 편집기 활용법

### 주요 3대 모드

1. **명령 모드 (Command Mode):** 최초 진입 상태. 이동, 복사, 삭제 수행
2. **입력 모드 (Insert Mode):** 실제 텍스트 입력 및 수정 (`-- INSERT --`)
3. **마지막 행 모드 (Last Line Mode):** 저장, 종료, 치환 등 명령 수행 (`:`)

### 단축키 요약

* **입력 전환:** `i` (현재 위치), `a` (커서 뒤), `o` (아래 줄 추가)
* **복사/붙여넣기/삭제:** `yy` (한 줄 복사), `dd` (한 줄 잘라내기/삭제), `p` (붙여넣기), `x` (한 글자 삭제), `u` (실행 취소)
* **저장 및 종료:** `:w` (저장), `:q` (종료), `:wq` (저장 후 종료), `:q!` (강제 종료), `:set nu` (줄 번호 표시)

---

## 6. 패키지 관리 및 파일 검색

### DNF / RPM 패키지 관리

* `dnf install -y [패키지명]`: 패키지 무응답 자동 설치
* `dnf remove [패키지명]`: 패키지 삭제
* `dnf search [키워드]`: 저장소 내 패키지 검색
* `dnf update [패키지명]`: 패키지 업데이트
* `rpm -qa | grep [패키지명]`: 시스템에 설치된 패키지 목록 필터링 조회

### find 명령어 활용

* **파일명 기준:** `find / -name "nginx.conf"`
* **대소문자 무시:** `find . -iname "*.png"`
* **타입 기준:** `find . -type f -name "test*"` (f: 파일, d: 디렉터리)
* **크기 기준:** `find /var/log -size +100M`
* **수정 시간 기준:** `find . -mtime -7` (최근 7일 이내)
* **검색 결과 일괄 처리:** `find . -name "*.tmp" -exec rm -f {} \;`

---

## 7. LAMP (Apache, PHP, MariaDB) 환경 구축

### 1) 패키지 설치

```bash
dnf install -y httpd php mariadb-server

```

### 2) 서비스 제어 (systemd)

* `systemctl start httpd` / `systemctl start mariadb` : 서비스 시작
* `systemctl enable httpd` / `systemctl enable mariadb` : 부팅 시 자동 시작 활성화
* `systemctl status [서비스명]` : 서비스 상태 점검

### 3) 방화벽 설정 (Firewalld)

```bash
firewall-cmd --permanent --add-port=80/tcp
# 또는 firewall-cmd --permanent --add-service=http
firewall-cmd --reload
firewall-cmd --list-all

```

### 4) Apache 웹 서버 주요 설정 (`/etc/httpd/conf/httpd.conf`)

* `Listen 80`: 웹 서비스 기본 포트
* `DocumentRoot "/var/www/html"`: 웹 콘텐츠 루트 디렉터리
* `ErrorLog "logs/error_log"`: 웹 서버 에러 로그 경로
* `LogLevel warn`: 로깅 레벨 기준 설정

---

## 8. MariaDB 데이터베이스 초기화 및 SQL 기초

### 초기 보안 설정

`mysql_secure_installation` 대화형 스크립트 실행:

* UNIX socket 인증 전환: `Y`
* DB root 비밀번호 설정: `Y`
* 익명 사용자(Anonymous Users) 삭제: `Y`
* 원격 root 로그인 차단: `Y`
* test 데이터베이스 삭제: `Y`
* 권한 테이블 재로드: `Y`

### DB 접속 및 기본 쿼리

```bash
mysql -u root -p

```

```sql
-- 데이터베이스 목록 확인
SHOW DATABASES;

-- 기본 mysql DB 선택
USE mysql;

-- 테이블 목록 및 테이블 구조 확인
SHOW TABLES;
DESC user;

-- 데이터 조회
SELECT * FROM user;
SELECT user, host, password FROM user;

```

### SQL 주요 자료형 분류

* **문자형:** `CHAR(N)` (고정 길이), `VARCHAR(N)` (가변 길이), `TEXT` (대용량 본문)
* **숫자형:** `INT` (기본 정수), `BIGINT` (대규모 정수), `DECIMAL` (고정소수점 실수)
* **날짜형:** `DATE` (YYYY-MM-DD), `DATETIME` (YYYY-MM-DD HH:MM:SS)

```

```