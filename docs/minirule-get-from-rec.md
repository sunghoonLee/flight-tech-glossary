# MiniRule_GetFromRec

Amadeus의 Mini Rule 조회 API인 `MiniRule_GetFromRec`의 기술 용어를 정리한 문서입니다.
PNR(예약 레코드)에 저장된 TST 또는 PQR로부터 **환불/변경/최소체류/최대체류** 등 핵심 운임 규정을 구조화된 데이터로 조회한다.

> 기반 문서: MiniRule_GetFromRec 23.1 Technical Reference

!!! info "WBS Integration Flow Step 6"
    이 API는 [WBS Integration Flow](amadeus-wbs-integration-flow.md)의 **가격 조회 및 규정 확인 단계 Step 6**에 해당합니다.

---

## 1. 개요

### MiniRule_GetFromRec

PNR에 연결된 가격 레코드(TST/PQR)로부터 **ATPCO 카테고리별 운임 규정 요약**을 기계 판독 가능한 구조로 반환하는 API. 전체 Fare Rule 텍스트를 파싱하지 않고도 환불 가능 여부, 변경 수수료, 사전 구매 조건 등을 프로그래밍 방식으로 처리할 수 있다.

```
[Query: 규정 조회 요청]
  PNR 내 TST/PQR 참조 번호, 언어, 필터 옵션
       |
       v
  Amadeus MiniRule Engine
       |
       v
[Reply: 규정 조회 결과]
  mnrByPricingRecord x N개 (TST/PQR별)
  +-- pricingRecordId: TST/PQR 참조
  +-- fareComponentInfo x 16 (운임 구성요소별)
  |   +-- fareQualifierDetails: Fare Basis Code
  |   +-- originAndDestination: 출발지/도착지
  |   +-- segmentRefernce: 구간 Tattoo
  |   +-- listSituation x 99 (상황별 규정)
  |       +-- situationCode: His(과거) / Crt(현재)
  |       +-- mnrRulesInfoGrp x 600 (카테고리별 규정)
  |           +-- mnrCatInfo: 카테고리 (PEN/ADV/MNS...)
  |           +-- mnrFCInfoGrp: Fare Component 참조
  |           +-- mnrDateInfoGrp: 날짜/기간 조건
  |           +-- mnrMonInfoGrp: 수수료/금액 정보
  |           +-- mnrRestriAppInfoGrp: 제한 적용 정보
  +-- paxRef: 승객 참조
  +-- offerRef: Offer Tattoo (PQR인 경우)
  +-- errorWarningGroup: 오류/경고
```

### Query / Reply 구조

| 구분 | 메시지 | 설명 |
|------|--------|------|
| **Query** | `MiniRule_GetFromRec 23.1.1A` | TST/PQR 참조 번호로 Mini Rule 조회 요청 |
| **Reply** | `MiniRule_GetFromRecReply 23.1.1A` | 카테고리별 구조화된 운임 규정 반환 |

---

## 2. Query 주요 구조 (조회 요청)

### miniRulesQueryOption (Attribute)

Mini Rule 조회 시 **질의 옵션**을 지정하는 속성 구조.

| 필드 | 설명 |
|------|------|
| `criteriaSetType` | 기준 유형 한정자. 메시지 식별 기준 또는 일반 기준 여부 결정 |
| `criteriaDetails.attributeType` | 속성 유형 식별자 (최대 an..25) |
| `criteriaDetails.attributeDescription` | 속성 설명 (최대 an..256) |

### language

조회 결과의 **언어**를 지정하는 구조.

| 필드 | 설명 |
|------|------|
| `languageQualifier` | 언어 코드 한정자 (Codeset: 3455 UN 02.A.68) |
| `languageDetails.languageCode` | 언어 코드 (예: `ENG`) |
| `languageDetails.languageName` | 언어 이름 |

#### languageQualifier 주요 값

| 코드 | 설명 |
|------|------|
| **1** | Language normally used, 일반 사용 언어 |
| **2** | Language understood, 이해 가능 언어 |
| **4** | Written language, 서면 언어 |
| **7** | Native language, 모국어 |

