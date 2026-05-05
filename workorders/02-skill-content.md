# 工单：<project> 工程标准化 阶段 2（SKILL.md 实质内容）

> 派工时间：2026-05-03
> 派工人：设计窗口
> 接工人：搭建窗口
> 前置：阶段 1 已完成验收（11/11 Pass，e6eaa65+83268a2 已提交）

---

## 0. 背景与目标

### 阶段 1 复盘
- ~/.claude/{rules,hooks,skills/<project>-production-standards} 骨架到位
- 4 个 rule 文件加载就绪（但目前是"按需加载"，不在 SessionStart 自动注入）
- 4 个新 hook 跑通（skill-eval 触发 + 3 个 verify-* 校验）
- SKILL.md 是占位，**只有 frontmatter，没有 body**

### 阶段 2 范围
**填 SKILL.md 实质内容**。skill 被触发时，Claude 读 SKILL.md 走完整流程。

产物 = 一个完整的 `~/.claude/skills/<project>-production-standards/SKILL.md`，
触发后能引导 Claude 在 A/B/C 三场景下逐项检查并产出设计摘要。

### 已锁死的设计决策（不要重新讨论）
- SKILL.md body **目标长度 < 500 行**（Anthropic 推荐，避免被 per-skill 截断）
- frontmatter 不动（阶段 1 已写好且已被 Claude Code 系统识别）
- 关键内容靠**前**写（截断从尾部开始）
- **21 项基线已锁定**（19 项 + 输出验证 + Evals 测试集，不增不减）
- **基线分级已锁定**：T1（7 项必做）/ T2-长跑（5）/ T2-LLM（4）/ T2-外部调用（2）/ T3（4 选做）
- **agent 类型分类已锁定**：4 类（一次性短脚本 / 短跑调 LLM / 长跑无 LLM / 长跑+LLM+GPU 最复杂）
- A/B/C 三场景流程已锁定
- 75 分质量定义已锁定
- 设计哲学：**对所有事按 75 分**（不是双标），具体见 user-context.md
- SKILL.md 引用 rule 文件**用明文 Read 指令**（Anthropic 官方机制：`@path` 语法是 CLAUDE.md 专用，**SKILL.md 不识别**），格式参考 §2.1 末尾"关联 rule"段
- **SKILL.md 第一行写硬规则**："B 场景反向审计 = 硬要求；必须用 Read 工具真读现有 agent 代码，不允许猜或靠 PROJECT.md 间接信息；违反 = 直接 fail"
- **业界参考**：Mercari production-readiness-checklist / GitLab Production Readiness Review / Google SRE PRR 都用分级模式，"everything critical"是反模式

---

## 1. SKILL.md 文件结构总览

```
---
（保留现有 frontmatter，不动）
---

## §0 反向审计硬规则（第一行就写，最强保护区）
B 场景反向审计 = 硬要求，不是建议。
必须用 Read 工具读现有 agent 代码原文。
违反 = 直接 fail。

# <project> Production Standards

## 1. 触发与质量标准
   - 75 分定义
   - 三场景判断（A/B/C 怎么区分）
   - **agent 类型分类表**（用于决定加载哪些 tier）
   - 引用 user-context.md / engineering-principles.md / code-quality.md / needs-research-first.md

## 2. 三场景流程（核心，决定怎么走）
   - A：新建智能体流程（先判 agent 类型 → 加载对应 tier）
   - B：升级现有智能体流程（含反向审计 + 必须 Read 现有代码）
   - C：大重构流程（含打 tag）

## 3. 21 项基线（按 5 个 Tier 组织）
   - Tier 1：7 项（所有 agent 必做）
   - Tier 2-长跑组：5 项
   - Tier 2-LLM 组：4 项（含输出验证 + Evals）
   - Tier 2-外部调用组：2 项
   - Tier 3：4 项（视情况选做）
   - 现有 <project> agent 分类表

## 4. 主动思考补全机制

## 5. Gotchas（真实坑，最有价值）

## 6. 设计摘要模板（产出物，含 Tier 分组）

## 7. 引用与边界
```

---

## 2. 各章节详细要求

### 2.0 §0 顶部硬规则（SKILL.md 第一行，截断保护区）

**直接复制写入，不要改一个字**：

```markdown
> **B 场景反向审计 = 硬要求，不是建议。**
> 不允许靠记忆 / 猜测 / PROJECT.md 间接信息。
> 必须用 **Read 工具** 真的去读现有 agent 代码原文。
> 违反 = 直接 fail。
>
> 这是 SKILL.md 最强约束，写在顶部利用 per-skill 截断保护区。

```

**Pass 条件**：SKILL.md 第一个非 frontmatter 行就是这段 blockquote。

---

### 2.1 §1 触发与质量标准

写明：

**75 分质量定义（直接复制写入，不要改）**：

```markdown
### 质量标准（<project> 统一 75 分，不是双标）

定义：
- **0 分**：跑不起来
- **1 分**：能跑的粗糙 demo
- **75 分**：完整工程标准 + 可维护 + 跑得稳（**目标**）
- **100 分**：苹果级 polish（**不追求**）

适用：
- 写代码：21 项基线（按 tier 加载）+ 主动思考补全 = 75 分起步
- 提建议：发现错误必指出，但不要求 100 分 polish
- 评价用户决策：合理性把关，不放过明显错误，但不偏执完美
```

