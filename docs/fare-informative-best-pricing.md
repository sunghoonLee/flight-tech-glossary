# Fare_InformativeBestPricingWithoutPNR

PNR 없이 여정과 승객 정보만으로 **최적 운임을 자동 계산**하는 API 용어집입니다.
시스템이 가장 유리한 운임 조합(Best Pricing)을 자동 선택하여 반환하며, 운임 확정 전 사전 검증 용도로 사용됩니다.

> 기반 문서: Fare_InformativeBestPricingWithoutPNR 18.1 Technical Reference

!!! info "WBS Integration Flow Step 3"
    이 API는 [WBS Integration Flow](amadeus-wbs-integration-flow.md)의 **가격 조회 및 규정 확인 단계 Step 3**에 해당합니다.

---

## 1. 개요

### Fare_InformativeBestPricingWithoutPNR

Amadeus GDS가 제공하는 **PNR 미생성 상태의 최적 운임 자동 계산 API**. 여정(Segment)과 승객(Passenger) 정보를 입력하면, 시스템이 가장 유리한 운임 조합을 자동 선택하여 가격을 계산한다. `Fare_InformativePricingWithoutPNR`(특정 운임 확정 조회)과 달리, 이 API는 시스템이 최적 조합을 **자동으로** 결정한다.

```
[Query: 가격 조회 요청]
  승객 정보, 여정 정보, 가격 옵션
       │
       ▼
  Amadeus Best Pricing Engine
  (최적 운임 조합 자동 선택)
       │
       ▼
[Reply: 가격 조회 결과]
  mainGroup × N개 (Pricing Group)
  ├─ generalIndicators: 국내/국제 구분 (SITI/SOTO)
  ├─ pricingGroupLevelGroup: 승객별 운임 상세
  │   ├─ fareInfoGroup: 운임 기본 정보
  │   ├─ segmentLevelGroup: 구간별 상세
  │   ├─ corporateGroup: 기업 운임 정보
  │   └─ negoFareGroup: 협정 운임 정보
  ├─ fareComponentDetailsGroup: Fare Component 상세
  └─ carrierFeeGroup: 항공사 수수료
```

| 항목 | 내용 |
|------|------|
| API 명 | `Fare_InformativeBestPricingWithoutPNR` |
| 버전 | 18.1 |
| 플로우 단계 | Step 3 -- 가격 조회 및 규정 확인 |
| 목적 | 시스템이 가장 유리한 운임 조합을 자동 선택하여 계산 |
| 이전 단계 | [Fare_MasterPricerTravelBoardSearch](master-pricer-travelboard-search.md) |
| 다음 단계 | [Fare_InformativePricingWithoutPNR](fare-informative-pricing.md) |

### Query / Reply 구조

| 구분 | 메시지 | 설명 |
|------|--------|------|
| **Query** | `Fare_InformativeBestPricingWithoutPNR` | 여정·승객·옵션 입력 (요청) |
| **Reply** | `Fare_InformativeBestPricingWithoutPNRReply` | 최적 운임 계산 결과 (응답) |

---

## 2. Query 주요 구조 (가격 조회 요청)

### originatorGroup

요청 **발신자 정보**를 담는 구조.

| 필드 | 설명 |
|------|------|
| `additionalBusinessInformation` | 발권 사무소, 판매 위치 등 부가 사업 정보 |

### passengersGroup (승객 정보)

**승객 정보**를 정의하는 구조. 최대 198회 반복 가능 (M, 198).

| 필드 | 설명 |
|------|------|
| `segmentRepetitionControl` | 구간 반복 제어 정보 |
| `specificTravellerDetails` | 개별 승객 상세 (이름, PTC 등) |

#### PTC (Passenger Type Code)

승객의 유형을 나타내는 코드. 운임 산정의 기준이 된다.

| 코드 | 설명 |
|------|------|
| **ADT** | Adult, 성인 (12세 이상) |
| **CHD** | Child, 소아 (2~11세) |
| **INF** | Infant, 유아 (2세 미만, 좌석 미사용) |
| **INS** | Infant with Seat, 유아 (좌석 사용) |
| **CRW** | Crew, 승무원 |
| **PAX** | Passengers, 일반 승객 |

### segmentGroup (여정 구간)

**여정 구간(Segment) 정보**를 정의하는 구조. 최대 99회 반복 가능 (M, 99).

| 필드 | 설명 |
|------|------|
| `segmentInformation.travelProductInformation` | 출발/도착지, 항공사, 편명, 날짜 등 |
| `segmentInformation.additionalProductDetails` | 부가 상품 정보 (기재, RBD 등) |
| `segmentInformation.productInformation` | 예약 클래스, 캐빈 정보 |
| `additionalInformation` | 구간 추가 정보 |

### pricingOptionGroup (가격 옵션)

**가격 계산 옵션**을 설정하는 구조. 최대 999회 반복 가능 (C, 999).

