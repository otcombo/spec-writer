# Format Selection Rules

Choose the least ambiguous format for each information type. This spec is consumed by AI agents, not humans — optimize for parsing accuracy and generation stability.

## Rules by Information Type

| Information Type | Recommended Format | When to Use Alternative | Alternative |
|-----------------|-------------------|------------------------|-------------|
| Data structures | Language-agnostic field table | Target language already decided | interface (TS) / dataclass (Python) |
| Linear flow | Numbered steps | Never | — |
| Branching logic | Pseudocode if/else or match/case | Never | — |
| Stateful objects | State transition table | Never | — |
| Priority conflicts | Single-line chain `A > B > C` | Never | — |
| Failure cases | Scenario/Behavior table | Never | — |
| API contracts | Field table + concrete JSON example | Never | — |
| Configuration | Markdown table (Field/Type/Default/Description) | Never | — |
| Architecture | Component list + dependency text | User wants human-readable version | Mermaid diagram (supplemental) |

## Format Examples

### Data Structure — Field Table (Default)

```
### UserProfile

| Field | Type | Required | Default | Description |
|-------|------|----------|---------|-------------|
| id | string (UUID) | yes | — | Unique identifier |
| name | string | yes | — | Display name, 1-100 chars |
| role | enum: admin, member, viewer | yes | — | Access level |
| created_at | datetime (ISO 8601) | yes | auto | Creation timestamp |
| preferences | object (see Preferences) | no | {} | User settings |
```

### Data Structure — Target Language (When Language Decided)

```python
@dataclass
class UserProfile:
    id: str          # UUID
    name: str        # 1-100 chars
    role: Role       # admin | member | viewer
    created_at: datetime  # auto-set
    preferences: Preferences = field(default_factory=Preferences)
```

### Linear Flow — Numbered Steps

```
1. Receive HTTP request with user_id parameter
2. Validate user_id is a valid UUID format
3. Query database for user record
4. If found: return 200 with user data
5. If not found: return 404 with error message
```

### Branching Logic — Pseudocode

```
match request.auth_type:
  case "api_key":
    validate api_key against store
    if expired → return 401 "Key expired"
    if invalid → return 401 "Invalid key"
    extract permissions from key metadata

  case "oauth":
    validate token with provider
    if token invalid → return 401 "Invalid token"
    extract permissions from token claims

  case _:
    return 400 "Unsupported auth type"
```

### State Transition Table

```
| Current State | Event | Next State | Action |
|--------------|-------|------------|--------|
| idle | start_requested | processing | Validate inputs, begin work |
| processing | work_complete | review | Notify reviewer |
| processing | error_occurred | failed | Log error, notify admin |
| review | approved | complete | Persist result, send notification |
| review | rejected | processing | Return to worker with feedback |
| failed | retry_requested | processing | Reset error state, retry |
```

### Priority Resolution

```
Configuration priority (highest to lowest):
  Request Override > Environment Variable > Config File > Default Value
```

### Failure Cases Table

```
| Scenario | Behavior |
|----------|----------|
| Database connection timeout | Retry 3 times with exponential backoff, then return 503 |
| Invalid input format | Return 400 with field-level error messages |
| Duplicate entry | Return 409 with existing resource ID |
| Upstream service unavailable | Return cached response if available (max 5min stale), else 502 |
| Permission denied | Return 403, do not reveal resource existence |
```

### API Contract — Field Table + Example

```
### POST /api/v1/tasks

#### Request

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| title | string | yes | Task title, 1-200 chars |
| assignee_id | string (UUID) | no | User to assign |
| priority | enum: low, medium, high | no | Default: medium |

#### Example

Request:
{
  "title": "Fix login timeout",
  "assignee_id": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
  "priority": "high"
}

Response (201):
{
  "id": "task_xyz789",
  "title": "Fix login timeout",
  "assignee_id": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
  "priority": "high",
  "status": "open",
  "created_at": "2026-04-07T10:30:00Z"
}
```

### Architecture — Text Description (Default)

```
## Architecture

Components:
- **API Gateway**: Receives HTTP requests, validates auth, routes to handlers
- **TaskService**: Business logic for task CRUD operations
- **NotificationService**: Sends email/push notifications on state changes
- **PostgreSQL**: Primary data store for tasks and users

Dependencies:
- API Gateway → TaskService (direct call)
- API Gateway → auth middleware (before handler)
- TaskService → PostgreSQL (async queries)
- TaskService → NotificationService (async, fire-and-forget)
- NotificationService → external email/push providers
```

## Anti-Patterns

| Don't | Do Instead |
|-------|-----------|
| Prose paragraphs describing data fields | Field table |
| Mermaid flowchart for simple if/else | Pseudocode |
| Vague error handling ("handle errors gracefully") | Exhaustive scenario table |
| Priority buried in prose ("X takes precedence over Y, which...") | `X > Y > Z` one-liner |
| Example with `...` or `// TODO` | Complete, runnable example |
| Architecture as only a Mermaid diagram | Text component list + dependencies |
