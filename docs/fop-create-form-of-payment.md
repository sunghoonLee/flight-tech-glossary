# FOP_CreateFormOfPayment

저장된 PNR에 **결제 수단(Form of Payment)을 등록**하는 API 용어집입니다.
발권 전 반드시 FOP가 PNR에 연결되어야 합니다.

!!! info "WBS Integration Flow Step 9"
    이 API는 [WBS Integration Flow](amadeus-wbs-integration-flow.md)의 **예약 생성 단계 Step 9**에 해당합니다.

---

!!! warning "추후 업데이트 예정"
    상세 용어 정리는 추후 업데이트될 예정입니다.

---

## 개요

| 항목 | 내용 |
|------|------|
| API 명 | `FOP_CreateFormOfPayment` |
| 플로우 단계 | Step 9 — 예약 생성 |
| 목적 | 신용카드 등 결제 수단 등록 및 FortKnox 토큰 발급 |
| 이전 단계 | [PNR_AddMultiElements](pnr-add-multi-elements.md) |
| 다음 단계 | [Fare_PricePNRWithBookingClass](fare-price-pnr-with-booking-class.md) |

---

## 참고

- [WBS Integration Flow - Step 9](amadeus-wbs-integration-flow.md)
- [PNR_AddMultiElements 용어집](pnr-add-multi-elements.md)
