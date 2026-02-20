# 용어 색인

전체 용어를 알파벳/가나다순으로 정리한 색인입니다.

---

## A

**Air_FlightInfo**
:   Amadeus API. 항공편 번호·날짜 기반으로 스케줄, 기재, 좌석 현황, 기내 서비스 등 상세 운항 정보를 조회. → [Air FlightInfo](air-flightinfo.md#1-개요)

**Air_SellFromRecommendation**
:   Amadeus API. MPTBS 검색 결과(Recommendation)를 기반으로 실제 항공편 예약(Booking)을 생성. → [Air SellFromRecommendation](air-sell-from-recommendation.md)

**ASC** (Advice of Schedule Change)
:   항공편 스케줄 변경 통보. GDS를 통해 여행사에 자동 전달. → [Air FlightInfo](air-flightinfo.md#7-메시지-기능-코드-주요)

**ATPCO** (Airline Tariff Publishing Company)
:   항공 운임 데이터를 관리·배포하는 기관. Fare Rule, 수하물 규정, Amenity 데이터 등 제공. → [MPTBS](master-pricer-travelboard-search.md#약어-모음), [Fare GetFareRules](fare-get-fare-rules.md)

**ACM** (Agent Credit Memo)
:   항공사가 여행사에 환급하는 문서. BSP 정산에서 차감 처리. → [BSP 정산 가이드](bsp-settlement-guide.md#4-핵심-문서전표)

**ADM** (Agent Debit Memo)
:   항공사가 여행사에 추가 청구하는 문서. 운임 차액 등 발생 시 발행. → [BSP 정산 가이드](bsp-settlement-guide.md#4-핵심-문서전표)

**AQO** (Access Queue Office)
:   Amadeus에서 OID 간 큐(Queue) 접근 권한을 설정하는 기능. → [Amadeus 용어](gds-terminology.md#4-amadeus-용어)

## B

**Base Fare**
:   세금·할증료를 제외한 기본 항공 운임. → [운임 구조](bsp-settlement-guide.md#10-운임-구조-fare-structure)

**Booking Class**
:   → RBD 참조

**Branch**
:   Sabre에서 PCC 간 연결/접근 권한 설정. Amadeus의 EOS에 대응. → [Sabre 용어](gds-terminology.md#3-sabre-용어)

**BSP** (Billing and Settlement Plan)
:   IATA가 운영하는 항공사-여행사 간 대금 정산 중개 시스템. → [BSP 개요](bsp-settlement-guide.md#1-bsp-개요)

**BSPlink**
:   BSP 온라인 포털. 여행사·항공사가 정산 내역을 조회·관리. → [BSP 구성 요소](bsp-settlement-guide.md#3-bsp-주요-구성-요소)

## C

**Carbon Footprint** (탄소 배출량)
:   항공편 운항으로 발생하는 CO2 배출 추정치. ICAO 또는 Travalyst(TVYS) 방법론으로 계산. MasterPricer에서 `carbonEmissionBySourceDetails` 추가로 조회 가능. → [WBS Integration Flow](amadeus-wbs-integration-flow.md)

**CTCE** (Contact Email)
:   IATA Mandate 830d에 따른 승객 이메일 SSR. `@`→`//`, `_`→`..` 변환 규칙 적용. → [WBS Integration Flow](amadeus-wbs-integration-flow.md)

**CTCM** (Contact Mobile)
:   IATA Mandate 830d에 따른 승객 휴대전화 SSR. 국가코드/국가ISO 형식. → [WBS Integration Flow](amadeus-wbs-integration-flow.md)

**Cert Mode** (Certification Mode)
:   Sabre 시스템 연동 시 테스트/인증 환경. → [Sabre 용어](gds-terminology.md#3-sabre-용어)

**Client-ID**
:   Sabre API 클라이언트 식별자. OAuth 인증 시 사용. → [Sabre Credential](gds-terminology.md#3-sabre-용어)

**Cabin Class** (캐빈 등급)
:   항공기 객실 등급. F=First, J/C=Business, W=Premium Economy, Y=Economy. → [Air FlightInfo](air-flightinfo.md#4-4-equipment-information-기재-정보)

**Codeshare** (공동운항)
:   하나의 항공편에 여러 항공사 편명이 붙는 것. → [Codeshare & Interline](bsp-settlement-guide.md#17-codeshare--interline)

**Coupon**
:   항공권 내 개별 구간. 하나의 항공권에 최대 4개. → [항공권 번호 체계](bsp-settlement-guide.md#8-항공권-번호-체계)

## D

**Days of Operation** (운항 요일)
:   항공편이 운항하는 요일을 7자리 문자열로 표현. 각 자리가 월~일 (예: `1234567`=매일). → [Air FlightInfo](air-flightinfo.md#4-2-additional-product-details-부가-상품-정보)

**Date Variation** (날짜 차이)
:   출발일 대비 도착일의 차이. 야간 비행·시차로 도착일이 다를 때 사용 (0=당일, 1=+1일). → [MPTBS](master-pricer-travelboard-search.md#10-기타-주요-용어)

**DSR** (Data Storage & Retrieval)
:   Amadeus 시스템의 예약/발권 데이터 저장·조회 시스템. → [Amadeus 용어](gds-terminology.md#4-amadeus-용어)

## E

**EMD** (Electronic Miscellaneous Document)
:   부가 서비스 전표. 수하물 추가, 좌석 업그레이드 등. → [핵심 문서/전표](bsp-settlement-guide.md#4-핵심-문서전표)

**EOS** (Extended Office Setup)
:   Amadeus에서 OID 간 연결/접근 권한 설정. Sabre의 Branch에 대응. → [Amadeus 용어](gds-terminology.md#4-amadeus-용어)

**EOT** (End of Transaction)
:   PNR 저장 완료 명령. PNR_AddMultiElements에서 옵션 코드 `ET`로 지정. → [PNR AddMultiElements](pnr-add-multi-elements.md)

**EPR** (Electronic Point of Reference)
:   Sabre API 접속 시 사용하는 엔드포인트 식별자. → [Sabre Credential](gds-terminology.md#3-sabre-용어)

**Equipment Type** (기재 유형)
:   항공기 기종을 나타내는 IATA 3자리 코드. 예: 388=A380, 789=B787-9, 77W=B777-300ER. → [MPTBS](master-pricer-travelboard-search.md#10-기타-주요-용어)

## F

**Fare_InformativeBestPricingWithoutPNR**
:   Amadeus API. PNR 없이 여정과 승객 정보만으로 최적 운임을 자동 계산(Best Pricer). → [WBS Integration Flow](amadeus-wbs-integration-flow.md#step-3-fare_informativebestpricingwithoutpnr)

**Fare_InformativePricingWithoutPNR**
:   Amadeus API. PNR 없이 특정 운임을 확정 조회. `uniqueOfferReference`를 반환하여 규정 조회·발권에 활용. → [WBS Integration Flow](amadeus-wbs-integration-flow.md#step-4-fare_informativepricingwithoutpnr)

**Fare_MasterPricerCalendar**
:   Amadeus API. 기준 날짜 전후 범위의 최저가를 달력 형태로 탐색. → [WBS Integration Flow](amadeus-wbs-integration-flow.md#step-1-fare_masterpricercalendar)

**Fare_PricePNRWithBookingClass**
:   Amadeus API. PNR 세그먼트 기반 운임 계산 후 TST(Transitional Stored Ticket) 생성. 발권 직전 운임 확정 단계. → [WBS Integration Flow](amadeus-wbs-integration-flow.md#step-10-fare_pricepnrwithbookingclass)

**FLIFO** (Flight Information)
:   항공편의 실시간 운항 정보. 게이트 변경, 지연/결항, 실제 출발/도착 시간 등 제공. → [Air FlightInfo](air-flightinfo.md#6-flifo-flight-information)

**Flight Indicator** (항공편 유형 지시자)
:   항공편의 서비스 유형을 나타내는 코드. D=직항, N=무기착, A=코드셰어, F=전세기 등. → [Air FlightInfo](air-flightinfo.md#flight-indicator-코드)

**Fare_GetFareRules**
:   Amadeus API. ATPCO Rule Category별 운임 규정(변경/환불 조건, 최소체류기간 등)을 조회. → [Fare GetFareRules](fare-get-fare-rules.md)

**Fare Basis Code**
:   항공 운임의 규정을 나타내는 코드. 예: YOWKR, HLE3M. → [운임 구조](bsp-settlement-guide.md#10-운임-구조-fare-structure)

**Fare Family** (브랜드 운임)
:   항공사가 부가서비스 포함 여부에 따라 운임을 등급화한 상품 체계. 예: LIGHT, STANDARD, FLEX. → [MPTBS](master-pricer-travelboard-search.md#5-fare-family-브랜드-운임)

**FOP** (Form of Payment)
:   결제 수단. 현금, 신용카드, 마일리지 등 항공권 결제에 사용되는 지불 방식. → [MPTBS](master-pricer-travelboard-search.md#약어-모음), [PNR AddMultiElements](pnr-add-multi-elements.md), [WBS Integration Flow](amadeus-wbs-integration-flow.md)

**FOP_CreateFormOfPayment**
:   Amadeus API. PNR에 결제 수단을 등록. 카드 정보를 FortKnox에 토큰화 저장. 발권 전 필수 단계. → [WBS Integration Flow](amadeus-wbs-integration-flow.md#step-9-fop_createformofpayment)

**FortKnox**
:   Amadeus 내부 보안 카드 정보 저장소. 카드 번호를 NOX 토큰으로 대체하여 PCI-DSS 준수. → [WBS Integration Flow](amadeus-wbs-integration-flow.md#step-9-fop_createformofpayment)

## G

**GDS** (Global Distribution System)
:   항공·호텔·렌터카 등 여행 상품을 예약하는 글로벌 유통 시스템. Amadeus, Sabre, Travelport 등. → [공통 기본 용어](gds-terminology.md#1-공통-기본-용어)

**Global Direction** (글로벌 방향)
:   운임 적용 방향. AT=대서양, PA=태평양, EH=동반구, WH=서반구 등. → [Fare GetFareRules](fare-get-fare-rules.md)

**GOL** (Global Online License)
:   온라인 여행사 전용 IATA 인증. → [규제 및 인증](bsp-settlement-guide.md#19-규제-및-인증)

**gscCode** (Global Service Code)
:   Sabre 서비스 식별 코드. → [Sabre Credential](gds-terminology.md#3-sabre-용어)

## H

**HOT** (Handing Over Time)
:   BSP 데이터 마감 시점. → [BSP 동작 흐름](bsp-settlement-guide.md#2-bsp-동작-흐름)

## I

**IATA** (International Air Transport Association)
:   국제항공운송협회. BSP 운영 기관. → [BSP 구성 요소](bsp-settlement-guide.md#3-bsp-주요-구성-요소)

**ICAO** (International Civil Aviation Organization)
:   국제민간항공기구. 항공 안전·보안 국제 표준 수립. → [Air FlightInfo](air-flightinfo.md#8-약어-모음)

**Interline** (타항공사 연결)
:   서로 다른 항공사 구간을 하나의 항공권으로 발권. → [Codeshare & Interline](bsp-settlement-guide.md#17-codeshare--interline)

**IT Fare** (Inclusive Tour Fare)
:   패키지 전용 운임. 단독 판매 불가. → [운임 구조](bsp-settlement-guide.md#10-운임-구조-fare-structure)

## L

**Last Ticketing Date** (발권 마감일)
:   TST에 기록된 운임으로 발권 가능한 최종 기한. 초과 시 TST 만료, 재운임 계산 필요. → [WBS Integration Flow](amadeus-wbs-integration-flow.md#step-10-fare_pricepnrwithbookingclass)

**LSA** (Last Seat Available)
:   마지막 남은 좌석에서도 해당 운임으로 예약 가능한지 여부. → [MPTBS](master-pricer-travelboard-search.md#10-기타-주요-용어)

## M

**Marriage Segment** (결합 구간)
:   GDS에서 여러 구간이 하나의 여정으로 결합된 상태. 개별 취소/변경 불가, 전체를 함께 처리해야 함. → [Air FlightInfo](air-flightinfo.md#4-5-marriage-control-결합-구간)

**Marketing Carrier** (마케팅 항공사)
:   항공권을 판매하는 항공사. 편명을 소유. 코드셰어에서 Operating Carrier와 다를 수 있다. → [MPTBS](master-pricer-travelboard-search.md#marketing-carrier-vs-operating-carrier)

**MCO** (Miscellaneous Charges Order)
:   기타 요금 전표. 페널티 등. → [핵심 문서/전표](bsp-settlement-guide.md#4-핵심-문서전표)

**Meta Channel** (메타채널)
:   여러 항공사/여행사의 가격을 비교해주는 플랫폼. 스카이스캐너, 카약, 구글 플라이트 등. → [공통 기본 용어](gds-terminology.md#1-공통-기본-용어)

**Mini Rules** (MNR)
:   운임의 변경/환불 규정 요약 정보. 전체 Fare Rule 없이 핵심 규정을 빠르게 확인. → [MPTBS](master-pricer-travelboard-search.md#4-mini-rules)

**MPTBS** (Master Pricer Travelboard Search)
:   Amadeus GDS의 항공편 검색 + 최저가 운임 조회 API. → [MPTBS](master-pricer-travelboard-search.md#1-개요)

## N

**NUC** (Neutral Unit of Construction)
:   IATA 운임 계산 기준 통화. 국제선 운임 계산 시 각국 통화를 NUC로 환산 후 합산하여 최종 통화로 변환. → [WBS Integration Flow](amadeus-wbs-integration-flow.md#step-2-fare_masterpricertravelboardsearch)

**NDC** (New Distribution Capability)
:   IATA가 만든 항공사-여행사 간 직접 연결 표준. GDS 중개 없이 API 직접 연결. → [NDC](bsp-settlement-guide.md#14-ndc-new-distribution-capability)

**Net Fare**
:   수수료 제외한 순수 운임. 여행사 마진 별도. → [운임 구조](bsp-settlement-guide.md#10-운임-구조-fare-structure)

## O

**Operating Carrier** (운항 항공사)
:   실제 항공편을 운항하는 항공사. 코드셰어에서 Marketing Carrier와 다를 수 있다. → [MPTBS](master-pricer-travelboard-search.md#marketing-carrier-vs-operating-carrier)

**OID** (Office ID)
:   Amadeus에서 업체를 식별하는 9자리 영숫자 코드. Sabre의 PCC에 대응. → [공통 기본 용어](gds-terminology.md#1-공통-기본-용어)

**OSI** (Other Service Information)
:   항공사에 전달하는 참고 정보. SSR과 달리 요청이 아닌 알림. → [SSR / OSI](bsp-settlement-guide.md#16-ssr--osi)

## P

**PAS** (Payment & Settlement)
:   Amadeus의 결제 및 정산 시스템. Sabre의 Payment Module에 대응. → [Amadeus 용어](gds-terminology.md#4-amadeus-용어)

**PCC** (Pseudo City Code)
:   Sabre에서 업체를 식별하는 4자리 영숫자 코드. Amadeus의 OID에 대응. → [공통 기본 용어](gds-terminology.md#1-공통-기본-용어)

**pccEnc** (PCC Encrypted)
:   암호화된 PCC 값. 결제 모듈 등에서 보안 목적으로 사용. → [Sabre Credential](gds-terminology.md#3-sabre-용어)

**PDT Mode** (Product Mode)
:   Amadeus 시스템의 운영 환경 모드. Sabre의 Cert Mode에 대응. → [Amadeus 용어](gds-terminology.md#4-amadeus-용어)

**PNR** (Passenger Name Record)
:   항공 예약 기록의 기본 데이터 단위. 승객 정보, 여정, 연락처, 발권 정보 포함. → [공통 기본 용어](gds-terminology.md#1-공통-기본-용어), [PNR AddMultiElements](pnr-add-multi-elements.md), [PNR Retrieve](pnr-retrieve.md)

**PNR_AddMultiElements**
:   Amadeus API. PNR에 승객 이름, 여정, SSR, OSI, 발권 정보, Remarks 등 다양한 요소를 추가·수정. → [PNR AddMultiElements](pnr-add-multi-elements.md)

**PNR_Retrieve**
:   Amadeus API. 예약 번호(Record Locator) 기반으로 PNR 전체 또는 특정 요소를 조회. → [PNR Retrieve](pnr-retrieve.md)

**Purge Date** (삭제 예정일)
:   PNR이 GDS에서 자동 삭제되는 날짜. 일반적으로 최종 구간 출발 후 일정 기간 경과 시. → [PNR Retrieve](pnr-retrieve.md)

**PTC** (Passenger Type Code)
:   승객 유형 코드. ADT=성인, CHD=소아, INF=유아. 운임 산정 기준. → [MPTBS](master-pricer-travelboard-search.md#ptc-passenger-type-code)

**PNR_Split Service**
:   Amadeus에서 PNR을 분리하는 서비스. → [Amadeus 용어](gds-terminology.md#4-amadeus-용어)

**Private Fare**
:   특정 여행사/기업 계약 할인 운임. PCC/OID에 매핑. → [운임 구조](bsp-settlement-guide.md#10-운임-구조-fare-structure)

**Published Fare**
:   항공사 공시 정가 운임. GDS에서 조회 가능. → [운임 구조](bsp-settlement-guide.md#10-운임-구조-fare-structure)

## Q

**Queue**
:   GDS에서 여행사에 전달되는 메시지 대기열. 일정 변경, 발권 기한 등 알림. → [Queue 시스템](bsp-settlement-guide.md#12-queue-시스템)

## R

**Record Locator** (예약 번호)
:   PNR을 식별하는 6자리 영숫자 코드. Amadeus에서는 RL, Sabre에서는 Confirmation Number라고도 함. → [PNR Retrieve](pnr-retrieve.md)

**Recommendation**
:   MPTBS 검색 결과의 단위. 하나의 여정 조합 + 운임 정보를 포함. → [MPTBS](master-pricer-travelboard-search.md#recommendation)

**RBD** (Reservation Booking Designator)
:   좌석 등급과 운임 수준을 나타내는 알파벳 1자리. F/J/Y/B/H/L 등. → [Booking Class & RBD](bsp-settlement-guide.md#13-booking-class--rbd)

**RD** (Remittance Date)
:   실제 대금 납부일. → [BSP 동작 흐름](bsp-settlement-guide.md#2-bsp-동작-흐름)

**Rule Category** (운임 규정 카테고리)
:   ATPCO가 정의한 운임 규정 분류. Cat 5=사전구매, Cat 6=최소체류, Cat 16=벌금, Cat 31=자발적 변경, Cat 33=자발적 환불. → [Fare GetFareRules](fare-get-fare-rules.md)

**RFIC** (Reason For Issuance Code)
:   EMD 발급 사유를 나타내는 코드. A=Air Transportation, C=Baggage 등. → [MPTBS](master-pricer-travelboard-search.md#rfic-reason-for-issuance-code)

**REISSUE** (재발행)
:   일정/구간 변경으로 항공권을 재발행. 정산대사에서 가장 복잡한 유형. → [발권 트랜잭션 유형](bsp-settlement-guide.md#11-발권-트랜잭션-유형)

**RP** (Remittance Period)
:   BSP 정산 주기. → [BSP 동작 흐름](bsp-settlement-guide.md#2-bsp-동작-흐름)

## S

**SalesReports_DisplayQueryReport**
:   Amadeus API. 오피스 판매 실적 보고서 조회. AlternativeCurrency(외화) 또는 DFC(기본 통화) 모드 지원. → [WBS Integration Flow](amadeus-wbs-integration-flow.md#step-13-salesreports-alternativecurrency)

**Segment Status** (구간 상태 코드)
:   항공편 구간의 예약 상태. HK=확약, NN=요청중, UC=좌석불가, WL=대기. → [Air SellFromRecommendation](air-sell-from-recommendation.md), [PNR AddMultiElements](pnr-add-multi-elements.md)

**SSIM** (Standard Schedules Information Manual)
:   IATA 표준 스케줄 정보 매뉴얼. 항공편 스케줄 데이터의 국제 표준 형식. → [Air FlightInfo](air-flightinfo.md#8-약어-모음)

**Sellconnect**
:   Amadeus의 데스크톱 예약 도구 연결 설정. Sabre의 SR360에 대응. → [Amadeus 용어](gds-terminology.md#4-amadeus-용어)

**SR360** (Sabre Red 360)
:   Sabre의 데스크톱 예약 에이전트 도구. → [Sabre 용어](gds-terminology.md#3-sabre-용어)

**SSR** (Special Service Request)
:   항공사에 특별 서비스를 요청하는 코드. WCHR, VGML, DOCS 등. → [SSR / OSI](bsp-settlement-guide.md#16-ssr--osi)

## T

**Tattoo** (타투)
:   PNR 내 각 요소(구간, SSR 등)에 부여되는 고유 식별 번호. 요소 참조 시 사용. → [PNR Retrieve](pnr-retrieve.md)

**TIDS** (Travel Industry Designator Service)
:   비IATA 여행사 등록 서비스. → [규제 및 인증](bsp-settlement-guide.md#19-규제-및-인증)

**TKT** (Ticket)
:   발권된 항공권. BSP를 통해 정산. → [핵심 문서/전표](bsp-settlement-guide.md#4-핵심-문서전표)

**Ticket_CancelDocument**
:   Amadeus API. 발행된 전자항공권을 취소. PNR_Cancel 전에 반드시 먼저 호출해야 하며, 티켓 수만큼 개별 호출 필요. → [WBS Integration Flow](amadeus-wbs-integration-flow.md#step-15-cancellation-flow)

**TST** (Transitional Stored Ticket)
:   Amadeus에서 운임 계산 결과를 PNR에 저장하는 레코드. 발권 전 운임 정보 보관. 승객 유형별(ADT/CHD/INF) 각각 생성. → [PNR Retrieve](pnr-retrieve.md), [WBS Integration Flow](amadeus-wbs-integration-flow.md#step-10-fare_pricepnrwithbookingclass)

**TTP** (Ticket Transaction Processing)
:   Amadeus 발권 처리 명령. TST 기반으로 전자항공권 발행. 실행 시 FOP의 카드 승인이 함께 처리됨. → [WBS Integration Flow](amadeus-wbs-integration-flow.md#step-12-pnr_retrieve-post-issuance)

**Tech Stop** (기술 착륙)
:   여객 탑승/하차 없이 연료 보급 등 목적으로 경유하는 공항. → [MPTBS](master-pricer-travelboard-search.md#10-기타-주요-용어)

## V

**Virtual Interlining** (가상 인터라인)
:   인터라인 계약이 없는 항공사 간 여정을 별도 항공권으로 각각 발권하되 하나의 여정으로 조합하는 기능. → [MPTBS](master-pricer-travelboard-search.md#6-virtual-interlining)

**VOID**
:   발권 당일 항공권 무효 처리. BSP에 리포팅되지 않음. → [발권 트랜잭션 유형](bsp-settlement-guide.md#11-발권-트랜잭션-유형)

## W

**WSAP** (Web Services Access Point)
:   Amadeus 웹서비스(API) 접속 포인트 설정. Sabre Webservice에 대응. → [Amadeus 용어](gds-terminology.md#4-amadeus-용어)

---

!!! tip "용어 추가"
    새로운 용어를 추가할 때는 알파벳 순서에 맞게 배치하고, 관련 문서 링크를 함께 달아주세요.
