# 工单：<project> 工程标准化 阶段 1（搭骨架）

> 派工时间：2026-05-03
> 派工人：设计窗口
> 接工人：搭建窗口

---

## 0. 背景与目标

### 背景
设计窗口与用户讨论后，决定建立 <project> 工程标准化体系，目标是让 Claude Code 在以下三种场景下自动按工程标准干活，用户不再操心标准组件：

- A. 新建智能体
- B. 升级现有智能体加新功能（agent1 V3 重构是首例）
- C. 大重构现有智能体结构（Skills/ 目录重组、agent4 拆 5 文件）

### 阶段 1 范围
**搭骨架，不填实质内容**。让 hook 能触发、rule 能加载、skill 文件占位。SKILL.md 实质内容由阶段 2 写。

### 已锁死的设计决策（不要重新讨论）
- 不装 Superpowers（跟"75 分够"原则冲突，详见会话记录）
- **路径放全局 `~/.claude/`**（不是项目级，本机所有 Claude Code 窗口都生效）
- **不创建新 settings.json**，直接 Edit `~/.claude/settings.json` 追加 hook 注册（避免覆盖现有 7 个 hook）
- skill 单数：`<project>-production-standards`
- 触发场景：A/B/C 三类
- hook 路径用绝对路径 `~/.claude/hooks/...`（全局 hook 不依赖 `$CLAUDE_PROJECT_DIR`）
- hook 类型：**校验类用 PreToolUse**（能 block 写入），**触发类用 UserPromptSubmit**
- 校验 hook 必须支持**测试文件豁免**（文件名含 `_test_` 或 `_dryrun` 时跳过）

### 现有 hook 清单（已确认全部不冲突，**全部保留不动**）

| hook | 事件 | 功能 |
|---|---|---|
| feasibility_check.sh | UserPromptSubmit | 搜索守门 |
| retry_guard_fail.sh | PostToolUseFailure on Bash | Bash 工具失败计数 + block（3 次失败硬规则的代码层实现） |
| retry_guard_reset.sh | PostToolUse on Bash | 重置失败计数 |
| hook_25_dirty_check.sh | PreToolUse on Bash (agent3_step1.py) | agent3_step1 防呆 |
| hook_25_dirty_clear.sh | PostToolUse on Bash (prepare_voiceover.py) | prepare_voiceover 跑后清理 |
| hook_25_dirty_set.sh | PostToolUse on Edit\|Write | 任何写文件后标记 dirty |
| SessionStart | SessionStart | 读容器状态 |

**新加 4 个 hook 跟它们并存**：skill-eval.sh + 3 个 verify-*.sh

### 重要参考材料
- 附录 §A：现有规则冲突消解清单（14 处）—— 改 rule 时必看
- 附录 §B：阶段 2 不在本工单范围的内容清单 —— 防越界做了阶段 2 的事

---

## 1. 抄录外部资源（步骤 2/3/5 用）

| 用途 | 来源 | 处理 |
|---|---|---|
| 通用工程原则 | `https://raw.githubusercontent.com/smartwhale8/claude-playbook/main/.claude/rules/engineering-principles.md` | 抄+本地化（删 DRY 段，加 <project> 说明） |
| 代码质量原则 | `https://raw.githubusercontent.com/smartwhale8/claude-playbook/main/.claude/rules/code-quality.md` | 抄+补充（吸收 build_window.md 复杂度反向条款） |
| 关键词触发机制 | `https://github.com/ChrisWiles/claude-code-showcase` 的 `.claude/hooks/skill-eval.{sh,js}` 和 `skill-rules.json` | 抄+替换关键词为 <project> 中英双语 |

抄录前必须先 fetch 实际内容确认 license（MIT/Apache/未声明 三种处理不同）。

---

## 2. 执行步骤（每步带 Pass 条件）

### 步骤 1：创建目录骨架

```bash
# rules/ 和 hooks/ 子目录现有 ~/.claude/ 里没有，新建
# skills/ 子目录已存在（含其他全局 skill），加 <project>-production-standards
mkdir -p ~/.claude/{rules,hooks,skills/<project>-production-standards}
```

**Pass 条件**：
- `ls ~/.claude/rules/`、`ls ~/.claude/hooks/`、`ls ~/.claude/skills/<project>-production-standards/` 都存在
- 现有的 `~/.claude/feasibility_check.sh`、`retry_guard_*.sh`、`hook_25_*.sh` 等文件**仍在原位置**（`ls ~/.claude/*.sh`），未被移动或删除

---

### 步骤 2：抄 engineering-principles.md 并本地化

**前置子步骤 2.0：检查 LICENSE**

```bash
curl -s https://raw.githubusercontent.com/smartwhale8/claude-playbook/main/LICENSE | head -5
```
- 是 MIT/Apache-2.0/BSD/MPL-2.0/未声明 → 继续抄
- 是 GPL/AGPL → 按 C 类升级（停下问设计窗口）

---

1. fetch 来源 raw 内容
2. 删除全部 DRY 相关段落（保留 YAGNI、KISS、SOLID 中适用项）；**如果原文没有 DRY 段，跳过此步**
3. 文件**头部**加一段 "<project> 本地化说明"：

   > **本项目不采用 DRY 原则**
   > 理由：<project> 是单人项目 + 4 个 agent 规模 + 高频迭代。
   > 抽象出公共框架的成本 > 复制几份的成本。
   > 单文件内部仍要避免复制粘贴 30 行（常识）；
   > 跨文件、跨 agent 的"抽象出公共框架"——禁止。

4. 保存到 `~/.claude/rules/engineering-principles.md`

**Pass 条件**：
- 文件存在
- 头部含 "<project> 本地化说明" 段落
- 全文 grep "DRY" 只出现在"为什么不抄 DRY"说明段落里（其他位置无）

---

