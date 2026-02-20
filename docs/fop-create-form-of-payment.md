# FOP_CreateFormOfPayment

저장된 PNR에 **결제 수단(Form of Payment)을 등록**하는 API 용어집입니다.
발권 전 반드시 FOP가 PNR에 연결되어야 하며, 신용카드 결제 시 FortKnox 토큰화와 3DS 인증을 처리합니다.

> 기반 문서: FOP_CreateFormOfPayment 19.2.1A Technical Reference

---

!!! info "WBS Integration Flow Step 9"
    이 API는 [WBS Integration Flow](amadeus-wbs-integration-flow.md)의 **예약 생성 단계 Step 9**에 해당합니다.

---

## 1. 개요

### FOP_CreateFormOfPayment

Amadeus GDS에서 PNR에 **결제 수단(Form of Payment)을 생성**하는 API. 신용카드, 현금, 수표 등 다양한 결제 수단을 등록하고, 신용카드의 경우 FortKnox 토큰화를 통해 카드 정보를 안전하게 저장한다. 최대 127개의 FOP를 하나의 PNR에 등록할 수 있다.

```
[Query: FOP 등록 요청]
  결제 수단 유형, 카드 정보, 승객 연결, OB Fee 옵션
       |
       v
  Amadeus FOP Engine
  +-- FortKnox 토큰화 (CC인 경우)
  +-- 3DS 인증 (필요 시)
  +-- 사기 방지 스크리닝 (선택)
       |
       v
[Reply: FOP 등록 결과]
  fopDescription x N개
  +-- fopReference: FP/SFP 타투 번호
  +-- mopDescription: 결제 수단 상세
  |   +-- fopPNRDetails: FOP 코드, 빌링 코드
  |   +-- mopDetails: 신용카드/현금 등 상세
  +-- pnrSupplementaryData: FOP 스위치/데이터
  +-- paymentModule: 결제 트랜잭션 전체 데이터
```

### Query / Reply 구조

| 구분 | 메시지 | 설명 |
|------|--------|------|
| **Query** | `FOP_CreateFormOfPayment 19.2.1A` | PNR에 결제 수단 생성 (요청) |
| **Reply** | `FOP_CreateFormOfPaymentReply 19.2.1A` | FOP 데이터 읽기 (응답) |

| 항목 | 내용 |
|------|------|
| API 명 | `FOP_CreateFormOfPayment` |
| 플로우 단계 | Step 9 -- 예약 생성 |
| 목적 | 신용카드 등 결제 수단 등록 및 FortKnox 토큰 발급 |
| 이전 단계 | [PNR_AddMultiElements](pnr-add-multi-elements.md) |
| 다음 단계 | [Fare_PricePNRWithBookingClass](fare-price-pnr-with-booking-class.md) |

---

## 2. Query 주요 구조 (FOP 등록 요청)

### transactionContext (Transaction Information for Ticketing)

**발권 트랜잭션 정보**를 전달하는 구조.

| 필드 | 형식 | 설명 |
|------|------|------|
| `transactionDetails.code` | an..4 | 트랜잭션 코드 (TKTA, RFND 등) |
| `transactionDetails.issueIndicator` | an1 | 발권 유형 (F=최초 발행, R=재발행 등) |

#### Transaction Code (트랜잭션 코드)

| 코드 | 설명 |
|------|------|
| **CANR** | Cancellation Refund, 취소 환불 |
| **MCOA** | MCO Automatic, MCO 자동 발행 |
| **MCOM** | MCO Manual, MCO 수동 발행 |
| **MDnn** | Manual Document, 수동 문서 (nn=번호) |
| **PTAM** | PTA Manual, PTA 수동 발행 |
| **ARVM** | Agent Receipt Voucher Manual, 에이전트 수령 바우처 |
| **TKTA** | Ticket Automatic, 자동 발권 |
| **TKTB** | Ticket Bulk, 대량 발권 |
| **TKTM** | Ticket Manual, 수동 발권 |
| **TKTT** | Ticket Transitional, 전환 발권 |
| **TORM** | Tour Order Manual, 투어 오더 수동 |
| **XSBA** | Exchange Super Bulk Automatic, 대량 교환 자동 |
| **XSBM** | Exchange Super Bulk Manual, 대량 교환 수동 |
| **ACMR** | Agency Credit Memo Refund, 대리점 크레딧 메모 환불 |
| **RENA** | Reissue Non-Automatic, 비자동 재발행 |
| **RENM** | Reissue Non-Manual, 비수동 재발행 |
| **RFND** | Refund, 환불 |
| **ACMA** | Agency Credit Memo Automatic, 대리점 크레딧 메모 자동 |
| **SSAC** | Self-Service Automatic Check-in, 셀프서비스 자동 체크인 |
| **TAAD** | Ticket Agent Automatic Document, 에이전트 자동 문서 |
| **BPAS** | Boarding Pass, 탑승권 |
| **CANN** | Cancellation, 취소 |
| **TID** | Transaction Identifier, 트랜잭션 식별자 |

#### Issue Indicator (발권 유형)

| 코드 | 설명 |
|------|------|
| **F** | First Issue, 최초 발행 |
| **R** | Reissue, 재발행 |
| **I** | First Issue of IT (Inclusive Tour), IT 최초 발행 |
| **Y** | Reissue of IT, IT 재발행 |
| **B** | First Issue of BT (Bulk Ticket), BT 최초 발행 |
| **W** | Reissue of BT, BT 재발행 |
| **OB** | OB Fee Calculation, OB 수수료 계산 |

