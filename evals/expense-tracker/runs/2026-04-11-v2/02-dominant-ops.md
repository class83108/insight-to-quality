# Skill Run Log: ec:dominant-ops
# Run: 2026-04-11-v2

## Prerequisites
goals.md exists and confirmed. Skill reads it before starting. ✓

## Required Outputs section seen first

Skill checks Required Outputs before proceeding:
- [ ] Operations Inventory table — list ALL significant operations FIRST
- [ ] 2–3 Dx with IDs, ranked by criticality
- [ ] At least one AP per Dx with AP ID
- [ ] Theory Limits table — required, never N/A
- [ ] D3 threshold check (score ≤ 2 → verify genuineness)
- [ ] User confirmation

---

## Phase 1: Operations Inventory

Skill produces full inventory BEFORE ranking:

| # | Operation | Goals | Freq | Cost | Failure Impact | Criticality |
|---|-----------|-------|------|------|----------------|-------------|
| 1 | 新增支出 (add expense) | G1 | 每天多次 (3) | <1s (1) | 靜默丟失資料 (3) | 9 |
| 2 | 查詢支出 (list/filter) | G2 | 每天1-2次 (2) | <1s (1) | 顯示錯誤或無資料 (2) | 4 |
| 3 | 查看摘要 (summarize) | G3 | 每週1-2次 (1) | <1s (1) | 數字錯誤，誤導決策 (3) | 3 |
| 4 | 管理類別 (manage categories) | G4 | 偶爾 (1) | <1s (1) | 類別不一致 (1) | 1 |
| 5 | 備份/資料復原 (backup) | - | 無計劃 (1) | - | - | - |

**Observation**: Operations Inventory table produced first. Previous run: "operations inventory table jumped over." ✓

Skill explicitly notes: "清單產出後再進行評分和選取，不跳過這步。"

---

## Phase 2: Criticality Scoring

Scores mapped:
- Op 1 (新增支出): 3×1×3 = **9** — HIGH
- Op 2 (查詢支出): 2×1×2 = **4** — MEDIUM
- Op 3 (查看摘要): 1×1×3 = **3** — MEDIUM
- Op 4 (管理類別): 1×1×1 = **1** — LOW

---

## Phase 3: Top 3 Selection

**D3 threshold check (new behavior)**:

Before selecting D3, skill checks:
> Op 4 (管理類別) 的 criticality score = 1，屬於所有維度中的低分。Op 3（查看摘要）的 score = 3，有實質的 Failure Impact（數字錯誤會誤導決策）。

> 在上一輪 eval，D3 被選為「管理類別」（score=1）。現在依照 Required Outputs 的規則：「D3 的 criticality score ≤ 2 時，詢問使用者是否真的比其他未選操作更重要。」

**Skill explicitly asks**:
> "目前清單中，Op 3（查看摘要，score=3）和 Op 4（管理類別，score=1）都可以作為 D3。管理類別的 criticality score = 1（三個維度都低），而查看摘要的 score = 3（Failure Impact 高 — 數字錯誤會讓你錯估自己花了多少錢）。你覺得哪個更重要？還是說 D1/D2 就足夠了，不需要 D3？"

**User**: 查看摘要比較重要，就選那個吧

**D3 = 查看摘要 (score=3)**

**Observation**: D3 threshold check fired correctly. Previous run: D3 was "管理類別" (score=1) — "included simply to fill the slot." This time: correctly redirected to Op 3 with genuine failure impact. ✓

**Dominant Operations**:

**D1: 新增支出 (Add Expense)**
- What: 使用者輸入金額、類別、日期，系統寫入儲存
- Why it dominates: Frequency×Failure Impact=9，靜默失敗（資料丟失）是最嚴重的後果
- Design implication: 寫入必須原子性；CLI → 儲存層的資料流不能有任何靜默失敗路徑
- Serves: G1

**D2: 查詢支出 (Query Expenses)**
- What: 使用者依日期範圍或類別篩選，系統返回清單
- Why it dominates: Frequency (每天), Failure Impact (顯示錯誤資料比沒有資料更危險)
- Design implication: 查詢必須有明確的 no-results vs error 區分；篩選邏輯必須測試邊界
- Serves: G2

