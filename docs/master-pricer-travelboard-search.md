# Master Pricer Travelboard Search (MPTBS)

Amadeus의 항공편 검색 및 운임 조회 API인 `Fare_MasterPricerTravelBoardSearch`의 기술 용어를 정리한 문서입니다.

> 기반 문서: Fare_MasterPricerTravelBoardSearch 24.6 Technical Reference

---

!!! info "WBS Integration Flow Step 2"
    이 API는 [WBS Integration Flow](amadeus-wbs-integration-flow.md)의 **검색 단계 Step 2**에 해당합니다.

---

## 1. 개요

### Master Pricer Travelboard Search

Amadeus GDS가 제공하는 **항공편 검색 + 최저가 운임 조회 API**. 출발지/도착지, 날짜, 승객 유형 등 조건을 입력하면 항공편 조합(Recommendation)과 운임 정보를 반환한다.

```
[Query: 검색 요청]
  출발지/도착지, 날짜, 승객 수, 옵션
       │
       ▼
  Amadeus Fare Engine
       │
       ▼
[Reply: 검색 결과]
  Recommendation × N개
  ├─ 항공편 조합 (Flight Option)
  ├─ 운임 (Fare / Price)
  ├─ Fare Family 정보
  ├─ Mini Rules (변경/환불 규정 요약)
  ├─ 수하물 허용량
  └─ 부가서비스 (Ancillary / EMD)
```

### Query / Reply 구조

| 구분 | 메시지 | 설명 |
|------|--------|------|
| **Query** | `Fare_MasterPricerTravelBoardSearch` | 검색 조건 입력 (요청) |
| **Reply** | `Fare_MasterPricerTravelBoardSearchReply` | 검색 결과 반환 (응답) |

| 항목 | 내용 |
|------|------|
| API 명 | `Fare_MasterPricerTravelBoardSearch` |
| 플로우 단계 | Step 2 -- 검색 |
| 목적 | 확정 날짜의 항공편 + 운임 조합 상세 검색 |
| 이전 단계 | [Fare_MasterPricerCalendar](fare-master-pricer-calendar.md) |
| 다음 단계 | [Fare_InformativeBestPricingWithoutPNR](fare-informative-best-pricing.md) |

---

## 2. Query 주요 구조 (검색 요청)

### numberOfUnit

검색 결과로 받을 **좌석 수와 Recommendation 수**를 지정하는 구조.

| 필드 | 설명 |
|------|------|
| `unitNumberDetail.numberOfUnits` | 요청할 Recommendation 수 또는 좌석 수 |
| `unitNumberDetail.typeOfUnit` | 단위 유형 (RC = Recommendation, PX = Passenger) |

### paxReference (Traveller Information)

**승객 정보**를 정의하는 구조. 최대 9개 그룹까지 지정 가능.

| 필드 | 설명 |
|------|------|
| `ptc` | Passenger Type Code. 승객 유형 코드 |
| `traveller.ref` | 승객 참조 번호 (1~9) |
| `traveller.infantIndicator` | 유아 동반 여부 |

#### PTC (Passenger Type Code)

승객의 유형을 나타내는 코드. 운임 산정의 기준이 된다.

| 코드 | 설명 |
|------|------|
| **ADT** | Adult, 성인 (12세 이상) |
| **CHD** | Child, 소아 (2~11세) |
| **INF** | Infant, 유아 (2세 미만, 좌석 미사용) |
| **INS** | Infant with Seat, 유아 (좌석 사용) |
| **YTH** | Youth, 청소년 할인 |
| **STU** | Student, 학생 할인 |
| **SRC** | Senior Citizen, 경로 할인 |

### itinerary (Origin and Destination)

**여정 정보**를 정의하는 구조. 출발지, 도착지, 날짜 등.

| 필드 | 설명 |
|------|------|
| `requestedSegmentRef.segRef` | 구간 번호 (1, 2, 3...) |
| `departureLocalization.departurePoint.locationId` | 출발지 공항/도시 코드 (IATA 3자리) |
| `arrivalLocalization.arrivalPointDetails.locationId` | 도착지 공항/도시 코드 |
| `timeDetails.firstDateTimeDetail.date` | 출발 희망 날짜 |
| `airportCityQualifier` | 공항(A) 또는 도시(C) 구분 |

### fareOptions (Fare Options)

