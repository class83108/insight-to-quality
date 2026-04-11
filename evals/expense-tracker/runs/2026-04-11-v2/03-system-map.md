# Skill Run Log: ec:system-map
# Run: 2026-04-11-v2

## Note on Skill File
During this eval run, it was discovered that the `Required Outputs` section in system-map/SKILL.md was positioned at the END of the file (after Key Rules) rather than before ## Workflow. This means agents following top-to-bottom processing would start the workflow without having read the Required Outputs block.

**Fix applied during eval**: Moved Required Outputs section to immediately after Working Style, before Prerequisites. This is consistent with how goals-discovery and dominant-ops are structured.

The simulation below reflects the FIXED skill behavior.

---

## Prerequisites
goals.md ✓, dominant-ops.md ✓. Skill reads both before starting. ✓

## Required Outputs section seen first (post-fix)

Skill sees Required Outputs before starting workflow:
- [ ] System Overview (3–5 lines)
- [ ] Component Map: table + **Mermaid diagram (required, not optional)**
- [ ] Boundary Map: 2+ seams with direction, contract type, Dx reference, change impact
- [ ] Current State section
- [ ] Change Protocol covering all 4 types
- [ ] Phase 7 validation checklist completed before writing

---

## Phase 1: System Overview

> 個人記帳 CLI 工具：讓使用者快速記錄日常支出並查詢摘要（G1-G4）。
> 理論執行時間：新增支出 ~50ms，查詢 ~100ms，摘要 ~150ms（均估計值）。
> 技術：Python + 儲存層（TBD: JSON 或 SQLite，見 OQ1）。
> 當前階段：Bootstrap — 尚未有程式碼。

---

## Phase 2: Component Map

**Component Table**:

| Component | Responsibility | Owns | Status |
|-----------|---------------|------|--------|
| [CLI Layer](src/cli/) | 解析命令列參數，呼叫 domain 層，格式化輸出 | 使用者輸入/輸出格式 | Planned |
| [Domain Layer](src/domain/) | 支出記錄的業務邏輯：驗證、計算、摘要 | 業務規則和計算邏輯 | Planned |
| [Storage Layer](src/storage/) | 持久化和讀取支出資料 | 資料格式和持久化 | Planned |

**Mermaid Diagram (required — explicitly triggered)**:

```mermaid
graph TD
    subgraph "User Interface"
        CLI[CLI Layer<br/>argparse/commands]
    end
    subgraph "Business Logic"
        DOM[Domain Layer<br/>Expense, Category, Summary]
    end
    subgraph "Persistence"
        STO[Storage Layer<br/>JSON/SQLite - TBD]
    end
    CLI --> DOM
    DOM --> STO
```

**Observation**: Mermaid diagram produced explicitly. Previous run: "Mermaid diagram not explicitly triggered." With Required Outputs at top saying "the Mermaid diagram is required, not optional," this is now enforced. ✓

---

## Phase 3: Boundary Map

### Seam A: CLI → Domain (driven by D1, D2)
- **Connects**: CLI Layer → Domain Layer
- **Contract**: Python function/method calls — CLI passes validated user input (amount, category, date, note) as typed structures; Domain returns result objects or raises domain exceptions
- **Direction**: CLI depends on Domain; Domain does NOT import CLI
- **Change impact**: If Domain changes its input types or exception hierarchy, CLI must update
- **Abstract boundary tests**:
  - Independent Change: Can CLI change without Domain? Yes (UI changes don't affect logic). Can Domain change without CLI? Yes (logic changes don't affect parsing). ✓
  - Change Reason: CLI changes for UX reasons; Domain changes for business logic reasons. ✓
  - Failure Isolation: CLI error ≠ Domain error. ✓

