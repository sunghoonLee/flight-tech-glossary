# Air FlightInfo (항공편 정보 조회)

Amadeus `Air_FlightInfo` API(버전 07.1)의 기술 용어를 정리한 문서입니다. 특정 항공편의 스케줄, 운항 정보, 좌석 가용성, 기내 서비스 등 상세 정보를 조회하는 API입니다.

---

## 1. 개요

**Air_FlightInfo**는 항공편 번호와 날짜를 기반으로 해당 항공편의 상세 운항 정보를 조회하는 Amadeus API입니다.

- **Request**: 항공편 번호, 날짜, 출발/도착 공항 등을 지정하여 조회
- **Reply**: 스케줄, 경유지, 기재 유형, 좌석 현황, 기내 서비스 등 상세 정보 반환

!!! info "용도"
    PNR 생성 전 항공편 상세 확인, 스케줄 변경 감지, 좌석 가용성 확인, 기재 변경 확인 등에 활용됩니다.

---

## 2. Query 구조 (요청)

### generalFlightInfo

항공편 정보 조회의 핵심 요청 구조입니다.

| 필드 | 설명 |
|------|------|
| `boardPointDetails` | 출발 공항 코드 (IATA 3자리). 예: ICN, NRT |
| `offPointDetails` | 도착 공항 코드 (IATA 3자리). 예: LAX, CDG |
| `companyDetails.marketingCompany` | 마케팅 항공사 코드 (IATA 2자리). 예: KE, OZ |
| `companyDetails.operatingCompany` | 운항 항공사 코드. 코드셰어 시 마케팅 항공사와 다를 수 있음 |
| `flightIdentification.flightNumber` | 편명 번호 (최대 4자리). 예: 0001, 5023 |
| `flightIdentification.operationalSuffix` | 편명 접미사. 동일 편명의 변형 구분용 |
| `flightTypeDetails.flightIndicator` | 항공편 유형 지시자. → [Flight Indicator 코드](#flight-indicator-코드) 참조 |

---

## 3. Reply 구조 (응답)

### flightScheduleDetails

항공편 스케줄 상세 정보를 담는 최상위 응답 구조입니다.

### boardPointAndOffPointDetails

출발/도착 공항별 상세 정보 그룹입니다. 경유편의 경우 구간(Leg)별로 반복됩니다.

---

## 4. 주요 데이터 구조

### 4-1. Travel Product Information (여행 상품 정보)

항공편의 기본 식별 정보입니다.

**flightDate** (운항 일자)

| 필드 | 설명 |
|------|------|
| `departureDate` | 출발일 (ddMMyy 형식) |
| `arrivalDate` | 도착일. 야간 비행·시차로 출발일과 다를 수 있음 |

**boardPointDetails / offPointDetails** (출발·도착지)

| 필드 | 설명 |
|------|------|
| `trueLocationId` | IATA 공항 코드 (3~5자리) |

**companyDetails** (항공사 정보)

| 필드 | 설명 |
|------|------|
| `marketingCompany` | 마케팅 항공사 코드 (최대 3자리) |
| `operatingCompany` | 실제 운항 항공사 코드. 코드셰어 구간에서 마케팅 항공사와 다름 |

**flightIdentification** (편명 식별)

| 필드 | 설명 |
|------|------|
| `flightNumber` | 항공편 번호 (최대 4자리) |
| `operationalSuffix` | 편명 접미사 (1자리). 스케줄 변경·특별 운항 구분 |

---

### 4-2. Additional Product Details (부가 상품 정보)

항공편의 운항 세부 사항입니다.

| 필드 | 설명 |
|------|------|
| `equipmentType` | IATA 기재 코드 (3자리). 예: 388(A380), 789(B787-9), 77W(B777-300ER) |
| `numberOfStops` | 경유 횟수. 0=직항, 1=1회 경유 |
| `duration` | 비행 시간 (분 단위). 예: 660 = 11시간 |
| `daysOfOperation` | 운항 요일. 7자리 문자열, 각 자리가 월~일 (예: `1234567` = 매일, `12345..` = 평일만) |
| `complexingFlightIndicator` | 복합 항공편 표시자. 한 편명이 여러 구간으로 나뉠 때 사용 |

**departureStationInfo / arrivalStationInfo** (공항 시설 정보)

| 필드 | 설명 |
|------|------|
| `gate` | 게이트 번호 |
| `terminal` | 터미널 번호. 예: 1, 2, INT |
| `concourse` | 탑승동(Concourse) 식별자 |

**기타 시간·거리 정보**

| 필드 | 설명 |
|------|------|
| `flightLegMileage` | 구간 거리 (마일 단위) |
| `travellerTimeDetails` | 승객 기준 소요 시간 (시차 포함) |
| `checkInDetails` | 체크인 관련 정보. 온라인 체크인 가능 여부, 마감 시간 등 |

---

### 4-3. Product Information (좌석·클래스 정보)

각 캐빈/예약 클래스별 좌석 현황입니다.

**bookingClassDetails**

| 필드 | 설명 |
|------|------|
| `designator` | RBD(Reservation Booking Designator). 예: F, J, C, Y, B, H, L 등 |
| `availabilityStatus` | 좌석 가용 상태 코드. → [Availability Status 코드](#availability-status) 참조 |
| `specialService` | 특별 서비스 코드. 해당 클래스에서 제공하는 특수 서비스 |

---

### 4-4. Equipment Information (기재 정보)

항공기 기종·캐빈 배치 정보입니다.

**cabinClassDetails**

| 필드 | 설명 |
|------|------|
| `classDesignator` | 캐빈 등급 코드. F=First, J/C=Business, W=Premium Economy, Y=Economy |
| `numberOfSeats` | 해당 캐빈의 총 좌석 수 |

**iataAircraftTypeCode**
:   IATA 항공기 기종 코드 (3자리). Equipment Type과 동일. 예: 388, 789, 77W, 332

---

### 4-5. Marriage Control (결합 구간)

여러 구간이 하나의 여정으로 묶이는 경우의 제어 정보입니다. GDS에서 구간 분리·취소 시 중요합니다.

| 필드 | 설명 |
|------|------|
| `relation` | 결합 관계 코드. → [Marriage Relation 코드](#marriage-relation-코드) 참조 |
| `marriageIdentifier` | 결합 그룹 식별 번호 (최대 2자리) |
| `lineNumber` | 구간 순서 번호 |
| `otherRelation` | 추가 관계 코드 |
| `carrierCode` | 결합 구간의 항공사 코드 |

---

### 4-6. Facilities Information (기내 서비스 정보)

기내 제공 서비스(식사, 음료 등) 정보입니다.

| 코드 | 서비스 |
|------|--------|
| `B` | Breakfast (아침 식사) |
| `C` | Alcoholic beverages - complimentary (무료 주류) |
| `D` | Dinner (저녁 식사) |
| `F` | Food for purchase (유료 식사) |
| `G` | Food and Beverages for purchase (유료 식음료) |
| `H` | Hot meal (온식) |
| `K` | Continental breakfast (콘티넨탈 조식) |
| `L` | Lunch (점심 식사) |
| `M` | Meal (식사 - 일반) |
| `N` | No meal service (기내식 없음) |
| `O` | Cold meal (냉식) |
| `P` | Alcoholic beverages for purchase (유료 주류) |
| `R` | Refreshments - Complimentary (무료 다과) |
| `S` | Snack or Brunch (스낵/브런치) |
| `V` | Refreshments for purchase (유료 다과) |

---

## 5. 주요 코드셋

### Flight Indicator 코드

항공편의 서비스 유형을 나타내는 코드입니다.

| 코드 | 의미 |
|------|------|
| `A` | Codeshare service (코드셰어 운항) |
| `B` | Bus (버스 연결) |
| `C` | Connection portion of journey (연결 구간) |
| `D` | Direct service (직항 서비스) |
| `E` | End of journey (여정 종료) |
| `ET` | Electronic ticket candidate (전자 발권 가능) |
| `EN` | Not electronic ticket candidate (전자 발권 불가) |
| `F` | Charter Flight (전세 항공편) |
| `I` | Inbound flight (인바운드 항공편) |
| `J` | Stopover permitted (스톱오버 허용) |
| `K` | Stopover not permitted (스톱오버 불가) |
| `M` | Marketing flight grouping indicator (마케팅 편명 그룹 표시) |
| `N` | Non-stop service (무기착 직항) |
| `O` | Operating flight (운항 항공편) |
| `S` | Start of journey (여정 시작) |
| `T` | Transfer (환승) |
| `TR` | Train (철도 연결) |
| `CR` | Car Rental (렌터카 연결) |
| `CS` | Cruise Ship (크루즈 연결) |
| `HT` | Hotel (호텔 연결) |

---

### Marriage Relation 코드

결합 구간의 관계 유형을 나타냅니다.

| 코드 | 의미 |
|------|------|
| `A` | Married (결합됨) |
| `B` | Non-dominant flight (비주도 항공편) |
| `C` | Potential marriage candidate (결합 후보) |
| `F` | First host cascading (첫 번째 호스트 전파) |
| `L` | Last host cascading (마지막 호스트 전파) |
| `M` | Middle host cascading (중간 호스트 전파) |

!!! note "Marriage Segment란?"
    GDS에서 여러 구간이 하나의 여정으로 **결합(Married)**된 상태를 말합니다. 결합된 구간은 개별 취소/변경이 불가하고 전체를 함께 처리해야 합니다. 예: ICN→NRT→LAX가 하나의 결합 여정인 경우, NRT→LAX만 단독 취소할 수 없습니다.

---

### Business Function 코드

서비스 유형을 분류하는 코드입니다.

| 코드 | 의미 |
|------|------|
| `1` | Air Provider (항공 서비스) |
| `2` | Car Provider / Hotel Provider |
| `3` | Hotel Provider |
| `4` | Ferry (페리) |
| `5` | Cruise (크루즈) |
| `6` | Rail (철도) |
| `7` | Tour (투어) |
| `10` | Air taxi (에어택시) |
| `17` | Charter (전세기) |
| `36` | Prepaid ticket (선불 항공권) |
| `37` | Seats (좌석) |
| `39` | Ticket (항공권) |
| `G` | Ground tracking (지상 추적) |

---

### Application Error 코드

| 코드 | 의미 |
|------|------|
| `AUE` | Flight cancelled (항공편 취소) |

---

### Text Subject Qualifier 코드

부가 텍스트 정보의 유형을 구분합니다.

| 코드 | 의미 |
|------|------|
| `CHG` | Change information (변경 정보) |
| `PRD` | Product information (상품 정보) |
| `SAF` | Safety information (안전 정보) |
| `SIM` | IATA SSIM defined information (SSIM 정보) |
| `SPH` | Special handling (특수 처리) |
| `STN` | Statutory notice (법적 고지) |
| `TRA` | Transportation information (교통 정보) |

---

## 6. FLIFO (Flight Information)

**FLIFO**는 항공편의 실시간 운항 정보를 의미합니다. Air_FlightInfo Reply의 Information Type 코드 `1`이 "Flifo exists"로, 해당 항공편에 실시간 운항 정보가 존재함을 나타냅니다.

FLIFO가 제공하는 주요 정보:

- 게이트 변경
- 지연/결항 상태
- 실제 출발/도착 시간
- 터미널 변경

---

## 7. 메시지 기능 코드 (주요)

Air_FlightInfo에서 사용되는 주요 메시지 기능 코드입니다.

| 코드 | 의미 | 설명 |
|------|------|------|
| `44` | Availability request | 가용성 요청 |
| `45` | Availability response | 가용성 응답 |
| `48` | Daily schedule request | 일간 스케줄 요청 |
| `49` | Daily schedule response | 일간 스케줄 응답 |
| `50` | Specific Schedule | 특정 스케줄 |
| `51` | Time table request | 시간표 요청 |
| `52` | Time table response | 시간표 응답 |
| `65` | Advice of Schedule Change (ASC) | 스케줄 변경 통보 |
| `82` | Flight information movement | 항공편 이동 정보 (FLIFO) |
| `83` | Flight information diversion | 항공편 회항 정보 |
| `130` | Issue/sale | 발행/판매 |
| `134` | Exchange/reissue | 교환/재발행 |
| `135` | Refund | 환불 |

---

## 8. 약어 모음

| 약어 | 풀네임 | 설명 |
|------|--------|------|
| ASC | Advice of Schedule Change | 스케줄 변경 통보 |
| BPR | Boarding Pass | 탑승권 발급 |
| DCI | Departure Control Information | 출발 통제 정보 |
| FLIFO | Flight Information | 실시간 항공편 운항 정보 |
| IATCI | IATA Common Use Terminal Interface | IATA 공용 터미널 인터페이스 |
| ICAO | International Civil Aviation Organization | 국제민간항공기구 |
| NRL | New Record Locator | 새 예약 번호 통보 |
| PDM | Possible Duplicate Message | 중복 가능 메시지 |
| RBD | Reservation Booking Designator | 예약 등급 지시자 |
| RQR | Request for Reply | 응답 요청 |
| SSIM | Standard Schedules Information Manual | IATA 표준 스케줄 정보 매뉴얼 |
| SSR | Special Service Request | 특별 서비스 요청 |

---

!!! tip "관련 문서"
    - [Master Pricer Travelboard Search](master-pricer-travelboard-search.md) - 항공편 검색 API (검색 후 상세 조회 시 FlightInfo 활용)
    - [GDS 용어집](gds-terminology.md) - GDS 시스템 공통 용어
    - [BSP 정산 가이드](bsp-settlement-guide.md) - 발권·정산 프로세스
