# PNR_AddMultiElements (PNR 요소 추가/수정)

Amadeus `PNR_AddMultiElements` API(버전 22.1)의 기술 용어를 정리한 문서입니다. PNR(Passenger Name Record)에 승객 이름, 여정, SSR, OSI, 발권 정보, Remarks, 결제 수단 등 PNR을 구성하는 거의 모든 요소를 생성하거나 수정하는 API입니다.

---

!!! info "WBS Integration Flow Step 8"
    이 API는 [WBS Integration Flow](amadeus-wbs-integration-flow.md)의 **예약 생성 단계 Step 8**에 해당합니다.

---

## 1. 개요

**PNR_AddMultiElements**는 항공 예약 시스템(GDS)에서 PNR을 생성하고 관리하는 Amadeus의 핵심 API입니다.

- **Query**: 승객 정보, 여정(항공 구간), 데이터 요소(SSR, OSI, Remarks, FOP, TK 등)를 PNR에 추가하거나 수정
- **Reply**: 처리 결과로 완성된 PNR 이미지(PNR_Reply) 반환

| 항목 | 내용 |
|------|------|
| API 명 | `PNR_AddMultiElements` |
| 플로우 단계 | Step 8 -- 예약 생성 |
| 목적 | PNR에 승객 정보, SSR, 연락처 등 추가 후 저장 (Commit) |
| 이전 단계 | [Air_SellFromRecommendation](air-sell-from-recommendation.md) |
| 다음 단계 | [FOP_CreateFormOfPayment](fop-create-form-of-payment.md) |

!!! info "용도"
    - PNR 최초 생성 (승객 이름 + 여정 + 필수 요소 추가 후 EOT)
    - 기존 PNR에 SSR, OSI, Remarks, FOP, TK 등 요소 추가
    - 승객 이름, 연락처, 좌석 요청 등 기존 요소 수정 및 삭제

!!! note "SBR vs PNR"
    Amadeus에서 SBR(Standard Booking Record)은 PNR의 내부 명칭입니다. Reply 구조에서 `sbrPOSDetails`, `sbrCreationPosDetails` 등의 필드가 PNR 생성/수정 관련 Point of Sale 정보를 담습니다.

---

## 2. Query 구조 (요청)

Query 메시지 참조: `PNR_AddMultiElements 22.1.1A`

### 최상위 구조

| Entity | Structure | St | Rep | 설명 |
|--------|-----------|-----|-----|------|
| `reservationInfo` | Reservation control information | C | 1 | 기존 PNR 참조 시 레코드 로케이터 지정 |
| `pnrActions` | Optional PNR actions | M | 1 | PNR에 적용할 액션 코드 (EOT, Ignore 등) |
| `travellerInfo` | Group | C | 100 | 승객 정보 그룹 (최대 100명) |
| `originDestinationDetails` | Group | C | 50 | 여정(O&D) 그룹 (최대 50개) |
| `dataElementsMaster` | Group | C | 1 | 데이터 요소 마스터 (DUM 구분자 + 개별 요소) |

---

### 2-1. pnrActions (PNR 액션)

PNR 처리 방식을 지정하는 필수 구조입니다.

