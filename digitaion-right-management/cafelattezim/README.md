## Requirements
1. DRM을 사용한 사용자 인증 제어가 반영되어야 한다.
2. 자격 있는 회원만 웹툰 이미지를 조회할 수 있다.

## diagram
```mermaid
sequenceDiagram
    participant User as 클라이언트
    participant API as API 서버
    participant DRM as DRM 서버
    participant Storage as 스토리지

    User->>API: 1. 웹툰 이미지 요청
    Note over API: 회원 권한 검증
    
    alt 권한 있음
        API->>DRM: 2. 라이선스 요청
        DRM-->>API: 3. DRM 라이선스 발급
        API->>Storage: 4. 암호화된 이미지 요청
        Storage-->>API: 5. 암호화된 이미지 전송
        
        API->>User: 6. 암호화된 이미지 + 라이선스 전송
        User->>DRM: 7. 복호화 키 요청
        DRM-->>User: 8. 복호화 키 제공
    else 권한 없음
        API-->>User: 접근 거부
    end
```

## 개념정리
### DRM 공급자
> pallycon 같은 중간 공급자가 필요한 이유는? 
> 플레이어에게 해당 콘텐츠 라이선스가 발급하는 역할?

### SPEKE

### Amazon CloudFront 보안
- 특정 콘텐츠에 대한 접근 제한 
- 인증된 사용자만 콘텐츠에 접근 가능  

이를 지원하는 두가지 방법으로 Signed URL, Signed Cookie가 있다. 
 
#### Signed URL
일반적인 콘텐츠 및 RTMP 서비스 사용시 권장한다. 

#### Signed Cookie
DASH, HLS와 같은 manifest file을 사용하는 미디어 재생 서비스에 권장한다.

### 참고
- [AWS Elemental MediaConvert및 Amazon CloudFront를 활용한 미디어 콘텐츠의 보안 강화 전략 – Part 2: VOD](https://aws.amazon.com/ko/blogs/tech/vod-media-security-on-aws/) 