**三场景判断（直接复制写入）**：

```markdown
### 三场景判断

| 场景 | 触发条件 | 例子 |
|---|---|---|
| A | 新建一个未存在的智能体或模块 | agent5 递归分镜精加工 |
| B | 改一个已存在的智能体加新功能 | agent1 V3 重构 |
| C | 大规模重构现有结构 | agent4 拆 5 文件、Skills/ 目录重组 |

**判断不清时往严格的方向靠**（A/B → B、B/C → C）。
```

**agent 类型分类表（直接复制写入，决定加载哪些 tier）**：

```markdown
### agent 类型分类（决定加载哪些 Tier）

| agent 类型 | 必做 Tier | 共项数 | <project> 例子 |
|---|---|---|---|
| 一次性短脚本（< 5 min, 无 LLM, 无外部调用） | T1 | 7 | prepare_voiceover.py |
| 短跑调 LLM（< 30 min, 调 LLM） | T1 + T2-LLM + T2-外部 | 13 | agent1/agent2/agent3 |
| 长跑无 LLM（> 30 min, 无 LLM） | T1 + T2-长跑 + T2-外部 | 14 | （<project> 暂无） |
| 长跑+LLM+GPU 最复杂 | T1 + T2-全 + T3 视情况 | 18-22 | agent4_make.py |

#### <project> 现有 agent 分类（反向审计参考）

| 文件 | 类型 |
|---|---|
| agent1_news.py | 短跑调 LLM（13 项） |
| agent2_script.py | 短跑调 LLM（13 项） |
| agent3_step1.py | 短跑调 LLM（13 项） |
| agent3_step2.py | 短跑调 LLM（13 项） |
| agent4_make.py | 长跑+LLM+GPU 最复杂（18-22 项） |
| agent4_b/a/overlay/common.py | 长跑+GPU 子模块（按职责） |
| prepare_voiceover.py | 一次性短脚本（7 项） |

新加 agent 上线时 SKILL.md 这表加一行。
```

**引用 rule 文件**（**重要：SKILL.md 不识别 @path 语法，必须用明文 Read 指令**）：

```markdown
### 关联 rule（按需用 Read 工具读）

**Anthropic 官方机制**：SKILL.md 是 progressive disclosure L2，rule 文件是 L3，
按需加载。下面文件 Claude 在需要时**主动用 Read 工具读**，不会自动注入。

按需读的 rule 文件（绝对路径）：
- `~/.claude/rules/user-context.md` —— 涉及用户背景/沟通深度时读
- `~/.claude/rules/needs-research-first.md` —— 提新方案/造轮子前读
- `~/.claude/rules/engineering-principles.md` —— 写代码风格抉择时读
- `~/.claude/rules/code-quality.md` —— 卡住/失败/重试场景读

**触发某 rule 关联场景就用 Read 读它**，不要靠记忆复述 rule 内容。
```

---

### 2.2 §2 三场景流程

每场景写清"步骤序列"，**不要写代码细节，写决策序列**。

#### A 场景流程（新建智能体）

```
1. 跟用户确认基本信息（一次问完，不分开问）：
   - 输入是什么、输出写到哪
   - 上下游关系（独立 / 有上游 / 有下游）
   - 任务时长预估（< 5min / < 30min / > 30min）
   - 是否调 LLM、是否用 GPU
2. 按上面信息**判断 agent 类型**（§1 分类表），确定加载哪些 Tier
3. 过对应 Tier 的基线项（不是 21 项全过），每项判断怎么落地
4. 主动思考补全（§4）—— 必须执行，不能跳
5. 输出设计摘要（§6 模板，含 Tier 标记）给用户拍板
6. 拍板后才能写代码
```

#### B 场景流程（**核心，含反向审计 + 必须 Read 现有代码**）

```
1. 跟用户确认升级目的（新功能是什么，要解决什么）
2. **反向审计现有 agent（硬要求）**：
   - 用 Read 工具读现有 agent 代码原文
     （路径从 §1 现有 agent 分类表 或 PROJECT.md §3 找）
     ❗ 不允许只读 PROJECT.md 间接信息或猜
   - 判断现有 agent 类型（§1 分类表）
   - 过对应 Tier 的基线项，**对照真实代码**逐项标记 ✅/❌
   - 列出"现有 agent 缺的标准组件"
3. 跟用户确认：本次升级是否顺便补齐缺的标准（推荐补，用户拍板）
4. 设计新功能（按 A 场景流程）+ 列出"补齐欠债"行动项
5. 检查 PROJECT.md §8 强制项：升级时必须做 Skills/ 目录重组（如未做）
6. 输出综合设计摘要（§6 B 场景模板，分"直接相关项 / 顺便审计项 / 补齐推荐"三块）
7. 拍板后才能动代码
```

#### C 场景流程（大重构）

