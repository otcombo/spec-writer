---
name: spec-writer
description: Generate AI-agent-friendly spec documents from verbal descriptions or existing requirement docs. Use this skill whenever users discuss requirements, features, or system designs they want formalized — even if they don't say "spec" explicitly. Triggers include "写spec", "spec文档", "write spec", "需求讨论", "specification", "设计文档", "write a spec for", "帮我理清需求", "这个功能怎么设计", "PRD", "把需求整理一下", or any conversation where the user is describing what to build and would benefit from a structured document before implementation begins.
metadata:
  version: 1.0.0
---

# Spec Writer

Generate spec documents optimized for AI agent comprehension and execution.

## Core Principle

**Draft-first, not interview-first.** Generate a v0 spec immediately from whatever the user provides. Mark gaps with tags instead of asking questions. Only ask when information is truly blocking.

## Execution Flow

### Step 1: Identify Input Mode

Determine what the user provided:

- **Verbal description only** → Oral mode. Proceed to Step 2.
- **File path or pasted document** → Document mode. Read the document first, then proceed to Step 2 using extracted content.

### Step 2: Extract Four Core Items

From the user's input (or document), extract:

| Item | What to extract | If missing |
|------|----------------|------------|
| **Goal** | What does this do? Why build it? | If completely unclear → ask (this is the only blocking item) |
| **Inputs/Outputs** | What goes in, what comes out, data shapes | Mark `[ASSUMPTION]` based on context |
| **Core Behavior** | Normal flow from trigger to completion | Mark `[ASSUMPTION]` for inferred steps |
| **Done When** | How to verify correctness | Infer from Goal if not stated |

These four items are the **minimum for generating v0 spec**. Do not ask about anything else at this stage.

### Step 3: Determine Complexity

Assess which tier applies:

```
if modifies existing system with invariants
   OR multi-module coordination
   OR priority conflicts between config sources
   OR complex state machine:
     → Full

elif multiple components interact
   OR has configuration options
   OR has non-obvious error branches:
     → Standard

else:
     → Light
```

Present the assessment to the user:

```
Complexity: [Light / Standard / Full]
Sections included: [list]
Sections skipped: [list with reason]
```

User can override ("add error handling" → add it).

Why tier matters: AI agents tend to over-generate sections when given a full template. A Light spec with 6 sections keeps the agent focused on a single responsibility; a Full spec with 16 sections provides guardrails for complex systems where missing a priority conflict or invariant can cause production bugs. Choosing the right tier balances thoroughness against noise.

### Step 4: Generate v0 Spec

Generate the spec immediately using the appropriate tier template from [references/spec-template.md](references/spec-template.md).

**Marking rules for incomplete information:**

| Tag | Meaning | When to use | Action |
|-----|---------|------------|--------|
| `[ASSUMPTION: ...]` | Reasonable inference from context | When you can make a sensible guess | No question asked. User corrects if wrong. |
| `[TBD: ...]` | Information missing, not blocking | Implementation can proceed with a reasonable default | No question asked. |
| `[BLOCKED: ...]` | Information missing, blocks implementation | Cannot proceed without this answer | Ask the user — but ONLY these items. |

**Output the v0 spec as a file** to the project's spec directory (or ask the user where to save it).

### Step 5: Iterate

After user reviews v0:

- User corrects assumptions → update the section
- User fills in TBD items → replace tags with content
- User requests section changes → add or remove sections
- User confirms → output final version

Each iteration overwrites the same file. Keep changes minimal and targeted.

## Format Selection Rules

Choose the least ambiguous format for each information type. See [references/format-rules.md](references/format-rules.md) for the full table.

Key rules:
- **Data structures** → Language-agnostic field tables by default. Use target language types (interface/dataclass) only when the implementation language is already decided.
- **Linear flow** → Numbered steps.
- **Branching logic** → Pseudocode if/else or match/case. NOT Mermaid flowcharts.
- **Stateful objects** → State transition table: `State | Event | Next State | Action`.
- **Priority conflicts** → Single-line explicit chain: `A > B > C`.
- **Failure cases** → `Scenario | Behavior` table. Exhaustive.
- **API contracts** → Field table + one concrete JSON example.
- **Architecture** → Component list with dependency directions in text. Mermaid is optional, only when user explicitly wants a human-readable diagram.

## Complexity Tier Details

