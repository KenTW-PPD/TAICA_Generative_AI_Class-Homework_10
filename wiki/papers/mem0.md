# Mem0: Building Production-Ready AI Agents with Scalable Long-Term Memory

## 白話一句話定義
Mem0 是一個專為生產環境 AI 代理人設計的可擴展長期記憶架構，透過專屬模組動態萃取對話要點、利用 LLM 工具調用執行四種記憶庫狀態更新（新增、更新、刪除、無操作），並透過圖譜擴充版（Mem0^g）將記憶建模為有向標記圖，在大幅降低延遲（p95 降低 91%）與 Token 成本（節省逾 90%）的同時，維持跨 Session 的長期一致性 (來源: Mem0.pdf, Abstract/Page 1, Section 2/Page 3-5)。

---

## 核心痛點
1. **固定上下文窗口無法支撐跨 Session 長期對話**：
   - 儘管主流模型窗口擴增至數十萬乃至千萬 tokens，但模型在資訊超出窗口後即被「重設」（Reset），無法在不同會話間保留使用者偏好與歷史事實（例如遺忘素食者偏好而推薦葷食）(來源: Mem0.pdf, Section 1/Page 1-2)。
2. **長文本推論的雜訊干擾與注意力衰退**：
   - 真實對話鮮少維持單一主題連貫性（例如提及飲食偏好後接著數小時無關的程式設計討論）。若直接將數萬 tokens 全量歷史餵給模型（Full-Context），關鍵事實極易被雜訊淹沒，且注意力機制會隨 token 距離拉長而衰減 (來源: Mem0.pdf, Section 1/Page 2-3)。
3. **Full-Context 的運算延遲與成本瓶頸**：
   - 在 LOCOMO 基準測試（平均對話約 26,000 tokens）中，Full-Context 每次查詢必須讀取全部歷史，導致 p95 總延遲高達 17.117 秒，且 Token 消耗量巨大，無法應用於互動式即時場景 (來源: Mem0.pdf, Section 4.3-4.4/Page 11, 13)。
4. **既有記憶系統與圖譜架構的工程缺陷**：
   - 傳統 RAG 檢索原始文本塊（Chunk）會混入大量雜訊，推論品質上限低（LLM-as-a-Judge 得分僅約 61%）(來源: Mem0.pdf, Section 4.3/Page 13)。
   - 現有專用記憶平台（如 Zep）在記憶圖譜節點中快取完整摘要，造成記憶庫 Token 體積嚴重膨脹（超過 600k tokens，為原始對話之 20 倍以上），且構圖依賴大量非同步後台 LLM 呼叫，需耗時數小時才能完成索引，無法即時檢索最新記憶 (來源: Mem0.pdf, Section 4.5/Page 14)。

---

## 系統架構與運作流程

Mem0 提出兩套互補的記憶管理架構：基礎版 **Mem0**（基於密集自然語言事實與向量索引）與圖譜增強版 **Mem0^g**（基於有向標記知識圖譜）(來源: Mem0.pdf, Section 2/Page 3)。

### 1. Mem0 基礎架構：雙階段增量管線
Mem0 採增量式處理範式，接收當前對話訊息對 $(m_{t-1}, m_t)$（通常為使用者訊息與助理回覆），劃分為兩大階段 (來源: Mem0.pdf, Section 2.1/Page 3-4, Figure 2/Page 4)：

```mermaid
flowchart TD
    A["新訊息對 (mt-1, mt)"] --> B["萃取階段 (Extraction Phase)"]
    S["非同步全局對話摘要 S"] --> B
    R["近期對話視窗 (m=10)"] --> B
    B -->|候選事實清單 Ω| C["更新階段 (Update Phase)"]
    DB[(向量資料庫 / Memory Store)] -->|檢索 Top-s 相似記憶 (s=10)| C
    C -->|LLM Function Calling| D{"操作分類 (Classify)"}
    D -->|ADD| E["新增事實 (GenerateUniqueID)"]
    D -->|UPDATE| F["更新增強 (Replace with richer info)"]
    D -->|DELETE| G["刪除矛盾 (Remove contradicted)"]
    D -->|NOOP| H["無操作 (No change)"]
    E --> DB
    F --> DB
    G --> DB
```

#### (1) 記憶萃取階段（Extraction Phase）
- **雙重脈絡注入**：為了讓 LLM 在萃取新訊息記憶時兼具全局主題與近期細節，構建綜合 Prompt $P = (S, \{m_{t-m}, \dots, m_{t-2}\}, m_{t-1}, m_t)$ (來源: Mem0.pdf, Section 2.1/Page 3-4)：
  - **全局摘要 $S$**：由獨立的非同步模組定期刷新，提供對話主題概覽，不阻塞主推論管線 (來源: Mem0.pdf, Section 2.1/Page 4)。
  - **近期對話視窗**：保留前 $m$ 則歷史訊息（實驗中設為 $m=10$），提供未被摘要收錄的精確近期語境 (來源: Mem0.pdf, Section 2.1/Page 4, 5)。
