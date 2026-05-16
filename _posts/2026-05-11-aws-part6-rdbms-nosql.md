---
layout: post
title: "🗄️ AWS 기초 스터디 Part 6. RDBMS vs NoSQL — 어떤 DB를 써야 할까?"
date: 2026-05-11 14:00:00 +0900
categories: [Study, Database]
tags: [database, rdbms, nosql, dynamodb, rds, aurora, aws]
---

## 데이터베이스란?

> **비유:** 데이터를 체계적으로 보관하는 창고입니다.  
> 창고도 종류가 다양하듯이 — 정리된 선반(RDBMS), 자유롭게 쌓는 창고(NoSQL) 등 상황에 맞는 방식이 다릅니다.

데이터베이스는 크게 두 가지 계열로 나뉩니다:
- **RDBMS:** 행과 열로 구성된 테이블에 정형화된 데이터를 저장
- **NoSQL:** 스키마 없이 다양한 형식의 데이터를 유연하게 저장

---

## RDBMS (Relational Database Management System)

> **비유:** 엑셀 스프레드시트처럼 **행(Row)과 열(Column)**로 데이터를 정리하는 방식입니다.  
> 각 시트(테이블)가 서로 연결(JOIN)될 수 있어서, 복잡한 데이터 관계를 표현하기에 적합합니다.

### 특징

- 데이터를 **2차원 테이블** 형태로 저장
- **SQL(Structured Query Language)**을 사용해 복잡한 쿼리와 JOIN 연산 가능
- 테이블 간의 **관계(Relation)**를 정의하여 데이터 중복 최소화
- 데이터를 넣기 전에 **스키마(구조)를 먼저 정의**해야 함

```sql
-- 예시: 주문과 고객 테이블을 JOIN해서 조회
SELECT 고객.이름, 주문.상품명, 주문.금액
FROM 주문
JOIN 고객 ON 주문.고객ID = 고객.ID
WHERE 주문.날짜 = '2026-05-16';
```

### ACID 트랜잭션

> **트랜잭션(Transaction):** 하나의 논리적 작업 단위 (예: 계좌 이체 = 출금 + 입금)

RDBMS는 ACID 원칙으로 데이터의 **신뢰성을 철저하게 보장**합니다.

| 원칙 | 이름 | 설명 | 비유 |
|---|---|---|---|
| **A** | Atomicity (원자성) | All or Nothing — 전부 성공하거나 전부 실패 | 계좌 이체: 출금만 되고 입금 안 되면 안 됨 |
| **C** | Consistency (일관성) | 트랜잭션 전후에 데이터가 항상 올바른 상태 유지 | 이체 후에도 잔고가 음수가 되면 안 됨 |
| **I** | Isolation (고립성) | 동시에 실행되는 트랜잭션들이 서로 간섭하지 않음 | 두 사람이 동시에 같은 좌석을 예약하면 하나만 성공 |
| **D** | Durability (지속성) | 성공한 트랜잭션 결과는 영구적으로 반영 | 이체가 완료되면 서버가 꺼져도 데이터는 남음 |

### AWS RDBMS 서비스

| 서비스 | 설명 |
|---|---|
| **Amazon RDS** | MySQL, PostgreSQL, MariaDB, Oracle, SQL Server 지원 |
| **Amazon Aurora** | AWS 자체 개발 고성능 DB (MySQL/PostgreSQL 호환, RDS보다 최대 5배 빠름) |
| **Amazon Redshift** | 대용량 데이터 분석용 데이터 웨어하우스 |

### 주요 사용 사례

- 쇼핑몰 주문/결제 시스템
- 은행 계좌 관리
- 병원 의료 기록
- 데이터 무결성과 정확성이 매우 중요한 대부분의 메인 DB

---

## NoSQL (Not Only SQL)

> **비유:** 자유형식 메모장처럼 형식에 구애받지 않고 원하는 대로 데이터를 저장하는 방식입니다.  
> 정해진 틀이 없으니 자유롭고 유연하지만, 복잡한 관계 표현은 어렵습니다.

### 특징

- **스키마 없음**: 미리 구조를 정하지 않아도 됨 → 유연한 데이터 저장
- 구조가 가볍고 **수평 확장(Scale-out)**이 용이
- 대부분 SQL을 사용하지 않으며, 복잡한 JOIN보다 **단순하고 빠른 조회**에 최적화

### NoSQL 주요 종류

