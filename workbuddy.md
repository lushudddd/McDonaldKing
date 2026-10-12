# WorkBuddy 开发对话上下文（workbuddy.md）

> **用途**：本文件用于麦当劳程序员节创意开发大赛 WorkBuddy 联动活动奖励核验，证明本项目使用 WorkBuddy 开发。
>
> **提交前必读**：请将本文件占位部分替换为从 WorkBuddy 导出的**完整开发对话上下文**（对话中导出/另存为对话记录后粘贴到此处），保留本段说明上方的标题行。文件名 `workbuddy.md` 不可改动。

---

## 项目开发记录（WorkBuddy 会话摘要）

- **开发工具**：WorkBuddy（赛事官方合作伙伴）
- **开发日期**：2026-10-12 起
- **MCP 环境**：开发会话中已连接 mcd-mcp 连接器（麦当劳 MCP Server，`https://mcp.mcd.cn`），Skill 中引用的工具参数均来自会话内实际加载的 MCP 工具 Schema，非文档誊抄。

### 开发过程纪要

1. **需求分析**：通过 WorkBuddy 抓取赛事仓库 [mcd-developer-innovation-challenge](https://github.com/M-China/mcd-developer-innovation-challenge) 与 [mcd-mcp-server](https://github.com/M-China/mcd-mcp-server)，确认参赛必备文件（README.md / CONTEST_DECLARATION.md / MCP_INTEGRATION.md / 源代码 / workbuddy.md）与 Skill 开发条件。
2. **方向决策**：在"麦门省钱助手 / 智能套餐推荐 / 麦麦活动日历"三个候选方向中，确定以 **麦门省钱助手** 为项目方向，主打"领券 → 比价 → 最优价下单 → 积分变现"完整省钱链路。
3. **工具调研**：在 WorkBuddy 会话中加载麦当劳 MCP 的 `available-coupons`、`auto-bind-coupons`、`query-store-coupons`、`calculate-price`、`create-order`、`query-my-account`、`mall-points-products` 等 12 个工具的真实 Schema，据此编写 `mcd-saver/references/tools-guide.md` 工具速查表。
4. **框架搭建**：使用 WorkBuddy 的 skill-creator 规范初始化 Skill 骨架，定制 `mcd-saver/SKILL.md`（工作流决策树 + 四大工作流 + 省钱策略规则），并完成 README.md、MCP_INTEGRATION.md、CONTEST_DECLARATION.md（官方原文）等参赛文件。
5. **后续迭代**：在 WorkBuddy 中使用真实 MCP Token 联调各工作流（领券 / 比价下单 / 积分兑换），依据实际返回结果修正 Skill 指令与工具文档。

---

<!-- ⬇⬇⬇ 提交前，将 WorkBuddy 导出的完整对话上下文粘贴替换下方占位区 ⬇⬇⬇ -->

## 完整开发对话上下文

（待导出替换：请在 WorkBuddy 对话中导出开发过程的对话记录，粘贴至此处）

<!-- ⬆⬆⬆ 提交前替换完毕，勿改动文件名 ⬆⬆⬆ -->