### 步骤 3：抄 code-quality.md 并补充

**前置子步骤 3.0**：步骤 2.0 已经检查过 LICENSE，跳过。

1. fetch 来源 raw 内容
2. 文件**末尾**加一段 "<project> 补充"：

   > **复杂度反向 = 方向反了**
   > 修同一问题：50 行 → 100 行 → 200 行，第 4 次该是 30 行。
   > 加 try/except / 适配层 / 缓存往往是创可贴，删比加便宜。
   >
   > **改实现不改测试**
   > 测试不过改实现，除非证明测试本身错。
   >
   > **LLM 不听话先查 prompt**
   > 不加正则兜底、不加重试次数。兜底只会掩盖根因，让错误装死。
   >
   > **删比加便宜**
   > 在加 try/except / 适配层 / 缓存吗？先删掉看真因。
   > 50 → 100 → 200 行 = 方向反了，第 4 次该是 30 行。
   >
   > **3 次失败硬规则（用户最在意，过去执行不到位）**
   >
   > 跟现有 retry_guard hook 协作覆盖所有失败场景：
   > - hook 管 Bash 工具失败计数 + 强制 block 第 4 次
   > - 本规则管所有非 Bash 失败：改 prompt 后输出仍不对、改代码后跑测试还失败、
   >   改配置后行为没变、修同一 bug 多次未解决——任何动作没解决问题都算 1 次失败
   >
   > **3 次失败 → 立即停下，禁止第 4 次试**
   >
   > 停下后必须执行下列至少一个动作才能再试：
   > 1. **WebSearch 搜新关键词**（之前没搜过的，且要报告搜索结果）
   > 2. **让 plan-eng-review skill 介入讨论**（让外部视角审一遍）
   > 3. **升级到设计窗口三方讨论**
   >
   > 未执行 1/2/3 任一就直接试第 4 次 = 严重违规。
   >
   > **本质相同的尝试不算新尝试**：改变量名 / 调顺序 / 换错误处理 / 调 prompt 措辞
   > 不算"本质不同"——仍算同一次重复尝试，计入失败次数。
   >
   > **30 分钟时间锁（次数+时间，先到先触发）**：
   > 同一问题持续修了 30 分钟还没解决 → 也强制停，走 1/2/3 流程。
   > 不要等"我觉得快好了"——机械时间锁，"是否卡住" Claude 自评不可信。
   >
   > 这条是 <project> 反复强调的硬规则，过去多次违反。本次必须严格执行。

3. 保存到 `~/.claude/rules/code-quality.md`

**Pass 条件**：
- 文件存在
- 含关键词："复杂度反向"、"改实现不改测试"、"LLM 不听话先查 prompt"、"删比加便宜"、"3 次失败硬规则"、"30 分钟时间锁"
- "3 次失败硬规则" 段含 "WebSearch"、"plan-eng-review"、"升级到设计窗口" 三个必选动作
- 明确说明跟现有 retry_guard hook 协作（不是替代）

---

### 步骤 4：自己写 needs-research-first.md

路径：`~/.claude/rules/needs-research-first.md`

内容必须包含：

1. **三问框架**（流程图）：
   ```
   想做一件新东西
       ↓ 先搜（2-3 个英文关键词）
       ├─ 有成熟方案 → 抄
       └─ 搜不到 → 三问
           ├─ Q1: 真的没人做过吗？换关键词再搜
           ├─ Q2: 别人没做是「难」还是「没价值」？
           │    - 难 → 你也做不到，放弃
           │    - 没价值 → 重新评估需求是否合理
           └─ Q3: 你现有能力/资源下，做坏了能承受吗？
                - 不能 → 放弃
                - 能 → 真造轮子（抄能抄的 + 写不能抄的）
   ```

2. **跟现有 `feasibility_check.sh` hook 的关系**（必须写明）：

   > 本 rule 是 hook 的**文档化版本**，两者必须并存：
   > - hook 在每次 prompt 提交时**强制提醒搜**（防止"忘了搜"）
   > - rule 在 Claude 上下文里**说明为什么搜 + 怎么用搜索结果**（防止"搜了但不会用"）
   >
   > 不要因为有了本 rule 就以为可以删 hook，反之亦然。
   > 用户已多次强调搜索功能"必须有，只能加强不能削弱"。

3. **工业先例引用**（至少 1 个，用于说服 LLM 这不是用户独家偏执）：
   - Joel Spolsky 《Things You Should Never Do, Part I》（反对从零重写）
   - AWS Bezos 「门和窗户决定」（不可逆决策慎重）
   - Lean Startup 的 Build vs Buy 决策树

4. **<project> 实际案例**：今天讨论的"工程标准化 skill+hook 系统"，正是通过三问后决定造轮子的真实例子。

**Pass 条件**：
- 文件存在
- 含完整三问流程图
- 至少 1 个工业先例引用
- 至少 1 个 <project> 实际案例

---

### 步骤 4.5：自己写 user-context.md

路径：`~/.claude/rules/user-context.md`

**用途**：让所有 Claude Code 会话加载时知道用户是谁、熟悉什么、不做什么、设计哲学是什么。
现有 `design_window.md` 第 5-9 行只有简单的"懂分布式系统"，本 rule 是其全局加强版。

**完整内容（按下文复制写入，不要改动文字）**：

