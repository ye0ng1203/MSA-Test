# 온라인 영화 예매 시스템 — 비기능적 요구사항 요약

# 1. 성능(Performance) 요구사항

* 일반 조회 API 평균 응답 속도 300ms 이하
* 좌석 조회 API 평균 응답 속도 500ms 이하
* 초당 대량 예약 요청 처리 가능해야 함
* 이벤트 처리 지연 최소화 필요
* 실시간 좌석 상태 반영 지원

---

# 2. 확장성(Scalability) 요구사항

* 서비스별 독립적인 수평 확장(Scale-Out) 지원
* 인기 영화 예매 오픈 시 Burst Traffic 대응 가능해야 함
* 캐시 기반 조회 성능 확장 지원
* MSA 기반 독립 배포 및 확장 가능해야 함

---

# 3. 가용성(Availability) 요구사항

* 고가용성(HA) 구조 지원
* 특정 서비스 장애가 전체 시스템 장애로 확산되면 안 됨
* 자동 장애 복구(Auto Recovery) 가능해야 함
* 무중단 배포 가능해야 함

---

# 4. 신뢰성(Reliability) 요구사항

* 좌석 중복 예약 방지 필수
* 데이터 정합성 유지 필요
* 메시지 중복 소비(Idempotency) 처리 지원
* 이벤트 유실 방지 필요
* Eventually Consistent 구조 허용
* 장애 발생 시 보상 처리(Saga) 가능해야 함

---

# 5. 보안(Security) 요구사항

* OAuth2/JWT 기반 인증 및 인가 지원
* 개인정보 암호화 저장 필요
* API Rate Limiting 적용 가능해야 함
* 감사 로그(Audit Log) 저장 필요
* 관리자 기능 접근 제어 필요

---

# 6. 유지보수성(Maintainability) 요구사항

* 서비스별 독립 개발 및 배포 가능해야 함
* 서비스 간 강결합 최소화
* API Versioning 지원
* 코드 구조 표준화 필요
* 도메인 중심 구조 유지 필요

---

# 7. 관측성(Observability) 요구사항

* 중앙 로그 수집 지원
* Distributed Tracing 지원
* 실시간 메트릭 수집 가능해야 함
* 장애 탐지 및 모니터링 가능해야 함
* 이벤트 흐름 추적 가능해야 함

---

# 8. 데이터(Data) 요구사항

* Database per Service 원칙 적용
* Shared Database 금지
* CQRS 기반 읽기 모델 분리 가능해야 함
* 캐시 전략 적용 가능해야 함
* 대규모 조회 성능 최적화 필요

---

# 9. 메시징(Messaging) 요구사항

* Event-Driven Architecture 지원
* Kafka/RabbitMQ 기반 비동기 메시징 지원
* DLQ(Dead Letter Queue) 지원
* Retry 및 재처리 전략 필요
* 이벤트 순서 및 중복 처리 고려 필요

---

# 10. 운영(Operation) 요구사항

* Kubernetes 기반 운영 가능해야 함
* CI/CD 기반 자동 배포 지원
* 서비스별 독립 롤백 가능해야 함
* 운영 중 서비스 확장 가능해야 함
* Blue-Green 또는 Canary 배포 가능해야 함

---

# 11. 핵심 아키텍처 요구사항

* MSA 기반 서비스 분리
* Event-Driven Architecture 기반 설계
* Saga 기반 분산 트랜잭션 처리
* 장애 격리(Fault Isolation)
* 비동기 이벤트 중심 서비스 협력
* 읽기/쓰기 모델 분리 가능성 고려
* 고트래픽 대응 가능한 구조 필요

---

# 12. 핵심 기술적 난제

## 좌석 동시성 문제

* 동일 좌석 동시 예약 경쟁 상태 처리 필요
* 실시간 Lock/Hold 전략 필요

## 분산 트랜잭션 문제

* 결제/예약/알림 간 정합성 유지 필요
* ACID 대신 Saga 기반 보상 처리 필요

## 이벤트 정합성 문제

* 메시지 순서 역전 가능성 고려
* 중복 이벤트 처리 필요
* Eventually Consistency 허용 범위 정의 필요