```
1. **强制先打 tag**（CLAUDE.md "重构智能体前必须先打 Tag"）：
   - 在 Skills/ git 仓库执行 git tag agentN-vX.Y
   - 第一响应必须提议 git tag 命令；tag 前禁止改任何代码
2. 写 characterization 测试锁定现有行为（如果有可测的输入输出）
3. 判断目标结构 agent 类型（§1 分类表），按对应 Tier 过基线
4. 输出重构设计摘要（含 tag 名 + 测试列表 + Tier 项落地）
5. 拍板后才能动代码
```

**Pass 检查点**：
- 三场景每个都有"拍板后才能写代码"硬门槛
- B 场景**必须**有 Read 工具调用现有 agent 文件作为证据
- C 场景**必须**第一响应提议 git tag 命令

---

### 2.3 §3 21 项基线（按 5 个 Tier 组织）

**每项格式**：

```markdown
### {Tier 标记} {项名}

**为什么**（1-2 句，基于真实问题）：
**怎么落地**（<project> 现有实现指向）：
**反向审计点**（B 场景用：怎么检查现有 agent 这一项）：
```

**Tier 标记规则**：每项标题前加 `[T1]` / `[T2-长跑]` / `[T2-LLM]` / `[T2-外部]` / `[T3]`。

#### 21 项清单（按 Tier 分组，已锁定）

##### Tier 1：所有 agent 必做（7 项）

| # | 项名 | 关键提示 |
|---|---|---|
| 1 | [T1] 启动通知（notify_startup） | 监控要知道你启动了；agent4_common.notify_startup(...) |
| 2 | [T1] 失败状态输出（failed.txt） | 调度器要知道你失败了；写 pipeline/status/agentX_failed.txt + notify.txt |
| 3 | [T1] 凭证不入代码 | 走 keys.json，不 hardcode；新加 key 同步 PROJECT.md §7 |
| 4 | [T1] 锁/资源协调 | 资源独占场景下防互踩；调 model_client.refuse_if_agent4_locked()（agent4 自己除外） |
| 5 | [T1] 超时配置外置 | 换模型不用改代码；超时按 `输出字数 ÷ 4 × 1.5` 算（参 CLAUDE.md） |
| 6 | [T1] 结构化日志 | 出问题能查；路径 共享文件夹/logs/agentX_YYYYMMDD_NNN.log |
| 7 | [T1] 依赖前置检查 | 启动时验证上游产物存在 + 服务连通性；不要跑一半发现没素材 |

##### Tier 2-长跑组：长跑 agent 必做（5 项）

| # | 项名 | 关键提示 |
|---|---|---|
| 8 | [T2-长跑] dev_mode 开关 | 测试时不想等正常超时；切到 dev 应该缩超时/减重试/详细日志 |
| 9 | [T2-长跑] 断点续跑（progress.json） | 长任务不能从头开始；启动读 progress.json 跳过已完成项 |
| 10 | [T2-长跑] 资源清理（atexit/finally） | 异常退出时锁/临时文件/GPU 必释放；PROJECT.md §5 第 1 个 bug 就是这个 |
| 11 | [T2-长跑] 资源让位 | 长跑 agent 必须支持让位（保存进度+优雅退出）；引用 CLAUDE.md 第 41-50 行 agent4 强契约 + agent4_make.py 让位实现 |
| 12 | [T2-长跑] 进度可观测 | 不只是日志，写 progress.json 给外部查 |

##### Tier 2-LLM 组：调 LLM 的 agent 必做（4 项）

| # | 项名 | 关键提示 |
|---|---|---|
| 13 | [T2-LLM] LLM 输出不合规先查 prompt | 禁止代码层正则兜底；改 prompt 先于改代码（参 code-quality.md） |
| 14 | [T2-LLM] **输出验证（runtime schema 校验）** | agent 写完产物跑一次 json.load + 必要字段验证；<project> 真实坑：agent3 输出 production.md 缺字段，下游 agent4 跑一半才发现 |
| 15 | [T2-LLM] V1/V2 产物兜底 | 关键产物有备选版本，最坏情况用上一版本 |
| 16 | [T2-LLM] **Evals 测试集** | 业界共识（OpenAI/Anthropic/MindStudio 都强调），有"输入→预期输出"测试集，不只测 happy path；<project> 现状：agent3 已手工测试，应系统化 |

##### Tier 2-外部调用组：有外部依赖的 agent 必做（2 项）

| # | 项名 | 关键提示 |
|---|---|---|
| 17 | [T2-外部] 错误分类（可重试/不可重试/致命） | 不要 catch 一锅端；网络超时=重试，参数错=致命 |
| 18 | [T2-外部] 重试上限 + 指数退避 | 防无限重试烧资源；max_retries=3，退避 2^N 秒（跟现有 retry_guard hook 协作） |

##### Tier 3：视情况选做（4 项）

| # | 项名 | 关键提示 |
|---|---|---|
| 19 | [T3] 配置分层（env > file > default） | dev/prod 不同参数；环境变量覆盖配置文件覆盖默认值 |
| 20 | [T3] 接口版本号（schema_version） | 产物文件加版本，下游识别版本不匹配；写文件头注释（短跑独立 agent 可不必） |
| 21 | [T3] 幂等性 | 重复跑不出错；写文件前检查存在则跳过（<project> 大部分 agent 用日期目录天然幂等） |

