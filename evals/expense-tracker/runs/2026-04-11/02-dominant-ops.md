# Skill Run: ec:dominant-ops

## Simulated Conversation

**Claude (following skill — reads goals.md first, then starts inventory):**
> 我讀了 goals.md。現在來分析壓力所在。先幫我走過一次完整的使用流程：你記帳時，從打開工具到記完，每一步是什麼？

**User:**
> 就是打指令，輸入金額、選分類、可能寫個備註，然後按 Enter。差不多這樣。

**Claude:**
> 一天大概記幾次？

**User:**
> 大概 3 到 10 次吧，吃飯、搭車、買東西都要記。

**Claude:**
> 如果「新增」這個動作失敗了——但你不知道它失敗——最壞的情況是什麼？

**User:**
> …就是那筆錢消失了，我完全不知道它沒被記進去。然後月底對帳的時候數字對不上，但我已經不記得那筆了。

**Claude:**
> 查詢呢？你說想看「這個月餐廳花了多少」。如果查詢結果是錯的——比如少算了幾筆——你會怎麼知道？

**User:**
> 其實不會知道，因為我沒有其他數字可以對照。我只能相信它。

**Claude:**
> 所以查詢如果靜默出錯，你不會察覺，但你會基於錯誤結果做決策（比如覺得這個月花不多，然後繼續花）。這個影響比「新增失敗」更難察覺。你同意嗎？

**User:**
> 對，確實。

**Claude:**
> 管理分類呢——大概多久設定一次？

**User:**
> 一開始可能設幾個，之後很少動。頂多偶爾新增一個。

**Claude:**
> 現在我要你在這三個操作裡強制排序：如果設計時只能把注意力放在一個，你覺得哪個最重要？

**User:**
> 新增吧，因為用最多。

**Claude:**
> 我覺得查詢其實更需要注意，因為它出錯你不會知道，但新增失敗通常你馬上看到。你怎麼看？

**User:**
> 說得有道理，那兩個都重要，但新增更頻繁。

**Claude (asks for anti-patterns from user experience):**
> 在用記事本記帳的過程中，你有沒有遇過「以為記了但其實沒記好」的情況？

**User:**
> 有，有時候複製貼上出錯，或者日期寫錯，後來就對不上了。

---

## Skill Observations

### ✅ Did well
- Skill correctly asked about frequency AND failure impact separately, not just listing operations
- The "if X fails silently, what's the worst case?" question is the skill's key value-add — it surfaced the query failure mode being worse than add failure mode
- Forced ranking was done: "如果只能把注意力放在一個" — pushed back on user's initial answer
- Asked user for their own anti-patterns from experience (the "記事本複製貼上出錯" example is exactly the kind of thing the skill should elicit)

### ⚠️ Gaps observed
- **Theory limits were not asked about.** The skill requires establishing theoretical lower bounds (best-case time, binding constraint). For a CLI tool writing to local disk, this is trivially fast, but the skill should still have asked "how fast could this theoretically run?" and noted the binding constraint (disk I/O, file parse time).
- **Operations inventory table was skipped.** The skill says to list ALL operations before selecting Top 3. In this simulation, the conversation jumped to "these three are the candidates" without a full inventory (e.g., "display summary" is a separate operation from "list expenses").
- **Anti-pattern IDs were assigned by Claude, not elicited from user.** The skill should ask the user to name the failure modes first, then formalize them. Here Claude led with the anti-pattern structure.
- **D3 (Manage Categories) is borderline:** criticality score of 1×1×1 = 1 is so low it arguably shouldn't be a dominant operation. The skill should have flagged this tension — either promote a real D3 candidate (e.g., "view summary" which is part of G4) or accept that there are only 2 dominant operations.
