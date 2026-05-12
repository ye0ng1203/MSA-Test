# 온라인 영화 예매 시스템 — MSA 기본 설계서

# 1. Architecture Overview

온라인 영화 예매 시스템은 대규모 동시성 환경에서:

* 영화 조회
* 실시간 좌석 선점
* 결제 처리
* 예약 확정
* 알림 발송

을 처리하는 Event-Driven MSA 시스템이다.

본 시스템은 다음 원칙 기반으로 설계한다.

* Domain-Driven Design (DDD)
* Bounded Context 기반 서비스 분리
* Database per Service
* Event-Driven Architecture
* Saga 기반 분산 트랜잭션
* CQRS 기반 조회 분리
* Eventually Consistent 구조
* 장애 격리(Fault Isolation)
* 독립 배포 및 독립 확장

---

# 2. Core Design Principles

| Principle              | Description      |
| ---------------------- | ---------------- |
| Domain Boundary First  | 기능이 아닌 도메인 기준 분리 |
| Database per Service   | 서비스별 독립 DB       |
| Event-Driven           | 서비스 간 비동기 이벤트 협력 |
| Loose Coupling         | 강한 동기 의존 최소화     |
| Fault Isolation        | 장애 전파 차단         |
| Independent Deployment | 독립 배포 가능         |
| Scalability            | 서비스별 독립 확장       |
| Eventually Consistent  | 최종 일관성 허용        |
| CQRS                   | 읽기/쓰기 분리 가능      |

---

# 3. High-Level Service Architecture

```text
[ Client / Mobile / Web ]
            |
            v
      [ API Gateway ]
            |
 ---------------------------------------------------------
 |        |         |         |         |        |       |
 v        v         v         v         v        v       v

User   Movie   Screening   Seat    Reservation Payment Notification
Svc    Svc      Svc         Svc        Svc       Svc        Svc

 ---------------------------------------------------------
                     |
                     v
              [ Event Broker ]
               (Kafka/RabbitMQ)

 ---------------------------------------------------------
 |                  Async Events                         |
 ---------------------------------------------------------
```

---

# 4. Bounded Context & Service Boundary

# 4.1 User Context

## 책임

* 회원 관리
* 로그인/인증
* 사용자 프로필 관리
* 사용자 권한 관리

## 소유 데이터

* User
* Credential
* UserProfile
* Role

## 외부 공개 기능

* 인증 토큰 발급
* 사용자 정보 조회

## 금지 책임

* 예약 상태 관리 금지
* 결제 상태 관리 금지

---

# 4.2 Movie Context

## 책임

* 영화 메타데이터 관리
* 영화 검색
* 영화 정보 제공

## 소유 데이터

* Movie
* Genre
* Actor
* Rating

## 특징

조회 중심 서비스

## 금지 책임

* 좌석 상태 관리 금지
* 상영 일정 관리 금지

---

# 4.3 Screening Context

## 책임

* 상영 일정 관리
* 상영관 매핑
* 영화-극장-시간 연결

## 소유 데이터

* Screening
* Theater
* Auditorium
* Showtime

## 특징

상영 단위 Aggregate 관리

## 금지 책임

* 좌석 예약 상태 관리 금지

---

# 4.4 Seat Inventory Context

## 책임

* 좌석 상태 관리
* 좌석 Hold 처리
* 좌석 점유 만료
* 좌석 가용성 관리

## 소유 데이터

* SeatInventory
* SeatHold
* SeatAllocation

## 핵심 역할

동시성 제어 핵심 서비스

## 특징

* Redis 활용 가능
* TTL 기반 Hold 처리
* Atomic Allocation 필요

## 금지 책임

* 결제 처리 금지
* 예약 완료 관리 금지

---

# 4.5 Reservation Context

## 책임

* 예약 생성
* 예약 상태 관리
* 예약 취소
* 예약 완료 처리

## 소유 데이터

* Reservation
* ReservationItem
* ReservationStatus

## 상태 흐름

```text
PENDING
 -> PAYMENT_WAITING
 -> CONFIRMED
 -> CANCELED
 -> EXPIRED
```

## 금지 책임

* PG 결제 직접 처리 금지

---

# 4.6 Payment Context

## 책임

* 결제 요청
* PG 연동
* 결제 승인/실패 처리
* 환불 처리

## 소유 데이터

* Payment
* PaymentTransaction
* Refund

## 특징

외부 시스템 의존성 격리

## 금지 책임

* 좌석 상태 관리 금지

---

# 4.7 Notification Context

## 책임

* 알림 발송
* 이메일/SMS/푸시 관리

## 소유 데이터

* Notification
* NotificationTemplate
* DeliveryHistory

## 특징

완전 비동기 서비스

## 금지 책임

* 예약 상태 변경 금지

---

# 4.8 Admin Context

## 책임

* 영화 등록
* 상영 일정 관리
* 운영 모니터링

## 특징

운영 도메인 분리

---

# 5. Service Responsibility Matrix

| Domain       | Owns State    | Publishes Event      | Consumes Event       |
| ------------ | ------------- | -------------------- | -------------------- |
| User         | User          | UserRegistered       | ReservationCreated   |
| Movie        | Movie         | MovieCreated         | -                    |
| Screening    | Screening     | ScreeningOpened      | MovieCreated         |
| Seat         | SeatInventory | SeatHeld             | ReservationRequested |
| Reservation  | Reservation   | ReservationConfirmed | PaymentApproved      |
| Payment      | Payment       | PaymentApproved      | ReservationCreated   |
| Notification | Notification  | NotificationSent     | ReservationConfirmed |

