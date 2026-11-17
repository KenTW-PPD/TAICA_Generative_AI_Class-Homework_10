# 圖譜記憶表徵（Graph Memory Representations）

## 概念定義
圖譜記憶表徵（Graph Memory Representations）指將對話互動中萃取的知識以「有向標記知識圖譜」（Directed Labeled Graph）的形式進行結構化儲存與關聯建模的技術，旨在顯式捕捉實體間的複雜拓撲關係與時序演變 (來源: Mem0.pdf, Section 2.2/Page 5)。

---

## 圖譜結構模型：$G = (V, E, L)$
在 Mem0^g 中，記憶圖譜形式化定義為 $G = (V, E, L)$ (來源: Mem0.pdf, Section 2.2/Page 5)：
1. **實體節點 $V$**：
   - 包含三項核心要素：
     - **實體類型分類**：標註實體所屬語意類別（如 Person, Location, Event 等）(來源: Mem0.pdf, Section 2.2/Page 5)。
     - **語意嵌入向量 $e_v$**：表徵實體概念的 Dense Embedding (來源: Mem0.pdf, Section 2.2/Page 5)。
     - **時間戳記 $t_v$**：記錄該實體節點的建立時間詮釋資料 (來源: Mem0.pdf, Section 2.2/Page 5)。
2. **關係邊 $E$**：
   - 組織為三元組 $(v_s, r, v_d)$，其中 $v_s$ 為來源實體，$v_d$ 為目標實體，$r$ 為帶有標籤的語意關係（例如 `lives_in`, `prefers`, `owns`, `happened_on`）(來源: Mem0.pdf, Section 2.2/Page 5, 6)。
3. **語意標籤 $L$**：
   - 用於對節點與邊進行型態約束與語意定義 (來源: Mem0.pdf, Section 2.2/Page 5)。

---

## 記憶生命週期機制

### 1. 兩階段圖譜萃取
- **實體萃取器（Entity Extractor）**：分析輸入文本的語意重要性、獨特性與持久性，識別出值得長期儲存的離散實體 (來源: Mem0.pdf, Section 2.2/Page 5)。
- **關係生成器（Relationship Generator）**：依據對話語境與常識，推論實體間的顯式陳述與隱式關係，生成語意關係三元組 (來源: Mem0.pdf, Section 2.2/Page 6)。

### 2. 衝突處理：標記失效 vs. 物理刪除（Temporal Invalidation）
在圖譜更新時，面對衝突或陳舊資訊，Mem0^g 採用「標記失效」（Mark as Invalid）策略，而非直接從資料庫物理抹除舊關係 (來源: Mem0.pdf, Section 2.2/Page 6)：
- **時序推理優勢**：若直接刪除舊關係，系統將喪失歷史記憶狀態；保留帶時間戳記的過期關係，使模型能夠執行「A 於 2021 年住在西雅圖，2023 年搬遷至舊金山」等時序推演任務 (來源: Mem0.pdf, Section 2.2/Page 6, Section 4.1/Page 10)。
- 此機制使 Mem0^g 在 LOCOMO 基準的 Temporal 題型中奪得全場最高分（$J = 58.13\%$），顯著超越 OpenAI（$J = 21.71\%$）與 Zep（$J = 49.31\%$）(來源: Mem0.pdf, Section 4.1/Page 9-10)。

### 3. 雙路整合檢索機制（Dual Retrieval Strategy）
Mem0^g 整合兩類互補檢索路徑 (來源: Mem0.pdf, Section 2.2/Page 6)：
1. **實體中心子圖遍歷（Entity-Centric Subgraph）**：定位查詢關鍵詞對應的錨點節點，遍歷其出入度相鄰邊，抽取局部子圖，捕捉高度聚焦的實體上下文 (來源: Mem0.pdf, Section 2.2/Page 6)。
2. **語意三元組匹配（Semantic Triplet Matching）**：將查詢整體編碼為向量，與圖譜中三元組的文本向量進行細粒度語意比對，回傳高於門檻值的三元組清單 (來源: Mem0.pdf, Section 2.2/Page 6)。

---

## 工程特性與 Trade-offs

| 特性維度 | 密集語言記憶（Mem0） | 圖譜記憶（Mem0^g） | 全文快取圖譜（Zep） |
| :--- | :--- | :--- | :--- |
| **底層技術堆疊** | 向量資料庫（Dense Vector DB）(來源: Mem0.pdf, Section 2.1/Page 5) | Neo4j 圖資料庫 + 向量索引 (來源: Mem0.pdf, Section 2.2/Page 6) | 專用時間知識圖譜 (來源: Mem0.pdf, Section 3.3/Page 9) |
| **對話儲存預算** | ~7k tokens / 對話 (來源: Mem0.pdf, Section 4.5/Page 14) | ~14k tokens / 對話 (來源: Mem0.pdf, Section 4.5/Page 14) | >600k tokens / 對話（膨脹逾 20 倍）(來源: Mem0.pdf, Section 4.5/Page 14) |
| **構圖完成耗時** | 即時增量處理 (來源: Mem0.pdf, Section 2.1/Page 3) | < 1 分鐘（最差情況）(來源: Mem0.pdf, Section 4.5/Page 14) | 數小時（非同步後台延遲）(來源: Mem0.pdf, Section 4.5/Page 14) |
| **Temporal 表現 ($J$)** | 55.51% (來源: Mem0.pdf, Table 1/Page 9) | **58.13%（全場最佳）** (來源: Mem0.pdf, Table 1/Page 9) | 49.31% (來源: Mem0.pdf, Table 1/Page 9) |
| **Multi-Hop 表現 ($J$)** | **51.15%（全場最佳）** (來源: Mem0.pdf, Table 1/Page 9) | 47.19%（圖遍歷可能帶來雜訊）(來源: Mem0.pdf, Section 4.1/Page 10) | 41.35% (來源: Mem0.pdf, Table 1/Page 9) |

---

## 相關引用與頁面
- 出處論文：[[mem0]]
- 狀態轉移操作：[[dynamic_memory_operations]]
- 成本延遲權衡：[[memory_latency_cost_tradeoffs]]
- 跨架構綜合對照：[[architecture_comparison]]