### bestEffort (Status Details)

**Best Effort 처리 상태**를 전달하는 세그먼트. 이 세그먼트가 미지정되면 Best Effort가 불가능한 것으로 간주한다.

| 필드 | 형식 | 설명 |
|------|------|------|
| `indicator` | an..3 | Best Effort 지시자 |
| `action` | an..3 | 액션 요청 코드 (KK=확인, UU=거부) |

### reservationControlInformation

**예약 통제 정보**를 전달하는 구조.

| 필드 | 형식 | 설명 |
|------|------|------|
| `reservation.companyId` | an..35 | 항공사 코드 |
| `reservation.controlNumber` | an..20 | PNR Record Locator |
| `reservation.controlType` | an1 | 예약 통제 유형 |
| `reservation.date` | an..35 | 날짜 |

### fopGroup

**FOP 그룹** -- 최대 127개의 서로 다른 FOP를 포함할 수 있는 핵심 구조.

```
fopGroup (M, 최대 127개)
+-- fopReference: FP/SFP 타투 번호
+-- passengerAssociation: 승객 연결 (최대 99)
+-- pnrElementAssociation: PNR 요소 연결 (최대 99)
+-- pricingTicketingDetails: 가격/발권 상세
+-- feeTypeInfo: OB Fee 유형
+-- feeDetailsInfoGroup: OB Fee 상세 (최대 99)
+-- fpProcessingOptions: FP 레벨 옵션
+-- mopDescription: 결제 수단 상세 (최대 99)
|   +-- fopSequenceNumber: FOP 시퀀스 번호
|   +-- fopMasterElementReference: 마스터 FOP 참조
|   +-- stakeholderPayerReference: 결제자 참조
|   +-- mopDetails: 결제 수단 일반 정보
|   |   +-- fopPNRDetails: PNR FOP 코드
|   |   +-- oldFopFreeflow: 이전 FOP 프리플로우
|   +-- pnrSupplementaryData: FOP 스위치/데이터
|   +-- paymentModule: 결제 모듈
```

| 필드 | 구조 | St | Rep | 설명 |
|------|------|----|-----|------|
| `fopReference` | Element management segment | M | 1 | FP/SFP 타투 번호 |
| `passengerAssociation` | Reference information | C | 99 | FOP에 연결된 승객 목록 |
| `pnrElementAssociation` | Reference information | C | 99 | FOP의 PNR 요소 링크 (MCO, Segment 등) |
| `pricingTicketingDetails` | Pricing/ticketing details | C | 1 | 가격/발권 날짜 오버라이드 |
| `feeTypeInfo` | Selection details | C | 1 | OB Fee 유형 (EX=전체 OB Fee 면제) |
| `mopDescription` | Group | C | 99 | 결제 수단(Mean of Payment) 정보 그룹 |

### fopSequenceNumber (Sequence Details)

FOP 라인 내 **결제 수단의 시퀀스 번호**를 전달하는 구조.

| 필드 | 형식 | 설명 |
|------|------|------|
| `number` | an..10 | MOP 시퀀스 번호. FOP 라인에 1개만 있으면 1로 설정. 최대 3개의 신규 MOP와 1개의 구 MOP 가능 |
| `identificationCode` | an..17 | 다른 시퀀스의 하위 요소인 경우 설정 (SUB) |

### feeTypeInfo / feeProcessingInfo (OB Fee 관리)

**OB Fee(Optionally Billable Fee)** 관리를 위한 구조.

| 필드 | 형식 | 설명 |
|------|------|------|
| `feeTypeInfo.option` | an..3 | OB Fee 유형 코드 |
| `feeTypeInfo.optionInformation` | an..35 | OB Fee 옵션 (EX=전체 면제) |
| `feeProcessingInfo.option` | an..3 | OB Fee 하위 유형 코드 |
| `feeProcessingInfo.optionInformation` | an..35 | FEX=하위 유형 제외, FIN=하위 유형 포함 |

---

## 3. 결제 수단 유형 (FOP Types)

### FOP 유형 코드

PNR에 등록할 수 있는 **결제 수단의 유형**. `fopPNRDetails.fopCode`에 설정된다.

| 코드 | 설명 | 비고 |
|------|------|------|
| **CC** | Credit Card, 신용카드 | 가장 일반적인 결제 수단, CCVI/CCCA/CCAX 등으로 표기 |
| **CA** | Cash, 현금 | 현금 결제 |
| **CH** | Cheque, 수표 | 수표 결제 |
| **SWI** | Swipe Card, 스와이프 카드 | 카드 리더기 사용 |
| **WA** | Web Account, 웹 계정 | PayPal 등 온라인 결제 서비스 |
| **WB** | Web Bank / Fund Transfer, 웹 뱅킹 | 온라인 은행 이체 (iDEAL 등) |
| **WW** | Web, 웹 결제 | 기타 웹 기반 결제 |

### fopPNRDetails (Ticketing Form of Payment)

PNR 내 **FOP 코드 및 관련 정보**를 전달하는 핵심 구조.

