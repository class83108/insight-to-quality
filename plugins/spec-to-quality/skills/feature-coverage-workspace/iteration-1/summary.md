# ec:feature-coverage Eval Summary — Iteration 1

## Scores

| Eval | with_skill | without_skill | Delta |
|------|-----------|---------------|-------|
| eval-1 (order-refund) | **6/6** | 3/6 | +3 |
| eval-2 (skip-analysis) | **5/7** | 2/7 | +3 |
| eval-3 (notification) | **6/6** | 3/6 | +3 |

## Assertion-level Breakdown

| Assertion | with_skill (3 evals) | without_skill (3 evals) | Skill 增值 |
|-----------|---------------------|------------------------|-----------|
| A1: 表格結構 | 3/3 ✅ | 0/3 ❌ | **高** — skill 統一了輸出格式 |
| A2: 判斷附理由 | 3/3 ✅ | 2/3 ⚠️ | **低** — CLAUDE.md 的 6 類 checklist 已足夠引導 |
| A3: 等待確認 | 3/3 ✅ | 3/3 ✅ | **無** — CLAUDE.md 的「絕對禁止」規則夠強 |
| A4: 不寫 Gherkin | 3/3 ✅ | 3/3 ✅ | **無** — 兩者都不會跳步 |
| A5: 交叉比對 | 2/3 ⚠️ | 0/3 ❌ | **高** — 沒有 skill 完全不會做這步 |
| A6: ec:gherkin 銜接 | 2/3 ⚠️ | 0/3 ❌ | **高** — 只有 skill 知道下一步是 ec:gherkin |
| A7: 拒絕跳過 | 1/1 ✅ | 1/1 ✅ | **無** — CLAUDE.md 規則足夠 |

## Key Findings

### Skill 的核心價值（without_skill 做不到的）
1. **結構化表格格式** — Skill 統一了 6 類分析的表格呈現，baseline 用 headers+bullets 零散列出
2. **交叉比對步驟** — 只有 skill 會主動去找既有 .feature 檔做比對，baseline 完全不做
3. **Skill 銜接（ec:gherkin）** — 只有 skill 知道確認後要交棒給 gherkin skill

### CLAUDE.md 已經覆蓋的（Skill 不額外加值）
1. **等待確認** — 「絕對禁止」規則讓 baseline 也不敢跳步
2. **拒絕跳過** — CLAUDE.md 的流程規定足夠強
3. **基本覆蓋分析** — 6 類 checklist 定義在 CLAUDE.md，baseline 也能做

### Skill 的弱點（需改進）
1. **eval-2 with_skill 缺少交叉比對** — 拒絕跳過後直接跳到分析，忘了 Step 3
2. **eval-2 with_skill 缺少 gherkin handoff** — 可能因為「拒絕跳過」的邏輯搶了注意力
3. **「待討論」標記** — eval-2 with_skill 正確使用了，但 eval-1/3 全標適用沒機會展示

## Improvement Suggestions for SKILL.md

1. 在 Step 2 結尾加強提醒：「完成分析表後，接著做 Step 3 交叉比對」
2. 在 Step 4 結尾固定加一句：「確認後，接續使用 ec:gherkin 撰寫 .feature 檔」
3. 考慮在 Example 2（跳過分析）中明確示範完整 4 步流程，而非只示範拒絕
