# Skill Run: ec:goals-discovery

## Simulated Conversation

**Claude (following skill):**
> 在開始之前，我想先了解一些背景。
> 1. 用一句話描述：這個工具解決了你的什麼問題？
> 2. 你現在沒有這個工具時，怎麼記帳的？哪裡讓你最不滿意？
> 3. 有沒有什麼技術或環境上已經確定的事情？

**User:**
> 問題就是記事本太散了，找帳的時候要翻很久，有時候當下懶得記就忘了。現在用手機備忘錄記，Python 是確定的，其他沒特別想法。

**Claude:**
> 除了你提到的「新增」和「查詢」，你有沒有想過這個工具「不做什麼」？
> 比如說：要不要支援多人共用？要不要做預算提醒？要不要匯出 Excel？

**User:**
> 不需要，就我自己用，這些都太複雜了。不需要 Excel，手機備忘錄就夠簡單所以我才一直用，我只是想讓查詢更方便。

**Claude:**
> 你說「分類」——分類是你自己定義的，還是工具給你選？例如你可能有一些很個人的分類，像「診所」或「訂閱費用」？

**User:**
> 對，我希望可以自己定，不是固定的那種。

**Claude (challenges abstraction level):**
> 你說「查詢」——可以說更具體一點嗎？你最常需要查什麼？

**User:**
> 就是看這個月餐廳花了多少，或者上週交通花了多少。

**Claude:**
> 好，那我們把目標整理一下。貨幣方面——只記台幣還是可能有其他幣別？

**User:**
> 只有台幣，沒有出國消費的需求。

**Claude (probing for NFRs):**
> 性能方面，你希望這個工具有多快？比如說「打開就要馬上可以用」——你有沒有具體的感受？

**User:**
> 沒有特別量過，就是感覺要快，不能等。

**Claude (pushes for quantification):**
> 「感覺要快」很難設計，我們用一個具體數字：如果打指令到出現結果超過 1 秒，你會覺得慢嗎？

**User:**
> 1 秒可以接受，超過 2 秒就很奇怪。

**Claude:**
> 資料儲存的地方——你有特別偏好嗎？存成 JSON 檔案，或者用 SQLite 資料庫？

**User:**
> 我沒有特別想法，你覺得哪個好就哪個。

→ *Skill correctly defers to Open Questions rather than deciding.*

**Claude:**
> 最後：有沒有你不確定的假設，如果後來發現不對，整個方向可能要改？

**User:**
> 我假設查詢功能我自己用得到，如果後來發現我其實不查，這個工具就沒意義了。

---

## Skill Observations

### ✅ Did well
- Phase 1 context gathering was appropriate — asked about users, current workflow, constraints
- Correctly redirected "storage format" question to Open Questions rather than making the decision
- Actively probed for non-goals ("什麼不做") — user gave a positive-only description, skill correctly pushed
- Caught the NFR quantification gap ("感覺要快" → pushed for numbers)
- Asked about categories being user-defined vs fixed — this was a hidden truth that needed eliciting

### ⚠️ Gaps observed
- **Did not apply the Three Quality Tests explicitly** to any goal before finalizing. The skill says to apply Replacement Test, Two-Design Test, and Six-Month Test to each goal, but in practice these were implicit at best.
- **Open Questions section was thin**: OQ1 (storage) was well-captured; but OQ2 (CLI framework choice) was not raised as a question even though it's a real constraint tradeoff.
- **Non-goals stopped at 5**: The skill says "non-goals often outnumber goals in a well-defined system" and "if user has 10 goals and 2 non-goals, push harder." Here we had 4 goals and 5 non-goals — reasonable but the skill could have probed more.
- **No explicit Stability tagging discussion**: The skill asks the agent to tag STABLE vs EVOLVING, but this happened without explaining to the user what the tags mean. A real user might be confused by [STABLE] appearing in the document without context.