```markdown
# <project> 用户背景

## 谁
- 了解并行文件系统
- 了解高性能计算

## 熟悉的领域（按这个深度讨论，不降深度）
- 分布式存储、NAS、元数据控制、RAID、存储系统架构
- 并行文件系统、并行 HPC 解决方案
- 底层系统、契约、模块边界

## 不做的事
- **不写代码**
- **不审代码**（不要让用户判断 syntax 或代码行为是否正确）
- **不负责设计**（只定方向和关键判断；具体设计由 Claude Code 出方案后用户拍板）
- 工程标准组件不熟悉（<project> 工程标准化项目就是为补这个盲区）

## 设计哲学（核心，违反这条会被用户顶回来）

### 数据管理倾向：文件系统 > 向量检索
现代智能体常用"向量检索 + 一大堆东西扔一起"——**用户反对这种做法**。
偏好：
- **文件系统管理数据**：清晰的目录结构、明确的路径
- **上下文管理一层一层写指针**：父文档指向子文档，不堆在一起
- **原始的设计理念**：用最朴素的机制（文件、路径、指针），不堆叠抽象

### 整体哲学
- **简洁高效优先**：能用文件解决的不引入数据库
- **复制比抽象便宜**：小规模下两份代码 < 一个通用框架
- **75 分够**：不追求苹果美工级 polish，但要工程完整（不是能跑 demo 那种）
- **先搜后做**：想做的事一定有人做过，搜不到就重新评估需求是否合理

## 协作分工

| | 用户 | Claude Code |
|---|---|---|
| 方向、目标、关键判断 | 定 | 不定 |
| 架构方案、详细设计 | 拍板 | 出方案 |
| 写代码 | 不做 | 做 |
| 审代码 | 不做 | 自己审（不让用户判断 syntax） |
| 跑测试 + 验证结果 | 不做 | 做（结果留证据给用户看） |
| 工程标准组件 | 不熟 | 自动按 <project> 标准实施 |

## 讨论方式
- 按 senior 级语言（不要解释什么是分布式锁、什么是元数据）
- 涉及代码层细节时不要要求用户判断对错，直接给方案让用户拍板
- 涉及架构 / 数据流 / 资源调度时可以深入，用户能跟上
```

**Pass 条件**：
- 文件存在
- 含关键词："并行文件系统"、"高性能计算"、"分布式存储"、"文件系统 > 向量检索"、"75 分够"、"先搜后做"
- 含"协作分工"表格

---

### 步骤 5：抄 skill-eval 触发机制

**前置子步骤 5.0：LICENSE + 输入格式核实**

```bash
# 5.0a 检查 LICENSE
gh api repos/ChrisWiles/claude-code-showcase | grep -i license
# 或 fetch 仓库 LICENSE 文件
# 不是 MIT/Apache/BSD/MPL → C 类升级

# 5.0b 拉 skill-eval.sh 源码看它从 stdin 期望什么 JSON 字段
curl -s https://raw.githubusercontent.com/ChrisWiles/claude-code-showcase/main/.claude/hooks/skill-eval.sh
# 找出实际字段名（可能是 "prompt" / "user_prompt" / "text" / "message"）
# 记录下来，步骤 5 末尾的测试要用这个字段名
```

**判断**：如果 skill-eval.sh 的实际逻辑跟"匹配关键词→建议 skill"差异很大（比如它是 RAG 而不是关键词匹配），按 B 类升级。

---

1. 从 ChrisWiles/claude-code-showcase 的 `.claude/hooks/` 拉以下三个文件，**保持原文件名**：
   - `skill-eval.sh`
   - `skill-eval.js`
   - `skill-rules.json`

2. 保存到 `~/.claude/hooks/`

3. **替换 `skill-rules.json` 为 <project> 关键词**（中英双语，结构按原 schema）：

   ```json
   {
     "rules": [
       {
         "id": "<project>-scenario-A",
         "keywords": ["新智能体", "新 agent", "新模块", "新脚本", "new agent", "new module"],
         "skill": "<project>-production-standards",
         "context": "scenario=A (new agent)"
       },
       {
         "id": "<project>-scenario-B",
         "keywords": ["升级", "重构", "V2", "V3", "加新功能", "upgrade", "refactor"],
         "skill": "<project>-production-standards",
         "context": "scenario=B (upgrade existing)"
       },
       {
         "id": "<project>-scenario-C",
         "keywords": ["拆分", "目录重组", "结构调整", "restructure", "split"],
         "skill": "<project>-production-standards",
         "context": "scenario=C (large restructure)"
       }
     ]
   }
   ```

4. **路径替换**：原脚本里如有绝对路径，**改为 `~/.claude/hooks/...` 绝对路径**（全局 hook 不用 `$CLAUDE_PROJECT_DIR`，因为不依赖 cwd）

**Pass 条件**：
- 三个文件存在
- 模拟测试（**用步骤 5.0b 核实出的实际字段名**，下例假设是 `prompt`）：
  ```bash
  echo '{"prompt": "我要做新智能体"}' | bash ~/.claude/hooks/skill-eval.sh
  ```
  输出包含 `<project>-production-standards` 字符串

---

### 步骤 6：写 3 个校验 hook（PreToolUse 类型）

**重要**：所有校验 hook 都用 **PreToolUse** 类型，能在写入前 block。

**前置子步骤 6.0：核实 PreToolUse hook 的 stdin JSON 格式**

```bash
# fetch 官方文档确认 PreToolUse 的输入格式
# https://code.claude.com/docs/en/hooks
# 重点：tool_name 字段名、tool_input 子字段（file_path、content/new_string/old_string）
# 记录实际字段名，6.1/6.2/6.3 hook 内部解析按实际字段写
```

如果官方格式跟下面假设的差异大 → 按 B 类升级问设计窗口。

---

统一公共逻辑（每个 hook 开头都要写）：

```bash
# 1. 从 stdin 读 JSON，用 jq 提取：
#    - tool_name（应该是 "Write" 或 "Edit"，否则 exit 0 不管）
#    - file_path（Write 用 tool_input.file_path；Edit 也用 tool_input.file_path）
#    - content（Write 用 tool_input.content；Edit 用 tool_input.new_string）
# 2. 路径不匹配 */Skills/agent*.py → exit 0
# 3. 文件名（basename）含 _test_ 或 _dryrun → exit 0（豁免，方便开发测试）
# 4. 否则进入校验逻辑
# 5. 必须容忍 jq 不在的环境（fallback 用 grep 或 python3 -c）
```

