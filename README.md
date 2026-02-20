# Flight Tech Glossary

항공 기술 도메인의 용어를 정리하는 사전입니다.

https://sunghoonlee.mintlify.app/

## 카테고리

### 공통

| 카테고리 | 설명 | 주요 키워드 |
|---------|------|------------|
| GDS 용어집 | Sabre & Amadeus GDS 시스템 용어 | PNR, PCC, OID, BSP, Webservice, WSAP |
| BSP 정산 가이드 | IATA BSP 정산 프로세스 전체 가이드 | ADM, ACM, DSR, Billing, NDC, RBD |

### Amadeus

WBS Integration Flow — IBE 예약 전체 플로우 가이드 (검색→발권→정산→취소)

#### 검색

| 카테고리 | 설명 | 주요 키워드 |
|---------|------|------------|
| Fare_MasterPricerCalendar | 날짜 유연 최저가 달력 검색 API | Calendar, dayInterval, rangeQualifier |
| Fare_MasterPricerTravelBoardSearch | 확정 날짜 항공편/운임 검색 API | MPTBS, Recommendation, Fare Family, Mini Rules |

#### 가격 조회 및 규정 확인

| 카테고리 | 설명 | 주요 키워드 |
|---------|------|------------|
| Fare_InformativeBestPricingWithoutPNR | 최적 운임 자동 계산 API | Best Pricer, Fare Family, ADT/CHD/INF |
| Fare_InformativePricingWithoutPNR | 운임 확정 조회 + Offer ID 발급 API | uniqueOfferReference, RLO, FBA |
| Fare_CheckRules | 운임 규정 전문 텍스트 조회 API | ATPCO Rule Category, Penalties |
| Fare_GetFareRules | 운임 규정 조회 API (PNR 기반) | ATPCO Rule Category, Fare Basis |
| MiniRule_GetFromRec | 환불/변경 규정 구조화 조회 API | Refund, Reissue, Fee Qualifier |

#### 예약 생성

| 카테고리 | 설명 | 주요 키워드 |
|---------|------|------------|
| Air_SellFromRecommendation | 검색 결과 기반 좌석 예약 API | Segment Status, Marriage, Itinerary |
| PNR_AddMultiElements | PNR 요소 추가/수정 API | SSR, OSI, FOP, Ticketing, Remarks |
| FOP_CreateFormOfPayment | 결제 수단 등록 API | Credit Card, FortKnox, CVV |

#### 발권

| 카테고리 | 설명 | 주요 키워드 |
|---------|------|------------|
| Fare_PricePNRWithBookingClass | TST 생성 (운임 확정) API | TST, lastTktDate, FCA |
| PNR_Retrieve | PNR 조회 API (발권 전/후 확인) | Record Locator, TST, FA, FB, TTP |

#### 후속 처리

| 카테고리 | 설명 | 주요 키워드 |
|---------|------|------------|
| SalesReports | 판매 실적 보고서 조회 API | AlternativeCurrency, DFC, SOF |
| Cancellation Flow | 취소 플로우 (3단계) | Ticket_CancelDocument, PNR_Cancel |

#### 기타

| 카테고리 | 설명 | 주요 키워드 |
|---------|------|------------|
| Air FlightInfo | 항공편 정보 조회 API | FLIFO, Marriage Segment, Equipment |

### 색인

| 카테고리 | 설명 |
|---------|------|
| 용어 색인 | 전체 용어 알파벳순 색인 (A-Z) |

## 기술 스택

- [Mintlify](https://mintlify.com/) 기반 문서 사이트
- GitHub 연동 자동 배포
