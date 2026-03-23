好，我已經讀取了 spec.md 和 translation.feature。現在進入 **Phase 0: Verification Ledger**。

---

## Phase 0: Verification Ledger（Mock 邊界審查）

### Step 1: SHALL 清單

從 spec.md 提取的 5 個 SHALL 語句：

| # | SHALL 語句 |
|---|-----------|
| 1 | SHALL 使用 xai_sdk.Client 呼叫 Grok API 進行翻譯 |
| 2 | SHALL 從 response.usage 提取 input_tokens 和 output_tokens |
| 3 | SHALL 支援 batch 翻譯（一次傳入多個文字段落） |
| 4 | SHALL 在 API 回傳 429 時自動重試（最多 3 次） |
| 5 | SHALL 將翻譯結果存入 TranslationRecord model |

### Step 2 & 3: Mock 邊界規劃 + Checkpoint

| SHALL | 驗證方式 | Mock 邊界 | Checkpoint A | Checkpoint B |
|-------|---------|----------|--------------|--------------|
| **1** 使用 xai_sdk.Client | Unit test | Mock `xai_sdk.Client` 的 `chat.completions.create` | ✅ 遮掉的是「外部 API 呼叫」，不是自己的邏輯 | N/A |
| **2** 從 response.usage 提取 tokens | Unit test | 同上，Fake response 包含 `.usage.input_tokens`, `.usage.output_tokens` | ✅ 遮掉的是「外部回傳值」 | ✅ 自己解析 usage 的邏輯被測試覆蓋 |
| **3** 支援 batch 翻譯 | Unit test | 同上 | ✅ 遮掉的是「外部 API」 | ✅ 多段輸入 → 多筆輸出的對應關係需驗證 |
| **4** 429 自動重試 | Unit test | Mock Client，讓前 N 次呼叫回傳 429 exception，第 N+1 次成功 | ✅ 遮掉的是「外部錯誤」 | ✅ 重試計數邏輯被測試覆蓋 |
| **5** 存入 TranslationRecord | Unit test | 不需 mock 外部服務，直接驗證回傳的 model | ✅ 這是自己的邏輯，不需 mock | ✅ model 欄位完整性需驗證 |

### Step 4: Verification Ledger

```markdown
# Verification Ledger — 翻譯功能

## Unit Test 覆蓋

- SHALL 1（使用 xai_sdk.Client）
  → ✅ mock 邊界：mock `xai_sdk.Client.chat.completions.create`
  → 驗證：呼叫時傳入正確的 model、messages 參數

- SHALL 2（提取 token 數量）
  → ✅ mock 邊界：Fake response 含 `.usage.input_tokens`, `.usage.output_tokens`
  → 驗證：回傳結果包含正確的 token 數量

- SHALL 3（batch 翻譯）
  → ✅ mock 邊界：同 SHALL 1
  → 驗證：傳入 N 段文字，回傳 N 筆結果

- SHALL 4（429 重試）
  → ✅ mock 邊界：mock Client 依序回傳 429 → 429 → 成功
  → 驗證：最多重試 3 次，超過則拋出例外

- SHALL 5（存入 TranslationRecord）
  → ✅ 無需 mock
  → 驗證：TranslationRecord 包含 source_text, translated_text, input_tokens, output_tokens

## 需要整合測試

（無 — 所有 SHALL 皆可透過 unit test + Fake 驗證）

## 明確不測（附理由）

（無）
```

---

### Mock 實作細節

我計畫使用 **自定義 Fake** 而非 MagicMock，來模擬 `xai_sdk.Client` 的回應：

```python
# Fake 結構（概念）
class FakeUsage:
    input_tokens: int
    output_tokens: int

class FakeMessage:
    content: str

class FakeChoice:
    message: FakeMessage

class FakeResponse:
    choices: list[FakeChoice]
    usage: FakeUsage
```

對於 429 重試測試，會用一個 counter 來控制「前幾次失敗、之後成功」的行為。

---

**請確認這個 Verification Ledger 是否 OK，確認後我就開始寫測試（進入 Phase 1: Red）。**
