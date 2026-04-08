# spec-writer

A Claude Code skill that generates AI-agent-friendly spec documents from verbal descriptions or existing requirement docs.

## What it does

Turns loose requirements into structured specification documents optimized for AI agent comprehension and execution. It follows a **draft-first** approach — generates a v0 spec immediately from whatever you provide, marks gaps with tags instead of asking questions, and iterates from there.

## Install

```bash
git clone https://github.com/otcombo/spec-writer ~/.claude/skills/spec-writer
```

## Usage

Trigger the skill by discussing requirements, features, or system designs in Claude Code. Example prompts:

- "Write a spec for a webhook relay service"
- "帮我写个 spec，用户注册流程"
- "I need to formalize these requirements into a spec"
- `/spec-writer`

The skill auto-detects input mode (verbal description or existing document) and generates a spec file in your project.

## How it works

1. **Extract core items** — Goal, Inputs/Outputs, Core Behavior, Done When
2. **Assess complexity** — Light / Standard / Full tier
3. **Generate v0 spec** — using the appropriate tier template, marking unknowns with `[ASSUMPTION]`, `[TBD]`, or `[BLOCKED]` tags
4. **Iterate** — user reviews, corrects assumptions, fills gaps; each iteration updates the same file

## Complexity tiers

| Tier | When | Sections |
|------|------|----------|
| **Light** | Single responsibility, linear logic | Purpose, Scope, Data Model, Core Behavior, Examples, Done When, Assumptions |
| **Standard** | Multi-component, has config or error branches | Light + Non-Goals, Constraints, Configuration, Error Handling |
| **Full** | Multi-module coordination, state systems, modifying existing systems | Standard + Priority Resolution, State/Lifecycle, Integration Points, Invariants, Open Questions |

## Marking tags

| Tag | Meaning |
|-----|---------|
| `[ASSUMPTION: ...]` | Reasonable inference — no question asked, user corrects if wrong |
| `[TBD: ...]` | Info missing but not blocking — implementation can proceed with a default |
| `[BLOCKED: ...]` | Info missing and blocking — the skill will ask about these |

## Format rules

The skill chooses the least ambiguous format for each information type:

- **Data structures** → Field tables (or typed definitions when language is decided)
- **Linear flow** → Numbered steps
- **Branching logic** → Pseudocode if/else
- **State** → State transition tables
- **Priority conflicts** → Single-line chain (`A > B > C`)
- **Failure cases** → Scenario/Behavior tables
- **API contracts** → Field table + concrete JSON example

## Language

The spec is written in the same language you use. Chinese input produces Chinese specs (with English technical terms). English input produces English specs.

## License

MIT