| 필드 | 형식 | 설명 |
|------|------|------|
| `fopCode` | an..20 | FOP를 식별하는 포맷 키 (CCVI, CA 등) |
| `fopMapTable` | an..20 | FOP 검증에 사용되는 FOP Map Table 이름 |
| `fopBillingCode` | an..10 | 빌링 코드 (CASH CA / Credit CC). MS 리포팅용 |
| `fopStatus` | an..3 | FOP 신규/구 여부 |
| `fopEdiCode` | an..10 | EDIFACT 코드. 구조화된 FOP 추가 시 유형 식별 |
| `fopReportingCode` | an..10 | 리포팅 코드 (@FPXXxx 값) |
| `fopPrintedCode` | an..20 | FOP 인쇄 코드 (@PR 값) |
| `fopElecTicketingCode` | an..10 | 전자 발권 코드. 항공사 전송 분류 및 발권 허용 여부 결정 |

### oldFopFreeflow (Free Text Information)

**이전(Old) FOP 프리플로우 텍스트**. 기존 FOP를 프리플로우 텍스트로 전달한다. 여러 개의 Old FOP도 하나의 프리플로우 텍스트로 간주된다. 예: `FP O/CA+CCVI+/CH CA and CCVI`

| 필드 | 형식 | 설명 |
|------|------|------|
| `freeText` | an..199 | Old FOP 프리플로우 텍스트 |
| `textSubjectQualifier` | an..3 | ZZZ (상호 합의 값) |
| `source` | an..3 | M (Manual) |
| `encoding` | an..3 | ZZZ (상호 합의 인코딩) |

---

## 4. 신용카드 데이터 (Credit Card Data)

### CREDITCARDDATAGROUPTYPE

신용카드 결제 시 전달되는 **카드 데이터 그룹**의 전체 구조.

```
CREDITCARDDATAGROUPTYPE
+-- creditCardDetails: 신용카드 상세 (M)
|   +-- vendorCode: 카드사 코드 (VI, CA, AX)
|   +-- cardNumber: 카드 번호
|   +-- securityId: CVV/CVV2
|   +-- expiryDate: 만료일 (MMYY)
|   +-- ccHolderName: 카드 소유자명
|   +-- tierLevel: 등급 (gold, platinum 등)
+-- fortknoxIds: FortKnox 토큰 ID (C, 최대 2)
+-- cardHolderAddress: 카드 소유자 주소 (C)
+-- virtualCreditCardData: 가상 카드 데이터 (C)
    +-- virtualCreditCardParameters: 가상 카드 파라미터
    +-- validityDate: 가상 카드 유효 기간
```

### creditCardDetails (Credit Card Data)

**신용카드 상세 정보**를 전달하는 구조.

| 필드 | 형식 | 설명 |
|------|------|------|
| `vendorCode` | an2 | 카드사 코드 (VI=Visa, CA=MasterCard, AX=Amex) |
| `vendorCodeSubType` | an..25 | 카드 하위 유형 (Maestro, Solo 등) |
| `cardNumber` | an..19 | 카드 번호 |
| `securityId` | an..4 | CVV/CVV2 보안 코드 (카드 뒷면 3~4자리) |
| `expiryDate` | an4 | 만료일 (MMYY 형식) |
| `startDate` | an4 | 카드 발급일 (UK Maestro 카드용) |
| `endDate` | an4 | 카드 종료일 (UK Maestro 카드용, 만료일과 다를 수 있음) |
| `ccHolderName` | an..99 | 카드 소유자 이름 (카드 표면 인쇄명) |
| `issuingBankName` | an2..3 | 발급 은행 코드 |
| `cardCountryOfIssuance` | an..3 | 카드 발급 국가 |
| `issueNumber` | n..3 | 카드 발급 번호 (Maestro 카드, 분실/재발급 시 증가) |
| `issuingBankLongName` | an..64 | 발급 기관 정식 명칭 |
| `track1` | an..108 | CC Track 1 데이터 (base64 인코딩) |
| `track2` | an..56 | CC Track 2 데이터 (base64 인코딩) |
| `track3` | an..144 | CC Track 3 데이터 (base64 인코딩) |
| `pinCode` | an..100 | PIN 코드 |
| `rawTrackData` | an..400 | 스와이프 카드의 전체 트랙 데이터 |
| `tierLevel` | an..20 | 카드 등급 (gold, platinum 등). 사기 방지/승인에 활용 |

#### Vendor Code (카드사 코드)

| 코드 | 카드사 |
|------|--------|
| **VI** | Visa |
| **CA** | MasterCard |
| **AX** | American Express |
| **DC** | Diners Club |
| **DS** | Discover |
| **JC** | JCB |
| **TP** | UATP (Universal Air Travel Plan) |

### cardHolderAddress (Address)

**카드 소유자 주소** 정보. AVS(Address Verification Service) 검증에 사용된다.

| 필드 | 형식 | 설명 |
|------|------|------|
| `format` | an..3 | 주소 형식 코드 (5=비구조화) |
| `line1` ~ `line6` | an..70 | 주소 텍스트 (각 줄) |
| `city` | an..35 | 도시명 |
| `zipCode` | an..17 | 우편번호 |
| `countryCode` | an..3 | 국가 코드 (ISO 3166) |

---

## 5. FortKnox 토큰화

### fortknoxIds (Reference Information)

