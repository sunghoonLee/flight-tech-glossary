# Cancellation Flow

발권 완료 후 **예약 및 티켓을 취소**하는 3단계 플로우 용어집입니다.
`PNR_Retrieve` → `Ticket_CancelDocument` → `PNR_Cancel` 순서로 진행합니다.

!!! info "WBS Integration Flow Step 15"
    이 플로우는 [WBS Integration Flow](amadeus-wbs-integration-flow.md)의 **후속 처리 단계 Step 15**에 해당합니다.

---

!!! warning "추후 업데이트 예정"
    상세 용어 정리는 추후 업데이트될 예정입니다.

---

## 개요

| 항목 | 내용 |
|------|------|
| 플로우 단계 | Step 15 — 후속 처리 |
| 목적 | 발권된 티켓 및 PNR 취소 |
| 구성 API | `PNR_Retrieve` → `Ticket_CancelDocument` → `PNR_Cancel` |

---

## 취소 순서

```
PNR_Retrieve → Ticket_CancelDocument (티켓 수만큼 반복) → PNR_Cancel
```

!!! warning "순서 주의"
    발권된 티켓이 있는 경우 **반드시 `Ticket_CancelDocument`를 먼저** 호출해야 합니다.
    티켓 취소 없이 `PNR_Cancel`만 호출하면 오류가 발생합니다.

---

## 참고

- [WBS Integration Flow - Step 15](amadeus-wbs-integration-flow.md)
- [PNR_Retrieve 용어집](pnr-retrieve.md)