| 필드 | 설명 |
|------|------|
| `pricingOptionKey.pricingOptionKey` | 가격 옵션 키 (Attribute Key) |
| `currency` | 통화 지정 |
| `dateInformation` | 날짜 관련 옵션 (예약일, 발권일 오버라이드) |
| `frequentFlyerInformation` | 상용 고객 정보 |
| `formOfPaymentInformation` | 결제 수단 정보 |
| `locationInformation` | 위치 정보 (POS, POT) |
| `paxSegTstReference` | 승객/구간/TST 참조 |

#### Pricing Option Key (Attribute Key) -- 주요 가격 옵션

가격 계산 시 적용할 옵션을 지정하는 키. 시스템 동작을 세밀하게 제어한다.

| 키 | 설명 |
|------|------|
| **AC** | Award / Mileage pricing (마일리지 운임) |
| **AT** | Agreement code pricing (계약 코드 운임) |
| **AWD** | Award Fare pricing (특가 운임) |
| **BND** | Bound level pricing (구간 단위 가격 책정) |
| **CAB** | Cabin class override (캐빈 클래스 오버라이드) |
| **CC** | Credit card for pricing (신용카드 결제 가격) |
| **CON** | Connection override (연결편 오버라이드) |
| **CRP** | Corporate fare (기업 운임) |
| **DAT** | Date override (날짜 오버라이드) |
| **DO** | Domestic itinerary pricing override (국내 여정 오버라이드) |
| **ET** | Electronic ticket restriction (전자 발권 제한) |
| **FBP** | Force pricing by fare basis (Fare Basis 강제 적용) |
| **FCO** | Fare Calculation Override (운임 계산 오버라이드) |
| **FCS** | Fare Currency Selection (운임 통화 선택) |
| **FOP** | Form of Payment (결제 수단) |
| **GRI** | Global Routing Indicator (글로벌 라우팅 지표) |
| **IP** | Instant pricing (즉시 가격 책정) |
| **MA** | Mileage Accrual (마일리지 적립) |
| **MBT** | Pricing by fare type (운임 유형별 가격) |
| **MC** | Miles & Cash pricing (마일+현금 운임) |
| **MIT** | Minimum connecting time (최소 환승 시간) |
| **NBP** | No breakpoint forced (브레이크포인트 미강제) |
| **NF** | No fare restriction (운임 제한 없음) |
| **NOP** | No option (옵션 없음) |
| **NS** | No surcharge (서차지 없음) |
| **NSD** | No stopover discount (스톱오버 할인 없음) |
| **NVO** | No validation override (유효성 검사 오버라이드 없음) |
| **OBF** | OB Fee (Optionally Billable Fee) |
| **PAX** | Passenger specific pricing (승객 특정 가격) |
| **PFF** | Price by Fare Family (Fare Family별 가격) |
| **PL** | Price List (가격 리스트) |
| **POS** | Point of Sale (판매 지점) |
| **POT** | Point of Ticketing (발권 지점) |
| **PRM** | Promotional fare (프로모션 운임) |
| **PSR** | Passenger Status/Residency (승객 상태/거주지) |
| **PTA** | Prepaid Ticket Advice (선불 항공권) |
| **PTC** | Passenger Type Code (승객 유형 코드) |
| **RC** | Return cabin pricing (귀국 캐빈 가격) |
| **RLA** | Rule override - all rules (전체 규정 오버라이드) |
| **RLI** | Rule override - IATA (IATA 규정 오버라이드) |
| **RLO** | Rule override - specific (특정 규정 오버라이드) |
| **RN** | Return non-homogeneous (비동질 귀국편) |
| **RP** | Published fare (공시 운임) |
| **RU** | Unifares (통합 운임) |
| **RW** | Lowest fare (최저 운임) |
| **STO** | Stopover override (스톱오버 오버라이드) |
| **TKT** | Ticketing date override (발권일 오버라이드) |
| **VC** | Validating Carrier (발권 항공사) |
| **WC** | Waiver Code (면제 코드) |
| **WQ** | Web Quota (웹 할당) |
| **WT** | Withhold Tax (원천징수 세금) |
| **ZAP** | ZapOff Discount (ZapOff 할인) |

---

## 3. Reply 주요 구조 (가격 조회 결과)

### messageDetails (메시지 상세)

응답 메시지의 **기본 식별 정보**.

| 필드 | 설명 |
|------|------|
| `messageFunctionDetails.messageFunction` | 메시지 기능 코드 (예: 223=Itinerary pricing) |

### errorGroup (오류 정보)

가격 조회 시 발생한 **오류 정보**를 담는 구조.

| 필드 | 설명 |
|------|------|
| `applicationError.applicationErrorDetail.error` | 오류 코드 |
| `errorText.freeText` | 오류 상세 메시지 텍스트 |

#### 주요 Application Error 코드

