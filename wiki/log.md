# Ingest 與 Lint 運作日誌

## Ingest: Mem0: Building Production-Ready AI Agents with Scalable Long-Term Memory (2025)
- **處理日期**：2026-11-17
- **Ingest 來源**：`raw/Mem0.pdf` (來源: Mem0.pdf)
- **異動與新增檔案清單**：
  - `wiki/papers/mem0.md` (新增單篇文獻深度筆記)
  - `wiki/concepts/dynamic_memory_operations.md` (新增 ADD/UPDATE/DELETE/NOOP 四態生命週期操作概念頁)
  - `wiki/concepts/graph_memory_representations.md` (新增有向標記圖、時序失效與雙路檢索概念頁)
  - `wiki/concepts/memory_latency_cost_tradeoffs.md` (新增記憶系統延遲、成本與準確率權衡概念頁)
  - `wiki/syntheses/architecture_comparison.md` (更新多架構並陳對比表，填入 Mem0 / Mem0^g 規格與多方觀點並陳)
  - `wiki/index.md` (更新首頁架構導圖、頁面清單與原始文獻庫)
  - `wiki/log.md` (追加日誌記錄)

### Lint 檢查結果
- [x] **連結檢查（Link Check）**：所有雙向連結語法皆正確對應到 `wiki/` 實體 `.md` 檔案，無任何斷鏈。
- [x] **孤島檢查（Island Check）**：所有 `wiki/` 頁面皆已登記於 `wiki/index.md`，且每一內容頁面皆引用 `raw/` 下之出處。
- [x] **出處檢查（Citation Check）**：逐段具備 `(來源: Mem0.pdf, <章節/頁碼>)` 或 `(來源: Context_Is_What_You_Need.pdf, <章節/頁碼>)` 精確標記；未出現於原文之細節標註 `[資料不足，待補充]`，符合零幻覺原則。
- [x] **並陳檢查（Balance Check）**：對比表與並陳章節完整保留了 Full-Context 之高準確率觀點、Mem0 密集記憶之低延遲與多跳優勢、Mem0^g 在時序推理之特長，以及 Zep 在開放領域與快取機制之差異，未刪除或偏廢任何一方觀點。
- **Lint 狀態**：全數通過 (PASSED)。

---

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