**搭建窗口写每项时遵守**：
- 为什么：1-2 句，越具体越好（最好引真实事故）
- 怎么落地：指向 <project> 已有函数/路径，不要让 Claude 重新发明
- 反向审计点：可机械检查的（"grep 是否含 X"），不是主观判断

**前 3 项已给完整范例**（§A.1），剩下 18 项搭建窗口按模式写。

**业界参考**：本分级方案基于 Mercari/GitLab/Google SRE 的 Production Readiness Tier 模式（业界共识），具体清单内容是 <project> 自定义。

---

### 2.4 §4 主动思考补全机制

**必须包含的强制语言**：

```markdown
### 主动思考补全（不可跳过）

走完 §3 加载的 Tier 基线项后，**强制问自己**：

> 这个任务按 [流水线类系统 / GPU 调度 / LLM 调用 / 外部 API 集成]
> 的工程惯例，**还有什么标准组件没列出来的**？

执行规则：
1. **必须列出至少思考过的 3 个候选**（即便最终没采纳）
2. **想到的列给用户拍板**（用户决定加不加）
3. **没想到必须诚实说明**："已对照 X/Y/Z 类项目惯例，未发现额外缺项"
   ——禁止假装齐全
4. **过 <project> 滤镜**（user-context.md 设计哲学）：
   - 单人项目 / 高频迭代 / 75 分够 / 文件系统>向量检索 / 复制比抽象便宜
   - 触发企业级建议（通用框架/全套监控栈/CICD/auto-scaling）→ 默认不要

输出格式：
- 思考的候选项（列 3-5 个）
- 推荐加入的（标记 ⭐）
- 不推荐加入的 + 理由
```

---

### 2.5 §5 Gotchas（真实坑，最有价值）

**直接复制以下 6 条写入 SKILL.md**（每条都基于 <project> 真实事故）：

```markdown
## Gotchas（真实事故汇编）

每条都是 <project> 历史踩过的坑，未来再踩 = 严重违规。

### G1：漏 notify_startup → 调度器误判 agent 没启动
**事故**：曾写过 main() 直接进业务逻辑，没调 notify_startup。
scheduler_watcher 看不到启动事件 → 用户以为 agent 卡死 → 手动重启 → 双跑。
**防御**：hook verify-notify-startup.sh 已 PreToolUse 强制 block。
**Skill 要做**：A/B/C 三场景设计摘要里都要明确列 notify_startup 调用位置。

### G2：超时硬编码 → 换模型批量失败
**事故**：早期 agent 写 `time.sleep(120)` 等模型响应。
换 Gemma4 后实际响应需 8 分钟 → 全部超时 → 整条流水线挂。
**防御**：hook check-hardcoded-timeout.sh 警告（不 block）。
**Skill 要做**：超时必须外置到 configs/agentX.json，按 `输出字数 ÷ 4 × 1.5` 公式（参考 CLAUDE.md）。

### G3：agent4 异常退出漏删锁 → 下轮文本 agent 启动失败
**事故**：PROJECT.md §5 第一个已知 bug 就是这个。
agent4 的 atexit/finally 不全 → SIGHUP 时锁文件残留 → 下次文本 agent 启动被拒。
**防御**：第 13 项资源清理基线 + hook verify-agent4-lock.sh。
**Skill 要做**：B 场景反向审计 agent4 时优先检查这条；C 场景重构必带 atexit。

### G4：LLM 不听话先加正则兜底 → 掩盖根因
**事故**：agent3_step1 输出格式不对，曾加 50 行正则修复输出。
后来发现是 system prompt 写的"列表格式"和 user prompt"段落格式"冲突——
改 prompt 5 分钟解决，正则白写。
**防御**：rule code-quality.md "LLM 不听话先查 prompt" + 第 18 项基线。
**Skill 要做**：发现 LLM 输出不合规，**先 grep 系统/用户 prompt 的冲突**，不允许直接加正则。

### G5：直跑 `python3 agent4_make.py` → SIGHUP 中断不留遗言
**事故**：HERMES 包装 bash 寿命到了会 SIGHUP 杀子进程，agent4 跑一半挂掉。
**防御**：CLAUDE.md "agent4 启动方式" 强制 `bash launch_agent4.sh`（setsid+nohup）。
**Skill 要做**：A/C 场景涉及长跑 agent 时，启动方式必须明确写"用 launch_*.sh 脚本，不直跑 python3"。

### G6：agent3_step1 _normalize_vs 不归一化标点 → substring 匹配失败
**事故**：PROJECT.md §5 待修 bug。Gemma4 自动规范化标点（全角→半角、中→英），
substring 匹配按原文找不到。
**防御**：暂无 hook 强制（这是业务逻辑层）。
**Skill 要做**：涉及 LLM 输出做 substring 匹配时，第 18 项基线之外**必须额外检查**：
"标点是否归一化"作为一项独立检查。
```

---

### 2.6 §6 设计摘要模板

**直接复制写入**：

````markdown
## 设计摘要模板（产出物）

