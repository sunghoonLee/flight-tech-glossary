# Fare_CheckRules

확정된 운임의 **상세 운임 규정(Fare Rules)을 전문 텍스트로 조회**하는 API 용어집입니다.
2단계 호출 구조로, 1차 Category 목록 조회 후 2차 특정 Category 전문을 조회합니다.

> 기반 문서: Fare_CheckRules 07.1 Technical Reference

---

## 1. 개요

### Fare_CheckRules

Amadeus GDS가 제공하는 **ATPCO Rule Category별 운임 규정 전문 텍스트 조회 API**. Fare Display에 연결하거나 독립적으로 호출하여, 항공사/운임 공급자가 게시한 운임 사용 조건 및 규정 텍스트를 표시한다.

| 항목 | 내용 |
|------|------|
| API 명 | `Fare_CheckRules` |
| 버전 | 07.1.1A |
| 플로우 단계 | Step 5 -- 가격 조회 및 규정 확인 |
| 목적 | ATPCO Rule Category별 운임 규정 전문 텍스트 조회 |
| 이전 단계 | [Fare_InformativePricingWithoutPNR](fare-informative-pricing.md) |
| 다음 단계 | [MiniRule_GetFromRec](minirule-get-from-rec.md) |

```
[Query: 규정 조회 요청]
  운임 식별 정보 (Fare Basis, 항공사, 구간, 날짜)
       │
       ▼
  Amadeus Fare Rules Engine
       │
       ▼
[Reply: 규정 조회 결과]
  tariffInfo × N개
  ├─ fareRuleInfo: Tariff / Rule 식별
  ├─ fareRuleText: 규정 전문 텍스트 (최대 999줄)
  ├─ flightDetails: 구간별 상세 (통화, 날짜 등)
  ├─ productInfo: Fare Class, Cabin 정보
  ├─ priceInfo: 금액 및 세금
  └─ fareRouteGrp: 적용 노선 정보
```

!!! info "WBS Integration Flow Step 5"
    이 API는 [WBS Integration Flow](amadeus-wbs-integration-flow.md)의 **가격 조회 및 규정 확인 단계 Step 5**에 해당합니다.

---

## 2. 2단계 호출 구조

Fare_CheckRules는 **2단계 호출(Two-Step Call)** 패턴으로 동작한다. 1차 호출로 사용 가능한 Category 목록을 받고, 2차 호출로 특정 Category의 전문 텍스트를 조회한다.

```
┌─────────────────────────────────────────────────────┐
│  1차 호출: Category 목록 요청 (LIST)                   │
│  fareType = "RL" (Rule List Display)                 │
│                                                     │
│  [Request]                                          │
│   ├─ msgType: 메시지 기능 (711 = Fare Display)       │
│   ├─ flightQualification: 운임 식별 조건              │
│   │   ├─ fareType: "RL"                             │
│   │   └─ fareCategories                             │
│   ├─ transportInformation: 항공편 정보               │
│   │   ├─ transportService: 항공사 코드               │
│   │   └─ routingInfo: 출발지/도착지                   │
│   └─ fare: 운임 상세                                 │
│       ├─ detailsOfFare: Fare Basis Code             │
│       └─ tarifFareRule: Tariff/Rule 번호            │
│                                                     │
│  [Response]                                         │
│   └─ tariffInfo: Category 목록                      │
│       ├─ fareRuleInfo: Rule/Category 코드 목록       │
│       └─ fareRuleText: 각 Category 제목 텍스트       │
└─────────────────────────────────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────────┐
│  2차 호출: 특정 Category 전문 요청                     │
│  fareType = "RD" (Rule Display)                      │
│                                                     │
│  [Request]                                          │
│   ├─ fareType: "RD"                                 │
│   ├─ fareCategories: 조회할 Category 코드             │
│   │   (예: CAT16 = Penalty, CAT06 = Min Stay)       │
│   └─ (1차와 동일한 운임 식별 정보)                     │
│                                                     │
│  [Response]                                         │
│   └─ tariffInfo: 선택 Category 전문                  │
│       ├─ fareRuleInfo: Rule/Tariff 식별              │
│       └─ fareRuleText × 999줄: 규정 전문 텍스트       │
└─────────────────────────────────────────────────────┘
```

---

## 3. Query / Reply 구조

### Query 메시지: Fare_CheckRules 07.1.1A

