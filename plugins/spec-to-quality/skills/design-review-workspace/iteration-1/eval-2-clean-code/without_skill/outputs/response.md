## Design Review

整體架構清晰，Protocol-first 設計正確，`MixingExecutor` 的職責單純。以下幾個值得思考的問題：

---

### 1. `MixingResult` 的歸屬

`MixingResult` 定義在 `engines/mixing.py`，但它其實是 `MixingEngine` Protocol 的回傳型別。

- `MixingOutput`（schema）和 `MixingResult`（engine result）目前是兩個幾乎相同的 dataclass，轉換發生在 `MixingExecutor` 裡。
- **問題**：`MixingEngine` Protocol 定義在 `pipeline/protocols.py`，但它的 return type 是 `MixingResult`（定義在具體 engine 實作裡）。這樣所有想實作 `MixingEngine` Protocol 的新 engine，都必須依賴 `engines/mixing.py` 的型別嗎？

> `MixingResult` 應該定義在 `pipeline/` 層（protocols 或 schemas），還是你刻意讓它住在 engine 層？這個決策的理由是什麼？

---

### 2. `_parse_output` 的缺口

```python
def _parse_output(self, stderr: str):
    # parse ffmpeg output for duration and peak loudness
    ...
```

`_execute` 呼叫了 `_parse_output`，但 `_parse_output` 回傳的是 `...`（Ellipsis）。`_execute` 用 `result.duration` 和 `result.peak_db` 讀取它的欄位。

- 這段在測試裡是怎麼被覆蓋的？是 mock 掉 `_execute`，還是 `_parse_output`？
- 如果 `_parse_output` 本身沒有測試，日後 ffmpeg stderr 格式改了，你會在哪裡感覺到？

> `_parse_output` 的 return type 是什麼？它應該有個明確的 dataclass，還是你認為 `MixingResult` 可以直接從這裡回傳？

---

### 3. `_build_filter_graph` 的正確性假設

```python
return ';'.join(delays) + f';[0]{mix_inputs}amix=inputs={len(segments)+1}'
```

如果 `segments` 是空 list，這段會產生：

```
;[0]amix=inputs=1
```

開頭多一個 `;`，且 `amix=inputs=1` 可能不是有意義的行為。

- 這個情境有被測試到嗎？
- 設計上，空 segments 應該是 precondition violation（應 raise），還是應該 fallback 到直接輸出 background audio？

---

### 4. `Path` 轉換邊界

`MixingExecutor.execute` 把 `str` 轉成 `Path`，`FFmpegMixingEngine.mix` 再把 `Path` 轉回 `str`（`str(bg)`）。

- 這個來回轉換的職責分配是刻意的嗎？
- 如果 Protocol 的 signature 用 `Path`，呼叫端（executor）就要負責 `str -> Path`；如果用 `str`，engine 內部處理。你目前的選擇合理，但值得確認這是有意識的決定，而不是意外對稱。

---

整體沒有明顯的設計缺陷，主要值得確認的是 **`MixingResult` 的層級歸屬** 和 **`_parse_output` 的可測性**。
