## Requirements
  ### Functional
  * 예약한 시각에 회원의 포인트가 이체된다.
  * 분단위로 예약시각을 설정할 수 있다.   
  ### Non-Functional
  * 트랜잭션의 수가 늘어나도 잡 실행 시각이 지연되면 안된다. 
  * 잡 실행에 실패한 경우, 최대 N번까지 재시도를 해야한다. 
  * 잡 실행이 지연되는 특수한 이벤트가 일반 이벤트 처리에 영향을 미치면 안된다.  
  ### Estimates
  * 
  
## Design
* 

## API
1. 포인트 이체 예약 등록 API  
2. 포인트 이체 예약 목록 조회 API 

## Consumer
* 스케쥴러에서 발행된 카프카 이벤트를 구독하는 컨슈머 

## Batch
* 1분마다 실행되는 스케쥴러  

## Reference
* [Design a Distributed Job Scheduler for Millions of Tasks in Daily Operations](https://medium.com/@mayilb77/design-a-distributed-job-scheduler-for-millions-of-tasks-in-daily-operations-4132dc6d645f)
* [Cassandra vs. MongoDB](https://aws.amazon.com/ko/compare/the-difference-between-cassandra-and-mongodb/)

## 이어서 
### 함께 논의하고 싶은 주제 
- 분산 스케쥴러가 필요한 실무 사례 
- 재시도 잡보다 첫시도 잡 실행이 우선적으로 처리될 수 있게 하는 방법  
 - retry용 kafka topic을 분리하기 
 
### 참고하기 좋은 기술 사례
* 