### groupRecords

**레코드 그룹**. 최대 99개. 각 그룹은 하나의 TST/PQR 참조와 선택적 필터를 포함한다.

| 필드 | 설명 |
|------|------|
| `recordID.referenceType` | 참조 유형 (TST, PQR, TKT 등) |
| `recordID.uniqueReference` | 참조 번호 (TST 번호 등) |
| `filteringSelection` | 필터링 참조 정보 |

#### referenceType 주요 값 (Codeset: 1153 1A 11.1.1749)

| 코드 | 설명 |
|------|------|
| **TST** | Transitional Stored Ticket, 가격 저장 레코드 |
| **PQR** | Product Quotation Record Reference, 상품 견적 레코드 |
| **TKT** | Ticket Number, 발권 번호 |
| **PNR** | Record Locator, PNR 로케이터 |
| **OF** | Offer element tattoo, 오퍼 요소 참조 |
| **FRN** | Fare Recommendation Number, 운임 추천 번호 |
| **FUN** | Fare Upsell reco. Number, 업셀 추천 번호 |

---

## 3. Reply 주요 구조 (조회 결과)

### responseDetails (응답 상태)

Mini Rule 조회 요청의 **처리 결과 상태**를 반환하는 구조.

| 필드 | 설명 |
|------|------|
| `statusCode` | 처리 상태 코드 (a..6) |

#### statusCode 값 (Codeset: 9869 1A 02.1.596)

| 코드 | 설명 |
|------|------|
| **O** | OK processed, 정상 처리. 후속 세그먼트에 추가 정보 포함 |
| **N** | Recoverable error, 복구 가능한 오류. 상세는 errorWarningGroup 참조 |

### mnrByPricingRecord

**TST 또는 PQR 단위의 Mini Rule 그룹**. 최대 999회 반복. 하나의 가격 레코드에 대한 모든 운임 규정 정보를 포함한다.

| 필드 | 구조 | St | Rep | 설명 |
|------|------|----|-----|------|
| `pricingRecordId` | Item references and versions | M | 1 | TST/PQR 참조 번호 |
| `errorWarningGroup` | Group | C | 1 | 해당 레코드의 오류/경고 |
| `offerRef` | Element management segment | C | 1 | Offer Tattoo (PQR인 경우에만 반환) |
| `paxRef` | Reference information | C | 1 | 승객 또는 유아 참조 |
| `paxTypeLoc` | Group | C | 99 | 승객 유형/위치 정보 |
| `fareComponentInfo` | Group | C | 16 | 운임 구성요소별 규정 |

### fareComponentInfo (운임 구성요소)

개별 **Fare Component**(운임 구성 단위)의 규정 정보. 하나의 TST에 최대 16개 Fare Component가 존재할 수 있다.

| 필드 | 구조 | St | Rep | 설명 |
|------|------|----|-----|------|
| `fareQualifierDetails` | Fare qualifier details | M | 1 | Fare Basis Code |
| `fareComponentRef` | Reference information | M | 1 | Fare Component ID 및 Pricing Unit ID |
| `originAndDestination` | Origin and destination details | M | 1 | 출발지/도착지 |
| `segmentRefernce` | Element management segment | M | 99 | 구간 Tattoo 참조 |
| `listSituation` | Group | C | 99 | 상황별 규정 목록 |

### fareQualifierDetails (Fare Basis 정보)

| 필드 | 설명 |
|------|------|
| `additionalFareDetails.rateClass` | Fare Basis Code (최대 an..35) |

### fareComponentRef (Fare Component 참조)

| 필드 | 설명 |
|------|------|
| `referenceDetails.type` | 참조 유형: `FC` = Fare Component, `PU` = Pricing Unit |
| `referenceDetails.value` | Fare Component ID 또는 Pricing Unit ID |

### originAndDestination (출발지/도착지)

| 필드 | 설명 |
|------|------|
| `origin` | 출발지 공항/도시 코드 (IATA 3자리). Codeset: 3225 IA 02.2.7182 |
| `destination` | 도착지 공항/도시 코드 (IATA 3자리). Codeset: 3225 IA 02.2.7182 |