운임 및 운임 관련 데이터 요청. 항공사/항공 서비스 공급자가 운임에 관련된 사용 조건 및 규정 텍스트 표시를 요청한다. Fare Display에 연결하거나 독립 표시(Standalone Display)할 수 있다.

| Entity | Structure | St | Rep | 설명 |
|--------|-----------|-----|-----|------|
| `msgType` | Message action details | M | 1 | 메시지 유형 및 비즈니스 기능 |
| `availcabinStatus` | Product information | C | 1 | 좌석 가용 상태 또는 캐빈 설정 |
| `conversionRate` | Conversion rate | C | 1 | 통화 환산 비율 |
| `pricingTickInfo` | Pricing/ticketing details | C | 1 | 가격 산정 및 발권 상세 |
| `multiCorporate` | Corporate number/name | C | 1 | 기업 계약 정보 |
| `itemNumber` | Item number | C | 1 | 항목 번호 |
| `dateOfFlight` | Date and time information | C | 1 | 항공편 날짜/시간 |
| `flightQualification` | Fare qualifier details | C | 99 | 운임 자격 조건 (fareType, fareCategories 포함) |
| `transportInformation` | Group | C | 99 | 항공편 정보 그룹 |
| -- `transportService` | Transport identifier | M | 1 | 항공사 코드 (Marketing/Operating) |
| -- `availCabinConf` | Product information | C | 1 | 캐빈 설정 |
| -- `routingInfo` | Routing information | C | 1 | 경유지 정보 |
| -- `selectionDetail` | Selection details | C | 1 | 조건 선택 상세 |
| `tripDescription` | Group | C | 99 | 여정 정보 그룹 |
| -- `origDest` | Origin and destination details | M | 1 | 출발지/도착지 |
| -- `dateFlightMovement` | Date and time information | C | 1 | 항공편 이동 날짜 |
| -- `routing` | Group | C | 99 | 경유 정보 그룹 |
| `pricingInfo` | Group | C | 9 | 가격 정보 그룹 |
| -- `numberOfUnits` | Number of units | M | 1 | 단위 수 |
| -- `ticketPricingDate` | Pricing/ticketing details | C | 3 | 발권/가격 일자 |
| `fare` | Group | C | 99 | 운임 그룹 |
| -- `detailsOfFare` | Fare information | M | 1 | 운임 상세 (Fare Basis 등) |
| -- `fareQualificationDetails` | Fare qualifier details | C | 99 | 운임 자격 조건 |
| `fareRule` | Group | C | 1 | 운임 규정 그룹 |
| -- `tarifFareRule` | Fare rules information | M | 1 | Tariff, 공급자, Paragraph 번호 |
| -- `travellerIdentification` | Reference information | C | 1 | 여행자 식별 참조 |
| -- `travellerDate` | Date and time information | C | 1 | 여행 날짜 |

### Reply 메시지: Fare_CheckRulesReply 07.1.1A

운임 및 운임 관련 데이터 응답.

| Entity | Structure | St | Rep | 설명 |
|--------|-----------|-----|-----|------|
| `transactionType` | Message action details | M | 1 | 가격 산정 트랜잭션 |
| `statusInfo` | Status details | C | 1 | 메시지 상태 |
| `fareRouteInfo` | Fare route information | C | 1 | 운임 노선 정보 |
| `infoText` | Interactive free text | C | 999 | 정보 텍스트 |
| `errorInfo` | Group | C | 1 | 오류 보고 그룹 |
| -- `rejectErrorCode` | Application error information | M | 1 | 애플리케이션 오류 코드 |
| -- `errorFreeText` | Interactive free text | C | 1 | 오류 자유 텍스트 |
| `tariffInfo` | Group | C | 999 | **운임/규정 정보 (핵심 응답 구조)** |
| -- `fareRuleInfo` | Fare rules information | M | 1 | Rule, Tariff, Category 코드 |
| -- `fareRuleText` | Interactive free text | C | 999 | **규정 전문 텍스트** |
| `flightDetails` | Group | C | 999 | 구간별 상세 (통화, 날짜 등) |
| -- `nbOfSegments` | Segment repetition control | M | 1 | 구간 수 |
| -- `amountConversion` | Conversion rate | C | 1 | 환산 금액 |
| -- `quantityValue` | Quantity | C | 1 | 수량 (마일리지 등) |
| -- `pricingAndDateInfo` | Pricing/ticketing details | C | 1 | 가격/날짜 정보 |
| -- `qualificationFareDetails` | Fare qualifier details | C | 99 | 운임 자격 상세 |
| -- `transportService` | Transport identifier | C | 4 | 항공사 정보 |
| -- `flightErrorCode` | Interactive free text | C | 999 | 구간별 오류 정보 |
| `productInfo` | Group | C | 99 | 상품 정보 (Fare Class 등) |
| -- `productDetails` | Product information | M | 1 | 캐빈/Booking Class 상세 |
| -- `productErrorCode` | Interactive free text | C | 99 | 상품 오류 정보 |
| `priceInfo` | Group | C | 99 | 금액 및 세금 정보 |

