# Fare_MasterPricerCalendar

Amadeus의 **날짜 유연 검색 API**인 `Fare_MasterPricerCalendar`의 기술 용어를 정리한 문서입니다. 기준 날짜 전후 범위의 최저가를 달력 형태로 탐색한다.

> 기반 문서: Fare_MasterPricerCalendar 20.2 Technical Reference

---

!!! info "WBS Integration Flow Step 1"
    이 API는 [WBS Integration Flow](amadeus-wbs-integration-flow.md)의 **검색 단계 Step 1**에 해당합니다.

---

## 1. 개요

### Fare_MasterPricerCalendar

Amadeus GDS가 제공하는 **달력 기반 최저가 탐색 API**. [Fare_MasterPricerTravelBoardSearch](master-pricer-travelboard-search.md)(MPTBS)와 동일한 메시지 구조를 사용하되, **날짜 범위(dayInterval)와 여행 기간 유연성(tripInterval)**을 지정하여 전후 날짜별 최저가를 달력 형태로 반환한다.

```
[Query: 달력 검색 요청]
  출발지/도착지, 기준 날짜, 날짜 범위(±N일), 승객 수
       |
       v
  Amadeus Fare Engine
       |
       v
[Reply: 날짜별 최저가]
  날짜별 Recommendation x N개
  +-- 각 날짜의 최저가 운임
  +-- 항공편 조합 (Flight Option)
  +-- Fare Family 정보
  +-- Mini Rules (변경/환불 규정 요약)
  +-- 수하물 허용량
  +-- 부가서비스 (Ancillary / EMD)
```

### MPTBS와의 차이점

| 항목 | MasterPricerCalendar | MasterPricerTravelBoardSearch |
|------|---------------------|-------------------------------|
| **목적** | 날짜 범위별 최저가 달력 탐색 | 특정 날짜의 상세 검색 결과 |
| **핵심 필드** | `dayInterval`, `rangeQualifier`, `tripInterval` | `firstDateTimeDetail.date` |
| **결과 형태** | 날짜별 최저가 목록 (달력 뷰) | 상세 Recommendation 목록 |
| **사용 시점** | Step 1 - 날짜 선택 전 탐색 | Step 1 - 날짜 확정 후 상세 검색 |
| **일반 흐름** | Calendar -> MPTBS -> 예약 | MPTBS -> 예약 |

### Query / Reply 구조

| 구분 | 메시지 | 버전 | 설명 |
|------|--------|------|------|
| **Query** | `Fare_MasterPricerCalendar` | 20.2.1A | 달력 검색 조건 입력 (요청) |
| **Reply** | `Fare_MasterPricerCalendarReply` | 20.2.1A | 날짜별 최저가 반환 (응답) |

| 항목 | 내용 |
|------|------|
| API 명 | `Fare_MasterPricerCalendar` |
| 플로우 단계 | Step 1 -- 검색 |
| 목적 | 기준 날짜 전후 범위의 최저가 달력 탐색 |
| 다음 단계 | [Fare_MasterPricerTravelBoardSearch](master-pricer-travelboard-search.md) |

---

## 2. Query 주요 구조 (검색 요청)

### numberOfUnit

검색 결과로 받을 **좌석 수와 Recommendation 수**를 지정하는 구조.

| 필드 | 설명 |
|------|------|
| `unitNumberDetail.numberOfUnits` | 요청할 Recommendation 수 또는 좌석 수 |
| `unitNumberDetail.typeOfUnit` | 단위 유형 (RC = Recommendation, PX = Passenger) |

### globalOptions

검색의 **전역 옵션**을 지정하는 속성(Attribute) 구조.

| 필드 | 설명 |
|------|------|
| `attributeDetails.attributeType` | 속성 유형 |
| `attributeDetails.attributeDescription` | 속성 설명 |

### paxReference (Traveller Information)

**승객 정보**를 정의하는 구조. 최대 9개 그룹까지 지정 가능.

| 필드 | 설명 |
|------|------|
| `ptc` | Passenger Type Code. 승객 유형 코드 |
| `traveller.ref` | 승객 참조 번호 (1~9) |
| `traveller.infantIndicator` | 유아 동반 여부 (1 = Infant) |

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

### passengerRange

**승객 범위**를 지정하는 구조. 승객 수와 범위를 정의한다.

