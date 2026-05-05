# Code Quality

These principles are non-negotiable. Evaluate every code change against them before implementation.

## No Dead Code

- Actively remove unused functions, classes, imports, variables, and commented-out code
- Never leave dead code "just in case" — version control is the safety net
- When removing a feature or refactoring, delete all traces: implementations, re-exports, stubs, `# removed` comments
- If a model, function, or variable has zero references, it gets deleted — not commented out, not left with a TODO

## Root-Cause Fixes Only

- Every fix must address the root cause, not the symptom
- Before implementing any fix, ask: **"Is this a workaround or a proper solution?"**
- Reject fixes that: suppress warnings, silence errors with try/except, add flags to skip broken paths, or wrap problems rather than solving them
- If the first solution that comes to mind is a patch — stop, investigate deeper, find the architectural fix

## No Shortcuts

- Follow solid architecture even when the proper fix requires more work
- If the right solution means removing duplicate code, refactoring imports, or restructuring modules — do that work
- Quick hacks compound into unmaintainable systems; always choose the clean path
- Don't add compatibility shims, feature flags for dead paths, or defensive code for impossible states

## Proactive Evaluation

- Before writing any fix, explicitly evaluate: "Is this a bandaid?"
- If yes, reject it and find the proper fix before writing code
- When proposing solutions, present only proper fixes — never offer a "quick workaround" as an option
- If a proper fix is significantly more complex, explain why it's worth the effort — don't default to the easy path

## No Over-Engineering

- Only make changes that are directly requested or clearly necessary
- Don't add features, refactor surrounding code, or make "improvements" beyond what was asked
- Don't add error handling for scenarios that can't happen, or validation for internal-only code paths
- Don't create helpers, utilities, or abstractions for one-time operations
- Three similar lines of code is better than a premature abstraction
- The right amount of complexity is the minimum needed for the current task

---

# <project> 补充

> 以下条款是 <project> 反复强调的硬规则，对外部抄来的原则做项目级补充。

## 复杂度反向 = 方向反了

修同一问题：50 行 → 100 行 → 200 行，第 4 次该是 30 行。
加 try/except / 适配层 / 缓存往往是创可贴，删比加便宜。

## 改实现不改测试

测试不过改实现，除非证明测试本身错。

## LLM 不听话先查 prompt

不加正则兜底、不加重试次数。兜底只会掩盖根因，让错误装死。

## 删比加便宜

在加 try/except / 适配层 / 缓存吗？先删掉看真因。
50 → 100 → 200 行 = 方向反了，第 4 次该是 30 行。

## 3 次失败硬规则（用户最在意，过去执行不到位）

跟现有 retry_guard hook 协作覆盖所有失败场景：

- hook 管 Bash 工具失败计数 + 强制 block 第 4 次
- 本规则管所有非 Bash 失败：改 prompt 后输出仍不对、改代码后跑测试还失败、
  改配置后行为没变、修同一 bug 多次未解决——任何动作没解决问题都算 1 次失败

**3 次失败 → 立即停下，禁止第 4 次试**

停下后必须执行下列至少一个动作才能再试：

1. **WebSearch 搜新关键词**（之前没搜过的，且要报告搜索结果）
2. **让 plan-eng-review skill 介入讨论**（让外部视角审一遍）
3. **升级到设计窗口三方讨论**

未执行 1/2/3 任一就直接试第 4 次 = 严重违规。

**本质相同的尝试不算新尝试**：改变量名 / 调顺序 / 换错误处理 / 调 prompt 措辞
不算"本质不同"——仍算同一次重复尝试，计入失败次数。

## 30 分钟时间锁（次数+时间，先到先触发）

同一问题持续修了 30 分钟还没解决 → 也强制停，走 1/2/3 流程。
不要等"我觉得快好了"——机械时间锁，"是否卡住" Claude 自评不可信。

这条是 <project> 反复强调的硬规则，过去多次违反。本次必须严格执行。

---

> 来源：smartwhale8/claude-playbook (MIT License)，末尾追加 <project> 项目硬规则。
