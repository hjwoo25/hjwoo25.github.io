---
layout: post
title: "🌍 AWS 기초 스터디 Part 3. DNS — 인터넷의 전화번호부"
date: 2026-05-11 11:00:00 +0900
categories: [Study, Network]
tags: [dns, network, route53, domain]
---

## DNS란?

> **비유:** 전화번호부와 같습니다.  
> "홍길동"이라는 이름으로 찾으면 전화번호(숫자)를 알려주듯이,  
> DNS는 `google.com`이라는 이름을 `142.250.207.206`이라는 IP 주소로 바꿔줍니다.

**DNS (Domain Name System)**는 사람이 읽기 편한 **도메인 이름**을 컴퓨터가 사용하는 **IP 주소**로 변환해주는 시스템입니다.

우리가 `https://google.com`을 주소창에 입력하면, 컴퓨터는 먼저 DNS에 "google.com의 IP가 뭐야?"라고 물어보고, 그 IP로 실제 접속합니다.

DNS는 **ICANN**(국제 인터넷 주소 관리 기구)에서 총괄 관리하는 전 세계적인 시스템입니다.

---

## 주요 용어 정리

| 용어 | 설명 | 예시 |
|---|---|---|
| **도메인 (Domain)** | IP와 매핑되는 사람이 읽을 수 있는 문자열 | `google.com` |
| **APEX 도메인** | 앞에 추가 문자열 없는 순수 최상위 도메인 | `example.com` |
| **서브도메인 (Subdomain)** | 도메인 앞에 추가 문자열이 붙은 형태 | `mail.google.com` |
| **레코드 (Record)** | 도메인이 어떤 데이터와 매핑되는지 기록 | A레코드, MX레코드 |
| **도메인 존 (Zone)** | 도메인의 레코드들을 담은 데이터 영역 | example.com 존 |
| **존 파일 (Zone File)** | 도메인 존 정보를 저장한 텍스트 파일 | - |
| **DNS 쿼리 (Query)** | 도메인에 해당하는 IP 정보를 요청하는 질의 | - |
| **TTL (Time To Live)** | DNS 응답을 얼마나 오래 캐시할지 나타내는 시간 | 300초 |

---

## DNS 레코드의 종류

DNS는 단순히 도메인 → IP 변환만 하는 게 아닙니다. 다양한 레코드 타입이 있습니다.

| 레코드 타입 | 역할 | 예시 |
|---|---|---|
| **A** | 도메인 → IPv4 주소 매핑 | `example.com → 1.2.3.4` |
| **AAAA** | 도메인 → IPv6 주소 매핑 | `example.com → 2001:db8::1` |
| **CNAME** | 도메인 → 다른 도메인으로 별칭 | `www.example.com → example.com` |
| **MX** | 메일 서버 지정 | `example.com → mail.example.com` |
| **NS** | 이 도메인의 네임서버 지정 | `example.com → ns1.example.com` |
| **TXT** | 임의 텍스트 (도메인 인증, SPF 등) | 도메인 소유권 인증 문자열 |
| **SOA** | 존(Zone)의 기본 정보 (관리자 정보 등) | - |

> **CNAME 비유:**  
> `www.example.com`을 `example.com`의 별명(닉네임)으로 등록해두면, 어느 쪽으로 접속해도 같은 서버로 연결됩니다.

---

## DNS 서버의 종류

### DNS Resolver (리졸버)

> **비유:** 도서관에서 원하는 책을 대신 찾아주는 사서입니다.  
> 이용자(사용자 컴퓨터)가 "google.com IP 알려줘"라고 하면, 사서(리졸버)가 여러 곳을 뒤져서 최종 답을 가져옵니다.

- 사용자와 네임서버 사이에서 **IP 주소를 대신 조회**해오는 서버입니다.
- 보통 통신사(ISP)나 구글(`8.8.8.8`), Cloudflare(`1.1.1.1`)가 운영합니다.
- 한 번 조회한 결과는 **TTL 시간 동안 캐시**에 저장해뒀다가 재사용합니다.

### 네임서버 (Name Server, NS)

- DNS 쿼리를 존 파일 기반으로 응답하는 서버입니다.
- **Authoritative NS:** 존 파일 원본을 직접 보유한 최종 결정권자
- **Non-authoritative NS:** 다른 서버를 조회 후 캐시해 응답하는 서버

---

## DNS 계층 구조와 동작 원리

전 세계의 방대한 DNS 쿼리를 처리하기 위해 **트리 계층 구조**를 사용합니다.

