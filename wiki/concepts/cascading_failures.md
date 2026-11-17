# 級聯失敗（Cascading Failures）

## 概念定義
級聯失敗（Cascading Failures）指在由多個智慧代理人（Agents）組成的鏈路系統中，由於單一 Agent 僅具備平庸或次佳的成功率，錯誤隨著鏈路傳播逐級放大，導致整體多 Agent 系統最終高機率徹底失敗的現象 (來源: Context_Is_What_You_Need.pdf, Section 7.3/Page 10, Appendix A2.2/Page 14)。

---

## 數學模型與實例
- **失敗率遞增效應**：
  若一個系統由 3 個互相串聯的 Agent 組成，且各 Agent 獨立運作時的單點成功率為 70%（即 0.7），整體鏈路成功率將降為：
  $$0.7 \times 0.7 \times 0.7 = 34.3\%$$
  這意味著系統在超過 65% 的情況下會以失敗告終 (來源: Context_Is_What_You_Need.pdf, Appendix A2.2/Page 14)。
- 若鏈路長度進一步延伸或單一 Agent 因長上下文使準確率跌至 50% 以下，系統成功率將迅速趨近於 0% (來源: Context_Is_What_You_Need.pdf, Section 7.3/Page 10)。

---

## 誘發根源：長上下文與記憶過載
1. **盲目信賴超大上下文窗口**：
   開發者常誤以為模型支援百萬級別的 Context Window 便可在 Agent 鏈路中無節制地累積歷史對話與檢索文件 (來源: Context_Is_What_You_Need.pdf, Section 7.3/Page 10)。
2. **突破 MECW 造成單點準確度崩塌**：
   當輸入 token 數超過該任務的 [[mecw_vs_mcw]] 臨界值時，模型的幻覺率急遽升高（在 2,000 tokens 時可達 99%），使上游 Agent 傳遞受污染或錯誤的上下文給下游，直接引發雪崩式的連鎖崩潰 (來源: Context_Is_What_You_Need.pdf, Section 7.3/Page 10)。

---

## 緩解策略與架構解法
1. **範圍縮減與 Agent 裂變（Splitting Agents）**：
   將龐大複雜任務拆解為職責單一的小型 Sub-agents，並嚴格限制每個 Sub-agent 的 Context Window 於其 MECW 之內，從源頭確保單點推論準確率接近 100% (來源: Context_Is_What_You_Need.pdf, Section 7.1/Page 9)。
2. **上下文截斷與分段摘要**：
   不直接傳遞原始對話全文或巨量 RAG 檢索塊，而採用摘要、過濾或結構化記憶萃取，使每個節點的 Working Memory 保持在安全區間 (來源: Context_Is_What_You_Need.pdf, Section 1/Page 2, Section 7.1/Page 9)。

---

## 相關引用與頁面
- 出處論文：[[context_is_what_you_need]]
- 核心理論：[[mecw_vs_mcw]]
- 退化模式：[[in_context_degradation]]
- 多系統並陳：[[architecture_comparison]]
