# ZEP: A Temporal Knowledge Graph Architecture for Agent Memory

## 白話一句話定義
Zep 是一套專為生產級 AI 代理人設計的時間感知知識圖譜記憶層服務（核心引擎為 Graphiti），透過階層式三層子圖（情節子圖、語意實體子圖、社群子圖）組織對話與企業數據，並引入雙時間軸模型（真實世界時間與資料庫交易時間）與邊失效機制，在超長對話（115k tokens）中實現準確度提升 18.5% 並降低 90% 響應延遲 (來源: ZEP.pdf, Abstract/Page 1, Section 2/Page 2-4, Section 4.3/Page 7)。

---

## 核心痛點
1. **傳統 RAG 對動態對話與企業演變資料的無力**：
   - 傳統 RAG 假設檢索語料庫是廣泛且靜態的（文件寫入後極少變更）(來源: ZEP.pdf, Section 1/Page 1)。然而 AI 代理人需要持續吸收使用者動態對話、企業業務數據與不斷演變的實體關係，靜態文件切塊檢索無法維護歷史關聯與時間有效性 (來源: ZEP.pdf, Section 1/Page 1-2)。
2. **長上下文窗口的推論衰減與極高延遲**：
   - 在企業級長程對話（平均 115,000 tokens）中，即便當前前沿模型能塞入此長度，Full-Context 推論仍引發顯著的注意力利用率下降、極端的高延遲（如 GPT-4o 需 28.9 秒）與高昂 Token 成本 (來源: ZEP.pdf, Section 1/Page 1, Section 4.3/Page 7)。
3. **既有評測基準（如 DMR）的侷限性**：
   - MemGPT 提出的 DMR（Deep Memory Retrieval）評測每段對話僅 60 則訊息，完全能塞入現代上下文窗口，且題目全為單輪事實檢索與模糊提問，無法真實反映企業代理人所需的時序推理、跨 Session 綜合與偏好維護能力 (來源: ZEP.pdf, Section 4.2/Page 6)。

---

## 系統架構與運作流程（Graphiti 核心引擎）

Zep 將記憶建模為時間感知動態知識圖譜 $\mathcal{G} = (\mathcal{N}, \mathcal{E}, \phi)$，其中 $\mathcal{N}$ 為節點，$\mathcal{E}$ 為邊，$\phi: \mathcal{E} \to \mathcal{N} \times \mathcal{N}$ 為關聯函數 (來源: ZEP.pdf, Section 2/Page 2)。

```mermaid
flowchart TD
    Raw["原始輸入 (訊息/JSON/文字) + t_ref"] --> Ge["1. 情節子圖 (Episode Subgraph Ge)<br/>非損資料儲存、雙向索引追溯"]
    Ge -->|實體辨識 + Reflexion 反思| Gs["2. 語意實體子圖 (Semantic Subgraph Gs)<br/>實體消歧、超邊多實體事實、邊失效"]
    Gs -->|動態標籤傳播 (Label Propagation)| Gc["3. 社群子圖 (Community Subgraph Gc)<br/>實體聚類、Map-Reduce 階層摘要"]
    
    Q["用戶查詢 α"] --> S["檢索搜尋 (Search ϕ)<br/>語意相似 + BM25 + 圖廣度優先 (BFS)"]
    S --> R["重排精煉 (Reranker ρ)<br/>RRF / MMR / 提及頻率 / 圖距離 / Cross-Encoder"]
    R --> C["上下文建構 (Constructor χ)<br/>格式化 FACTS (含時間範圍) 與 ENTITIES"]
    C --> Out["推論提示詞 β"]
```

### 1. 三層階層式子圖架構（Hierarchical Subgraphs）
借鑒認知心理學對人類記憶的劃分（情節記憶 vs. 語意記憶）(來源: ZEP.pdf, Section 2/Page 2)：
1. **情節子圖（Episode Subgraph, $\mathcal{G}_e$）**：
   - **節點 $n_i \in \mathcal{N}_e$**：儲存原始未加工資料（對話訊息、文字或 JSON），保留 actor 與訊息參照時間戳記 $t_{\text{ref}}$，構成非損（Non-lossy）基礎儲存庫 (來源: ZEP.pdf, Section 2/Page 2, Section 2.1/Page 2)。
   - **邊 $e_i \in \mathcal{E}_e$**：雙向索引邊，將情節節點與抽取出的語意實體相連，支援雙向遍歷（由事實反查原始對話出處以供引用，或由對話迅速檢索關聯實體）(來源: ZEP.pdf, Section 2/Page 2, Section 2.1/Page 3)。
