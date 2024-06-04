# Requirements

ex) OTT 정기구독, 특정일자 대금 납부, 할부

## Functional

- 구독 정보에 맞춰 결제 수행
- 결제 성공/실패 안내
- 결제 이력 관리
- 구독 상태 관리

## Non-Functional

- 결제수단 정보에 무단접근 방지를 위한 안전 저장
- 유연성 처리 (결제일/카드 변경, 구독취소)
- 결제처리 데이터 일관성 보장
- 추후 재결제 시도

## Estimates

- 일일 활성 사용자 수 10만명 + 사용자당 평균 요청 수 5회  
  -> 5.79 TPS
- 특정 마케팅 캠페인으로 2시간동안 위의 조건으로 유입되는 경우  
  -> 69.44 TPS

## Design

[![](https://mermaid.ink/img/pako:eNqNVF1vgjAU_StNX6fZOw9LVJbNZNmM6F7ALF25jEZoTVs0RPzvK58Wtsl4offr3HN7D5wxFSFgB0eJONGYSI02bsCReRYJA679-rVD0-lDsXrzNuheZZ-KSnbQTPACzVbLJ6LhRHLfHFFzRndolul4V0Ndc0oY5FkAvm0gD-SRUWjKeqGy_ztJWGiA0IrkqWGFljwSRWu1NNpgYzdg_aQKbg06kxxtxB7MIHa3P_p75Ah9b03Adrnz_kjufHcL7ZkpLWReoK0Caac0AQNXRvrVTewGdjPaGtRBcAX2mn5dSb3loB2cxhBmCUi_O3WrqeBdwpIcLWKg--H0H1bx6D3Y6TbvdoUvZtDiSmfArtakFBSUakuGarDo_EcXt-mY68ySMUKvQrMo73Irk1FSid02BmLvheyV_OywPVQfwbh0rOFHRIQnOAWZEhaaf8G5bBpgHUMKAXbMMSRyH-CAX0weybTwck6xo2UGE5xVbFxGviRJsRORRHXex5CZHo3z8g1Ic5Y0?type=png)](https://mermaid.live/edit#pako:eNqNVF1vgjAU_StNX6fZOw9LVJbNZNmM6F7ALF25jEZoTVs0RPzvK58Wtsl4offr3HN7D5wxFSFgB0eJONGYSI02bsCReRYJA679-rVD0-lDsXrzNuheZZ-KSnbQTPACzVbLJ6LhRHLfHFFzRndolul4V0Ndc0oY5FkAvm0gD-SRUWjKeqGy_ztJWGiA0IrkqWGFljwSRWu1NNpgYzdg_aQKbg06kxxtxB7MIHa3P_p75Ah9b03Adrnz_kjufHcL7ZkpLWReoK0Caac0AQNXRvrVTewGdjPaGtRBcAX2mn5dSb3loB2cxhBmCUi_O3WrqeBdwpIcLWKg--H0H1bx6D3Y6TbvdoUvZtDiSmfArtakFBSUakuGarDo_EcXt-mY68ySMUKvQrMo73Irk1FSid02BmLvheyV_OywPVQfwbh0rOFHRIQnOAWZEhaaf8G5bBpgHUMKAXbMMSRyH-CAX0weybTwck6xo2UGE5xVbFxGviRJsRORRHXex5CZHo3z8g1Ic5Y0)

## API

1. 구독 등록
   POST /subscription
2. 구독 정보 변경
   PATCH /subsciption/{id}
3. 구독 취소
   DELETE /subsciption/{id}
4. 구독 내역 확인
   GET /subscription

## Reference

### 함께 논의하고 싶은 주제

- 특정기간 할인 프로모션 적용 방식(프로모션 대상자 확인과정 캐싱 전략)

### 참고하기 좋은 기술 사례

[토스페이먼츠 자동결제](https://docs.tosspayments.com/reference#%EC%9E%90%EB%8F%99%EA%B2%B0%EC%A0%9C)  
[반복결제 설계 관련](https://www.geeksforgeeks.org/system-design-pattern-for-recurring-payments/)
