# Flight Tech Glossary

항공 기술 도메인의 용어를 정리하는 사전입니다.

---

## 카테고리

### 공통

| 카테고리 | 설명 | 주요 키워드 |
|---------|------|------------|
| [GDS 용어집](gds-terminology.md) | Sabre & Amadeus GDS 시스템 용어 | PNR, PCC, OID, BSP, Webservice, WSAP |
| [BSP 정산 가이드](bsp-settlement-guide.md) | IATA BSP 정산 프로세스 전체 가이드 | ADM, ACM, DSR, Billing, NDC, RBD |

### Amadeus

| 카테고리 | 설명 | 주요 키워드 |
|---------|------|------------|
| [Master Pricer Travelboard Search](master-pricer-travelboard-search.md) | 항공편 검색 + 최저가 운임 조회 API | MPTBS, Recommendation, Fare Family, Mini Rules |
| [Air FlightInfo](air-flightinfo.md) | 항공편 정보 조회 API | FLIFO, Marriage Segment, Equipment, Meal Service |
| [Air SellFromRecommendation](air-sell-from-recommendation.md) | 검색 결과 기반 예약 생성 API | Segment Status, Marriage, Itinerary |
| [Fare GetFareRules](fare-get-fare-rules.md) | 운임 규정 조회 API | ATPCO Rule Category, Fare Basis, Global Direction |
| [PNR AddMultiElements](pnr-add-multi-elements.md) | PNR 요소 추가/수정 API | SSR, OSI, FOP, Ticketing, Remarks |
| [PNR Retrieve](pnr-retrieve.md) | PNR 조회 API | Record Locator, TST, DCS, History |

### 색인

| 카테고리 | 설명 | 주요 키워드 |
|---------|------|------------|
| [용어 색인](glossary.md) | 전체 용어 알파벳순 색인 | A-Z |

---

## 용어 추가 방법

`docs/` 폴더에 새로운 `.md` 파일을 추가하고, `mkdocs.yml`의 `nav` 섹션에 등록하면 자동으로 사이트에 반영됩니다.

```
docs/
  ├── index.md                            # 홈
  ├── gds-terminology.md                  # [공통] GDS 용어집
  ├── bsp-settlement-guide.md             # [공통] BSP 정산 가이드
  ├── master-pricer-travelboard-search.md # [Amadeus] MPTBS 용어
  ├── air-flightinfo.md                   # [Amadeus] Air FlightInfo 용어
  ├── air-sell-from-recommendation.md     # [Amadeus] Air SellFromRecommendation 용어
  ├── fare-get-fare-rules.md              # [Amadeus] Fare GetFareRules 용어
  ├── pnr-add-multi-elements.md           # [Amadeus] PNR AddMultiElements 용어
  ├── pnr-retrieve.md                     # [Amadeus] PNR Retrieve 용어
  ├── glossary.md                         # 용어 색인
  └── (새 문서).md                         # 새 카테고리 추가
```
