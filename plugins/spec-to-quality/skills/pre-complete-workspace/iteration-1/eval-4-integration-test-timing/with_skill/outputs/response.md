我注意到這個目錄**不是 git repo**，也**沒有可執行的 Python 專案結構**（沒有 pyproject.toml）。在這種情況下，我無法執行完整的 pre-complete checklist。

讓我向你說明目前的狀況：

---

## ⚠️ Pre-Complete Verification 無法執行

目前目錄 `/tmp/eval-workspace` 缺少以下必要元件：

| 項目 | 狀態 | 問題 |
|------|------|------|
| Git repo | ❌ 缺失 | 無 `.git` 目錄，無法執行 `git diff`/`git status` |
| pyproject.toml | ❌ 缺失 | 無法執行 `uv run pytest`、`ruff`、`pyright` |
| Python 原始碼 | ❓ 未知 | 沒有可測試的 code |

---

## 關於整合測試缺口

你提到這是 **conversation 模組的最後一個 spec**，而且 verification.md 有 **2 個等級 2 項目**：
- WebRTC offer/answer 交換
- GeminiHandler Live API session

根據整合測試時機規則，**這是應該主動建議補寫整合測試的時機**，因為：
1. ✅ Ledger 中有等級 2 項目
2. ✅ 這是模組的最後一個 spec

**建議**：在 commit 前，至少補寫以下整合測試之一：
1. **WebRTC signaling smoke test** — 驗證 offer/answer 能正確交換
2. **FakeGeminiSession contract test** — 驗證 Live API session 建立的資料流

---

## 關於 Delta Spec 同步

你提到 openspec 的 change 目錄裡 `specs/` 有修改過的 spec.md。這需要同步到 `openspec/specs/` 主規格。

---

## 下一步

請問：
1. 這個專案的**實際路徑**在哪裡？（我需要切到正確的目錄執行驗證）
2. 整合測試你想**現在補**還是**先記錄到 issue 追蹤**？