---

#### 6.1 `verify-notify-startup.sh`

路径：`~/.claude/hooks/verify-notify-startup.sh`

校验逻辑（公共前置过滤通过后）：
- **同时满足三个条件**才 block：(a) 含 `def main`、(b) 含 `if __name__ == "__main__"`、(c) 不含 `notify_startup`
- 满足 → `exit 2` + stderr 输出错误信息（block 写入）
- 否则 → `exit 0`

判断 (b) 是为了排除工具文件（如 `comfyui_utils.py` 可能有 main 测试函数但不是 agent 入口）。

错误信息模板（**统一用"智能体X"，不用"agent X"**，遵守 CLAUDE.md "智能体命名"约束）：
```
[hook] verify-notify-startup: 检测到智能体入口（def main + __main__ 调用）但缺少 notify_startup()。
按 <project> 工程标准（CLAUDE.md "通知规范"），所有智能体 main() 开头必须调用
agent4_common.notify_startup(agent, folder, fresh, progress)。请补充后重试。
开发测试请将文件名改为含 _test_ 或 _dryrun 的形式跳过此校验。
```

---

#### 6.2 `verify-agent4-lock.sh`

路径同前缀。

校验逻辑（公共前置过滤通过后）：
- **额外排除**：`agent4_*.py`（这家自己就是 lock 持有者）
- **同时满足**才 block：(a) 含 `def main`、(b) 含 `if __name__ == "__main__"`、(c) 含 `model_client`、(d) 不含 `refuse_if_agent4_locked`
- 满足 → `exit 2` + stderr 错误信息
- 否则 → `exit 0`

错误信息：参照 6.1 风格，**统一用"智能体4"不用"agent4"**，提示按 CLAUDE.md "智能体4 最高优先级强契约第 3 条" 调 `model_client.refuse_if_agent4_locked()`。

---

#### 6.3 `check-hardcoded-timeout.sh`

路径同前缀。

校验逻辑（公共前置过滤通过后）：
- grep -E `time\.sleep\([0-9]{3,}\)|timeout=[0-9]{3,}` → `exit 1` + stderr warning（**不 block**，仅警告）
  - 阈值都用 `[0-9]{3,}`（≥100 秒），避免误伤合理的 `timeout=30` HTTP 连接超时
- 否则 → `exit 0`

警告信息：**统一用"智能体X"措辞**，提醒"≥100 秒的超时应外置到配置文件，按 <project> 工程标准第 2 项"。

---

**Pass 条件**：
- 三个文件存在且可执行（`chmod +x`）
- 单元测试（用 mock JSON 走 stdin）：

  | 测试用例 | 预期 |
  |---|---|
  | 写 `Skills/agent_demo.py`，含 `def main` + `__main__`，无 `notify_startup` | verify-notify-startup → exit 2 |
  | 写 `Skills/agent_demo_test_dryrun.py`，同样内容 | 任意 hook → exit 0（豁免） |
  | 写 `Skills/utility.py`（不匹配 agent*.py） | 任意 hook → exit 0 |
  | 写 `Skills/agent_demo.py`，含 `time.sleep(300)` | check-hardcoded-timeout → exit 1（warn） |
  | 写 `Skills/agent_demo.py`，含 `timeout=30` | check-hardcoded-timeout → exit 0（不警告） |

---

### 步骤 7.0：备份现有 settings.json（防回滚需要）

```bash
cp ~/.claude/settings.json ~/.claude/settings.json.bak.20260503
```

**Pass 条件**：备份文件存在。

---

### 步骤 7：Edit 现有 ~/.claude/settings.json，**追加** 新 hook 注册

**重要**：不创建新 settings.json。**直接 Edit 现有文件**，把新 hook 追加到对应事件的 `hooks` 数组里。

#### 7.1 现有 settings.json 的 hooks 字段结构（已确认，参考用）

```json
{
  "hooks": {
    "PostToolUseFailure": [{...retry_guard_fail.sh...}],
    "PreToolUse": [{...hook_25_dirty_check.sh on Bash...}],
    "PostToolUse": [
      {...retry_guard_reset.sh + hook_25_dirty_clear.sh on Bash...},
      {...hook_25_dirty_set.sh on Edit|Write...}
    ],
    "UserPromptSubmit": [{...feasibility_check.sh...}],
    "SessionStart": [{...容器状态...}]
  }
}
```

#### 7.2 要追加的内容

**在 `UserPromptSubmit` 数组里追加**：

```json
{
  "hooks": [
    {
      "type": "command",
      "command": "bash ~/.claude/hooks/skill-eval.sh",
      "timeout": 3
    }
  ]
}
```

**在 `PreToolUse` 数组里追加**（注意：现有 PreToolUse 的 matcher 是 Bash，新加的 matcher 是 Edit|Write，作为新对象追加，不要混进现有 Bash 那个对象）：

```json
{
  "matcher": "Write|Edit",
  "hooks": [
    {
      "type": "command",
      "command": "bash ~/.claude/hooks/verify-notify-startup.sh",
      "timeout": 5
    },
    {
      "type": "command",
      "command": "bash ~/.claude/hooks/verify-agent4-lock.sh",
      "timeout": 5
    },
    {
      "type": "command",
      "command": "bash ~/.claude/hooks/check-hardcoded-timeout.sh",
      "timeout": 5
    }
  ]
}
```

#### 7.3 不要动的内容

- 现有 7 个 hook 注册**一个字都不动**
- `permissions` 段不动
- `extraKnownMarketplaces` / `skipDangerousModePermissionPrompt` / `agentPushNotifEnabled` / `model` 不动