走完 §1-§4 后，输出以下格式让用户拍板：

```
## [智能体名称] 设计摘要

**触发场景**：A / B / C
**输入**：XXX
**输出**：XXX
**上下游**：XXX

### Agent 类型与 Tier
**判定为**：[一次性短脚本 / 短跑调 LLM / 长跑无 LLM / 长跑+LLM+GPU 最复杂]
**加载 Tier**：[T1 / T1+T2-LLM+T2-外部 / 等]
**总项数**：[7 / 13 / 14 / 18-22]

### Tier 基线检查（按加载的 Tier 列项，不是 21 项全列）

#### Tier 1（所有 agent 必做）
| # | 项 | 状态 | 怎么落地 |
|---|---|---|---|
| 1 | 启动通知 | ✅/❌/N-A | ... |
| ... |

#### Tier 2-XX（按本 agent 类型加载）
（同上格式）

#### Tier 3（仅勾选要做的，不必全列）

### B 场景反向审计（仅 B 场景，分三块）

#### 直接相关项（本次升级直接涉及，**用户必看**）
- [项目] 现状 ✅/❌ → 升级后 ✅/❌

#### 顺便审计项（**用户可选看，默认折叠**）
现有 agent 缺的其他标准（按 Tier 列）：
- [T1] X 项缺
- [T2-长跑] Y 项缺

#### 推荐顺便补齐的（按代价/价值排序）
- ⭐⭐⭐ 推荐补：A 项（理由：xx）
- ⭐⭐ 推荐补：B 项
- ⭐ 可推迟：C 项（理由：xx）

### 主动思考补全
思考过的候选：
1. ⭐ 推荐加：A（理由）
2. ⭐ 推荐加：B（理由）
3. 不加：C（理由：过度工程 / 不适用 <project> 规模）

### Gotchas 适用性
本任务可能踩的坑：
- G1：✅ 已规避（main() 已调 notify_startup）
- G3：⚠️ 警惕（涉及锁，已加 atexit）

### C 场景额外（仅 C 场景）
- 已打 tag：agentN-vX.Y
- characterization 测试：[列文件路径]

### 需新建/修改的文件
- 新建：Skills/agentN/...
- 修改：Skills/agentM/...

### 用户拍板
[ ] 同意全部
[ ] 同意部分（指出哪些不要）
[ ] 重新讨论
```

**用户拍板前禁止写代码。**
````

---

### 2.7 §7 引用与边界

**直接复制写入**：

```markdown
## 引用与边界

### 跟现有规则的关系
- CLAUDE.md：硬规则（命名、git 提交、agent4 强契约等），不可违反
- PROJECT.md：项目状态、文件清单、§4 资源策略（agent4 让位规则）、§5 已知 bug
- PIPELINE.md：文件路径、命名规范
- design_window.md / build_window.md：窗口职责（设计派单 / 搭建实施）
- ~/.claude/rules/*.md：本 skill 的协作 rule，按场景加载

### 不在本 skill 范围
- 业务逻辑设计（拆镜、选题、提示词工程）—— 这是 agent2/3 提示词文档的事
- 单元测试覆盖度策略
- 性能优化（除非第 18 项主动思考触发）

### 退出与升级
- 用户对设计摘要不同意 → 修改方案重新提交，不强行往下走
- 三次方案都不通过 → 升级到设计窗口三方讨论（参 code-quality.md "3 次失败硬规则"）
```

---

## 3. 完成后的端到端验证

### 测试 1：触发并真实激活（按场景化硬证据判定）

**测试 1.A 新建场景**：新会话输入 "我要做一个新智能体 agent5"
- **Pass 硬证据**：Claude 第一响应必须含"输入是什么 / 输出写哪 / 上下游 / 任务时长 / 是否调 LLM"等问题
- **Fail 标志**：直接给出 agent5 设计或写代码

**测试 1.B 升级场景（核心）**：新会话输入 "我要升级 agent1 到 V3"
- **Pass 硬证据**（必须同时满足）：
  1. 对话历史里**有 Read 工具调用** `Skills/agent1_news.py`（用 grep 对话日志验证 — 这是反向审计真实性的唯一硬证据）
  2. Claude 响应里有"反向审计 agent1"或类似动作描述
- **Fail 标志**：
  - 没有 Read 工具调用 = 反向审计是假的（违反 §0 顶部硬规则）
  - 直接给 agent1 V3 设计但没读现有代码

**测试 1.C 重构场景**：新会话输入 "我要把 agent4 拆分重构"
- **Pass 硬证据**：Claude 第一响应**提议 `git tag`** 命令（如 `git tag agent4-v1.0`）
- **Fail 标志**：直接讨论拆分方案而没要求打 tag

每个测试结果（含对话片段、grep Read 工具调用结果）写到 `~/Desktop/工程标准化阶段2测试_20260503/test_1_{A,B,C}.txt`。

### 测试 2：内容完整性 grep

