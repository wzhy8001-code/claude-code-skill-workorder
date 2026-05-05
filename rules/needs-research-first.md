# 先搜后做（needs-research-first）

> 想做新东西之前，先搜。这不是程序员美德，是工程纪律。

## 三问框架

```
想做一件新东西
    ↓ 先搜（2-3 个英文关键词）
    ├─ 有成熟方案 → 抄
    └─ 搜不到 → 三问
        ├─ Q1: 真的没人做过吗？换关键词再搜
        │       （中→英、技术名→现象描述、产品→协议）
        ├─ Q2: 别人没做是「难」还是「没价值」？
        │    - 难  → 你也做不到，放弃
        │    - 没价值 → 重新评估需求是否合理
        └─ Q3: 你现有能力/资源下，做坏了能承受吗？
             - 不能 → 放弃
             - 能   → 真造轮子（抄能抄的 + 写不能抄的）
```

**关键判断**：用户的启发式 ——「没人做过的事，大部分情况是做不到」。
不要对抗这条经验法则。

## 跟现有 `feasibility_check.sh` hook 的关系

本 rule 是 hook 的**文档化版本**，两者必须并存：

- hook 在每次 prompt 提交时**强制提醒搜**（防止"忘了搜"）
- rule 在 Claude 上下文里**说明为什么搜 + 怎么用搜索结果**（防止"搜了但不会用"）

不要因为有了本 rule 就以为可以删 hook，反之亦然。
用户已多次强调搜索功能"必须有，只能加强不能削弱"。

## 工业先例（这不是用户独家偏执）

- **Joel Spolsky《Things You Should Never Do, Part I》**
  反对从零重写——"the single worst strategic mistake that any software company can make"。
  现成代码的价值远超你以为的"看起来很乱"。
  https://www.joelonsoftware.com/2000/04/06/things-you-should-never-do-part-i/

- **AWS Bezos「Type 1 / Type 2 决策」**（即"门和窗户"决策）
  不可逆决策（Type 1）必须慎重；可逆决策（Type 2）快试。
  造轮子大多是 Type 1（沉没成本难回收），先搜避免它。

- **Lean Startup 的 Build vs Buy 决策树**
  默认 Buy（含开源），只有当核心竞争优势才 Build。
  <project> 的核心是"内容创作 pipeline"，不是"造 hook 引擎"——造 hook 引擎要先搜。

## <project> 实际案例

**今天讨论的"工程标准化 skill+hook 系统"**，正是通过三问后决定造轮子的真实例子：

- Q1（搜了吗）：搜了 Anthropic Skills 文档、smartwhale8/claude-playbook、ChrisWiles/claude-code-showcase
- Q2（难还是没价值）：业界有 skill 触发机制成熟方案（UserPromptSubmit hook + 关键词），不难但需要本地化
- Q3（能承受吗）：能。bash + jq + python3 都已具备，最坏情况删 hook 重来

→ 决定造，但**最大限度抄**（engineering-principles.md / code-quality.md 抄 smartwhale8 MIT 部分；
skill-eval.sh 抄自家 feasibility_check.sh 模板）。
真正"自写"的只有 user-context.md 和 3 个 verify-* hook——因为这些是 <project> 私有规则，外部找不到。

## 何时跳过本规则

- 项目内部决策（路径命名、目录组织）
- bug 修复（先看堆栈，不是先搜"为什么会有这个 bug"）
- 文档格式调整
- 参数调优

**核心**：内置知识可能旧版本、似是而非、无出处。**有出处的资料 > 凭直觉推导**。
被问"查了吗"答不出 = 工作没做到位。
