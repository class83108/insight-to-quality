# Skill Run: ec:system-map

## Simulated Conversation

**Claude (following skill — reads goals.md and dominant-ops.md):**
> 讀完了 goals.md 和 dominant-ops.md。我們來建立 SYSTEM_MAP。
> 先問你：如果一個新開發者加入這個小專案，你希望他在 5 分鐘內理解哪些「區塊」？

**User:**
> 就是怎麼輸入指令、怎麼處理邏輯、怎麼存資料吧。

**Claude:**
> 好，那我看到三個自然的元件：CLI（指令解析）、Domain（業務邏輯）、Storage（資料層）。
> 有一個問題：CLI 和 Storage 中間，你覺得「指令驗證」要在哪裡做？比如「金額不能是負數」這個規則——是 CLI 負責，還是 Domain？

**User:**
> 應該 Domain 吧？CLI 只是接指令。

**Claude:**
> 正確方向。那我們有清楚的兩條邊界：
> - CLI ↔ Domain（CLI 傳請求，Domain 傳結果）
> - Domain ↔ Storage（Domain 發讀寫，Storage 執行）
>
> 如果我改了 Storage 的實作（例如從 JSON 換成 SQLite），Domain 應該不需要改動——你同意嗎？

**User:**
> 對，這樣比較彈性。

**Claude:**
> 好。我們把這兩條邊界的「合約」先留空，在 ec:align-internals 再填細節。

---

## Skill Observations

### ✅ Did well
- Correctly asked "what buckets does a newcomer need?" rather than listing components itself
- The boundary discipline question ("CLI 和 Storage 中間，驗證在哪？") is the skill's key value — it got the user to explicitly decide that validation belongs in Domain
- Correctly deferred contract details to ec:align-internals (good handoff discipline)
- Boundary test was implicitly applied: "如果改了 Storage 的實作，Domain 不需要改" = Independent Change test

### ⚠️ Gaps observed
- **Mermaid diagram was not explicitly prompted for.** The skill requires a component diagram. In this simulation, it happened implicitly, but the skill should have a clearer instruction to always produce the diagram.
- **Change Protocol was not discussed with user.** The four change types (goal change, contract/boundary change, internal change, new component) should have been presented and confirmed. In this simulation, the skill jumped to the output document without walking through the protocol.
- **"Newcomer test" and "Change simulation" validation steps were skipped.** The skill has a Phase 7 with three validation exercises — none were done.
- **Current State section was thin.** The skill says to track in-flight work and known gaps. Since this is a new project, the section would be mostly empty, but the skill should still ask "what do you think is missing from your current design?" to populate Gaps.