**운임 검색 옵션**을 설정하는 구조.

| 필드 | 설명 |
|------|------|
| `pricingTickInfo.priceType` | 운임 유형 지정 (Published, Unifares, Corporate 등) |
| `corporate.corporateId` | Corporate Fare용 계약 코드 |
| `conversionRate.conversionRateDetail.currency` | 통화 강제 지정 |
| `formOfPayment` | 결제 수단 정보 |
| `frequentTravellerInfo` | 마일리지 프로그램 정보 |
| `monetaryCabinInfo` | 캐빈 클래스별 예산 상한 |
| `priceToBeat` | Price to Beat - 이 가격보다 저렴한 결과만 반환 |

#### priceType 주요 값

| 코드 | 설명 |
|------|------|
| **RP** | Published Fare, 공시 운임 |
| **RU** | Unifares, 통합 운임 |
| **RC** | Corporate Fare, 기업 계약 운임 |
| **RW** | Lowest Fare, 최저가 |
| **ET** | Electronic Ticket 가능 운임만 |
| **TAC** | Ticket After Confirmation, 확인 후 발권 |
| **NF** | No Fee, 수수료 없는 운임 |

### solutionFamily (Fare Family 검색)

**Fare Family(브랜드 운임)** 기준으로 검색할 때 사용하는 구조.

| 필드 | 설명 |
|------|------|
| `familyInformation.fareFamilyname` | Fare Family 이름 (예: LIGHT, STANDARD, FLEX) |
| `familyInformation.hierarchy` | Fare Family 계층 순서 |
| `commercialFamilyDetails.commercialFamily` | 상업용 Fare Family 이름 |

### fareFamilies (Fare Family Criteria)

특정 **Fare Family 속성**으로 필터링할 때 사용하는 구조.

| 필드 | 설명 |
|------|------|
| `familyCriteria.carrierId` | 항공사 코드 |
| `familyCriteria.rdb` | RBD (Reservation Booking Designator) |
| `familyCriteria.cabinProduct.cabinDesignator` | 캐빈 클래스 지정 |
| `familyCriteria.fareProductDetail.fareBasis` | Fare Basis Code |
| `familyCriteria.fareProductDetail.fareType` | 운임 유형 (Public, Private 등) |

### searchOptions (검색 옵션 속성)

검색 동작을 제어하는 **속성(Attribute) 목록**. 최대 100개 지정 가능.

주요 속성 예시:

| 속성 | 설명 |
|------|------|
| `FLYDAY` | 요일 기반 검색 |
| `PERIOD` | 기간 범위 검색 |
| `MAX` | 최대값 제한 |

### carbonEmissionBySourceDetails

**탄소 배출량** 계산 엔진과 방법론을 지정하는 구조. 운송 수단별로 다른 탄소 배출 제공자를 설정할 수 있다.

| 필드 | 설명 |
|------|------|
| `carbonEmissionProviderCd` | 탄소 배출 데이터 제공자 코드 |
| `computationMethod` | 배출량 계산 방법론 |
| `transportationModeType` | 운송 수단 유형 (항공, 철도 등) |

---

## 3. Reply 주요 구조 (검색 결과)

### Recommendation

검색 결과의 **하나의 여정 조합 + 운임**을 나타내는 단위. 하나의 Recommendation은 항공편 조합, 가격, Fare Family 정보를 포함한다.

```
Recommendation #1
├─ recPriceInfo: 총 운임 528,400원
│   ├─ monetaryDetail: Base Fare 450,000
│   └─ monetaryDetail: Total Tax 78,400
├─ segmentFlightRef: 항공편 참조
│   └─ referencingDetail: Flight Option #1 → KE001
├─ paxFareProduct: 승객별 운임 상세
│   ├─ fareDetails: Fare Basis, RBD
│   ├─ fare: 운임 금액
│   └─ taxInformation: 세금 상세
└─ specificRecDetails: 추가 상세 정보
```

### flightDetails (항공편 상세)

**개별 항공편 구간(Segment)**의 상세 정보.