**Pass 条件**：
- `python3 -c "import json; json.load(open('~/.claude/settings.json'))"` 不报错
- `diff` 备份和新文件，**新增行包含**：`skill-eval.sh`、`verify-notify-startup.sh`、`verify-agent4-lock.sh`、`check-hardcoded-timeout.sh`
- `diff` **没有删除任何现有 hook 注册行**（grep 备份和新文件，现有 7 个 hook 名都在）

---

### 步骤 8：占位 SKILL.md

路径：`~/.claude/skills/<project>-production-standards/SKILL.md`

内容（仅 frontmatter + 占位）：

```markdown
---
name: <project>-production-standards
description: Use this skill when the user asks to (1) create a new agent or pipeline module, (2) upgrade an existing agent with new features, or (3) restructure existing agent code. Triggers include "新智能体", "新模块", "新脚本", "升级", "重构", "V2/V3", "加新功能", "拆分", "目录重组". This skill enforces 19 <project> production standards (dev_mode, externalized timeout, failure status, startup notify, lock check, retry limit, idempotency, logging, config layering, schema version, error classification, resource cleanup, secrets externalization, dependency precheck, progress observability, resource yielding, LLM-prompt-first debugging, V1/V2 fallback) before code is written or modified. For upgrades, ALSO audits the existing agent for missing standards and proposes fixing them in the same upgrade pass. MUST be invoked before drafting agent code — undertriggering causes agents to ship without standard components and break the pipeline (real incidents: agent4 lock leak, hardcoded timeouts breaking after model swap).
---

# <project> Production Standards

> **占位文件 — 阶段 2 填充实质内容**
>
> 阶段 2 工单将填充：
> - 75 分质量标准定义
> - 19 项基线标准（详细每项的"为什么/怎么落地/反向审计点"）
> - 主动思考补全机制
> - A/B/C 三场景流程
> - B 场景反向审计逻辑
> - Gotchas 区块（基于 PROJECT.md §5 + CLAUDE.md 真实坑）
> - 设计摘要模板
```

**Pass 条件**：
- 文件存在
- frontmatter 校验：`name` ≤ 64 字符且符合 `[a-z0-9-]+`、`description` ≤ 1024 字符且非空
- 含"占位"和"阶段 2"字样

---

### 步骤 9：端到端集成测试（**留证据到桌面，不污染 Skills/，不依赖自我汇报**）

**用户原则**：CLAUDE.md "测试输出规范" 要求测试文件放 `~/Desktop/具名文件夹_YYYYMMDD/`。
**用户原话**："Claude Code 说测试跑了，确实跑了，但他没验证，写桌面避免偷懒"。

**所有测试结果必须写桌面文件夹**让用户能直接看，不允许只在对话里说"通过了"。

#### 9.0 准备

```bash
mkdir -p ~/Desktop/工程标准化测试_20260503/
```

#### 9.1 触发测试（验证 skill-eval 命中关键词）

在**新 Claude Code 会话**（任意 cwd）输入：`我要做一个新智能体`

把以下三样东西保存到 `~/Desktop/工程标准化测试_20260503/test_9_1_trigger.txt`：
1. 用户输入的 prompt 原文
2. UserPromptSubmit hook 的 stderr 输出（如果有）
3. Claude 响应的前 5 行（看是否提到 `<project>-production-standards`）

**Pass 条件**：文件存在 + 含 `<project>-production-standards` 字符串。

#### 9.2 拦截测试（验证 PreToolUse hook 能 block 写入）

**用 mock 直接调 hook 脚本，不真的让 Claude 写文件**——这样不污染 Skills/，只测 hook 行为：

```bash
cat > ~/Desktop/工程标准化测试_20260503/test_9_2_input.json <<'EOF'
{
  "tool_name": "Write",
  "tool_input": {
    "file_path": "~/<project>/Skills/agent_demo.py",
    "content": "def main():\n    print('hello')\n\nif __name__ == '__main__':\n    main()\n"
  }
}
EOF

cat ~/Desktop/工程标准化测试_20260503/test_9_2_input.json \
  | bash ~/.claude/hooks/verify-notify-startup.sh \
  > ~/Desktop/工程标准化测试_20260503/test_9_2_block_stdout.txt \
  2> ~/Desktop/工程标准化测试_20260503/test_9_2_block_stderr.txt

echo "Exit code: $?" >> ~/Desktop/工程标准化测试_20260503/test_9_2_block_stderr.txt
```

**Pass 条件**：
- `test_9_2_block_stderr.txt` 含 `verify-notify-startup` 错误信息 + 含 "智能体" 字样
- `test_9_2_block_stderr.txt` 末尾的 `Exit code:` 非 0

#### 9.3 豁免测试（验证 _dryrun 后缀豁免）

```bash
cat > ~/Desktop/工程标准化测试_20260503/test_9_3_input.json <<'EOF'
{
  "tool_name": "Write",
  "tool_input": {
    "file_path": "~/<project>/Skills/agent_demo_test_dryrun.py",
    "content": "def main():\n    print('hello')\n\nif __name__ == '__main__':\n    main()\n"
  }
}
EOF

cat ~/Desktop/工程标准化测试_20260503/test_9_3_input.json \
  | bash ~/.claude/hooks/verify-notify-startup.sh \
  > ~/Desktop/工程标准化测试_20260503/test_9_3_exempt_stdout.txt \
  2>&1
echo "Exit code: $?" >> ~/Desktop/工程标准化测试_20260503/test_9_3_exempt_stdout.txt
```

**Pass 条件**：`test_9_3_exempt_stdout.txt` 末尾 `Exit code: 0`。

#### 9.4 现有 hook 不被破坏验证