Amadeus **FortKnox 데이터베이스**에 저장된 CVV와 신용카드 번호의 토큰 ID를 전달하는 구조. 카드 원본 데이터 대신 토큰을 사용하여 PCI DSS 보안 요구사항을 충족한다.

```
FortKnox 토큰화 흐름:

[신용카드 원본 데이터]
  카드 번호: 4111-1111-1111-1111
  CVV: 123
       |
       v
[FortKnox 토큰화 엔진]
  +-- 카드 번호 -> 토큰 ID (NOX)
  +-- CVV -> 토큰 ID (CVV)
       |
       v
[PNR에 저장되는 데이터]
  fortknoxIds[0]: type=NOX, value=FK_TOKEN_001  (카드 번호 토큰)
  fortknoxIds[1]: type=CVV, value=FK_TOKEN_002  (CVV 토큰)
  * 원본 카드 번호/CVV는 저장되지 않음
```

| 필드 | 형식 | 설명 |
|------|------|------|
| `referenceDetails.type` | an..10 | 토큰 유형 (NOX=카드 번호, CVV=보안 코드) |
| `referenceDetails.value` | an..60 | FortKnox 토큰 ID 값 |

#### FortKnox 토큰 유형

| 유형 | 설명 |
|------|------|
| **NOX** | 신용카드 번호 FortKnox 토큰 ID |
| **CVV** | CVV/CVV2 보안 코드 FortKnox 토큰 ID |

---

## 6. CVV 처리 (보안 코드 검증)

### securityId

신용카드의 **CVV/CVV2 보안 코드**. 카드 뒷면(또는 Amex의 경우 앞면)에 인쇄된 3~4자리 숫자. `creditCardDetails.securityId` 필드(an..4)에 설정된다.

### CVV 검증 결과 코드

카드사/발급사에서 반환하는 **CVV 검증 결과**. `transactionStatus` 그룹 내에서 CVV 관련 상태로 전달된다.

| 코드 | 설명 |
|------|------|
| **A** | CVV Security ID Approved, 승인됨 |
| **I** | CVV Cardholder Stated Security ID is Illegible, 판독 불가 |
| **M** | CVV Cardholder Stated Security ID is Not on the Card (Missing), 카드에 없음 |
| **N** | CVV Security ID Not Processed, 처리되지 않음 |
| **S** | CVV Security ID Should Be on the Card but Merchant Indicates It Is Not, 카드에 있어야 하나 가맹점이 없다고 표시 |
| **U** | CVV Issuer Not Certified, or User Unregistered, 발급사 미인증/미등록 |
| **X** | CVV Security ID Rejected, 거부됨 |

### transactionStatus

신용카드 결제에 관련된 **다양한 하위 상태**를 전달하는 구조 (최대 7개 반복).

| 코드 | 설명 |
|------|------|
| **CVV** | CVV Return Code, CVV 검증 결과 |
| **AVS** | Address Verification Return Code, 주소 검증 결과 |
| **AUT** | Authorization Return Code, 승인 결과 |
| **ATN** | Authentication Return Code, 인증 결과 |
| **PNR** | PNR Update Return Code, PNR 업데이트 결과 |
| **SET** | Settlement Return Code, 정산 결과 |

---

## 7. 3DS 인증 (3-D Secure)

### THREEDOMAINSECUREGROUPTYPE

**3-D Secure(3DS) 인증** 데이터를 전달하는 그룹 구조. Visa의 Verified by Visa, MasterCard의 SecureCode 등 카드사 3DS 프로토콜을 지원한다.

```
3DS 인증 흐름:

[1] 고객 -> 가맹점: 결제 요청
[2] 가맹점 -> Amadeus: FOP_CreateFormOfPayment (3DS 데이터 포함)
       |
       v
[3] Amadeus -> ACS (Access Control Server)
    +-- acsURL: 카드사 인증 서버 URL (최대 2048자)
    +-- authenticationData: 인증 데이터
    +-- tdsBlobData: 암호화된 3DS 데이터
       |
       v
[4] ACS -> 고객: 비밀번호 입력 요청 (리다이렉트)
[5] 고객 -> ACS: 비밀번호 입력
[6] ACS -> Amadeus: 인증 결과
       |
       v
[7] Reply: Payment Status = W (Web Redirection) 또는 V (Validated)
```

| 필드 | 설명 |
|------|------|
| `authenticationData` | 3DS 인증 데이터 |
| `acsURL` | Access Control Server URL (최대 2048자). 카드 소유자 인증 페이지 |
| `tdsBlobData` | 3DS 암호화 데이터 블롭 |

### tdsInformation

`CREDITCARDSTATUSGROUPTYPE` 내에서 **3DS 관련 데이터**를 전달하는 그룹.

### schemeAuthenticationData

**카드 스킴별 인증 데이터**를 전달하는 그룹. IATA 및 ARC의 RET/SPRF 리포팅에 필요한 데이터를 포함한다.

| 필드 | 설명 |
|------|------|
| `schemeCompany` | 카드 스킴 회사 (VIS, MAS, AMX, DIN, DIS) |
| `schemeDataElement` | 결제 승인 시 카드 스킴에서 생성된 데이터 요소 (최대 99개) |

#### Card Scheme Company (카드 스킴 회사)

| 코드 | 카드사 |
|------|--------|
| **VIS** | VISA |
| **MAS** | MasterCard |
| **AMX** | American Express |
| **DIN** | Diners Club |
| **DIS** | Discover |

