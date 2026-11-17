# Ingest 與 Lint 運作日誌

## Ingest: Context Is What You Need: The Maximum Effective Context Window for Real World Limits of LLMs (2025)
- **處理日期**：2026-11-17
- **Ingest 來源**：`raw/Context_Is_What_You_Need.pdf` (來源: Context_Is_What_You_Need.pdf)
- **異動與新增檔案清單**：
  - `wiki/index.md` (新增/更新知識庫首頁、閱讀路徑與全頁面索引)
  - `wiki/papers/context_is_what_you_need.md` (新增單篇深度筆記)
  - `wiki/concepts/mecw_vs_mcw.md` (新增最大有效上下文窗口概念頁)
  - `wiki/concepts/cascading_failures.md` (新增 Agent 鏈路級聯失敗概念頁)
  - `wiki/concepts/in_context_degradation.md` (新增任務複雜度注意力退化模式頁)
  - `wiki/syntheses/architecture_comparison.md` (建立多架構並陳對比表，建立傳統 Context Window 基準)
  - `wiki/log.md` (新增日誌紀錄)

### Lint 檢查結果
- [x] **連結檢查（Link Check）**：所有雙向連結語法皆正確對應到 `wiki/` 實體 `.md` 檔案，無任何斷鏈。
- [x] **孤島檢查（Island Check）**：所有 `wiki/` 頁面皆已登記於 `wiki/index.md`，且每一內容頁面皆引用 `raw/Context_Is_What_You_Need.pdf` 出處。
- [x] **出處檢查（Citation Check）**：逐段具備 `(來源: Context_Is_What_You_Need.pdf, <章節/頁碼>)` 精確標記；未出現於原文之細節標註 `[資料不足，待補充]`，符合零幻覺規範。
- [x] **並陳檢查（Balance Check）**：對比表已保留傳統 Context Window（Prompt Concatenation）之 Working Memory 特性，並預留後續外部記憶架構（Mem0、A-Mem、ZEP、MemMachine）之並陳欄位，未預設立場刪除或偏廢任何機制。
- **Lint 狀態**：全數通過 (PASSED)。