| 코드 | 설명 |
|------|------|
| **101** | INVALID PASSENGER CODE OR DISCOUNT |
| **115** | FARE NOT APPLICABLE FOR PASSENGER TYPE |
| **117** | UNABLE TO PRICE DUE TO NO MATCH ON CARRIER |
| **125** | PRIVATE/NEGOTIATED/NET FARE USED |
| **131** | CARRIER RESTRICTION APPLIES |
| **141** | TAX NOT FOUND |
| **169** | CHECK FARE NOTE |
| **432** | VERIFY RESTRICTION |

### mainGroup (메인 가격 그룹)

**가격 계산 결과의 핵심 구조**. 하나의 mainGroup이 하나의 완전한 가격 계산 결과를 나타낸다.

```
mainGroup
├─ generalIndicators           ← 국내/국제 구분 (SITI/SOTO)
├─ pricingGroupLevelGroup (×99) ← 승객 유형별 가격 상세
│   ├─ fareInfoGroup (×1)      ← 운임 기본 정보
│   │   ├─ fareInformation     ← 운임 유형, Base Fare 등
│   │   ├─ fareAmount          ← 운임 금액 상세
│   │   ├─ taxInformation      ← 세금 내역
│   │   └─ fareRulesInformation ← 운임 규정 참조
│   ├─ segmentLevelGroup (×99) ← 구간별 운임 상세
│   │   ├─ travelProductInformation ← 항공편 정보
│   │   ├─ fareQualifierDetails    ← Fare Basis, RBD 등
│   │   └─ excessBaggageDetails    ← 수하물 정보
│   ├─ corporateGroup          ← 기업 운임 정보
│   ├─ negoFareGroup           ← 협정 운임 정보
│   └─ carrierFeeGroup         ← 항공사 수수료
└─ fareComponentDetailsGroup   ← Fare Component 상세 분해
```

### generalIndicators (국내/국제 지표)

여정의 **국내/국제 구분** 및 판매/발권 위치 정보.

| 필드 | 설명 |
|------|------|
| `productDateTimeDetails.departureDate` | 출발일 |
| `productDateTimeDetails.departureTime` | 출발 시간 |
| `productDateTimeDetails.arrivalDate` | 도착일 |
| `productDateTimeDetails.arrivalTime` | 도착 시간 |

#### SITI / SOTO (판매·발권 위치 지표)

여정의 판매(Sold) 및 발권(Ticketed) 위치와 여정의 출발/도착(Inside/Outside) 관계를 나타내는 지표.

| 코드 | 설명 |
|------|------|
| **II** (SITI) | Sold Inside / Ticketed Inside -- 출발국에서 판매 및 발권 |
| **IO** (SITO) | Sold Inside / Ticketed Outside -- 출발국에서 판매, 타국에서 발권 |
| **OI** (SOTI) | Sold Outside / Ticketed Inside -- 타국에서 판매, 출발국에서 발권 |
| **OO** (SOTO) | Sold Outside / Ticketed Outside -- 타국에서 판매 및 발권 |

### pricingGroupLevelGroup (승객별 가격 상세)

**승객 유형별 상세 운임 정보**. 최대 99회 반복.

| 필드 | 설명 |
|------|------|
| `fareInfoGroup` | 운임 기본 정보 (금액, 유형, 규정) |
| `segmentLevelGroup` | 구간별 운임 상세 |
| `corporateGroup` | 기업 운임 관련 정보 |
| `negoFareGroup` | 협정/네고 운임 관련 정보 |
| `carrierFeeGroup` | 항공사 추가 수수료 (OB Fee 등) |

---

## 4. 운임 금액 구조 (Monetary Information)

### Monetary Amount Type Qualifier (금액 유형)

운임 응답에 포함되는 **금액의 유형**을 구분하는 코드.

| 코드 | 설명 |
|------|------|
| **712** | Total fare amount (총 운임 금액) |
| **B** | Base fare (기본 운임) |
| **BFA** | Base Fare Amount (기본 운임 금액) |
| **E** | Equivalent fare (환산 운임) |
| **ACC** | Amount of Cash Converted (현금 환산 금액) |
| **MAC** | Mileage Accrual (마일리지 적립) |
| **OB** | OB fee Amount (OB 수수료 금액) |
| **OBA** | Original Base Fare (원본 기본 운임) |
| **OCA** | Original Total Fare (원본 총 운임) |
| **OPA** | Original Base Fare in award currency (어워드 통화 원본 기본 운임) |
| **OTX** | Original Total Taxes (원본 총 세금) |
| **PND** | PTC non discounted amount (PTC 비할인 금액) |
| **PTS** | Fare amount converted in points (포인트 환산 운임) |
| **RCP** | Remaining Cash to Pay - without taxes (잔여 현금 지급액, 세금 제외) |
| **TRC** | Total Amount to be paid in cash (현금 지불 총액) |
| **TTC** | Total Taxes to be paid in Cash (현금 지불 총 세금) |
| **WBD** | Web Discount (웹 할인) |
| **XOB** | Total amount OB fee excluded (OB 수수료 제외 총액) |

### Monetary Amount Type -- 상세 분류

