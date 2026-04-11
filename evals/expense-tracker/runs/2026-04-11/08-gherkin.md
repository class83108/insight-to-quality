# Skill Run: ec:gherkin

## Simulated Conversation

**Claude:**
> 根據確認的覆蓋率分析，我來寫 `add_expense.feature`。
> 關鍵字使用英文（Feature/Scenario/Given/When/Then），步驟描述使用繁體中文。

**User:**
> 好。

---

## Output: `features/add_expense.feature`

```gherkin
Feature: 新增支出

  作為個人記帳使用者
  我希望可以快速新增一筆支出紀錄
  以便追蹤我的消費

  Background:
    Given 系統已初始化，資料儲存可正常寫入
    And 使用者已建立「餐廳」分類

  # --- Happy Path ---

  Scenario: 成功新增一筆支出，日期預設為今天
    When 使用者新增一筆 120 元的「餐廳」支出，描述為「午餐」
    Then 應回傳成功訊息，包含金額 120、分類「餐廳」、描述「午餐」、日期為今天
    And 該紀錄可透過查詢取得

  Scenario: 成功新增一筆支出，指定過去日期
    When 使用者新增一筆 250 元的「交通」支出，日期為「2026-04-10」
    Then 應回傳成功訊息，日期為「2026-04-10」
    And 該紀錄可透過查詢取得

  # --- Error / Failure Paths ---

  Scenario: 分類不存在時拒絕新增（AP2）
    When 使用者新增一筆 100 元的「不存在分類」支出
    Then 應回傳錯誤：分類「不存在分類」不存在
    And 不應有任何新紀錄被儲存

  Scenario: 金額為負數時拒絕新增
    When 使用者新增一筆 -50 元的「餐廳」支出
    Then 應回傳錯誤：金額必須大於零
    And 不應有任何新紀錄被儲存

  Scenario: Storage 寫入失敗時 fail fast（AP1）
    Given 資料儲存目前無法寫入
    When 使用者新增一筆 100 元的「餐廳」支出
    Then 應回傳儲存錯誤
    And 不應有任何部分資料被寫入

  # --- Boundary & Edge Cases ---

  Scenario: 金額為零時拒絕新增
    When 使用者新增一筆 0 元的「餐廳」支出
    Then 應回傳錯誤：金額必須大於零

  Scenario: 描述為空時仍可成功新增
    When 使用者新增一筆 80 元的「餐廳」支出，不填描述
    Then 應回傳成功訊息，描述欄位為空

  # --- State Mutation ---

  Scenario: 新增後紀錄可被查詢取得
    When 使用者新增一筆 150 元的「餐廳」支出
    Then 系統的支出列表中應包含這筆紀錄

  Scenario: 相同內容執行兩次，產生兩筆獨立紀錄（重複新增為預期行為）
    When 使用者新增一筆 120 元的「餐廳」支出，描述為「午餐」
    And 使用者再次新增一筆 120 元的「餐廳」支出，描述為「午餐」
    Then 系統的支出列表中應有兩筆相同內容的紀錄

  # --- Output Contract ---

  Scenario: 成功新增時，回傳的紀錄格式正確
    When 使用者新增一筆 120 元的「餐廳」支出，描述為「午餐」
    Then 回傳的紀錄應包含：金額（數字）、分類名稱（字串）、描述（字串）、日期（YYYY-MM-DD 格式）
```

---

## Skill Observations

### ✅ Did well
- All keywords in English (Feature, Background, Scenario, Given, When, Then, And)
- All step content in Traditional Chinese
- Scenarios map to the confirmed coverage categories
- AP1 and AP2 references appear in scenario names/comments
- Steps describe behavior, not implementation ("應回傳成功訊息" not "呼叫 add_expense() 函式")
- "重複新增為預期行為" scenario correctly captures the user's explicit decision from Step 5 of coverage

### ⚠️ Gaps observed
- **Rule/Feature structure not used.** The skill references Feature/Rule/Scenario hierarchy but this .feature file only uses Feature/Scenario. For this size, Rule blocks aren't needed, but the skill doesn't explain when to use Rule vs not — it just says the hierarchy is available.
- **Background section uses a non-standard setup.** "系統已初始化，資料儲存可正常寫入" is a broad Given that hides setup complexity. The skill doesn't flag this — it should note that Background steps that require complex setup tend to make tests brittle.
- **The "Output Contract" scenario duplicates the Happy Path scenario** in content. The skill should catch this: the output contract check should test the *shape* of the output separately from the happy path (e.g., using Scenario Outline or a distinct check). In practice, these two scenarios will produce near-identical test code.
- **Category 3 (Boundary): "極大值" was dropped** from the analysis → gherkin. The skill did not flag that a confirmed "Applicable" scenario direction was missing from the final .feature file.