---

# 6. Event-Driven Architecture

# 핵심 이벤트 흐름

```text
ReservationRequested
    ↓
SeatHoldRequested
    ↓
SeatHeld
    ↓
PaymentRequested
    ↓
PaymentApproved
    ↓
ReservationConfirmed
    ↓
NotificationSent
```

---

# 7. Saga-Based Reservation Flow

# 예약 Saga 흐름

```text
1. Reservation Requested
2. Seat Hold Requested
3. Seat Held
4. Payment Requested
5. Payment Approved
6. Reservation Confirmed
7. Notification Triggered
```

---

# 결제 실패 보상 흐름

```text
Payment Failed
    ↓
Seat Hold Released
    ↓
Reservation Canceled
```

---

# 8. Database per Service Strategy

| Service              | Database           |
| -------------------- | ------------------ |
| User Service         | PostgreSQL         |
| Movie Service        | PostgreSQL         |
| Screening Service    | PostgreSQL         |
| Seat Service         | Redis + PostgreSQL |
| Reservation Service  | PostgreSQL         |
| Payment Service      | PostgreSQL         |
| Notification Service | MongoDB            |
| Search Query Model   | Elasticsearch      |

---

# 9. Communication Strategy

# 9.1 Synchronous Communication

## 사용 목적

즉각 응답 필요 API

## 사용 예시

* 인증
* 사용자 조회
* 영화 조회

## 기술

* REST/gRPC

---

# 9.2 Asynchronous Communication

## 사용 목적

상태 전파 및 Saga

## 사용 예시

* 예약 생성
* 결제 완료
* 좌석 반환
* 알림 발송

## 기술

* Kafka

---

# 10. Seat Reservation Concurrency Design

# 핵심 전략

## 1. Seat Hold 기반 예약

좌석 즉시 확정 금지.

먼저:

* HOLD 상태 생성
* TTL 부여
* 결제 완료 후 CONFIRMED

---

## 2. Redis Atomic Operation

```text
SETNX seat:{screeningId}:{seatNo}
```

기반 점유 처리.

---

## 3. TTL 기반 자동 해제

```text
Hold Expiration = 5 minutes
```

---

## 4. Idempotency Key

중복 결제 및 중복 예약 방지.

---

# 11. CQRS Design

# Command Side

* Reservation Create
* Payment Request
* Seat Hold

# Query Side

* Movie Search
* Seat View
* Reservation History

---

# Read Model

```text
ReservationQueryModel
MovieSearchModel
SeatAvailabilityView
```

---

# 12. Cache Strategy

| Target             | Cache |
| ------------------ | ----- |
| Movie List         | Redis |
| Popular Movies     | Redis |
| Seat Availability  | Redis |
| Screening Schedule | Redis |

---

# 13. Failure Recovery Strategy

# 핵심 전략

## Retry

* Exponential Backoff

## DLQ

* Failed Event Isolation

## Circuit Breaker

* PG 장애 격리

## Timeout

* 동기 호출 보호

## Outbox Pattern

* 이벤트 유실 방지

---

# 14. Observability Design

| Area      | Tool         |
| --------- | ------------ |
| Metrics   | Prometheus   |
| Dashboard | Grafana      |
| Logging   | ELK          |
| Tracing   | Jaeger       |
| Alert     | AlertManager |

---

# 15. Deployment Architecture

```text
Kubernetes Cluster
    ├── API Gateway
    ├── User Service
    ├── Movie Service
    ├── Screening Service
    ├── Seat Service
    ├── Reservation Service
    ├── Payment Service
    ├── Notification Service
    ├── Kafka Cluster
    ├── Redis Cluster
    └── Monitoring Stack
```

---

# 16. Scalability Strategy

| Service      | Scaling Strategy |
| ------------ | ---------------- |
| Movie        | Read Replica     |
| Seat         | Redis Cluster    |
| Reservation  | Horizontal Scale |
| Payment      | Partitioning     |
| Notification | Consumer Scaling |

---

# 17. Security Design

| Area               | Strategy      |
| ------------------ | ------------- |
| Authentication     | OAuth2/JWT    |
| Authorization      | RBAC          |
| API Security       | Rate Limiting |
| Data Encryption    | AES-256       |
| Transport Security | TLS           |

---

# 18. Trade-offs

| Decision     | Benefit        | Cost          |
| ------------ | -------------- | ------------- |
| Event-Driven | Loose Coupling | Complexity 증가 |
| CQRS         | Read 성능 향상     | 데이터 동기화 필요    |
| Saga         | 분산 트랜잭션 해결     | 보상 로직 복잡      |
| Redis Hold   | 빠른 좌석 처리       | 캐시 정합성 고려     |

---

# 19. Final Architecture Summary

본 시스템은:

* DDD 기반 서비스 경계 분리
* Event-Driven MSA
* Saga 기반 분산 트랜잭션
* Redis 기반 좌석 동시성 처리
* CQRS 기반 조회 최적화
* Kafka 기반 이벤트 스트리밍
* Kubernetes 기반 운영

을 핵심 아키텍처 원칙으로 한다.

특히:

* Seat Inventory
* Reservation
* Payment

를 명확히 분리함으로써:

* 장애 격리
* 독립 확장
* 트랜잭션 안정성
* 고동시성 대응

이 가능한 구조를 제공한다.
