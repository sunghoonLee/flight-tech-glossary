# Flight Tech Glossary

항공 기술 도메인의 용어를 정리하는 사전입니다.

https://sunghoonlee.github.io/flight-tech-glossary/

## 카테고리

### 공통

| 카테고리 | 설명 | 주요 키워드 |
|---------|------|------------|
| GDS 용어집 | Sabre & Amadeus GDS 시스템 용어 | PNR, PCC, OID, BSP, Webservice, WSAP |
| BSP 정산 가이드 | IATA BSP 정산 프로세스 전체 가이드 | ADM, ACM, DSR, Billing, NDC, RBD |

### Amadeus

| 카테고리 | 설명 | 주요 키워드 |
|---------|------|------------|
| WBS Integration Flow | IBE 예약 전체 플로우 가이드 (검색→발권→정산→취소) | Calendar, TravelBoard, TST, FOP, SalesReports, Carbon |
| Master Pricer Travelboard Search | 항공편 검색 + 최저가 운임 조회 API | MPTBS, Recommendation, Fare Family, Mini Rules |
| Air FlightInfo | 항공편 정보 조회 API | FLIFO, Marriage Segment, Equipment, Meal Service |
| Air SellFromRecommendation | 검색 결과 기반 예약 생성 API | Segment Status, Marriage, Itinerary |
| Fare GetFareRules | 운임 규정 조회 API | ATPCO Rule Category, Fare Basis, Global Direction |
| PNR AddMultiElements | PNR 요소 추가/수정 API | SSR, OSI, FOP, Ticketing, Remarks |
| PNR Retrieve | PNR 조회 API | Record Locator, TST, DCS, History |

### 색인

| 카테고리 | 설명 |
|---------|------|
| 용어 색인 | 전체 용어 알파벳순 색인 (A-Z) |

## 용어 추가 방법

1. `docs/` 폴더에 새 `.md` 파일을 추가하거나 기존 파일을 편집합니다.
2. `mkdocs.yml`의 `nav` 섹션에 새 문서를 등록합니다.
3. `main` 브랜치에 push하면 GitHub Actions가 자동으로 사이트를 빌드·배포합니다.

```
docs/
├── index.md                            # 홈
├── gds-terminology.md                  # [공통] GDS 용어집
├── bsp-settlement-guide.md             # [공통] BSP 정산 가이드
├── amadeus-wbs-integration-flow.md     # [Amadeus] WBS Integration Flow
├── master-pricer-travelboard-search.md # [Amadeus] MPTBS 용어
├── air-flightinfo.md                   # [Amadeus] Air FlightInfo 용어
├── air-sell-from-recommendation.md     # [Amadeus] Air SellFromRecommendation 용어
├── fare-get-fare-rules.md              # [Amadeus] Fare GetFareRules 용어
├── pnr-add-multi-elements.md           # [Amadeus] PNR AddMultiElements 용어
├── pnr-retrieve.md                     # [Amadeus] PNR Retrieve 용어
├── glossary.md                         # 용어 색인
└── (새 문서).md                         # 새 카테고리 추가
```

## 로컬 개발

```bash
pip install mkdocs mkdocs-material
mkdocs serve
```

http://127.0.0.1:8000 에서 미리보기가 가능합니다.

## 기술 스택

- [MkDocs](https://www.mkdocs.org/) + [Material for MkDocs](https://squidfunk.github.io/mkdocs-material/)
- GitHub Actions → GitHub Pages 자동 배포
