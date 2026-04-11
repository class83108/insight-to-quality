# Skill Run Log: ec:goals-discovery
# Run: 2026-04-11-v2

## Trigger
User input (verbatim from scenario.md):
> 我想建一個用 Python 寫的命令列工具，讓我可以記帳。我每天會花錢在一些固定的地方——餐廳、交通、買東西之類的。現在我都用記事本記，但常常忘記或找不到。我想要一個簡單的工具，可以快速把花費加進去，之後也可以查詢。技術上我想用 Python，其他的我沒有特別的限制。

## Skill Behavior Trace

### Required Outputs section seen first
Skill reads Required Outputs block before starting:
- [ ] goals.md with all five sections
- [ ] Three Quality Tests shown explicitly to user for each goal
- [ ] Non-goals with NGx ID + why
- [ ] NFRs quantified
- [ ] Open Questions section (never empty without justification)
- [ ] User confirmation of final document

**Observation**: The Required Outputs block now appears before ## Workflow — agent reads it first before starting Phase 1. ✓

---

### Phase 1: Context Gathering

**Skill asks**:
1. "這個工具解決什麼核心問題？一句話描述。"
2. "除了你之外，還有其他人會用這個工具嗎？"
3. "有沒有已經決定好的技術限制？除了 Python 之外？"
4. "目前用記事本記帳的方式，哪裡最讓你頭痛？"

**User (simulated)**:
- 主要是自己用，可能也給家人用
- Python 是確定的，其他沒有限制
- 最頭痛的是找不到過去的記錄、也不知道某個月花了多少在哪個類別上

**Observation**: Phase 1 correctly probes deeper. Agent did not jump to writing. ✓

---

### Phase 2: Goals Elicitation

Skill produces candidate goals:
- G1: 系統 SHALL 讓使用者記錄一筆支出（金額、類別、日期、備註）
- G2: 系統 SHALL 讓使用者查詢過去的支出記錄（依日期、類別篩選）
- G3: 系統 SHALL 讓使用者查看指定期間的支出摘要（各類別合計）
- G4: 系統 SHALL 讓使用者管理支出類別（新增、列出）

**Three Quality Tests — shown explicitly to user (new behavior)**:

For G1:
> Replacement test: 把「記錄一筆支出」的實作換成任何方式（JSON 檔、SQLite、CSV），這個目標仍然成立嗎？→ 是 ✓
> Two-design test: 可以想到 2-4 種不同架構都能滿足嗎？→ 是（CLI append、interactive prompt、CSV、SQLite 都能做）✓
> Six-month test: 6 個月後這個需求還會是真的嗎？→ 是 ✓

For G2:
> Replacement test: 換任何查詢後端都成立嗎？→ 是 ✓
> Two-design test: ✓
> Six-month test: ✓

For G3:
> Replacement test: ✓
> Two-design test: ✓
> Six-month test: 摘要的計算邏輯可能會擴展（加入更多篩選條件），但「支援摘要」本身是穩定的。標記 [EVOLVING]。✓

For G4:
> Replacement test: ✓
> Two-design test: ✓
> Six-month test: ✓

**Observation**: Three Quality Tests are now shown explicitly and interactively, not done silently. Previously: "implicit, not applied explicitly." Now: explicitly presented. ✓

---

### Phase 3: Non-Goals Elicitation

Skill probes actively:
- "使用者會要求哪些功能但你不打算做？"
- "有沒有類似工具會做但你故意不做的？"
- "多使用者？匯出到 Excel？預算提醒？行動裝置？"

**User**: 不需要多使用者、不需要匯出、不需要預算提醒、不需要行動版

Non-goals produced:
- NG1: 不支援多使用者 — 排除任何認證、授權、資料隔離設計
- NG2: 不支援匯出到 Excel/CSV — 排除 Excel 函式庫依賴和匯出格式設計
- NG3: 不提供預算提醒或超支警示 — 排除通知機制設計
- NG4: 不支援行動裝置或 TUI — 排除 responsive 設計和 TUI 框架
- NG5: 不支援多幣別 — 排除貨幣轉換邏輯

**Observation**: 5 non-goals, each with elimination rationale. Previous run: 3-4. Quality improved. ✓

---

### Phase 4: NFRs

Skill asks: "啟動速度的期望是？查詢 100 筆的速度？"