| 코드 | 설명 |
|------|------|
| **700** | One way (편도) |
| **701** | Round trip (왕복) |
| **702** | PFC - Passenger Facility Charge 금액 표시 |
| **703** | Stopover (스톱오버) |
| **704** | Open Jaw surcharge (오픈 조 서차지) |
| **706** | Miscellaneous (기타) |
| **707** | Fixed whole amount (고정 금액) |
| **708** | Percentage (백분율) |
| **712** | Total fare amount (총 운임) |
| **713** | Total amount of all surcharges (전체 서차지 합계) |
| **714** | Refund amount (환불 금액) |
| **715** | Fare difference amount (운임 차액) |
| **716** | Change fee - penalty and/or administrative fee (변경 수수료) |

---

## 5. 운임 유형 구조 (Fare Information)

### priceTariffType (운임 타리프 유형)

운임의 **출처/유형**을 나타내는 코드. Best Pricing에서 시스템이 자동 선택한 운임이 어떤 유형인지 식별한다.

| 코드 | 설명 |
|------|------|
| **P** | PRIVATE -- 사설(비공시) 운임 |
| **A** | ATAF -- ATAF 운임 |
| **I** | IATA -- IATA 공시 운임 |
| **M** | NEGO CONS -- 네고 콘솔리데이터 운임 |
| **N** | NEGO -- 네고(협정) 운임 |
| **D** | DDF CORP -- DDF 기업 운임 |
| **T** | TOUR -- 투어 운임 |
| **K** | DDF INC TOUR -- DDF 인클루시브 투어 운임 |
| **L** | DDF BULK TOUR -- DDF 벌크 투어 운임 |
| **O** | OVERRIDE -- 오버라이드 운임 |

### Rate Type Identification (운임 유형 식별)

운임의 **카테고리**를 구분하는 코드.

| 코드 | 설명 |
|------|------|
| **700** | Normal fares (일반 운임) |
| **701** | Special Fares (특별 운임) |
| **702** | APEX Fares (사전 구매 할인 운임) |
| **703** | Super APEX Fares (슈퍼 APEX 운임) |
| **704** | APEX and Super APEX Fares |
| **705** | Round the World Fare (세계 일주 운임) |
| **706** | Circle Trip/Triangle Fares (써클 트립/삼각 운임) |
| **707** | Excursion Fares (여행 운임) |
| **708** | PEX Fares (PEX 운임) |
| **715** | All Fares (전체 운임) |
| **716** | Discount Fares (할인 운임) |
| **717** | First Class travel fares (퍼스트 클래스 운임) |
| **718** | Intermediate class travel fares (비즈니스 클래스 운임) |
| **719** | Economy class travel fares (이코노미 클래스 운임) |
| **720** | Unsaleable Fares (판매 불가 운임) |

---

## 6. 세금 구조 (Tax Details)

### Duty/Tax/Fee Category (세금 카테고리)

| 코드 | 설명 |
|------|------|
| **700** | Tax on base fare (기본 운임 세금) |
| **701** | Tax on total amount (총액 세금) |
| **702** | Tax exempt (세금 면제) |
| **703** | Tax on commission (커미션 세금) |
| **704** | Tax on specific amount (특정 금액 세금) |
| **705** | Tax on penalty amount (패널티 세금) |
| **706** | Tax on service fee (서비스 수수료 세금) |
| **707** | Tax on OB fee (OB 수수료 세금) |
| **D** | Tax surcharge (세금 서차지) |
| **E** | Exempt (면제) |
| **I** | International tax (국제 세금) |
| **N** | National tax (국내 세금) |
| **Q** | Surcharge (서차지) |
| **T** | Tax (세금) |

### Monetary Function (운임 기능)

세금 및 운임의 **환불/패널티 관련 상태**를 나타내는 코드.

| 코드 | 설명 |
|------|------|
| **700** | Base fare (기본 운임) |
| **701** | Total fare (총 운임) |
| **702** | Tickets are non-refundable (환불 불가 항공권) |
| **703** | Tickets are non-refundable after departure (출발 후 환불 불가) |
| **704** | Penalties apply (패널티 적용) |
| **705** | Subject to cancellation/change penalty (취소/변경 패널티 대상) |
| **706** | Tickets are non-refundable before departure (출발 전 환불 불가) |
| **EXF** | Exclude Fee (수수료 제외) |
| **INF** | Include Fee (수수료 포함) |

---

## 7. Price Type Qualifier (가격 유형 한정자)

가격 계산 결과의 **특성과 상태**를 나타내는 코드. 응답에서 운임의 성격을 설명한다.

### 운임 계산 관련