```bash
# 测 feasibility_check.sh 仍然正常
echo '{"prompt":"测试"}' | bash ~/.claude/feasibility_check.sh \
  > ~/Desktop/工程标准化测试_20260503/test_9_4_feasibility.txt 2>&1
echo "Exit code: $?" >> ~/Desktop/工程标准化测试_20260503/test_9_4_feasibility.txt

# 列出 settings.json hooks 字段，确认 7 个现有 hook 全在
python3 -c "
import json
data = json.load(open('~/.claude/settings.json'))
existing = ['feasibility_check', 'retry_guard_fail', 'retry_guard_reset',
            'hook_25_dirty_check', 'hook_25_dirty_clear', 'hook_25_dirty_set']
new = ['skill-eval', 'verify-notify-startup', 'verify-agent4-lock', 'check-hardcoded-timeout']
config_str = json.dumps(data['hooks'])
print('=== 现有 6 个 hook 检查 ===')
for h in existing:
    print(f'{h}: {\"OK\" if h in config_str else \"❌ MISSING\"}')
print('=== 新加 4 个 hook 检查 ===')
for h in new:
    print(f'{h}: {\"OK\" if h in config_str else \"❌ MISSING\"}')
" > ~/Desktop/工程标准化测试_20260503/test_9_4_hook_inventory.txt
```

**Pass 条件**：
- `test_9_4_feasibility.txt` 含可信输出（说明 feasibility_check 仍能跑）
- `test_9_4_hook_inventory.txt` 现有 6 个 hook 全 OK + 新 4 个 hook 全 OK

---

#### 桌面文件夹清理

**搭建窗口完成测试后不要删除桌面文件夹**，让用户验收。
**用户验收通过后**用户自己 `rm -rf ~/Desktop/工程标准化测试_20260503/`。

**总 Pass 条件**：9.1 + 9.2 + 9.3 + 9.4 全部通过，桌面文件夹含至少 6 个证据文件。

---

## 3. 不做的事（防越界）

| 项 | 阶段 1 不做的原因 |
|---|---|
| 写 SKILL.md 实质内容 | 阶段 2 任务，本阶段只占位 |
| 改 CLAUDE.md / PROJECT.md | 等 agent1 V3 验证后再瘦身（决策已锁） |
| 改 design_window.md / build_window.md | 同上 |
| 改任何 `Skills/agent*.py` | 这是 hook 校验对象，不是改造对象 |
| 安装 Superpowers 插件 | 已决定不装 |
| 实现"反向审计"逻辑 | 这是 SKILL.md 内容，阶段 2 做 |
| 给 hook 加复杂逻辑（AST 解析等） | grep 够 75 分；用复杂方案违反 KISS |
| 实现"自动让位策略" | 这是 PROJECT.md 项目策略，不是 hook 内容 |

---

## 4. 完成后交付

按 build_window.md 第 47 行 "Pass 条件 = 任务完成的唯一定义"：

1. **9 个步骤的 Pass 条件全部通过**，每步给出实际命令输出/文件路径作为证据
2. **集成测试 9.1/9.2/9.3 的实际效果**（文字记录对话片段）
3. **过程中发现的问题**列表（如有）：例如 license 不兼容、抄来的 rule 跟 <project> 现有规则冲突等
4. **git 提交**（如 `共享文件夹/` 在 git 管理下）；提交信息：`engineering-standards/phase1: 搭建 .claude/ 骨架（rules + hooks + skill 占位）`
5. **完成报告**：搭建窗口**用 Edit 工具直接追加到本工单 §6**（不要创建新文件，不要在对话里输出后让用户复制——那会丢信息）

### 完成报告交付协议

```
搭建窗口完成所有步骤后：
  1. Edit 本工单 §6（在表格里填 Pass/未通过 + 证据）
  2. 在对话里告诉用户：
     "工单已完成并填写报告，文件路径：
      ~/<project>/shared/核心，总体！/WORKORDER_engineering_standards_phase1.md
      请把整个文件贴回设计窗口验收。"
  3. 不要主动启动 Claude 测试，让用户验收后再说
```

---

## 5. 升级路径（按 build_window.md §1）

| 结论 | 处理 |
|---|---|
| A 类（小偏差） | 自决，备注在完成报告里 |
| B 类（路径调整） | **不动手**，对话框输出"原方案 + 替代方案 + 理由"，等用户复制给设计窗口 |
| C 类（目标问题） | **停下**，对话框输出"原方案问题 + 替代思路"，三方对齐 |

**特别提示**：
- 步骤 5 抄 ChrisWiles 时，如发现 license 不允许直接复制 → 按 B 类升级（停下，问"换一个 MIT 的来源 OK 吗？"）
- 步骤 6 hook 用 grep 写不出来某条规则（比如 agent4 lock 检测涉及上下文） → 按 B 类升级
- 步骤 9 集成测试 hook 不触发 → 按 C 类升级（结构性问题）

---

## 6. 完成报告（搭建窗口填）

> 此段由搭建窗口完成所有步骤后填写。

### 6.1 各步骤 Pass 证据

