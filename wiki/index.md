# LLM Memory Management 知識庫

歡迎查閱 LLM Memory Management 知識庫。本知識庫專門服務資工系與 AI 領域研究生，系統性梳理大語言模型記憶管理的核心挑戰、生命週期（Extraction, Consolidation, Retrieval）、架構演進與系統工程 Trade-offs。

---

## 推薦閱讀路徑

```mermaid
flowchart TD
    A["傳統 Context Window 限制<br/>(MCW vs. MECW)"] -->|啟發外部記憶需求| B["外部動態記憶架構"]
    A -->|長上下文過載| C["Agent 鏈路系統風險<br/>(Cascading Failures)"]
    
    B --> M0["生產級記憶系統<br/>(Mem0 / Mem0^g)"]
    B --> AM["卡片盒演化記憶系統<br/>(A-Mem)"]
    
    M0 --> D["四態操作生命週期<br/>(Dynamic Operations)"]
    M0 --> E["有向圖譜與雙路檢索<br/>(Graph Representations)"]
    
    AM --> ZK["卡片盒原子筆記模型<br/>(Zettelkasten Memory)"]
    AM --> ME["自主連結與反向演化<br/>(Linking & Evolution)"]
    
    D --> TR["多維度工程權衡<br/>(Latency-Cost-Accuracy)"]
    E --> TR
    ZK --> TR
    ME --> TR
    C --> TR
    
    TR --> COMP["多架構橫向對比矩陣<br/>(Architecture Comparison)"]
```

1. **第一步：理解純上下文窗口的物理與行為極限**
   - 閱讀 [[context_is_what_you_need]] 掌握模型宣稱窗口（MCW）與實際有效窗口（MECW）之巨大差距 (來源: Context_Is_What_You_Need.pdf, Abstract/Page 1)。
   - 研讀核心概念 [[mecw_vs_mcw]] 與 [[in_context_degradation]]，了解任務複雜度對注意力退化的非線性影響 (來源: Context_Is_What_You_Need.pdf, Section 1/Page 2, Section 3.1.2/Page 5-6)。
   - 查閱 [[cascading_failures]]，掌握多 Agent 鏈路在長上下文輸入下的級聯崩潰數學機制 (來源: Context_Is_What_You_Need.pdf, Appendix A2.2/Page 14, Section 7.3/Page 10)。
2. **第二步：研讀生產級外部動態記憶架構（Mem0 / Mem0^g）**
   - 閱讀 [[mem0]] 掌握生產級記憶架構如何透過萃取與更新兩階段管線實現跨 Session 長期記憶 (來源: Mem0.pdf, Section 2/Page 3-5)。
   - 深入概念條目 [[dynamic_memory_operations]]，學習 ADD / UPDATE / DELETE / NOOP 四態生命週期操作與語意衝突裁決 (來源: Mem0.pdf, Section 2.1/Page 4, Appendix B/Page 21)。
   - 深入概念條目 [[graph_memory_representations]]，學習有向標記圖建模、實體子圖與三元組雙路檢索，以及時序失效標記（Temporal Invalidation）(來源: Mem0.pdf, Section 2.2/Page 5-6)。
3. **第三步：研讀免綱要之自主演化記憶架構（A-Mem）**
   - 閱讀 [[a_mem]] 掌握如何借鑒卡片盒筆記法（Zettelkasten）打破圖資料庫預定義 Schema 的僵化限制 (來源: A-Mem.pdf, Section 1/Page 1-2, Section 3/Page 3-5)。
   - 深入概念條目 [[zettelkasten_agentic_memory]]，理解原子筆記之七元組結構與多重虛擬盒子（Virtual Boxes）機制 (來源: A-Mem.pdf, Section 3.1/Page 4, Figure 2/Page 3)。
   - 深入概念條目 [[memory_evolution_and_linking]]，掌握新經驗寫入如何反向促使歷史鄰近筆記進行屬性重寫與動態網絡演化 (來源: A-Mem.pdf, Section 3.2-3.3/Page 4-5, Table 3/Page 7)。