---

## 4. listSituation / mnrRulesInfoGrp (규정 상세 구조)

### listSituation

Fare Component에 적용되는 **규정 상황(Situation)**을 나타내는 그룹. `His`(과거 규정) 또는 `Crt`(현재 규정) 구분을 통해 시점에 따른 규정 차이를 표현한다.

| 필드 | 구조 | St | Rep | 설명 |
|------|------|----|-----|------|
| `listSituationDum` | Dummy segment | M | 1 | 더미 세그먼트 (구조 식별용) |
| `situationCode` | Status details | M | 1 | 규정 적용 시점 (His/Crt) |
| `situationDescription` | Free text information | M | 1 | 상황 설명 텍스트 |
| `mnrRulesInfoGrp` | Group | C | 600 | 카테고리별 규정 정보 |

#### situationCode indicator 값 (Codeset: 1245 1A 11.1.15)

| 코드 | 설명 |
|------|------|
| **His** | Historical, 과거(변경 전) 규정 |
| **Crt** | Current, 현재 적용 중인 규정 |

### mnrRulesInfoGrp (MINIRULESREGULPROPERTIESTYPE)

**개별 ATPCO 카테고리**의 규정 정보를 담는 핵심 그룹 구조. Mini Rule의 실질적인 데이터가 이 구조 안에 포함된다.

```
mnrRulesInfoGrp (최대 600회 반복)
+-- mnrCatInfo ........... 카테고리 번호/코드/이름
+-- mnrCatLoc ............ 카테고리 순서/가정(assumption)
+-- mnrFCInfoGrp x 16 ... Fare Component별 카테고리 적용 정보
|   +-- refInfo .......... 카테고리 번호 + FC 참조
|   +-- locationInfo ..... 위치(공항/도시) 정보
+-- mnrDateInfoGrp x 16 . 날짜/기간 조건
|   +-- dateInfo ......... 날짜/시간
|   +-- valueInfo ........ 단위 수량
|   +-- mnrDateLoc x 20 . 날짜 규정 상세
+-- mnrMonInfoGrp x 99 .. 금액/수수료 정보
|   +-- monetaryInfo ..... 금액, 통화
|   +-- penaltyInfo ...... 패널티 상세 (BDP/BNP/ADP/ANP)
|   +-- valueInfo ........ 단위 수량
|   +-- mnrMonLoc x 20 .. 금액 규정 상세
+-- mnrRestriAppInfoGrp .. 제한/적용 정보
    +-- mnrRestriAppInfo . 제한 적용 상태
    +-- mnrRestriAppLoc .. 제한 규정 상세
```

---

## 5. mnrCatInfo (카테고리 설명)

ATPCO 카테고리 번호와 코드를 나타내는 구조. Mini Rule에서 조회 가능한 규정 카테고리를 식별한다.

| 필드 | 설명 |
|------|------|
| `descriptionInfo.number` | ATPCO 카테고리 번호 (n..3). 예: C05, C06, C16 등 |
| `descriptionInfo.code` | ATPCO 카테고리 코드 (a..3). 예: ADV, MNS, PEN 등 |
| `descriptionInfo.name` | 카테고리 이름 (an..20). 예: ADVANCE PURCHASE |
| `descriptionInfo.language` | 언어 코드. Codeset: 3453 IA 02.2.9730 |
| `processIndicator` | 카테고리 처리 표시자 (an..3) |

### 주요 ATPCO 카테고리