2. **語意實體子圖（Semantic Entity Subgraph, $\mathcal{G}_s$）**：
   - **實體節點 $n_i \in \mathcal{N}_s$**：代表抽取並消歧後的實體。
   - **實體邊（語意邊）$e_i \in \mathcal{E}_s$**：代表實體間的關係與事實，支援超邊（Hyper-edges）以表達多實體複合事實 (來源: ZEP.pdf, Section 2/Page 2, Section 2.2.2/Page 3)。
3. **社群子圖（Community Subgraph, $\mathcal{G}_c$）**：
   - **社群節點 $n_i \in \mathcal{N}_c$**：代表高度連通的實體聚類，內含 Map-Reduce 式高階抽象摘要與主題命名，提供對領域的全域宏觀理解 (來源: ZEP.pdf, Section 2/Page 2, Section 2.3/Page 4)。
   - **社群邊 $e_i \in \mathcal{E}_c$**：將社群節點與其成員實體相連 (來源: ZEP.pdf, Section 2/Page 2)。

### 2. 雙時間軸模型（Bi-Temporal Modeling）
Zep 創新性地同時維護兩條時間維度，為邊上的每個事實標註四個時間戳記 (來源: ZEP.pdf, Section 2.1/Page 2, Section 2.2.3/Page 3)：
- **交易時間軸（Transactional Timeline, $T'$）**：記錄系統資料庫的操作審計順序。
  - $t'_{\text{created}}$：事實寫入圖譜的時間 (來源: ZEP.pdf, Section 2.2.3/Page 3)。
  - $t'_{\text{expired}}$：事實在系統中被廢棄/失效的時間 (來源: ZEP.pdf, Section 2.2.3/Page 3)。
- **真實世界時間軸（Chronological Timeline, $T$）**：記錄事實與事件在真實世界中實際發生的時間跨度。
  - $t_{\text{valid}}$：事實開始成立的真實時間（例如由「兩週前」透過 $t_{\text{ref}}$ 換算之標準 ISO 8601 日期）(來源: ZEP.pdf, Section 2.2.3/Page 3, Section 6.1.5/Page 10)。
  - $t_{\text{invalid}}$：事實失效或結束的真實時間 (來源: ZEP.pdf, Section 2.2.3/Page 3)。

---

## 寫入 / 更新機制

### 1. 實體辨識、Reflexion 與消歧（Resolution）
- **脈絡視窗**：輸入當前訊息及前 $n=4$ 則訊息（提供兩輪完整對話脈絡），發話者自動辨識為實體 (來源: ZEP.pdf, Section 2.2.1/Page 3)。
- **Reflexion 反思機制**：抽取後引入 Reflexion 自我檢驗步驟，以最大化實體覆蓋率並壓制 LLM 幻覺 (來源: ZEP.pdf, Section 2.2.1/Page 3)。
- **多路消歧（Resolution）**：實體名稱透過 BGE-m3 映射至 1024 維向量空間進行餘弦相似搜尋，同時對既有實體名稱與摘要執行 Full-Text 搜尋；篩選之候選實體交由 LLM 判定 `is_duplicate`，判定重複則更新名稱與摘要 (來源: ZEP.pdf, Section 2.2.1/Page 3, Section 4.1/Page 5, Section 6.1.2/Page 9)。
- **預定義 Cypher 查詢**：寫入 Neo4j 時採用**預先撰寫的 Cypher 模板**，堅決杜絕 LLM 即時生成資料庫查詢，從工程底層杜絕語法幻覺與安全風險 (來源: ZEP.pdf, Section 2.2.1/Page 3)。

### 2. 事實邊去重與邊失效機制（Edge Invalidation）
- **去重空間約束**：邊去重的混合搜尋僅侷限於相同實體對（Same Entity Pairs）之間，大幅縮減搜尋複雜度並防止跨實體錯連 (來源: ZEP.pdf, Section 2.2.2/Page 3)。
- **邊失效（Edge Invalidation）**：
  - 當新邊加入時，LLM 檢驗其與既有語意相關邊是否存在時序重疊的邏輯衝突 (來源: ZEP.pdf, Section 2.2.3/Page 3)。
  - 若偵測到衝突矛盾，依據交易時間軸 $T'$「新資訊優先」原則，系統將受影響舊邊的 $t_{\text{invalid}}$ 設為新邊的 $t_{\text{valid}}$，標記其過期失效而非物理抹除，兼顧動態更新與歷史軌跡回溯 (來源: ZEP.pdf, Section 2.2.3/Page 3)。