| 필드 | 설명 |
|------|------|
| `flightInformation.productDateTime.dateOfDeparture` | 출발일 |
| `flightInformation.productDateTime.timeOfDeparture` | 출발 시간 |
| `flightInformation.productDateTime.dateOfArrival` | 도착일 |
| `flightInformation.productDateTime.timeOfArrival` | 도착 시간 |
| `flightInformation.productDateTime.dateVariation` | 날짜 차이 (0=당일, 1=+1일) |
| `flightInformation.location[0].locationId` | 출발 공항 코드 |
| `flightInformation.location[1].locationId` | 도착 공항 코드 |
| `flightInformation.location.terminal` | 터미널 정보 |
| `flightInformation.companyId.marketingCarrier` | 마케팅 항공사 (판매 항공사) |
| `flightInformation.companyId.operatingCarrier` | 운항 항공사 (실제 운항) |
| `flightInformation.flightOrtrainNumber` | 편명 번호 |
| `flightInformation.productDetail.equipmentType` | 기재 유형 (항공기 기종 코드) |
| `flightInformation.productDetail.operatingDay` | 운항 요일 |
| `flightInformation.productDetail.techStopNumber` | 기술 착륙 횟수 |
| `flightInformation.addProductDetail.electronicTicketing` | 전자 발권 가능 여부 |
| `flightInformation.addProductDetail.lastSeatAvailable` | LSA (Last Seat Available) 여부 |

#### Marketing Carrier vs Operating Carrier

| 구분 | 설명 | 예시 |
|------|------|------|
| **Marketing Carrier** | 항공권을 **판매**하는 항공사. 편명을 소유 | DL (델타항공) |
| **Operating Carrier** | 실제 **운항**하는 항공사 | KE (대한항공) |

코드셰어(Codeshare) 항공편에서 두 값이 다르다. 예: DL9000편으로 판매되지만 실제 운항은 KE가 수행.

### cabinProduct (캐빈 정보)

항공편의 **캐빈 클래스와 좌석 등급** 정보.

| 필드 | 설명 |
|------|------|
| `rbd` | Reservation Booking Designator (예: Y, M, H) |
| `cabin` | 캐빈 클래스 코드 |
| `avlStatus` | 좌석 가용 상태 (Posting Level) |
| `bookingModifier` | 예약 변경자 |

#### Cabin Class 코드

| 코드 | 설명 |
|------|------|
| **F** | First Class (일등석) |
| **C** | Business Class (비즈니스석) |
| **W** | Premium Economy (프리미엄 이코노미) |
| **Y** | Economy Class (이코노미석) |

### Codeshare Details (공동운항 상세)

**코드셰어 계약** 정보를 나타내는 구조.

| 필드 | 설명 |
|------|------|
| `codeshareDetails.codeShareType` | 코드셰어 유형 |
| `codeshareDetails.airlineDesignator` | 코드셰어 항공사 코드 |
| `codeshareDetails.flightNumber` | 코드셰어 편명 |
| `otherCodeshareDetails` | 추가 코드셰어 정보 (다중 코드셰어) |

### recPriceInfo (운임 정보)

Recommendation의 **가격 정보**.

| 필드 | 설명 |
|------|------|
| `monetaryDetail.amountType` | 금액 유형 |
| `monetaryDetail.amount` | 금액 |
| `monetaryDetail.currency` | 통화 코드 (ISO) |

#### amountType 주요 값

| 코드 | 설명 |
|------|------|
| **B** | Base Fare (기본 운임) |
| **T** | Total Amount (총액) |
| **E** | Equivalent Amount (환산 금액) |
| **TAX** | Tax (세금) |

### paxFareProduct (승객별 운임)

**승객 유형별 상세 운임** 정보.

| 필드 | 설명 |
|------|------|
| `paxFareDetail.totalFareAmount` | 총 운임 |
| `paxFareDetail.totalTaxAmount` | 총 세금 |
| `fareDetails.groupOfFares.productInformation.fareProductDetail.fareBasis` | Fare Basis Code |
| `fareDetails.groupOfFares.productInformation.fareProductDetail.fareType` | 운임 유형 |
| `fareDetails.groupOfFares.productInformation.cabinProduct.rbd` | RBD |
| `fareDetails.groupOfFares.productInformation.cabinProduct.cabin` | 캐빈 |

---

## 4. Mini Rules

운임의 **변경/환불 규정 요약 정보**. 전체 Fare Rule을 조회하지 않고도 핵심 규정을 빠르게 확인할 수 있다.