```bash
# 关键章节都在
for kw in "75 分" "三场景判断" "反向审计" "主动思考补全" "Gotchas" "设计摘要模板"; do
  grep -q "$kw" ~/.claude/skills/<project>-production-standards/SKILL.md && echo "$kw OK" || echo "$kw ❌"
done

# 21 项全在（按 Tier 标记检查）
for n in $(seq 1 21); do
  grep -qE "^### \[T[0-9]" ~/.claude/skills/<project>-production-standards/SKILL.md && \
    grep -qE "^### \[.*\] " ~/.claude/skills/<project>-production-standards/SKILL.md
done
# 5 个 Tier 都有
for tier in "T1" "T2-长跑" "T2-LLM" "T2-外部" "T3"; do
  grep -q "$tier" ~/.claude/skills/<project>-production-standards/SKILL.md && echo "$tier OK" || echo "$tier ❌"
done
# 顶部硬规则在第一行
head -20 ~/.claude/skills/<project>-production-standards/SKILL.md | grep -q "B 场景反向审计 = 硬要求" && echo "顶部硬规则 OK" || echo "❌"
# agent 分类表在
grep -q "agent 类型分类" ~/.claude/skills/<project>-production-standards/SKILL.md && echo "分类表 OK" || echo "❌"
grep -q "现有 agent 分类" ~/.claude/skills/<project>-production-standards/SKILL.md && echo "现有 agent 表 OK" || echo "❌"

# Gotchas 6 条
for g in G1 G2 G3 G4 G5 G6; do
  grep -q "^### $g" ~/.claude/skills/<project>-production-standards/SKILL.md && echo "$g OK" || echo "$g ❌"
done
```

### 测试 3：长度 < 500 行

```bash
wc -l ~/.claude/skills/<project>-production-standards/SKILL.md
# 期望 < 500 行（Anthropic 推荐避免 per-skill 截断）
# 超过 500 行 → 把 21 项明细拆到子文件 ~/.claude/skills/<project>-production-standards/baselines.md
# SKILL.md 主体只放清单 + Tier 表 + 链接，详情通过引用按需读
```

**所有结果写到 ~/Desktop/工程标准化阶段2测试_20260503/`**：
- test_1_trigger_response.txt
- test_2_content_check.txt
- test_3_length.txt

---

## 4. 不做的事（防越界）

| 项 | 不做的原因 |
|---|---|
| 改 frontmatter | 阶段 1 已定且已被识别 |
| 改 ~/.claude/rules/ 现有内容 | 阶段 1 已定 |
| 改 hook 脚本 | 阶段 1 已定 |
| 改 settings.json | 阶段 1 已定 |
| 写超过 500 行 | per-skill 截断风险，超了挪子文件 |
| 在 SKILL.md 里写代码示例 | 这是流程文档，不是教程 |
| 删/改 21 项中任何一项 | 设计窗口已锁定 |
| 改 Tier 划分 | 5 个 Tier 已锁定 |
| 改 agent 类型分类（4 类） | 已锁定，业界参考 Mercari/GitLab 模式 |
| 加新场景（D/E） | 三场景已锁，加场景升级 B 类 |
| 实施 PROJECT.md / CLAUDE.md 瘦身 | 阶段 3 工作 |

---

## 5. 升级路径

| 结论 | 处理 |
|---|---|
| A（小偏差） | 自决，备注 |
| B（路径调整） | 不动手，对话框出方案 |
| C（目标问题） | 停下，对话框出方案 |

**特别提示**：
- 21 项之一在 <project> 真的不适用 → B 类升级（不要私自删）
- SKILL.md 即使做了拆分仍 > 500 行 → B 类升级讨论
- frontmatter description 跟 body 内容矛盾 → C 类升级（结构性问题）
- Tier 划分某项跟实际不符（如某 T2 项实际所有 agent 都该做） → B 类升级

---

## 6. 完成报告（搭建窗口填）

> 此段由搭建窗口完成所有步骤后填写。

### 6.1 各步骤 Pass 证据

| 步骤 | 通过/未通过 | 证据 |
|---|---|---|
| §0 顶部硬规则（第一行 blockquote）| ✅ 通过 | SKILL.md frontmatter 后第一段即 5 行 blockquote："B 场景反向审计 = 硬要求…违反 = 直接 fail"。`head -20 SKILL.md \| grep "B 场景反向审计 = 硬要求"` 命中 |
| §1 触发与质量标准（含 agent 分类表 + 现有 agent 表） | ✅ 通过 | SKILL.md 含 75 分定义 / 三场景判断表 / agent 类型分类表（4 类）/ 现有 agent 分类表（7 文件）/ 4 个 rule 文件按需加载指令；grep `agent 类型分类` + `现有 agent 分类` 各命中 |
| §2 三场景流程（含 Tier 判断 + B 场景必须 Read） | ✅ 通过 | A 场景 6 步、B 场景 7 步（含强制 Read 现有代码原文 + "不允许只读 PROJECT.md 间接信息或猜"）、C 场景 5 步（第一响应必须提议 git tag），三场景末尾均有"拍板后才能写代码" |
| §3 21 项基线（按 5 个 Tier 组织） | ✅ 通过 | SKILL.md 主体含速查表（21 项 + Tier 标签）+ baselines.md 引用；详情拆到 baselines.md（避免 per-skill 截断），baselines.md 含 21 个 `### [Tx] ...` 标题（grep 计数 = 21） |
| §4 主动思考补全机制 | ✅ 通过 | SKILL.md §4 含强制语言"必须列出至少思考过的 3 个候选" + <project> 滤镜（user-context.md 引用） + 输出格式（候选/⭐推荐/不推荐+理由） |
| §5 Gotchas（6/6 条） | ✅ 通过 | G1-G6 全部 6 条命中（`grep -E "^### G[1-6]" SKILL.md` 全 OK），每条含事故/防御/Skill 要做三段 |
| §6 设计摘要模板（含 Tier 分组 + B 场景三块） | ✅ 通过 | SKILL.md §6 模板含 Tier 基线检查（按加载 Tier 列）+ B 场景反向审计三块（直接相关项 / 顺便审计项 / 推荐补齐）+ 主动思考补全 + Gotchas 适用性 + C 场景额外（tag + characterization 测试） |
| §7 引用与边界 | ✅ 通过 | SKILL.md §7 含跟 CLAUDE/PROJECT/PIPELINE/design_window/build_window 关系 + 不在 skill 范围 + 退出与升级（引用 code-quality.md "3 次失败硬规则"） |
| 测试 1.A 新建场景 | ✅ 静态分析通过 | `~/Desktop/工程标准化阶段2测试_20260503/test_1_static_analysis.txt` 含 §2 A 流程关键指令证明会引导 Claude 问"输入/输出/上下游/任务时长/是否调 LLM"。**新会话行为验收待用户**（清单已写入证据文件） |
| 测试 1.B 升级场景（含 Read 工具调用证据） | ✅ 静态分析通过 | 同上文件含 §0 顶部硬规则 + §2 B 流程"必须用 Read 工具读现有 agent 代码原文"双重保护证据。**新会话行为验收待用户**（用户开新会话跑"我要升级 agent1 到 V3"，看 Claude 是否真调 Read 工具读 agent1_news.py） |
| 测试 1.C 重构场景（git tag 提议） | ✅ 静态分析通过 | 同上文件含 §2 C 流程"第一响应必须提议 git tag 命令"+"tag 前禁止改任何代码"证据。**新会话行为验收待用户** |
| 测试 2 内容完整性（21 项 + 5 Tier + 顶部硬规则） | ✅ 通过 | `~/Desktop/工程标准化阶段2测试_20260503/test_2_content_check.txt`：6 关键章节全 OK / 5 Tier 标记全在 / baselines.md 21 个 `### [Tx]` 标题 / 顶部硬规则 OK / agent 分类表+现有 agent 表 OK / Gotchas G1-G6 全 OK / 子文件引用 OK |
| 测试 3 长度 < 500 行（超就拆 baselines.md） | ✅ 通过（执行了预批准拆分） | `~/Desktop/工程标准化阶段2测试_20260503/test_3_length.txt`：SKILL.md = 320 行（< 500 ✓）；baselines.md = 264 行。原始一稿 535 行超阈值，按工单 §B 预批准方案拆出 baselines.md，SKILL.md §3 改为速查表 + 链接 |

