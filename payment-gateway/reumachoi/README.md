# Requirements

## Functional

- pg 도입을 통해 결제 연동
- 구매자로부터 대금을 수신 처리
- 판매자에게 대금을 정산 처리

## Non-Functional

- 내부-외부(pg) 시스템간 정산 일치여부 확인
- 결제 실패 처리 멱등성 보장

## Estimates

- 사용자 600만, 1달 내 인당 평균 5번 구매(100만번/일)
  -> 약 12TPS

## Design

[![](https://mermaid.ink/img/pako:eNqdVltrGkEU_ivDPiUQCX3pwz7kJZE0pG2kBkpAKMPuaJbuxe4lIMGHBi0lsTSlpjFGxRZpCbWgRiEt-UWZs_-hs7d4WVfFfXHG-c53vnPmnLN7zAmaSDieM8g7i6gC2ZJwRsdKSkXsyWLdlAQpi1UTbcoSYT_YQPb7Pq3fQ70P3-7s07swNIFzCsMmZGymNV1JEv2I6I7lQ68DzRqyyyXa-G1_vkdQqNFeIczwioiS4VjQ5kdaLsNpC8G_czirhaG7OP0WO1BvwWTRXmumLF0TiGFIaiYkDHoV-rMdqSqxnZRM4uAT23DSXtnXDCNgNVbD-NdYlok59OLtI-mfEzFD9CHc2z_CPQPvGmIbG1OTzD9GUi1D7w9aSewl93l0rORETcGSml_P-nKDxfrRE1-5qrHYdClzaCIt7fvh0TOCRYd3RyRKliFUIRfbJTnGubMVf5HY24-_3Dx4sxs_yHssU2Uxue6N8ojedOnXLhS6yD5pIfvqAur-TWHZnDiF731otL1T53EpYvNDL3QfbgfO7uH2HtFOxb6qDEmm62OkQcDjLFD_Qs8GnjmRDTIp8fID1EuLS5xmE5Uxt57DF6proiUQfXVI4AJHfE7Ud4hiU1MNSxmjiDB1SN2a572KH2caMXdBsVkaxu3drK7A9V_7uriQjKnJ8K94kZzM7JKAZ0ZiIssFBjX4UUQsOKjeQON8vGJUMejbuaEFw-GySD912BRjIywUWRDV6GCJsJyIZa77YNgs4n50UEVYjrj3bGVNyyL6qwV1NvUrLWgW0NPHeb5AG0DzAk46bCZ07GppuTaYoFiqDXwOP9JCZ5lWCHOMlMxClzUeyfJtMJ1nIjFuEXNrHPuHvUJE9q1w7BykOPOQKCTF8WwpkjS2ZDPFpdQ8g2LL1JI5VeB4U7fIGmdlRWwGnxYcn8Zskub_AxCfjZw?type=png)](https://mermaid.live/edit#pako:eNqdVltrGkEU_ivDPiUQCX3pwz7kJZE0pG2kBkpAKMPuaJbuxe4lIMGHBi0lsTSlpjFGxRZpCbWgRiEt-UWZs_-hs7d4WVfFfXHG-c53vnPmnLN7zAmaSDieM8g7i6gC2ZJwRsdKSkXsyWLdlAQpi1UTbcoSYT_YQPb7Pq3fQ70P3-7s07swNIFzCsMmZGymNV1JEv2I6I7lQ68DzRqyyyXa-G1_vkdQqNFeIczwioiS4VjQ5kdaLsNpC8G_czirhaG7OP0WO1BvwWTRXmumLF0TiGFIaiYkDHoV-rMdqSqxnZRM4uAT23DSXtnXDCNgNVbD-NdYlok59OLtI-mfEzFD9CHc2z_CPQPvGmIbG1OTzD9GUi1D7w9aSewl93l0rORETcGSml_P-nKDxfrRE1-5qrHYdClzaCIt7fvh0TOCRYd3RyRKliFUIRfbJTnGubMVf5HY24-_3Dx4sxs_yHssU2Uxue6N8ojedOnXLhS6yD5pIfvqAur-TWHZnDiF731otL1T53EpYvNDL3QfbgfO7uH2HtFOxb6qDEmm62OkQcDjLFD_Qs8GnjmRDTIp8fID1EuLS5xmE5Uxt57DF6proiUQfXVI4AJHfE7Ud4hiU1MNSxmjiDB1SN2a572KH2caMXdBsVkaxu3drK7A9V_7uriQjKnJ8K94kZzM7JKAZ0ZiIssFBjX4UUQsOKjeQON8vGJUMejbuaEFw-GySD912BRjIywUWRDV6GCJsJyIZa77YNgs4n50UEVYjrj3bGVNyyL6qwV1NvUrLWgW0NPHeb5AG0DzAk46bCZ07GppuTaYoFiqDXwOP9JCZ5lWCHOMlMxClzUeyfJtMJ1nIjFuEXNrHPuHvUJE9q1w7BykOPOQKCTF8WwpkjS2ZDPFpdQ8g2LL1JI5VeB4U7fIGmdlRWwGnxYcn8Zskub_AxCfjZw)

### 정책

**Cache**

- Data Types: Sets
- TTL: 24H

**PG 외부연동 정책**

1. 재시도 정책
   - 기본 대기 시간: 결제 요청에 대한 API 응답을 기본 3초간 대기
   - timeout 처리:
     - 재시도 횟수: 최대 5회까지 재시도를 수행
     - 재시도 지연: 각 재시도 사이에 1초의 delay 처리
     - Kafka를 통해 재시도 관리를 하며, 재시도 로직은 Kafka의 retries 및 delay 설정 사용
   - HTTP 상태 코드 처리:
     - 4xx 및 5xx 응답을 받은 경우, 해당 메시지는 Dead Letter Queue(DLQ)로 후처리
2. 결과 응답 처리
   - Webhook을 통한 결과 처리:
     - PG사로부터의 결제 처리 결과는 Webhook을 통해 비동기 수신
     - Webhook 결과에 따라 클라이언트에 최종 응답을 제공
   - Dead Letter Queue(DLQ) 메시지 처리:
     - DLQ로 이동된 메시지에 대해서도 클라이언트에 응답을 제공
     - DLQ 메시지 처리 자동화 처리나 기록 필요

**정산**

- 트래픽 적은 시간대인 오전 6시에 수행하여 서비스 내 기록과 pg사 정산기록을 비교 및 업데이트 진행

## API

### Payment Platform Server

1. 결제 요청
   - POST: {mydomain}/payments/payment/v1
     - Header: `Idempotency-Key: {IDEMPOTENCY_KEY}`
2. 결제 조회
   - GET: {mydomain}/payments/payment/v1/{paymentId}
3. 결제 취소
   - DELETE: {mydomain}/payments/payment/v1/{paymentId}

### Payment Processing Server

1. pg 웹훅
   - POST: {mydomain}/payments/webhook

## Plus

### 보안

- AWS WAF를 통해 애플리케이션 보호
- IAM, KMS를 통해 RDS, MSK 등 접근 시 인증/인가 및 암호화 진행

### 함께 논의하고 싶은 주제

- 오픈뱅킹 점검시간에 결제는 어떻게 진행할까?
  - 웹 내 차단&API 거부 방식인지?
  - PG API 에러코드를 확인해서 응답하는 방식인지?
    - toss 에러코드([참조 링크](https://docs.tosspayments.com/reference/error-codes#결제-승인)): `NOT_AVAILABLE_BANK`

## Reference

[결제시스템 인프라 아키텍쳐 다이어그램-AWS](https://docs.aws.amazon.com/architecture-diagrams/latest/payment-system-interface-modernization/payment-system-interface-modernization.html)  
[결제시스템에 대한 설명이 좋음](http://austincorso.com/2020/03/10/payments-system.html)

토스 페이먼츠  
https://docs.tosspayments.com/guides/learn/payment-flow  
https://docs.tosspayments.com/reference/using-api/authorization#api-멱등성이란  
https://docs.tosspayments.com/reference/using-api/api-keys  
https://docs.tosspayments.com/guides/webhook
