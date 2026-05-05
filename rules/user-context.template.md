# [PROJECT NAME] User Context

> **Template**: Replace bracketed sections with your actual content.
> This file teaches Claude Code who you are, what you know, what you don't do,
> and your design philosophy. It's read by SKILL.md when relevant.

## Who

- [Your role / company / background — e.g., "Co-founder of an HPC storage company, designed a parallel filesystem"]

## What I know (discuss at this depth, don't dumb it down)

- [Domain expertise — e.g., "distributed storage, NAS, metadata, RAID, parallel HPC"]
- [Architectural concepts you handle natively — e.g., "contracts, module boundaries, system design"]

## What I don't do

- [ ] Write code
- [ ] Review code (don't ask me to judge syntax or behavior correctness)
- [ ] Detailed design (I set direction and key decisions; Claude Code proposes designs, I approve)
- [ ] [Anything else specific to your role]

## Design philosophy (violating this gets pushback)

### Data management preferences

- [State your preferences — e.g., "filesystem > vector retrieval, parent docs point to child docs, no DBs unless needed"]

### Engineering preferences

- [e.g., "75 points is enough — complete engineering, but not Apple-tier polish"]
- [e.g., "Copy is cheaper than abstraction at small scale (< N use sites)"]
- [e.g., "Search before building — if no one's done it, reconsider the requirement"]

## Collaboration division

| Domain | I do | Claude Code does |
|---|---|---|
| Direction, goals, key decisions | Set | Doesn't decide |
| Architecture proposals | Approve | Proposes |
| Writing code | No | Yes |
| Reviewing code | No (don't ask me syntax) | Self-reviews, leaves evidence for me |
| Running tests + verifying results | No | Yes (writes evidence to a folder I can see) |
| Engineering standard components | Don't know them | Apply [PROJECT] standards automatically |

## Discussion style

- Use senior-level language. Don't explain what a [domain concept you know natively] is.
- For code-level details, give me a recommendation to approve, don't ask me to judge correctness.
- For architecture / data flow / resource scheduling, you can go deep — I'll keep up.

---

**How to use this file**: After filling it in, place it at `~/.claude/rules/user-context.md`.
SKILL.md will reference it via Read tool when relevant context is needed.
