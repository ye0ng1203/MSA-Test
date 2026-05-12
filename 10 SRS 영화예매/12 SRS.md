# 온라인 영화 예매 시스템 — SRS (Software Requirements Specification)

# 1. Introduction

## 1.1 Purpose

본 문서는 “온라인 영화 예매 시스템”의 기능적 요구사항(Function Requirements)과 비기능적 요구사항(Non-Functional Requirements)을 정의한다.

본 시스템은 사용자가 영화 조회, 상영 일정 확인, 좌석 선택, 결제 및 예매 완료까지 수행할 수 있는 대규모 온라인 예매 플랫폼을 목표로 한다.

또한 본 시스템은 MSA(Microservice Architecture), Event-Driven Architecture, 고가용성(HA), 확장성(Scalability)을 고려하여 설계된다.

---

## 1.2 Scope

시스템은 다음 기능을 제공한다.

* 영화 조회
* 극장 및 상영 일정 조회
* 좌석 상태 조회
* 좌석 선점 및 예약
* 결제 처리
* 예매 취소 및 환불
* 사용자 인증 및 회원 관리
* 알림 발송
* 관리자 운영 기능
* 이벤트 기반 서비스 연동

---

# 2. System Overview

온라인 영화 예매 시스템은 다수의 사용자가 동시에 접속하여 실시간으로 좌석을 예약하는 분산 시스템이다.

시스템은 다음 특성을 가진다.

* 실시간 좌석 선점 처리
* 대량 트래픽 처리
* 분산 트랜잭션 대응
* 이벤트 기반 비동기 처리
* 서비스 단위 독립 배포
* 장애 격리 및 자동 복구

---

# 3. Actors

| Actor                | Description |
| -------------------- | ----------- |
| Guest User           | 비회원 사용자     |
| Registered User      | 회원 사용자      |
| Admin                | 시스템 관리자     |
| Theater Operator     | 극장 운영자      |
| Payment Gateway      | 외부 결제 시스템   |
| Notification Service | 외부 알림 시스템   |
| Identity Provider    | 인증 제공 시스템   |

---

# 4. Functional Requirements

# 4.1 User Management

## FR-001 회원 가입

* 사용자는 이메일/휴대폰 기반 회원 가입 가능해야 한다.
* OAuth 기반 소셜 로그인 지원 가능해야 한다.

## FR-002 로그인

* 사용자는 JWT/OAuth 기반 인증 가능해야 한다.
* Access Token 및 Refresh Token 지원해야 한다.

## FR-003 사용자 정보 조회

* 사용자는 자신의 예매 내역 조회 가능해야 한다.

---

# 4.2 Movie Management

## FR-004 영화 목록 조회

* 사용자는 현재 상영 영화 목록 조회 가능해야 한다.

## FR-005 영화 상세 조회

* 영화 정보, 상영 시간, 등급, 러닝타임 조회 가능해야 한다.

## FR-006 영화 검색

* 제목, 장르, 배우 기준 검색 가능해야 한다.

---

# 4.3 Theater & Schedule Management

## FR-007 극장 조회

* 지역 기반 극장 목록 조회 가능해야 한다.

## FR-008 상영 일정 조회

* 특정 영화의 상영 시간표 조회 가능해야 한다.

## FR-009 좌석 배치도 조회

* 상영관 좌석 구조 조회 가능해야 한다.

---

# 4.4 Seat Reservation

## FR-010 좌석 상태 조회

* 사용자는 실시간 좌석 상태 조회 가능해야 한다.

## FR-011 좌석 선점

* 사용자는 좌석 임시 점유 가능해야 한다.
* 동일 좌석 중복 선점 방지해야 한다.

## FR-012 좌석 점유 만료

* 일정 시간 내 결제 미완료 시 자동 해제해야 한다.

## FR-013 예매 확정

* 결제 성공 시 예약 상태 확정되어야 한다.

---

# 4.5 Payment

## FR-014 결제 요청

* 카드/간편결제 지원 가능해야 한다.

## FR-015 결제 승인 처리

* 외부 PG 연동 지원해야 한다.

## FR-016 결제 실패 처리

* 결제 실패 시 좌석 자동 해제되어야 한다.

## FR-017 환불 처리

* 예매 취소 시 환불 처리 가능해야 한다.

---

# 4.6 Notification

## FR-018 예매 완료 알림

* SMS/이메일/푸시 알림 발송 가능해야 한다.

## FR-019 결제 실패 알림

* 결제 실패 시 사용자에게 통지 가능해야 한다.

---

# 4.7 Administration

## FR-020 영화 등록

* 관리자는 영화 등록 가능해야 한다.

## FR-021 상영 일정 관리

* 상영 일정 생성/수정 가능해야 한다.

## FR-022 상영관 관리

* 좌석 구조 및 상영관 정보 관리 가능해야 한다.

## FR-023 모니터링

* 운영자는 예약/결제 상태 모니터링 가능해야 한다.

---

# 5. Non-Functional Requirements

# 5.1 Performance Requirements

## NFR-001 응답 속도

* 일반 조회 API 응답 시간은 평균 300ms 이하이어야 한다.
* 좌석 조회 API는 평균 500ms 이하이어야 한다.

## NFR-002 동시 접속 처리

