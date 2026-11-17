# 記憶架構多維度並陳對比表

## 架構對比矩陣

本表橫向對比五大 LLM 記憶管理架構的核心機制、生命週期與工程權衡。所有系統之實證數據與機制觀點**全部保留並陳對照**，不偏廢或刪除任一方觀點。

| 維度 / 架構 | 傳統 Context Window（In-Context / Context Stuffing） | Mem0 / Mem0^g | A-Mem | ZEP (Graphiti) | MemMachine |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **文獻出處** | [[context_is_what_you_need]] (來源: Context_Is_What_You_Need.pdf) | [[mem0]] (來源: Mem0.pdf) | [[a_mem]] (來源: A-Mem.pdf) | [[zep]] (來源: ZEP.pdf) | [[mem_machine]] (來源: MemMachine.pdf) |
| **記憶載體與儲存結構** | 推論期上下文（KV-Cache / Prompt 文本），無外部儲存 (來源: Context_Is_What_You_Need.pdf, Section 1/Page 2, Section 3.2/Page 6) | **外部專屬資料庫**：<br/>- Mem0: 稠密向量資料庫 (Dense Vector DB)<br/>- Mem0^g: Neo4j 圖資料庫（預定義實體節點與關係三元組）(來源: Mem0.pdf, Section 2.1-2.2/Page 4-6) | **卡片盒原子筆記（Zettelkasten Notes）**：<br/>免預定義 Schema；以七元組 $(c_i, t_i, K_i, G_i, X_i, e_i, L_i)$ 儲存，組織為多重虛擬盒子（Virtual Boxes）(來源: A-Mem.pdf, Section 3.1/Page 4, Figure 2/Page 3) | **三層階層式知識圖譜**：<br/>$\mathcal{G} = (\mathcal{N}, \mathcal{E}, \phi)$，劃分為情節子圖 $\mathcal{G}_e$（原始非損訊息）、語意實體子圖 $\mathcal{G}_s$（實體與超邊事實）、社群子圖 $\mathcal{G}_c$（聚類抽象摘要）(來源: ZEP.pdf, Section 2/Page 2) | **真實情節兩層儲存 + 畫像庫**：<br/>- 短期記憶（STM）：即時滑動視窗 + 會話摘要<br/>- 長期記憶（LTM）：原始情節非損儲存 + NLTK 句子級索引 (pgvector + Neo4j)<br/>- Profile Memory：SQL 儲存結構化偏好 (來源: MemMachine.pdf, Section 4.1-4.7/Page 4-6) |
| **寫入與萃取機制（Extraction）** | 靜態拼接（Prompt Concatenation）；推論時單次傳入全部內容 (來源: Context_Is_What_You_Need.pdf, Section 3.2/Page 6) | **雙階段增量萃取**：<br/>非同步全局摘要 $S$ + 近期視窗（$m=10$）+ 當前輪次 $(m_{t-1}, m_t)$，由 LLM 萃取候選事實 $\Omega$ (來源: Mem0.pdf, Section 2.1/Page 3-4) | **多屬性原子生成**：<br/>LLM 依 Prompt $P_{s1}$ 自主提煉關鍵詞 $K_i$、分類標籤 $G_i$ 與上下文描述 $X_i$，並串接計算嵌入向量 $e_i$ (來源: A-Mem.pdf, Section 3.1/Page 4) | **對話情節脈絡抽取**：<br/>取當前訊息與前 $n=4$ 則訊息，自動提取發話者實體，引入 Reflexion 反思機制校驗，以預定義 Cypher 模板寫入 (來源: ZEP.pdf, Section 2.2.1/Page 3) | **最小化 LLM 依賴之原始保存**：<br/>無常態性逐則訊息 LLM 抽取；原始情節直接切為句子並繼承母情節元數據，僅在 STM 溢出與畫像提取時調用 LLM (來源: MemMachine.pdf, Section 1.1/Page 2, Section 6.1/Page 9) |
| **更新與整合機制（Consolidation）** | 無動態更新機制（Stateless）；無狀態覆寫，外部整合機制標註為 `[資料不足，待補充]` (來源: Context_Is_What_You_Need.pdf, Section 3.1-3.2/Page 5-6) | **前向工具調用四態操作**：<br/>- 向量檢索 Top-$s$ 舊記憶，由 LLM 工具調用決定 ADD / UPDATE / DELETE / NOOP (來源: Mem0.pdf, Section 2.1/Page 4, Appendix B/Page 21)<br/>- Mem0^g 衝突採標記失效（Invalidate）(來源: Mem0.pdf, Section 2.2/Page 6) | **雙向演化與連結機制**：<br/>- 連結生成（LG）：向量篩選 Top-$k$ 近鄰 + LLM 生成關聯 $L_i$<br/>- 記憶演化（ME）：新筆記反向刺激近鄰舊記憶，重寫其 $X_j, K_j, G_j$（強化/合併/修剪）(來源: A-Mem.pdf, Section 3.2-3.3/Page 4-5) | **雙時間軸邊失效與動態標籤傳播**：<br/>- 雙時間軸（$T$ 與 $T'$）標記四時間戳記 ($t_{\text{valid}}, t_{\text{invalid}}, t'_{\text{created}}, t'_{\text{expired}}$)<br/>- 衝突邊將舊邊 $t_{\text{invalid}}$ 設為新邊 $t_{\text{valid}}$<br/>- 社群節點採單步標籤傳播動態歸併 (來源: ZEP.pdf, Section 2.2.3/Page 3, Section 2.3/Page 4) | **畫像 SQL 覆寫 + 情節時序索引**：<br/>- 個人畫像（Profile）支援 SQL 級別即時覆寫更新<br/>- 情節記憶不進行破壞性改寫，依賴時間戳記篩選與脈絡化聚類保持時序正確 (來源: MemMachine.pdf, Section 3.4/Page 4, Section 4.7/Page 6) |
| **檢索機制（Retrieval）** | 全注意力矩陣計算（Full Self-Attention），無外部索引檢索 (來源: Context_Is_What_You_Need.pdf, Section 1/Page 1-2, Section 2.1/Page 3) | **多路/雙路檢索**：<br/>- Mem0: 稠密向量檢索 (來源: Mem0.pdf, Section 2.1/Page 4)<br/>- Mem0^g: 實體中心子圖遍歷 + 語意三元組匹配 (來源: Mem0.pdf, Section 2.2/Page 6) | **向量檢索 + 虛擬盒子連帶存取**：<br/>查詢向量餘弦比對檢索 Top-$k$ 筆記；命中筆記所在 Box 內之相連記憶可連帶自動存取 (來源: A-Mem.pdf, Section 3.4/Page 5, Figure 2/Page 3) | **三階段檢索管線 $f = \chi(\rho(\phi))$**：<br/>- 搜尋 $\phi$：語意相似 ($\phi_{\text{cos}}$) + BM25 全文 ($\phi_{\text{bm25}}$) + 情節種子之圖 BFS ($\phi_{\text{bfs}}$)<br/>- 重排 $\rho$：RRF / MMR / 提及頻率 / 圖拓撲距離 / Cross-Encoder<br/>- 組裝 $\chi$：格式化 XML 帶時間區間之 `<FACTS>` 與 `<ENTITIES>` (來源: ZEP.pdf, Section 3/Page 4-5) | **脈絡化擴展 + 檢索代理人（Retrieval Agent）**：<br/>- 句子向量命中核心情節（Nucleus），前後擴展（前1後2）形成聚類並進行 Cross-Encoder 重排<br/>- ToolSelectAgent 路由至 ChainOfQuery（多跳迭代）、SplitQuery（並行拆解）或直接檢索<br/>- 多查詢串接重排 (來源: MemMachine.pdf, Section 4.6/Page 6, Section 5/Page 6-7) |
| **真實情節保留度（Ground Truth Preserved）** | 完全保留（全量原始文本）(來源: MemMachine.pdf, Table 16/Page 16) | 部分保留（依賴事實抽取）(來源: MemMachine.pdf, Table 16/Page 16) | 部分保留（依賴原子筆記與描述）(來源: A-Mem.pdf, Section 3.1/Page 4) | 部分保留（情節子圖保留原文，但主推論多依賴抽取事實與摘要）(來源: MemMachine.pdf, Table 16/Page 16) | **完全保留（Ground-Truth-Preserved）**：原始對話原樣儲存，無任何機率性抽取的失真漂移 (來源: MemMachine.pdf, Abstract/Page 1, Table 16/Page 16) |
| **每則訊息 LLM 調用量（LLM Calls per Msg）** | 無（0 次）(來源: MemMachine.pdf, Table 16/Page 16) | 高（High，每則訊息皆需抽取與狀態裁決）(來源: MemMachine.pdf, Table 16/Page 16) | 高（每則筆記需生成 $K, G, X$ 及近鄰演化）(來源: A-Mem.pdf, Section 4.3/Page 7) | 中度（Moderate，實體與邊抽取、消歧、社群維護）(來源: MemMachine.pdf, Table 16/Page 16) | **極低（Low）**：常態寫入 0 調用，僅 STM 溢出與畫像更新時調用 (來源: MemMachine.pdf, Section 6.1/Page 9, Table 16/Page 16) |
| **Token 消耗與成本表現** | 受限於 MECW；長上下文成本超線性攀升 (來源: Context_Is_What_You_Need.pdf, Section 7.3/Page 10) | 長期儲存 7k~14k tokens；推論傳入 ~1,764 tokens 事實，節省 90% 以上 Context Token (來源: Mem0.pdf, Table 2/Page 11, Section 4.5/Page 14) | 推論檢索 1,216~2,520 tokens；單次記憶操作約 1,200 tokens（成本 <$0.0003）(來源: A-Mem.pdf, Table 1/Page 6, Section 4.3/Page 7) | 檢索傳入 1.6k~3.9k tokens；但在 LOCOMO 上圖譜全量快取儲存膨脹逾 600k tokens (來源: ZEP.pdf, Table 2/Page 7; Mem0.pdf, Section 4.5/Page 14) | **極致節省**：<br/>在 LoCoMo 上較 Mem0 **節省約 78%~80% 輸入 Token**（4.20M vs 19.21M tokens）(來源: MemMachine.pdf, Table 8/Page 10) |
| **延遲表現與響應速度** | 搜尋延遲 0s；端到端總延遲極高（p50=9.87s, p95=17.117s；115k tokens 下達 28.9s~31.3s）(來源: Mem0.pdf, Table 2/Page 11; ZEP.pdf, Table 2/Page 7) | **極致低延遲**：<br/>搜尋 p50=0.148s, p95=0.200s；總延遲 p50=0.708s, p95=1.440s (來源: Mem0.pdf, Table 2/Page 11) | 100 萬筆筆記檢索時間僅 $3.70 \mu\text{s}$；寫入耗時 GPT-4o-mini 為 5.4s，Llama-3.2-1B 為 1.1s (來源: A-Mem.pdf, Section 4.3/Page 7, Table 4/Page 8) | 超長文本響應顯著加速（115k tokens 下總延遲由 28.9s 降至 **2.58s**）(來源: ZEP.pdf, Table 2/Page 7) | 寫入速度提升 ~75%，搜尋速度提升 ~75% (來源: MemMachine.pdf, Section 8.3/Page 11) |
| **LoCoMo 評測成績** | LLM-as-a-Judge J=72.90% (來源: Mem0.pdf, Table 2/Page 11)；純模型 baseline: 0.5290 (來源: MemMachine.pdf, Table 11/Page 12) | LOCOMO J=66.88% (來源: Mem0.pdf, Table 2/Page 11)；MemMachine 重測: 0.6688 (來源: MemMachine.pdf, Table 11/Page 12) | 原文 F1=27.02 (ROUGE-L=44.27) (來源: A-Mem.pdf, Table 1/Page 6)；*(Mem0 覆測 J=48.38%)* (來源: Mem0.pdf, Table 2/Page 11) | 原文 DMR=98.2% (來源: ZEP.pdf, Table 1/Page 6)；Mem0 覆測 LOCOMO J=65.99% (來源: Mem0.pdf, Table 2/Page 11)；MemMachine 重測: 0.7514 (來源: MemMachine.pdf, Table 11/Page 12) | **全面領先（SOTA）**：<br/>- gpt-4o-mini: **0.8747**（Single-hop: 0.9465, Multi-hop: 0.8759）(來源: MemMachine.pdf, Table 11/Page 12)<br/>- gpt-4.1-mini: **0.9169** (來源: MemMachine.pdf, Table 10/Page 12) |
| **LongMemEval（115k）評測成績** | Full-Context 得分 55.4%~60.2% (來源: ZEP.pdf, Table 2/Page 7) | [未於該基準公開完整消融] | [未於該基準公開完整消融] | Zep 得分 **71.2%**（相對提升 +18.5%）(來源: ZEP.pdf, Table 2/Page 7) | **全面刷新頂尖紀錄**：<br/>- 消融最佳（C15, GPT-5-mini, $k=100$）：**93.0%**<br/>- Pareto 最優（C12, GPT-5-mini, $k=20$）：**92.2%** (僅耗 2.58M Token) (來源: MemMachine.pdf, Table 1/Page 1, Table 12/Page 13) |