```
Mini Rules 구조:
  mnrGrp
  └─ mnrDetails (최대 999개)
      ├─ mnrRef: 규정 참조 번호
      ├─ dateInfo: 적용 기간
      └─ catGrp: 카테고리별 규정
          ├─ catInfo: 카테고리 정보 (변경/환불 등)
          ├─ monInfo: 수수료 금액
          └─ mnrTimeBoundPenalties: 시간대별 패널티
```

### 주요 카테고리

| 카테고리 | 설명 |
|---------|------|
| **PTC** | Passenger Type Code, 적용 승객 유형 |
| **PEN** | Penalty, 변경/취소 수수료 |
| **ADV** | Advance Purchase/Reservation, 사전 구매/예약 기한 |
| **MNS** | Minimum Stay, 최소 체류 기간 |
| **MXS** | Maximum Stay, 최대 체류 기간 |
| **SUR** | Surcharge, 추가 요금 |

---

## 5. Fare Family (브랜드 운임)

항공사가 **부가서비스 포함 여부**에 따라 운임을 등급화한 상품 체계. 같은 이코노미석이라도 수하물, 좌석 선택, 변경 가능 여부 등이 다르다.

```
Fare Family 계층 예시 (대한항공):

  LIGHT        수하물 미포함, 변경 불가, 환불 불가
  STANDARD     수하물 포함, 유료 변경, 유료 환불
  FLEX         수하물 포함, 무료 변경, 유료 환불
  PRESTIGE     수하물 포함, 무료 변경, 무료 환불
```

### 관련 구조

| 필드 | 설명 |
|------|------|
| `fareFamilyname` | Fare Family 이름 (Short Name) |
| `hierarchy` | 계층 순서 (높을수록 상위 상품) |
| `commercialFamily` | 상업용 Fare Family 이름 |
| `refNumber` | Fare Family 참조 번호 |

---

## 6. Virtual Interlining

서로 **인터라인 계약이 없는 항공사 간 여정**을 하나의 검색 결과로 조합하는 기능. 전통적인 Interline과 달리 별도 항공권을 각각 발권하되, 하나의 여정으로 제안한다.

```
전통 Interline:
  ICN → NRT (KE) + NRT → LAX (NH) = 하나의 항공권, 수하물 연결

Virtual Interlining:
  ICN → NRT (제주항공) + NRT → LAX (Peach) = 별도 항공권 2매
  → 환승 보장 없음, 수하물 재체크인 필요
  → 하지만 훨씬 저렴할 수 있음
```

### 관련 구조

| 필드 | 설명 |
|------|------|
| `virtualInterlining.itinerary` | 가상 인터라인 여정 |
| `virtualInterlining.recommendation` | 가상 인터라인 추천 조합 |
| `combinabilityIds` | 조합 가능 그룹 ID |

---

## 7. Offer / Offer Item (NDC)

**NDC (New Distribution Capability)** 흐름에서 반환되는 상품 단위.

| 용어 | 설명 |
|------|------|
| **Offer** | 하나의 가격 제안. 항공편 + 운임 + 부가서비스의 묶음 |
| **Offer Item** | Offer를 구성하는 개별 항목 (좌석, 수하물 등) |
| **Time Limit** | Offer의 유효 기한 (이 시간 내 구매 필요) |

```
Offer #1
├─ offerDetails: Offer 식별 정보
├─ timeLimits: 유효 기한
└─ offerItems
    ├─ Offer Item #1: 항공편 좌석
    ├─ Offer Item #2: 위탁 수하물 23kg
    └─ Offer Item #3: 좌석 선택
```

---

## 8. EMD (Electronic Miscellaneous Document)

항공권 외의 **부가 서비스 전표**. 수하물 추가, 좌석 업그레이드, 기내식 선택 등 Ancillary 서비스의 결제·정산에 사용된다.

### MPTBS 내 EMD 구조

| 필드 | 설명 |
|------|------|
| `emdReference` | EMD 참조 정보 (쿠폰 정보) |
| `emdPaxReference` | EMD 적용 승객 |
| `emdAmounts` | EMD 금액 (총액, 세금) |
| `emdRecommendation` | EMD 추천 조합 |

### RFIC (Reason For Issuance Code)

EMD 발급 사유를 나타내는 코드.

| 코드 | 설명 |
|------|------|
| **A** | Air Transportation |
| **B** | Surface Transportation / Non-Air Services |
| **C** | Baggage (수하물) |
| **D** | Financial Impact (재무 영향) |
| **E** | Airport Services |