---

## 4. ATPCO Rule Categories

ATPCO(Airline Tariff Publishing Company)가 정의한 **운임 규정 카테고리 체계**. 각 Category는 운임 사용 조건의 특정 측면을 규정한다. `ruleSectionId` 필드에 Category 코드가 전달된다.

### 주요 Category 목록

| Category | 코드 | 명칭 | 설명 |
|----------|------|------|------|
| CAT 1 | `1` | Eligibility | 적용 대상 승객 자격 조건 (군인, 노인, 학생 등) |
| CAT 2 | `2` | Day/Time Application | 요일/시간대별 적용 조건 |
| CAT 3 | `3` | Seasonality | 계절별 적용 조건 (성수기/비수기) |
| CAT 4 | `4` | Flight Application | 적용 항공편 제한 (특정 편명, 코드셰어 등) |
| CAT 5 | `5` | Advance Reservation/Ticketing | 사전 예약/발권 기한 (AP = Advance Purchase) |
| CAT 6 | `6` | Minimum Stay | 최소 체류 기간 (예: 토요일 포함 필수) |
| CAT 7 | `7` | Maximum Stay | 최대 체류 기간 (예: 출발일로부터 1개월 이내) |
| CAT 8 | `8` | Stopovers | 경유(Stopover) 허용 여부 및 조건 |
| CAT 9 | `9` | Transfers | 환승(Transfer) 허용 여부 및 조건 |
| CAT 10 | `10` | Combinations | 운임 조합 규정 (편도/왕복/오픈죠 등) |
| CAT 11 | `11` | Blackout Dates | 적용 불가 기간 (블랙아웃 날짜) |
| CAT 12 | `12` | Surcharges | 추가 요금 (유류할증료, 보험료 등) |
| CAT 13 | `13` | Accompanied Travel | 동반 여행 조건 |
| CAT 14 | `14` | Travel Restrictions | 여행 제한 사항 |
| CAT 15 | `15` | Sales Restrictions | 판매 제한 (판매 지역, 대리점 제한 등) |
| **CAT 16** | `16` | **Penalties** | **변경/환불 수수료 (핵심 규정)** |
| CAT 17 | `17` | HIP/Mileage Exceptions | HIP(Higher Intermediate Point) 마일리지 예외 |
| CAT 18 | `18` | Ticket Endorsements | 항공권 배서 제한 (타 항공사 이용 불가 등) |
| CAT 19 | `19` | Children Discounts | 소아 할인 |
| CAT 20 | `20` | Tour Conductor Discounts | 투어 컨덕터 할인 |
| CAT 21 | `21` | Agent Discounts | 대리점 할인 |
| CAT 22 | `22` | All Other Discounts | 기타 할인 |
| CAT 23 | `23` | Miscellaneous Fare Tags | 기타 운임 태그 |
| CAT 25 | `25` | Fare By Rule | 규정 기반 운임 (다른 운임에서 파생) |
| CAT 26 | `26` | Groups | 단체 운임 조건 |
| CAT 27 | `27` | Tours | 투어 운임 조건 |
| CAT 28 | `28` | Visit Another Country | 타국 방문 조건 |
| CAT 29 | `29` | Deposits | 예치금/보증금 조건 |
| CAT 31 | `31` | Voluntary Changes | 자발적 변경 규정 |
| CAT 33 | `33` | Voluntary Refunds | 자발적 환불 규정 |
| CAT 35 | `35` | Baggage Provisions | 수하물 규정 |
| CAT 50 | `50` | Application | 적용 범위 (General Rule) |

### Category 조회 흐름 다이어그램