| 카테고리 번호 | 코드 | 이름 | 설명 |
|------------|------|------|------|
| C05 | **ADV** | ADVANCE PURCHASE | 사전 구매/예약 기한. 출발 며칠 전까지 발권해야 하는지 |
| C06 | **MNS** | MINIMUM STAY | 최소 체류 기간. 목적지에서 최소한 며칠 체류해야 하는지 |
| C07 | **MXS** | MAXIMUM STAY | 최대 체류 기간. 귀환편까지 최대 며칠인지 |
| C08 | **STP** | STOPOVERS | 경유(Stopover) 제한. 허용 여부, 횟수, 추가 요금 |
| C09 | **TRF** | TRANSFERS | 환승 제한. 허용 여부, 횟수 |
| C11 | **BLK** | BLACKOUT DATES | 적용 제외일 (특정 날짜에 사용 불가) |
| C14 | **TRV** | TRAVEL RESTRICTIONS | 여행 제한. 특정 요일/시간대 제한 |
| C15 | **SLE** | SALES RESTRICTIONS | 판매 제한. 특정 지역/채널에서만 판매 |
| C16 | **PEN** | PENALTIES | 변경/취소 수수료. 환불/재발행 패널티 금액 |
| C18 | **HIP** | TICKET ENDORSEMENT | 발권 배서/제한 사항 |
| C19 | **CHD** | CHILDREN DISCOUNT | 소아 할인 규정 |
| C20 | **AGT** | AGENT DISCOUNT | 에이전트 할인 규정 |
| C23 | **MIS** | MISCELLANEOUS | 기타 규정 |
| C25 | **FRE** | FARE BY RULE | 규정 기반 운임 |
| C31 | **VOL** | VOLUNTARY CHANGES | 자발적 변경 규정 |
| C33 | **VRF** | VOLUNTARY REFUNDS | 자발적 환불 규정 |
| C35 | **ELG** | ELIGIBILITY | 자격 제한. 특정 승객만 구매 가능 |
| C50 | **CMB** | COMBINABILITY | 운임 결합 규정 |
| - | **SUR** | SURCHARGES | 추가 요금 (서차지) |
| - | **PTC** | PASSENGER TYPE | 적용 승객 유형 |

---

## 6. 패널티 상세 구조 (penaltyInfo)

### penaltyDetails

**시점별 패널티(수수료)** 정보를 나타내는 구조. 출발 전/후, 노쇼 전/후 등 상황별로 다른 수수료가 적용된다.

| 필드 | 형식 | 설명 |
|------|------|------|
| `qualifier` | an..3 | 패널티 적용 시점 한정자 (Codeset: 6808 1A 23.1.1) |
| `isApplicable` | n1 | 적용 가능 여부. `1` = 적용, `0` = 미적용 |
| `amount` | n..35 | 패널티 금액 |
| `currency` | an..3 | 통화 코드. Codeset: 6345 1A 02.2.705 |
| `numberOfMonths` | n..3 | 제한 기간 (월) |
| `numberOfDays` | n..3 | 제한 기간 (일) |
| `numberOfHours` | n..3 | 제한 기간 (시간) |
| `numberOfMinutes` | n..3 | 제한 기간 (분) |

#### qualifier (패널티 시점 한정자)

| 코드 | 설명 | 상세 |
|------|------|------|
| **BDP** | Before Departure Penalty | 출발 전 패널티 (변경/환불 수수료) |
| **BNP** | Before No-show Penalty | 노쇼 발생 전 패널티 |
| **ADP** | After Departure Penalty | 출발 후 패널티 |
| **ANP** | After No-show Penalty | 노쇼 발생 후 패널티 |

#### isApplicable 해석

`isApplicable` 필드의 의미는 카테고리 문맥에 따라 달라진다.

| 값 | Reissue(재발행) 문맥 | Refund(환불) 문맥 |
|---|----------------------|------------------|
| **1** | 교환(변경) **가능** | 환불 **가능** |
| **0** | 교환(변경) **불가** | 환불 **불가** |

```
패널티 판독 흐름:

  penaltyDetails 수신
       |
       +--[qualifier = BDP]---> 출발 전 수수료
       |   +-- isApplicable=1, amount=50000, currency=KRW
       |   "출발 전 변경 시 50,000원 수수료"
       |
       +--[qualifier = ADP]---> 출발 후 수수료
       |   +-- isApplicable=0
       |   "출발 후 변경 불가"
       |
       +--[qualifier = BNP]---> 노쇼 전 수수료
       |   +-- isApplicable=1, amount=100000, currency=KRW
       |   "노쇼 전 100,000원 수수료"
       |
       +--[qualifier = ANP]---> 노쇼 후 수수료
            +-- isApplicable=0
            "노쇼 후 환불/변경 불가"
```

