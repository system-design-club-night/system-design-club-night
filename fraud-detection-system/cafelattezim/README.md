> [카카오뱅크: 킁킁!킁! 어디서 사기 냄새 안나요? : FDS 시스템에 AI 적용하기](https://tech.kakaobank.com/posts/2310-applying-ai-into-fds-system/)을 읽고, FDS 설계를 고민해보자. 

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

#### 상세 구현 예시 
> 
1. 머신러닝 모델 만들기:
   - Python을 사용하여 모델을 만든다. (예: scikit-learn 또는 TensorFlow/Keras 사용).
   - 데이터를 준비하고, 모델을 훈련시킨다. 
   - 모델을 저장한다. (예: pickle 형식).

2. Spring 프로젝트 설정:
   - Spring Boot 프로젝트를 생성한다.
   - DJL 의존성을 추가합니다 (build.gradle).

3. 모델 로딩:
   - Spring 애플리케이션에서 DJL을 사용하여 저장된 모델을 로드한다.

4. 예측 서비스 구현:
   - 모델을 사용하여 예측을 수행하는 서비스 클래스를 만든다.

5. REST API 구현:
   - 예측 서비스를 호출하는 컨트롤러를 만든다.

##### 예시 코드

1. 의존성 추가 (build.gradle):

```gradle
dependencies {
    implementation 'ai.djl:api:0.20.0'
    implementation 'ai.djl.pytorch:pytorch-engine:0.20.0'
    implementation 'redis.clients:jedis:4.3.1'
}
```

2. 모델 로딩 및 예측 서비스:

```java
import ai.djl.inference.Predictor;
import ai.djl.ndarray.NDList;
import ai.djl.repository.zoo.Criteria;
import ai.djl.repository.zoo.ZooModel;
import org.springframework.stereotype.Service;
import redis.clients.jedis.Jedis;

@Service
public class FdsPredictionService {
    private final Jedis jedis;

    public PredictionService() {
        this.jedis = new Jedis("localhost", 6379); // Redis 연결
    }

    public FdsPredictionResponse predict(FdsPredictionRequest request) throws Exception {
        try {
            // Reids Cache에 저장된 Feature Store으로 모델 로딩
            String key = String.format("KYE:CSSTNO:%s:XFEPE_X", request.getUserId());
            String modelPath = jedis.get(key);
            
            Criteria<NDList, NDList> criteria = Criteria.builder()
                    .setTypes(NDList.class, NDList.class)
                    .optModelPath(modelPath)
                    .build();

            ZooModel<NDList, NDList> model = criteria.loadModel();
            this.predictor = model.newPredictor();
        } catch (Exception e) {
            throw new RuntimeException("모델 로딩 실패", e);
        }
        ..

        // request으로 input 데이터 생성 후, 예측 
        NDList prediction = predictor.predict(input);
        ...
        // 예측 데이터로 응답 매핑 
        return fdsPredictionResponse;
    }
}
```

3. REST API 컨트롤러:

```java
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/api/fds/")
@RequiredArgsConstructor
public class FdsPredictionController {
    private final FdsPredictionService fdsPredictionService;

    @PostMapping("predict")
    public FdsPredictionResponse predict(@RequestBody FdsPredictionRequest request) throws Exception {
        return fdsPredictionAppService.predict(request);
    }
}
```

- `PredictionService`가 애플리케이션 시작 시 모델을 로드한다.
- `predict` 메서드를 통해 입력을 받아 예측을 수행한다.
- `PredictionController`는 REST API 엔드포인트를 제공하여 클라이언트가 예측을 요청할 수 있게 한다. 

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

### 스트리밍 플랫폼 
#### Apache Flink
* FDS 사례 샘플 코드: [Apache Flink Docs: Fraud Detection with the DataStream API](https://nightlies.apache.org/flink/flink-docs-release-1.20/docs/try-flink/datastream/)
* [Flink 외 스트리밍 플랫폼 비교](https://carpe08.tistory.com/360)
* [Flink와 SQL](https://nightlies.apache.org/flink/flink-docs-release-1.20/docs/dev/table/sql/overview/)

#### Kafka Streams 
* [무신사: Kafka Streams를 활용한 실시간 이상 로그인 감지 시스템 도입하기](https://medium.com/musinsa-tech/허튼짓은-그만-kafka-streams를-활용한-실시간-이상-로그인-감지-시스템-도입하기-d05768b78c86)

#### Kafka KsqlDB
* <img width=400 src="https://github.com/user-attachments/assets/a318f85d-d197-48fc-93fa-df03fb2b4791">)
* [ksqldb]https://docs.ksqldb.io/en/latest/operate-and-deploy/how-it-works/
* [토스: 토스 ksqlDB를 활용한 증권사의 실시간 데이터 처리하기](https://toss.tech/article/ksqldb-realtime-data)
* [KSQL (KsqlDB)](https://always-kimkim.tistory.com/entry/kafka101-ksql)

### DJL: Deep Java Library
* AWS에서는 Java로 딥 러닝 모델을 개발하기 위한 오픈 소스 라이브러리
* 참고: [DJL: Deep Java Library](https://docs.djl.ai/master/index.html)