---

## 關鍵機制差異與觀點並陳

### 1. 核心設計哲學張力：激進壓縮（Compression）vs. 真實情節保留（Preservation）
- **激進壓縮觀點（Mem0, Zep, Mastra）**：
  主張長程對話必須在寫入期提煉為緊湊的語意事實、三元組或高階摘要，將上下文窗口限制在最小尺寸（例如 Mem0 的 1,764 tokens），以實現極致的單次查詢延遲並降低干擾 (來源: Mem0.pdf, Section 1/Page 2, Section 4.3/Page 13)。
- **真實情節保留觀點（MemMachine）**：
  嚴厲指出「常態性 LLM 抽取」是成本浪費與事實漂移的罪魁禍首。機率性抽取不可避免地會丟失細節，且隨著多會話推演，抽取錯誤會產生滾雪球般的複合性偏差（Compounding Error）。MemMachine 主張保留原始情節非損儲存，只在檢索時透過「脈絡化擴展」動態重建上下文，既能達到 0.9169 的超高 LoCoMo 準確率，又能比 Mem0 節省 80% 的 Token (來源: MemMachine.pdf, Section 1/Page 2, Section 6.3/Page 9-10, Table 10/Page 12)。

### 2. 工程回報律：寫入端抽取（Ingestion）vs. 檢索端重排（Retrieval Stage Dominance）
- **傳統觀點（Zep, Mem0^g）**：
  投入大量工程精力於寫入期的圖譜構建、實體消歧、超邊抽取與社群劃分，認為高品質的記憶結構是推論正確的前提 (來源: ZEP.pdf, Section 2/Page 2-4; Mem0.pdf, Section 2.2/Page 5-6)。