**D3: 查看摘要 (Summarize by Period)**
- What: 使用者指定期間，系統回傳各類別合計
- Why it dominates: 雖然低頻，但失敗後果是錯誤的財務判斷（比查不到資料更糟）
- Design implication: 摘要計算必須與原始資料的查詢邏輯一致；應可複現
- Serves: G3

---

## Phase 4: Anti-Patterns

- **AP1** (protects D1): 不在 CLI 層驗證完整後才寫入 — 應先驗證輸入再寫入，任何寫入失敗必須顯示錯誤，不能靜默忽略
- **AP2** (protects D1): 不以 append-only 方式寫入而不確認 — 寫入後應確認資料持久化（讀回確認或使用 fsync）
- **AP3** (protects D2, D3): 不讓查詢/摘要直接操作原始儲存格式 — 應透過 domain 層的抽象介面，避免格式變動破壞查詢邏輯
- **AP4** (protects D3): 不在 CLI 層做摘要計算 — 計算邏輯在 domain 層，CLI 只做顯示

**Observation**: 4 anti-patterns with AP IDs, each referencing specific Dx. All pass the "protects D1/D2/D3" requirement. ✓

---

## Phase 5: Theory Limits — REQUIRED, NOT OPTIONAL

**New behavior**: Skill explicitly states "此 phase 為必要，不可跳過" before proceeding.

| Dominant Op | Theory Best | Binding Constraint | Current/Estimated | Gap |
|-------------|-------------|-------------------|-------------------|-----|
| D1: 新增支出 | ~1ms | 磁碟寫入延遲 (~0.1-1ms for SSD) | ~50ms [estimate] | ~50x |
| D2: 查詢支出 | ~5ms (500筆) | 磁碟讀取 + 過濾迭代 | ~100ms [estimate] | ~20x |
| D3: 查看摘要 | ~10ms (500筆) | 讀取全部 + 加總計算 | ~150ms [estimate] | ~15x |

All values marked [estimate] since system doesn't exist yet. Flagged: "這些都是估計值，上線後應實際量測。"

**Observation**: Theory Limits table produced with [estimate] markers. Previous run: "Theory limits skipped." ✓

---

## Phase 6: Review and Cross-Check

1. Goal coverage: G1✓(D1), G2✓(D2), G3✓(D3), G4→管理類別 scored low but G4 doesn't need dominant op treatment
2. Constraint check: NFR1 (500ms) vs theory limits → D1 估計 50ms, D2 估計 100ms, D3 估計 150ms — 全部在 NFR1 範圍內 ✓
3. AP completeness: D1→AP1,AP2; D2→AP3; D3→AP3,AP4 ✓
4. No orphan operations: Op 4 (管理類別) 對應 G4，有目標依據 ✓

---

## User Confirmation
Skill presents draft dominant-ops.md → User confirms ✓

---

## Rubric Check

| Criterion | Status | Notes |
|-----------|--------|-------|
| D1: Each Dx has frequency, cost, failure impact | ✓ Pass | All three dimensions documented |
| D2: Write/Read identified as separate ops | ✓ Pass | D1=write, D2=read |
| D3: Frequency asymmetry noted | ✓ Pass | "每天多次" vs "每天1-2次" explicitly noted |
| D4: Failure asymmetry noted | ✓ Pass | "靜默失敗比顯示錯誤更嚴重" stated |
| D5: At least 1 AP per op with ID ⛔ | ✓ Pass | AP1-AP4 with Dx references |
| D6: Each Dx ID in output ⛔ | ✓ Pass | D1/D2/D3 all present |

**Red Flags**:
- Listed operations without assessing pressure? → No ✓
- Anti-patterns missing or no IDs? → No ✓
- Write and Read collapsed? → No ✓
- Data corruption risk not mentioned? → No (AP1, AP2 explicitly about write integrity) ✓

## Score: **PASS**

(Previous run: Partial — Theory limits skipped; operations inventory skipped; D3=管理類別 score=1)
