# 最大有效上下文窗口（MECW）vs. 最大架構窗口（MCW）

## 概念定義
- **最大上下文窗口（Maximum Context Window, MCW）**：
  模型在架構設計或工程實作上所能容納的最高輸入 token 數量限制（例如宣稱的 128k、100 萬或 1,000 萬 tokens）(來源: Context_Is_What_You_Need.pdf, Section 1/Page 1)。此數值僅代表模型的硬性輸入容量上限，並不保證推論品質 (來源: Context_Is_What_You_Need.pdf, Section 1/Page 1-2)。
- **最大有效上下文窗口（Maximum Effective Context Window, MECW）**：
  針對特定任務類型，模型在輸出品質開始出現可度量的衰退前，所能處理的最大輸入 token 長度（"The longest span of token input, for a given problem type, for which incremental tokens degrade the model's output with a measurable effect"）(來源: Context_Is_What_You_Need.pdf, Section 1/Page 2, Appendix A2.1/Page 14)。

---

## 核心特徵與實證結論

### 1. 動態光譜而非靜態單一數值
傳統觀點常將 Context Window 視為一個固定的硬體參數；但實證表明，MECW 是一組隨任務複雜度動態浮動的光譜 (來源: Context_Is_What_You_Need.pdf, Section 1/Page 2)。相同模型在單一實體檢索任務中可耐受數萬 tokens，但在多實體聚合、排序或全量統計等任務中，MECW 可能迅速降至數百至一千 tokens (來源: Context_Is_What_You_Need.pdf, Section 3.1.2/Page 5-6, Figure 5/Page 8)。

### 2. 宣稱值與實測值的巨大鴻溝
在包含 11 款 Frontier 模型的全面測試中，所有受測模型的實際 MECW 皆遠低於宣稱的 MCW，差距幅度高達 >99% (來源: Context_Is_What_You_Need.pdf, Abstract/Page 1)。部分頂尖模型甚至在上下文僅達 100 tokens 時便發生錯誤，大多數模型在 1,000 tokens 時出現嚴重的準確度崩塌 (來源: Context_Is_What_You_Need.pdf, Abstract/Page 1)。

### 3. 認知科學視角：工作記憶 vs. 長期記憶
在認知科學比擬中，MECW 更近似於人類的「工作記憶」（Working Memory），負責當下有限容量的高強度邏輯運算；而長文本上下文窗口若被誤當作「長期記憶」（Long-term Memory）使用，將直接導致注意力發散與嚴重幻覺 (來源: Context_Is_What_You_Need.pdf, Section 1/Page 2)。

---

## 工程啟示與應用設計
1. **RAG 管道長度優化**：
   RAG 系統若無限制地填充檢索內容（Context Stuffing），超過 MECW 後會導致準確率自近 100% 暴跌至趨近 0%，幻覺率劇增至 99% (來源: Context_Is_What_You_Need.pdf, Section 6.1/Page 9, Section 7.3/Page 10)。RAG 檢索片段必須嚴格截斷在 MECW 安全門檻內 (來源: Context_Is_What_You_Need.pdf, Section 1/Page 2, Section 7.3/Page 10)。
2. **模型選擇的性價比反轉**：
   在 500 tokens 以內的短上下文範圍內，開源輕量模型（如 DeepSeek-R1）表現與昂貴商用模型無異；系統應依據任務之 MECW 精準選型，而非一味採用大模型或最大窗口 (來源: Context_Is_What_You_Need.pdf, Section 6.2/Page 9)。

---

## 相關引用與頁面
- 出處論文：[[context_is_what_you_need]]
- 衍生系統風險：[[cascading_failures]]
- 退化模式細節：[[in_context_degradation]]
- 多系統並陳：[[architecture_comparison]]