| 필드 | 설명 |
|------|------|
| `segmentControlDetails.quantity` | 승객 수 |
| `segmentControlDetails.numberOfUnits` | 승객 범위 |

### itinerary (Origin and Destination)

**여정 정보**를 정의하는 구조. 출발지, 도착지, 날짜 등을 포함하며, 달력 검색의 핵심인 **timeDetails**를 통해 날짜 범위를 지정한다. 최대 18개 여정 지정 가능.

| 필드 | 설명 |
|------|------|
| `requestedSegmentRef.segRef` | 구간 번호 (1, 2, 3...) |
| `departureLocalization.departurePoint.locationId` | 출발지 공항/도시 코드 (IATA 3자리) |
| `arrivalLocalization.arrivalPointDetails.locationId` | 도착지 공항/도시 코드 |
| `timeDetails` | **날짜/시간 상세 (달력 검색 핵심 구조)** |
| `flightInfo` | 항공편 옵션 (직항, 경유 등) |
| `familyInformation` | 구간별 Fare Family 지정 |

### fareOptions (Fare Options)

**운임 검색 옵션**을 설정하는 구조.

| 필드 | 설명 |
|------|------|
| `pricingTickInfo.priceType` | 운임 유형 지정 (Published, Unifares, Corporate 등) |
| `corporate.corporateId` | Corporate Fare용 계약 코드 |
| `ticketingPriceScheme.referenceNumber` | PSR (Price Scheme Reference) 번호 |
| `conversionRate.conversionRateDetail.currency` | 통화 강제 지정 |
| `formOfPayment` | 결제 수단 정보 |
| `frequentTravellerInfo` | 마일리지 프로그램 정보 |
| `monetaryCabinInfo` | 캐빈 클래스별 예산 상한 |
| `priceToBeat` | Price to Beat - 이 가격보다 저렴한 결과만 반환 |
| `travelFlightInfo` | 항공편 상세 옵션 (직항, 경유, 항공사 지정 등) |

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

특정 **Fare Family 속성**으로 필터링할 때 사용하는 구조. 최대 20개 지정 가능.

| 필드 | 설명 |
|------|------|
| `familyInformation` | Fare Family 대상 구간 지정 |
| `familyCriteria.carrierId` | 항공사 코드 |
| `familyCriteria.rdb` | RBD (Reservation Booking Designator) |
| `familyCriteria.cabinProduct.cabinDesignator` | 캐빈 클래스 지정 |
| `familyCriteria.fareProductDetail.fareBasis` | Fare Basis Code |
| `familyCriteria.fareProductDetail.fareType` | 운임 유형 (Public, Private 등) |

### searchOptions (검색 옵션 속성)

검색 동작을 제어하는 **속성(Attribute) 목록**. 최대 10개 지정 가능.

| 속성 | 설명 |
|------|------|
| `FLYDAY` | 요일 기반 검색 |
| `PERIOD` | 기간 범위 검색 |
| `MAX` | 최대값 제한 |

### buckets

검색 결과를 **버킷(그룹) 단위**로 분류하기 위한 구조. 최대 10개.

| 필드 | 설명 |
|------|------|
| `bucketInfo.number` | 버킷 번호 |
| `bucketInfo.name` | 버킷 이름 |
| `bucketInfo.completion` | 완성도 |
| `bucketInfo.mode` | 모드 |
| `bucketInfo.weight` | 가중치 |
| `bucketInfo.count` | 솔루션 수 |
| `bucketDetails` | 버킷 상세 조건 (최대 15개) |

### officeIdDetails

**Office ID 정보**를 지정하는 구조. 최대 20개.

| 필드 | 설명 |
|------|------|
| `officeIdInformation` | Office ID 식별 정보 |
| `nbOfUnits` | 단위 수 |
| `uidOption` | UID 옵션 |
| `pricingTickInfo` | 가격/발권 정보 |
| `corporateFareInfo` | 기업 운임 정보 |
| `travelFlightInfo` | 항공편 정보 |

### feeOption

**수수료 옵션** 구조. 최대 9개.

| 필드 | 설명 |
|------|------|
| `feeTypeInfo` | 수수료 유형 (OB, OC 등) |
| `rateTax` | 관련 세율 |
| `feeDetails.feeInfo` | 수수료 상세 정보 |
| `feeDetails.associatedAmounts` | 관련 금액 |

