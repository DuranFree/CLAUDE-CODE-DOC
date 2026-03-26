---
name: full-dev-workflow
description: End-to-end development workflow that takes a raw idea all the way to implemented, tested GitHub issues. Runs in stages with user confirmation at each gate. Covers architecture review, requirement discovery (grill-me), PRD writing, issue breakdown, and TDD implementation. Use when user wants to build a new feature from scratch, says "let's build X", or wants a structured end-to-end dev flow.
---

# Full Dev Workflow

A single entry point that chains five skills in sequence, pausing for your confirmation at each stage gate before continuing.

**Stage overview:**

```
[0] Architecture Review  →  Surface any structural debt before building
[1] Grill Me             →  Stress-test the idea, reach shared understanding
[2] Write a PRD          →  Turn shared understanding into a durable spec
[3] PRD to Issues        →  Turn the spec into a kanban board of vertical slices
[4] TDD                  →  Implement each issue with red-green-refactor
```

At the end of each stage, pause and ask: **"Ready to continue to [next stage]? (or type any changes first)"**

Do not proceed until the user confirms.

---

## Stage 0 — Architecture Review (optional but recommended)

> Run this before building anything new. A messy codebase produces messy AI output.

Explore the codebase like an AI would, surfacing architectural friction and opportunities to deepen shallow modules. A **deep module** (John Ousterhout) has a small interface hiding a large implementation — more testable, more AI-navigable.

### Process

**0.1 Explore**

Navigate the codebase organically. Note friction points:

- Where does understanding one concept require bouncing between many small files?
- Where are modules so shallow the interface is nearly as complex as the implementation?
- Where have pure functions been extracted just for testability, but real bugs hide in how they're called?
- Where do tightly-coupled modules create integration risk at the seams?
- Which parts are untested or hard to test?

The friction you encounter IS the signal.

**0.2 Present candidates**

Present a numbered list of deepening opportunities. For each:

- **Cluster**: which modules/concepts are involved
- **Why they're coupled**: shared types, call patterns, co-ownership of a concept
- **Dependency category**: in-process / local-substitutable / ports & adapters / true external (mock)
- **Test impact**: what existing tests would be replaced by boundary tests

Do NOT propose interfaces yet. Ask: *"Which of these would you like to explore, or shall we skip to Stage 1?"*

**0.3 (If user picks a candidate) Design multiple interfaces**

Spawn 3+ sub-agents in parallel using the Agent tool. Each produces a **radically different** interface for the deepened module:

- Agent 1: "Minimise the interface — aim for 1–3 entry points max"
- Agent 2: "Maximise flexibility — support many use cases and extension"
- Agent 3: "Optimise for the most common caller — make the default case trivial"

Each sub-agent outputs: interface signature, usage example, what complexity it hides, dependency strategy, trade-offs.

After comparing, give your recommendation. If elements combine well, propose a hybrid. Be opinionated.

**0.4 Create GitHub issue**

Create a refactor RFC as a GitHub issue with `gh issue create`. Use this template:

```
## Problem
- Which modules are shallow and tightly coupled
- What integration risk exists at the seams
- Why this makes the codebase harder to navigate

## Proposed Interface
- Interface signature (types, methods, params)
- Usage example
- What complexity it hides internally

## Dependency Strategy
- In-process / Local-substitutable / Ports & adapters / Mock

## Testing Strategy
- New boundary tests to write
- Old tests to delete
- Test environment needs

## Implementation Recommendations
- What the module should own
- What it should hide
- What it should expose
- How callers should migrate
```

---

**⏸ STAGE GATE 0→1**: *"Architecture review complete. Ready to move to Stage 1 (Grill Me)?"*

---

## Stage 1 — Grill Me

> Stress-test the idea before committing to paper.

Interview the user relentlessly about every aspect of the plan until reaching shared understanding. Walk down each branch of the design tree, resolving dependencies between decisions one-by-one.

**Rules:**

- For each question, provide your own recommended answer so the user can confirm or correct rather than starting from scratch
- If a question can be answered by exploring the codebase, explore the codebase instead of asking
- Do not move on from a branch until its dependencies are resolved
- Sessions of 16–50 questions are normal for complex features

---

**⏸ STAGE GATE 1→2**: *"Grilling complete. Ready to write the PRD?"*

---

## Stage 2 — Write a PRD

> Turn shared understanding into a durable GitHub issue.

You may skip steps if clearly unnecessary (e.g. if grilling already covered the interview, jump to step 4).

**2.1** Ask the user for a long, detailed description of the problem and any potential solutions (skip if Stage 1 already covered this).

**2.2** Explore the repo to verify assertions and understand current state.

**2.3** If shared understanding is not yet complete, interview the user relentlessly (re-run Stage 1 logic).

**2.4** Sketch out the major modules to build or modify. Actively look for opportunities to extract deep modules testable in isolation. Confirm with user which modules need tests.

**2.5** Write the PRD using the template below and submit as a GitHub issue.

