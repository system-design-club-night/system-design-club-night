##Requirements
  ### Functional
  * 
  * 
  *
  ### Non-Functional
  * 
  * 
  *
  ### Estimates
  * 
  * 
  *
## Design
<img src="payment-gateway-design-1.png">

## API
1. 체크아웃/결제수단조회 
2. 주문접수
3. 결제상태조회 
4. 결제요청-인증 webhook

## Consumer
* 결제요청-인증 이벤트 구독 

## Batch
* 승인대사 배치 

## Reference
* tosspayments 개발자센터
  * [결제흐름 이해하기](https://docs.tosspayments.com/guides/learn/payment-flow#%EA%B8%B0%EB%B3%B8-%EA%B0%9C%EB%85%90-%EC%9A%94%EC%B2%AD-%EC%9D%B8%EC%A6%9D-%EC%8A%B9%EC%9D%B8) 
  * [redirect url](https://docs.tosspayments.com/reference/js-sdk#%EA%B2%B0%EC%A0%9C-%EC%9A%94%EC%B2%AD-%EC%9D%91%EB%8B%B5-%EC%B2%98%EB%A6%AC)
  * [webhook 개념](https://docs.tosspayments.com/resources/glossary/webhook)
  * [webhook 상세](https://docs.tosspayments.com/guides/webhook)
* [요즘IT: 이커머스 페이지 기획 시 알아야 하는 기본 용어](https://yozm.wishket.com/magazine/detail/664/)

## 이어서 
### 함께 논의하고 싶은 주제 
- PG 이중화 
  - ref. [배달의민족 기술블로그: 외부 시스템 장애에 대처하는 우리의 자세](https://techblog.woowahan.com/6447/)
  - 관련 키워드: #circuit breaker #벤더이중화 
- 결제승인연동 실패 케이스 대사작업 
   - ref. [tosspayments 개발자센터: 거래와 대사작업](https://docs.tosspayments.com/reference#%EA%B1%B0%EB%9E%98)   

### 참고하기 좋은 기술 사례
* 카카오페이 기술블로그
  * [MSA 환경에서 네트워크 예외를 잘 다루는 방법](https://tech.kakaopay.com/post/msa-transaction/)
  * [온라인 결제 서비스 2.5배 성능 개선기](https://tech.kakaopay.com/post/improve-service-performance/)
  * [끊김없는 게임 플레이를 실현한 카카오페이 결제 특허](https://tech.kakaopay.com/post/kakaopay-payment-patent/#dqr%EC%9D%80-%EB%AD%90%EA%B3%A0-%EC%96%B4%EB%96%A4-%EB%B6%80%EB%B6%84%EC%9C%BC%EB%A1%9C-%ED%8A%B9%ED%97%88%EB%A5%BC-%EB%82%B8%EA%B1%B0%EC%95%BC)
  * [카카오페이 기술블로그: WebFlux와 코루틴으로 BFF(Backend For Frontend) 구현하기](https://tech.kakaopay.com/post/bff_webflux_coroutine/#%EB%82%B4-%EC%A3%BC%EB%B3%80-%EB%A7%A4%EC%9E%A5-%EC%B0%BE%EA%B8%B0-bff-%EC%95%84%ED%82%A4%ED%85%8D%EC%B2%98)
  
* 배달의민족
   * [결제 담당자가 장애에 대응하는 방법](https://techblog.woowahan.com/15236/)
   * [우아콘: 대규모 트랜잭션을 처리하는 배민 주문시스템 규모에 따른 진화](https://www.youtube.com/watch?v=704qQs6KoUk)
