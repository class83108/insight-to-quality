## Design Review — `services/order_service.py`

前置條件確認：測試綠燈，ruff + pyright 通過。

---

### 發現的問題

**[職責分離] order_service.py — `process_refund` 做了五件不同的事**

> 目前的寫法：同一個 function 裡包含 DB 查詢與寫入、外部 API 呼叫、寄送 email、寫 log 檔案，共四種不同的 side effect，全部集中在一個 method 裡。
>
> 考慮的方向：如果退款 API 的呼叫方式改變，你需要動 `process_refund`；如果通知客戶的邏輯改變，你也要動同一個 function——你覺得這樣合理嗎？把「協調流程」和「執行具體動作」分開，哪些部分會變得更容易替換？
>
> 為什麼值得思考：Single Responsibility 的實用意義不是「每個 class 只有一個 method」，而是「改動一個需求，只需要動一個地方」。這個 function 有五個改動的理由，代表它承擔了五個責任。

---

**[依賴方向] order_service.py:11 — `import requests` 在 function 內部**

> 目前的寫法：`import requests` 放在 function 體裡，直接依賴具體的 HTTP library。
>
> 考慮的方向：如果之後想換成 `httpx`，或是想在測試裡替換掉外部 HTTP 呼叫，你需要怎麼做？把 payment gateway 的呼叫抽成一個 Protocol 或獨立的 client class，對這件事有什麼幫助？
>
> 為什麼值得思考：內層業務邏輯直接依賴外層 infrastructure（requests library + 具體 URL），違反依賴反轉原則。import 放在 function 內部通常是個訊號——這個依賴可能不應該在這裡。

---

**[錯誤處理] order_service.py — 沒有 transaction，部分失敗會留下不一致的狀態**

> 目前的寫法：`RefundRecord` 先被 `create`，API 呼叫再發出。如果 API 呼叫失敗，refund record 已經存在但狀態是 `failed`；如果 `refund.save()` 或 `order.save()` 失敗，外部退款可能已經成功但 DB 沒有更新。
>
> 考慮的方向：在這個流程裡，哪些操作應該是原子性的？哪些 side effect（寄信、寫 log）本來就不能被 transaction 保護？你怎麼決定邊界在哪裡？
>
> 為什麼值得思考：DB 操作和外部 API 呼叫同時存在時，「部分成功」是最難 debug 的狀態。不一定要完美解決，但需要明確決定這個 function 在 failure 時保證什麼、不保證什麼。

---

**[可測試性] order_service.py — 四個隱藏依賴讓單元測試幾乎不可能**

> 目前的寫法：`Order.objects.get`（DB）、`requests.post`（外部 HTTP）、`send_mail`（SMTP）、`open('/var/log/refunds.log')`（檔案系統）都是直接呼叫，沒有注入點。
>
> 考慮的方向：如果你要為「退款金額正確傳給 API」這個行為補一個測試，你需要 mock 幾個東西？這個數字讓你覺得測試是在驗證邏輯，還是在和 mock 搏鬥？
>
> 為什麼值得思考：需要 mock 四個以上東西的 function，通常是職責過多的訊號。可測試性差往往是設計問題的結果，不是原因。

---

### 總結

| 維度 | 狀態 | 備註 |
|------|------|------|
| 職責分離 | 有發現 | 一個 function 承擔 5 個責任 |
| 依賴方向 | 有發現 | 直接依賴 requests + 寫死 URL + import 在 function 內 |
| 命名語意 | OK | `process_refund` 名稱尚可接受，雖然略為模糊 |
| 錯誤處理 | 有發現 | 無 transaction，存在部分失敗的不一致風險 |
| 可測試性 | 有發現 | 4 個隱藏依賴，單元測試需要大量 mock |
| 一致性 | 無法判斷 | 需要看其他 service 的實作方式 |

整體來說，這個 class 目前是個「上帝方法」——它知道太多事情，也做太多事情。最核心的問題是職責分離，解決它之後，依賴方向和可測試性的問題通常會自然跟著改善。
