---
layout: post
title: "🏰 AWS 기초 스터디 Part 27. S3 스토리지 클래스"
date: 2026-07-20 07:20:00 +0900
categories: [Study, AWS]
tags: [aws, s3, storage-class, glacier, standard, ia, intelligent-tiering]
---

모든 데이터를 비싼 곳에 보관할 필요는 없습니다. S3는 데이터의 접근 빈도와 중요도, 예산에 따라 선택할 수 있는 총 9가지의 스토리지 클래스를 제공합니다.

---

## S3 스토리지 클래스 한눈에 보기

> **비유:**
> - **Standard (옷장):** 자주 입는 옷을 손 닿는 곳에 둠. (보관비 비쌈, 꺼낼 때 쌈)
> - **IA (베란다 창고):** 계절 지난 옷 보관. (보관비 저렴, 꺼낼 때 비쌈)
> - **Glacier (시골 다락방):** 몇 년간 안 입을 한복 보관. (보관비 초저렴, 꺼낼 때 오래 걸림)

| 분류 | 스토리지 클래스 | 특징 |
|---|---|---|
| **기본 (빠름)** | **S3 Standard** | 99.99% 가용성, 최소 3개 AZ 분산 보관. 최소 보관 기간/용량 제한 없음[cite: 8]. |
| **특수 (매우 빠름)** | **S3 Express One Zone** | 1개 AZ에 위치. Standard 대비 응답속도 10배 빠르고 요청 비용 50% 저렴[cite: 8]. |
| **가끔 접근 (IA)** | **S3 Standard-IA** | 자주 쓰지 않지만 중요한 데이터. 최소 30일/128kb 보관 조건. 요청 비용이 비쌈[cite: 8]. |
| | **S3 One Zone-IA** | 중요하지 않은 데이터. 1개 AZ에만 보관하여 Standard-IA보다 보관비가 더 저렴[cite: 8]. |
| **장기 보관 (아카이브)** | **S3 Glacier Instant Retrieval** | 최소 90일 보관. 아카이브지만 즉시(Instant) 액세스 가능[cite: 8]. |
| | **S3 Glacier Flexible Retrieval** | 백업용. 데이터에 접근하는 데 분~시간 단위 소요[cite: 8]. |
| | **S3 Glacier Deep Archive** | 법적 보관 서류용. 가져오는 데 12~48시간 소요되나 보관비용이 가장 저렴[cite: 8]. |
| **자동화** | **S3 Intelligent-Tiering** | 머신 러닝이 접근 패턴을 분석하여 퍼포먼스 손해 없이 자동으로 클래스를 변경해 요금 최적화[cite: 8]. |
