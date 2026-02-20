# Fare_InformativePricingWithoutPNR

PNR 없이 특정 운임을 **확정 조회**하고 `uniqueOfferReference`를 발급받는 API 용어집입니다.
FBA(Fare Basis Assignment) 옵션을 사용한 운임 기저 지정 가격 조회도 이 API를 통해 수행합니다.

!!! info "WBS Integration Flow Step 4 / Step 16"
    이 API는 [WBS Integration Flow](amadeus-wbs-integration-flow.md)의 **가격 조회 및 규정 확인 단계 Step 4** 및 **후속 처리 단계 Step 16 (FBA)**에 해당합니다.

---

!!! warning "추후 업데이트 예정"
    상세 용어 정리는 추후 업데이트될 예정입니다.

---

## 개요

| 항목 | 내용 |
|------|------|
| API 명 | `Fare_InformativePricingWithoutPNR` |
| 플로우 단계 | Step 4 — 가격 조회 및 규정 확인 / Step 16 — FBA 옵션 |
| 목적 | 운임 확정 조회 + Offer ID 발급, FBA 옵션으로 특정 Fare Basis 지정 가격 산출 |
| 이전 단계 | [Fare_InformativeBestPricingWithoutPNR](fare-informative-best-pricing.md) |
| 다음 단계 | [Fare_CheckRules](fare-check-rules.md) |

---

## 참고

- [WBS Integration Flow - Step 4](amadeus-wbs-integration-flow.md)
- [WBS Integration Flow - Step 16 (FBA)](amadeus-wbs-integration-flow.md)