* 최소 100,000 동시 사용자 처리 가능해야 한다.

## NFR-003 예약 처리량

* 초당 최소 10,000건 예약 요청 처리 가능해야 한다.

## NFR-004 이벤트 처리

* 이벤트 메시지 처리 지연은 평균 1초 이하이어야 한다.

---

# 5.2 Scalability Requirements

## NFR-005 수평 확장

* 서비스별 독립적 Scale-Out 가능해야 한다.

## NFR-006 트래픽 급증 대응

* 인기 영화 오픈 시 Burst Traffic 대응 가능해야 한다.

## NFR-007 캐시 확장

* 조회 트래픽 분산을 위한 Cache Layer 지원해야 한다.

---

# 5.3 Availability Requirements

## NFR-008 고가용성

* 시스템 가용성 99.95% 이상 유지해야 한다.

## NFR-009 장애 격리

* 특정 서비스 장애가 전체 시스템 중단으로 이어지지 않아야 한다.

## NFR-010 자동 복구

* 서비스 인스턴스 장애 시 자동 재기동 가능해야 한다.

---

# 5.4 Reliability Requirements

## NFR-011 데이터 정합성

* 좌석 중복 예약 발생하면 안 된다.

## NFR-012 이벤트 중복 처리

* 메시지 중복 소비(Idempotency) 지원해야 한다.

## NFR-013 Eventually Consistency

* 서비스 간 데이터는 Eventually Consistent 허용 가능해야 한다.

---

# 5.5 Security Requirements

## NFR-014 인증 및 인가

* OAuth2/JWT 기반 인증 적용해야 한다.

## NFR-015 개인정보 보호

* 개인정보 암호화 저장해야 한다.

## NFR-016 API 보안

* Rate Limiting 적용 가능해야 한다.

## NFR-017 감사 로그

* 주요 이벤트 Audit Logging 저장해야 한다.

---

# 5.6 Maintainability Requirements

## NFR-018 독립 배포

* 서비스별 독립 배포 가능해야 한다.

## NFR-019 코드 독립성

* 서비스 간 내부 코드 의존성 최소화해야 한다.

## NFR-020 API 버전 관리

* API Versioning 지원해야 한다.

---

# 5.7 Observability Requirements

## NFR-021 중앙 로그 수집

* Distributed Logging 지원해야 한다.

## NFR-022 분산 추적

* Distributed Tracing 지원해야 한다.

## NFR-023 메트릭 수집

* Prometheus 기반 Metrics 수집 가능해야 한다.

## NFR-024 실시간 모니터링

* 실시간 장애 탐지 가능해야 한다.

---

# 5.8 Data Requirements

## NFR-025 Database per Service

* 서비스별 독립 DB 사용해야 한다.

## NFR-026 캐시 전략

* 좌석 조회 및 영화 조회 캐시 가능해야 한다.

## NFR-027 CQRS 지원

* 읽기 모델 분리 가능해야 한다.

---

# 5.9 Messaging Requirements

## NFR-028 이벤트 기반 아키텍처

* 서비스 간 비동기 이벤트 통신 지원해야 한다.

## NFR-029 메시지 브로커

* Kafka/RabbitMQ 기반 메시징 지원해야 한다.

## NFR-030 DLQ 지원

* 실패 메시지 재처리 가능해야 한다.

---

# 6. Business Rules

| Rule ID | Description               |
| ------- | ------------------------- |
| BR-001  | 동일 좌석은 동시에 하나의 사용자만 점유 가능 |
| BR-002  | 좌석 선점 시간 초과 시 자동 해제       |
| BR-003  | 결제 성공 전까지 예약 확정 불가        |
| BR-004  | 결제 실패 시 좌석 즉시 반환          |
| BR-005  | 상영 시작 이후 환불 불가            |
| BR-006  | 관리자만 영화 및 상영 일정 수정 가능     |
| BR-007  | 예매 완료 시 알림 발송 필수          |

---

# 7. Assumptions & Constraints

* 외부 PG 시스템 장애 가능성 고려
* 네트워크 지연 및 메시지 중복 발생 가능성 고려
* 이벤트 순서 역전 가능성 고려
* 좌석 데이터 정합성이 최우선 요구사항
* 서비스 간 분산 트랜잭션 직접 사용 지양
* Saga 기반 보상 트랜잭션 허용

---

# 8. Future Expansion

향후 다음 기능 확장 가능해야 한다.

* 실시간 추천 시스템
* AI 기반 영화 추천
* 쿠폰/포인트 시스템
* 다국어 지원
* OTT 연계
* 동적 가격 정책(Dynamic Pricing)
* 멤버십 등급 시스템
* 광고/프로모션 플랫폼
* 실시간 대기열 시스템

---

# 9. Summary

온라인 영화 예매 시스템은 대규모 동시성 환경에서
실시간 좌석 예약과 결제 정합성을 유지해야 하는
고복잡도 분산 시스템이다.

따라서 본 시스템은 다음 원칙 기반으로 설계되어야 한다.

* MSA 기반 서비스 분리
* Event-Driven Architecture
* Eventually Consistent 구조
* CQRS 고려
* Saga 기반 트랜잭션 관리
* 고가용성 및 장애 격리
* 실시간 관측성 확보
* 대규모 트래픽 대응 가능한 확장성 확보
