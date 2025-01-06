# Requirements

권리 소유자 관계 관리를 포함하여 유형 및 무형 자산에 대한 모든 형태의 권리 사용에 대한 설명, 식별, 거래, 보호, 모니터링 및 추적

## Functional

- IP 자산 생성이 가능해야 한다.
  - 콘텐츠 생성 가능한 권한을 가진 사용자인지 확인ㅍ
  - 워크플로우 과정을 통해 생성 진행
- IP 자산 관리가 가능해야 한다.
  - 접근 및 검색 기능 제공 (메타데이터 관리 포함)
- IP 자산 사용이 가능해야 한다.
  - 권한을 가진 사용자만 사용 가능
  - 사용자 활동 추적 필수

## Non-Functional

- 보안
  - 권한이 없는 사용자는 콘텐츠 생성, 접근, 검색이 불가능해야 한다.
  - 모든 데이터는 전송 시 HTTPS, 저장 시 AES-128 방식으로 암호화되어야 한다.
- 성능 및 확장성
  - 동시 사용자 1,000명 이상 처리가 가능해야 한다.
  - 사용량 증가에 따라 확장 가능한 구조를 가져야 한다.
- 사용자 활동 추적 및 저장
  - 모든 사용자 활동은 추적 가능해야 하며, 1년간 저장되어야 한다.

## Estimates

## Design

### 콘텐츠 암호화 및 패키징

- DRM 시스템으로 유명한 Widevine 및 FairPlay 에서도 'AES-128' 암호화 방식을 사용
- 안전한 키 교환 프로토콜: 인증 토큰 및 서명된 URL

장치/브라우저 콘텐츠 복호화 모듈(CDM) <-> License Server +사이에서 중개 및 통신담당 EME API

1. CDM
   - 디지털 콘텐츠의 복호화를 처리하는 모듈 [관련글](https://wormwlrm.github.io/2023/03/05/DRM-Contents-on-Web.html)
2. License Server
   - 콘텐츠 암호화에 사용한 키를 관리함
3. EME API

### DataBase

1. 로그용 DB

- 목적: 사용자 활동 추적 및 시스템 로그 저장
- 주요 데이터: 사용자 활동 로그, 시스템 이벤트 로그
- 특징: 쓰기 집중, 보존 주기, 확장성
- 기술스택: Elasticsearch

2. 유저용 DB

- 목적: 사용자 정보 및 인증/인가 데이터 관리
- 주요 데이터: 사용자 프로필, 인증 정보, 활동 상태, 역할 및 권한
- 특징: 읽기/쓰기 혼합, 보안, 확장성
- 기술스택: PostgreSQL

3. 컨텐츠용 DB

- 목적: IP 자산(콘텐츠) 및 관련 메타데이터 저장
- 주요 데이터: 콘텐츠 메타데이터, 검색 인덱싱 데이터, 파일 스토리지 연동 정보
- 특징: 읽기 집중, 빠른 검색, 확장성
- 기술스택: S3

## API

## Reference

### 함께 논의하고 싶은 주제

### 참고하기 좋은 기술 사례

[AES 암호화 관련](https://www.vdocipher.com/blog/2020/11/aes-128-encryption-video-drm-secure/)  
https://www.dlib.org/dlib/june01/iannella/06iannella.html
https://medium.com/@hedhav.haresh/breaking-down-drm-and-its-implications-d8c4d2d6c51f
