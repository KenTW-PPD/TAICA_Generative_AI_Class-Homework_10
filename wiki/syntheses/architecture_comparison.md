# 記憶架構多維度並陳對比表

## 架構對比矩陣

本表旨在橫向對比不同 LLM 記憶管理架構的核心機制、生命週期與工程權衡。依據知識庫規範，本表將隨著各篇文獻之 Ingest 逐步擴充，完整保留各系統機制的差異並陳，不偏廢任一方觀點。

| 維度 / 架構 | 傳統 Context Window（In-Context / Context Stuffing） | Mem0 | A-Mem | ZEP | MemMachine |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **文獻出處** | [[context_is_what_you_need]] (來源: Context_Is_What_You_Need.pdf) | [待 Ingest 補充] | [待 Ingest 補充] | [待 Ingest 補充] | [待 Ingest 補充] |
| **記憶載體與儲存位置** | 推論期上下文（KV-Cache / Prompt 文本），無外部儲存 (來源: Context_Is_What_You_Need.pdf, Section 1/Page 2, Section 3.2/Page 6) | [待 Ingest 補充] | [待 Ingest 補充] | [待 Ingest 補充] | [待 Ingest 補充] |
| **寫入機制（Extraction）** | 靜態拼接（Prompt Concatenation）；推論時單次傳入全部內容 (來源: Context_Is_What_You_Need.pdf, Section 3.2/Page 6) | [待 Ingest 補充] | [待 Ingest 補充] | [待 Ingest 補充] | [待 Ingest 補充] |
| **更新與整合機制（Consolidation）** | 無動態更新機制（Stateless）；無持久化狀態覆寫或壓縮機制，外部記憶庫機制標註為 `[資料不足，待補充]` (來源: Context_Is_What_You_Need.pdf, Section 3.1-3.2/Page 5-6) | [待 Ingest 補充] | [待 Ingest 補充] | [待 Ingest 補充] | [待 Ingest 補充] |
| **檢索機制（Retrieval）** | 全注意力矩陣計算（Full Transformer Self-Attention），無外部索引檢索 (來源: Context_Is_What_You_Need.pdf, Section 1/Page 1-2, Section 2.1/Page 3) | [待 Ingest 補充] | [待 Ingest 補充] | [待 Ingest 補充] | [待 Ingest 補充] |
| **有效工作容量（Effective Capacity）** | **受限於 MECW 光譜**：單針檢索達數萬 tokens，但多目標/排序/統計任務在 100~1,000 tokens 即崩塌，2,000 tokens 幻覺率可達 99% (來源: Context_Is_What_You_Need.pdf, Abstract/Page 1, Section 7.3/Page 10) | [待 Ingest 補充] | [待 Ingest 補充] | [待 Ingest 補充] | [待 Ingest 補充] |
| **核心工程優點** | 零外部組件依賴；在極短上下文（<500 tokens）內準確率接近 100%，能壓低基礎幻覺率 (來源: Context_Is_What_You_Need.pdf, Section 6.1/Page 9, Section 7.3/Page 10) | [待 Ingest 補充] | [待 Ingest 補充] | [待 Ingest 補充] | [待 Ingest 補充] |
| **主要痛點與 Trade-offs** | 超過 MECW 後準確率迅速趨零；於多 Agent 鏈路中會誘發毀滅性的 [[cascading_failures]]；推論成本隨輸入線性/二次方膨脹 (來源: Context_Is_What_You_Need.pdf, Section 7.1/Page 9, Section 7.3/Page 10) | [待 Ingest 補充] | [待 Ingest 補充] | [待 Ingest 補充] | [待 Ingest 補充] |

---

## 關鍵機制差異與觀點並陳

### 1. 工作記憶（Working Memory）與長期記憶（Long-Term Memory）的邊界
- **純上下文觀點**：[[context_is_what_you_need]] 實證指出，Context Window 的本質是暫時性的 Working Memory，其可信容量（MECW）受任務複雜度極度限縮，絕不可將其視為無限容量的 Long-Term Memory (來源: Context_Is_What_You_Need.pdf, Section 1/Page 2, Section 7.1/Page 9)。
- **外部系統觀點**：待後續文獻（Mem0、A-Mem、ZEP、MemMachine）Ingest 後並陳對照外部記憶層如何彌補 Context Window 的有效容量缺陷。

### 2. 檢索增強（RAG）的雙刃劍效應
- **受控上下文增益**：當檢索內容體積控制在 MECW 之內時，RAG 能有效提升模型準確度至近 100% (來源: Context_Is_What_You_Need.pdf, Section 6.1/Page 9, Section 7.3/Page 10)。
- **過載上下文衰減**：一旦檢索內容超出 MECW（例如達到 2,000 tokens 以上），RAG 不僅無法提高準確率，反而使模型陷入極端幻覺（高達 99% 錯誤率），甚至比無 RAG 時表現更差 (來源: Context_Is_What_You_Need.pdf, Section 6.1/Page 9, Section 7.3/Page 10)。

---

## 相關引用與頁面
- 出處論文深度筆記：[[context_is_what_you_need]]
- 核心概念條目：[[mecw_vs_mcw]]、[[cascading_failures]]、[[in_context_degradation]]