### ndcQueryParameters

**NDC (New Distribution Capability)** 흐름의 특수 파라미터.

| 필드 | 설명 |
|------|------|
| `ndcMarker` | NDC Item Count (Offer) |
| `ndcSpecialParameters` | NDC 특수 파라미터 (로열티, 운임, PTC 오버라이드 등) |

### combinationFareFamilies

**Fare Family 조합** 지정. 최대 2000개.

| 필드 | 설명 |
|------|------|
| `itemFFCNumber` | 아이템 번호 |
| `nbOfUnits` | 단위 수 |
| `referenceInfo` | 구간 참조 정보 (최대 6개) |

---

## 3. Reply 주요 구조 (검색 결과)

### replyStatus

응답의 **상태 정보**. 프로세스 유형, 리전, CPU 등.

| 필드 | 설명 |
|------|------|
| `statusInformation.indicator` | 상태 표시자 (Historical/Current) |
| `statusInformation.action` | 액션 코드 |

### errorMessage

**오류 메시지** 그룹.

| 필드 | 설명 |
|------|------|
| `applicationError` | 애플리케이션 오류 상세 |
| `errorMessageText` | 오류 메시지 텍스트 |

### conversionRate

응답에서 사용된 **통화 변환율** 정보.

| 필드 | 설명 |
|------|------|
| `conversionRateDetail.conversionType` | 변환 유형 |
| `conversionRateDetail.currency` | 통화 코드 (ISO) |

### amountInfoForAllPax

**전체 승객 합산 금액** 정보.

| 필드 | 설명 |
|------|------|
| `itineraryAmounts` | 여정별 금액 |
| `amountsPerSgt.sgtRef` | 구간 참조 |
| `amountsPerSgt.amounts` | 구간별 금액 (총액, 세금, 환불 불가 세금) |

### amountInfoPerPax

**승객별 금액** 정보. 최대 20개.

| 필드 | 설명 |
|------|------|
| `paxRef` | 승객 참조 |
| `paxAttributes` | 승객 속성 (유아 표시자 등) |
| `itineraryAmounts` | 여정별 금액 |
| `amountsPerSgt.sgtRef` | 구간 참조 |
| `amountsPerSgt.amounts` | 구간별 금액 |

### flightIndex

**항공편 인덱스** 구조. 구간별 항공편 목록을 담는다. 최대 6개 구간.