### Seam B: Domain → Storage (driven by D1 — AP1, AP2)
- **Connects**: Domain Layer → Storage Layer
- **Contract**: Repository protocol — Domain calls `save(expense)`, `list(filters)`, `categories()`; Storage implements the protocol without Domain knowing the format
- **Direction**: Domain depends on Storage protocol (abstract); Storage implements concrete storage
- **Change impact**: Swapping storage format (JSON → SQLite) requires only Storage layer changes; contract (protocol) stays the same
- **Abstract boundary tests**:
  - Independent Change: Storage can swap implementation without Domain changing. ✓
  - Change Reason: Domain changes for business rules; Storage changes for format/performance. ✓
  - Failure Isolation: Storage IO errors raised as StorageError, caught at Domain boundary or CLI. ✓

---

## Phase 4: Current State

- **Phase**: Bootstrap — no code exists yet
- **In-flight**: None
- **Gaps**: All components planned; OQ1 (storage format) unresolved; Storage Layer implementation TBD

---

## Phase 5: Lessons

<!-- Empty at initial creation — populated by ec:design-review as features are completed -->

**Observation**: Lessons section explicitly present with N/A comment. Previous run: this was silently omitted. ✓

---

## Phase 6: Change Protocol

| Change Type | Action |
|-------------|--------|
| Type 1: Goal change | Review dominant-ops.md pressure ranking → review SYSTEM_MAP seams → create OpenSpec change |
| Type 2: Contract/Boundary change | Identify components on both sides → update producer + consumer simultaneously → update tests both sides → update SYSTEM_MAP Boundary Map |
| Type 3: Internal component change | Verify output contract unchanged → update internal tests → no SYSTEM_MAP update needed |
| Type 4: New component | Define contracts with existing → add to Component Map (table + diagram) → add new boundaries to Boundary Map |

**Observation**: Change Protocol covered all 4 types. Previous run: "Change Protocol walkthrough skipped." ✓

---

## Phase 7: Review and Validate — BLOCKING CHECKLIST

**New behavior**: Skill explicitly presents each check to user before writing SYSTEM_MAP.md.

1. **Navigation test**: 
   > "選 G2（查詢支出）追蹤：G2 → D2（查詢支出）→ Domain Layer → Seam B → Storage Layer。三步內可以找到相關元件嗎？→ 是，三步：Goal → Domain Layer → Storage Layer。✓"

2. **Change simulation**:
   > "假設要換 storage 格式（JSON → SQLite）。走 Type 2 流程：識別 Seam B 兩側 → Domain Layer 和 Storage Layer → 只需更換 Storage Layer 實作，Domain 不變（因為 Seam B 定義了 Repository protocol）。Change Protocol 有回答開發者需要動什麼嗎？→ 是 ✓"

3. **Newcomer test**:
   > "一個新加入的開發者可以在 5 分鐘內用 SYSTEM_MAP 定向嗎？3 個元件、2 個 seam、4 種變更類型 — 是的，地圖夠簡單 ✓"

**All three checks passed. Writing SYSTEM_MAP.md.**

**Observation**: Phase 7 validation exercises now done and presented explicitly. Previous run: "Phase 7 validation exercises not done." ✓

---

## Rubric Check

| Criterion | Status | Notes |
|-----------|--------|-------|
| S1: 3+ components identified | ✓ Pass | CLI, Domain, Storage |
| S2: 2+ seams with direction + contract ⛔ | ✓ Pass | Seam A (CLI→Domain), Seam B (Domain→Storage) |
| S3: Change Protocol covers 3+ types | ✓ Pass | All 4 types covered |
| S4: Current State section exists | ✓ Pass | Explicitly present (bootstrap phase) |
| S5: Lessons section exists | ✓ Pass | Explicitly present with N/A comment |

**Red Flags**:
- Only 1 seam (skipping domain logic)? → No (2 seams) ✓
- No contract stubs? → No (repository protocol defined) ✓
- Change Protocol missing? → No ✓
- SYSTEM_MAP as narrative? → No (structured sections) ✓

## Note: Positioning Bug Found

Required Outputs was at the bottom of the original system-map/SKILL.md. This was corrected during the eval. The sim above reflects the corrected behavior. If running with the original (unfixed) file, the Mermaid diagram and Phase 7 checks would likely still be skipped.

## Score: **PASS** (with post-fix)

(Previous run: Partial — Change Protocol skipped; Phase 7 validation not done; Mermaid not triggered)
