# Day 8: MariaDB CRUD 심화, HTTPS/SSL 적용, FTP/FTPS/SFTP 및 DNS 서버 구축

## 1. SQL CRUD 심화 및 데이터 정의/조작 (DDL & DML)

### CRUD와 SQL 명령어 매핑
* **Create (생성):** `INSERT`
* **Read (조회):** `SELECT`
* **Update (수정):** `UPDATE`
* **Delete (삭제):** `DELETE`

### 데이터베이스 및 테이블 구조 제어 (DDL)
```sql
-- 데이터베이스 및 테이블 구조 조회
SHOW DATABASES;
SHOW TABLES;
DESC [테이블명];

-- 데이터베이스 생성 및 선택
CREATE DATABASE school;
USE school;

-- 테이블 생성 (student)
CREATE TABLE student (
    name VARCHAR(8),
    date INT(4),
    phone INT(10)
);

```

### ALTER 문을 활용한 테이블 구조 수정

* **컬럼 추가:**
```sql
ALTER TABLE student ADD address VARCHAR(8);
ALTER TABLE student ADD age INT(3) AFTER date;     -- 특정 컬럼 뒤에 추가 (맨 앞은 FIRST)
ALTER TABLE student ADD (age2 INT(3), age3 INT(3)); -- 다중 컬럼 추가

```


* **컬럼 위치 및 속성 수정:**
```sql
ALTER TABLE student MODIFY name VARCHAR(8) FIRST;       -- 위치를 맨 앞으로 변경
ALTER TABLE student MODIFY age INT(3) AFTER address;    -- 위치를 특정 컬럼 뒤로 변경
ALTER TABLE student CHANGE mbti subject VARCHAR(4);     -- 컬럼명 변경 (기존이름 새이름 자료형)

```


* **컬럼 삭제:**
```sql
ALTER TABLE student DROP age2;

```



### 데이터 조작어 (DML) 및 객체 삭제

* **INSERT (데이터 삽입):**
```sql
INSERT INTO student VALUES ('kim', 2026, 1012345678, 'seoul', 20);
INSERT INTO student (name, date) VALUES ('lee', 2026);

```


* **UPDATE (데이터 수정):**
```sql
UPDATE student SET date = 2026;                               -- 전체 행의 date 컬럼값 일괄 수정
UPDATE student SET address = 'daegu' WHERE name = 'kim';      -- 특정 행의 단일 컬럼 수정
UPDATE student SET name = 'park', date = 2020 WHERE name = 'kim'; -- 다중 컬럼 동시 수정

```


* **DELETE & DROP (데이터 및 객체 삭제):**
```sql
DELETE FROM student WHERE name = 'park'; -- 특정 행 삭제
DELETE FROM student;                     -- 테이블 내 전체 데이터 삭제
DROP TABLE student;                      -- 테이블 객체 완전 삭제
DROP DATABASE school;                    -- 데이터베이스 객체 완전 삭제

```



### DDL/DML 종합 실습 예제 (Teacher 테이블)

```sql
-- 1. Database 및 Table 생성
CREATE DATABASE School;
USE School;
CREATE TABLE Teacher (
    Name VARCHAR(10),
    Age INT(3),
    Subject VARCHAR(8)
);

-- 2. 컬럼 추가 (Address, Phone)
ALTER TABLE Teacher ADD Address VARCHAR(12);
ALTER TABLE Teacher ADD Phone INT(12) AFTER Age;

-- 3. 데이터 입력 (3건)
INSERT INTO Teacher VALUES ('ga', 13, 1234123412, 'math', 'daegu');
INSERT INTO Teacher VALUES ('na', 25, 2345234523, 'english', 'seoul');
INSERT INTO Teacher VALUES ('da', 30, 3456345634, 'science', 'busan');

-- 4. 데이터 수정
UPDATE Teacher SET Subject = 'Security';              -- 전체 Subject 변경
UPDATE Teacher SET name = 'gga' WHERE name = 'ga';    -- 특정 데이터 수정

```

---

