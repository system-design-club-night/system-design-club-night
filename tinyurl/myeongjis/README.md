# Functional

- 대상 URL의 unique한 alias를 짧게 생성할 수 있어야 한다.
- shortURL을 통해 사용자가 origin URL로 접근할 수 있어야한다.
- shortURL은 사용자의 요구사항에 따라 기본 만료 시간이 있어야한다.

# Non-Functional

- 가용성
- 확장성
- 가독성
- 대기 시간

## Estimates

- 매일 1억 개의 단축 URL 생성 시
- 초당 쓰기 연산 100,000,000 / 24 \* 3600 => 약 1160
- 읽기 연산과 쓰기 연산의 비율이 10:1이라고 가정할 때, 읽기 연산은 초당 11,600
- 서비스 운영 기간을 10년으로 가정 시 1억 _ 365 _ 10 = 3650 억개의 레코드 저장 필요
- 축약 전의 URL의 길이는 평균 100 byte로, 10년 동안의 저장 용량은 약 3650억 \* 100byte = 36.5TB

## API

- URL 단축용 엔드포인트 (POST /api/v1/data/shorten)

- URL 리다이렉션용 엔드포인트 (GET /api/v1/shortUrl)

## 단축기 설계

shortURL 이 www.tinyurl.com/{hashValue}와 같은 형태라고 가정하면, shortner server는 hashValue를 통해 original URL을 찾을 수 있어야한다.

### 해시 값 길이 설정하기

hashValue는 [0-9, a-z, A-Z]의 문자들로 구성된다. 따라서 사용할 수 있는 문자의 개수는 10 + 26 + 26 = 62개이며, 추정치로 설정했던 3650억개의 URL을 만들기 위해서는 적어도 7자릿 수의 해시 값을 설정해야한다.(62^7 => 약 3.5조개 (3,521,614,606,208)

#### 구현 방법 1 - 해시함수 이용하기

해시 함수의 종류에는 CRC32, MD5, SHA-1와 같은 것들이 있으며, 다음은 예시문자열인 https://en.wikipedia.org/wiki/Systems_design을 축약한 결과다.

| **hash function** |         **result(Hexadecimal)**          |
| :---------------: | :--------------------------------------: |
|       CRC32       |                 5cb54054                 |
|        MD5        |     5a62509a84df93303fe1230b9df8b84e     |
|       SHA-1       | 0eeae7916c06853901d9ccbefbfcaf4de57ed85b |

그렇지만, 가장 결과값이 짧은 CRC32를 써도 7자릿 수를 넘어버린다.

#### 구현 방법2 - base-62 변환

> base-62 변환 과정 (10진수 11157을 62진수로 변환하기)
>
> - 0 -> 0, 9 -> 9, 10 -> a, 35 -> z, 36 -> A …61 -> Z로 대응시키면 된다. 즉, ‘a’는 10101, ‘Z’는 6110이다.
> - 11157 = 2 x 62^2 + 55 x 62 + 59 x 1 = [2, 55, 59] => [2, T, X] = 2TX이다.
> - 따라서 단축 URL은 https://tinyurl.com/2TX가 된다.


![image](https://github.com/system-design-club-night/system-design-club-night/assets/31172248/34553d67-2f70-43c2-8405-b3633b295d1b)

# Reference
- https://developers.naver.com/docs/utils/shortenurl/
- https://swengineer7.tistory.com/70
- 가장 면접 사례로 배우는 대규모 시스템 설계 기초 7장

  
# ETC
## 실무사례
- 특정 플랫폼의 임베디드된 광고 및 이벤트에 대한 트래픽 트래킹
- QR코드
