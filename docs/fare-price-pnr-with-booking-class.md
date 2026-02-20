# Fare_PricePNRWithBookingClass

PNR의 항공편 세그먼트를 기반으로 운임을 계산하고 **TST(Transitional Stored Ticket)를 생성**하는 API 용어집입니다.
발권 직전 단계에서 최종 운임을 확정하고 저장하는 역할을 합니다.

!!! info "WBS Integration Flow Step 10"
    이 API는 [WBS Integration Flow](amadeus-wbs-integration-flow.md)의 **발권 단계 Step 10**에 해당합니다.

---

!!! warning "추후 업데이트 예정"
    상세 용어 정리는 추후 업데이트될 예정입니다.

---

## 개요

| 항목 | 내용 |
|------|------|
| API 명 | `Fare_PricePNRWithBookingClass` |
| 플로우 단계 | Step 10 — 발권 |
| 목적 | TST 생성 — 승객 유형별 운임 확정 및 PNR에 저장 |
| 이전 단계 | [FOP_CreateFormOfPayment](fop-create-form-of-payment.md) |
| 다음 단계 | [PNR_Retrieve](pnr-retrieve.md) (발권 전 확인) |

---

## 참고

- [WBS Integration Flow - Step 10](amadeus-wbs-integration-flow.md)
- [PNR_Retrieve 용어집](pnr-retrieve.md)
