## Requirements
  ### Functional
  #### 포인트 서비스
  * 회원의 포인트를 예약한 시각에 이체해야한다.
  * 포인트 이체 예약 일정을 삭제할 수 있다. 
  #### 스케쥴러 서비스
  * 예약 실행 작업을 등록/삭제할 수 있다.
    * 실패 재시도 횟수를 정할 수 있다. 
  * 스케쥴러는 여러 유형의 비즈니스 요구사항을 지원해야한다.  
      * 이메일, 푸시 발송, 쿠폰 발송 등등     
  * JOB은 1회성 실행 혹은 반복 실행으로 선택할 수 있다.   
      * 반복 실행 주기는 사용자가 설정할 수 있다.
  * 최소 예약 설정 단위는 분단위이다.    
  ### Non-Functional
  #### 스케쥴러 서비스
  * 트랜잭션의 수가 늘어나도 잡 실행 시각이 지연되면 안된다.  
  * 잡 실행이 지연되는 특수한 이벤트가 일반 이벤트 처리에 영향을 미치면 안된다.
  ### Estimates
  #### 스케쥴러 서비스
  * 일일 평균 예상 작업 실행 건수: 5000만건 
  * 분당 평균 예상 처리량: 34722건 (daily tasks)/(24*60)
   
## Design
> 다음 설계는 다양한 사례를 지원하는 다목적 아키텍처다. 

<img src="최종.png"> 

## API
### 포인트 이체 도메인
1. 포인트 이체 예약 등록 API
2. 포인트 이체 예약 목록 조회 API  
3. 포인트 이체 예약 삭제 API 

### JOB 도메인
##### 1. JOB 등록 API  
 * 반복실행 JOB인 경우,  
##### 2. JOB 상태 변경 API
 * `task_execution_history` 테이블의 상태를 변경한다. 
 * 만약 `FAILED`이고, `max_retry_count`가 남아있는 경우, retry topic에 kafka 이벤트를 발행한다.
        
##### 3. JOB 삭제 API

## Consumer
* 스케쥴러에서 발행된 카프카 이벤트를 구독하는 컨슈머이다. 
* 원본 토픽 컨슈머와 retry 토픽 컨슈머를 분리하여 구독한다.
    * retry 토픽 구독 예시
        * `point-transfer-retry-*` 
            * `point-transfer-retry-0`, `point-transfer-retry-1`, `point-transfer-retry-2`
    * 문제가 되는 이벤트가 일반 이벤트 구독을 방해하는 것을 방지하고자, retry consumer를 따로 관리한다.  
* 처리에 완료하면, JOB 상태 변경 API를 호출한다.  
     
## Batch
* Batch은 `Master` / `Worker`으로 운영된다. 
    * `Master`
        * `Worker`의 health check를 담당한다.
        * `Worker`에 segment를 부여한다.
            * segment는 worker의 수보다 적어야 한다.      
    * `Worker` 
        * `Worker`를 여러대로 도입해야하는 배경 
          1. 이벤트 발행 처리량을 높이려면 `Worker`를 여러대로 운영해야 한다.
            * 분당 처리량 / Worker 개수가 분당 처리량이 된다.
             * 예시:
                 * 평균 3만건이 발행된다고 가정했을 때: 3만건 / 10대 = 1분당 3000건 처리 
                 * 피크 타임인 경우, 10만건이 발행된다고 가정했을 때: 10만건 / 10대 = 1분당 1만건 처리
          2. 배치앱을 1개로 운영하는 것은 SPOF (Single Point Of Failure) 리스크가 있다. 
        * 각 `Worker`은 segment를 부여받는다. 
        * 1분마다 예약된 태스크를 조회하여, job에 지정된 topic에 kafka message를 produce한다.  
        ```sql
        select * from task_schedule where next_execution_time = 1717420260 ans segment = (1,2,3)
        ```
        * job에 등록한 topic에 event를 produce한다.
        
## Schema
### 스케쥴러 서비스
#### job 
JOB에 대한 메타데이터 관리

|name|column|example|partition-key|sort-key|
|---|---|---|---|---|
|회원아이디|user_id|"3254723"|O||
|작업식별자|id|21BX5ZZKBKACTAV9WEVGEMMVS1||O|
|이벤트명|topic_name|point_transfer|||
|반복실행여부|is_recurring|false|||
|반복실행주기|interval|PT24H|||
|최대재시도횟수|max_retry_count|3|||
|생성시각|created_at|2024-06-03T00:00:00+09:00|||

#### task_schedule 
|name|column|example|partition-key|sort-key|
|---|---|---|---|---|
|다음실행시각|next_execution_time|1717420260|O||
|작업식별자|job_id|21BX5ZZKBKACTAV9WEVGEMMVS1||O|
|세그먼트|segment|1|||

##### 기타
* PT24H은 PeriodTime24Hours 축약어이다.
* next_execution_time은 분단위로 절삭한 후, unix timestatmp 형태로 변환하여 저장한다.
 ```python
    import datetime
    now = (datetime.datetime.now() + datetime.timedelta(hours=1)).replace(second=0, microsecond=0) 
    int(now.timestamp())
```
* segment은 크론잡 워커에서 자신이 속한 세그먼트 잡을 식별할 때 사용된다.
   * segment를 정하지 않으면, batch worker에서 이벤트를 발행할 때, `race condition`이 발생할 수 있다.
* `next_execution_time`을 기준으로 파티셔닝을 해야 모든 파티션을 스캔하는 문제를 피할 수 있다. 
  
#### task_execution_history 
|name|column|example|partition-key|sort-key|
|---|---|---|---|---|
|작업식별자|job_id|21BX5ZZKBKACTAV9WEVGEMMVS1||O|
|실행시각|execution_at|2024-06-03T00:00:00+09:00||O|
|상태|status|- SCHEDULED<br>- COMPLETED<br>- FAILED<br>- EXECUTED|||
|재시도횟수|retry_count|0|||
|수정시각|updated_at|2024-06-03T00:00:00+09:00|O||

### 포인트 서비스
#### point_transfer_transaction
|name|column|example|partition-key|sort-key|
|---|---|---|---|---|
|잡아이디|job_id|21BX5ZZKBKACTAV9WEVGEMMVS1||O|
|보낸회원|from_member_id|"3254723"|O||
|받는회원|to_member_id|"5345436"|||
|포인트|point|10000|||

## Event Payload
```
{
  "jobId"; "21BX5ZZKBKACTAV9WEVGEMMVS1"
}
```
## Reference
* [Design a Distributed Job Scheduler for Millions of Tasks in Daily Operations](https://medium.com/@mayilb77/design-a-distributed-job-scheduler-for-millions-of-tasks-in-daily-operations-4132dc6d645f)
* [Cassandra vs. MongoDB](https://aws.amazon.com/ko/compare/the-difference-between-cassandra-and-mongodb/)

## 이어서 
### 함께 논의하고 싶은 주제 
- 분산 스케쥴러가 필요한 실무 사례 
  - 광고: 광고주 광고 예산 자동 충전
  - 결제: 구독모델 자동 결제 
  - 마케팅: 쿠폰 예약 발급 및 푸시 발송
  - 콘텐츠: 블로그송 포스 예약 발행 및 푸시 발송 
- 재시도 잡보다 첫시도 잡 실행이 우선적으로 처리될 수 있게 하는 방법  
  - retry용 kafka topic 분리 
- dlt topic의 발행 주체 
  - topic을 여러 도메인 consumer가 구독하는 경우, dlt topic을 발행하는게 의미있을지 
 
