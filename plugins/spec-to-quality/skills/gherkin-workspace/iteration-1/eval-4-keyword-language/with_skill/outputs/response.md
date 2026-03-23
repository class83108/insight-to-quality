權限問題，讓我先把完整的 `.feature` 檔內容給你確認：

---

```gherkin
Feature: 使用者檔案上傳
  作為 使用者
  我想要 上傳圖片檔案
  以便 將圖片儲存到雲端並取得可存取的 URL

  Rule: 只接受支援的圖片格式且大小在限制內

    Scenario: 成功上傳 JPG 圖片
      Given 使用者準備了一個有效的 JPG 圖片檔案
      And 檔案大小為 2MB
      When 使用者上傳該檔案
      Then 檔案應成功儲存到 S3
      And 系統應回傳該檔案的存取 URL

    Scenario: 成功上傳 PNG 圖片
      Given 使用者準備了一個有效的 PNG 圖片檔案
      And 檔案大小為 3MB
      When 使用者上傳該檔案
      Then 檔案應成功儲存到 S3
      And 系統應回傳該檔案的存取 URL

  Rule: 檔案格式與大小驗證必須在上傳前執行

    Scenario: 拒絕不支援的檔案格式
      Given 使用者準備了一個 GIF 圖片檔案
      When 使用者上傳該檔案
      Then 系統應拒絕上傳
      And 回傳錯誤訊息「不支援的檔案格式，僅接受 JPG 或 PNG」

    Scenario: 拒絕超過大小限制的檔案
      Given 使用者準備了一個有效的 JPG 圖片檔案
      And 檔案大小為 6MB
      When 使用者上傳該檔案
      Then 系統應拒絕上傳
      And 回傳錯誤訊息「檔案大小超過限制，上限為 5MB」

    Scenario: 拒絕空檔案
      Given 使用者準備了一個空的圖片檔案
      When 使用者上傳該檔案
      Then 系統應拒絕上傳
      And 回傳錯誤訊息「檔案不可為空」

  Rule: S3 上傳失敗時應提供明確錯誤

    Scenario: S3 服務暫時無法使用
      Given 使用者準備了一個有效的 JPG 圖片檔案
      And 檔案大小為 2MB
      And S3 服務目前無法連線
      When 使用者上傳該檔案
      Then 系統應回傳錯誤訊息「檔案儲存失敗，請稍後再試」
```

---

**結構說明：**

| Rule | 對應覆蓋類別 | Scenario 數量 |
|------|-------------|--------------|
| 只接受支援的圖片格式且大小在限制內 | Happy path | 2（JPG、PNG 各一） |
| 檔案格式與大小驗證必須在上傳前執行 | Input boundary | 3（格式錯誤、超過大小、空檔案） |
| S3 上傳失敗時應提供明確錯誤 | Error paths | 1（S3 連線失敗） |

---

**確認事項：**

1. 是否需要補充「邊界值 5MB」的 scenario（剛好 5MB 應該通過）？
2. 錯誤訊息的用語是否需要調整？
3. 還有其他 S3 失敗情境需要涵蓋嗎（例如權限錯誤、timeout）？

確認沒問題的話，請授權寫入權限，我會把檔案寫到 `tests/features/file_upload.feature`。