| 코드 | 설명 |
|------|------|
| **700** | Present credit card indicator (신용카드 제시 지표) |
| **701** | Fare basis in fare calculation (운임 계산 내 Fare Basis) |
| **702** | Currency override (통화 오버라이드) |
| **703** | Calculation line restriction (계산 라인 제한) |
| **704** | Global indicator in fare calculation (글로벌 지표) |
| **705** | No advance purchase (사전 구매 없음) |
| **706** | No maximum and minimum stay fares (최대/최소 체류 없음) |
| **707** | No minimum stay fares (최소 체류 없음) |
| **708** | No maximum stay fares (최대 체류 없음) |
| **709** | No penalty fares (패널티 없는 운임) |
| **710** | No restriction fares (제한 없는 운임) |
| **711** | Override booking date (예약일 오버라이드) |
| **712** | Price at specified passenger type only (지정 승객 유형만 가격 적용) |
| **713** | Refundable fares (환불 가능 운임) |
| **714** | Spaces in fare calculation (운임 계산 공백) |
| **715** | Q surcharge withheld (Q 서차지 보류) |
| **716** | Convert directly from NUQ to Euro (NUQ→Euro 직접 환산) |
| **717** | Only electronic ticket fares (전자 발권 운임만) |
| **718** | Exclude Electronic ticket fares (전자 발권 운임 제외) |

### 체류 및 사전 구매 관련

| 코드 | 설명 |
|------|------|
| **719** | Minimum stay (최소 체류) |
| **720** | Maximum stay (최대 체류) |
| **721** | Advance purchase (사전 구매) |
| **722** | Override class criteria (클래스 기준 오버라이드) |
| **723** | Check interline agreement (인터라인 협정 확인) |
| **724** | Override the rules (규정 오버라이드) |

### 여정 및 판매 지표

| 코드 | 설명 |
|------|------|
| **D** | Domestic itinerary (국내 여정) |
| **I** | International itinerary (국제 여정) |
| **II** | Sold in/ticket in (국내 판매, 국내 발권) |
| **IO** | Sold in/ticket out (국내 판매, 국외 발권) |
| **OI** | Sold out/ticket in (국외 판매, 국내 발권) |
| **OO** | Sold out/ticket out (국외 판매, 국외 발권) |

### 환불/패널티 지표

| 코드 | 설명 |
|------|------|
| **NR** | Non refundable (환불 불가) |
| **NE** | Non-endorseable (배서 불가) |
| **PA** | Penalty applies indicator (패널티 적용 지표) |
| **LF** | Low fare finder - rebooking recommended (최저가 재예약 권장) |
| **NF** | No low fare finder options found (최저가 옵션 없음) |

### 기타 운임 지표

| 코드 | 설명 |
|------|------|
| **DF** | Discount fares were applied (할인 운임 적용됨) |
| **G** | Subject to government approval (정부 승인 대상) |
| **HF** | Horizontal fare ladder (수평 운임 래더) |
| **VF** | Vertical fare ladder (수직 운임 래더) |
| **R** | Net reporting indicator (순보고 지표) |
| **RD** | Rate desk priced indicator (Rate Desk 가격 지표) |
| **RI** | Rules Source Override - IATA (IATA 규정 출처 오버라이드) |
| **RN** | Rules Source Override - Negotiated (협정 규정 출처 오버라이드) |
| **S** | Self Sale (자사 판매) |
| **SC** | Soft currency processing required (소프트 통화 처리 필요) |
| **T** | Tax on commission ind (커미션 세금 지표) |
| **TA** | Data for display and ticketing is allowed (표시 및 발권 허용) |
| **TF** | Inclusive tour fare amounts not to be printed (투어 운임 비인쇄) |
| **TN** | Data for display only - ticketing not allowed (표시만 허용, 발권 불가) |
| **TW** | Warning information included (경고 정보 포함) |
| **X** | Ticketing Mode Indicator Option 5 (발권 모드 옵션 5) |
| **N** | Ticketing Mode Indicator Not Option 5 (발권 모드 비옵션 5) |
| **SP** | PNR Split needed (PNR 분할 필요) |
| **ANS** | Availability not sufficient (가용성 부족) |

---

## 8. 결제 수단 (Form of Payment)

### Form of Payment Identification

운임 계산 시 적용하는 **결제 수단 유형**.

| 코드 | 설명 |
|------|------|
| **AGT** | Sales Agent 발행 문서 대리 결제 |
| **CA** | Cash (현금) |
| **CC** | Credit Card (신용카드) |
| **CK** | Check (수표) |
| **GR** | Government transportation request (정부 운송 요청) |
| **MS** | Miscellaneous (기타) |
| **NR** | Non-refundable (refund restricted) (환불 제한) |
| **PT** | Prepaid Ticket Advice (PTA) (선불 항공권) |
| **SGR** | Single government transportation request (단일 정부 운송 요청) |
| **UN** | United Nations Transportation Request (유엔 운송 요청) |

---

## 9. 위치 관련 코드 (Location)

### Location Function Code Qualifier

위치의 **기능/역할**을 구분하는 코드.

