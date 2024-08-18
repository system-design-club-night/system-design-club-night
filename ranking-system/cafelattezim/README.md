## Functional Requirements
* 랭킹 페이지에서 사용되는 집계 데이터를 적재해야한다.
   * <img width=400 src="올리브영_랭킹.png">
* 집계 데이터는 성별 x 연령 x 기간 조건으로 필터링할 수 있어야한다. 
  * 기간의 종류는 다음과 같다.
    * 실시간
    * 일간
    * 주간 
    * 월간
* 주문 후 취소/교환/반품을 반복하는 비정상적인 주문 방식이 랭킹에 집계되면 안된다.

## Non-Functional Requirements
* 주문 데이터가 급증하는 세일기간에도 처리 속도가 지연되면 안된다.   
   * AWS Glue 병렬화 수준 조정을 통해 해결할 수 있다. 
* 비즈니스 요구사항 변화로 집계 로직을 쉽게 변경할 수 있어야한다.
 
## Estimates
-- 

## 설계 
### 데이터구조 
> 뇌피셜로 상상해본 데이터 구조

<img width=500 src="https://github.com/user-attachments/assets/06ecb7bf-b65a-4aae-8a6b-ac2b246d69f0">

### 파이프라인 
<img width=400 src="https://github.com/user-attachments/assets/2d705c75-b33f-4b05-a598-2db6eae0e0c9">

### 특이사항
* 데이터 파이프라인은 특정 주기에 따라 실행된다. (Amazon EventBridge)
  * 스트리밍 서비스를 활용한 실시간 주문 이벤트 집계는 다음 요건에 따라 제외되었다. 
    * 비정상적인 주문 집계 제외 
    * 집계 로직의 유연성
    

## 개념사전
### AWS Solution 
#### [Amazon EventBridge 스케줄러](https://aws.amazon.com/ko/eventbridge/scheduler/)
<img width=400 src="https://github.com/user-attachments/assets/9053b935-f6e9-4b4f-9012-7ff702fc1677">

#### Glue
#### Glue Catalog
#### Athena S3 
* Event Notification의 종류 
##### Lambda

### 파일 
##### [Apache Parquet (파케이)](https://www.databricks.com/kr/glossary/what-is-parquet)
* 컬럼 중심의 데이터 형식 파일
* 데이터 압축률이 좋음 
* 집계 쿼리를 수행할 때 효율적임 
  * CSV 같은 행 기반 파일에 비해 효율적임
  * 개연성 없는 데이터는 질의에서 건너뛸 수 있음 
* Spark에서 사용하기 편함 ([Spark SQL Guide](https://spark-korea.github.io/docs/sql-data-sources-parquet.html))

## 이어서 
### 함께 논의하고 싶은 주제
* 스케쥴러로 적합한 솔루션은?
  * Amazon EventBridge 스케줄러
  * 젠킨스
  * Batch App
  * Kubernetes CronJob
* Glue를 활용한 데이터 파이프라인 모니터링
  * Amazon EventBridge -> Amazon SNS 이벤트 발행 -> CloudWatch에서 구독 + 임계치 이상 발생 시 -> 슬랙 알럿 

### 참고하기 좋은 기술 사례
* [CJ 올리브영의 서버리스 랭킹 시스템 구축기](https://aws.amazon.com/ko/blogs/tech/oliveyoung-serverless-ranking-system/) 