| 종류 | 설명 | 예시 | AWS 서비스 |
|---|---|---|---|
| **Key-Value** | Key로 Value를 바로 조회 (초고속) | 딕셔너리/해시맵과 유사 | DynamoDB, ElastiCache |
| **Document** | JSON 형태의 문서로 데이터 저장 | 각 사용자 프로필이 하나의 문서 | DocumentDB, MongoDB |
| **Graph** | 데이터 간 '연결(관계)'을 중심으로 저장 | SNS 친구 관계, 추천 알고리즘 | Amazon Neptune |
| **Wide Column** | 행마다 다른 컬럼을 가질 수 있는 구조 | 시계열 데이터, 로그 | Keyspaces (Cassandra) |

### BASE 모델

RDBMS의 엄격한 ACID와 달리, NoSQL은 **분산 환경에 유연한 BASE 모델**을 따릅니다.

| 원칙 | 이름 | 설명 |
|---|---|---|
| **BA** | Basically Available | 일부 서버가 장애나도 나머지 서버에서 응답 가능 |
| **S** | Soft-state | DB 상태가 고정되지 않고 자연스럽게 변화할 수 있음 |
| **E** | Eventual Consistency | 즉시 일관성은 없지만, 시간이 지나면 결국 모든 노드가 동일한 데이터를 갖게 됨 |

> **Eventual Consistency 비유:**  
> SNS에서 게시글을 올리면, 서울 서버에는 즉시 반영되지만 미국 서버에는 잠깐 뒤에 반영될 수 있습니다.  
> 하지만 결국(Eventually)에는 전 세계 서버에 동일하게 반영됩니다.

### AWS NoSQL 서비스

| 서비스 | 종류 | 주요 용도 |
|---|---|---|
| **Amazon DynamoDB** | Key-Value / Document | 게임 랭킹, 쇼핑카트, 세션 데이터 |
| **Amazon ElastiCache** | Key-Value (Redis/Memcached) | 캐시, 세션, 실시간 랭킹 |
| **Amazon MemoryDB** | Key-Value (Redis 호환) | 고속 인메모리 DB (영속성 지원) |
| **Amazon Neptune** | Graph | SNS 관계, 추천 시스템 |

---

## RDBMS vs NoSQL 비교

| 항목 | RDBMS | NoSQL |
|---|---|---|
| 데이터 구조 | 테이블 (행 + 열) | 다양 (Key-Value, Document 등) |
| 스키마 | 엄격하게 사전 정의 | 유연 (없거나 느슨함) |
| 확장 방식 | 수직 확장 (Scale-up: 서버 사양 업그레이드) | 수평 확장 (Scale-out: 서버 대수 추가) |
| 일관성 | ACID (강함) | BASE (느슨함) |
| 조회 성능 | JOIN 포함 복잡한 쿼리에 강함 | 단순 조회에 매우 빠름 |
| 유연성 | 낮음 (스키마 변경 어려움) | 높음 (필드 추가가 자유로움) |
| 대표 서비스 | MySQL, PostgreSQL, Aurora | DynamoDB, MongoDB, Redis |

---

## 언제 무엇을 써야 할까?

### RDBMS를 선택해야 할 때

- 데이터 간의 **관계가 복잡**하고 JOIN이 필요한 경우
- **데이터 무결성**이 최우선인 경우 (금융, 의료, 주문)
- 복잡한 **집계 쿼리**가 필요한 경우

### NoSQL을 선택해야 할 때

- **초고속 읽기/쓰기**가 필요한 경우 (캐시, 게임 랭킹)
- 데이터 구조가 **유동적**이고 자주 바뀌는 경우
- 수천만~수억 건의 **대용량 데이터**를 다루는 경우
- 지리적으로 분산된 **글로벌 서비스**

### 실무에서는?

대부분의 서비스는 **RDBMS + NoSQL을 함께** 사용합니다.

```
[예: 쇼핑몰 아키텍처]

주문/결제 데이터 → RDS (Aurora)  ← 정확성이 중요
상품 목록 캐시   → ElastiCache (Redis)  ← 빠른 응답
장바구니         → DynamoDB  ← 유연한 구조, 고속
상품 추천        → Neptune (Graph DB)  ← 관계 기반
```

---

## 정리

```
강한 일관성이 필요하다 → RDBMS (Aurora, RDS)
빠른 속도가 필요하다  → NoSQL (DynamoDB, ElastiCache)
둘 다 필요하다        → 혼합해서 사용
```

> **시리즈 마무리:**  
> Part 1~6을 통해 AWS 핵심 서비스부터 네트워크, DNS, 캐싱, 암호화, DB까지 살펴봤습니다.  
> 각 개념은 독립적이지 않고 서로 연결되어 있으니, 다시 돌아와서 복습하며 연결고리를 찾아보세요!