---

## 8. 결제 모듈 (Payment Module)

### paymentModule

결제 트랜잭션과 관련된 **모든 데이터**를 포함하는 최상위 결제 구조.

```
paymentModule
+-- PAYMENTDATAGROUPTYPE
|   +-- merchantInformation: 가맹점 정보 (항공사, 보험사 등)
|   +-- monetaryInformation: 금액 정보 (최대 999)
|   +-- currenciesRatesGroup: 환율 정보 (최대 9)
|   +-- paymentId: Payment Record ID (최대 99)
|   +-- extendedPaymentInfo: 할부 결제 정보
|   +-- transactionDateTime: 트랜잭션 일시
|   +-- expirationPeriod: 결제 유효 기간 (초 단위)
|   +-- distributionChannelInformation: 유통 채널 정보
|   +-- purchaseDescription: 구매 설명
|   +-- association: Pricing Context 연결 (최대 99)
|   +-- fraudScreeningData: 사기 방지 데이터
|   +-- paymentDataMap: 결제 부가 정보 (최대 99)
+-- PAYMENTSTATUSGROUPTYPE
|   +-- paymentStatusInformation: 결제 상태
|   +-- paymentStatusHistory: 상태 이력 (최대 9)
|   +-- paymentStatusError: 결제 오류
|   +-- fraudScreeningResult: 사기 스크리닝 결과
+-- PNRSUPPLEMENTARYDATATYPE
    +-- dataAndSwitchMap: FOP 스위치 및 데이터 맵
```

### merchantInformation (Company Information)

**가맹점(Merchant) 정보**. 결제를 요청하는 주체(항공사, 보험사 등)를 식별한다.

### monetaryInformation (Monetary Information)

결제 관련 **금액 정보**. 최대 999개 반복.

| 필드 | 설명 |
|------|------|
| `amount` | 결제 금액 |
| `currency` | 통화 코드 (ISO 4217) |

### distributionChannelInformation

**유통 채널 및 트랜잭션 컨텍스트** 정보.

| 필드 | 형식 | 설명 |
|------|------|------|
| `distributionChannelField` | an..3 | 유통 채널 코드 |
| `subGroup` | an..3 | 하위 그룹 |
| `accessType` | an..3 | 접근 유형 |
| `originatorType` | a1 | 발신자 유형 |
| `transactionCondition` | an..25 | 트랜잭션 조건 요약 |
| `identityVerifiedBy` | an..25 | 본인 확인 수행 주체 |
| `remoteCommerceIndicators` | an..25 | 원격 상거래 지표 |
| `authorCharacteristicInd` | a1 | 승인 특성 지표 (Y=참여, R=반복, I=상향, P=우대) |
| `authorLifecycleLimit` | n..15 | 승인 수명 제한 (일 단위) |

#### Distribution Channel (유통 채널 코드)

| 코드 | 설명 |
|------|------|
| **05** | API |
| **0** | MOTO (Mail Order / Telephone Order) |
| **1** | e-Commerce (Internet) |

---

## 9. 사기 방지 스크리닝 (Fraud Screening)

### FRAUDSCREENINGGROUPTYPE

**사기 방지(Fraud Screening)** 데이터를 전달하는 그룹 구조. 온라인 결제 시 위험 평가를 위한 다양한 정보를 포함한다.

```
fraudScreeningData
+-- pointOfService: 판매 시점 정보
|   +-- saleIndicator: 판매 유형 (S=스와이프, I=인터넷, A=콜센터)
|   +-- cardPresence: 카드 제시 여부 (CP/CNP)
|   +-- cardCapture: 카드 캡처 가능 여부
+-- posOperations: POS 운영 정보
|   +-- connectivityCapacity: 연결 용량 (Able/Unable/Unknown)
+-- paymentTerminal: 결제 단말기 정보
|   +-- terminalID: ATID 번호 (an8)
+-- ipAdress: 고객 IP 주소
+-- merchantURL: 가맹점 URL
+-- payerPhoneOrEmail: 결제자 전화번호/이메일
+-- browserInformation: 브라우저 정보
+-- shopperSession: 쇼퍼 세션 정보
|   +-- securityCode: Device Finger Print (기기 지문)
```

#### Sale Indicator (판매 유형)

| 코드 | 설명 |
|------|------|
| **S** | Swipe, 스와이프 (카드 리더기) |
| **I** | Internet/Online, 인터넷/온라인 |
| **A** | Call Center/IVR, 콜센터/자동응답 |
| **P** | Kiosk, 키오스크 |
| **E** | Offline Travel Agency, 오프라인 여행사 |
| **T** | Telephone, 전화 |

#### Card Presence (카드 제시 여부)

| 코드 | 설명 |
|------|------|
| **CP** | Card Present, 카드 제시 (대면 거래) |
| **CNP** | Card Not Present, 카드 미제시 (비대면 거래) |
| **NoContact** | Contactless, 비접촉 |

### fraudScreeningResult (Measurements)

**사기 스크리닝 결과** 정보.

| 필드 | 형식 | 설명 |
|------|------|------|
| `measurementQualifier` | an..3 | 측정 유형 (FRA=사기 방지) |
| `significance` | an..3 | 스크리닝 결과 |
| `unit` | an..3 | 단위 (P=포인트) |
| `value` | n..18 | 리스크 관리 점수 (예: 300 Points) |
| `surfaceLayerIndicator` | an..3 | PSP/은행의 스크리닝 처리 결과 |