- **MemMachine 實證發現（Retrieval Stage Dominates Accuracy）**：
  在 LongMemEvalS 嚴格的控制變因消融實驗中，檢索深度調整帶來 **+4.2%**、上下文斷行格式帶來 **+2.0%**、搜尋提示詞設計帶來 **+1.8%**、使用者偏差修正帶來 **+1.4%**；而寫入端的句子切塊僅帶來 **+0.8%**。數據證明：**只要底層完整保存了 Ground Truth，檢索端的調優效益遠遠主導了寫入端的複雜抽取** (來源: MemMachine.pdf, Section 9.1/Page 13, Table 13/Page 13)。

### 3. 多跳推理的解法衝突：預建圖譜走訪（Zep & Mem0^g）vs. 動態檢索代理人（MemMachine）
- **預建圖譜走訪路徑**：
  Zep 與 Mem0^g 透過實體關係三元組與圖拓撲走訪（如 BFS），在結構化節點中沿邊傳播以解決多跳實體關聯 (來源: ZEP.pdf, Section 3.1/Page 4-5; Mem0.pdf, Section 2.2/Page 6)。
- **動態檢索代理人路徑（Retrieval Agent）**：
  MemMachine 認為圖譜走訪必須預先構圖且容易損耗對話細節。MemMachine 提出 Tool Tree 路由架構，將多跳依賴問題指派給 **ChainOfQuery** 進行最多 3 輪的前向鏈迭代與實體改寫，並透過多查詢重排（Multi-Query Reranking）解決「遲滯綁定（Late Binding）」問題，在 HotpotQA hard 取得 93.2% 準確率，證明免構圖之動態檢索管線在多跳推理上同樣具備強大威力 (來源: MemMachine.pdf, Section 5/Page 6-8, Table 4/Page 8)。