| 步骤 | 通过/未通过 | 证据 |
|---|---|---|
| 1. 目录骨架（不破坏现有 ~/.claude/ 文件） | ✅ 通过 | `ls ~/.claude/{rules,hooks,skills/<project>-production-standards}/` 全部存在；现有 6 个 .sh 仍在 `~/.claude/`：feasibility_check / hook_25_dirty_check / hook_25_dirty_clear / hook_25_dirty_set / retry_guard_fail / retry_guard_reset |
| 2. engineering-principles.md | ✅ 通过 | 文件 92 行，头部含 <project> 本地化说明段；`grep DRY` 仅 2 处（line 5 本地化标题、line 92 来源说明），主体无 DRY 段落 |
| 3. code-quality.md（含 3 次失败硬规则 + 30 分钟时间锁 + 删比加便宜 + 跟 retry_guard 协作） | ✅ 通过 | 关键词全部命中：复杂度反向(1) / 改实现不改测试(1) / LLM 不听话先查 prompt(1) / 删比加便宜(2) / 3 次失败硬规则(1) / 30 分钟时间锁(1) / WebSearch(1) / plan-eng-review(1) / 升级到设计窗口(1) / retry_guard(1) |
| 4. needs-research-first.md（含跟 feasibility_check 协作说明） | ✅ 通过 | 含完整三问框架(Q1/Q2/Q3 各 2 次)、feasibility_check 协作段(2 次)、3 个工业先例(Joel Spolsky/Bezos/Lean Startup)、<project> 实际案例段 |
| 4.5. user-context.md（用户背景 + 设计哲学） | ✅ 通过 | 关键词全部命中：并行文件系统 / 高性能计算 / 分布式存储 / 文件系统 > 向量检索 / 75 分够 / 先搜后做 / 协作分工表格 |
| 5. skill-eval 触发 | ✅ 通过 | **方案 ④ 实施**（设计窗口拍板，非工单原方案）：抄自 <project> 自家 `feasibility_check.sh` 模板，无 LICENSE 风险。不创建 .js 和 .json。模拟测试：`echo '{"prompt":"我要做一个新智能体"}' | bash hooks/skill-eval.sh` 输出含 "<project>-production-standards" 字串；不命中关键词时 exit 0 静默 |
| 6. 3 个校验 hook（错误信息用"智能体X"） | ✅ 通过 | 三文件存在 + chmod +x。10 个单元测试 100% 通过：缺 notify_startup → exit 2、_test_dryrun 豁免 → exit 0、utility.py 不匹配 → exit 0、含 notify_startup → exit 0、time.sleep(300) → exit 1、timeout=30 → exit 0、缺 refuse_if_agent4_locked → exit 2、agent4_make.py 豁免 → exit 0。错误信息全部用"智能体X"措辞 |
| 7.0 备份现有 settings.json | ✅ 通过 | `~/.claude/settings.json.bak.20260503` 存在，4041 字节 |
| 7. Edit settings.json 追加 hook（**现有 7 个 hook 一字不动**） | ✅ 通过 | `python3 -c "json.load(...)"` 解析无错；clean diff 仅含 `+` 行无 `-` 行；现有 6 hook 全 OK + 新 4 hook 全 OK（注：工单原文写"7 个现有 hook"，实际 settings.json 中是 6 个独立脚本 + 1 个 SessionStart inline 命令，共 7 个 hook 项；6 个独立脚本均完整保留，inline 的 SessionStart 也未动） |
| 8. SKILL.md 占位 | ✅ 通过 | frontmatter `name=<project>-production-standards`(25 字符 OK，正则 OK)、`description` 987 字符 OK；含"占位"和"阶段 2"字样；运行时已被 Claude Code 识别（system-reminder 自动列出 `<project>-production-standards` skill） |
| 9.1 触发测试（桌面证据） | ✅ 通过 | `~/Desktop/工程标准化测试_20260503/test_9_1_trigger.txt` 含 `<project>-production-standards` 字样 2 次 |
| 9.2 拦截测试（mock，桌面证据） | ✅ 通过 | `test_9_2_block_stderr.txt` 含 verify-notify-startup 错误信息 + "智能体" 字样 + `Exit code: 2` |
| 9.3 豁免测试（mock，桌面证据） | ✅ 通过 | `test_9_3_exempt_stdout.txt` 末尾 `Exit code: 0` |
| 9.4 现有 hook 不被破坏（桌面证据） | ✅ 通过 | `test_9_4_feasibility.txt` 含可信 feasibility_check 输出；`test_9_4_hook_inventory.txt` 现有 6 hook 全 OK + 新 4 hook 全 OK |

### 6.2 集成测试结果

**9.1 触发测试**：本会话直接调 skill-eval.sh 模拟 UserPromptSubmit。`{"prompt":"我要做一个新智能体"}` 输入下，hook 输出"命中场景 A（新建智能体），关键词：新智能体"+"请激活 ~/.claude/skills/<project>-production-standards/SKILL.md"。同时 feasibility_check.sh 在该 prompt 下 exit 0 不重复触发（"新智能体" 不命中可行性守门关键词），两 hook 行为正交不冲突。

**运行时验证（额外证据）**：步骤 9 跑完后系统 system-reminder 自动加载了 `<project>-production-standards` skill 的 description，证明 SKILL.md 已被 Claude Code 运行时正确识别和加载。

**9.2 拦截测试**：mock JSON 模拟 PreToolUse Write 事件写 `Skills/agent_demo.py`（含 `def main` + `__main__` 但缺 `notify_startup`）。verify-notify-startup.sh stderr 输出标准错误信息 + exit 2，符合 block 语义。

**9.3 豁免测试**：同样内容，仅文件名改为 `agent_demo_test_dryrun.py`。hook exit 0 放行，证明 `_test_` / `_dryrun` 豁免逻辑生效。

**9.4 现有 hook 不破坏**：feasibility_check.sh 仍能正常对"测试" prompt 输出可行性守门提示（命中"测试"关键词，不在 skip 列表）；inventory 检查 6 个现有 hook 名 + 4 个新 hook 名全在 settings.json hooks 字段中。

### 6.3 过程中发现的问题

**问题 1（C 类升级，已与设计窗口对齐 → 改方案 ④）**：ChrisWiles/claude-code-showcase 仓库**无 LICENSE 文件**（curl 返回 404），按工单 §5.0a 规则不能直接抄。设计窗口拍板改用方案 ④：抄 <project> 自家 `feasibility_check.sh` 作模板，改关键词为 <project> 三场景 A/B/C。**附带消除问题 2**（skill-rules.json 实际 schema 跟工单 §5.3 简化版本不一致），因为方案 ④ 不再需要 JSON 配置文件，规则直接写进 .sh。