```markdown
## Problem Statement
The problem the user is facing, from the user's perspective.

## Solution
The solution, from the user's perspective.

## User Stories
A long, numbered list. Format:
1. As a <actor>, I want <feature>, so that <benefit>

Cover ALL aspects of the feature exhaustively.

## Implementation Decisions
- Modules to build/modify
- Interface changes
- Architectural decisions
- Schema changes
- API contracts
- Specific interactions

Do NOT include file paths or code snippets (they go stale).

## Testing Decisions
- What makes a good test for this feature
- Which modules will be tested
- Prior art in the codebase

## Out of Scope
What this PRD does NOT cover.

## Further Notes
Anything else worth capturing.
```

---

**⏸ STAGE GATE 2→3**: *"PRD created at #[issue]. Ready to break it into issues?"*

---

## Stage 3 — PRD to Issues

> Turn the destination into a journey.

**3.1 Locate the PRD**

Ask for the PRD issue number (or URL). Fetch with `gh issue view <number>` if not in context.

**3.2 Explore the codebase** (if not already done in Stage 0 or 2).

**3.3 Draft vertical slices**

Break the PRD into **tracer bullet** issues. Each is a thin vertical slice cutting through ALL integration layers end-to-end — NOT a horizontal slice of one layer.

Classify each slice:
- **HITL** — requires human interaction (architectural decision, design review)
- **AFK** — can be implemented and merged without human interaction

Rules:
- Each slice delivers a narrow but COMPLETE path through every layer (schema, API, UI, tests)
- A completed slice is demoable or verifiable on its own
- Prefer many thin slices over few thick ones
- Prefer AFK over HITL where possible

**3.4 Quiz the user**

Present the proposed breakdown as a numbered list. For each slice show: title, type (HITL/AFK), blocked-by, user stories covered.

Ask:
- Does the granularity feel right?
- Are dependency relationships correct?
- Should any slices be merged or split?
- Are HITL/AFK labels correct?

Iterate until the user approves.

**3.5 Create GitHub issues**

Create in dependency order (blockers first) using `gh issue create`. Template:

```markdown
## Parent PRD
#<prd-issue-number>

## What to build
A concise description of this vertical slice. End-to-end behavior, not layer-by-layer. Reference parent PRD rather than duplicating.

## Acceptance criteria
- [ ] Criterion 1
- [ ] Criterion 2

## Blocked by
- Blocked by #<issue-number>
(or "None - can start immediately")

## User stories addressed
- User story 3
- User story 7
```

Do NOT close or modify the parent PRD issue.

---

**⏸ STAGE GATE 3→4**: *"Issues created. Ready to start TDD implementation? If so, tell me which issue to tackle first."*

---

## Stage 4 — TDD (per issue)

> Implement each issue with red-green-refactor. Repeat this stage for every issue.

### Philosophy

Tests verify behavior through public interfaces, not implementation details. Code can change entirely; tests shouldn't.

**Good tests** are integration-style: they exercise real code paths through public APIs. They describe *what* the system does, not *how*. A good test reads like a spec — "user can checkout with valid cart."

**Bad tests** are coupled to implementation: they mock internal collaborators, test private methods, or verify through external means. Warning sign: test breaks on refactor but behavior hasn't changed.

**Do NOT write all tests first then all implementation** (horizontal slicing). This produces tests that test imagined behavior, not actual behavior. Use vertical slices: one test → one implementation → repeat.

```
WRONG (horizontal):
  RED:   test1, test2, test3, test4
  GREEN: impl1, impl2, impl3, impl4

RIGHT (vertical):
  RED→GREEN: test1→impl1
  RED→GREEN: test2→impl2
  ...
```

### Workflow

**4.1 Planning**

Before writing any code:
- Confirm with user what interface changes are needed
- Confirm which behaviors to test (prioritise — you can't test everything)
- Identify opportunities for deep modules (small interface, deep implementation)
- Design interfaces for testability (accept dependencies, return results, small surface area)
- List the behaviors to test (not implementation steps)
- Get user approval on the plan

**4.2 Tracer Bullet**

Write ONE test that confirms ONE thing about the system:
```
RED:   Write test for first behavior → fails
GREEN: Write minimal code to pass → passes
```

**4.3 Incremental Loop**

For each remaining behavior:
```
RED:   Write next test → fails
GREEN: Minimal code to pass → passes
```

Rules:
- One test at a time
- Only enough code to pass current test
- Don't anticipate future tests
- Keep tests focused on observable behavior

**4.4 Refactor**

After all tests pass, look for refactor candidates:
- Extract duplication
- Deepen modules (move complexity behind simple interfaces)
- Apply SOLID principles where natural
- Consider what new code reveals about existing code
- Run tests after each refactor step

**Never refactor while RED. Get to GREEN first.**

### Mocking rules

Mock at **system boundaries only**:
- External APIs (payment, email, etc.)
- Databases (sometimes — prefer test DB)
- Time / randomness
- File system (sometimes)

Do NOT mock your own classes, internal collaborators, or anything you control.

### Per-cycle checklist

```
[ ] Test describes behavior, not implementation
[ ] Test uses public interface only
[ ] Test would survive internal refactor
[ ] Code is minimal for this test
[ ] No speculative features added
```

---

**⏸ STAGE GATE (loop)**: *"Issue #X complete. Move to next issue, or is the workflow done?"*