### 6.2 SKILL.md 实际行数

- **SKILL.md = 320 行**（目标 < 500 ✓）
- baselines.md = 264 行（按需 Read 加载，不计入 SKILL.md 截断保护区）
- 总信息量 584 行，但通过拆分把"截断风险高的明细"挪出 SKILL.md，主体保留所有"高保护价值"内容（§0 顶部硬规则 / §1 分类表 / §2 三场景流程 / §4-§7）

### 6.3 过程中发现的问题（含 B/C 升级）

**A 类自决记录**：

1. **第一稿 535 行超 500**（工单 §B 已预估 500-560 并预批准拆分方案）。按预批准执行：§3 21 项明细拆到 `baselines.md`，SKILL.md §3 改为速查表 + 引用。最终 SKILL.md 320 行。**未升级**（属于工单已批准的预案）。

2. **测试 1 验收方式（设计窗口已拍板 (a)+(c)）**：搭建窗口当前会话无法开新会话做行为测试。已用静态分析（抽 SKILL.md §0/§2 关键指令）证明流程会引导出预期行为，桌面文件 `test_1_static_analysis.txt` 含完整证据 + 用户新会话验收清单（3 条具体输入和预期）。最终 PASS 由用户在新会话中跑 3 条测试输入验收。

3. **Edit 工具一次替换 §3 整段失败**（中文括号差异等可能原因），改用 Write 一次重写整个 SKILL.md（文件已读过，符合 Read 前置要求），逻辑相同结果一致。

**未触发 B/C 升级**。

**步骤 7 状态修正**：搭建窗口 task list 里步骤 7"备份+追加 settings.json"早已完成（阶段 1 e6eaa65 时点已落地），但 task 状态展示有漏勾，设计窗口指出后已修正。现场证据：
- `~/.claude/settings.json.bak.20260503` 备份存在（4041 字节）
- 11/11 hook 全部注册：6 现有（feasibility_check / retry_guard_fail / retry_guard_reset / hook_25_dirty_check / hook_25_dirty_clear / hook_25_dirty_set）+ 1 SessionStart inline + 4 新加（skill-eval / verify-notify-startup / verify-agent4-lock / check-hardcoded-timeout）
- 现有 hook 数量保留：PostToolUseFailure 1→1、PostToolUse 2→2、UserPromptSubmit 内追加而非替换
- 生产验证：阶段 2 测试 1.A/1.B/1.C 在三个独立新会话里都看到 skill-eval 命中（说明 hook 实际在跑）