```
1차 호출 (LIST)                      2차 호출 (DETAIL)
fareType="RL"                        fareType="RD"
    │                                    │
    ▼                                    ▼
┌──────────────┐                  ┌──────────────────┐
│ Category 목록  │                  │ Category 전문      │
├──────────────┤                  ├──────────────────┤
│ CAT 5  AP    │  ── 선택 ──▶     │ ADVANCE PURCHASE │
│ CAT 6  MNS   │                  │ RESERVATIONS MUST│
│ CAT 7  MXS   │                  │ BE MADE AT LEAST │
│ CAT 8  STP   │                  │ 14 DAYS BEFORE   │
│ CAT 10 CMB   │                  │ DEPARTURE.       │
│ CAT 14 TVL   │                  │ TICKETING MUST BE│
│ CAT 15 SLS   │                  │ COMPLETED WITHIN │
│ CAT 16 PEN   │  ── 선택 ──▶     │ 3 DAYS AFTER...  │
│ CAT 18 END   │                  └──────────────────┘
│ CAT 35 BAG   │
└──────────────┘
```

---

## 5. 주요 Simple Structure

### FARE QUALIFIER DETAILS

운임 자격을 정의하는 핵심 구조. `fareType`으로 요청 유형(RL/RD)을 지정하고, `fareCategories`로 Category 코드를 전달한다.

| Entity | Structure | St | Rep | Fmt | 설명 |
|--------|-----------|-----|-----|-----|------|
| `movementType` | Movement type, coded | C | 1 | an..3 | 운임의 글로벌 방향. 빈 값이면 전체 노선. Codeset: 8335 |
| `fareCategories` | Fare category codes | C | 1 | -- | 확장 파라미터 및 요청 유형 |
| -- `fareType` | Rate type identification | M | 1~9 | an..20 | **요청 유형 및 확장 파라미터**. Codeset: 5263 |
| -- `otherFareType` | Rate type identification | C | 8 | an..20 | 추가 요청 유형. Codeset: 5263 |
| `fareDetails` | Fare details | C | 1 | -- | 운임 상세 |
| -- `qualifier` | Number of units qualifier | C | 1 | an..3 | 승객 유형 (ATPCO PTC). Codeset: 6353 |
| -- `rate` | Percentage | C | 1 | n..8 | 할인 운임 비율 |
| -- `country` | Country, coded | C | 1 | an..3 | ISO 국가 코드 |
| -- `fareCategory` | Fare classification type | C | 1 | an..3 | ATPCO 운임 분류 유형. Codeset: 9878 |
| `additionalFareDetails` | Additional fare qualifier | C | 1 | -- | 기타 상세 |
| -- `rateClass` | Rate/Tariff class | C | 1 | an..35 | **Fare Basis Code** |
| -- `commodityCategory` | Commodity/rate identification | C | 1 | an..18 | Ticket Designator |
| -- `pricingGroup` | Pricing Group | C | 1 | an..35 | 가격 그룹 |
| `discountDetails` | Discount/penalty information | C | 9 | -- | 할인/패널티 상세 |
| -- `fareQualifier` | Fare qualifier | M | 1 | an..3 | Max/Min Stay, AP 등. Codeset: 9910 |
| -- `rateCategory` | Rate/Tariff class | C | 1 | an..35 | 적용 Rate Category |
| -- `amount` | Monetary amount | C | 1 | n..18 | 패널티 금액 |
| -- `percentage` | Percentage | C | 1 | n..8 | 패널티 비율 |

### FARE RULES INFORMATION

Tariff, 항공사, Rule 번호를 식별하는 구조. Query의 `tarifFareRule`과 Reply의 `fareRuleInfo`에 사용된다.

| Entity | Structure | St | Rep | Fmt | 설명 |
|--------|-----------|-----|-----|-----|------|
| `tariffClassId` | Rate/tariff class identification | C | 1 | an..9 | **Tariff 번호** (Reply) / Rule 번호 (Query) |
| `companyDetails` | Company identification | C | 1 | -- | 항공사 상세 |
| -- `marketingCompany` | Company identification | C | 1 | an..3 | 마케팅 항공사 코드. Codeset: 9906 |
| -- `operatingcompany` | Company identification | C | 1 | an..3 | 운항 항공사 코드. Codeset: 9906 |
| -- `otherCompany` | Company identification | C | 1 | an..3 | 기타 항공사 코드. Codeset: 9906 |
| `ruleSectionId` | Rule part identification | C | 99 | an..7 | **Fare Rule Paragraph 번호 / Category 코드**. `700` = 전체 Paragraph. Codeset: 7175 |

