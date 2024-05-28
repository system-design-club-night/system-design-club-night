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
1. 스케쥴러 잡 등록 API  
2. 회원이 등록한 잡 목록 조회 API 

## Consumer
* 스케쥴러에서 발행된 카프카 이벤트를 구독하는 컨슈머 

## Batch
* 1분마다 실행되는 스케쥴러  

## Reference
* [Cassandra vs. MongoDB](https://aws.amazon.com/ko/compare/the-difference-between-cassandra-and-mongodb/)

## 이어서 
### 함께 논의하고 싶은 주제 
- 

### 참고하기 좋은 기술 사례
* 