---

## 7. 금액 및 날짜 관련 구조

### monetaryInfo (금액 정보)

수수료, 차액, 환불 금액 등 **금전 정보**를 나타내는 구조.

| 필드 | 형식 | 설명 |
|------|------|------|
| `monetaryDetails.typeQualifier` | an..3 | 금액 유형 한정자 (Codeset: 5025 1A 02.2.826) |
| `monetaryDetails.amount` | an..35 | 금액 |
| `monetaryDetails.currency` | an..3 | 통화 코드. Codeset: 6345 1A 02.2.705 |

#### typeQualifier 주요 값 (Codeset: 5025 1A 02.2.826)

| 코드 | 설명 |
|------|------|
| **700** | One way, 편도 금액 |
| **701** | Round trip, 왕복 금액 |
| **707** | Fixed whole amount, 고정 금액 |
| **708** | Percentage, 백분율 |
| **709** | Days, 일수 |
| **710** | Months, 개월수 |
| **711** | Hours, 시간수 |
| **712** | Total fare amount, 총 운임 금액 |
| **713** | Total amount of all surcharges, 전체 서차지 합계 |
| **714** | Refund amount, 환불 금액 |
| **715** | Fare difference amount, 운임 차액 |
| **716** | Change fee - penalty and/or administrative fee, 변경 수수료 |

### dateInfo (날짜/시간 정보)

규정의 **적용 기간 및 시점**을 나타내는 구조.

| 필드 | 형식 | 설명 |
|------|------|------|
| `dateAndTimeDetails.qualifier` | an..3 | 날짜/시간 한정자 (Codeset: 2005 1A 10.1.1401) |
| `dateAndTimeDetails.date` | an..35 | 날짜 |
| `dateAndTimeDetails.time` | n4 | 시간 |

#### qualifier 주요 값 (날짜 관련)

| 코드 | 설명 |
|------|------|
| **701** | Ticket effective date, 발권 유효 시작일 |
| **704** | Days earlier, 며칠 전 |
| **705** | Days later, 며칠 후 |
| **710** | Date Ticketed, 발권일 |
| **A** | Not Valid After - Last Travel Date, 유효 종료일 |
| **B** | Not Valid Before - First Travel Date, 유효 시작일 |

### valueInfo (단위 수량)

규정 내 **수량(일수, 개월수, 횟수 등)** 정보.

| 필드 | 형식 | 설명 |
|------|------|------|
| `quantityDetails.numberOfUnit` | n..15 | 수량 값 |
| `quantityDetails.unitQualifier` | an..3 | 단위 한정자 (Codeset: 6353 1A 10.1.3666) |

---

## 8. 승객 및 참조 구조

### paxRef (승객 참조)

Mini Rule이 적용되는 **승객**을 식별하는 구조.

| 필드 | 형식 | 설명 |
|------|------|------|
| `passengerReference.type` | an..10 | 승객 유형. Codeset: 1153 1A 10.1.184 |
| `passengerReference.value` | n..60 | 승객 또는 유아 Tattoo |

#### type 값 (Codeset: 1153 1A 10.1.184)

| 코드 | 설명 |
|------|------|
| **P** | Passenger/traveller reference number, 승객 참조 번호 |
| **PA** | Adult Passenger, 성인 승객 |
| **PI** | Infant Passenger, 유아 승객 |

### paxCode (승객 유형 코드)

`paxTypeLoc` 그룹 내에서 승객의 **유형 상태**를 나타낸다.

| 필드 | 형식 | 설명 |
|------|------|------|
| `statusInformation.indicator` | an..3 | 상태 표시자 (His/Crt). Codeset: 1245 1A 11.1.15 |
| `statusInformation.action` | an..3 | 액션 코드. Codeset: 1229 1A 02.2.137 |
| `statusInformation.type` | an..3 | 상태 유형. Codeset: 9015 IA 02.2.2408 |

### offerRef (Offer 참조)

PQR(상품 견적 레코드)에서 조회한 경우에만 반환되는 **Offer Tattoo**.