### FARE ROUTE INFORMATION

운임 노선 정보를 제공하는 구조. Reply에서 해당 운임이 적용되는 노선의 상세를 전달한다.

| Entity | Structure | St | Rep | Fmt | 설명 |
|--------|-----------|-----|-----|-----|------|
| `dayOfWeek` | Days of operation | C | 1 | an..7 | 운임 적용 요일 |
| `fareQualifierDetails` | Fare qualifier information | C | 1 | -- | Fare Qualifier 상세 |
| -- `fareQualifier` | Fare qualifier | C | 3 | an..3 | Round Trip, Base Fare, Unsealable 등. Codeset: 9910 |
| `identificationNumber` | Identity number | C | 1 | an..35 | Routing 번호 |
| `validityPeriod` | Valid date information | C | 1 | -- | 적용 기간 |
| -- `firstDate` | First date | C | 1 | n..6 | 노선 유효 시작일 |
| -- `secondDate` | Second date | C | 1 | n..6 | 노선 유효 종료일 |

### INTERACTIVE FREE TEXT

규정 전문 텍스트를 전달하는 구조. Reply의 `fareRuleText`가 이 구조를 사용한다.

| Entity | Structure | St | Rep | Fmt | 설명 |
|--------|-----------|-----|-----|-----|------|
| `freeTextQualification` | Free text qualification | C | 1 | -- | 텍스트 분류 |
| -- `textSubjectQualifier` | Text subject qualifier | M | 1 | an..3 | 텍스트 유형 (코드/자유). Codeset: 4451 |
| -- `informationType` | Information type | C | 1 | an..4 | 메시지 유형 코드. Codeset: 9980 |
| `freeText` | Free text | C | 99 | an..70 | **규정 전문 텍스트** (최대 99줄 x 70자) |

### MONETARY INFORMATION

금액 정보를 전달하는 구조.

| Entity | Structure | St | Rep | Fmt | 설명 |
|--------|-----------|-----|-----|-----|------|
| `monetaryDetails` | Monetary information | M | 1 | -- | 운임 금액 및 유형 (OW, RT 등) |
| -- `typeQualifier` | Monetary amount type qualifier | M | 1 | an..3 | One Way / Round Trip 등. Codeset: 5025 |
| -- `amount` | Allowance or charge number | M* | 1 | an..18 | 금액 |
| -- `currency` | Currency, coded | C | 1 | an..3 | 통화 코드 (ISO). Codeset: 6345 |

---

## 6. Fare_CheckRules vs Fare_GetFareRules

| 항목 | Fare_CheckRules | Fare_GetFareRules |
|------|----------------|-------------------|
| **입력** | Recommendation 기반 (운임 검색 결과) | Stored Pricing / PNR 기반 |
| **용도** | 예약 전 운임 규정 조회 | 예약 후 운임 규정 조회 |
| **플로우 위치** | Step 5 (가격 조회 단계) | 예약 후 규정 재확인 |
| **PNR 필요 여부** | 불필요 | PNR 필수 |
| **호출 방식** | 2단계 (LIST → DETAIL) | 직접 조회 |
| **주요 사용처** | 예약 전 고객에게 규정 안내 | 발권 전 규정 최종 확인 |

---

## 7. 주요 Codeset 테이블

### fareType (Rate Type Identification) -- Codeset 5263

운임 요청 유형 및 확장 파라미터를 지정한다. `flightQualification.fareCategories.fareType`에 사용.

| 값 | 설명 | 비고 |
|------|------|------|
| `RL` | Rule List Display | 1차 호출: Category 목록 조회 |
| `RD` | Rule Display (Detail) | 2차 호출: 특정 Category 전문 조회 |
| `RP` | Published Fare | 공시 운임 |
| `RU` | Unifares | 통합 운임 |
| `RC` | Corporate Fare | 기업 계약 운임 |
| `RW` | Lowest Fare | 최저가 운임 |

### messageFunction -- Codeset 1225

메시지 기능 코드. `msgType.messageFunction`에 사용.

