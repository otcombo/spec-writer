# Spec Templates

Three tiers of spec templates. Select tier based on complexity assessment.

## Marking Syntax

Use these tags for incomplete information:

```
[ASSUMPTION: <reasoning>. <assumed value/behavior>]
[TBD: <what's missing>. <suggested default if any>]
[BLOCKED: <what's missing>. <why it blocks implementation>]
```

---

## Light Template

Use when: single responsibility, no external dependencies, linear logic.

```markdown
# <Module Name>

<1-2 sentences: what this does and why it exists.>

## Scope

What this module covers:
- <capability 1>
- <capability 2>

## Data Model

### <TypeName>

| Field | Type | Required | Default | Description |
|-------|------|----------|---------|-------------|
| | | | | |

## Core Behavior

1. <step>
2. <step>
3. <step>

## Examples

### <Scenario Name>

Input:
```json
{ ... }
```

Output:
```json
{ ... }
```

## Done When

- [ ] <verifiable criterion — test command, observable output, or measurable condition>
- [ ] <verifiable criterion>

## Assumptions

- [ASSUMPTION: ...]
```

---

## Standard Template

Use when: multiple components interact, has configuration, has non-obvious error paths.
Includes all Light sections plus the following.

```markdown
## Non-Goals

What this module does NOT do. Be specific.

- Does NOT <specific thing the AI might try to add>
- Does NOT modify <existing module/file the AI should not touch>
- Does NOT add <capability not requested: caching, auth, logging, etc.>

## Constraints

- <Non-functional requirement: performance, security, compatibility>
- <Technical constraint: must use X library, must run on Y>
- <Business constraint: must comply with Z>

## Configuration

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| | | | |

## Error Handling

| Scenario | Behavior |
|----------|----------|
| <specific error condition> | <specific response/action> |
| <specific error condition> | <specific response/action> |
```

---

## Full Template

Use when: multi-module coordination, priority conflicts, state systems, modifying existing systems.
Includes all Standard sections plus the following.

```markdown
## Priority Resolution

<Explicit precedence chain for conflicting configurations/rules>

```
Priority (highest to lowest):
  <Source A> > <Source B> > <Source C>
```

## State / Lifecycle

| Current State | Event | Next State | Action |
|--------------|-------|------------|--------|
| | | | |

## Integration Points

| Module | Direction | Interaction | Reference |
|--------|-----------|-------------|-----------|
| <module name> | → (this calls it) | <how: API call, import, event> | <spec path or doc link> |
| <module name> | ← (it calls this) | <how> | <spec path or doc link> |

## Invariants

What must NOT break. Mandatory when modifying existing systems.

- Must NOT change: <existing API endpoint shape/response format>
- Must NOT break: <existing data format/schema>
- Must NOT alter: <permission/auth behavior>
- Must NOT remove: <currently used interface/export>

## Open Questions

- [TBD: ...]
- [BLOCKED: ...]
```

---

## Section Ordering

Sections should appear in this order within each tier:

**Light:** Purpose → Scope → Data Model → Core Behavior → Examples → Done When → Assumptions

**Standard:** Purpose → Scope → Non-Goals → Data Model → Core Behavior → Constraints → Configuration → Error Handling → Examples → Done When → Assumptions

**Full:** Purpose → Scope → Non-Goals → Data Model → Core Behavior → Priority Resolution → State/Lifecycle → Constraints → Configuration → Error Handling → Integration Points → Invariants → Examples → Done When → Assumptions → Open Questions

## Tips for Each Section

**Purpose**: One sentence for what, one for why. No background history.

**Scope**: Bullet list of capabilities. If something is borderline in/out, put it here or in Non-Goals.

**Non-Goals**: The most important section for preventing AI over-implementation. Write it as if the AI will try to do each thing you list — because it probably will.

**Data Model**: Default to language-agnostic field tables. Include type constraints (min/max, regex, enum values) inline. If the implementation language is already decided, you may use typed definitions (TypeScript interface, Python dataclass) instead.

**Core Behavior**: Use numbered steps for linear flows. Switch to pseudocode for branching. Switch to state table for stateful objects. See format-rules.md.

**Priority Resolution**: Only include when multiple configuration sources exist. State the full chain on one line.

**Configuration**: Every config field must have a sensible default or be marked required.

**Error Handling**: Every row must specify the exact behavior, not "handle gracefully" or "log and continue". State the HTTP status, the error message, the retry behavior, or the fallback.

**Integration Points**: State direction (who calls whom), mechanism (HTTP, import, event), and link to the other module's spec.

**Invariants**: Write as "Must NOT" statements. Be concrete: name the API endpoint, the data field, the permission check.

**Examples**: Every example must be copy-paste ready. No `...`, no `// TODO`, no `<placeholder>`. Use realistic but simple data. Exception: for inherently variable-length content (HTML body, large binary), use a short realistic snippet (3-5 lines) rather than truncation markers.

**Done When**: Every criterion must be verifiable. Prefer: "run `make test` and all pass", "POST /api/x returns 201", "dashboard shows Y". Avoid: "code is clean", "works correctly".

**Assumptions**: Include reasoning. "[ASSUMPTION: No auth required because this is an internal service behind the VPN]" is better than "[ASSUMPTION: No auth]".