| 코드 | 설명 |
|------|------|
| **A** | Arrival (도착) |
| **D** | Departure (출발) |
| **DT** | Direction of travel (여행 방향) |
| **FC** | Fare Component (운임 구성요소) |
| **J** | Journey (여정) |
| **PDS** | Point of Delivery of the service (서비스 제공 지점) |
| **POR** | Portion (구간) |
| **PT** | Point of ticketing (발권 지점) |
| **PU** | Pricing Unit (가격 단위) |
| **S** | Point of sale (판매 지점) |
| **SEC** | Sector (섹터) |
| **X** | Point of delivery of the srv (서비스 배달 지점) |
| **POS** | Point Of Sale (판매 지점) |
| **POT** | Point Of Ticketing (발권 지점) |
| **WT** | Withhold Tax (원천징수 세금) |

### Place/Location Identification

위치 식별에 사용되는 **특수 코드**.

| 코드 | 설명 |
|------|------|
| **ARNK** | ARNK -- 라우팅 목적 전용 (Surface Segment) |
| **ZZZ** | 모든 도시를 지정하는 데 사용 |

---

## 10. Data Indicator / Data Type (데이터 지표)

### Data Indicator

여정의 **성격**을 나타내는 지표.

| 코드 | 설명 |
|------|------|
| **DOM** | Domestic (국내) |
| **INT** | International (국제) |
| **EMP** | Empty (비어 있음) |
| **LOC** | Local (로컬) |
| **STP** | Technical Stopover (기술 착륙) |

### Data Type

승객이나 데이터의 **유형**을 나타내는 코드.

| 코드 | 설명 |
|------|------|
| **ADT** | Adult (성인) |
| **CHD** | Child (소아) |
| **INF** | Infant (유아) |
| **CRW** | Crew (승무원) |
| **PAX** | Passengers (승객) |

---

## 11. Information Type (정보 유형)

응답 메시지에 포함되는 **자유 텍스트 정보의 유형**을 구분하는 코드.

### 주요 Information Type 코드

| 코드 | 설명 |
|------|------|
| **1** | Flifo Exists (운항 정보 존재) |
| **10** | Endorsement information (배서 정보) |
| **13** | Special remarks (특별 비고) |
| **14** | Fare quoted at time of booking (예약 시점 운임) |
| **15** | Fare calculation at time of ticketing (발권 시점 운임 계산) |
| **16** | Form of payment information (결제 수단 정보) |
| **17** | Ticketing information (발권 정보) |
| **27** | Ticketing time limit (발권 시한) |
| **32** | Endorsement box text information (배서란 텍스트) |
| **33** | Pricing/ticketing warning information (가격/발권 경고) |
| **37** | Horizontal Fare Calculation (수평 운임 계산) |
| **700** | Agent Alert (에이전트 알림) |
| **701** | Fare Warning Message (운임 경고 메시지) |
| **710** | Itinerary remarks (여정 비고) |
| **726** | Restrictions (제한 사항) |
| **727** | Fare rules token (운임 규정 토큰) |

### 1A Extended Information Type (Amadeus 확장 코드)

| 코드 | 설명 |
|------|------|
| **1A0** | Dynamic Discounted Fare (동적 할인 운임) |
| **1A1** | Specific Process: Validate Rules (규정 검증) |
| **1A3** | TKT by Fare Basis (Fare Basis별 발권) |
| **1AF** | Private Fare Used (사설 운임 사용) |
| **1AG** | Dynamic Discounted Fare Used (동적 할인 운임 사용) |
| **1AH** | HIP may apply / Unable to Verify (HIP 적용 가능/확인 불가) |
| **1AM** | Negotiated V2 Fares Used (V2 협정 운임 사용) |
| **1AP** | Lowest Possible Fare Override Used (최저 운임 오버라이드 사용) |
| **1AX** | Corporate Negotiated V2 Fares Used (기업 V2 협정 운임 사용) |
| **1P1** | Non Refundable Tickets are Non Refundable (환불 불가 항공권) |
| **1P3** | Penalty Applies (패널티 적용) |
| **1P4** | Penalty applies / Subject to cancellation/change penalty (취소/변경 패널티 대상) |
| **1P5** | Non refundable / Tickets are non-refundable before departure (출발 전 환불 불가) |

---

## 12. Fare Component 구조

### Fare Component Details Group

운임을 **개별 구성요소(Fare Component)**로 분해한 상세 정보. 복수 구간으로 이루어진 여정에서 구간별 운임 계산의 근거를 확인할 수 있다.

```
fareComponentDetailsGroup
├─ fareComponentInformation
│   ├─ fareBasisCode          ← Fare Basis Code (예: YOWKR)
│   ├─ fareComponentAmount    ← 구성요소 운임 금액
│   └─ fareComponentCurrency  ← 통화 코드
├─ segmentReference           ← 적용 구간 참조
├─ ruleInformation           ← 적용 규정 정보
└─ couponDetails             ← 쿠폰 상세
```

### Charge Category (운임 계산 방식)

Fare Component의 **운임 계산 방법**을 나타내는 코드.

