# 测试用例与 Badcase 记录

## 测试用例 1：正常询盘

**输入**：

> Dear Supplier, We are interested in ST-CUP-001, quantity 2000 pcs, delivery to Los Angeles port, lead time 30 days. Please quote FOB price.

**输出**：正确生成英文报价单，包含产品、数量、单价、总价、港口、交期、有效期。

**是否通过**：✅

---

## 测试用例 2：型号不存在

**输入**：

> Dear Supplier, We are interested in ST-TABLE-999, quantity 100 pcs, delivery to New York port.

**输出**：模型礼貌说明无法报价，请客户确认型号。

**是否通过**：✅（修复后）

**修复过程**：

- 第一次测试时，模型没有正确判断 error 字段，反而生成了一份带 0 价格的草稿。
- 定位到问题：LLM 节点没有引用 error 变量。
- 修复：在 Prompt 里绑定 error 变量，并明确“error 不为空时不生成报价单”。

---

## 测试用例 3：数量低于起订量

**输入**：

> Dear Supplier, We want to order ST-CUP-001, quantity 100 pcs, delivery to Los Angeles port, lead time 15 days.

**输出**：模型礼貌说明起订量要求，建议客户调整数量。

**是否通过**：✅

**修复过程**：

- 第一次测试时，模型输出里出现了未替换的占位符 `{{error}}`。
- 定位到问题：LLM 节点的 Prompt 里没有正确绑定 error 变量。
- 修复：重新绑定变量后，模型能正确识别错误信息并礼貌说明。

---

## 测试用例 4：缺少交期

**输入**：

> Dear Supplier, We are interested in ST-CUP-002, quantity 800 pcs, delivery to Rotterdam port. Please quote CIF price.

**输出**：生成报价单，交期写 "To be advised"，没有停下来追问。

**是否通过**：✅（修复后）

**修复过程**：

- 第一次测试时，模型主动停下来询问交期，没有直接生成报价单。
- 定位到问题：Prompt 里没有说明“交期为空时该怎么处理”。
- 修复：加了一句“如果交期为空，请在报价单中写 To be advised，不要停下来询问”。

---

## 测试结论

| 测试用例    | 输入               | 预期结果                    | 实际结果 | 是否通过 |
| ------- | ---------------- | ----------------------- | ---- | ---- |
| 正常询盘    | ST-CUP-001, 2000 | 生成报价单                   | 正确生成 | ✅    |
| 型号不存在   | ST-TABLE-999     | 礼貌说明无法报价                | 正确说明 | ✅    |
| 数量低于起订量 | ST-CUP-001, 100  | 礼貌说明无法报价                | 正确说明 | ✅    |
| 缺少交期    | ST-CUP-002, 800  | 生成报价单，交期写 To be advised | 正确生成 | ✅    |

**4 个测试用例全部通过。**

## 迭代心得

1. **Prompt 要明确异常处理**：不仅要告诉模型做什么，还要告诉它遇到异常时该怎么处理。
2. **变量引用要准确**：LLM 节点里必须正确引用前序节点的输出，否则会出现未替换的占位符。
3. **AI 与规则分工要清晰**：信息提取和文案生成交给 AI，价格核算和异常判断交给代码，避免 AI 编数字。
