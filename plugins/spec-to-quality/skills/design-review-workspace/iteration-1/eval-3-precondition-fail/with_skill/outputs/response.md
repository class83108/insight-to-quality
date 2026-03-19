根據 skill 指引，design review 的前置條件尚未滿足：

- 測試有 2 個 failing（需要全部通過）
- ruff 有 3 個 warning（需要 lint 通過）

建議先完成以下步驟，再進行 design review：

1. 修復 2 個 failing tests（`pytest` 全部綠燈）
2. 修復 ruff warnings（`ruff check . --fix && ruff format .`）
3. 確認 pyright 通過

完成後告訴我，就可以開始 review `services/notification.py`。