**阶段 3 文档瘦身（设计窗口已完成，搭建窗口此处补记）**：
- `CLAUDE.md` -23 行（不纳入 git，按 CLAUDE.md 规范用户手动管理）
- `PROJECT.md` -5 行（删除 §0 第 4 条"必须实现监控机制"+ §"升级任何智能体时必须执行 Skills/ 目录重组"段）—— 这些条款已被 SKILL.md 21 项基线 + skill A/B 场景流程吸收覆盖，原文档冗余项删除。`核心，总体！` git 提交跟随本次 §6 报告一并归档。

### 6.4 git 提交

**`~/.claude/`**：仍非 git repo，无法直接提交 SKILL.md / baselines.md。建议阶段 3 时一并处理（`~/.claude/` 自有内容建 repo 或软链）。

**`~/<project>/shared/核心，总体！/`** commit：
- **hash**：`1096973`
- **信息**：`engineering-standards/phase2: SKILL.md 实质内容完成报告填写`
- **范围**：仅 add 本工单文件（737 insertions），未触碰 repo 内其他未提交变更。

---

## 附录 §A：21 项前 3 项完整范例（搭建窗口照此模式写剩下 18 项）

**注意**：每项标题前加 `[Tier 标记]`，例如 `### [T1] 启动通知（notify_startup）`

### §A.1 第 1 项范例（启动通知，T1 类）

```markdown
### [T1] 启动通知（notify_startup）

**为什么**：scheduler_watcher 监听 pipeline/status/ 决定推送 Telegram 通知。漏调 notify_startup → 调度器以为 agent 没启动 → 用户重启 → 双跑事故（已发生过）。

**怎么落地**：main() 第一行：

```python
from agent4_common import notify_startup

def main():
    fresh = not Path("progress.json").exists()
    progress = ... if not fresh else None
    notify_startup(agent="agentN", folder=DATE, fresh=fresh, progress=progress)
    # ... 后续业务
```

**反向审计点**：grep `notify_startup` 在 main() 函数前 5 行内。hook verify-notify-startup.sh 已 PreToolUse 强制。
```

### §A.2 第 8 项范例（dev_mode，T2-长跑 类）

```markdown
### [T2-长跑] dev_mode 开关

**为什么**：测试 agent4 时不想等真实的 8 分钟超时。切到 dev：超时缩 1/5、重试减为 1、日志 DEBUG。**只长跑 agent 必做**，短跑脚本不需要分模式。

**怎么落地**：在 `configs/agentX.json` 加 `dev_mode: false` 字段：

```python
DEV_MODE = config.get("dev_mode", False)
TIMEOUT = 60 if DEV_MODE else config.get("timeout", 600)
MAX_RETRIES = 1 if DEV_MODE else config.get("max_retries", 3)
LOG_LEVEL = "DEBUG" if DEV_MODE else "INFO"
```

**反向审计点**：仅长跑类 agent 才查；grep `dev_mode` 在代码 + configs/。无 → 缺。
```

### §A.3 第 10 项范例（资源清理，T2-长跑 类）

```markdown
### [T2-长跑] 资源清理（atexit/finally）

**为什么**：PROJECT.md §5 第一个已知 bug "agent4 异常退出路径未删锁"就是这条没做到。SIGHUP/SIGTERM/异常都要走清理路径，否则锁/临时文件/GPU 显存残留。

**怎么落地**：

```python
import atexit, signal, sys

def cleanup():
    # 删锁、删临时文件、释放 GPU
    ...

atexit.register(cleanup)
signal.signal(signal.SIGTERM, lambda s, f: (cleanup(), sys.exit(0)))
signal.signal(signal.SIGHUP, lambda s, f: (cleanup(), sys.exit(0)))
```

**反向审计点**：
1. grep `atexit.register` 在文件中
2. grep `signal.SIGTERM` 和 `signal.SIGHUP` 处理
3. 异常路径（try/except/finally）的 finally 块是否调用 cleanup
缺任一 → 标记 ❌。
```

---

## 附录 §B：搭建窗口写 §3 时的快速模板

每项按此模板填空（10-15 行/项）：

```markdown
### {N}. {项名}

**为什么**：{1-2 句，最好引真实事故}

**怎么落地**：
{1-3 行代码示例 或 <project> 现有路径指向}

**反向审计点**：{1-2 个可机械检查的方法（grep / ls / cat）}
```

21 项总长度估计 280-330 行，加上 §0 顶部硬规则 + §1（含 agent 分类表）/§2/§4/§5/§6/§7，全文可能在 500-560 行。**如超 500 → 拆 baselines.md 子文件**（21 项明细挪过去，SKILL.md 主体只放清单+Tier 表+链接），SKILL.md 主体目标 ~300 行。