## 2. Apache 웹 서버 SSL/TLS (HTTPS) 보안 적용

### OpenSSL 키 및 자체 서명(Self-Signed) 인증서 발급

```bash
# 인증서 저장용 디렉터리 생성 및 이동
mkdir -p /etc/httpd/conf/ssl
cd /etc/httpd/conf/ssl

# [개념] Triple-DES로 암호화된 RSA 개인키 생성 방식 (참고)
# openssl genrsa -des3 -out test.com.key

# 암호 없는(nodes) 2048비트 RSA 개인키 및 1년 유효기간 X.509 인증서 발급
openssl req -newkey rsa:2048 -nodes -keyout /etc/httpd/conf/ssl/test.com.key \
-x509 -days 365 -out /etc/httpd/conf/ssl/test.com.crt

```

**인증서 발급 시 입력 정보:**

* Country Name: `KR`
* State or Province Name: `Daegu`
* Locality Name: `Daegu`
* Organization Name: `KIT`
* Organizational Unit Name: `Security`
* Common Name: `test.com`
* Email Address: `admin@test.com`

```bash
# 발급된 인증서 내용(텍스트 평문) 검증
openssl x509 -noout -text -in test.com.crt

```

### mod_ssl 모듈 설치 및 Apache 설정 (`/etc/httpd/conf.d/ssl.conf`)

```bash
# ssl_module 유무 확인 후 미설치 시 패키지 설치
httpd -M | grep ssl
dnf install -y mod_ssl

```

* **`/etc/httpd/conf.d/ssl.conf` 주요 설정 라인:**
* Line 5: `Listen 443 https`
* Line 33: `SSLCryptoDevice builtin`
* Line 54: `SSLEngine on` (필수 활성화)
* Line 85: `SSLCertificateFile /etc/httpd/conf/ssl/test.com.crt`
* Line 93: `SSLCertificateKeyFile /etc/httpd/conf/ssl/test.com.key`



```bash
# 서비스 재시작 및 방화벽 설정
systemctl restart httpd
firewall-cmd --permanent --add-service=https
firewall-cmd --reload
firewall-cmd --list-all

```

> *참고: 자체 서명(Self-Signed) 인증서를 사용하므로 브라우저 접속 시 '주의 요함(보안 경고)'이 표시됩니다.*

---

## 3. 파일 전송 서비스 (FTP / FTPS / SFTP)

### FTP 포트 구성 및 연결 모드

* **포트 구성:**
* **21번 포트:** 제어 채널 (로그인, 명령어 송수신)
* **20번 포트:** 데이터 채널 (Active 모드 파일 전송)
* **1024번 이상 임의 포트:** 데이터 채널 (Passive 모드 파일 전송)


* **연결 모드:**
* **Active 모드:** 클라이언트(21번 접속) $\rightarrow$ 서버(20번 포트를 통해 클라이언트로 역방향 연결)
* **Passive 모드:** 클라이언트(21번 접속) $\rightarrow$ 클라이언트가 서버의 안내 포트로 직접 순방향 연결



### 1) vsftpd 기본 구축

```bash
dnf install -y vsftpd

```

* **`/etc/vsftpd/vsftpd.conf` 주요 설정:**
* `anonymous_enable=NO`: 익명 사용자 접속 차단
* `local_enable=YES`: 로컬 계정 접속 허용
* `write_enable=YES`: 쓰기(업로드, 수정, 삭제) 기능 허용
* `dirmessage_enable=YES`: 디렉터리 안내 메시지 활성화
* `xferlog_enable=YES`: 전송 로그 기록
* `connect_from_port_20=YES`: 20번 데이터 전송 포트 활성화
* `#idle_session_timeout=600`: 주석 해제 시 600초 무응답 시 자동 접속 종료
* `ascii_upload_enable=YES` / `ascii_download_enable=YES`: 텍스트 파일 개행문자 자동 변환
* `listen=NO`
* `max_client=10`: 최대 동시 접속자 수 제한 (하단 추가)
* `local_max_rate=100000`: 로컬 사용자 전송 속도 제한 (하단 추가)
* `use_localtime=YES`: 타임스탬프를 서버 현지 시간으로 표기 (하단 추가)



