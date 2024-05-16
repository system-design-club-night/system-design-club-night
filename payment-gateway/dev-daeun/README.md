## Functional Requirements
* 고객은 결제를 정상적으로 할 수 있어야 한다.
* 고객은 정해진 기한 내에 결제를 취소할 수 있어야 한다.
* 고객이 제품을 수령하면 결제 대금이 판매자에게 지급되어야 한다.

## Non-Functional Requirements
* FE 단에서 타임아웃 등의 사유로 요청을 재시도 하면 중복 처리가 되지 않아야 한다. (멱등성을 보장해야 한다.)
* 결제 과정 중 일부 결함이 발생해도 고객 입장에서는 결제 완료 처리가 되어야 하고, 내부적으로는 결함에 대해 후 보정이 가능해야 한다. (내결함성을 보장해야 한다.)
* 결제 대금과 연관된 데이터의 갱신/생성 과정에서 롤백이 발생하면 원자성을 보장해야 한다.


## Estimations
* 트랜잭션의 정확하고 안정적인 처리가 가장 중요하므로 트래픽을 크게 가정하진 않을 거 같은데.. 
* DAU = 100만으로 가정 (FYI, [쿠팡이츠 & 요기요 DAU = 100만, 배달의민족 DAU = 600만](https://www.mk.co.kr/news/business/10954092))
* 유저 당 평균 일일 주문 수 = 1건 
* TPS(Transaction Per Second) = 100만 / 60 * 60 * 24 ≈ 11

## APIs

### 결제 생성
* method: `POST`
* path : `/v1/payment`
* request body

    | key           | value         | description         |
    | -------------------- |---------------|---------------------|
    | card_info                                | json          | 카드 정보               |
    | checkout_id                              | string (uuid) | 결제 식별자 (멱등성 보장에 필요) |
    | payments                                 | list[payment] | 결제 정보 목록            |
    | payment                                  | json          | 결제 정보               |
    | payment.vendor_id                        | integer       | 판매자 식별자             |
    | payment.total_amount                     | string        | 결제 대금               |


### 결제 취소
* method: `DELETE`
* path : `/v1/payment/{checkout_id}`




## Design
![payment-gateway-design](./payment-gateway-design.png)


### states

#### 결제처리 서비스
* `created`: 결제 정보 생성
* `waiting_for_psp`: 결제대행 대기
* `psp_succeed`: 결제대행 성공
* `psp_failed`: 결제대행 실패
* `waiting_for_purchase_confirm`: 구매 확정 대기중
* `purchase_confirmed`: 구매 확정
* `completed`: 결제 종료


#### 대금지급 서비스
* `waiting_for_transfer`: 대금 지급 대기중
* `transfer_succeed`: 대금지급 완료
* `transfer_failed`: 대금지급 실패

### 원장 서비스
* 여긴 잘 모르겠습니다..

[질문]
1. `card_info` 에는 보통 뭐가 들어가나요? 
2. 결제 처리 API request body 위/변조는 FE에서만 방지하나요? 
