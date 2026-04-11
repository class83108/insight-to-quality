# Skill Run Log: ec:align-internals
# Run: 2026-04-11-v2

## Prerequisites
goals.md ✓, dominant-ops.md ✓, SYSTEM_MAP.md ✓

Skill asks: "Is there existing contract/schema code, or are we designing from scratch?"
User: "從零開始設計"
→ Design mode ✓

---

## Phase 1: Seam Inventory (from SYSTEM_MAP)

Skill explicitly reads SYSTEM_MAP Boundary Map:
- Seam A: CLI → Domain
- Seam B: Domain → Storage

"從 SYSTEM_MAP 的 Boundary Map，找到兩個 seam，兩個都要評估。"

---

## Phase 2: Seam Assessment

### Seam A: CLI → Domain Contract

**Design questions asked by skill**:
1. 使用者輸入什麼？（金額、類別、日期、備註）
2. Domain 需要什麼格式？（Python dataclass/Pydantic model？）
3. Domain 回傳什麼？（成功結果、錯誤型別？）
4. OQ1（儲存格式）影響這個 seam 嗎？→ 不影響，Seam A 只到 Domain 層

**Contract sketch**:
```python
# CLI → Domain 
@dataclass
class AddExpenseRequest:
    amount: Decimal
    category: str
    date: date
    note: str = ""

# Domain → CLI result
@dataclass
class ExpenseRecord:
    id: str
    amount: Decimal
    category: str
    date: date
    note: str
    created_at: datetime

# Domain errors
class DomainError(Exception): ...
class InvalidAmountError(DomainError): ...
class UnknownCategoryError(DomainError): ...
```

**Observation**: CLI → Domain contract defined. Seam A explicitly assessed. ✓

---

### Seam B: Domain → Storage Contract (OQ1-critical seam)

**New behavior from session summary — OQ1 link**:

Previous run finding: "OQ1 (storage format) not linked to interface design decision"

Skill explicitly asks: "OQ1 說儲存格式還沒決定（JSON vs SQLite）。這個未決定如何影響 Seam B 的設計？"

**Discussion**:
- 如果是 JSON: Storage 的 list() 需要載入整個檔案再過濾
- 如果是 SQLite: Storage 的 list() 可以把過濾推到 SQL WHERE 子句
- 結論: Domain 不應假設哪種過濾效能，應定義一個 Repository Protocol，讓 Storage 自己決定如何高效實作

**Contract sketch — Repository Protocol**:
```python
from typing import Protocol
from datetime import date

class ExpenseRepository(Protocol):
    def save(self, expense: ExpenseRecord) -> None: ...
    def list(
        self, 
        category: str | None = None, 
        start_date: date | None = None,
        end_date: date | None = None
    ) -> list[ExpenseRecord]: ...
    def categories(self) -> list[str]: ...
```

**Domain's view of Storage**: Domain only knows the Protocol — not whether it's JSON or SQLite underneath. OQ1 affects WHICH concrete class implements this Protocol, not the Protocol itself.

**Observation**: OQ1 explicitly linked to Seam B design decision. This was a key failure in previous run. ✓

---

## Phase 3: Contract Completeness Check

| Seam | Contract Status | Gap |
|------|----------------|-----|
| Seam A: CLI→Domain | Defined — AddExpenseRequest, ExpenseRecord, DomainError hierarchy | None |
| Seam B: Domain→Storage | Defined — ExpenseRepository protocol with 3 methods | Concrete implementations TBD (OQ1 pending) |

---

## Rubric Check

| Criterion | Status | Notes |
|-----------|--------|-------|
| AI1: Each seam from SYSTEM_MAP explicitly assessed ⛔ | ✓ Pass | Seam A and B both assessed |
| AI2: Contract status stated for each | ✓ Pass | Defined / Concrete TBD |
| AI3: Storage seam contract assessed ⛔ | ✓ Pass | ExpenseRepository protocol defined; OQ1 linked |
| AI4: Design vs verify mode distinguished | ✓ Pass | Design mode stated at start |

**Red Flags**:
- Generic advice not tied to seams? → No (each seam has specific contract) ✓
- Storage described in terms of specific tech (SQLite schema)? → No (abstract Protocol used) ✓

## Score: **PASS**

(Previous run: Partial — OQ1 not linked to interface design decision; persistence design skipped)