- **候選事實生成**：由 LLM 萃取函數 $\phi(P)$ 生成候選事實集合 $\Omega = \{\omega_1, \omega_2, \dots, \omega_n\}$ (來源: Mem0.pdf, Section 2.1/Page 4)。

#### (2) 記憶更新階段（Update Phase）
- 針對每一筆候選事實 $\omega_i \in \Omega$，透過向量嵌入從資料庫檢索出前 $s$ 筆語意最相似的既有記憶（實驗設 $s=10$）(來源: Mem0.pdf, Section 2.1/Page 4, 5)。
- 透過 Tool Call / Function Calling 將候選事實與相似記憶提交給 LLM，直接利用 LLM 的語意推理能力決定記憶操作 (來源: Mem0.pdf, Section 2.1/Page 4)。

### 2. Mem0^g 架構：圖譜增強記憶表徵
Mem0^g 將記憶組織為有向標記圖 $G = (V, E, L)$ (來源: Mem0.pdf, Section 2.2/Page 5, Figure 3/Page 5)：
- **節點 $V$（實體）**：包含實體類型分類（Person, Location, Event 等）、語意嵌入向量 $e_v$、建立時間戳記 $t_v$ 等詮釋資料 (來源: Mem0.pdf, Section 2.2/Page 5)。
- **邊 $E$（關係）**：以三元組 $(v_s, r, v_d)$ 儲存，標記關係標籤 $r$（如 `lives_in`, `prefers`, `owns`, `happened_on`）(來源: Mem0.pdf, Section 2.2/Page 5, 6)。
- **標籤 $L$**：指定節點的語意類型 (來源: Mem0.pdf, Section 2.2/Page 5)。
- **底層儲存**：圖資料庫採用 Neo4j；LLM 萃取與更新引擎採用 `gpt-4o-mini`，嵌入模型採用 `text-embedding-small-3` (來源: Mem0.pdf, Section 2.2/Page 5, 6, Section 3.3/Page 8)。

---

## 寫入 / 更新機制

### 1. Mem0 四大記憶庫操作原子邏輯
Mem0 不依賴外部獨立分類器，而是由 LLM 推理判定並調用工具執行（Algorithm 1）(來源: Mem0.pdf, Section 2.1/Page 4, Appendix B/Page 21)：
1. **ADD（新增）**：當候選事實與既有記憶無語意相似項時（`¬SemanticallySimilar(f, M)`），生成唯一識別碼 `GenerateUniqueID()`，並以 `(id, f, "ADD")` 寫入記憶庫 (來源: Mem0.pdf, Appendix B/Page 21)。
2. **UPDATE（更新）**：當候選事實補充或擴充了既有相似記憶（`Augments(f, M)`），且新事實資訊量更高時（`InformationContent(f) > InformationContent(mi)`），將舊記憶替換為更豐富的新事實 `(idi, f, "UPDATE")` (來源: Mem0.pdf, Appendix B/Page 21)。
3. **DELETE（刪除）**：當新事實與既有記憶產生語意衝突或矛盾時（`Contradicts(f, M)`），自記憶庫中物理剔除被推翻的舊記憶 $M \setminus \{m_i\}$ (來源: Mem0.pdf, Appendix B/Page 21)。
4. **NOOP（無操作）**：當事實已存在或屬於無關雜訊時，不對記憶庫進行任何修改 (來源: Mem0.pdf, Appendix B/Page 21)。

### 2. Mem0^g 衝突偵測與標記失效（Temporal Invalidation）
在圖譜更新中，若新關係與既有關係發生衝突，Mem0^g 採用「標記失效」（Mark as Invalid）而非直接物理刪除 (來源: Mem0.pdf, Section 2.2/Page 6)：
- 透過 LLM 更新解析器（Update Resolver）判定舊關係已過期，將其狀態更新為無效 (來源: Mem0.pdf, Section 2.2/Page 6)。
- **時間維度推理優勢**：保留帶有時間戳記的過期關係節點，讓模型能夠回溯歷史狀態並正確回答時間先後順序問題（如「去年去過哪裡」與「現在住哪裡」之演變）(來源: Mem0.pdf, Section 2.2/Page 6, Section 4.1/Page 10)。

---

## 記憶檢索機制

### 1. Mem0 的密集語意檢索
- 查詢時依據輸入 Prompt 計算 Dense Embedding，於向量資料庫中檢索語意相關之密集自然語言事實，過濾雜訊後傳入生成模型 (來源: Mem0.pdf, Section 2.1/Page 4-5, Section 4.4/Page 13)。

