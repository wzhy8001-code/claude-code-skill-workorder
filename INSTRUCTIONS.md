# How to use these workorders

> Audience: a coding agent (Claude Code, Cursor, Codex, etc.) **and** the
> non-coding human directing it.

## TL;DR

1. **Human**: Adapt `rules/user-context.template.md` for your project, save it as `~/.claude/rules/user-context.md`.
2. **Human**: Read `workorders/01-scaffold.md`, replace the project-specific examples with yours.
3. **Human**: Open a fresh Claude Code session, paste the adapted workorder, say *"You are the implementer. Follow this workorder. Ask me questions where you need decisions."*
4. **Agent**: Reads workorder, asks questions, builds `.claude/` directory, runs the tests in the workorder, writes evidence to `~/Desktop/test-evidence-YYYYMMDD/`.
5. **Human**: Reviews evidence (file paths in workorder §6), approves or sends back for fixes.
6. **Repeat** for `workorders/02-skill-content.md` (writes the actual SKILL.md content).

## Phase 1: scaffold

What gets built:

- `~/.claude/rules/` directory with 3 rule files (copy from `rules/` in this repo) + your adapted `user-context.md`
- `~/.claude/hooks/` directory with 4 shell scripts (1 keyword-trigger + 3 validators)
- `~/.claude/skills/<your-skill-name>/SKILL.md` placeholder
- `~/.claude/settings.json` updated to register the 4 new hooks (without breaking your existing hooks)

What you (the human) decide during scaffold:

- **Skill name** (e.g., `<project>-production-standards`, or whatever fits your project)
- **Trigger keywords** for your project's "new agent / upgrade / refactor" terms
- **Whether to install globally (`~/.claude/`) or per-project (`<project>/.claude/`)**
- **Names of any `verify-*` validation hooks** specific to your project's hard rules

Estimated time: agent ~30-45 minutes, human ~5 minutes of decisions + 2 minutes of review.

## Phase 2: skill content

After scaffold passes, the SKILL.md is just a placeholder. Phase 2 fills it.

What gets written into SKILL.md:

- **Top-line hard rule**: "B-scenario reverse audit MUST use Read tool on existing code"
- **Quality definition**: 0/1/75/100 score scale (0 = doesn't run, 75 = production complete, 100 = polished — pick your target)
- **Agent type taxonomy**: 4 types (one-shot / short LLM / long no-LLM / long + LLM + GPU)
- **3-scenario flow**: A (new), B (upgrade + reverse audit), C (large refactor with tag)
- **22 baselines in 5 tiers** with examples for each
- **Active thinking gap-filling**: agent must explicitly think "what else is missing"
- **Gotchas**: 6 real incidents that hooks/skill prevent
- **Design summary template**: structured output the agent produces for human approval

What you decide during phase 2:

- **The 22 baselines content** — what does production-ready mean *for your project*? My human's list is 80% transferable (timeout config, retry limits, resource cleanup, etc.) but 20% project-specific (e.g., "must call `notify_startup`" — your project might have a different scheduler protocol).
- **Your real gotchas** — incidents your project actually had. Generic gotchas are useless.
- **Existing agent classification table** — list your real agents and which type each one is.

Estimated time: agent ~2-3 hours, human ~15 minutes of decisions + 10 minutes of review.

## How to test that it actually works

After both phases:

1. Open a **new** Claude Code session (not the one that built the skill).
2. Type: *"I want to upgrade `<your-existing-agent-name>` with `<new-feature>`."*
3. Watch the agent's first response. It should:
   - Match your trigger keywords (you'll see the hook output in the session)
   - Activate the skill (`Skill(...) Successfully loaded skill`)
   - Use `Read` tool on the existing agent's code (this is the reverse-audit hard rule)
   - List what's missing in the existing agent (your 22 baselines)
   - Ask if you want to fix the missing items in this upgrade
   - **Not write any code yet**

If the agent immediately starts writing code, the skill didn't activate. Check
your `~/.claude/settings.json` and the keyword list.

## What if my agent isn't Claude Code?

The workorders use Claude Code-specific concepts:

- `~/.claude/` directory layout
- `SKILL.md` frontmatter convention
- `settings.json` hook configuration

If you're on Cursor / Codex / Gemini CLI / OpenCode, the directory layout and
hook mechanism will differ. The **structure** (rules, hooks, skill, scaffold,
content) transfers. The **paths and config syntax** don't.

You'll need to adapt the workorder's commands to your agent's conventions. The
22-baselines list and 3-scenario flow are agent-agnostic.

## Recommended flow if you're starting fresh

Don't try to do everything in one session.

```
Day 1: Read workorder 01, fill out your user-context.md, decide your skill name
Day 2: Run phase 1 with your agent, review, approve
Day 3: Read workorder 02, decide your 22 baselines, decide your gotchas
Day 4: Run phase 2 with your agent, review
Day 5: Open a fresh session, do the trigger test
Day 6+: Use it in real work, iterate when something feels off
```

## When to *not* use this

- You write your own code and have an existing engineering standards process — this workorder is overhead for you.
- Your "project" is a single script — the 22-baseline tier system is overkill.
- Your team already has CI / linting / code review covering the same ground — duplication.

This workflow targets *non-coding humans running multi-agent pipelines* (or
coding humans who specifically want to delegate the engineering standards
layer). Outside that, simpler approaches exist.

## Source

These workorders were originally written for an AI video generation pipeline
(see `README.md` for context). They're lightly desensitized — agent names and
project specifics are kept as real examples; absolute paths are placeholdered.

Adapting them to your project takes ~30 minutes of reading + ~30 minutes of
deciding what to replace.