```
flightIndex (구간별)
+-- requestedSegmentRef: 구간 참조
+-- groupOfFlights (최대 100000개)
    +-- propFlightGrDetail: 제안 항공편 그룹 상세
    +-- flightDetails (최대 4개): 개별 항공편 정보
        +-- flightInformation: 항공편 상세 (출발/도착, 편명, 기재)
        +-- avlInfo: 예약 클래스 및 가용성
        +-- technicalStop: 기술 착륙 정보
        +-- commercialAgreement: 코드셰어 계약
        +-- addInfo: 추가 정보
        +-- flightCharacteristics: 항공편 특성
        +-- flightServices: 캐빈별 서비스
        +-- mealServices: 기내식 서비스
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

### recommendation (날짜별 추천 운임)

달력 검색 결과의 **하나의 여정 조합 + 운임**을 나타내는 단위. 각 날짜에 대해 최저가 Recommendation이 반환된다.

```
Recommendation #1 (예: 3월 15일 출발)
+-- itemNumber: 추천 번호
+-- recPriceInfo: 총 운임 528,400원
|   +-- monetaryDetail: Base Fare 450,000
|   +-- monetaryDetail: Total Tax 78,400
+-- segmentFlightRef: 항공편 참조
|   +-- referencingDetail: Flight Option #1 -> KE001
+-- miniRule: 변경/환불 규정 요약
+-- paxFareProduct: 승객별 운임 상세
|   +-- paxFareDetail: Fare Basis, 금액
|   +-- fare: 운임 상세 (Fare Basis, 패널티, 발권 기한)
|   +-- fareDetails: 구간별 운임 (RBD, 캐빈, Fare Family)
+-- specificRecDetails: 추가 상세 정보
```

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

**승객 유형별 상세 운임** 정보. 최대 10개.

| 필드 | 설명 |
|------|------|
| `paxFareDetail.paxFareNum` | 승객 운임 번호 |
| `paxReference` | 승객 참조 |
| `passengerTaxDetails` | 승객별 세금 상세 |
| `fare.pricingMessage` | Last Date to Ticket, 패널티 정보 |
| `fare.monetaryInformation` | 패널티/할증 금액 |
| `fareDetails.segmentRef` | 구간 참조 |
| `fareDetails.groupOfFares.productInformation.fareProductDetail.fareBasis` | Fare Basis Code |
| `fareDetails.groupOfFares.productInformation.fareProductDetail.fareType` | 운임 유형 |
| `fareDetails.groupOfFares.productInformation.cabinProduct.rbd` | RBD |
| `fareDetails.groupOfFares.productInformation.cabinProduct.cabin` | 캐빈 클래스 |
| `fareDetails.groupOfFares.productInformation.cabinProduct.avlStatus` | 좌석 가용 상태 |

### otherSolutions

**추가 솔루션** (철도 등 대체 교통수단 포함). 최대 100009개.

| 필드 | 설명 |
|------|------|
| `reference` | 솔루션 참조 (Sequence Details) |
| `amtGroup.ref` | 금액 참조 (구간별, 날짜별) |
| `amtGroup.amount` | 금액 상세 |
| `psgInfo` | 승객 관련 정보 (할인카드, PTC, 운임, 금액 등) |

### serviceFeesGrp / serviceCoverageInfoGrp

**부가 서비스 수수료 및 커버리지** 정보 그룹.

| 필드 | 설명 |
|------|------|
| `serviceFeesGrp` | 서비스 수수료 그룹 |
| `serviceCoverageInfoGrp` | 서비스 커버리지 정보 |
| `serviceFeeInfoGrp` | 서비스 수수료 상세 |
| `serviceDetailsGrp` | 서비스 상세 그룹 |
| `freeBagAllowanceGrp` | 무료 수하물 허용량 |

---

## 4. 달력 검색 핵심 구조 (Calendar-Specific)

달력 검색에서 가장 중요한 구조는 `itinerary.timeDetails`의 **DATE AND TIME INFORMATION**이다. 이 구조를 통해 날짜 범위와 여행 기간 유연성을 지정한다.

### timeDetails (Date and Time Information)

여정의 **날짜, 시간, 날짜 범위, 여행 기간 유연성**을 정의하는 구조. 달력 검색의 핵심.

```
timeDetails
+-- firstDateTimeDetail        (기준 날짜/시간)
|   +-- timeQualifier          (날짜 구분자: 출발/도착)
|   +-- date                   (기준 날짜: DDMMYY)
|   +-- time                   (기준 시간: HHMM)
|   +-- timeWindow             (시간 윈도우: 시간 단위)
|
+-- rangeOfDate                (날짜 범위 - 달력 핵심)
|   +-- rangeQualifier         (범위 유형: Plus/Minus/Combined)
|   +-- dayInterval            (전후 일수: +-N일)
|   +-- timeAtdestination      (도착지 현지 시간)
|
+-- tripDetails                (여행 기간 유연성)
    +-- flexibilityQualifier   (유연성 유형: Plus/Minus/Combined)
    +-- tripInterval           (전후 일수: +-N일)
    +-- tripDuration           (여행 기간: 일수)
