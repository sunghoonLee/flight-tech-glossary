# SalesReports_DisplayQueryReport

특정 날짜의 오피스 판매 실적 보고서를 조회하는 API 용어집입니다.
**AlternativeCurrency**(외화 기준)와 **DFC**(현지 통화 기준) 두 가지 모드를 지원합니다.

!!! info "WBS Integration Flow Step 13-14"
    이 API는 [WBS Integration Flow](amadeus-wbs-integration-flow.md)의 **후속 처리 단계 Step 13 (AlternativeCurrency)** 및 **Step 14 (DFC)**에 해당합니다.

---

!!! warning "추후 업데이트 예정"
    상세 용어 정리는 추후 업데이트될 예정입니다.

---

## 개요

| 항목 | 내용 |
|------|------|
| API 명 | `SalesReports_DisplayQueryReport` |
| 플로우 단계 | Step 13-14 — 후속 처리 |
| 목적 | 일별 판매 실적 보고서 조회 (외화/현지 통화) |
| 이전 단계 | [PNR_Retrieve](pnr-retrieve.md) (발권 후 확인) |

---

## AlternativeCurrency vs DFC

| 구분 | AlternativeCurrency (Step 13) | DFC (Step 14) |
|------|-------------------------------|---------------|
| `currencyInfo` 블록 | **포함** (통화 코드 지정) | **미포함** |
| 응답 통화 | 요청한 대체 통화 (USD 등) | 오피스 기본 통화 (ARS 등) |
| 활용 | 외화 기준 정산 | 현지 통화 일일 정산 |

---

## 참고

- [WBS Integration Flow - Step 13-14](amadeus-wbs-integration-flow.md)
