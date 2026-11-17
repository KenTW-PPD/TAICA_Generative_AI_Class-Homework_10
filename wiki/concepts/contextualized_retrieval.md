# 脈絡化擴展檢索（Contextualized Retrieval）

## 概念定義
脈絡化擴展檢索（Contextualized Retrieval）指在對話記憶系統中，為了解決對話輪次高度相互依賴與嵌入不相似問題，在向量搜尋命中核心情節（Nucleus Episode）後，自動抓取其在對話時序上的相鄰情節拼裝為「情節聚類（Episode Cluster）」，並進行二次重排與時序排序後再輸入推論模型的檢索技術 (來源: MemMachine.pdf, Abstract/Page 1, Section 4.6/Page 6)。

---

## 對話嵌入不相似性困境（The Embedding Dissimilarity Problem）

```mermaid
flowchart LR
    Turn1["Turn t-1 (前置限制)<br/>'我想找安靜且有戶外座位的餐廳'"] 
    --> Turn2["Turn t (核心命中 Nucleus)<br/>'那我推薦 Blue Hill Café'"]
    --> Turn3["Turn t+1 (後續追問)<br/>'那邊有插座可以用筆電嗎？'"]
    --> Turn4["Turn t+2 (補充說明)<br/>'戶外區沒有，室內吧檯才有'"]

    Q["用戶查詢: '推薦餐廳有什麼限制？'"] -.->|純向量搜尋易僅命中| Turn2
    Turn2 -.->|脈絡化擴展 (-1, +2)| Cluster["情節聚類: Turn t-1 至 Turn t+2<br/>完整還原約束與後續細節"]
```

- **文件 RAG vs. 對話記憶**：
  在傳統文件 RAG 中，每個文本區塊（Chunk）通常自成獨立語意段落；然而在人類多輪對話中，語句高度依賴上下文代詞與前置條件 (來源: MemMachine.pdf, Section 4.6/Page 6)。
- **資訊斷裂風險**：
  若使用者查詢「當時對餐廳推薦提出了什麼要求？」，向量搜尋往往僅精準比對到提及店名的句子（Nucleus），而遺漏了前一輪提出預算與素食偏好、或是後兩輪確認營業時間的發言 (來源: MemMachine.pdf, Section 4.6/Page 6)。

---

## 四步運作流程

MemMachine 設計了標準化的脈絡化檢索管線 (來源: MemMachine.pdf, Section 4.6/Page 6)：
1. **核心情節定位（Nucleus Episode Location）**：
   - 透過查詢向量與句子嵌入的相似度比對，定位語意最相關的句子，並溯源至其所屬之母情節 (來源: MemMachine.pdf, Section 4.5.1/Page 5, Section 4.6/Page 6)。
2. **前後文鄰域擴展（Neighboring Episode Expansion）**：
   - 自動提取核心情節**前 1 則、後 2 則**的相鄰情節，拼裝為一個兼具因果前因與後續反應的「情節聚類（Episode Cluster）」(來源: MemMachine.pdf, Section 4.6/Page 6)。
3. **聚類去重與跨編碼器重排（Cross-Encoder Reranking）**：
   - 消除不同聚類間重疊的情節，並採用 Cross-Encoder 或高階重排模型（如 AWS Cohere rerank-v3-5:0）對候選聚類進行整體語意精確評分 (來源: MemMachine.pdf, Section 4.6/Page 6, Section 7.3/Page 10)。
4. **時序排序與注入（Chronological Sorting & Injection）**：
   - 將篩選出的 Top-$k$ 個聚類依真實對話發生的時間戳記重新排序，以符合自然對話流的方式注入 Prompt (來源: MemMachine.pdf, Section 4.5.2/Page 5-6, Section 4.6/Page 6)。

---

## 多查詢重排機制（Multi-Query Reranking）
在複雜或多跳對話檢索中，系統會生成多個衍生子查詢或改寫題 (來源: MemMachine.pdf, Section 5.5/Page 7)：
- 最終重排器接收**所有檢索階段所使用查詢的串接字串**（包含原始查詢、改寫查詢與分解子查詢），而非僅使用原始問題 (來源: MemMachine.pdf, Section 5.5/Page 7)。
- **實質效益**：確保多跳鏈條中關鍵的中介事實（Intermediate Facts，其字面內容未包含於原始問題中）亦能在最終重排評分中名列前茅，徹底改善複雜對話的召回率 (來源: MemMachine.pdf, Section 5.5/Page 7)。

---

## 相關引用與頁面
- 出處論文：[[mem_machine]]
- 情節保留哲學：[[ground_truth_preserving_architecture]]
- 多跳路由：[[late_binding_and_retrieval_agents]]
- 綜合對比：[[architecture_comparison]]