| 코드 | 설명 |
|------|------|
| **700** | Lowest combination (최저 조합) |
| **701** | Mileage (마일리지 기반) |
| **702** | Routing (라우팅 기반) |
| **704** | Specific routing (특정 라우팅) |
| **706** | Mixed (혼합) |
| **710** | Zone (구역 기반) |
| **715** | Construction point (구성 지점) |
| **720** | Double open jaw (이중 오픈 조) |
| **725** | Single open jaw (단일 오픈 조) |
| **730** | Circle trip (써클 트립) |
| **735** | Round trip (왕복) |
| **740** | One way trip (편도) |
| **745** | Normal (일반) |

### Item Number Type -- Fare Component 참조

| 코드 | 설명 |
|------|------|
| **BND** | Bound (구간 단위) |
| **FC** | Fare component (운임 구성요소) |

---

## 13. Movement Type (여정 글로벌 지표)

여정의 **지리적 이동 경로**를 나타내는 코드. 국제 운임 계산에서 IATA 지역 간 이동 방향을 결정한다.

| 코드 | 설명 |
|------|------|
| **7AP** | AP -- 동반구(TC2)↔동반구(TC3) via 대서양·태평양 |
| **7AT** | AT -- 동반구(TC2&3)↔서반구(TC1) via 대서양 |
| **7CA** | CA -- Domestic Canada (캐나다 국내) |
| **7CT** | CT -- Circle trip (써클 트립, 같은 지역 내) |
| **7EH** | EH -- 동반구(TC2&3) 내 (PO, TS 제외) |
| **7FE** | FE -- 러시아 연방(우랄 서쪽)↔우크라이나↔TC3 직접 |
| **7PA** | PA -- 동반구(TC2&3)↔서반구(TC1) via 태평양 |
| **7RW** | RW -- Round the World (세계 일주) via 대서양·태평양 |
| **7TB** | TB -- Trans border (국경 횡단) |
| **7TS** | TS -- 동반구(TC2)↔동반구(TC3) via 시베리아 |
| **7US** | US -- Within US (미국 내) |
| **7WH** | WH -- 서반구(TC1) 내 |
| **WX** | Weather (기상) |

---

## 14. Conversion Rate (환율 정보)

### Conversion Type (환산 유형)

통화 환산 시 적용하는 **환산 기준**.

| 코드 | 설명 |
|------|------|
| **700** | Fares (운임 환산) |
| **704** | Equivalent fare (등가 운임 환산) |
| **705** | Origin of travel (여행 출발지 기준 환산) |

### Rate Type Qualifier (환율 유형)

| 코드 | 설명 |
|------|------|
| **700** | IATA clearinghouse rate (ICH) (IATA 결제소 환율) |
| **BBR** | Bankers buyer rate (은행 매입률) |
| **BSR** | Bankers seller rate (은행 매도율) |
| **ROE** | IATA ROE (IATA 환율 기준) |
| **USR** | User specified rate (사용자 지정 환율) |

---

## 15. 좌석 가용성 및 상품 구조

### Article Availability (좌석 가용 상태)

| 코드 | 설명 |
|------|------|
| **AVL** | Available (가용) |
| **C** | Closed (마감) |
| **L** | Waitlist only (대기 목록만) |
| **N** | Near to sell (판매 임박) |
| **R** | Request only (요청 시만) |
| **X** | Closed to arrival (도착 마감) |

### Product Details Qualifier (상품 상세 한정자)

| 코드 | 설명 |
|------|------|
| **ECO** | Economy Class (이코노미) |
| **ECP** | Economy Premium Class (프리미엄 이코노미) |
| **CLB** | Club Class (클럽 클래스) |
| **FST** | First Class (퍼스트 클래스) |
| **DOM** | Domestic (국내) |
| **INT** | International (국제) |
| **SCH** | Schengen (솅겐) |
| **SHU** | Shuttle (셔틀) |

### Item Description Identification (항목 설명)

| 코드 | 설명 |
|------|------|
| **1** | Change in class en route (경로 내 클래스 변경) |
| **2** | Invalid through class (무효 통과 클래스) |
| **700** | Non-dominant limit sales override (비주도 판매 한도 오버라이드) |
| **701** | Non-dominant capacity override (비주도 용량 오버라이드) |
| **702** | Dominant (주도적) |
| **N** | Night class (야간 클래스) |

---

## 16. Reference Qualifier (참조 한정자)

응답 메시지에서 **참조 정보의 유형**을 구분하는 코드.

| 코드 | 설명 |
|------|------|
| **1** | Unique passenger reference identification (고유 승객 참조) |
| **2** | Passenger sequence number (승객 순서 번호) |
| **5** | Passenger ticket number (승객 항공권 번호) |
| **7** | Date of birth (생년월일) |
| **700** | Exceptional PNR Security Identification (예외 PNR 보안) |
| **701** | Agency grouping identification (대리점 그룹 식별) |
| **702** | Ticketing data (발권 데이터) |
| **A** | Account/Product reference number (계좌/상품 참조) |
| **P** | Passenger/traveller reference number (승객 참조 번호) |
| **S** | Segment/service reference number (구간/서비스 참조) |
| **E** | Element (요소) |
| **PA** | Adult Passenger (성인 승객) |
| **PI** | Infant Passenger (유아 승객) |
| **T** | TST (Transitional Stored Ticket) |

