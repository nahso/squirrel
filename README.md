【鼠鬚管】輸入法（Fork）
===

本倉庫是 [rime/squirrel](https://github.com/rime/squirrel) 的 fork，基於官方 **1.1.2** 版本，並額外包含一項針對終端全角標點輸入的修復。

完整的使用說明、安裝步驟與定製文檔，請參閱上游項目的 [README.md](https://github.com/rime/squirrel/blob/master/README.md)。

## 本 fork 的改動

### 問題

在 VSCode 內置終端、Alacritty、Neovide 等基於 **xterm.js** 的應用中，使用鼠鬚管輸入中文全角標點（如 `。`、`，`、`《`、`》` 等）時，字符無法正確送達應用。這是客戶端對 US 鍵盤佈局下中文標點處理不當所致；macOS 系統拼音輸入法則能正常工作。

相關討論：

- [rime/squirrel#827](https://github.com/rime/squirrel/issues/827)
- [fcitx/fcitx5-macos#223](https://github.com/fcitx/fcitx5-macos/issues/223) — 在 VSCode 內置終端和 Neovide 中無法輸入中文句號、書名號等全角標點

### 修復思路

本修復借鑒了 [fcitx/fcitx5-macos#348](https://github.com/fcitx/fcitx5-macos/pull/348) 的做法：在應用層尚未正確處理該問題之前，由輸入法臨時繞過。

1. **切換鍵盤佈局**：當 Rime 方案啟用中文標點（`ascii_punct` 為 `false`）且用戶配置為默認/US 佈局時，將客戶端鍵盤佈局覆寫為 `com.apple.keylayout.PinyinKeyboard`（與系統拼音相同）。這樣 `NSEvent` 送出的標點字符已是中文全角形式，終端等客戶端可以正確接收。

2. **反向映射 keysym**：佈局切換後，Rime 引擎收到的 keysym 也會變成中文標點，而 Rime 仍期望 ASCII keysym。因此在 `MacOSKeyCodes.swift` 中，當 `PinyinKeyboard` 佈局生效時，將標點鍵的 keysym 映射回對應的 ASCII 值，供 Rime 正常處理。

3. **動態跟隨方案狀態**：在 session 創建、狀態更新等時機調用 `overrideKeyboardLayoutIfNeeded()`，根據當前 `ascii_punct` 選項自動切換佈局。

### 已知限制

- 雙引號 `"` 等部分標點尚未覆蓋。這並非本 patch 的做法有問題——本修復與系統拼音採用相同的 `PinyinKeyboard` 佈局繞過方式；在 VSCode 內置終端等 xterm.js 環境中，**macOS 自帶拼音輸入法同樣無法正確輸入雙引號**，屬於客戶端與該佈局之間的既有局限。
- 非 ABC/US 佈局不受影響，仍按用戶配置使用

## 授權

與上游相同，授權條款：[GPL v3](https://www.gnu.org/licenses/gpl-3.0.en.html)
