## Requirements
  ### Functional
  * 신규 사기 유형에 유연하게 대응할 수 있어야한다.
    * 참고: [FDS 시스템: Rule 기반 vs. AI 기반](https://tech.kakaobank.com/posts/2310-applying-ai-into-fds-system/)
  * 오탐률과 미탐률을 개선할 수 있어야한다.   
  ### Non-Functional
  * ML 서빙은 200ms 미만 응답을 보장해야한다.
  * 분산 환경을 고려해야한다. 
  * JVM 기반 ML 추론
    * Java 언어를 사용하고 있으므로 JVM 기반 ML 추론이 가능한 라이브러리를 선정해야한다.
      * JVM 기반으로 추론을 하기 위해 아마존이 개발한 `DJL(Deep Java Library)` 을 사용한다. 
      * DJL이 여러 라이브러리와 비교해서 성능이 더 뛰어나다. 
      * 실제 ML 성능 테스트 결과에서도 10ms 정도의 응답 속도를 보여줬다.  
  * 분산 환경 고려: Kubernetes 및 AWS 환경을 고려하여 ML 서빙 시스템 설계
  
  ### Estimates
  * 🤫
  
## Design
<img width=900 src="/fraud-detection-system/cafelattezim/FEATURE_STORE_데이터_파이프라인.png">

## 개념정리
### FDS와 AML의 차이 
|-|  FDS (Fraud Detection System) |  AML (Anti-Money Laundering) |
|---|---|---|
|요약|이상거래탐지|자금세탁방지|
|목적|금융 거래에서 발생하는 즉각적인 사기 탐지 및 방지| 자금 세탁 활동을 식별하고 방지하며, 불법적인 금융 활동 차단  ||
|범위|준실시간으로 탐지 |보통 더 긴 시간 범위에 걸쳐 패턴을 분석하며, 일부 실시간 모니터링도 포함| 
|대상|주로 개별 거래나 짧은 기간 동안의 거래 패턴을 분석|장기적인 거래 패턴, 고객 프로필, 외부 데이터 소스 등을 종합적으로 분석|
|사례|신용카드 사기, 계정 탈취, 신원 도용 등 즉각적인 금융 사기|복잡한 자금 세탁 구조, 탈세 등 더 복잡하고 은밀한 금융 범죄| 
|대응|의심스러운 거래를 즉시 차단하거나 추가 인증을 요구|의심스러운 활동을 모니터링하고 보고하며, 필요시 관계 당국에 신고|

###  gridgain
* GridGain은 분산 컴퓨팅 및 인메모리 컴퓨팅 플랫폼

### feature store
* 머신러닝 모델에 사용되는 특징(feature)들을 저장, 관리, 제공하는 중앙 집중식 플랫폼
* 참고: 
  * [ML의 Feature Store이란?](https://zzsza.github.io/mlops/2020/02/02/feature-store/) 
### Apache Flink
* FDS 사례 샘플 코드: [Apache Flink Docs: Fraud Detection with the DataStream API](https://nightlies.apache.org/flink/flink-docs-release-1.20/docs/try-flink/datastream/)
* [Flink 외 스트리밍 플랫폼 비교(https://carpe08.tistory.com/360)
* [Flink와 SQL](https://nightlies.apache.org/flink/flink-docs-release-1.20/docs/dev/table/sql/overview/)
### DJL: Deep Java Library
* 참고: [DJL: Deep Java Library](https://docs.djl.ai/master/index.html)

## 참고
* [카카오뱅크: 킁킁!킁! 어디서 사기 냄새 안나요? : FDS 시스템에 AI 적용하기](https://tech.kakaobank.com/posts/2310-applying-ai-into-fds-system/)
* [토스: 토스 ksqlDB를 활용한 증권사의 실시간 데이터 처리하기](https://toss.tech/article/ksqldb-realtime-data)