---

## 17. Relation (구간 관계)

항공편 구간 간의 **관계 유형**을 나타내는 코드. 마케팅 항공사와 운항 항공사의 관계, 코드셰어 형태 등을 식별한다.

| 코드 | 설명 |
|------|------|
| **A** | Married on-line (온라인 결합) |
| **B** | Non-Dominant flight (비주도 항공편) |
| **C** | Potential marriage candidate (결합 후보) |
| **F** | First host cascading (첫 번째 호스트 캐스케이딩) |
| **I** | Married interline (인터라인 결합) |
| **L** | Last host cascading (마지막 호스트 캐스케이딩) |
| **M** | Middle host cascading (중간 호스트 캐스케이딩) |

---

## 18. Special Condition (특수 조건)

운임 적용 시 발생하는 **특수 조건/경고**.

| 코드 | 설명 |
|------|------|
| **FL** | Flight number restriction may apply (편명 제한 적용 가능) |
| **RB** | Missing or incorrect RBD (RBD 누락 또는 오류) |
| **RE** | Booking/ticketing conditions may apply (예약/발권 조건 적용 가능) |
| **RO** | User specified RBD has been overridden (사용자 지정 RBD 오버라이드됨) |
| **SR** | Other sales restrictions (기타 판매 제한) |

---

## 19. Sector/Subject Identification Qualifier

**패널티/할인 정보**의 출처를 구분하는 코드.

| 코드 | 설명 |
|------|------|
| **1** | Creator (생성자) |
| **2** | Owner (소유자) |
| **3** | Same as Originator details (발신자와 동일) |
| **700** | Penalty information (패널티 정보) |
| **701** | Discount information (할인 정보) |
| **OBF** | OB Fees Information (OB 수수료 정보) |
| **ZAP** | ZapOff Discount Information (ZapOff 할인 정보) |

---

## 20. 메시지 구조 용어

기술 문서에서 사용되는 **메시지 구조 정의 용어**.

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
| **ATAF** | Air Transport Association Fare | 항공 운송 협회 운임 |
| **ATPCO** | Airline Tariff Publishing Company | 항공 운임 데이터 제공 기관 |
| **DDF** | Dynamic Discounted Fare | 동적 할인 운임 |
| **EMD** | Electronic Miscellaneous Document | 부가서비스 전표 |
| **FBA** | Fare Basis Assignment | 운임 기저 지정 |
| **FOP** | Form of Payment | 결제 수단 |
| **GDS** | Global Distribution System | 글로벌 유통 시스템 |
| **HIP** | Higher Intermediate Point | 고차 중간 지점 (운임 계산 검증) |
| **IATA** | International Air Transport Association | 국제항공운송협회 |
| **ICH** | IATA Clearing House | IATA 결제소 |
| **MPM** | Maximum Permitted Mileage | 최대 허용 마일리지 |
| **NUC** | Neutral Unit of Construction | 중립 구성 단위 (통화 중립 운임) |
| **NUQ** | Neutral Unit of Quotation | 중립 견적 단위 |
| **OB Fee** | Optionally Billable Fee | 선택적 청구 수수료 |
| **PFC** | Passenger Facility Charge | 공항 시설 사용료 |
| **PNR** | Passenger Name Record | 승객 예약 기록 |
| **POS** | Point of Sale | 판매 지점 |
| **POT** | Point of Ticketing | 발권 지점 |
| **PTC** | Passenger Type Code | 승객 유형 코드 |
| **RBD** | Reservation Booking Designator | 예약 클래스 코드 |
| **ROE** | Rate of Exchange | 환율 |
| **SITI** | Sold Inside Ticketed Inside | 출발국 내 판매·발권 |
| **SITO** | Sold Inside Ticketed Outside | 출발국 판매, 타국 발권 |
| **SOTI** | Sold Outside Ticketed Inside | 타국 판매, 출발국 발권 |
| **SOTO** | Sold Outside Ticketed Outside | 타국 판매·발권 |
| **TC1** | Traffic Conference 1 | IATA 교통 회의 구역 1 (아메리카) |
| **TC2** | Traffic Conference 2 | IATA 교통 회의 구역 2 (유럽·아프리카·중동) |
| **TC3** | Traffic Conference 3 | IATA 교통 회의 구역 3 (아시아·태평양) |
| **TPM** | Ticketed Point Mileage | 발권 지점 마일리지 |
| **TST** | Transitional Stored Ticket | 과도 저장 항공권 |
| **WBS** | Web-Based Services | 웹 기반 서비스 |
| **ZP** | Zone Pricing | 구역 기반 가격 책정 |

---

## 참고

- [WBS Integration Flow - Step 3](amadeus-wbs-integration-flow.md)
- [Fare_MasterPricerTravelBoardSearch](master-pricer-travelboard-search.md)
- [Fare_InformativePricingWithoutPNR](fare-informative-pricing.md)
- [Fare_CheckRules](fare-check-rules.md)