| 값 | 설명 |
|------|------|
| `711` | Fare Display by City Pair |
| `712` | Display Specific Fare Rules |
| `713` | Display Fare Routing Information |
| `714` | City Text Information |
| `715` | Fare Mileage Display (primary) |
| `724` | Reservation Booking (Code) Designator Display |
| `725` | Fare Construction Display |

### fareQualifier -- Codeset 9910

운임 자격 한정자. `discountDetails.fareQualifier` 및 `fareRouteInfo.fareQualifier`에 사용.

| 값 | 설명 |
|------|------|
| `700` | Add-on origin |
| `701` | Add-on destination |
| `702` | Advances purchase period |
| `709` | Cancellation fee details |
| `722` | Fare display with rules |
| `730` | Fares with no penalties |
| `731` | Fares with penalties |
| `736` | Indicates fare is refundable |
| `737` | Indicates no refund restriction |
| `741` | Indicates the fare is non-refundable |
| `745` | Minimum stay in days |
| `746` | Maximum stay in days |
| `756` | One way |
| `763` | Round trip |
| `764` | Rule list display |
| `765` | Rule number |
| `795` | Tickets are non-refundable |
| `797` | Penalties applies |
| `798` | Subject to cancellation/change penalty |

### typeQualifier (Monetary Amount Type) -- Codeset 5025

운임 금액 유형. `monetaryDetails.typeQualifier`에 사용.

| 값 | 설명 |
|------|------|
| `700` | One way |
| `701` | Round trip |
| `702` | PFC (Passenger Facilities Charge) |
| `703` | Stopover |
| `704` | Open Jaw surcharge |
| `708` | Percentage |
| `712` | Total fare amount |
| `713` | Total amount of all surcharges |
| `714` | Refund amount |
| `715` | Fare difference amount |
| `716` | Change fee - penalty and/or administrative fee |
| `B` | Base fare |
| `E` | Equivalent fare |
| `H` | Net fare amount |
| `M` | Ticket total amount |
| `T` | Ticket document amount (base, tax, fee) |

### conversionType -- Codeset 9875

통화 환산 유형. `conversionRate.conversionType`에 사용.

| 값 | 설명 |
|------|------|
| `700` | Fares |
| `701` | Not rounded and if insufficient space, truncated |
| `702` | City |
| `703` | Country |
| `704` | Equivalent fare |
| `705` | Origin of travel |
| `706` | Private alternative currency |
| `707` | Equivalent conversion currency |
| `B` | Base |
| `C` | Net Base Fare |
| `D` | Total Base Fare related to all passengers |
| `E` | Public Equivalent Fare |
| `F` | Public Base Fare |
| `G` | Grand Total Fare related to all passengers |
| `H` | Total Tax Fare related to all passengers |
| `M` | Maximum Penalty amount |
| `T` | Tax |

### movementType (Global Direction) -- Codeset 8335

운임의 글로벌 방향 지시자. `flightQualification.movementType`에 사용.

| 값 | 설명 |
|------|------|
| `7AP` | AP - Eastern Hemisphere(TC2) ↔ Eastern Hemisphere(TC3) via Atlantic and Pacific |
| `7AT` | AT - Eastern Hemisphere(TC2&3) ↔ Western Hemisphere(TC1) via Atlantic |
| `7CA` | CA - Domestic Canada |
| `7CT` | CT - Circle Trip |
| `7EH` | EH - Within Eastern Hemisphere |
| `7FE` | FE - Far East |
| `7PA` | PA - TC2&3 ↔ TC1 via Pacific |
| `7RU` | RU - Russia |
| `7RW` | RW - Round the World |
| `7SA` | SA - South Atlantic |
| `7TS` | TS - Trans-Siberian |
| `7WH` | WH - Within Western Hemisphere |

### Information Type -- Codeset 9980

정보 유형 코드. `freeTextQualification.informationType`에 사용.

| 값 | 설명 |
|------|------|
| `BAT` | Base Fare rule category |
| `CAT` | Fare rule category |
| `FTC` | Fare Type Code full text |
| `PTC` | Passenger Type Code full text |
| `TXT` | Other conditions - see text |

### corporateQualifier (Price Type) -- Codeset 5387

기업 운임 유형 구분. `multiCorporate.corporateQualifier`에 사용.

| 값 | 설명 |
|------|------|
| `RC` | Corporate |
| `RD` | DDF Corporate |
| `RB` | ATPCO Corporate |
| `RR` | ATPCO Private Corporate |
| `RZ` | Corporate Unifare |

