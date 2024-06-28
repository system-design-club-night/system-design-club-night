
## Functional Requirements

* `https://some-commerce-system.com/some-queryparam-very-long?=blahblahblah...blah` 와 같이 긴 url 을 짧은 URL 으로 단축시켜 제공해야 한다.
* 단축 url 은 원천 url 으로 리다이렉트 시켜야 한다.
* 한번 생성된 단축 url 은 영구적으로 원천 url 에 1:1 대응되어 쓰여야 한다.
* 단축 url 에는 숫자와 영어 대소문자만 허용한다. (특수문자는 허용하지 않는다.)



## Non-Functional Requirements
* 단축 url 통해 원천 url 으로 리다이렉트 하는 데에 레이턴시가 짦아야 한다.
* 고가용성, 규모 확장성이 가능해야 한다.


## Estimates
* 매일 1억개의 고유한 단축 url 생성 필요. -> 쓰기 요청 : 1억 / 60 / 60 / 24 = 약 1157 TPS
* 매일 10억개의 단축 url 방문 -> 읽기 요청 : 10억 / 60 / 60 / 24 = 약 11157 TPS
* 단축 url 서비스를 10년 간 운영한다면 총 3,650억개의 단축 url 생성 필요


## APIs

### 단축 URL 생성

**request**
* method : POST
* endpoint : `url-shortening-service.com/shortened-url`
* body
```
{
  "long_url" : "https://some-commerce-system.com/some-queryparam-very-long?=blahblahblah...blah"
}
```
**response**
* status code : 201
* body
```
{
  "shortened-url": "shortened.com/a30sbd"
}
```


### 단축 URL 으로 리소스 요청
**request**
* method : GET
* endpoint : `shortened.com/a30sbd`

**response**
* status code: 301
* headers
```
Location: https://some-commerce-system.com/some-queryparam-very-long?=blahblahblah...blah
```


## Design
https://broadleaf-hubcap-b1d.notion.site/shortening-url-8c0af7edbb2e4b5db6c3d9ffaca94385?pvs=4


## 논의하고 싶은 주제
* shortened url 은 상태코드 몇 번으로 응답하는 게 적절한가? (30x 상태코드들의 각 용도)
  * 레이턴시 줄이기 vs 단축 url 방문자 분석하기 
* 해싱 충돌을 방지하기 위해 보통 어떤 방법을 사용하시나요? 



## 참고하기 좋은 기술 사례
* [BloomFilter는 언제 쓰나요?](https://meetup.nhncloud.com/posts/192)
* [자바의 HashMap 은 어떻게 동작하는가?](https://d2.naver.com/helloworld/831311)

## 기타 레퍼런스
* [레이턴시 추산할 때 참고 할 만한 레퍼런스](https://gist.github.com/jboner/2841832)
* [Redirections in HTTP](https://developer.mozilla.org/en-US/docs/Web/HTTP/Redirections)
* [Which redirection code should I use for short url](https://linkila.com/blog/301-vs-302-vs-303-vs-307-vs-308-whic-redirection-code-should-i-use-for-short-urls/)
* [URL shortening system design](https://systemdesign.one/url-shortening-system-design/)
* [bloom filter](https://systemdesign.one/bloom-filters-explained/)
* [consistent hashing](https://binux.tistory.com/119)
* [Implementing bloom filters in Redis](https://aws.amazon.com/ko/blogs/database/powering-recommendation-models-using-amazon-elasticache-for-redis-at-coffee-meets-bagel/)