4. **第四步：系統工程評估與多架構橫向綜合對比**
   - 研讀 [[memory_latency_cost_tradeoffs]]，分析 Full-Context、RAG、Dense Memory、Graph Memory、演化筆記在延遲、Token 消耗與準確率間的 Trade-offs (來源: Mem0.pdf, Table 2/Page 11; A-Mem.pdf, Table 1/Page 6, Table 4/Page 8)。
   - 查閱 [[architecture_comparison]]，掌握各世代記憶管理機制之存儲載體、生命週期操作，以及跨團隊評測指標分歧之完整並陳對照。

---

## 知識庫全頁面清單

### 單篇文獻筆記（Papers）
- [[context_is_what_you_need]] — *Context Is What You Need: The Maximum Effective Context Window for Real World Limits of LLMs* (Norman Paulsen, 2025) (來源: Context_Is_What_You_Need.pdf, Page 1)
- [[mem0]] — *Mem0: Building Production-Ready AI Agents with Scalable Long-Term Memory* (Prateek Chhikara et al., 2025) (來源: Mem0.pdf, Page 1)
- [[a_mem]] — *A-Mem: Agentic Memory for LLM Agents* (Wujiang Xu et al., 2025) (來源: A-Mem.pdf, Page 1)

### 核心概念條目（Concepts）
- [[mecw_vs_mcw]] — 最大有效上下文窗口（MECW）與最大架構窗口（MCW）定義、光譜特性與實證落差 (來源: Context_Is_What_You_Need.pdf, Section 1/Page 1-2, Appendix A2.1/Page 14)
- [[cascading_failures]] — 多 Agent 鏈路中的級聯失敗效應與上下文過載分析 (來源: Context_Is_What_You_Need.pdf, Section 7.3/Page 10, Appendix A2.2/Page 14)
- [[in_context_degradation]] — 隨任務複雜度與 token 數量攀升的記憶崩塌模式 (來源: Context_Is_What_You_Need.pdf, Section 3.1.2/Page 5-6, Figure 5/Page 8)
- [[dynamic_memory_operations]] — ADD / UPDATE / DELETE / NOOP 四態記憶生命週期操作與衝突解決 (來源: Mem0.pdf, Section 2.1/Page 4, Appendix B/Page 21)
- [[graph_memory_representations]] — 有向標記圖表徵、雙路檢索與時序失效機制 (來源: Mem0.pdf, Section 2.2/Page 5-6)
- [[zettelkasten_agentic_memory]] — 卡片盒式原子筆記模型、七元組屬性與多重虛擬盒子 (來源: A-Mem.pdf, Section 3.1/Page 4, Figure 2/Page 3)
- [[memory_evolution_and_linking]] — 自主連結生成（LG）與反向歷史記憶演化（ME）機制 (來源: A-Mem.pdf, Section 3.2-3.3/Page 4-5, Table 3/Page 7)
- [[memory_latency_cost_tradeoffs]] — 記憶系統在品質、搜尋延遲、總延遲與 Token 儲存體積之權衡矩陣 (來源: Mem0.pdf, Table 2/Page 11; A-Mem.pdf, Table 4/Page 8)

### 跨架構綜合分析（Syntheses）
- [[architecture_comparison]] — 傳統 Context Window、Mem0/Mem0^g、A-Mem、ZEP 與 MemMachine 多維度並陳對比表

### 運作日誌（Log）
- [[log]] — 知識庫 Ingest 歷程與 Lint 檢查紀錄

---

## 原始文獻庫（Raw Sources）
- `raw/Context_Is_What_You_Need.pdf` — *Context Is What You Need: The Maximum Effective Context Window for Real World Limits of LLMs* (Norman Paulsen, 2025)
- `raw/Mem0.pdf` — *Mem0: Building Production-Ready AI Agents with Scalable Long-Term Memory* (Prateek Chhikara et al., 2025)
- `raw/A-Mem.pdf` — *A-Mem: Agentic Memory for LLM Agents* (Wujiang Xu et al., 2025)
