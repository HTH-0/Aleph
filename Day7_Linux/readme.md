```markdown
# Rocky Linux 9 & APM Server Infrastructure Guide

VirtualBox 기반 Rocky Linux 9 환경 구축부터 기본 명령어, VI 편집기, 파일 권한 관리 및 APM(Apache, PHP, MariaDB) 웹 서버 구축까지의 정리 문서입니다.

---

### 1. VirtualBox 환경 구성 및 VM 생성

**가상머신(VM) 사양 설정**
* **가상머신 이름:** rocky
* **OS 유형:** Red Hat (64-bit)
* **가상 하드 디스크:** 10 GB

**시스템 및 디바이스 상세 설정**

| 구분 | 메뉴 위치 | 설정 값 | 비고 |
| :--- | :--- | :--- | :--- |
| **부팅 순서** | 시스템 > 마더보드 | 하드 드라이브, 광 드라이브 (플로피 해제) | 부팅 우선순위 지정 |
| **가상화** | 시스템 > 프로세서 | Nested VT-x/AMD-V 활성화 | 중첩 가상화 지원 |
| **OS ISO** | 저장소 > 컨트롤러: IDE | Rocky Linux ISO 이미지 탑재 | OS 설치 매체 로드 |
| **장치 최적화** | 오디오 / USB | 오디오 및 USB 컨트롤러 비활성화 | 불필요 리소스 차단 |
| **네트워크** | 네트워크 > 어댑터 1 | 네트워크 브리지 / 무차별 모드: 거부 | 호스트와 동일 대역 IP 할당 |

**환경 설정 및 OS 설치 요약**
* **Host Key 변경:** `파일` > `환경설정` > `입력` > `Host key combo` -> `Left Windows` (마우스 커서 제어)
* **설치 요약:**
  * **설치 목적지:** 자동 파티션 설정
  * **Root 계정:** 비밀번호 설정 / `root 계정 잠금` 해제 / `SSH 로그인 허용` 체크

---

### 2. 가상머신 관리 및 원격 접속

**스냅샷 및 복제**
* **스냅샷 (Snapshot):** 특정 시점의 전체 시스템 상태를 저장하는 복원 지점
* **완전한 복제 (Full Clone):** 원본과 독립된 새로운 가상머신 생성
* **연결된 복제 (Linked Clone):** 원본의 특정 스냅샷을 읽기 전용으로 공유하며, 차이점 데이터만 별도 저장

**PuTTY 원격 접속**
* **접속 방법:** PuTTY 실행 후 Hostname에 리눅스 IP 주소 입력 (22번 포트)
* **쉘 프롬프트 구조 (`[root@localhost ~]#`):**
  * `root`: 현재 로그인한 계정명
  * `localhost`: 호스트 이름
  * `~`: 현재 위치 (홈 디렉토리)
  * `#`: 관리자(root) 권한 (`$`는 일반 사용자)

---

### 3. 리눅스 핵심 명령어 및 디렉토리 구조

**주요 기초 명령어**

| 명령어 | 기능 | 사용 예시 |
| :--- | :--- | :--- |
| `pwd` | 현재 작업 디렉토리 경로 출력 | `pwd` |
| `cd` | 디렉토리 이동 | `cd /etc` (절대경로), `cd ..` (상대경로) |
| `ls` | 디렉토리 목록 출력 | `ls -al` (전체/상세), `ls -h` (용량 단위 표기) |
| `man` | 명령어 매뉴얼 확인 | `man ls` |
| `mkdir` | 디렉토리 생성 | `mkdir -p test/test2` |
| `rmdir` | 비어있는 디렉토리 삭제 | `rmdir test` |
| `rm` | 파일 및 디렉토리 강제 삭제 | `rm -rf test` |
| `touch` | 빈 파일 생성 | `touch test.txt` |
| `cat` | 파일 내용 출력 | `cat test2.txt \| grep W` |
| `su` | 사용자 전환 | `su test` |

**주요 디렉토리 구조**
* `/` (최상위 루트): 리눅스 파일 시스템의 최상위 디렉토리
* `/bin`: 일반 사용자 및 관리자용 기본 명령어 (`ls`, `cd`, `cp` 등)
* `/sbin`: 시스템 관리자(root) 전용 명령어 (`fdisk`, `reboot` 등)
* `/boot`: 커널 이미지 및 부팅 관련 핵심 파일
* `/etc`: 시스템 전체 제어 및 프로그램 설정 파일 모음
* `/home`: 일반 사용자들의 개인 홈 디렉토리
* `/root`: 최고 관리자(root) 전용 개인 홈 디렉토리
* `/lib`, `/lib64`: 명령어 실행에 필요한 C 공유 라이브러리 파일
* `/usr`: 일반 프로그램 및 라이브러리가 설치되는 저장소
* `/var`: 로그, 메일, 캐시 등 가변 데이터 저장소
* `/dev`: 하드디스크, 키보드 등 장치(Device) 파일
* `/proc`: 실행 중인 프로세스와 메모리 정보를 담은 가상 디렉토리 (RAM 상 존재)
* `/sys`: 커널 및 장치 드라이버 상태 정보 가상 디렉토리
* `/mnt`, `/media`: 외장 저장장치 임시 마운트 및 자동 연결 경로
* `/opt`: 서드파티(Third-party) 상용 소프트웨어 설치 경로
* `/tmp`: 임시 파일 저장소 (재부팅 시 삭제 가능)

