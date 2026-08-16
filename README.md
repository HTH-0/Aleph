Aleph 과정 정리

---

```markdown
# Network Engineering Study Archive

네트워크 인프라 기초부터 라우팅/스위칭 및 보안 설정 실습을 체계적으로 아카이빙하는 리포지토리입니다.

---

## 🗓️ 학습 인덱스 (Study Index)

| 일차 | 주제 | 핵심 키워드 | 바로가기 |
| :--- | :--- | :--- | :--- |
| **Day 01** | 네트워크 기초 & 시스코 기본 설정 | IT 인프라, OSI 7계층, IP 주소 체계, CLI 모드 구조, Line 보안, Default Gateway | [📖 Day 01 바로가기](./Day01/README.md) |
| **Day 03** | 서브네팅 & RIP 라우팅 | AND 연산, 서브네팅 공식, DTE/DCE, RIP v1/v2, 3-Router 연동 실습 | [📖 Day 03 바로가기](./Day03/README.md) |
| **Day 04** | EIGRP 라우팅 & VLAN/DHCP | EIGRP 메트릭(K-Value), Router-on-a-Stick, IEEE 802.1Q, DHCP Pool & Relay | [📖 Day 04 바로가기](./Day04/README.md) |

---

## 📋 폴더 구조 표준 (Repository Structure)

```text
DayXX/
├── README.md              # [일차 허브] 전체 요약 및 하위 문서 링크
├── Theory/
│   └── README.md          # [이론 정리] 통신 모델, 프로토콜 동작 원리, 계산 공식
└── Lab/ (또는 Cisco/)
    ├── README.md          # [실습 가이드] 토폴로지 IP 플랜, CLI 구성 명령어, 검증 절차
    ├── topology.png       # 토폴로지 구성도 캡처
    └── practice.pkt       # 패킷 트레이서 실습 원본 파일

```

---

## ⚙️ 문서 정리 자동화 프롬프트 (Documentation Prompt)

> 수업 내용이나 메모를 복습 정리할 때, 아래 프롬프트 블록을 복사하여 AI에게 전달하면 동일한 표준 포맷의 마크다운으로 산출됩니다.

```text
[역할]
너는 네트워크 엔지니어링 기술 문서 작성 전문가야.
내가 전달하는 수업 메모 및 실습 내용을 기반으로 마크다운(.md) 문서를 작성해 줘.

[작성 규칙]
1. 불필요한 이모티콘은 제거하고, 담백하고 기술적인 어조를 유지할 것.
2. 기호 사용을 최소화하고, 가독성을 위한 마크다운 테이블과 코드 블록(cisco/text)을 적극 활용할 것.
3. 원본 내용의 핵심 개념, 파라미터, 명령어, 설명은 빠짐없이 100% 보존할 것.
4. 문서는 다음 3개 영역으로 분리하여 출력할 것:

---
[출력 1] DayXX/README.md (허브 문서)
- 일차 대제목 및 핵심 키워드
- Theory / Lab 링크 테이블 (구분, 주제, 핵심 키워드, 링크)

[출력 2] DayXX/Theory/README.md (이론 문서)
- 기초 개념, 동작 원리, 비교 분석 테이블, 계산 공식

[출력 3] DayXX/Lab/README.md (실습 문서)
- IP 및 인터페이스 할당 테이블
- 하드웨어 구성 및 단계별 CLI 설정 명령어
- 상태 확인 및 검증(Troubleshooting) 명령어
---

[정리할 원본 내용]:
(여기에 수업 메모, 텍스트 내용 붙여넣기)

```

```

```
