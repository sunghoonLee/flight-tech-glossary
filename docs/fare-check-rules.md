# Fare_CheckRules

확정된 운임의 **상세 운임 규정(Fare Rules)을 전문 텍스트로 조회**하는 API 용어집입니다.
2단계 호출 구조로, 1차 Category 목록 조회 후 2차 특정 Category 전문을 조회합니다.

!!! info "WBS Integration Flow Step 5"
    이 API는 [WBS Integration Flow](amadeus-wbs-integration-flow.md)의 **가격 조회 및 규정 확인 단계 Step 5**에 해당합니다.

---

!!! warning "추후 업데이트 예정"
    상세 용어 정리는 추후 업데이트될 예정입니다.

---

## 개요

| 항목 | 내용 |
|------|------|
| API 명 | `Fare_CheckRules` |
| 플로우 단계 | Step 5 — 가격 조회 및 규정 확인 |
| 목적 | ATPCO Rule Category별 운임 규정 전문 텍스트 조회 |
| 이전 단계 | [Fare_InformativePricingWithoutPNR](fare-informative-pricing.md) |
| 다음 단계 | [MiniRule_GetFromRec](minirule-get-from-rec.md) |

---

## Fare_CheckRules vs Fare_GetFareRules

| 항목 | Fare_CheckRules | Fare_GetFareRules |
|------|----------------|-------------------|
| 입력 | Recommendation 기반 | Stored Pricing / PNR 기반 |
| 용도 | 예약 전 운임 규정 조회 | 예약 후 운임 규정 조회 |
| 플로우 위치 | Step 5 (가격 조회) | 예약 후 규정 재확인 |

---

## 참고

- [WBS Integration Flow - Step 5](amadeus-wbs-integration-flow.md)
- [Fare_GetFareRules 용어집](fare-get-fare-rules.md)
- [MiniRule_GetFromRec 용어집](minirule-get-from-rec.md)
