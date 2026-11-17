# TAICA_生成式AI：文字與圖像生成的原理與實務_作業 ： 建立簡報網站
## 網頁連結：https://kentw-ppd.github.io/TAICA_Generative_AI_Class-Homework_10/slides_html/index.html

## 介紹：
本專案為生成式AI課程作業，使用`AGENTS.md`建立專案規則，並使用 Antigravity 完成專案。

### 主題：
大型語言模型(LLM)的記憶管理簡報

### 收錄內容
簡報一共收入五篇LLM記憶管理與記憶架構論文
- **Context Is What You Need**: The Maximum Effective Context Window for Real World Limits of LLMs
- **Mem0**: Building Production-Ready AI Agents with Scalable Long-Term Memory
- **A-Mem**: Agentic Memory for LLM Agents
- **ZEP**: A TEMPORAL KNOWLEDGE GRAPH ARCHITECTURE FOR AGENT MEMORY
- **MemMachine**: A Ground-Truth-Preserving Memory System for Personalized AI Agents

### 網頁功能
- 支援現代美學排版與響應式互動教學簡報。
- 具備鍵盤左右方向鍵切換導覽與視覺進度條。
- 完整呈現各主流記憶架構的原理、對比與工程取捨。

在 AI 協助產生程式碼後，再由本人進行測試。

## Vibe Coding 開發流程：
1. **規範定義 (AGENTS.md)：** 建立嚴格的專案規則與零幻覺來源追溯約束。
2. **文獻解析 (Ingest Loop)：** 逐篇萃取五篇論文的核心架構、痛點與機制，寫入 `wiki/papers/`。
3. **知識庫合成 (Synthesis)：** 於 `wiki/syntheses/` 進行跨系統並陳對比，確保客觀性。
4. **前端生成與驗證 (HTML Slides & Lint)：** 將 Wiki 知識庫轉化為互動式網頁，並執行斷鏈與孤島檢查。
5. **部署上線 (GitHub Pages)：** 透過 Git 版本控制推送到儲存庫並完成線上靜態部署。
   
## 專案結構：
``` text
TAICA_Generative_AI_Class-Homework_10/
├── wiki/
│   ├── index.md               
│   ├── papers/                  
│   ├── syntheses/
│   └── log.md                  
├── slides_html/
│   └── index.html       
└── README.md                   
```
## 使用方式：
  1. 直接點擊上方 GitHub Pages 連結 線上瀏覽互動簡報。
  2. 使用鍵盤的 左右方向鍵 (← / →) 或滑鼠點擊控制按鈕來翻閱簡報內容。
  3. 若需檢視原始研究筆記與對比分析，可直接參閱倉庫中的 wiki/ 目錄。
