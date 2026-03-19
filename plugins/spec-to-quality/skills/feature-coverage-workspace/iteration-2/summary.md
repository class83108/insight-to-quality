# ec:feature-coverage Eval Summary — Iteration 2 (SKILL.md 修改後)

## Scores: Iteration 1 vs 2

| Eval | with_skill (iter1 → iter2) | without_skill (iter1 → iter2) |
|------|---------------------------|-------------------------------|
| eval-1 (order-refund) | 6/6 → **6/6** (穩定) | 3/6 → **3/6** (穩定) |
| eval-2 (skip-analysis) | 5/7 → **5/5** (設計改善) | 2/7 → **2/7** (穩定) |
| eval-3 (notification) | 6/6 → **6/6** (穩定) | 3/6 → **3/6** (穩定) |

## Eval-2 的行為變化（SKILL.md 修改的直接效果）

### Iteration 1（舊 SKILL.md: 「拒絕跳過，強制分析」）
- 拒絕跳過 ✅
- 直接開始分析 ✅
- **但忘了 Step 3 和 ec:gherkin handoff** ❌❌

### Iteration 2（新 SKILL.md: 「提醒價值，尊重選擇」）
- 提供兩個選項 ✅
- 解釋為什麼分析有價值 ✅
- **選項 1 明確提及 ec:gherkin** ✅ ← iter1 的 A6 fail 已修復！
- 等使用者決定 ✅

### 結論
新設計更合理：
1. 不再強制分析（使用者堅持跳過就交 ec:gherkin）
2. ec:gherkin handoff 自然出現在選項裡（不需要走完 Step 1-4 才提及）
3. 價值提醒讓使用者有足夠資訊做決定

## 穩定性觀察

| 比較項目 | 結論 |
|----------|------|
| with_skill eval-1/3 | 兩次都 6/6，行為穩定 |
| without_skill 全部 | 兩次結果完全一致（3/6, 2/7, 3/6），baseline 極穩定 |
| Delta (with - without) | 穩定 +3 分，skill 效果可重複驗證 |

## SKILL.md 修改項與效果

| 修改 | 效果 |
|------|------|
| Step 2 結尾加「接著做 Step 3」 | eval-1/3 原本就有 Step 3（iter1 就 pass），無法單獨驗證此修改的增量效果 |
| Step 4 結尾固定 handoff 句 | eval-1/3 兩次都有 handoff（iter1 就 pass），同上 |
| Example 2 改為「提醒 + 尊重決定」 | **直接驗證成功** — eval-2 行為從「強制分析但遺漏步驟」變為「靈活選項且提及 ec:gherkin」 |