#### Fraud Screening 결과 코드

| 코드 | 설명 |
|------|------|
| **OK** | Fraud Screening Result Approved, 승인 |
| **KO** | Fraud Screening Result Declined, 거부 |
| **WRN** | Fraud Screening Result Warning, 경고 |

---

## 10. 가상 신용카드 (Virtual Credit Card)

### virtualCreditCardParameters (Virtual Card Parameters)

**가상 신용카드 생성** 파라미터를 전달하는 구조.

| 필드 | 형식 | 설명 |
|------|------|------|
| `vendorCode` | an2 | 카드사 코드 (VI, CA, AX). CA 입력 시 MasterCard 가상 카드 생성 |
| `maximumAuthorizations` | n..4 | 가상 카드에 허용되는 최대 승인 횟수 |
| `currency` | an..3 | 통화 제한 (최대 5개, ISO 4217) |

### virtualCreditCardStatusGroup

응답에서 반환되는 **가상 신용카드 상태** 그룹.

| 필드 | 설명 |
|------|------|
| `virtualCreditCardParameters` | 가상 카드 파라미터 |
| `virtualCreditCardData` | 가상 카드 번호, 만료일, CVV, 카드사, 소유자명 |
| `fortknoxIds` | FortKnox 토큰 ID (카드 번호 + CVV) |
| `vCCAssociatedAdress` | 가상 카드의 AVS 검증용 주소 |

---

## 11. Reply 주요 구조 (FOP 등록 결과)

### fopDescription

응답에서 반환되는 **FOP 데이터** 구조 (최대 127개).

| 필드 | 구조 | St | Rep | 설명 |
|------|------|----|-----|------|
| `fopReference` | Element management segment | M | 1 | FP/SFP 타투 번호 |
| `passengerAssociation` | Reference information | C | 99 | FOP에 연결된 승객 |
| `pnrElementAssociation` | Reference information | C | 99 | PNR 요소 링크 (MCO, Segment 등) |
| `additionalMonetaryData` | Coded attribute | C | 1 | 추가 금액 데이터 |
| `freeFlowFop` | Free text information | C | 1 | PNR에 표시되는 FOP 전체 텍스트 |
| `fpElementError` | Group | C | 1 | FOP 읽기 중 발생한 오류 |
| `mopDescription` | Group | C | 99 | 결제 수단 상세 정보 |
| `mopElementError` | Group | C | 1 | MOP 읽기 중 발생한 오류 |
| `paymentModule` | Group | C | 1 | 결제 트랜잭션 전체 데이터 |

### transmissionError

**전송 중 발생한 오류** 정보 그룹.

### paymentStatusInformation (Response Analysis Details)

**결제 상태**를 전달하는 구조.

| 필드 | 형식 | 설명 |
|------|------|------|
| `responseType` | a1 | 결제 상태 유형 코드 |
| `statusCode` | a..6 | 처리 상태 (OK 또는 NOK) |

#### Payment Status responseType (결제 상태 유형)

| 코드 | 설명 |
|------|------|
| **C** | Payment Created, 결제 생성됨 |
| **G** | Payment Got/Captured, 결제 캡처됨 |
| **D** | Payment Deleted, 결제 삭제됨 |
| **U** | Payment Updated, 결제 업데이트됨 |
| **V** | Payment Validated, 결제 검증됨 |
| **R** | Payment Refund, 결제 환불 |
| **S** | Payment Reversed, 결제 취소(반전) |
| **W** | Payment with Web Redirection, 웹 리다이렉트 결제 |

---

## 12. 승인 및 검증 (Authorization)

### CREDITCARDSTATUSGROUPTYPE

신용카드 **승인 상태 그룹**. ISO8583 표준에 따른 승인 데이터를 전달한다.

| 필드 | 구조 | 설명 |
|------|------|------|
| `authorisationSupplementaryData` | Specific visa link credit card information | ISO8583 표준 링크 데이터 |
| `approvalDetails` | Generic authorisation result | 승인 코드/소스 |
| `localDateTime` | Structured date time information | 일시 (GMT, UTC, Local). 최대 3개 |
| `authorisationInformation` | Transaction information for ticketing | 승인 메시지 유형 (bulk, superbulk 등), STAN 번호 |
| `browserData` | Group | 고객 브라우저 정보 |
| `tdsInformation` | Group | 3DS 관련 데이터 |
| `cardSupplementaryData` | Attribute | 신용카드 추가 데이터 전송 (최대 99) |
| `transactionStatus` | Group | CVV, AVS, AUT, ATN 등 하위 상태 (최대 7) |
| `schemeAuthenticationData` | Group | 카드 스킴 인증 데이터 |

### approvalDetails (Generic Authorisation Result)

**승인 결과** 정보.

| 필드 | 형식 | 설명 |
|------|------|------|
| `approvalCode` | an..20 | 결제 승인 코드 값 |
| `sourceOfApproval` | an..3 | 승인 출처 (A=자동, M=수동 입력) |

### authorisationSupplementaryData (VISA Link)

**ISO8583 표준** 기반의 카드 승인 보충 데이터.