### 2. Mem0^g 雙路整合檢索策略（Dual-Approach Strategy）
Mem0^g 結合實體中心與語意三元組兩種存取路徑 (來源: Mem0.pdf, Section 2.2/Page 6)：
1. **實體中心子圖檢索（Entity-Centric Subgraph）**：
   - 識別查詢中的關鍵實體，透過語意相似度定位圖譜中的錨點節點（Anchor Nodes），沿其入邊與出邊遍歷，構建具備上下文脈絡的相鄰子圖 (來源: Mem0.pdf, Section 2.2/Page 6)。
2. **語意三元組檢索（Semantic Triplet Matching）**：
   - 將完整查詢編碼為稠密向量，與圖譜中各關係三元組的文本編碼進行細粒度相似度計算，依得分排序並回傳高於門檻之三元組 (來源: Mem0.pdf, Section 2.2/Page 6)。
- 雙路機制兼顧特定實體的精確追蹤與廣泛概念查詢之涵蓋率 (來源: Mem0.pdf, Section 2.2/Page 6)。

---

## 評測實驗與實證數據

### 1. LOCOMO 基準與評測指標
- **資料集規格**：10 段長對話，每段約 600 個輪次（Dialogues）、平均 26,000 tokens，每段對話涵蓋約 200 道題，分類為 Single-hop、Multi-hop、Open-domain、Temporal 四類 (來源: Mem0.pdf, Section 3.1/Page 6)。
- **評測指標**：
  - 品質指標：$F_1$ 分數、$B_1$（BLEU-1），以及由能力更強之外部 LLM 評判的 **LLM-as-a-Judge ($J$)**（採 10 次獨立運行平均 $\pm$ 1 標準差）(來源: Mem0.pdf, Section 3.2/Page 7)。論文特別指出，詞彙重疊度指標（F1/BLEU）在事實性評估上存在嚴重缺陷（例如將「Alice 出生於 3 月」誤答為「7 月」仍可獲極高 F1），因此必須以 $J$ 指標為核心 (來源: Mem0.pdf, Section 3.2/Page 7)。
  - 部署指標：檢索 Context Token 消耗量、檢索搜尋延遲（Search Latency: p50, p95）與整體端到端延遲（Total Latency: p50, p95）(來源: Mem0.pdf, Section 3.2/Page 7)。

### 2. 跨題型品質對比（LOCOMO 實驗結果）
依據 Table 1 數據 (來源: Mem0.pdf, Section 4.1/Page 9, Table 1/Page 9)：
- **Single-Hop（單跳檢索）**：
  - Mem0 取得最佳表現（$F_1=38.72, B_1=27.13, J=67.13 \pm 0.65$）。
  - Mem0^g 表現略低（$J=65.71$），證實單一輪次事實無需複雜圖結構，密集語言記憶效率最高 (來源: Mem0.pdf, Section 4.1/Page 9)。
- **Multi-Hop（多跳跨 Session 推理）**：
  - Mem0 表現最優（$F_1=28.64, J=51.15 \pm 0.31$），大幅領先 A-Mem（$J=18.85$）與 Zep（$J=41.35$）。
  - Mem0^g 在此題型得分略降（$J=47.19$），顯示複雜整合推理中，遍歷過度冗餘的圖譜路徑可能引發開銷與雜訊 (來源: Mem0.pdf, Section 4.1/Page 10)。
- **Temporal（時間序推理）**：
  - **Mem0^g 奪得全場最高分**（$F_1=51.55, B_1=40.28, J=58.13 \pm 0.44$），顯著超越 Mem0（$J=55.51$）與 Zep（$J=49.31$），而 OpenAI 內建記憶在此題型完全失能（$J=21.71$），主因是 OpenAI 萃取記憶時普遍缺失時間戳記 (來源: Mem0.pdf, Section 4.1/Page 10)。
- **Open-Domain（開放領域與常識整合）**：
  - Zep 略微領先（$J=76.60$），Mem0^g 以 $J=75.71$ 緊追其後，Mem0 則為 $J=72.93$ (來源: Mem0.pdf, Section 4.1/Page 10)。
- **總體品質（Overall $J$）** (來源: Mem0.pdf, Table 2/Page 11)：
  - Full-Context (72.90%) > Mem0^g (68.44%) > Mem0 (66.88%) > Zep (65.99%) > Best RAG ($k=2, 256$, 60.97%) > LangMem (58.10%) > OpenAI (52.90%) > A-Mem (48.38%)。

