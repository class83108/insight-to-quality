## Design Review

### 發現的問題

**[依賴方向] engines/mixing.py:42 — `_execute` 內部 lazy import `subprocess`**

> 目前的寫法：`subprocess` 在 method body 裡才 import，而不是在 module 頂層。
>
> 考慮的方向：這個 lazy import 是有意為之（例如為了加速 import 或避免某些環境問題），還是只是沒有注意到？如果是後者，放到頂層是不是更符合「閱讀 import 就能知道這個模組的外部依賴」的原則？
>
> 為什麼值得思考：module-level import 讓依賴可見性更高——任何人打開檔案，掃一眼頂部就知道這個模組依賴什麼。Lazy import 通常是有明確理由才用（circular import、optional heavy dep），沒有理由時反而增加閱讀負擔。

---

**[命名語意] engines/mixing.py:36 — `_build_command` 的參數 `bg`, `fg` 過於縮寫**

> 目前的寫法：`bg` 代表 background audio path，`fg` 代表 filter graph string，在 function signature 裡用縮寫。
>
> 考慮的方向：`fg` 在這個脈絡裡讀者會不會直覺聯想到 filter graph？還是需要多花一秒才能解碼？如果改成 `filter_graph`，對閱讀幫助大嗎？
>
> 為什麼值得思考：縮寫節省打字，但閱讀時的認知成本由未來所有讀者共同承擔。`bg` 算常見縮寫還行，但 `fg` 作為 "filter graph" 並不是業界慣用縮寫，可能造成誤解。

---

**[可測試性] engines/mixing.py:42-44 — `_execute` 同時負責呼叫外部 process 和 parse output**

> 目前的寫法：`_execute` 跑 `subprocess.run`，拿到 `stderr` 後直接呼叫 `_parse_output`——兩件事在同一個 method 裡串在一起。
>
> 考慮的方向：如果你只想測試 `_parse_output` 的邏輯（不實際跑 ffmpeg），你需要怎麼做？現在的結構下，`_execute` 可以直接被測試嗎？
>
> 為什麼值得思考：`_execute` 把「執行外部 command」和「parse 結果」耦合在一起，導致測試 parse 邏輯時必須 mock `subprocess.run`。如果 `_parse_output` 只接受 raw stderr string，它本身已經是純函式可以單獨測試——但 `_execute` 的存在方式讓這件事變得不直覺。這不一定要改，但值得想想測試策略。

---

### 總結

| 維度 | 狀態 | 備註 |
|------|------|------|
| 職責分離 | OK | `FFmpegMixingEngine` 的子職責拆分清楚；`MixingExecutor` 只做資料轉換 |
| 依賴方向 | 有發現 | `subprocess` lazy import 位置可討論 |
| 命名語意 | 有發現 | `_build_command` 的 `fg` 縮寫不直覺 |
| 錯誤處理 | OK | `check=True` 讓 subprocess 失敗時直接拋出，fail fast 策略合理 |
| 可測試性 | 有發現 | `_execute` 把 process 執行與 parse 耦合，測試 parse 邏輯時有間接成本 |
| 一致性 | OK | Protocol-first 設計、Pydantic schema 傳遞、依賴注入——與專案慣例一致 |

整體設計乾淨，架構方向正確。三個發現都是程度不重的問題，優先順序依序是：可測試性 > 命名語意 > lazy import。