```
.  (루트)
├── com  (TLD)
│   ├── google.com  (도메인)
│   │   └── mail.google.com  (서브도메인)
│   └── example.com
└── kr  (TLD)
    └── naver.com ... (실제로는 naver.com이지만 비유)
```

### 실제 조회 과정 (google.com 접속 시)

```
① 내 컴퓨터: "google.com IP 알려줘!" → DNS Resolver
② Resolver → Root 서버: ".com 담당 TLD가 어디야?"
③ Root 서버: "Verisign이 .com TLD 관리해, 주소는 여기야"
④ Resolver → Verisign(TLD 서버): "google.com 담당 NS가 어디야?"
⑤ TLD 서버: "구글 NS 서버 주소는 ns1.google.com이야"
⑥ Resolver → Google NS: "google.com IP가 뭐야?"
⑦ Google NS: "142.250.207.206!" (Authoritative 응답)
⑧ Resolver → 내 컴퓨터: "142.250.207.206" (캐시에도 저장)
```

> **💡 왜 이렇게 복잡하게 나눌까요?**  
> 전 세계에 수십억 개의 도메인이 있습니다. 만약 하나의 서버가 모두 관리한다면 서버가 터질 겁니다.  
> 계층 구조로 나누면 각 서버가 담당 영역만 관리하면 돼서 효율적입니다.

### 각 계층 상세 설명

**① DNS Root (루트 서버)**
- 리졸버가 처음 찾아가는 곳입니다.
- IANA에서 조율하는 **13개(A~M) 관리 주체**가 운영 (실제로는 애니캐스트로 수천 대)
- `Root Hints File`: 루트 서버 주소가 담긴 파일, 리졸버에 하드코딩되어 있음

**② TLD (Top Level Domain) 서버**
- `.com`, `.org`, `.net`, `.kr` 등 최상위 도메인을 관리합니다.
- 각 TLD별 Registry가 관리 (`.kr` → KISA, `.com` → Verisign)
- 도메인별 담당 NS 서버 주소를 알려줍니다.

**③ NS 서버 (Authoritative Name Server)**
- 해당 도메인의 실제 레코드(IP 주소 등)를 보유합니다.
- 최종 답변을 내려주는 곳입니다.

---

## TTL과 DNS 캐싱

> **비유:** 한 번 전화번호를 찾아봤으면, 메모해뒀다가 다음에는 바로 씁니다.  
> DNS도 같은 방식으로, 한 번 조회한 결과를 TTL 시간 동안 캐시해둡니다.

- **TTL(Time To Live):** DNS 응답을 캐시에 얼마나 오래 저장할지 나타내는 초 단위 값
- TTL이 `300`이면 5분 동안 캐시 → 5분 뒤에 다시 조회

**TTL 설정의 trade-off:**

| TTL | 장점 | 단점 |
|---|---|---|
| 짧게 (60초) | IP 변경이 빠르게 반영 | 조회 횟수 많아짐 (느림, 서버 부하) |
| 길게 (86400초) | 조회 속도 빠름, 부하 적음 | IP 변경 시 반영에 시간 걸림 |

> **실무 팁:** 도메인 이전이나 서버 교체를 앞두고 있다면, **미리 TTL을 짧게 줄여두세요!**  
> 그래야 IP 변경 후 사용자들이 빠르게 새 서버로 연결됩니다.

---

## 도메인 등록 방법

```
[도메인 등록 흐름]
사용자 → Registrar(도메인 등록 대행) → TLD Registry → Root 서버
```

- **Registrar(등록 대행):** ICANN 인증을 받고 도메인을 대신 등록해주는 업체
  - 예: 가비아, GoDaddy, Cafe24, **AWS Route 53**
- 등록 시 TLD Registry에 내 **NS 서버 주소를 등록**합니다.
- NS 서버는 직접 구축하거나, 가비아·**AWS Route 53** 같은 DNS 호스팅 서비스를 이용합니다.

### AWS Route 53

AWS의 DNS 서비스로, 다음 기능을 제공합니다:

| 기능 | 설명 |
|---|---|
| 도메인 등록 | Registrar 역할 |
| DNS 호스팅 | Authoritative NS 역할 |
| 헬스 체크 | 서버 장애 감지 및 자동 우회 |
| 라우팅 정책 | 지리적 위치, 가중치, 지연 시간 기반 라우팅 |

> **Route 53 이름의 유래:** DNS가 기본적으로 사용하는 포트 번호가 **53번**이라서 Route 53입니다.

---

> **다음 글:** [Part 4. 캐싱 — 빠른 응답의 비결](/posts/aws-part4-caching/)에서 자주 쓰는 데이터를 더 빠르게 제공하는 방법을 알아봅니다.
