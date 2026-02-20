# Flight Tech Glossary

항공 기술 도메인의 용어를 정리하는 사전입니다.

---

## 카테고리

### 공통

| 카테고리 | 설명 | 주요 키워드 |
|---------|------|------------|
| [GDS 용어집](gds-terminology.md) | Sabre & Amadeus GDS 시스템 용어 | PNR, PCC, OID, BSP, Webservice, WSAP |
| [BSP 정산 가이드](bsp-settlement-guide.md) | IATA BSP 정산 프로세스 전체 가이드 | ADM, ACM, DSR, Billing, NDC, RBD |

### Amadeus

[WBS Integration Flow](amadeus-wbs-integration-flow.md) — IBE 예약 전체 플로우 가이드 (검색→발권→정산→취소)

#### 검색

| 카테고리 | 설명 | 주요 키워드 |
|---------|------|------------|
| [Fare_MasterPricerCalendar](fare-master-pricer-calendar.md) | 날짜 유연 최저가 달력 검색 API | Calendar, dayInterval, rangeQualifier |
| [Fare_MasterPricerTravelBoardSearch](master-pricer-travelboard-search.md) | 확정 날짜 항공편/운임 검색 API | MPTBS, Recommendation, Fare Family, Mini Rules |

#### 가격 조회 및 규정 확인

| 카테고리 | 설명 | 주요 키워드 |
|---------|------|------------|
| [Fare_InformativeBestPricingWithoutPNR](fare-informative-best-pricing.md) | 최적 운임 자동 계산 API | Best Pricer, Fare Family, ADT/CHD/INF |
| [Fare_InformativePricingWithoutPNR](fare-informative-pricing.md) | 운임 확정 조회 + Offer ID 발급 API | uniqueOfferReference, RLO, FBA |
| [Fare_CheckRules](fare-check-rules.md) | 운임 규정 전문 텍스트 조회 API | ATPCO Rule Category, Penalties, Sales Restrictions |
| [Fare_GetFareRules](fare-get-fare-rules.md) | 운임 규정 조회 API (PNR 기반) | ATPCO Rule Category, Fare Basis, Global Direction |
| [MiniRule_GetFromRec](minirule-get-from-rec.md) | 환불/변경 규정 구조화 조회 API | Refund, Reissue, Fee Qualifier |

#### 예약 생성

| 카테고리 | 설명 | 주요 키워드 |
|---------|------|------------|
| [Air_SellFromRecommendation](air-sell-from-recommendation.md) | 검색 결과 기반 좌석 예약 API | Segment Status, Marriage, Itinerary |
| [PNR_AddMultiElements](pnr-add-multi-elements.md) | PNR 요소 추가/수정 API | SSR, OSI, FOP, Ticketing, Remarks |
| [FOP_CreateFormOfPayment](fop-create-form-of-payment.md) | 결제 수단 등록 API | Credit Card, FortKnox, CVV, PCI-DSS |

#### 발권

| 카테고리 | 설명 | 주요 키워드 |
|---------|------|------------|
| [Fare_PricePNRWithBookingClass](fare-price-pnr-with-booking-class.md) | TST 생성 (운임 확정) API | TST, lastTktDate, FCA |
| [PNR_Retrieve](pnr-retrieve.md) | PNR 조회 API (발권 전/후 확인) | Record Locator, TST, FA, FB, TTP |

#### 후속 처리

| 카테고리 | 설명 | 주요 키워드 |
|---------|------|------------|
| [SalesReports](sales-reports.md) | 판매 실적 보고서 조회 API | AlternativeCurrency, DFC, SOF |
| [Cancellation Flow](cancellation-flow.md) | 취소 플로우 (3단계) | Ticket_CancelDocument, PNR_Cancel |

#### 기타

| 카테고리 | 설명 | 주요 키워드 |
|---------|------|------------|
| [Air FlightInfo](air-flightinfo.md) | 항공편 정보 조회 API | FLIFO, Marriage Segment, Equipment, Meal Service |

### 색인

| 카테고리 | 설명 | 주요 키워드 |
|---------|------|------------|
| [용어 색인](glossary.md) | 전체 용어 알파벳순 색인 | A-Z |

---

## 용어 추가 방법

`docs/` 폴더에 새로운 `.md` 파일을 추가하고, `mkdocs.yml`의 `nav` 섹션에 등록하면 자동으로 사이트에 반영됩니다.

```
docs/
  ├── index.md                            # 홈
  ├── gds-terminology.md                  # [공통] GDS 용어집
  ├── bsp-settlement-guide.md             # [공통] BSP 정산 가이드
  ├── amadeus-wbs-integration-flow.md     # [Amadeus] WBS Integration Flow
  ├── fare-master-pricer-calendar.md      # [검색] Fare_MasterPricerCalendar
  ├── master-pricer-travelboard-search.md # [검색] Fare_MasterPricerTravelBoardSearch
  ├── fare-informative-best-pricing.md    # [가격/규정] Fare_InformativeBestPricing
  ├── fare-informative-pricing.md         # [가격/규정] Fare_InformativePricing
  ├── fare-check-rules.md                 # [가격/규정] Fare_CheckRules
  ├── fare-get-fare-rules.md              # [가격/규정] Fare_GetFareRules
  ├── minirule-get-from-rec.md            # [가격/규정] MiniRule_GetFromRec
  ├── air-sell-from-recommendation.md     # [예약] Air_SellFromRecommendation
  ├── pnr-add-multi-elements.md           # [예약] PNR_AddMultiElements
  ├── fop-create-form-of-payment.md       # [예약] FOP_CreateFormOfPayment
  ├── fare-price-pnr-with-booking-class.md # [발권] Fare_PricePNRWithBookingClass
  ├── pnr-retrieve.md                     # [발권] PNR_Retrieve
  ├── sales-reports.md                    # [후속] SalesReports
  ├── cancellation-flow.md                # [후속] Cancellation Flow
  ├── air-flightinfo.md                   # [기타] Air FlightInfo
  ├── glossary.md                         # 용어 색인
  └── (새 문서).md                         # 새 카테고리 추가
```
