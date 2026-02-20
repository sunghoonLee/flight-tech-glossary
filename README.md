# Travel Tech Glossary

항공 및 여행 기술 도메인의 용어를 정리하는 사전입니다.

https://sunghoonlee.github.io/travel-tech-glossary/

## 카테고리

| 카테고리 | 설명 |
|---------|------|
| GDS 용어집 | Sabre & Amadeus GDS 시스템 용어 |
| BSP 정산 가이드 | IATA BSP 정산 프로세스 전체 가이드 |
| 용어 색인 | 전체 용어 알파벳순 색인 |

## 용어 추가 방법

1. `docs/` 폴더에 새 `.md` 파일을 추가하거나 기존 파일을 편집합니다.
2. `mkdocs.yml`의 `nav` 섹션에 새 문서를 등록합니다.
3. `main` 브랜치에 push하면 GitHub Actions가 자동으로 사이트를 빌드·배포합니다.

```
docs/
├── index.md                 # 홈
├── gds-terminology.md       # GDS 용어집
├── bsp-settlement-guide.md  # BSP 정산 가이드
└── glossary.md              # 용어 색인
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