```

### firstDateTimeDetail

**기준 날짜와 시간**을 지정하는 구조.

| 필드 | 형식 | 설명 |
|------|------|------|
| `timeQualifier` | an..3 | 날짜/시간 구분자. Codeset 2005 IA 97.2.105 |
| `date` | n6 | 기준 날짜 (DDMMYY 형식) |
| `time` | n4 | 기준 시간 (HHMM 형식) |
| `timeWindow` | an..3 | 시간 윈도우 크기 (시간 단위) |

### rangeOfDate (날짜 범위)

달력 검색의 **핵심 구조**. 기준 날짜 전후로 몇 일 범위를 탐색할지 지정한다.

| 필드 | 형식 | 설명 |
|------|------|------|
| `rangeQualifier` | an..3 | 범위 유형 (Plus, Minus, Combined). Codeset 2005 IA 97.2.105 |
| `dayInterval` | n..6 | **기준 날짜 전후 탐색 일수**. 예: 3이면 전후 3일 |
| `timeAtdestination` | n4 | 도착지 현지 시간 (HHMM 형식) |

#### rangeQualifier / flexibilityQualifier 코드 (Codeset 2005 IA 97.2.105)

| 코드 | 설명 | 날짜 범위 예시 (기준일: 3/15, dayInterval: 3) |
|------|------|----------------------------------------------|
| **C** | Minus and Plus Combined (전후 결합) | 3/12 ~ 3/18 (전후 3일) |
| **M** | Minus (이전만) | 3/12 ~ 3/15 (이전 3일) |
| **P** | Plus (이후만) | 3/15 ~ 3/18 (이후 3일) |
| **TA** | Arrival by (도착 기준) | 해당 시간까지 도착 |
| **TD** | Depart from (출발 기준) | 해당 시간 이후 출발 |

### tripDetails (여행 기간 유연성)

**여행 기간(체류 일수)**의 유연성을 지정하는 구조. 왕복 검색 시 출발일~귀국일 간 기간을 유연하게 설정할 수 있다.

| 필드 | 형식 | 설명 |
|------|------|------|
| `flexibilityQualifier` | an..3 | 유연성 유형 (Plus, Minus, Combined). Codeset 2005 IA 97.2.105 |
| `tripInterval` | n..6 | **여행 기간 전후 유연 일수**. 예: 2이면 기간 +-2일 |
| `tripDuration` | n..4 | 출발일~도착일 사이 기간 (일수) |

### 달력 검색 흐름 예시

```
[검색 조건]
  출발: ICN -> NRT
  기준 출발일: 2024-03-15
  날짜 범위: rangeQualifier=C, dayInterval=3  (전후 3일)
  여행 기간: tripDuration=5, flexibilityQualifier=C, tripInterval=2 (5일 +-2일)

       |
       v

[Query - timeDetails 설정]
  firstDateTimeDetail:
    date = 150324 (DDMMYY)
  rangeOfDate:
    rangeQualifier = C (Combined, 전후)
    dayInterval = 3 (+-3일)
  tripDetails:
    flexibilityQualifier = C (Combined, 전후)
    tripInterval = 2 (+-2일)
    tripDuration = 5 (5일)

       |
       v

[Reply - 날짜별 최저가 결과]
  3/12 출발 (3~7일 체류): 최저가 328,000원
  3/13 출발 (3~7일 체류): 최저가 298,000원  <-- 최저
  3/14 출발 (3~7일 체류): 최저가 345,000원
  3/15 출발 (3~7일 체류): 최저가 412,000원
  3/16 출발 (3~7일 체류): 최저가 389,000원
  3/17 출발 (3~7일 체류): 최저가 356,000원
  3/18 출발 (3~7일 체류): 최저가 310,000원

       |
       v

[사용자가 3/13 선택]
  -> Fare_MasterPricerTravelBoardSearch 로 상세 검색
```

---

## 5. Mini Rules

운임의 **변경/환불 규정 요약 정보**. 전체 Fare Rule을 조회하지 않고도 핵심 규정을 빠르게 확인할 수 있다.

```
Mini Rules 구조:
  recommendation.miniRule (최대 9개)
  +-- category: 카테고리 정보
      +-- category: 제한 유형 (PTC, Max Adv Pur, Days 등)
      +-- code: ATPCO 컴포넌트 코드
      +-- processIndicator: 처리 표시자
```

### 주요 카테고리

| 카테고리 코드 | 설명 |
|-------------|------|
| **PTC** | Passenger Type Code, 적용 승객 유형 |
| **ADV** | Advance Purchase, 사전 구매 기한 |
| **MNS** | Minimum Stay, 최소 체류 기간 |
| **MXS** | Maximum Stay, 최대 체류 기간 |
| **PEN** | Penalty, 변경/취소 수수료 |
| **STP** | Stopover, 경유 제한 |
| **ELG** | Eligibility, 자격 제한 |

### ATPCO Category Code (Mini Rules 내)

| 코드 | 설명 |
|------|------|
| C05 | Advance Purchase Restrictions (사전 구매 제한) |
| C06 | Minimum Stay (최소 체류) |
| C07 | Maximum Stay (최대 체류) |
| C08 | Stopovers (경유) |
| C16 | Penalties (변경/취소 수수료) |

---

## 6. Fare Family (브랜드 운임)

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

### Fare qualifier 코드 (주요값)

| 코드 | 설명 |
|------|------|
| **ALT** | Alternate fare family (대체 Fare Family) |
| **NCO** | Non combinable fare family (결합 불가 Fare Family) |

---

## 7. Virtual Interlining

서로 **인터라인 계약이 없는 항공사 간 여정**을 하나의 검색 결과로 조합하는 기능.

```
전통 Interline:
  ICN -> NRT (KE) + NRT -> LAX (NH) = 하나의 항공권, 수하물 연결

