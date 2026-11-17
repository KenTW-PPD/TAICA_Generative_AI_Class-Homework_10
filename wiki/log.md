# Ingest 與 Lint 運作日誌

## Release: Interactive Educational Presentation (slides_html/index.html)
- **處理日期**：2026-11-17
- **生成檔案**：`slides_html/index.html` (互動式教學型 HTML 簡報)
- **覆蓋文獻**：5 篇核心文獻全數納入（Context 瓶頸、Mem0、A-Mem、ZEP、MemMachine）
- **頁面規格**：共 13 頁 16:9 比例深色學術風格簡報，內建鍵盤導航（← / → / 空格）、大綱抽屜（O / Esc）、進度條與 KaTeX 數學排版渲染。
- **出處標記**：每頁底部具備對應 `wiki/` 來源檔案精確標註，100% 源自知識庫。

### Lint 與整合檢查結果
- [x] **結構檢查**：單一 HTML 檔案包含完整 Tailwind CSS CDN、KaTeX CDN 與 Vanilla JS 控制邏輯，無任何標籤錯誤或未閉合語法。
- [x] **出處檢查**：全數 13 頁投影片皆具備清晰的 `wikiSource` 標記與底部 Badge，無任何外部推測或無來源資訊。
- [x] **並陳檢查**：第 12 與 13 頁完整收錄多架構對照矩陣，保留評測指標歧異（F1 vs Judge）、儲存膨脹爭端與寫入抽取 vs 檢索主導之三大爭端。
- [x] **互動性檢查**：鍵盤事件、按鈕切換、大綱快速跳轉與自適應內部捲動均正常運作。
- **狀態**：全數通過 (PASSED)。

---

## Ingest: MemMachine: A Ground-Truth-Preserving Memory System for Personalized AI Agents (2026)
- **處理日期**：2026-11-17
- **Ingest 來源**：`raw/MemMachine.pdf` (來源: MemMachine.pdf)
- **異動與新增檔案清單**：
  - `wiki/papers/mem_machine.md` (新增單篇文獻深度筆記)
  - `wiki/concepts/ground_truth_preserving_architecture.md` (新增真實情節非損保存與檢索端主導律概念頁)
  - `wiki/concepts/contextualized_retrieval.md` (新增脈絡化擴展檢索與多查詢重排概念頁)
  - `wiki/concepts/late_binding_and_retrieval_agents.md` (新增遲滯綁定難題與 Tool Tree 檢索代理人概念頁)
  - `wiki/concepts/memory_latency_cost_tradeoffs.md` (更新加入 MemMachine 節省 80% Token 與 Pareto 最優實證)
  - `wiki/syntheses/architecture_comparison.md` (更新多架構對比表，填入 MemMachine 完整規格並深度並陳壓縮 vs 保留之設計張力)
  - `wiki/index.md` (更新首頁架構導圖、五大架構閱讀路徑與全頁面索引)
  - `wiki/log.md` (追加日誌記錄)

### Lint 檢查結果
- [x] **連結檢查（Link Check）**：所有雙向連結語法皆正確對應到 `wiki/` 實體 `.md` 檔案，無任何斷鏈。
- [x] **孤島檢查（Island Check）**：所有 `wiki/` 頁面皆已登記於 `wiki/index.md`，且每一內容頁面皆引用 `raw/` 下之出處。
- [x] **出處檢查（Citation Check）**：逐段具備 `(來源: MemMachine.pdf, <章節/頁碼>)`、`(來源: ZEP.pdf, <章節/頁碼>)`、`(來源: Mem0.pdf, <章節/頁碼>)`、`(來源: A-Mem.pdf, <章節/頁碼>)` 或 `(來源: Context_Is_What_You_Need.pdf, <章節/頁碼>)` 精確標記；未出現於原文之細節標註 `[資料不足，待補充]`，符合零幻覺規範。
- [x] **並陳檢查（Balance Check）**：對比表與並陳章節完整保留了 MemMachine「保留真實情節、檢索優化主導」之突破性成果，同時客觀對照了 Mem0、Zep 與 A-Mem 在各異任務與指標上的優劣勢與爭議，未偏廢任何機制。
- **Lint 狀態**：全數通過 (PASSED)。

---