| 필드 | 형식 | 설명 |
|------|------|------|
| `retrievalReferenceNumber` | an..12 | Retrieval Reference Number (Field 37). 카드 소유자 트랜잭션 추적용 |
| `authorCharacteristicIndicator` | a1 | 승인 특성 지표 (Field 62.1). A/C/E/F/K/M/S/U/V/W/R/I/P/N/T |
| `authorResponseCode` | an2 | 승인 응답 코드 (Field 39) |
| `cardLevelResult` | an2 | Card Level Result (Field 62.23). 제품 식별 |
| `terminalType` | an1 | POS 추가 정보 (Field 60.1). CAT 또는 UAT |
| `transacIdentifier` | an..15 | 트랜잭션 식별자 (Field 62.2). Visa 고유 ID |
| `validationCode` | an..4 | 검증 코드 (Field 62.3) |
| `banknetRefNumber` | an6..9 | Banknet 참조 번호 (Field 62.17, Position 8-13) |
| `banknetDate` | an4 | Banknet 날짜 (Field 62.17, mmdd 형식) |

---

## 13. 비동기 결제 (Asynchronous Payment)

### ASYNCHPAYMENTSTATUSGROUPTYPE

**비동기 결제 상태** 그룹. 은행/PSP에서 비동기적으로 처리되는 결제의 상태를 전달한다.

| 필드 | 설명 |
|------|------|
| `approvalReferenceNumber` | 비동기 결제 승인 참조 번호 |
| `asyncPaymentUrl` | 은행/PSP 제공 비동기 결제 URL |

### ASYNCHPAYMENTGROUPTYPE

**비동기 결제** 그룹.

| 필드 | 설명 |
|------|------|
| `asunchronousPaymentDetails` | 계좌 번호 및 만료일 저장 |

---

## 14. AMOP (Alternative Method of Payment)

### AMOP 구조

**대안 결제 수단(Alternative Method of Payment)** 처리를 위한 구조. PayPal, iDEAL 등 전통적 카드 결제가 아닌 방식을 지원한다.

```
paymentModule
+-- amopDetailedData: AMOP 상세 데이터
|   +-- stepDefinition: 생성(BUILD) 또는 위임(DELEG) 단계
|   +-- messageVersion: 메시지 버전
|   +-- paymentDataMap: 결제 정보 (최대 99)
|   +-- groupAmopProcess: AMOP 처리 데이터
|   +-- groupAmopParameters: AMOP 파라미터
|   +-- amopGroupUrl: AMOP URL (최대 2)
|   +-- amopPayload: AMOP 페이로드 (최대 99)
+-- groupAmopContext: AMOP 컨텍스트
|   +-- clientTokenId: 클라이언트 토큰 ID
|   +-- amopContextData: 컨텍스트 데이터 (최대 99)
|   +-- communication: 통신 정보 (최대 99)
+-- transactionResult: 트랜잭션 결과
+-- errorGroup: 오류 그룹 (최대 9)
```

### stepDefinition

AMOP 처리의 **단계 정의**.

| 코드 | 설명 |
|------|------|
| **BUILD** | Creation Step, 생성 단계 |
| **DELEG** | Delegation Step, 위임 단계 |
| **FECBR** | Frontend Callback Response, 프런트엔드 콜백 응답 |
| **WEBRD** | Web Redirection Data, 웹 리다이렉트 데이터 |
| **PROCC** | AMOP Process Details, AMOP 처리 상세 |
| **PARAM** | AMOP Parameters, AMOP 파라미터 |

---

## 15. 주요 코드셋 (Codesets)

### Application Error 코드

| 코드 | 설명 |
|------|------|
| **04691** | NO TATTOO OR LINE MATCH FOUND, 타투/라인 매치 없음 |
| **08000** | INVALID - DUPLICATE INPUTS NOT ALLOWED, 중복 입력 불가 |
| **02213** | INVALID FORM OF PAYMENT, 유효하지 않은 결제 수단 |
| **02312** | INVALID SEQUENCE NUMBER, 유효하지 않은 시퀀스 번호 |
| **294** | Invalid Format, 유효하지 않은 형식 |

### Application Error 상태 코드 (계정)

| 코드 | 설명 |
|------|------|
| **C** | Closed, 폐쇄 |
| **E** | Expired, 만료 |
| **F** | Frozen, 동결 |
| **O** | Open for Use, 사용 가능 |
| **R** | Refunded, 환불됨 |
| **TX** | Transaction In Progress, 거래 진행 중 |

### Business Function 코드

| 코드 | 설명 |
|------|------|
| **1** | Air Provider, 항공 서비스 |
| **2** | Car Provider (CCR), 렌터카 |
| **3** | Hotel Provider (HHL), 호텔 |
| **4** | Ferry, 페리 |
| **5** | Cruise, 크루즈 |
| **6** | Rail, 철도 |
| **7** | Tour, 투어 |
| **10** | Air Taxi (ATX), 에어 택시 |
| **17** | Charter, 전세기 |
| **29** | Insurance, 보험 |

### Attribute Type 코드 (FOP 요소 유형)

| 코드 | 설명 |
|------|------|
| **AO** | Authorisation Only, 승인만 |
| **BUILD** | Creation Step, 생성 단계 |
| **DELEG** | Delegation Step, 위임 단계 |
| **FC** | FC Element, FC 요소 |
| **FP** | FP Element, FP 요소 |
| **PAY** | PAY Element, PAY 요소 |