| 필드 | 형식 | 설명 |
|------|------|------|
| `reference.type` | an..10 | `OF` = Offer element tattoo. Codeset: 1153 1A 10.1.223 |
| `reference.value` | an..60 | Offer Tattoo 번호 |

### segmentRefernce (구간 참조)

Fare Component에 해당하는 **항공편 구간(Segment)**의 Tattoo.

| 필드 | 형식 | 설명 |
|------|------|------|
| `reference.type` | an..10 | `ST` = Segment Tattoo. Codeset: 1153 1A 10.1.200 |
| `reference.value` | n..60 | 구간 Tattoo 번호 |

---

## 9. 오류 처리 구조 (ERRORGROUPTYPE)

### errorWarningGroup

최상위 및 `mnrByPricingRecord` 내부에 존재하는 **오류/경고 그룹**.

| 필드 | 구조 | St | Rep | 설명 |
|------|------|----|-----|------|
| `errorOrWarningCodeDetails` | Application error information | M | 1 | 오류/경고 코드 상세 |
| `errorWarningDescription` | Free text information | C | 1 | 오류/경고 설명 텍스트 |

### errorOrWarningCodeDetails (오류 코드)

| 필드 | 형식 | 설명 |
|------|------|------|
| `errorDetails.errorCode` | an..5 | 오류 코드. Codeset: 9321 1A 10.2.76 |
| `errorDetails.errorCategory` | an..3 | 코드 리스트 식별. Codeset: 1131 IA 02.2.1905 |
| `errorDetails.errorCodeOwner` | an..3 | 코드 리스트 책임 기관. Codeset: 3055 1A 10.1.1482 |

---

## 10. 주요 Codeset 테이블

### Processing status code (Ref: 9869 1A 02.1.596)

| 값 | 설명 |
|----|------|
| **N** | Recoverable error, 복구 가능 오류 |
| **O** | OK processed, 정상 처리 완료 |

### Reference qualifier - recordID (Ref: 1153 1A 11.1.1749)

| 값 | 설명 |
|----|------|
| **FRN** | Fare Recommendation Number, 운임 추천 번호 |
| **FUN** | Fare Upsell reco. Number, 업셀 추천 번호 |
| **OF** | Offer element tattoo, 오퍼 요소 참조 |
| **PNR** | Record Locator, PNR 로케이터 |
| **PQR** | Product Quotation Record Reference, 상품 견적 레코드 |
| **TKT** | Ticket Number, 발권 번호 |
| **TST** | Transitional Stored Ticket, 가격 저장 레코드 |

### Reference qualifier - fareComponentRef (Ref: 1153 1A 10.1.201)

| 값 | 설명 |
|----|------|
| **FC** | Fare component reference, 운임 구성요소 참조 |
| **PU** | Pricing Unit, 가격 단위 |

### Reference qualifier - paxRef (Ref: 1153 1A 10.1.420)

| 값 | 설명 |
|----|------|
| **P** | Pax, 승객 |
| **PA** | Adult Passenger, 성인 승객 |
| **PI** | Infant Passenger, 유아 승객 |
| **S** | Segment, 구간 |

### Monetary amount type qualifier (Ref: 5025 1A 02.2.826)

| 값 | 설명 |
|----|------|
| **700** | One way, 편도 |
| **701** | Round trip, 왕복 |
| **702** | PFC - to indicate PFC amount |
| **703** | Stopover, 경유 |
| **704** | Open Jaw surcharge |
| **707** | Fixed whole amount, 고정 금액 |
| **708** | Percentage, 백분율 |
| **709** | Days, 일수 |
| **710** | Months, 개월수 |
| **711** | Hours, 시간수 |
| **712** | Total fare amount, 총 운임 |
| **713** | Total amount of all surcharges, 전체 서차지 |
| **714** | Refund amount, 환불 금액 |
| **715** | Fare difference amount, 운임 차액 |
| **716** | Change fee, 변경 수수료 |

### Currency, coded (Ref: 6345 1A 02.2.705)

| 값 | 설명 |
|----|------|
| **777** | Neutral Unit of Construction (NUC) |
| **M** | Miles, 마일리지 |
| **P** | Points, 포인트 |
| **V** | Voucher, 바우처 |