Virtual Interlining:
  ICN -> NRT (제주항공) + NRT -> LAX (Peach) = 별도 항공권 2매
  -> 환승 보장 없음, 수하물 재체크인 필요
  -> 하지만 훨씬 저렴할 수 있음
```

### 관련 구조

| 필드 | 설명 |
|------|------|
| `identifier` | 식별자 |
| `totalAmount` | 세금 포함 총액 |
| `totalTaxes` | 총 세금 |
| `otherAmount` | 기타 금액 (최대 20개) |

---

## 8. Offer / Offer Item (NDC)

**NDC (New Distribution Capability)** 흐름에서 반환되는 상품 단위.

| 용어 | 설명 |
|------|------|
| **Offer** | 하나의 가격 제안. 항공편 + 운임 + 부가서비스의 묶음 |
| **Offer Item** | Offer를 구성하는 개별 항목 (좌석, 수하물 등) |
| **Time Limit** | Offer의 유효 기한 (이 시간 내 구매 필요) |

### Offer 구조

| 필드 | 설명 |
|------|------|
| `reference` | 참조 번호 |
| `offerId` | Offer 식별자 |
| `uniqueOfferReference` | 고유 Offer 참조 번호 |

### OfferItem 구조

| 필드 | 설명 |
|------|------|
| `offerItemId` | Offer Item 식별자 |
| `status` | 상태 코드 |

---

## 9. EMD (Electronic Miscellaneous Document)

항공권 외의 **부가 서비스 전표**. 수하물 추가, 좌석 업그레이드, 기내식 선택 등 Ancillary 서비스의 결제/정산에 사용된다.

### EMD 구조

| 필드 | 설명 |
|------|------|
| `emdReference` | EMD 참조 정보 (쿠폰 정보) |
| `emdPaxReference` | EMD 적용 승객 |
| `emdAmounts` | EMD 금액 (총액, 세금) |
| `emdRecommendation` | EMD 추천 조합 |
| `edmRecoId` | EMD Recommendation 식별자 |
| `emdRecoAmounts` | EMD Recommendation 금액 |

---

## 10. Free Baggage Allowance

**무료 수하물 허용량** 정보. Fare Family별로 다르게 적용된다.

| 필드 | 설명 |
|------|------|
| `baggageDetails.freeAllowance` | 허용 개수 또는 무게 |
| `baggageDetails.quantityCode` | 허용 유형 (개수 또는 무게) |
| `bagTagDetails.identifier` | 수하물 태그 식별자 |
| `bagTagDetails.number` | 수하물 수량 |

```
수하물 허용 예시:
  LIGHT:    위탁 수하물 없음 (기내 7kg만)
  STANDARD: 위탁 수하물 1PC (23kg)
  FLEX:     위탁 수하물 2PC (23kg x 2)