### 3. 部署效率與開銷對比
依據 Table 2 及 Section 4.4-4.5 數據 (來源: Mem0.pdf, Table 2/Page 11, Section 4.4-4.5/Page 13-14)：
- **搜尋延遲（Search Latency）**：
  - Mem0 為全場最低：p50 為 0.148 秒，p95 僅 0.200 秒 (來源: Mem0.pdf, Table 2/Page 11, Section 4.4/Page 13)。
  - Mem0^g 次之：p50 為 0.476 秒，p95 為 0.657 秒。
  - 對照組：Zep p50=0.513s / p95=0.778s；A-Mem p50=0.668s / p95=1.485s；LangMem 出現極高延遲（p50=17.99s, p95=59.82s）(來源: Mem0.pdf, Table 2/Page 11)。
- **端到端總延遲（Total Latency）**：
  - Mem0 p50=0.708 秒，p95=1.440 秒（相較 Full-Context 的 17.117 秒，延遲下降 91.6%）(來源: Mem0.pdf, Table 2/Page 11, Section 4.3/Page 13)。
  - Mem0^g p50=1.091 秒，p95=2.590 秒（相較 Full-Context 下降 84.9%）(來源: Mem0.pdf, Table 2/Page 11, Section 4.3/Page 13)。
- **記憶體儲存預算（Token Footprint per conversation）**：
  - Mem0 平均每段對話僅佔用 **7k tokens** 儲存空間 (來源: Mem0.pdf, Section 4.5/Page 14)。
  - Mem0^g 因節點與邊的關係擴充，佔用約 **14k tokens** (來源: Mem0.pdf, Section 4.5/Page 14)。
  - 原始全量上下文為 **26k tokens** (來源: Mem0.pdf, Section 4.5/Page 14)。
  - Zep 記憶圖譜膨脹超過 **600k tokens**（約為原始對話之 23 倍），嚴重浪費空間與 token (來源: Mem0.pdf, Section 4.5/Page 14)。
- **圖譜構建耗時（Construction Latency）**：
  - Mem0^g 構圖在最差情況下可在 **1 分鐘內** 完成，可即時用於後續查詢 (來源: Mem0.pdf, Section 4.5/Page 14)。
  - Zep 由於依賴多輪非同步 LLM 後台呼叫，需延遲 **數小時** 後重新搜尋才能正確返回結果，不具備生產即時性 (來源: Mem0.pdf, Section 4.5/Page 14)。

---

## 優缺點與 Trade-offs

### 優點
1. **兼顧推論精度與生產級低延遲**：
   - 相較 Full-Context，在僅微幅讓步品質（約 4~6% J 分數差距）的前提下，節省逾 90% 的 Context Token，並將 p95 延遲由 17 秒壓低至 1.44 秒 (來源: Mem0.pdf, Abstract/Page 1, Section 4.3/Page 13)。
2. **高度結構化且精簡的狀態操作**：
   - 藉由 ADD、UPDATE、DELETE、NOOP 四種語意操作，有效解決對話中陳舊與矛盾資訊的清理問題，避免上下文無限膨脹 (來源: Mem0.pdf, Section 2.1/Page 4, Appendix B/Page 21)。
3. **時間推理能力卓越（Mem0^g）**：
   - 透過有向圖的時間標籤與「標記失效」機制，Mem0^g 在 Temporal 任務上顯著擊敗所有基準系統 (來源: Mem0.pdf, Section 4.1/Page 10)。

### 缺點與工程 Trade-offs
1. **多跳推理中的圖譜導航開銷**：
   - 實驗顯示 Mem0^g 在 Multi-Hop 上不如純文本的 Mem0（J 分數 47.19 vs. 51.15），顯示圖譜在更複雜的多步推理中可能引入過度關聯或冗餘節點干擾 (來源: Mem0.pdf, Section 4.1/Page 10, Section 4.2/Page 10-11)。
2. **圖譜構建與儲存翻倍**：
   - 引入 Mem0^g 需額外維護 Neo4j 資料庫，儲存 Token 翻倍（7k $\to$ 14k tokens），且搜尋延遲增加約 3 倍（0.148s $\to$ 0.476s）(來源: Mem0.pdf, Section 4.4-4.5/Page 11, 14)。
3. **依賴 LLM Function Calling 的穩定性**：
   - 寫入與衝突裁決高度仰賴 `gpt-4o-mini` 的推理能力，若模型誤判可能導致不當覆寫或漏刪矛盾記憶 (來源: Mem0.pdf, Section 2.1/Page 4)。

---

## 相關核心條目
- [[dynamic_memory_operations]]：ADD / UPDATE / DELETE / NOOP 四態記憶生命週期管理。
- [[graph_memory_representations]]：有向標記圖、實體子圖與語意三元組雙路檢索機制。
- [[memory_latency_cost_tradeoffs]]：全上下文、RAG、Dense Memory 與 Graph Memory 的延遲-成本權衡矩陣。
- [[architecture_comparison]]：跨架構橫向對比表（傳統 Context Window vs. Mem0 vs. 後續架構）。