## Ingest: ZEP: A Temporal Knowledge Graph Architecture for Agent Memory (2025)
- **處理日期**：2026-11-17
- **Ingest 來源**：`raw/ZEP.pdf` (來源: ZEP.pdf)
- **異動與新增檔案清單**：
  - `wiki/papers/zep.md` (新增單篇文獻深度筆記)
  - `wiki/concepts/temporal_knowledge_graphs.md` (新增雙時間軸模型與邊失效機制概念頁)
  - `wiki/concepts/hierarchical_memory_subgraphs.md` (新增三層記憶子圖與動態標籤傳播概念頁)
  - `wiki/concepts/multi_faceted_graph_retrieval.md` (新增多面搜尋、多策略重排與 XML 組裝概念頁)
  - `wiki/syntheses/architecture_comparison.md` (更新多架構對比表，填入 Zep 規格並完整並陳 Zep 原文與 Mem0 覆測爭議)
  - `wiki/index.md` (更新首頁架構導圖、閱讀路徑與全頁面索引)
  - `wiki/log.md` (追加日誌記錄)

### Lint 檢查結果
- [x] **連結檢查（Link Check）**：所有雙向連結語法皆正確對應到 `wiki/` 實體 `.md` 檔案，無任何斷鏈。
- [x] **孤島檢查（Island Check）**：所有 `wiki/` 頁面皆已登記於 `wiki/index.md`，且每一內容頁面皆引用 `raw/` 下之出處。
- [x] **出處檢查（Citation Check）**：逐段具備 `(來源: ZEP.pdf, <章節/頁碼>)`、`(來源: Mem0.pdf, <章節/頁碼>)`、`(來源: A-Mem.pdf, <章節/頁碼>)` 或 `(來源: Context_Is_What_You_Need.pdf, <章節/頁碼>)` 精確標記；未出現於原文之細節標註 `[資料不足，待補充]`，符合零幻覺規範。
- [x] **並陳檢查（Balance Check）**：對比表與並陳章節完整保留了 Zep 在 LongMemEval 超長上下文（115k tokens）取得 71.2%（+18.5%）的實證主張，同時完整並陳了 Mem0 團隊在 LOCOMO 上質疑 Zep 儲存膨脹逾 20 倍（>600k tokens）與構圖延遲數小時之反向觀點，未偏廢任何機制。
- **Lint 狀態**：全數通過 (PASSED)。

---

## Ingest: A-Mem: Agentic Memory for LLM Agents (2025)
- **處理日期**：2026-11-17
- **Ingest 來源**：`raw/A-Mem.pdf` (來源: A-Mem.pdf)
- **異動與新增檔案清單**：
  - `wiki/papers/a_mem.md` (新增單篇文獻深度筆記)
  - `wiki/concepts/zettelkasten_agentic_memory.md` (新增卡片盒原子筆記模型概念頁)
  - `wiki/concepts/memory_evolution_and_linking.md` (新增自主連結生成與反向記憶演化概念頁)
  - `wiki/syntheses/architecture_comparison.md` (更新多架構對比表，填入 A-Mem 規格並陳述與 Mem0 評測指標分歧)
  - `wiki/index.md` (更新首頁架構導圖、閱讀路徑與全頁面索引)
  - `wiki/log.md` (追加日誌記錄)

### Lint 檢查結果
- [x] **連結檢查（Link Check）**：所有雙向連結語法皆正確對應到 `wiki/` 實體 `.md` 檔案，無任何斷鏈。
- [x] **孤島檢查（Island Check）**：所有 `wiki/` 頁面皆已登記於 `wiki/index.md`，且每一內容頁面皆引用 `raw/` 下之出處。
- [x] **出處檢查（Citation Check）**：逐段具備 `(來源: A-Mem.pdf, <章節/頁碼>)`、`(來源: Mem0.pdf, <章節/頁碼>)` 或 `(來源: Context_Is_What_You_Need.pdf, <章節/頁碼>)` 精確標記；未出現於原文之細節標註 `[資料不足，待補充]`，符合零幻覺規範。
- [x] **並陳檢查（Balance Check）**：對比表與並陳章節完整保留了 A-Mem 團隊在 ROUGE/BLEU 與開源小模型之優異實測主張，同時並陳 Mem0 團隊在 LLM-as-a-Judge 嚴格語意評判下對 A-Mem 的覆測結果與延遲對照，未偏廢任何機制與論述。
- **Lint 狀態**：全數通過 (PASSED)。

---

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