---

### 4. VI 편집기 사용법

**3대 모드 전환**
* **입력 모드 (Insert Mode):** `i`, `a`, `o` 키 입력
* **명령 모드 (Command Mode):** `Esc` 키 입력
* **콜론 모드 (Last Line Mode):** 명령 모드에서 `:` 입력

**주요 단축키**
* **입력 모드 전환 (명령 모드 기준):**
  * `i`: 현재 커서 위치에서 입력
  * `a`: 현재 커서 다음 위치에서 입력
  * `o`: 현재 줄 아래에 새 줄을 만들고 입력
* **삭제 및 복사 (명령 모드 기준):**
  * `x`: 커서 위치 글자 1개 삭제
  * `dd`: 현재 줄 전체 삭제 (잘라내기)
  * `yy`: 현재 줄 전체 복사
  * `p`: 복사/잘라낸 내용 커서 아래 붙여넣기
  * `u`: 작업 취소 (Undo)
* **저장 및 종료 (콜론 모드 기준):**
  * `:w` (저장) / `:q` (종료) / `:wq` (저장 후 종료) / `:q!` (강제 종료)
  * `:set nu` (행 번호 표시)

---

### 5. 퍼미션(권한) 및 파일 검색

**파일 권한 구조 (`ls -al` 결과 해석)**
* **표기 예시:** `drwxr-xr-x. 2 root root 62 8월 20 10:37 libssh`
* **파일 타입:** `d` (디렉토리), `-` (일반 파일), `l` (심볼릭 링크)
* **권한 표기:** `r` (Read=4), `w` (Write=2), `x` (Execute=1)
  * `rwx`: 소유자 권한 (7)
  * `r-x`: 그룹 권한 (5)
  * `r-x`: 기타 사용자 권한 (5)
* **권한 변경 (`chmod`):**
  * `chmod 750 test.txt` (숫자 방식)
  * `chmod u+x test.txt` (문자 방식)
* **소유권 변경 (`chown`):**
  * `chown -R user1:group1 /test` (`-R`: 하위 디렉토리 포함)

**파일 검색 (`find`)**

```bash
# 1. 이름으로 검색
find / -name "nginx.conf"       # 최상위 디렉토리부터 정확한 파일명 검색
find . -iname "*.png"           # 대소문자 구분 없이 검색

# 2. 타입으로 검색
find . -type f -name "test*"    # 일반 파일만 검색
find . -type d -name "test*"    # 디렉토리만 검색

# 3. 크기로 검색
find /var/log -size +100M       # 100MB 이상 대용량 파일 검색

# 4. 검색 결과에 명령어 실행
find . -name "*.tmp" -exec rm -f {} \;   # 검색된 .tmp 파일 일괄 삭제

```

---

### 6. APM (Apache, PHP, MariaDB) 웹 서버 구축

**패키지 설치 및 서비스 제어**

```bash
# APM 패키지 통합 설치
dnf install -y httpd php mariadb-server

# 부팅 시 서비스 자동 실행 등록 및 즉시 시작
systemctl enable --now httpd mariadb

```

* **패키지 관리자 (`dnf`):** `install`, `remove`, `search`, `update`
* **서비스 관리자 (`systemctl`):** `start`, `stop`, `restart`, `status`, `enable`, `disable`
* **RPM 패키지 조회:** `rpm -qa | grep httpd`

**방화벽 설정 (`firewalld`)**

```bash
# HTTP(80/tcp) 포트 영구 허용
firewall-cmd --permanent --add-port=80/tcp

# 방화벽 설정 재로드
firewall-cmd --reload

# 방화벽 허용 목록 확인
firewall-cmd --list-all

```

**Apache 주요 설정 파일 (`/etc/httpd/conf/httpd.conf`)**

* **34열:** `ServerRoot "/etc/httpd"`
* **47열:** `Listen 80`
* **124열:** `DocumentRoot "/var/www/html"`
* **187열:** `ErrorLog "logs/error_log"`

**MariaDB 보안 설정 및 접속**

```bash
# MariaDB 초기 보안 설정 스크립트 실행
mysql_secure_installation

# MariaDB 접속
mysql -u root -p

```

**MariaDB SQL 명령어 및 자료형**

* **기본 SQL 명령어:**
* `SHOW DATABASES;`: 데이터베이스 목록 조회
* `USE mysql;`: 데이터베이스 선택
* `SHOW TABLES;`: 테이블 목록 조회
* `DESC user;`: 테이블 구조 확인
* `SELECT user, host FROM user;`: 특정 컬럼 데이터 조회


* **주요 데이터 타입:**
* **문자:** `CHAR(N)` (고정 길이), `VARCHAR(N)` (가변 길이), `TEXT` (장문)
* **숫자:** `INT` (정수), `BIGINT` (대용량 정수), `DECIMAL` (고정소수점)
* **날짜:** `DATE` (`YYYY-MM-DD`), `DATETIME` (`YYYY-MM-DD HH:MM:SS`)



```

```