**问题 1 带来的偏离**（已被设计窗口接受，记录于此）：
- 步骤 5 不创建 `skill-eval.js` 和 `skill-rules.json`，只产 `skill-eval.sh`（自含规则）
- 步骤 5 Pass 条件按设计窗口新拟：模拟输入命中关键词时输出含 `<project>-production-standards` 字符串

**附录 §A 表 A.1 第 1 项已自动落实**：抄 engineering-principles.md 时按工单删除了 DRY 段，仅保留 YAGNI/KISS/SOLID 等。Modularity 段中"Shared utilities belong in a `common/` or `core/` layer — not duplicated in each feature"这条 DRY-flavored 建议保留原文，但在头部 <project> 本地化说明里加了一句"在 <project> 语境下读作'判断需要时再做'"，避免与"复制比抽象便宜"原则正面冲突。

**字段名核实结果**（步骤 5.0b、6.0）：本次直接复用 `feasibility_check.sh` 的 jq 解析模式，prompt 字段名为 `prompt`；3 个 verify hook 用 python3 解析 PreToolUse JSON，字段为 `tool_name` / `tool_input.file_path` / `tool_input.content`（Write）/ `tool_input.new_string`（Edit），跟 Anthropic 官方文档一致。

### 6.4 git 提交记录

`~/.claude/` **不在 git 管理下**（运行 `git status` 报 "not a git repository"），无法对本次 hook/rule/skill 文件提交。

`~/<project>/shared/核心，总体！/` commit：
- **hash**：`e6eaa65`
- **信息**：`engineering-standards/phase1: 搭建 .claude/ 骨架完成报告填写`
- **范围**：仅 add 本工单文件（826 insertions），未触碰 repo 内其他未提交变更（删除的 prompts/p_*v_storyboard.md 等是别人未完成的工作，搭建窗口不动）。

**建议**：阶段 2 之前考虑把 `~/.claude/` 中 <project> 自有的内容（rules/、hooks/、skills/<project>-production-standards/）建一个 git repo 或软链到现有 git repo，便于追踪后续变更——本条作为可选优化交设计窗口决定，阶段 1 不做。

---

## 附录 §A：现有规则冲突消解清单（14 处）

> **改 rule、写 hook、写测试时遇到边界情况，先翻这表**。
> 设计窗口与用户已就以下消解方案达成共识，搭建窗口不要重新讨论。

### A.1 直接冲突（已消解）

| # | 新规则 | 现有规则 | 消解方式 |
|---|---|---|---|
| 1 | DRY | design_window.md "复制比抽象便宜" | **不抄 DRY**，改抄"先搜后做"原则到 needs-research-first.md |
| 2 | "资源让位"（18 项第 17 条） | CLAUDE.md "agent4 强契约" | 让位策略写 PROJECT.md，skill 只写通用框架，不在 hook 里硬编码 |
| 3 | "降级路径" | CLAUDE.md "LLM 不听话先查 prompt 不加兜底" | **删降级路径**，加 LLM-prompt-first（第 18 项）和 V1/V2 兜底（第 19 项） |
| 4 | "主动思考机制" | design_window.md "认知偏差警告" | 改"双标原则"为"统一 75 分标准"——对所有方面都按 75 分（工程完整但不极致 polish） |

### A.2 弱化项（新规则吸收旧规则）—— 暂不删，等 agent1 V3 验证后处理

| # | 新方案接管 | 现有规则要改的位置 |
|---|---|---|
| 5 | hook verify-notify-startup.sh | CLAUDE.md "强制启动通知"段（暂留） |
| 6 | hook verify-agent4-lock.sh | CLAUDE.md "agent2/3 主动调 refuse_if_agent4_locked"段（暂留） |
| 7 | hook check-hardcoded-timeout.sh | CLAUDE.md "超时按公式算"段（暂留） |
| 8 | skill C 场景 | CLAUDE.md "重构前必须先打 Tag"（暂留） |
| 9 | skill B 场景 | PROJECT.md §8 "升级时必须 Skills/ 目录重组"（暂留） |
| 10 | skill 18 项基线 | PROJECT.md §0 第 4 条"必须实现监控机制"（暂留） |

### A.3 重复项（措辞统一）

| # | 新规则 | 旧规则 | 统一为 |
|---|---|---|---|
| 11 | 18 项第 3 条"failed.txt" | CLAUDE.md "通知规范" | 以 PIPELINE.md 路径规范为单一真源 |
| 12 | 18 项第 6 条"重试上限" | CLAUDE.md 第 75 行"3 次未解决停止" | 统一 max_retries=3 + 指数退避 + 超限写 failed.txt |
| 13 | 18 项第 14 条"凭证不入代码" | PROJECT.md §7 + CLAUDE.md "新加 Key 同步" | 统一为"凭证走 keys.json，新加 key 同步 PROJECT.md §7" |
| 14 | 抄的 code-quality.md "防创可贴" | build_window.md "改实现不改测试" + "复杂度反向" | 步骤 3 已要求把 build_window.md 例子吸收进 code-quality.md |

---

## 附录 §B：阶段 2 不在本工单范围（防越界做了下一阶段的事）

以下都是阶段 2 工单的事，**阶段 1 严禁实现**：

- 19 项基线标准的实质内容（每项的"为什么/怎么落地/反向审计点"）
- 75 分质量标准定义段
- 主动思考补全机制
- A/B/C 三场景的具体流程（设计摘要模板等）
- B 场景反向审计逻辑
- Gotchas 区块（基于 PROJECT.md §5 + CLAUDE.md 真实坑）
- 让位策略的具体规则（要写 PROJECT.md 不是 skill）

阶段 1 只产出：目录骨架 + 抄的 2 个 rule + 自写 1 个 rule + 触发 hook + 3 个校验 hook + skill 占位文件 + settings.json + 端到端测试通过。