### 4. 模型與提示詞之協同優化爭議（Model-Prompt Co-Optimization）
- **常規直覺**：一般認為參數量更大、推理能力更強的模型（如 GPT-5）在長記憶評測上必然優於輕量級模型（如 GPT-5-mini）。
- **MemMachine 實測反常現象**：在 LongMemEvalS 評測中，當搭配簡潔直接的 Edwin3 提示詞時，**GPT-5-mini 反超 GPT-5 達 2.6%**（0.922 vs. 0.896），且在深度檢索 $k=30$ 與 $k=50$ 下均維持領先。分析指出，小模型的直接指令遵循能力與精簡 Prompt 高度契合，而大模型內建的 CoT 推理在面對過度引導時反而產生反效果。此發現打破了盲目堆砌頂級模型的迷思 (來源: MemMachine.pdf, Section 8.4.6/Page 12, Section 9.2/Page 13)。

---

## 相關引用與頁面
- 基礎限制論文：[[context_is_what_you_need]]
- 記憶架構論文：[[mem0]]、[[a_mem]]、[[zep]]、[[mem_machine]]
- 核心技術概念：
  - [[mecw_vs_mcw]]（有效上下文窗口臨界）
  - [[ground_truth_preserving_architecture]]（真實情節非損保存）
  - [[contextualized_retrieval]]（脈絡化擴展與多查詢重排）
  - [[late_binding_and_retrieval_agents]]（遲滯綁定難題與檢索代理人）
  - [[dynamic_memory_operations]]（Mem0 四態生命週期操作）
  - [[graph_memory_representations]]（圖譜表徵與雙路檢索）
  - [[zettelkasten_agentic_memory]]（卡片盒原子筆記模型）
  - [[memory_evolution_and_linking]]（自主連結與歷史反向演化）
  - [[temporal_knowledge_graphs]]（雙時間軸模型與邊失效）
  - [[hierarchical_memory_subgraphs]]（三層子圖與動態標籤傳播）
  - [[multi_faceted_graph_retrieval]]（多面檢索與重排管線）
  - [[memory_latency_cost_tradeoffs]]（延遲成本準確率權衡）
  - [[cascading_failures]]（Agent 級聯失敗機制）