---

## 9. Free Baggage Allowance

**무료 수하물 허용량** 정보. Fare Family별로 다르게 적용된다.

| 필드 | 설명 |
|------|------|
| `freeBagAllownceInfo` | 무료 수하물 허용 정보 |
| `itemNumberInfo` | 허용 개수 또는 무게 |

```
수하물 허용 예시:
  LIGHT:    위탁 수하물 없음 (기내 7kg만)
  STANDARD: 위탁 수하물 1PC (23kg)
  FLEX:     위탁 수하물 2PC (23kg × 2)
```

---

## 10. 기타 주요 용어

### Last Seat Available (LSA)

좌석이 단 한 자리만 남은 상태에서도 **해당 운임으로 예약 가능**한지 여부. Y/N 값으로 표시.

### Electronic Ticketing

해당 항공편/운임이 **전자 발권(e-Ticket)** 가능한지 여부. Y/N 값으로 표시. 거의 모든 현대 항공편은 전자 발권 가능.

### Date Variation

출발일 대비 **도착일의 차이**. 야간 비행 또는 시차로 인해 도착일이 다를 때 사용.

| 값 | 의미 |
|----|------|
| 0 | 당일 도착 |
| 1 | 익일 도착 (+1일) |
| 2 | +2일 도착 |

### Equipment Type

**항공기 기종 코드**. IATA 표준 3자리 코드.

| 코드 | 기종 |
|------|------|
| **388** | Airbus A380 |
| **359** | Airbus A350-900 |
| **789** | Boeing 787-9 |
| **77W** | Boeing 777-300ER |
| **321** | Airbus A321 |
| **738** | Boeing 737-800 |

### Tech Stop

여객이 탑승/하차하지 않는 **기술 착륙**. 연료 보급 등의 목적으로 경유하는 공항. 검색 결과에서 경유지와 기술 착륙은 구분하여 표시된다.

### Miles Accrual

항공편 이용 시 적립되는 **마일리지 정보**. 프로그램별 적립률이 다를 수 있다.

| 필드 | 설명 |
|------|------|
| `milesAccrualId` | 마일리지 프로그램 식별자 |
| `milesAccrualDetails` | 적립 상세 (적립률, 프로그램 코드) |

### Carbon Emission (탄소 배출)

항공편의 **탄소 배출량 추정치**. ATPCO 등 외부 제공자의 데이터를 기반으로 계산.

| 필드 | 설명 |
|------|------|
| `referenceCarbonEmission` | 기준 탄소 배출량 (전체 여정) |
| `estimatedCarbonEmission` | 구간별 추정 탄소 배출량 |

---

## 11. 메시지 구조 용어

MPTBS 기술 문서에서 사용되는 **메시지 구조 정의 용어**.

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
| `an..35` | 가변 영숫자 최대 35자리 | `HONG/GILDONG MR` |
| `n..18` | 가변 숫자 최대 18자리 | `528400` (금액) |

---

## 약어 모음

| 약어 | 정식 명칭 | 설명 |
|------|----------|------|
| **MPTBS** | Master Pricer Travelboard Search | Amadeus 항공편 검색 API |
| **PTC** | Passenger Type Code | 승객 유형 코드 |
| **RBD** | Reservation Booking Designator | 예약 클래스 |
| **LSA** | Last Seat Available | 마지막 좌석 가용 여부 |
| **EMD** | Electronic Miscellaneous Document | 부가서비스 전표 |
| **RFIC** | Reason For Issuance Code | EMD 발급 사유 코드 |
| **NDC** | New Distribution Capability | 항공사 직접 연결 표준 |
| **ATPCO** | Airline Tariff Publishing Company | 항공 운임 데이터 제공 기관 |
| **FOP** | Form of Payment | 결제 수단 |
| **MNR** | Mini Rules | 운임 규정 요약 |

---

## 참고

- [WBS Integration Flow - Step 2](amadeus-wbs-integration-flow.md)
- [Fare_MasterPricerCalendar 용어집](fare-master-pricer-calendar.md)
- [Fare_InformativeBestPricingWithoutPNR 용어집](fare-informative-best-pricing.md)
- [Fare_InformativePricingWithoutPNR 용어집](fare-informative-pricing.md)
