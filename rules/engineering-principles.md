---
<project> 本地化说明（重要，先读）
---

> **本项目不采用 DRY 原则**
> 理由：<project> 是单人项目 + 4 个 agent 规模 + 高频迭代。
> 抽象出公共框架的成本 > 复制几份的成本。
> 单文件内部仍要避免复制粘贴 30 行（常识）；
> 跨文件、跨 agent 的"抽象出公共框架"——禁止。
>
> 下文 "Modularity & Reuse" 段中关于"共享工具应集中在 common/core 层"的建议，
> 在 <project> 语境下读作"判断需要时再做"，不是"先建抽象层"。
> 默认选择：复制 2-3 份具体实现，等第 4 个相似用例出现时再考虑提取。

---

# Engineering Principles

Core software engineering principles that apply to every part of the codebase.

## YAGNI — You Aren't Gonna Need It

- Don't build for hypothetical future requirements — build for what's needed now
- Don't add configuration options "in case someone needs it later"
- Don't build abstractions for a single use case — wait until you have 2-3 concrete cases before generalizing
- Features that "might be useful someday" cost maintenance today and may never be used
- If you're wrong and do need it later, building it then (with real requirements) produces a better design than guessing now

## KISS — Keep It Simple

- The simplest solution that meets the requirements is the best solution
- Complexity must be justified by concrete requirements, not theoretical benefits
- If you can't explain your solution in a short paragraph, it's probably too complex
- Clever code is not good code — readable, obvious code is good code
- When choosing between two approaches, prefer the one that's easier to understand and debug

## Single Responsibility Principle

- Every function does one thing and does it well
- Every class/module has one reason to change
- If a function name contains "and" (e.g., `validateAndSave`), it's doing too much — split it
- A function should fit on one screen (~30 lines). If longer, extract sub-functions with clear names

## Code Smells — Recognize and Fix

- **Long functions** (>40 lines): Break into named sub-functions
- **Deep nesting** (>3 levels): Use early returns, extract helpers, or restructure
- **God objects**: Classes/modules that know or do too much — split by responsibility
- **Feature envy**: A function that uses more data from another module than its own — move it
- **Primitive obsession**: Passing raw strings/numbers everywhere instead of domain types (e.g., `user_id: str` everywhere instead of a `UserId` type)
- **Shotgun surgery**: One change requires edits across many files — indicates poor cohesion, consolidate related logic
- **Boolean blindness**: Functions with multiple boolean params (`fn(true, false, true)`) — use named options or separate functions

## Modularity & Reuse

- Design modules with clear boundaries and minimal interfaces — the less one module knows about another, the better
- Expose only what's needed: public API should be small, internal implementation can be complex
- Shared utilities belong in a `common/` or `core/` layer — not duplicated in each feature
- Reusable code must be genuinely reusable: if it requires 5 parameters and 3 config flags to work for different callers, it's not reusable — it's coupled

## Naming

- Names should reveal intent: `getActiveUsersByGroup` — not `getData` or `process`
- Functions: verb + noun — `calculateTotal`, `sendNotification`, `validateEmail`
- Booleans: `is_`, `has_`, `can_`, `should_` prefix — `isActive`, `hasPermission`, `canEdit`
- Constants: UPPER_SNAKE_CASE — `MAX_RETRY_COUNT`, `DEFAULT_PAGE_SIZE`
- Avoid abbreviations unless universally understood (`id`, `url`, `api` are fine; `usr`, `msg`, `btn` are not)
- Name things for what they represent, not how they're implemented: `userCache` — not `redisHashMap`

## Composition Over Inheritance

- Prefer composing objects/functions from smaller pieces over building deep class hierarchies
- Inheritance creates tight coupling — changes to parent classes ripple to all children
- Use interfaces/protocols to define contracts, composition to assemble behavior
- Mixins and multiple inheritance create complexity — prefer explicit delegation

## Dependency Injection

- Functions/classes should receive their dependencies as parameters, not create them internally
- This makes code testable (inject mocks), flexible (swap implementations), and explicit (dependencies are visible in the signature)
- Framework-provided DI (FastAPI `Depends`, Spring `@Inject`, etc.) is preferred over manual wiring

## Defensive Programming — At Boundaries Only

- Validate at system boundaries: user input, API requests, external service responses, file reads
- Trust internal code: don't re-validate data that was already validated at the boundary
- Internal functions can assume their callers send valid data — that's the caller's contract
- Excessive internal validation adds noise and hides the actual business logic

---

> 来源：smartwhale8/claude-playbook (MIT License)，已按 <project> 本地化删除 DRY 段并加说明头。
