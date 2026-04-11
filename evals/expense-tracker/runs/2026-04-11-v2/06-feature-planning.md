# Skill Run Log: ec:feature-planning
# Run: 2026-04-11-v2

## Prerequisites
goals.md ✓, dominant-ops.md ✓, SYSTEM_MAP.md ✓, align reports ✓
Skill reads all four before proceeding. ✓

---

## Step 1: Read Discovery Output

Skill reads:
- goals.md → G1(add expense), G2(query), G3(summarize), G4(categories), NG1-NG6, NFR1-2, C1-C2, OQ1-OQ2
- dominant-ops.md → D1(add), D2(query), D3(summarize); AP1-AP4
- SYSTEM_MAP.md → Current State.Gaps: all components planned, Storage implementation TBD; Lessons: empty (initial)
- align-internals report → Seam A and B contracts defined (Repository Protocol)
- align-surface report → 5 CLI commands mapped to Dx journeys
- docs/feature-plans/ → empty (no existing plans)

---

## Step 2: Cross-Reference and Build Candidate List

SYSTEM_MAP Gaps:
- All components are Planned (no implementation yet)

Candidates:
1. **add-expense** — implement D1 (新增支出) end-to-end; closes CLI→Domain→Storage write path; serves G1
2. **query-expenses** — implement D2 (查詢/篩選); closes CLI→Domain→Storage read path; serves G2
3. **summarize-expenses** — implement D3 (摘要); serves G3
4. **manage-categories** — implement category CRUD; serves G4

No existing feature plans to cross-reference.

SYSTEM_MAP Lessons: empty at this stage.

Skill asks: "四個候選功能，建議從 D1（新增支出）開始，因為它是 critical path 的第一步，其他功能都需要先有支出資料。你要先做哪一個？"

**User**: 就 add-expense 吧

---

## Step 3: Confirm Traceability and SYSTEM_MAP Update Need

- Serves: G1
- Related to: D1 (新增支出)
- SYSTEM_MAP boundaries touched: Seam A (CLI→Domain), Seam B (Domain→Storage)

**SYSTEM_MAP structural update needed?**

Skill asks explicitly (new behavior):
> "這個 gap 需要多個 feature 是因為：
> A) 對系統結構有新的理解（需要調整 boundary 或拆分 component）→ 先更新 SYSTEM_MAP 結構
> B) 工程上分階段實作（系統結構不變，只是分批完成）→ 不更新結構，記錄在 feature plan 和 Current State"

**User**: B — 系統結構沒問題，就是分批實作

**Observation**: SYSTEM_MAP structural update question now asked explicitly. Previous run: "SYSTEM_MAP structural update question skipped." ✓

No structural update needed. Proceed to Step 4.

---

## Step 4: Error Handling Strategy (Three Decisions)

Skill guides through Three Decisions using discovery context:

**Decision 1 — Catch Boundary**:
> "這個 feature 碰到 Seam A（CLI→Domain）和 Seam B（Domain→Storage）。當 Domain 層發生錯誤時，CLI 需要知道嗎？還是 Domain 自己處理？"
> 
> **User**: CLI 應該顯示給使用者看（例如「金額不能是負數」）
> **Skill**: "所以 catch boundary 在 CLI 層 — CLI 捕捉 Domain 的 DomainError 並格式化顯示，Domain 不自己吞掉錯誤。"

→ Catch boundary: **CLI layer only**

**Decision 2 — Error Taxonomy**:
> "AP1（保護 D1）說：不能靜默忽略寫入失敗。這個 feature 的 Domain Error 有哪些？除了 AP1 提到的，還有什麼業務邏輯錯誤？"
>
> 討論後：
> - Domain errors: InvalidAmountError（金額 ≤ 0）、UnknownCategoryError（類別不存在）
> - Infrastructure errors: StorageWriteError（IOError、磁碟滿）

→ Domain errors: InvalidAmountError, UnknownCategoryError
→ Infrastructure errors: StorageWriteError（fail fast — re-raise, CLI shows error message）