### Code List Qualifier (Return Code 유형)

| 코드 | 설명 |
|------|------|
| **ATN** | Authentication Return Code, 인증 반환 코드 |
| **AUT** | Authorization Return Code, 승인 반환 코드 |
| **AVS** | Address Verification Return Code, 주소 검증 반환 코드 |
| **CVV** | CVV Return Code, CVV 반환 코드 |
| **PNR** | PNR Update Return Code, PNR 업데이트 반환 코드 |
| **SET** | Settlement Return Code, 정산 반환 코드 |
| **EC** | Error Codes, 오류 코드 |
| **WEC** | Warning Code, 경고 코드 |
| **ACC** | Account Return Code, 계정 반환 코드 |

### Communication Address Code Qualifier (URL 유형)

| 코드 | 설명 |
|------|------|
| **DEF** | Default URL, 기본 URL |
| **ERC** | Error Callback URL, 오류 콜백 URL |
| **FCB** | Fail Callback URL, 실패 콜백 URL |
| **KAC** | Keepalive Callback URL, 킵얼라이브 콜백 URL |
| **PCB** | Pending Callback URL, 보류 콜백 URL |
| **R** | Return URL, 반환 URL |
| **RED** | Redirection URL, 리다이렉트 URL |
| **RES** | Resource URL, 리소스 URL |
| **RSP** | Response URL, 응답 URL |
| **SCB** | Success Callback URL, 성공 콜백 URL |
| **C** | Cancel URL, 취소 URL |
| **AH** | World Wide Web, 월드 와이드 웹 |
| **BO** | Boleto FOP URL, 볼레토 FOP URL |

---

## 16. 메시지 구조 용어

FOP_CreateFormOfPayment 기술 문서에서 사용되는 **메시지 구조 정의 용어**.

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
| `a1` | 고정 1자리 문자 | `F` (Issue Indicator) |
| `an2` | 고정 2자리 영숫자 | `VI` (Vendor Code) |
| `an..4` | 가변 영숫자 최대 4자리 | `123` (CVV) |
| `an..19` | 가변 영숫자 최대 19자리 | `4111111111111111` (카드 번호) |
| `an..99` | 가변 영숫자 최대 99자리 | `HONG/GILDONG` (카드 소유자명) |
| `n..15` | 가변 숫자 최대 15자리 | `528400` (금액) |
| `an4` | 고정 4자리 영숫자 | `1225` (만료일 MMYY) |

---

## 약어 모음

| 약어 | 정식 명칭 | 설명 |
|------|----------|------|
| **FOP** | Form of Payment | 결제 수단 |
| **MOP** | Mean of Payment | 결제 방법 (FOP 라인 내 개별 결제 수단) |
| **CC** | Credit Card | 신용카드 |
| **CA** | Cash | 현금 |
| **CH** | Cheque | 수표 |
| **CVV** | Card Verification Value | 카드 보안 코드 (3~4자리) |
| **3DS** | 3-D Secure | 3도메인 보안 인증 프로토콜 |
| **ACS** | Access Control Server | 3DS 카드 소유자 인증 서버 |
| **AVS** | Address Verification Service | 주소 검증 서비스 |
| **PCI DSS** | Payment Card Industry Data Security Standard | 결제 카드 산업 데이터 보안 표준 |
| **AMOP** | Alternative Method of Payment | 대안 결제 수단 (PayPal, iDEAL 등) |
| **PSP** | Payment Service Provider | 결제 서비스 제공자 |
| **OB Fee** | Optionally Billable Fee | 선택적 청구 수수료 |
| **MCO** | Miscellaneous Charge Order | 기타 요금 지시서 |
| **SFP** | Structured Form of Payment | 구조화된 결제 수단 |
| **FP** | Form of Payment (PNR Element) | PNR 내 결제 수단 요소 |
| **PAY** | Payment Element | 결제 요소 |
| **STAN** | System Trace Audit Number | 시스템 추적 감사 번호 (CC 승인 쌍 식별) |
| **ATID** | Amadeus Terminal ID | Amadeus 터미널 ID |
| **ISO8583** | ISO 8583 | 금융 거래 카드 메시지 표준 |
| **MOTO** | Mail Order / Telephone Order | 우편/전화 주문 |
| **CNP** | Card Not Present | 카드 미제시 (비대면 거래) |
| **CP** | Card Present | 카드 제시 (대면 거래) |
| **RET** | Reporting of Electronic Ticketing | 전자 발권 리포팅 |
| **SPRF** | Sales/Payment Report File | 판매/결제 리포트 파일 |
| **PNR** | Passenger Name Record | 승객 예약 기록 |
| **ETS** | Electronic Ticketing Server | 전자 발권 서버 |
| **VCC** | Virtual Credit Card | 가상 신용카드 |
| **NOX** | Non-sensitive Encrypted ID | FortKnox 카드 번호 토큰 ID |
| **EDIFACT** | Electronic Data Interchange For Administration, Commerce and Transport | 전자 데이터 교환 표준 |

---

## 참고

- [WBS Integration Flow - Step 9](amadeus-wbs-integration-flow.md)
- [PNR_AddMultiElements 용어집](pnr-add-multi-elements.md)
- [Fare_PricePNRWithBookingClass 용어집](fare-price-pnr-with-booking-class.md)
- [BSP 정산 가이드](bsp-settlement-guide.md)