### Status indicator, coded (Ref: 1245 1A 11.1.15)

| 값 | 설명 |
|----|------|
| **His** | Historical, 과거 규정 |
| **Crt** | Current, 현재 규정 |

주요 상태 코드(발권/운임 관련):

| 값 | 설명 |
|----|------|
| **700** | Fare basis may vary by carrier |
| **701** | Fares based on passenger type and/or discount input |
| **705** | Fares and/or rates for future ticketing are subject to change |
| **708** | No IATA fares |

### Action request / notification (Ref: 1229 1A 02.2.137)

| 값 | 설명 |
|----|------|
| **0** | No |
| **1** | Yes |
| **FO** | Forbidden, 금지 |
| **MA** | Mandatory, 필수 |
| **OP** | Optional, 선택 |

### Special condition code (Ref: 4183 1A 02.A.369)

| 값 | 설명 |
|----|------|
| **CRI** | Collection request intended, 징수 요청 |
| **DRI** | Delivery request intended, 배송 요청 |

### Information type (Ref: 9980 IA 02.2.6176)

| 값 | 설명 |
|----|------|
| **10** | Endorsement information, 배서 정보 |
| **17** | Ticketing information, 발권 정보 |
| **RCD** | Reason code, 사유 코드 |
| **TXT** | Other conditions - see text, 기타 조건 |

---

## 11. 메시지 구조 용어

MiniRule_GetFromRec 기술 문서에서 사용되는 **메시지 구조 정의 용어**.

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
| `a3` | 고정 3자리 문자 | `PEN` (카테고리 코드) |
| `n1` | 고정 1자리 숫자 | `1` (isApplicable) |
| `an..35` | 가변 영숫자 최대 35자리 | `Y1OWKR` (Fare Basis) |
| `n..15` | 가변 숫자 최대 15자리 | `50000` (패널티 금액) |

---

## 약어 모음

| 약어 | 정식 명칭 | 설명 |
|------|----------|------|
| **MNR** | Mini Rules | 운임 규정 요약 |
| **TST** | Transitional Stored Ticket | PNR 내 가격 저장 레코드 |
| **PQR** | Product Quotation Record | NDC 기반 상품 견적 레코드 |
| **PNR** | Passenger Name Record | 승객 예약 기록 |
| **ATPCO** | Airline Tariff Publishing Company | 항공 운임 데이터 제공 기관 |
| **FC** | Fare Component | 운임 구성요소 (구간별 운임 단위) |
| **PU** | Pricing Unit | 가격 단위 (왕복 등 묶음) |
| **PEN** | Penalty | 변경/취소 수수료 |
| **ADV** | Advance Purchase | 사전 구매/예약 기한 |
| **MNS** | Minimum Stay | 최소 체류 기간 |
| **MXS** | Maximum Stay | 최대 체류 기간 |
| **STP** | Stopovers | 경유 제한 |
| **TRF** | Transfers | 환승 제한 |
| **ELG** | Eligibility | 자격 제한 |
| **SUR** | Surcharges | 추가 요금 |
| **PTC** | Passenger Type Code | 승객 유형 코드 |
| **BDP** | Before Departure Penalty | 출발 전 패널티 |
| **BNP** | Before No-show Penalty | 노쇼 전 패널티 |
| **ADP** | After Departure Penalty | 출발 후 패널티 |
| **ANP** | After No-show Penalty | 노쇼 후 패널티 |
| **NUC** | Neutral Unit of Construction | 운임 계산용 중립 통화 단위 |
| **RBD** | Reservation Booking Designator | 예약 클래스 |
| **NDC** | New Distribution Capability | 항공사 직접 연결 표준 |
| **EMD** | Electronic Miscellaneous Document | 부가서비스 전표 |

---

## 참고

- [WBS Integration Flow - Step 6](amadeus-wbs-integration-flow.md)
- [Fare_CheckRules 용어집](fare-check-rules.md)
- [Master Pricer Travelboard Search 용어집](master-pricer-travelboard-search.md)