### 3. 動態標籤傳播（Dynamic Label Propagation）社群更新
- 社群劃分捨棄靜態全局的 Leiden 演算法，改採**標籤傳播演算法（Label Propagation Algorithm）** (來源: ZEP.pdf, Section 2.3/Page 4)。
- 當新實體節點 $n_i \in \mathcal{N}_s$ 寫入時，僅需單步遞迴遍歷鄰居節點的社群分佈，直接歸併至多數鄰居所屬之社群並更新社群摘要，避免了每次寫入皆需全局重新聚類的巨大延遲與 LLM 調用開銷（輔以定期背景全局刷新）(來源: ZEP.pdf, Section 2.3/Page 4)。

---

## 記憶檢索機制（$f(\alpha) \to \beta$）

檢索管線由搜尋、重排、組裝三步組合而成：$f(\alpha) = \chi(\rho(\phi(\alpha))) = \beta$ (來源: ZEP.pdf, Section 3/Page 4)：
1. **多路複合搜尋（Search, $\phi$）**：
   - 轉化為語意邊 $\mathcal{E}_s$、實體節點 $\mathcal{N}_s$、社群節點 $\mathcal{N}_c$ 的候選三元組 (來源: ZEP.pdf, Section 3/Page 4)。
   - **$\phi_{\text{cos}}$（餘弦語意搜尋）**：捕捉實體與事實的語意相似性 (來源: ZEP.pdf, Section 3.1/Page 4-5)。
   - **$\phi_{\text{bm25}}$（BM25 全文檢索）**：Lucene 關鍵詞精確比對 (來源: ZEP.pdf, Section 3.1/Page 4-5)。
   - **$\phi_{\text{bfs}}$（圖廣度優先走訪）**：以近期情節（Episodes）為種子節點，在圖譜中進行 $n$-hop 鄰近擴展，檢索上下文拓撲關聯 (來源: ZEP.pdf, Section 3.1/Page 4-5)。
2. **重排精煉（Reranker, $\rho$）**：
   - 支援 RRF（Reciprocal Rank Fusion）、MMR（Maximal Marginal Relevance）(來源: ZEP.pdf, Section 3.2/Page 5)。
   - **情節提及頻率重排**（Graph-based Episode-mentions Reranker）：優先呈現高頻提及事實 (來源: ZEP.pdf, Section 3.2/Page 5)。
   - **節點距離重排**（Node Distance Reranker）：依據相對於指定中心節點的圖拓撲距離排序 (來源: ZEP.pdf, Section 3.2/Page 5)。
   - **Cross-Encoder 重排**：使用 BGE-m3 或 LLM 交叉注意力精確評分 (來源: ZEP.pdf, Section 3.2/Page 5, Section 4.1/Page 5)。
3. **上下文組裝（Constructor, $\chi$）**：
   - 將篩選出的 Top-20 實體與事實結構化格式化為 XML 區塊（`<FACTS>` 標註事實文字與 `[t_valid, t_invalid]` 區間，`<ENTITIES>` 標註實體名稱與摘要），傳入 Agent 作為 Working Memory (來源: ZEP.pdf, Section 3/Page 4, Section 4/Page 5)。

---

## 評測實驗與實證數據

### 1. DMR 基準評測（Deep Memory Retrieval）
依據 Table 1 數據 (來源: ZEP.pdf, Section 4.2/Page 6, Table 1/Page 6)：
- 使用 GPT-4-turbo：Zep 準確率達 **94.8%**（超越 MemGPT 的 93.4%、Full-Conversation 的 94.4% 與遞迴摘要的 35.3%）。
- 使用 GPT-4o-mini：Zep 準確率達 **98.2%**（超越 Full-Conversation 的 98.0% 與 Session Summaries 的 88.0%）。

### 2. LongMemEval 企業長對話基準（115,000 tokens）
評測橫跨 6 大題型（包含單會話使用者、單會話助理、使用者偏好、多會話綜合、知識更新、時序推理）(來源: ZEP.pdf, Section 4.3/Page 6-7, Table 2-3/Page 7)：
- **端到端表現（Table 2）**：
  - **GPT-4o-mini**：
    - Full-Context（115k tokens）：得分 55.4%，延遲 31.3 秒。
    - **Zep（1.6k tokens）**：得分 **63.8%**（相對提升 **15.2%**），延遲僅 **3.20 秒**（延遲降低 **89.8%**）。
  - **GPT-4o**：
    - Full-Context（115k tokens）：得分 60.2%，延遲 28.9 秒。
    - **Zep（1.6k tokens）**：得分 **71.2%**（相對提升 **18.5%**），延遲僅 **2.58 秒**（延遲降低 **91.1%**）。