```bash
# 서비스 구동 및 방화벽 등록
systemctl start vsftpd
systemctl enable vsftpd
firewall-cmd --permanent --add-service=ftp
firewall-cmd --reload

# 실습 계정 및 테스트 디렉터리 생성
useradd ftptest
passwd ftptest
cd /home/ftptest
mkdir document music
touch ftptest.txt

```

### 2) FTPS (FTP over SSL/TLS) 암호화 구성

```bash
cd /etc/vsftpd
openssl req -newkey rsa:2048 -nodes -keyout /etc/vsftpd/vsftpd.pem \
-x509 -days 365 -out /etc/vsftpd/vsftpd.pem

```

* **인증서 정보 입력:** Country: `kr`, State/Locality: `daegu`, Org: `kit`, Unit: `security`, CN: `test.com`, Email: `test@test.com`
* **`/etc/vsftpd/vsftpd.conf` 하단 추가 내용:**
```text
# SSL Configuration
ssl_enable=YES
rsa_cert_file=/etc/vsftpd/vsftpd.pem
force_local_logins_ssl=YES
ssl_tlsv1=YES
allow_anon_ssl=NO
pasv_enable=YES
pasv_min_port=50000
pasv_max_port=55000

```



```bash
# Passive 포트 범위 개방 및 서비스 재시작
firewall-cmd --permanent --add-port=50000-55000/tcp
firewall-cmd --reload
systemctl restart vsftpd

```

### 3) FileZilla 클라이언트 접속 설정

* **일반 FTP:** 호스트 `[서버IP]`, 사용자명 `ftptest`, 비밀번호 입력
* **FTPS:** 사이트 관리자 $\rightarrow$ 새 사이트 $\rightarrow$ 프로토콜 `FTP` $\rightarrow$ 암호화 `TLS를 통한 명시적 FTP가 가능한 경우 사용` $\rightarrow$ 호스트 `ftps://[서버IP]`
* **SFTP:** 사이트 관리자 $\rightarrow$ 새 사이트 $\rightarrow$ 프로토콜 `SFTP` $\rightarrow$ 호스트 `sftp://[서버IP]`

### 4) SFTP 및 SSH 서버 설정

* `systemctl status sshd`: SSH 및 SFTP 서비스 구동 상태 확인
* `/etc/ssh/sshd_config` 파일의 `Subsystem sftp internal-sftp` 설정을 통해 OpenSSH 기반 SFTP 서비스 관리

---

## 4. DNS (Domain Name System) 서버 구축

### DNS 개념

* 도메인 이름(예: `google.com`)과 IP 주소(예: `142.250.196.142`)를 상호 변환해 주는 시스템
* 클라이언트는 로컬의 `hosts` 파일을 먼저 조회한 후 DNS 서버로 질의하므로 호스트 파일 보안이 중요함

### 패키지 설치

```bash
dnf install -y bind bind-utils

```

### 주 설정 파일 수정 (`/etc/named.conf`)

* **Line 11:** `listen-on port 53 { any; };` (127.0.0.1 $\rightarrow$ any로 변경)
* **Line 13:** `directory "/var/named";` (존 파일 기본 디렉터리)
* **Line 19:** `allow-query { any; };` (localhost $\rightarrow$ any로 변경)
* **Line 31:** `recursion yes;` (재귀적 질의 허용: 외부 도메인 질의 시 상위 DNS에 질의하여 최종 IP를 찾아 안내, `no` 설정 시 자신이 관리하는 Zone 파일 외 외부 도메인은 응답 거부)
* **하단 도메인 Zone 블록 추가:**
```text
zone "htht.com" IN {
    type master;
    file "htht.com.zone";
    allow-update { none; };
};

```



### 설정 검증 및 존 파일 생성

```bash
# named.conf 문법 오류 검사
named-checkconf /etc/named.conf

# 루트 디렉터리로 이동 및 zone 파일 생성
cd /var/named
touch htht.com.zone

```