### Light (single responsibility, no external dependencies, linear logic)

Sections: Purpose, Scope, Data Model, Core Behavior, Examples, Done When, Assumptions

Use when: utility function, simple endpoint, standalone script, single-file module.

### Standard (Light + multi-component, has config, has error branches)

Additional sections: Non-Goals, Constraints, Configuration, Error Handling

**Non-Goals is mandatory at Standard and above.** Research shows 26% of AI agent commits are unrequested refactoring. This section prevents over-implementation.

Non-Goals should explicitly state:
- What NOT to build
- What existing code NOT to modify
- What NOT to add (caching, auth, logging, etc.) unless requested

### Full (Standard + multi-module, priority conflicts, state systems, modifying existing systems)

Additional sections: Priority Resolution, State/Lifecycle, Integration Points, Invariants, Open Questions

**Invariants is mandatory when modifying existing systems.** State what must NOT break:
- Existing API shape
- Data format compatibility
- Permission/auth behavior
- External contract obligations

## Example

User: "我要做一个 webhook 转发服务，接收 GitHub webhook 然后转发到飞书群"

**Step 1** — Input mode: Verbal description → Oral mode.

**Step 2** — Extract core items:
- **Goal**: Relay GitHub webhook events to Feishu group chat
- **Inputs/Outputs**: GitHub POST payload → Feishu bot message
- **Core Behavior**: Receive → validate → transform → forward → respond
- **Done When**: `curl -X POST /webhook` with sample payload returns 200, Feishu group receives formatted message

**Step 3** — Complexity: **Light** (single service, linear flow, no config conflicts)

**Step 4** — Generate v0 spec:

```markdown
# GitHub-Feishu Webhook Relay

Receives GitHub webhook events and forwards formatted notifications to a Feishu group bot.

## Scope
- Accept GitHub webhook POST requests
- Validate webhook signature
- Transform payload to Feishu message card format
- Forward to Feishu bot via incoming webhook URL

## Data Model

### WebhookPayload (inbound)

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| action | string | yes | Event action (opened, closed, etc.) |
| repository | object | yes | Repo name, URL |
| sender | object | yes | Who triggered the event |

[ASSUMPTION: Only push and pull_request events needed, based on user mentioning "webhook" generically]

## Core Behavior

1. Receive POST at `/webhook`
2. Validate `X-Hub-Signature-256` header against secret
3. Parse event type from `X-GitHub-Event` header
4. Transform payload into Feishu message card JSON
5. POST to Feishu bot webhook URL
6. Return 200 to GitHub

## Done When

- [ ] `curl -X POST /webhook` with valid signature returns 200
- [ ] Feishu group receives formatted message within 3 seconds
- [ ] Invalid signature returns 401

## Assumptions

- [ASSUMPTION: Single Feishu group target. Based on user saying "飞书群" (singular)]
- [ASSUMPTION: No retry on Feishu delivery failure. Not mentioned by user]
```

**Step 5** — User reviews, corrects assumptions, iterates.

## Quality Checks

Before delivering any spec version, verify:

- [ ] Every section uses the format from format-rules.md (not prose where a table fits)
- [ ] All code/JSON examples are copy-paste ready (no placeholders like `...` or `TODO`)
- [ ] No section repeats information from another section
- [ ] ASSUMPTION tags include the reasoning ("based on X, assuming Y")
- [ ] Non-Goals section (if present) is specific, not generic ("don't refactor auth" not "don't do extra work")
- [ ] Done When criteria are verifiable (testable commands, observable outputs)

## Language

Write the spec in the same language the user uses. If the user speaks Chinese, write in Chinese with technical terms in English (URL, HTTP, API, etc.). If the user speaks English, write entirely in English. If the user mixes languages (e.g., Chinese with English technical jargon), follow the dominant language for prose and keep technical terms in their original form. When in doubt, match the language of the user's most recent message.

## What This Skill Does NOT Do

- Does not implement the spec (that's a separate task)
- Does not generate project scaffolding or boilerplate
- Does not create CLAUDE.md or AGENTS.md files
- Does not manage spec versioning or history
- Does not ask more than 1 round of clarifying questions before generating v0 — draft first, iterate later
- Does not generate code alongside the spec (examples in spec use pseudocode or JSON, not runnable code)
- Does not expand scope beyond what the user described (no unsolicited "you should also consider..." sections)
