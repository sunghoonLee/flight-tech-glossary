# Flight Tech Glossary

항공 기술 도메인의 용어를 정리하는 사전입니다.

---

## 카테고리

| 카테고리 | 설명 | 주요 키워드 |
|---------|------|------------|
| [GDS 용어집](gds-terminology.md) | Sabre & Amadeus GDS 시스템 용어 | PNR, PCC, OID, BSP, Webservice, WSAP |
| [BSP 정산 가이드](bsp-settlement-guide.md) | IATA BSP 정산 프로세스 전체 가이드 | ADM, ACM, DSR, Billing, NDC, RBD |
| [Master Pricer Travelboard Search](master-pricer-travelboard-search.md) | Amadeus 항공편 검색 API 기술 용어 | MPTBS, Recommendation, Fare Family, Mini Rules |
| [Air FlightInfo](air-flightinfo.md) | Amadeus 항공편 정보 조회 API 기술 용어 | FLIFO, Marriage Segment, Equipment, Meal Service |
| [용어 색인](glossary.md) | 전체 용어 알파벳순 색인 | A-Z |

---

## 용어 추가 방법

`docs/` 폴더에 새로운 `.md` 파일을 추가하고, `mkdocs.yml`의 `nav` 섹션에 등록하면 자동으로 사이트에 반영됩니다.

```
docs/
  ├── index.md              # 홈
  ├── gds-terminology.md    # GDS 용어집
  ├── bsp-settlement-guide.md  # BSP 정산 가이드
  ├── master-pricer-travelboard-search.md  # MPTBS 용어
  ├── air-flightinfo.md     # Air FlightInfo 용어
  ├── glossary.md           # 용어 색인
  └── (새 문서).md           # 새 카테고리 추가
```
