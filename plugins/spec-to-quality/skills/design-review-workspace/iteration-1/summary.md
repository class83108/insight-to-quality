# ec:design-review Eval Summary — Iteration 1

## Scores

| Eval | with_skill | without_skill | Delta |
|------|-----------|---------------|-------|
| eval-1 (mixed responsibility) | **5/5** | 2/5 | +3 |
| eval-2 (clean code) | **5/5** | 1/5 | +4 |
| eval-3 (precondition fail) | **1/1** | 1/1 | 0 |

## Assertion-level Breakdown

| Assertion | with (3 evals) | without (3 evals) | Skill 增值 |
|-----------|---------------|-------------------|-----------|
| A1: 6 維度表格 | 2/2 ✅ | 0/2 ❌ | **高** — 只有 skill 統一表格輸出 |
| A2: 提問式回饋 | 2/2 ✅ | 2/2 ✅ | **無** — baseline 也會用提問方式！ |
| A3: 設計原則說明 | 2/2 ✅ | 0/2 ❌ | **高** — 結構化「為什麼值得思考」是 skill 獨有 |
| A4: [維度] 格式 | 2/2 ✅ | 0/2 ❌ | **高** — 統一 review 格式 |
| A5: 不吹毛求疵 | 1/1 ✅ | 0/1 ❌ | **中** — with_skill 正確說「程度不重」，without_skill 更深入但偏 nitpicky |
| A6: 前置條件 | 3/3 ✅ | 3/3 ✅ | **無** — CLAUDE.md TDD 流程已覆蓋 |

## Key Findings

### Skill 的核心價值
1. **結構化 review 格式** — [維度] 檔案:行號 + 「目前/考慮/為什麼」三段式
2. **教育性設計原則** — 「為什麼值得思考」是 linter 和一般 review 不會做的
3. **6 維度總結表格** — 讓使用者一眼看到全貌
4. **calibration（校準）** — clean code 時正確評估為「程度不重」，不硬找問題

### Baseline 意外好的地方
1. **提問方式** — without_skill 也用了提問式 review！Claude 似乎天然傾向這樣做
2. **前置條件** — CLAUDE.md 的 TDD 流程讓 baseline 也能正確拒絕

### 觀察
- Design review 的 delta (+3/+4) 比 feature-coverage (+3) 更大
- 原因：design review 有更多「格式規範」（6 維度、[維度] 格式、三段式），skill 的結構化引導效果更顯著
- Baseline 在「提問 vs 指令」這個維度不需要 skill 幫助，但在「校準嚴重程度」上 skill 有優勢

## 無需修改 SKILL.md
本次 eval 未發現需要修改的問題：
- eval-1/2 with_skill 都 5/5
- eval-3 正確拒絕
- 不存在 feature-coverage 那樣的「異常路徑遺漏步驟」問題