| 필드 | 설명 |
|------|------|
| `optionCode` | PNR 처리 옵션 코드 (n..3). 최대 40개 반복. → [Option Code 코드](#option-code-pnr-액션) 참조 |

---

### 2-2. travellerInfo (승객 정보)

승객 이름 및 기본 정보를 담는 그룹입니다.

**elementManagementPassenger** (Element Management Segment)

| 필드 | 설명 |
|------|------|
| `reference.qualifier` | 참조 유형. `OT`=객체 타투, `PT`=승객 타투, `PR`=승객 클라이언트 참조, `ST`=구간 타투 |
| `reference.number` | 참조 번호 (an..5). 기존 PNR 요소 식별 |
| `segmentName` | PNR 세그먼트/요소 이름 코드 (an..3). 예: `NM`(이름), `SSR`, `OSI` → [세그먼트 코드](#pnr-세그먼트-코드) 참조 |

**passengerData > travellerInformation** (승객 이름 정보)

| 필드 | 설명 |
|------|------|
| `traveller.surname` | 승객 성 (an..57). 그룹의 경우 그룹명 |
| `traveller.qualifier` | 승객 유형. `G`=그룹 |
| `traveller.quantity` | 그룹 인원 수 (n..2) |
| `passenger.firstName` | 이름 + 호칭 (an..56). 예: `GILDONG/MR` |
| `passenger.type` | 승객 타입 코드 (an..3). 예: `ADT`, `CHD`, `INF` → [승객 타입 코드](#승객-타입-코드) 참조 |
| `passenger.infantIndicator` | 유아 동반 지시자. `1`=유아 동반 |

**passengerData > dateOfBirth** (생년월일)

| 필드 | 설명 |
|------|------|
| `dateAndTimeDetails.date` | 생년월일 (DDMMYYYY 형식). 예: `01121990` |

**enhancedPassengerData** (확장 승객 정보)

UTF-8 인코딩 이름(다국어 이름, 원어명) 처리에 사용하는 대체 구조입니다.

| 필드 | 설명 |
|------|------|
| `enhancedTravellerInformation` | UTF-8 이름 전송 시 반드시 사용. `travellerInformation` 대신 이 구조를 사용 |
| `nameDetails.nameType` | 이름 유형 코드. Native name, Universal name 등 구분 |
| `nameDetails.surname` | 성 (an..57) |
| `nameDetails.givenName` | 이름 (an..56) |
| `nameDetails.title` | 호칭. 이름 필드와 분리 전송 시 사용 (an..70) |

---

### 2-3. originDestinationDetails (여정)

항공 구간 정보를 담는 그룹입니다.

**originDestination** (출발/도착지)

| 필드 | 설명 |
|------|------|
| `origin` | 출발 도시/공항 코드 (a3). IATA 코드 |
| `destination` | 도착 도시/공항 코드 (a3). IATA 코드 |

!!! note "Connected Segment"
    `originDestination.origin`이 공백이 아닌 경우, 해당 ODI(Origin Destination Indicator) 이후에 오는 구간들은 연결(Connected) 구간으로 처리됩니다. 최대 6개의 TVL 구간이 하나의 ODI에 연결될 수 있습니다.

**itineraryInfo > elementManagementItinerary**

| 필드 | 설명 |
|------|------|
| `segmentName` | 세그먼트 유형. `AIR`=항공, `ARN`=도착 미정, `SUR`=지상 구간 |

**itineraryInfo > airAuxItinerary > travelProduct** (항공편 정보)

| 필드 | 설명 |
|------|------|
| `product.depDate` | 출발일 (n6, DDMMYY 형식) |
| `product.depTime` | 출발시간 (n4, HHMM 형식) |
| `product.arrDate` | 도착일 (n6) |
| `product.arrTime` | 도착시간 (n4) |
| `boardpointDetail.cityCode` | 출발 공항/도시 코드 (an..5) |
| `offpointDetail.cityCode` | 도착 공항/도시 코드 (an..5) |
| `company.identification` | 항공사 코드 (an..3). IATA 2자리 코드 |
| `productDetails.identification` | 편명 번호 또는 `OPEN`/`ARNK` |
| `productDetails.classOfService` | 예약 클래스 (RBD, a1). 예: Y, B, H, M |
| `productDetails.subtype` | 편명 접미사 (a1). A, B, C, D, E |
| `productDetails.description` | 야간 클래스 지시자. `N`=Night class |

**messageAction** (세그먼트 액션)

| 필드 | 설명 |
|------|------|
| `business.function` | 세그먼트 유형 코드 (business function). 항공=`1` → [Business Function 코드](#business-function-코드) 참조 |

**relatedProduct** (예약 관련 정보)

| 필드 | 설명 |
|------|------|
| `quantity` | 예약 인원 수 (n..2) |
| `status` | 예약 상태 코드 (a..2). 예: `NN`, `HK`, `SS`, `LL` → [세그먼트 상태 코드](#세그먼트-상태-코드) 참조 |

**selectionDetailsAir** (판매 방식)

| 필드 | 설명 |
|------|------|
| `selection.option` | 판매 레벨 코드. 예: `0`=Long Sell, `1`=Short Sell, `P2`=Amadeus Access → [Option 코드](#option-코드-판매-방식) 참조 |

---

### 2-4. dataElementsMaster (데이터 요소)

DUM(Dummy) 구분자 이후에 오는 모든 PNR 데이터 요소 그룹입니다.

**dataElementsIndiv** (개별 데이터 요소, 최대 250개)

| 필드 | 설명 |
|------|------|
| `elementManagementData` | 요소 관리 세그먼트. `segmentName`으로 요소 유형 식별 |
| `serviceRequest` | SSR 요소. 특별 서비스 요청 |
| `miscellaneousRemark` | Remarks 요소. RM, RC, RI 등 각종 비고 |
| `ticketElement` | TK 요소. 발권 정보 (TK TL/TK OK/TK XL 등) |
| `freetextData` | AP, OS 등 자유 형식 텍스트 요소 |
| `formOfPayment` | FP 요소. 결제 수단 |
| `frequentTravellerData` | FF 요소. 상용 고객 프로그램 정보 |
| `seatGroup` | 좌석 요청 요소 |
| `fareElement` | 운임 요소 (FN, FE, FS 등) |
| `commission` | FM 요소. 수수료 |
| `tourCode` | FT 요소. 투어 코드 |
| `ticketingCarrier` | FV 요소. 발권 항공사 지시자 |
| `formOfPayment` + `fopExtension` | FP 요소 + FOP 확장 정보 |
| `accessLevel` | ES 요소. 개별 PNR 보안 설정 |

---

## 3. Reply 구조 (응답)

Reply 메시지 참조: `PNR_Reply 22.1.1A`

### 최상위 구조

| Entity | Structure | St | Rep | 설명 |
|--------|-----------|-----|-----|------|
| `generalErrorInfo` | Group | C | 99 | 일반 오류 정보 |
| `pnrHeader` | Group | C | 198 | PNR 헤더 정보 |
| `travellerInfo` | Group | C | 100 | 승객 이름 요소 |
| `originDestinationDetails` | Group | C | 50 | 여정(항공 구간) 정보 |
| `dataElementsMaster` | Group | C | 1 | PNR 데이터 요소 전체 |

---

### 3-1. pnrHeader (PNR 헤더)

| Entity | 설명 |
|--------|------|
| `reservationInfo` | 레코드 로케이터(PNR 번호) 및 항공사 RCI |
| `securityInformation` | PNR 보안 정보 (오너십, 접근 권한) |
| `queueInformations` | 큐 배치 정보 (오피스, 큐 번호, 카테고리) |
| `numberOfUnits` | 승객 수 등 단위 정보 |
| `pnrType` | PNR 특수 유형 코드 |
| `pnrHeaderTag` | PNR 헤더 태그 (TST, TSM, DCS, MAR 등) |
| `historyData` | PNR 이력 데이터 (최대 999개) |

**sbrPOSDetails / sbrCreationPosDetails / sbrUpdatorPosDetails** (Point of Sale 정보)

| 구조 | 설명 |
|------|------|
| `sbrPOSDetails` | PNR 소유 오피스의 Point of Sale 정보 |
| `sbrCreationPosDetails` | PNR 생성 오피스 정보 |
| `sbrUpdatorPosDetails` | 최종 수정 오피스 정보 |

**technicalData** (기술 데이터)

| 필드 | 설명 |
|------|------|
| `enveloppeNumberData` | 마지막 EOT 시 발급된 Envelope 번호 |
| `purgeDateData` | PNR 자동 삭제(Purge) 예정일 |

---

### 3-2. travellerInfo (승객 이름 - Reply)

| Entity | 설명 |
|--------|------|
| `elementManagementPassenger` | 요소 참조 정보 (타투 번호, 세그먼트명) |
| `passengerData.travellerInformation` | 승객 이름, 타입, 인원 수 |
| `passengerData.groupCounters` | 그룹 예약 시 예약/취소/분리 인원 수 |
| `passengerData.dateOfBirth` | 유아/소아 생년월일 |
| `enhancedPassengerData` | UTF-8 인코딩 이름 (다국어) |
| `nameError` | 이름 오류 정보 |

---

### 3-3. itineraryInfo (여정 - Reply)

| Entity | 설명 |
|--------|------|
| `elementManagementItinerary` | 구간 요소 참조 정보 |
| `travelProduct` | 출발/도착지, 편명, 날짜/시간, 클래스 |
| `itineraryMessageAction` | 세그먼트 액션 (business function) |
| `itineraryReservationInfo` | 항공사 예약 레코드 로케이터 |
| `relatedProduct` | 예약 인원 수, 상태 코드 |
| `elementsIndicators` | 세그먼트 과금/면제 상태 |
| `flightDetail` | 추가 항공편 정보 (기재, 운항 시간 등) |
| `cabinDetails` | 캐빈 등급 상세 |
| `carbonDioxydeInfo` | CO2 배출량 정보 |
| `selectionDetails` | 판매 방식 지시자 |

---

## 4. 주요 데이터 구조

### 4-1. SSR (Special Service Request)

특별 서비스 요청 정보를 담는 구조입니다.

**serviceRequest 구조**

| 필드 | 설명 |
|------|------|
| `ssr.type` | SSR 코드 4자리. 예: `WCHR`, `VGML`, `BLND` → [주요 SSR 코드](#주요-ssr-코드) 참조 |
| `ssr.status` | 서비스 상태 코드. 예: `HK`=Confirmed, `NN`=Need, `UC`=Unable |
| `ssr.quantity` | 수량 (n..2) |
| `ssr.companyId` | 서비스를 제공하는 항공사 코드 (an..3) |
| `ssr.freetext` | SSR 자유 텍스트 (an..70). 상세 정보 |
| `ssr.segmentNumber` | 연결된 항공 구간 번호 |

---

### 4-2. FP (Form of Payment / 결제 수단)

**formOfPayment 구조**

| 필드 | 설명 |
|------|------|
| `fop.type` | 결제 수단 코드 → [FOP 코드](#form-of-payment-코드) 참조 |
| `fop.amount` | 결제 금액 |
| `fop.creditCardCode` | 신용카드 회사 코드 (예: VI, CA, AX, DC) |
| `fop.accountNumber` | 카드 번호 또는 계좌번호 |
| `fop.expiryDate` | 카드 유효기간 (MMYY) |

**fopExtension** (FOP 확장 정보, 최대 3개 반복)

| 필드 | 설명 |
|------|------|
| 추가 FOP 상세 | 복합 결제(복수 카드 등) 시 사용 |

---

### 4-3. TK (Ticket Element / 발권 요소)

**ticketElement 구조**

| 필드 | 설명 |
|------|------|
| `ticket.indicator` | 발권 지시자 코드 → [TK 지시자 코드](#tk-발권-지시자) 참조 |
| `ticket.date` | 발권 기한일 또는 발권일 (DDMMYY 형식) |
| `ticket.time` | 발권 기한 시간 (HHMM) |
| `ticket.officeId` | 발권 오피스 ID |
| `ticket.airlineCode` | 발권 항공사 코드 |

---

### 4-4. AP (Contact Element / 연락처)

**freetextData > freetextDetail** 구조로 전송합니다.

| 필드 | 설명 |
|------|------|
| `freetextDetail.subjectQualifier` | 텍스트 주제 코드. 연락처는 `3`(Literal text) |
| `freetextDetail.type` | 연락처 유형 코드. 예: `P`=전화, `E`=이메일, `M`=휴대폰, `F`=팩스 |
| `longFreetext` | 실제 연락처 정보 텍스트 (an..199) |

---

### 4-5. OS (Other Service Information / OSI)

항공사에 전달하는 정보성 메시지입니다.

| 필드 | 설명 |
|------|------|
| `freetextDetail.type` | OSI 유형 코드 |
| `freetextDetail.companyId` | 대상 항공사 코드 |
| `longFreetext` | OSI 텍스트 내용 (an..199) |

---

### 4-6. Remarks (비고 요소)

**miscellaneousRemark 구조**

| 필드 | 설명 |
|------|------|
| `remarks.type` | Remark 유형 코드 → [Remark 유형 코드](#remark-유형-코드) 참조 |
| `remarks.category` | Remark 카테고리 |
| `remarks.freetext` | Remark 내용 텍스트 |

---

### 4-7. FF (Frequent Traveller / 상용 고객)

**frequentTravellerData 구조**

| 필드 | 설명 |
|------|------|
| `frequentTraveller.companyId` | 항공사 코드 또는 파트너사 코드 |
| `frequentTraveller.membershipNumber` | 마일리지 회원번호 |
| `frequentTraveller.tierLevel` | 회원 등급 |

---

### 4-8. FM (Commission / 수수료)

**commission 구조**

| 필드 | 설명 |
|------|------|
| `commission.type` | 수수료 유형. `P`=퍼센트, `A`=금액 |
| `commission.percentage` | 수수료율 (%) |
| `commission.amount` | 수수료 금액 |
| `commission.currencyCode` | 통화 코드 |

---

### 4-9. reservationInfo (레코드 로케이터)

PNR 식별을 위한 예약 참조 정보입니다.

| 필드 | 설명 |
|------|------|
| `reservation.companyId` | 항공사 또는 GDS 코드. Amadeus는 `1A` |
| `reservation.controlNumber` | 레코드 로케이터 (PNR 번호, an..19). 예: `ABCDEF` |

---

## 5. 주요 코드셋

### Option Code (PNR 액션)

`pnrActions.optionCode`에 사용하는 코드입니다. 복수 지정 가능합니다.

| 코드 | 의미 |
|------|------|
| `0` | No special processing (특별 처리 없음) |
| `10` | End transact (ET) - PNR 저장 후 종료 |
| `11` | End transact with retrieve (ER) - PNR 저장 후 재조회 |
| `12` | End transact and change advice codes (ETK) |
| `13` | End transact with retrieve and change advice codes (ERK) |
| `14` | End transact split PNR (EF) - PNR 저장 및 분리 |
| `15` | Cancel itinerary for all AXR-connected PNRs and ET (ETX) |
| `16` | Cancel itinerary for all AXR-connected PNRs and ER (ERX) |
| `20` | Ignore (IG) - 변경사항 취소 |
| `21` | Ignore and retrieve (IR) - 취소 후 재조회 |
| `30` | Show warnings at first EOT |
| `50` | Reply with short message (축약 응답) |
| `267` | Stop EOT if segment sell error |

---

### 세그먼트 상태 코드

`relatedProduct.status`에 사용하는 주요 코드입니다.

| 코드 | 의미 |
|------|------|
| `HK` | Holding Confirmed (확약) |
| `HL` | Holding Waitlist (대기 확약) |
| `HN` | Holding Need (요청 중) |
| `HX` | Have Cancelled (취소됨) |
| `KK` | Confirming (확약 응답) |
| `LK` | Guaranteed Sell (보장 판매) |
| `LL` | Please Waitlist (대기 요청) |
| `NN` | Need Segment (구간 필요) |
| `NO` | No Action Taken |
| `NS` | Infant, No Seat (유아, 좌석 없음) |
| `SS` | Sold from Availability (가용성 기반 판매) |
| `UC` | Unable - Closed (판매 불가, 마감) |
| `UN` | Unable (판매 불가) |
| `WL` | Waitlisted (대기자) |
| `XL` | Cancel Waitlisted Segment (대기 취소) |
| `XX` | Cancel Confirmed/Requested/Waitlisted (확약/요청/대기 취소) |

!!! note "상태 코드 흐름"
    일반적인 흐름: `NN` (요청) → `KK` (확약 응답) → `HK` (확약 보유)

---

### 주요 SSR 코드

`serviceRequest.ssr.type`에 사용하는 4자리 코드입니다.

**신체적 지원**

| 코드 | 의미 |
|------|------|
| `WCHR` | Wheelchair - Ramp (탑승구까지 휠체어) |
| `WCHS` | Wheelchair - Steps (계단 이동 불가) |
| `WCHC` | Wheelchair - Cabin Seat (기내 좌석까지 지원) |
| `BLND` | Blind Passenger (시각 장애) |
| `DEAF` | Deaf Passenger (청각 장애) |
| `MEDA` | Medical Case (의료적 도움 필요) |
| `STCR` | Stretcher (들것 운반) |

**식이 요청**

| 코드 | 의미 |
|------|------|
| `VGML` | Vegetarian Meal (채식) |
| `AVML` | Asian Vegetarian Meal (아시아 채식) |
| `HNML` | Hindu Meal (힌두 식단) |
| `MOML` | Moslem Meal (이슬람 할랄 식단) |
| `KSML` | Kosher Meal (유대교 코셔 식단) |
| `DBML` | Diabetic Meal (당뇨 식단) |
| `LSML` | Low Salt Meal (저염 식단) |
| `CHML` | Child Meal (어린이 식사) |
| `BBML` | Baby Meal (유아 식사) |
| `SFML` | Seafood Meal (해산물) |
| `FPML` | Fruit Platter Meal (과일 식단) |

**승객 유형/운영**

| 코드 | 의미 |
|------|------|
| `UMNR` | Unaccompanied Minor (비동반 소아) |
| `PETC` | Pet in Cabin (기내 애완동물) |
| `AVIH` | Animal in Hold (화물칸 동물) |
| `EXST` | Extra Seat (추가 좌석) |
| `CBBG` | Cabin Baggage (기내 수하물) |
| `FQTV` | Frequent Traveller (상용 고객 번호) |
| `DOCS` | Travel Document (여행 서류. 여권 정보) |
| `DOCA` | Address (목적지 주소. API 정보) |
| `DOCO` | Other Document (기타 서류. 비자 등) |
| `OTHS` | Other Service (기타 서비스) |

---

### Form of Payment 코드

`formOfPayment.fop.type`에 사용하는 코드입니다.

| 코드 | 의미 |
|------|------|
| `CA` | Cash (현금) |
| `CC` | Credit Card (신용카드) |
| `CK` | Check (수표) |
| `MS` | Miscellaneous (기타) |
| `NR` | Non-refundable (환불 불가) |
| `PT` | Prepaid Ticket Advice (PTA) |
| `GR` | Government Transportation Request |
| `UN` | United Nations Transportation Request |
| `INV` | Invoice (인보이스) |
| `AGT` | Agent (대리점) |
| `BA` | Bank Account (은행 계좌) |

---

### TK (발권 지시자)

Ticket Element의 발권 유형을 지정합니다.

| 코드 | 의미 |
|------|------|
| `TL` | Ticket Time Limit (발권 기한 설정). `TK TL DDMMYY` |
| `OK` | Ticket OK (발권 완료 확인). `TK OK` |
| `XL` | Ticket Cancel (발권 기한 취소). `TK XL` |
| `RF` | Received From (입력자 정보). PNR 저장 시 필수 |

---

### Remark 유형 코드

`miscellaneousRemark.remarks.type`에 사용하는 코드입니다.

| 코드 | 의미 |
|------|------|
| `RM` | General Remark (일반 비고) |
| `RC` | Confidential Remark (기밀 비고) |
| `RI` | Invoice Remark (인보이스 비고) |
| `RIR` | Itinerary Remark (여정 비고) |
| `RIF` | Invoice Remark (인보이스 전용) |
| `RII` | Itinerary & Invoice Remark (여정+인보이스) |
| `RQ` | Quality Control Remark (품질 관리 비고) |
| `GNL` | General Comment |
| `GTE` | Gate Comment |

---

### Business Function 코드

세그먼트 유형을 분류하는 코드입니다. `messageAction.business.function`에 사용합니다.

| 코드 | 의미 |
|------|------|
| `1` | Air Provider (항공 구간) |
| `2` | Car Provider |
| `3` | Hotel Provider |
| `4` | Ferry (페리) |
| `5` | Cruise (크루즈) |
| `6` | Rail (철도) |
| `7` | Tour (투어) |
| `10` | Air Taxi (에어택시) |
| `17` | Charter (전세기) |
| `36` | Prepaid Ticket (선불 항공권) |
| `39` | Ticket (항공권) |

---

### PNR 세그먼트 코드

`elementManagementData.segmentName`에 사용하는 코드입니다.

**승객/이름 요소**

| 코드 | 의미 |
|------|------|
| `NM` | Name Element (이름) |
| `NG` | Group Name Element (그룹 이름) |
| `NZ` | Non-Commercial PNR Name Element |

**여정 요소**

| 코드 | 의미 |
|------|------|
| `AIR` | Air Segment (항공 구간) |
| `SUR` | Ground Transportation Segment (지상 구간) |
| `TRN` | Train Amtrak Segment (철도) |
| `CRU` | Cruise Distribution Segment |
| `CCR` | Automated Car Auxiliary Segment |
| `HHL` | Automated Hotel Auxiliary Segment |
| `INS` | Insurance Segment |

**발권/운임 요소**

| 코드 | 의미 |
|------|------|
| `FA` | Ticket Number - Automated Tickets |
| `FB` | AIR Sequence Number |
| `FD` | Fare Discount (FD 요소) |
| `FE` | Endorsements/Restrictions (FE 요소) |
| `FH` | Manual Document Registration (FH 요소) |
| `FI` | Automated Invoice Number |
| `FM` | Commission (FM 요소) |
| `FN` | Transmission Control Number |
| `FO` | Original Issue / Issue in Exchange For |
| `FP` | Form of Payment (결제 수단) |
| `FS` | Miscellaneous Ticketing Information |
| `FT` | Tour Code (투어 코드) |
| `FV` | Ticketing Carrier Designator (발권 항공사) |
| `TK` | Ticket Element (발권 요소) |

**연락처/서비스 요소**

| 코드 | 의미 |
|------|------|
| `AP` | Contact Element (연락처) |
| `OS` | Other Special Information (OSI) |
| `SSR` | Special Service Request |
| `AB` | Billing Address |
| `AM` | Mailing Address |
| `AQ` | Address Verification |
| `AI` | Accounting Information |

**비고 요소**

| 코드 | 의미 |
|------|------|
| `RM` | General Remark |
| `RC` | Confidential Remark |
| `RIF` | Invoice Remark |
| `RIR` | Itinerary Remark |
| `RQ` | Quality Control Remark |
| `RF` | Receive From |
| `RR` | Associated Cross Reference Record |

**기타 요소**

| 코드 | 의미 |
|------|------|
| `ES` | Individual PNR Security (개별 보안) |
| `OP` | Option Element (옵션) |
| `SP` | Split Party Element (분리) |
| `SK` | Special Keywords |
| `NFP` | Neutral Form of Payment |
| `PL` | Priority Line |
| `WA` | Warning Line |

---

### Option 코드 (판매 방식)

`selectionDetailsAir.selection.option`에 사용하는 코드입니다.

| 코드 | 의미 |
|------|------|
| `0` | Long Sell (풀 정보 판매) |
| `1` | Short Sell (간략 판매) |
| `P2` | Amadeus Access (아마데우스 다이렉트) |
| `P3` | Negotiated Space (네고 공간) |
| `P4` | Direct Access (항공사 직접) |
| `P10` | Standard Access |
| `BBY` | Best Buy |
| `MPE` | Master Pricer Expert |

---

### 승객 타입 코드

`passengerData.travellerInformation.passenger.type`에 사용하는 주요 코드입니다.

| 코드 | 의미 |
|------|------|
| `ADT` | Adult (성인, 만 12세 이상) |
| `CHD` | Child (소아, 만 2~11세) |
| `INF` | Infant (유아, 만 2세 미만, 좌석 없음) |
| `INS` | Infant with Seat (유아, 좌석 있음) |
| `STU` | Student (학생) |
| `SRC` | Senior Citizen (시니어) |
| `MIL` | Military (군인) |
| `UAM` | Unaccompanied Minor (비동반 소아) |
| `SEA` | Seaman (선원) |
| `MED` | Medical (의료 목적) |

---

### 좌석 특성 코드

좌석 요청 시 사용하는 코드입니다.

| 코드 | 의미 |
|------|------|
| `A` | Aisle Seat (복도 쪽) |
| `W` | Window Seat (창가) |
| `B` | Seat with Bassinet Facility (바시넷) |
| `K` | Bulkhead Seat (격벽 앞, 칸막이) |
| `H` | Handicapped/Incapacitated Seat |
| `I` | Seat for Adult with Infant |
| `U` | Unaccompanied Minor Seat |
| `E` | Exit Row Seat (비상구 좌석) |
| `CH` | Chargeable Seat (유료 좌석) |
| `MA` | Medically OK for Travel |
| `N` | No Smoking Seat |

---

## 6. 약어 모음

| 약어 | 풀네임 | 설명 |
|------|--------|------|
| API | Advance Passenger Information | 사전 승객 정보 (여권, 주소 등) |
| ARK | Arrival Unknown | 도착 미정 구간 |
| ARNK | Arrival Unknown | 여정 중 이동 수단 불명 구간 (Surface 구간) |
| AXR | Associated Cross Reference | PNR 간 교차 참조 연결 |
| C | Conditional | 조건부 필드 (옵션) |
| DUM | Dummy Segment | Query에서 데이터 요소 시작을 알리는 구분자 |
| EOT | End of Transaction | PNR 저장 트랜잭션 종료 |
| ET | End Transact | PNR 저장 (=EOT) |
| FOP | Form of Payment | 결제 수단 |
| FF | Frequent Flyer | 상용 고객 |
| GDS | Global Distribution System | 전 세계 예약 유통 시스템 |
| IG | Ignore | 미저장 취소 |
| INF | Infant | 유아 (만 2세 미만) |
| IR | Ignore and Retrieve | 취소 후 재조회 |
| M | Mandatory | 필수 필드 |
| M* | Mandatory for implementation | 구현 시 필수 |
| O&D | Origin and Destination | 출발지-목적지 쌍 |
| ODI | Origin Destination Indicator | 출발-도착 지시자 (연결 구간 그룹) |
| OP | Option | 옵션 요소 (발권 기한 등) |
| OSI | Other Service Information | 기타 서비스 정보 |
| PNR | Passenger Name Record | 승객 예약 기록 |
| POS | Point of Sale | 판매 지점 (발권 오피스 정보) |
| PTA | Prepaid Ticket Advice | 선불 항공권 지시 |
| RBD | Reservation Booking Designator | 예약 등급 지시자 (클래스 코드) |
| RCI | Reservation Control Information | 예약 관리 정보 (레코드 로케이터 포함) |
| RFIC | Reason for Issuance Code | 발행 이유 코드 (IATA 서비스 수수료) |
| RFISC | Reason for Issuance Sub-Code | 발행 이유 세부 코드 |
| SBR | Standard Booking Record | Amadeus 내부의 PNR 명칭 |
| SSR | Special Service Request | 특별 서비스 요청 |
| TK | Ticket Element | 발권 요소 (TL/OK/XL) |
| TSM | Transitional Stored Miscellaneous | 임시 저장 기타 서비스 |
| TST | Transitional Stored Ticket | 임시 저장 항공권 |
| TVL | Travel Segment | 여행 구간 세그먼트 |
| UM | Unaccompanied Minor | 비동반 소아 |

---

## 참고

- [WBS Integration Flow - Step 8](amadeus-wbs-integration-flow.md)
- [Air_SellFromRecommendation 용어집](air-sell-from-recommendation.md)
- [FOP_CreateFormOfPayment 용어집](fop-create-form-of-payment.md)
- [PNR_Retrieve 용어집](pnr-retrieve.md)
- [BSP 정산 가이드](bsp-settlement-guide.md)