```

### Measure Unit Qualifier (수하물 단위)

| 코드 | 설명 |
|------|------|
| **K** | Kilograms (킬로그램) |
| **L** | Pounds (파운드) |

---

## 11. 기타 주요 용어

### Marketing Carrier vs Operating Carrier

| 구분 | 설명 | 예시 |
|------|------|------|
| **Marketing Carrier** | 항공권을 **판매**하는 항공사. 편명을 소유 | DL (델타항공) |
| **Operating Carrier** | 실제 **운항**하는 항공사 | KE (대한항공) |

코드셰어(Codeshare) 항공편에서 두 값이 다르다. 예: DL9000편으로 판매되지만 실제 운항은 KE가 수행.

### Cabin Class 코드

| 코드 | 설명 |
|------|------|
| **F** | First Class (일등석) |
| **C** | Business Class (비즈니스석) |
| **W** | Premium Economy (프리미엄 이코노미) |
| **Y** | Economy Class (이코노미석) |

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

### Last Seat Available (LSA)

좌석이 단 한 자리만 남은 상태에서도 **해당 운임으로 예약 가능**한지 여부. Y/N 값으로 표시.

### Electronic Ticketing

해당 항공편/운임이 **전자 발권(e-Ticket)** 가능한지 여부. Y/N 값으로 표시.

### Tech Stop

여객이 탑승/하차하지 않는 **기술 착륙**. 연료 보급 등의 목적으로 경유하는 공항.

### Miles Accrual

항공편 이용 시 적립되는 **마일리지 정보**. 프로그램별 적립률이 다를 수 있다.

| 필드 | 설명 |
|------|------|
| `milesAccrualId` | 마일리지 프로그램 식별자 |
| `milesAccrualDetails` | 적립 상세 (적립률, 프로그램 코드) |

---

## 12. 주요 코드셋 (Codesets)

### Form of Payment (결제 수단) - Codeset 9888 IA 02.2.238

| 코드 | 설명 |
|------|------|
| **AGT** | Sales Agent (판매 대리점 대행) |
| **CA** | Cash (현금) |
| **CC** | Credit Card (신용카드) |
| **CK** | Check (수표) |
| **MS** | Miscellaneous (기타) |
| **NR** | Non-refundable (환불 제한) |
| **PT** | Prepaid Ticket Advice (선불 항공권) |

### Flight Characteristic - Codeset 121Z 1A 15.1.1

| 코드 | 설명 |
|------|------|
| **ANY** | Generic Proposed Segment (일반 제안 구간) |
| **CDC** | Cross-over Date Combi (날짜 교차 조합) |
| **I** | Issued Flight (발권된 항공편) |
| **NAV** | No Availability (가용성 없음) |
| **NFA** | No Fare (운임 없음) |
| **NIT** | No Itinerary (여정 없음) |
| **SOC** | Sols-out Connection (매진 연결편) |
| **SOF** | Sold-out Flight (매진 항공편) |

### Identity Number (Corporate) - Codeset 7402 1A 08.1.1

| 코드 | 설명 |
|------|------|
| **C** | Amadeus Nego Corporate |
| **D** | Corporate Unifare |

### Interpretation (날짜/시간 단위) - Codeset 8883 1A 01.0.30

| 코드 | 설명 |
|------|------|
| **D** | Day (일) |
| **H** | Hour (시간) |
| **M** | Month (월) |
| **MON~SUN** | 요일 (월~일) |

### Level of Access - Codeset 9932 1A 12.1.1

| 코드 | 설명 |
|------|------|
| **1A** | Amadeus Access sell and update |
| **AS** | Amadeus Access sell only |
| **AU** | Amadeus Access update only |
| **DA** | Direct Access |
| **LC** | Low Cost Carrier / TLA |
| **LK** | Airline accessed by 1A direct access |
| **OA** | Other airlines |

### Item Number Type - Codeset 7143 1A 10.3.11

| 코드 | 설명 |
|------|------|
| **FIC** | Fictitious (가상) |
| **M** | Multiple Tickets (복수 항공권) |
| **SUB** | 요청 구간 1개만 포함하는 Recommendation |
| **VIS** | Virtual Interline Recommendation (전체 여정 불일치) |

---

## 13. 메시지 구조 용어

Fare_MasterPricerCalendar 기술 문서에서 사용되는 **메시지 구조 정의 용어**.

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

### Repetitions and Statuses

| Status | Repetitions | 의미 |
|--------|-------------|------|
| C | n | 0 ~ n개 반복 가능 (Conditional) |
| M | n | 1 ~ n개 반복 필수 (Mandatory) |
| M* | n | IATA 표준에서는 Conditional이나 비즈니스상 필수 |

---

## 약어 모음

| 약어 | 정식 명칭 | 설명 |
|------|----------|------|
| **FMPC** | Fare_MasterPricerCalendar | Amadeus 달력 기반 최저가 검색 API |
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
| **PSR** | Price Scheme Reference | 가격 체계 참조 번호 |
| **GDS** | Global Distribution System | 글로벌 유통 시스템 |
| **IATA** | International Air Transport Association | 국제항공운송협회 |
| **WBS** | Web Services | 웹 서비스 |

---

## 참고

- [WBS Integration Flow - Step 1](amadeus-wbs-integration-flow.md)
- [Fare_MasterPricerTravelBoardSearch 용어집](master-pricer-travelboard-search.md)