---

## 8. 주요 오류 코드 (Fare Rules 관련)

Reply의 `rejectErrorCode.errorCode`에 반환되는 주요 오류 코드. Codeset: 9321.

| 코드 | 설명 |
|------|------|
| `719` | No fares available (사용 가능한 운임 없음) |
| `720` | No rules exist for this fare (해당 운임에 규정 없음) |
| `722` | Invalid rule (유효하지 않은 규정) |
| `723` | Invalid category (유효하지 않은 카테고리) |
| `724` | Invalid routing (유효하지 않은 라우팅) |
| `715` | Invalid fare basis (유효하지 않은 Fare Basis) |
| `730` | No fare on this market and/or carrier |
| `734` | Too many fares. Enter specific date and/or fare type |
| `376` | Pricing/ticketing error, text information specified |
| `378` | No rule pricing error, text information specified |
| `404` | No Service Between Requested Cities/Airports |
| `900` | Inactivity Time Out Value Exceeded |
| `911` | Unable to process - system error |
| `912` | Incomplete message - data missing in query |

---

## 9. 메시지 구조 용어

Fare_CheckRules 기술 문서에서 사용되는 **메시지 구조 정의 용어**.

| 용어 | 설명 |
|------|------|
| **Entity** | 메시지 내 데이터 항목의 참조 이름 |
| **Structure** | Entity의 정식 명칭과 참조 번호 |
| **Rep (Repetitions)** | 상위 구조 내에서의 반복 횟수 |
| **St (Status)** | 필수 여부. M=Mandatory, C=Conditional, M*=구현 시 필수 |
| **Fmt (Format)** | 데이터 형식. a=문자, n=숫자, an=영숫자, ..x=가변 길이 |
| **Grouped Structure** | 하위 구조를 포함하는 복합 구조 (계층 구조) |
| **Simple Structure** | 데이터 요소만 포함하는 단순 구조 |
| **Codeset** | 코드화된 데이터 항목의 가능한 값 목록 |

### 데이터 형식 표기법

| 표기 | 의미 | 예시 |
|------|------|------|
| `a3` | 고정 3자리 문자 | `ICN` |
| `n6` | 고정 6자리 숫자 | `150326` (날짜) |
| `an..35` | 가변 영숫자 최대 35자리 | `YOWKR` (Fare Basis) |
| `n..18` | 가변 숫자 최대 18자리 | `528400` (금액) |
| `an..7` | 가변 영숫자 최대 7자리 | `16` (Category 코드) |

---

## 10. 약어 모음

| 약어 | 정식 명칭 | 설명 |
|------|----------|------|
| **ATPCO** | Airline Tariff Publishing Company | 항공 운임 데이터 제공 기관 |
| **AP** | Advance Purchase | 사전 구매/예약 기한 |
| **CAT** | Category | ATPCO Rule Category |
| **FBC** | Fare Basis Code | 운임 기준 코드 |
| **GI** | Global Indicator | 글로벌 방향 지시자 (AT, PA, AP 등) |
| **HIP** | Higher Intermediate Point | 중간 지점 초과 요금 검사 |
| **MNS** | Minimum Stay | 최소 체류 기간 |
| **MXS** | Maximum Stay | 최대 체류 기간 |
| **NUC** | Neutral Unit of Construction | 국제 운임 계산 기준 통화 단위 |
| **OW** | One Way | 편도 |
| **PEN** | Penalty | 변경/취소 수수료 |
| **PTC** | Passenger Type Code | 승객 유형 코드 (ADT, CHD, INF 등) |
| **RBD** | Reservation Booking Designator | 예약 클래스 코드 |
| **RL** | Rule List | 규정 목록 조회 (1차 호출) |
| **RD** | Rule Display | 규정 전문 조회 (2차 호출) |
| **RT** | Round Trip | 왕복 |
| **STP** | Stopover | 경유 (24시간 이상 체류) |
| **TPM** | Ticketed Point Mileage | 발권 지점 간 마일리지 |

---

## 참고

- [WBS Integration Flow - Step 5](amadeus-wbs-integration-flow.md)
- [Fare_GetFareRules 용어집](fare-get-fare-rules.md)
- [MiniRule_GetFromRec 용어집](minirule-get-from-rec.md)
- [Fare_InformativePricingWithoutPNR 용어집](fare-informative-pricing.md)