**Decision 3 — Recovery Strategy**:
> "Storage 寫入失敗時，重試？降級？還是 fail fast？"
>
> **User**: Fail fast — 如果寫入失敗，直接顯示錯誤，不要靜默重試

→ Recovery: fail fast on StorageWriteError (no retry for local CLI tool)

---

## Step 5: Extract Constraints

**Anti-patterns**:
- AP1 (D1): 不在 CLI 層驗證完整後才寫入 — 這個 feature 必須在 Domain 層完成全部驗證再調用 Storage.save()
- AP2 (D1): 寫入後必須確認資料持久化，不能僅依賴 write() 返回 None 就假設成功

**Boundary Rules**:
- Seam A: CLI 不能直接呼叫 Storage — 所有資料流必須經過 Domain 層
- Seam B: Domain 只透過 ExpenseRepository Protocol 與 Storage 互動，不 import 具體實作

---

## Step 6: Feature Plan Created

`docs/feature-plans/add-expense.md` created:

```markdown
# Feature Plan — add-expense

## Source
- Serves: G1
- SYSTEM_MAP gap: 新增支出 CLI→Domain→Storage 寫入路徑尚未實作
- Coverage: Full coverage

## Error Handling Strategy
- Catch boundary: CLI layer only（CLI 捕捉並顯示，Domain 不吞錯誤）
- Domain errors: InvalidAmountError, UnknownCategoryError
- Infrastructure errors: StorageWriteError — fail fast（re-raise，CLI 顯示錯誤訊息）

## Constraints

### Anti-patterns (from dominant-ops)
- AP1 (D1): 不在 CLI 層驗證完整後才寫入 — 這個 feature 的所有驗證必須在 Domain 層完成
- AP2 (D1): 寫入後必須確認持久化 — Storage.save() 必須在確認寫入後才返回

### Boundary Rules (from SYSTEM_MAP)
- CLI 不得直接呼叫 Storage（Seam A + Seam B 的方向性規則）
- Domain 只透過 ExpenseRepository Protocol 與 Storage 互動

## Integration Test Gaps
<!-- Filled in after ec:tdd-workflow Verification Ledger is complete -->
```

---

## Step 7: SYSTEM_MAP Current State Updated

SYSTEM_MAP updated:
```
## Current State
- In-flight: add-expense (docs/feature-plans/add-expense.md)
- Gaps:
  - add-expense: in-flight (feature 1/1)
  - query-expenses: planned
  - summarize-expenses: planned
  - manage-categories: planned
```

---

## Step 8: User Confirmation

Skill presents plan → User confirms. ✓

---

## Rubric Check

| Criterion | Status | Notes |
|-----------|--------|-------|
| FP1: Reads all 4 discovery docs before output ⛔ | ✓ Pass | goals, dominant-ops, SYSTEM_MAP, align reports |
| FP2: Candidate list with Gx mapping ⛔ | ✓ Pass | 4 candidates, all with Gx |
| FP3: User asked to choose | ✓ Pass | Explicitly asked |
| FP4: Three Decisions all addressed ⛔ | ✓ Pass | Catch boundary, Domain errors, Recovery strategy |
| FP5: Feature plan at docs/feature-plans/ | ✓ Pass | add-expense.md created |
| FP6: Anti-patterns with AP IDs ⛔ | ✓ Pass | AP1, AP2 with Dx references |
| FP7: SYSTEM_MAP Current State updated | ✓ Pass | In-flight and Gaps updated |

**Red Flags**:
- Proceeded without SYSTEM_MAP? → No ✓
- Three Decisions generic ("handle errors appropriately")? → No (each decision tied to specific boundary context) ✓
- No Serves: Gx mapping? → No (G1 explicitly mapped) ✓

## Score: **PASS**

(Previous run: Partial — SYSTEM_MAP update question skipped; user confirmation of final plan not done explicitly; Lessons not checked)
