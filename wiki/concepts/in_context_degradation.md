# 上下文長度退化模式（In-Context Degradation）

## 概念定義
上下文長度退化模式（In-Context Degradation）描述 LLM 在推論時，隨著輸入 token 總量持續增長，模型在注意力分配、多步驟推理及資訊檢索能力上呈現的非線性性能衰減行為 (來源: Context_Is_What_You_Need.pdf, Section 1/Page 1-2, Section 2.1/Page 3)。

---

## 四類任務的退化差異光譜
論文定義了四類評測任務，揭示了任務複雜度與退化速度的直接關聯 (來源: Context_Is_What_You_Need.pdf, Section 3.1.2/Page 5-6, Appendix A1/Page 14)：

1. **單一實體檢索（Needle-in-a-Haystack）**：
   - 特徵：從大語料中定位單一鍵值（Key-Value）。
   - 退化特性：模型表現最佳、容忍長度最高；但當 token 數累積至數萬時，各模型準確率依然顯著下降，無一能抵達標稱 MCW (來源: Context_Is_What_You_Need.pdf, Section 3.1.2/Page 5, Figure 5/Page 8, Figure A3.1/Page 15)。
2. **多目標檢索與求和（Needles-in-a-Haystack）**：
   - 特徵：多個分佈於各處的實體檢索並進行數值累加。
   - 退化特性：頂尖模型與後段模型分化劇烈。優秀推理模型會主動採取「先過濾至精簡清單、再進行求和」的內部推理鏈 (來源: Context_Is_What_You_Need.pdf, Section 3.1.2/Page 6, Figure A3.2/Page 15)。
3. **全域聚合統計（Summarization）**：
   - 特徵：對上下文中所有實體進行完全累計。
   - 退化特性：表現普遍劣於多目標檢索（Needles）。原因在於全域統計無法透過前置過濾機制來壓縮上下文長度，導致最後一步加總時仍需背負完整長文本的注意力開銷 (來源: Context_Is_What_You_Need.pdf, Section 3.1.2/Page 6, Figure A3.4/Page 16)。
4. **複合排序檢索（Find and Sort）**：
   - 特徵：條件過濾、字母序排序與多值字串串接。
   - 退化特性：衰退最為迅猛。在 100 至 500 tokens 區間內，即有大量模型準確度跌破 50%，於 1,000 至 1,500 tokens 區間幾乎全線歸零 (來源: Context_Is_What_You_Need.pdf, Section 3.1.2/Page 6, Figure A3.3/Page 16)。

---

## 關鍵退化臨界點數據
- **100 tokens**：少數模型即開始出現錯誤 (來源: Context_Is_What_You_Need.pdf, Abstract/Page 1)。
- **500 tokens**：如 GPT-5 等頂尖模型維持 0 幻覺率；但開源或弱推理模型在排序與統計任務上已出現顯著下滑 (來源: Context_Is_What_You_Need.pdf, Section 6.2/Page 9, Section 7.3/Page 10)。
- **1,000 tokens**：絕大多數模型在複雜任務（非單針檢索）中的正確率跌落至 50% 以下 (來源: Context_Is_What_You_Need.pdf, Abstract/Page 1, Figure 5/Page 8)。
- **2,000 tokens**：部分模型幻覺率飆升至 99%，推論準確度趨近於 0% (來源: Context_Is_What_You_Need.pdf, Section 7.3/Page 10)。

---

## 相關引用與頁面
- 出處論文：[[context_is_what_you_need]]
- 理論框架：[[mecw_vs_mcw]]
- 系統性後果：[[cascading_failures]]
- 多系統並陳：[[architecture_comparison]]
