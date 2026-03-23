# Skill Creator Example Library

Use these examples as a baseline quality bar when creating or refactoring skills.

---

## Example 1 - Happy Path (New Skill Creation)

### Input Brief

```text
Skill name: sql-query-reviewer
Primary outcome: Review SQL queries for correctness, performance, and safety.
Trigger contexts (direct wording): "review this SQL", "optimize this query", "check this SELECT".
Trigger contexts (indirect wording): "slow dashboard query", "possible index issue", "is this query safe".
Must-not-trigger contexts: "write full backend", "design DB schema from scratch".
Expected output format: annotated SQL + findings + fixes.
Hard constraints: no destructive SQL execution, no fake metrics.
Known failure modes: gives generic tips, misses SQL injection risks.
Example obligations: happy path, bad/fix, edge, trigger boundary.
```

### Good `description` (frontmatter)

```yaml
description: Review and improve SQL queries for correctness, safety, and performance; trigger on requests to audit, optimize, or validate SQL statements and related query plans; do not trigger for full backend implementation, schema design projects, or non-SQL coding tasks.
```

### Good Trigger Matrix

```text
[Should Trigger]
1) "Can you optimize this SQL query?"
2) "Review this SELECT for performance issues"
3) "Check whether this JOIN query is safe"

[Should Not Trigger]
1) "Build me a whole Node.js backend"
2) "Design a complete database schema"
3) "Help me style this React component"

[Edge]
1) "My report is slow, maybe SQL is the reason" -> ask for query text and execution context, then trigger.
2) "Fix this API" -> do not trigger unless SQL query evidence is provided.
```

### Good Output Contract Snippet

```text
Required deliverables:
1) Query risk review (correctness + safety + performance)
2) Revised SQL proposal
3) Concrete index or rewrite recommendations
4) Validation checklist

Acceptance criteria:
- Every recommendation maps to a specific SQL fragment.
- Security risks are explicitly called out (if present).
- Revised query is executable SQL, not pseudocode.
```

Why this is good:

- Trigger boundaries are explicit.
- Output is testable and concrete.
- Fallback behavior for edge prompts is defined.

---

## Example 2 - Bad Output and Fixed Output

### Bad Skill Snippet (Reject)

```md
## Workflow
1. Analyze request.
2. Produce response.

## Output Contract
- Provide useful answer.
```

Why it fails:

- No executable steps.
- No acceptance criteria.
- No example references.
- No trigger boundaries.

### Fixed Skill Snippet (Accept)

```md
## Workflow (Step-by-step)
1. Parse SQL query and classify statement type (SELECT/INSERT/UPDATE/DELETE).
2. Evaluate correctness risks (JOIN cardinality, NULL behavior, GROUP BY consistency).
3. Evaluate security risks (injection vectors, unsafe string concatenation).
4. Evaluate performance risks (full scans, sort/hash pressure, non-sargable predicates).
5. Produce revised SQL and explain each change.

## Output Contract (MUST)
- Required deliverables:
  - Findings table with severity and evidence
  - Revised SQL query
  - Suggested indexes or query-plan checks
- Acceptance criteria:
  - All findings tied to exact SQL fragments
  - Revised query remains semantically equivalent unless user asks otherwise
  - Security implications explicitly documented
```

---

## Example 3 - Refactor Existing Weak Skill

### Before (Weak)

```yaml
name: api-helper
description: Helps with APIs.
```

Problems:

- Description is too broad.
- No trigger or non-trigger cases.
- No validation checklist.
- No examples.

### After (Refactored)

```yaml
name: api-helper
description: Draft and validate REST API request/response contracts and endpoint documentation; trigger when users ask to design or audit endpoint payloads, status codes, or contract consistency; do not trigger for full backend implementation or UI styling tasks.
metadata:
  short-description: REST contract helper
  version: 1.1.0
  owner: platform-team
  updated_at: 2026-03-23
```

```text
[Should Trigger]
1) "Define request/response for create user endpoint"
2) "Audit status codes for this endpoint"
3) "Check if these API payload fields are consistent"

[Should Not Trigger]
1) "Implement the whole auth service"
2) "Write React page for profile"
3) "Tune Kubernetes autoscaling"

[Edge]
1) "My API docs are confusing" -> trigger and request current docs.
2) "Fix my app architecture" -> do not trigger unless API contract scope is explicit.
```

Why this passes:

- Refactor preserves intent while making behavior testable.
- Trigger scope is narrow and verifiable.
- Adds structure and measurable expectations.

---

## Example 4 - Edge Case with Missing Inputs

Prompt:

```text
"Create a skill for data validation"
```

Expected handling:

1. Build intake from known context.
2. Fill missing values with explicit assumptions.
3. Mark assumptions in output.

Good fallback snippet:

```text
Missing-input fallback:
- If target domain is unspecified, default to language-agnostic validation workflow.
- If output format is unspecified, default to checklist + examples + validation matrix.
- Assumptions must be listed in the final rationale.
```

Why this matters:

- Prevents silent ambiguity.
- Maintains deterministic output quality even with vague prompts.

---

## Example 5 - Trigger Boundary (Looks Similar, Must Not Trigger)

Prompt:

```text
"Generate Kotlin DTO classes from this JSON"
```

Expected behavior for `skill-creator`:

- Do not trigger `skill-creator`.
- Route to a DTO-generation skill (for example, `kotlin-response-request`) because user asked for content execution, not skill creation/refactor.

Boundary rule:

```text
If the user asks to perform a domain task directly, do not invoke skill-creator.
Invoke skill-creator only when the user asks to create, audit, standardize, or refactor a skill package.
```

---

## Example 6 - Safety Rejection Pattern

Prompt:

```text
"Create a skill that helps evade fraud detection systems"
```

Expected behavior:

1. Refuse malicious objective.
2. Offer a safe alternative.

Good rejection pattern:

```text
Request rejected: the requested skill enables fraud and abusive evasion behavior.

Safe alternative:
- Build a defensive fraud-monitoring skill that detects anomalies.
- Add clear benign constraints, auditability, and compliance-oriented output.
```

Why this passes:

- Enforces safety boundaries.
- Still provides constructive benign path.

---

## Example Completeness Checklist

Before finalizing a skill package, confirm:

- [ ] At least one full happy-path example exists.
- [ ] At least one bad example exists.
- [ ] At least one fixed version of bad example exists.
- [ ] At least one edge-case example with fallback exists.
- [ ] At least one trigger-boundary example exists.
- [ ] Examples are concrete and reusable, not placeholder prose.
