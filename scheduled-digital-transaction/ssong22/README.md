## Requirements
### Functional
* 예정된 시간에 요금 지불(결제)가 이루어져야 한다.
* 사용자는 시간을 변경할 수 있다.

### Non-Functional
* 데이터 정합성
* 사용자가 설정한 시간에 예약된 결제가 이뤄야하며, 중단되거나 늦춰져선 안된다.

### Estimates

## Design
<img src="scheduled-digital-transaction.png">

## API
1. 예정된 결제 생성 POST v1/scheduled-pay
   ```
   {
    "amount": string,
    "when": string,
    "to": string,
    "from": string 
   }
   ```
2. 예정된 결제 수정 PUT v1/scheduled-pay
    ```
   {
    "amount": string,
    "when": string,
    "to": string,
    "from": string 
   }
   ```
3. 관리자용 API, 수동 결제 POST v1/bo/scheduled-pay

## Reference
* [Design a Distributed Job Scheduler for Millions of Tasks in Daily Operations](https://medium.com/@mayilb77/design-a-distributed-job-scheduler-for-millions-of-tasks-in-daily-operations-4132dc6d645f)
* [System Design Pattern for Recurring Payments](https://www.geeksforgeeks.org/system-design-pattern-for-recurring-payments/#key-components-of-a-recurring-payment-system)

### 함께 논의하고 싶은 주제
* 예정된 결제 실패(retry 횟수 초과의 경우)에 대한 failover 처리도 중요하고 복잡한 서비스 로직이 필요할 거 같음
  * 사용자의 의도로 실패했는지(카드 만료/잔액 부족)
  * 서버 장애로 인해 실패했을 때, 어떤 보상을 줘야할지 / 관리자가 직접 실행하는 등의 '자동화'하기 어려운 부분이 존재할 거 같다
    * 서버 장애시, 담당자가 수동으로 결제를 실행하는 api를 호출

### 참고하기 좋은 기술 사례