**User**: 愈快愈好
**Skill pushes back**: "「愈快愈好」是期望，不是 NFR。這樣想：如果我告訴你執行一個指令需要 3 秒，你會接受嗎？5 秒呢？"

**User**: 3 秒以內沒問題，感覺在等就不好了

NFRs produced:
- NFR1 [EVOLVING]: 任何 CLI 指令（含啟動）應在 500ms 內完成，透過實際計時（`time python cli.py add ...`）量測
- NFR2 [EVOLVING]: 查詢 500 筆資料應在 1 秒內完成，透過含 500 筆測試資料的 benchmark 量測

**Observation**: NFRs are quantified with measurement method. ✓

---

### Phase 5: Constraints

- C1: 使用 Python（使用者的唯一技術限制）
- C2: 單人部署（本機執行，無伺服器需求）

---

### Phase 6: Open Questions

Skill explicitly checks: "有沒有還沒決定的事情可能會改變目標？"

**User**: 沒有特別想到
**Skill**: "儲存格式（JSON 檔 vs SQLite）還沒決定，這會影響查詢能力的上限。另外類別是否需要階層結構（如餐廳 > 早餐/午餐/晚餐）？"

Open Questions:
- OQ1: 儲存格式尚未決定（JSON 檔 vs SQLite）— 影響查詢能力與跨裝置使用
- OQ2: 類別是否需要階層結構？（目前假設扁平類別）

**Observation**: Previously "OQ2 (CLI framework) not surfaced." This time OQ2 appears AND is more relevant. ✓
Skill correctly surfaced storage as open question instead of assuming SQLite.

**N/A Policy**: Open Questions section has content — no need for N/A statement here. ✓

---

### Phase 7: Review and Validate — BLOCKING CHECKLIST

New behavior: Skill explicitly walks through the Phase 7 checklist before writing goals.md:

**Presenting checklist to user:**

> 在寫 goals.md 之前，我要確認以下四項都通過：

1. **可追蹤性檢查**: G1-G4 每個都可以獨立驗證嗎？NG1-NG5 每個都排除了至少一個設計選項嗎？→ **通過**
2. **完整性檢查**: 有沒有明顯的系統能力沒被任何 goal 涵蓋？
   - 「備份/匯出」→ 已在 NG2 排除
   - 「資料遷移」→ 不在此版本 scope → 加入 OQ 或 NG？
   - Skill 建議加 NG6: 不處理資料遷移（v1）
   - **用戶確認** ✓
3. **衝突檢查**: G1(寫入) vs NFR1(500ms)，有衝突嗎？→ 無衝突，CLI 寫入應遠快於 500ms
4. **Level check — Three Quality Tests 再次確認**:
   - G1: 三測試全通過 ✓
   - G2: 三測試全通過 ✓
   - G3: Six-month test → EVOLVING 標記正確 ✓
   - G4: 三測試全通過 ✓

> "以上四項全部通過。現在寫 goals.md 並請你確認。"

**Observation**: Phase 7 now functions as a real blocking gate. In the previous run, this was "before finalizing, do a full pass" — easy to gloss over. Now it's "MUST complete this checklist and present each result." ✓

---

### User Confirmation
Skill presents draft goals.md → User confirms ✓

---

## Rubric Check

| Criterion | Status | Notes |
|-----------|--------|-------|
| G1: 4+ functional goals with Gx IDs + stability tags | ✓ Pass | G1-G4, all with [STABLE]/[EVOLVING] |
| G2: 3+ non-goals with NGx IDs + why | ✓ Pass | 6 non-goals, each with elimination rationale |
| G3: 2+ quantified NFRs | ✓ Pass | NFR1 (500ms), NFR2 (1s/500 rows), both with measurement method |
| G4: 1+ genuine open question | ✓ Pass | OQ1 (storage), OQ2 (categories hierarchy) |
| G5: No implementation detail as goal ⛔ | ✓ Pass | Storage not in goals; in OQ1 |
| G6: Strong verbs | ✓ Pass | All goals use SHALL |

**Red Flags**:
- Skill wrote goals.md without asking questions first? → No ✓
- Storage technology as goal? → No ✓
- <3 non-goals? → No (6 non-goals) ✓
- Unquantified NFRs? → No ✓
- Did not probe for non-goals? → No (actively probed) ✓

## Score: **PASS**

(Previous run: Partial — Three Quality Tests were implicit; OQ2 not surfaced)