- **關鍵細項題型提升（Table 3, GPT-4o）**：
  - **Single-session-preference（偏好追蹤）**：由 20.0% 暴增至 **56.7%**（相對增長 **184%**）(來源: ZEP.pdf, Table 3/Page 7)。
  - **Temporal-reasoning（時序推理）**：由 45.1% 躍升至 **62.4%**（相對增長 **38.4%**）(來源: ZEP.pdf, Table 3/Page 7)。
  - **Multi-session（跨會話綜合）**：由 44.3% 提升至 **57.9%**（相對增長 **30.7%**）(來源: ZEP.pdf, Table 3/Page 7)。
  - **Single-session-assistant 異常**：GPT-4o 下由 94.6% 降至 80.4%，GPT-4o-mini 下由 81.8% 降至 75.0%，論文坦承檢索助理過往發言之能力仍需改進 (來源: ZEP.pdf, Table 3/Page 7, Section 4.3.2/Page 8)。

---

## 優缺點與 Trade-offs

### 優點
1. **強大的時間感知與雙時間軸歷史追蹤**：
   - 透過 $T$ 與 $T'$ 雙時間軸以及邊失效（$t_{\text{valid}}, t_{\text{invalid}}$），能夠精確解析相對時間並追蹤實體關係狀態隨時間的變更 (來源: ZEP.pdf, Section 2.1/Page 2, Section 2.2.3/Page 3)。
2. **超長對話下極致的壓縮比與低延遲**：
   - 將 115,000 tokens 的龐大企業對話壓縮至僅 1,600 tokens 結構化上下文（壓縮率逾 98%），同時將響應時間自約 30 秒縮減至 2.58 秒（延遲下降 91%），並大幅提高準確率 (來源: ZEP.pdf, Section 4.3.2/Page 7)。
3. **工程穩定性保障**：
   - 採用預定義 Cypher 查詢模板，避免 LLM 動態生成資料庫語句之語法錯誤與安全風險 (來源: ZEP.pdf, Section 2.2.1/Page 3)。

### 缺點與跨文獻 Trade-offs / 觀點衝突（必備並陳對照）
1. **助理歷史語句檢索之退化（Single-Session-Assistant Drop）**：
   - 在 LongMemEval 中，對助理先前發言的檢索準確率下降約 9%~17.7%，顯示實體圖譜更側重使用者資訊與客觀事實，對助理自體發言之保留相對較弱 (來源: ZEP.pdf, Table 3/Page 7, Section 4.3.2/Page 8)。
2. **跨文獻實測觀點衝突（Zep 原文 vs. Mem0 論文）**：
   - **Zep 團隊觀點（Rasmussen et al., 2025）**：
     在 115k tokens 的真實企業基準 LongMemEval 上，Zep 取得了超越 Full-Context 的 71.2% 準確率，推論延遲僅 2.58 秒，檢索上下文僅 1.6k tokens，且動態標籤傳播顯著降低了社群維護延遲 (來源: ZEP.pdf, Section 2.3/Page 4, Table 2/Page 7)。
   - **Mem0 團隊實測質疑（Chhikara et al., 2025）**：
     Mem0 團隊在 LOCOMO（26k tokens）評測 Zep 商業平台版指出：
     1. **儲存體積嚴重膨脹**：Zep 在圖譜各節點快取完整抽象摘要並於邊上記錄事實，導致單段對話儲存膨脹超過 **600k tokens**（約為原始對話之 23 倍，相較 Mem0 僅 7k tokens 與 Mem0^g 之 14k tokens）(來源: Mem0.pdf, Section 4.5/Page 14)。
     2. **非同步構圖延遲過長**：Zep 構圖依賴大量非同步後台 LLM 調用，新記憶寫入後需延遲數小時才能被正確檢索，不具備即時互動能力 (來源: Mem0.pdf, Section 4.5/Page 14)。
     3. **LOCOMO 總體評分**：Zep 總體 Judge 得分為 65.99%，雖在 Open-Domain 奪冠（76.60%），但在 Multi-Hop（41.35%）與 Temporal（49.31%）上均顯著落後於 Mem0 與 Mem0^g (來源: Mem0.pdf, Table 1-2/Page 9, 11)。
   - *（依據知識庫規範，上述各方實測數據與架構機制皆已完整保留並陳，詳見 [[architecture_comparison]]）*。

---

## 相關核心條目
- [[temporal_knowledge_graphs]]：雙時間軸模型（$T$ 與 $T'$）與邊失效機制。
- [[hierarchical_memory_subgraphs]]：情節子圖、語意實體子圖、社群子圖與動態標籤傳播。
- [[multi_faceted_graph_retrieval]]：多路搜尋（語意+BM25+圖BFS）、多策略重排與上下文組裝。
- [[graph_memory_representations]]：Mem0^g 與 Zep 之圖譜建模對比。
- [[memory_latency_cost_tradeoffs]]：全上下文、RAG、Mem0 與 Zep 之成本與延遲綜合分析。
- [[architecture_comparison]]：跨文獻記憶架構總對比表。
