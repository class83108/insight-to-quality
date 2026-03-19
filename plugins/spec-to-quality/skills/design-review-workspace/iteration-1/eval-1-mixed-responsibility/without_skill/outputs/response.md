<command-name>/design-review</command-name>

# Design Review — `OrderRefundService.process_refund`

測試綠燈、lint 通過，進入設計品質 review。以引導提問為主，不直接改 code。

---

## 1. 單一職責 (SRP)

`process_refund` 目前在做幾件事：

1. 查詢並驗證訂單狀態
2. 建立 RefundRecord
3. 呼叫外部付款 API
4. 更新退款與訂單狀態
5. 寄通知信
6. 寫檔案 log

**問：** 如果「通知方式」從 email 改成 LINE Notify，或「log 目的地」從 file 改成 CloudWatch，你需要修改哪個類別？這種修改對 `OrderRefundService` 來說合理嗎？

---

## 2. 依賴方向與可測試性

目前 `requests.post`、`send_mail`、`open(...)` 都是在函式內硬呼叫。

**問：** 如果你想對「付款 API 回傳 500」這個場景寫單元測試，你現在需要 mock 幾個地方？這個測試寫起來會不會讓你覺得「測試在對抗程式碼結構」？

---

## 3. `import requests` 在函式內

**問：** 你認為這個 import 放在函式內的動機是什麼？這個動機成立嗎？它帶來什麼隱性成本（例如：靜態分析、dependency 追蹤）？

---

## 4. 交易一致性 (Transaction)

目前流程是：

```
建立 RefundRecord (pending)
→ 呼叫外部 API (成功)
→ refund.save() + order.save()
→ send_mail()
```

**問：** 如果 `refund.save()` 成功、`order.save()` 拋出例外，系統狀態會是什麼？`order.status` 還是 `'completed'`，但 `refund.status` 已經是 `'approved'`，這兩者不一致的情況，你的系統有辦法偵測或修復嗎？

---

## 5. 外部 API 失敗的語意

目前只檢查 `status_code == 200`，其他都當作失敗。

**問：** `status_code 201`、`202`（Accepted，非同步退款）、`429`（rate limit，可重試）、`500`（伺服器錯誤，可重試）這些情況，現在都會走同一條 `raise RuntimeError` 路徑。這樣的錯誤分類對呼叫端有足夠的資訊做決策嗎？

---

## 6. `requested_by` 的型別語意

`requested_by` 直接被當成 `requested_by_id` 傳給 ORM，但參數名稱沒有傳達「這是 ID」的語意。

**問：** 如果未來某個呼叫端傳的是 User 物件而不是 ID，這個 bug 會在哪個階段被發現？型別標註或 Pydantic model 能在這裡幫什麼忙？

---

## 整體觀察（不評分，只描述）

這段程式碼的**邏輯本身是對的**，但它把「協調多個副作用」的責任全部壓在一個函式裡。目前可以運作，但每次需求變更（通知方式、log 格式、API 版本）都會觸碰同一個函式，修改風險會隨功能增長累積。

以上問題沒有標準答案，是用來幫你思考下一步重構的切入點。你對哪個問題有想法，或有哪個地方想討論？
