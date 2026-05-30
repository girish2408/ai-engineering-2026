# SDD Commands & Workflow

**Topic:** The five core commands and two optional quality gates that drive spec-driven development — with detail on each phase, what artifacts are produced, and why each step exists

---

## Visual Workflow

```
INITIAL BUILD (new project)
────────────────────────────────────────────────────────────────
/constitution → /specify → /plan → /tasks → /implement
                    ↓                  ↓
                /clarify           /analyze
               (optional)         (optional)

ADDING A NEW FEATURE (project already set up)
────────────────────────────────────────────────────────────────
/specify → /plan → /tasks → /implement
    ↓                 ↓
/clarify           /analyze
```

**/constitution** and **/specify** are the two entry points (bright purple in the diagram — they require your direct input). The rest of the pipeline is agent-driven, with you in a review/approval role.*

---

## Why This Pipeline Exists

Before diving into each command, it helps to understand the two core problems SDD solves.

### Problem 1: Context Decay Between Sessions

AI agents have no memory across sessions. Every new session starts cold. Without a written spec, the agent re-derives architectural decisions, makes subtly different choices, and the codebase slowly drifts from the original intent. This is **context decay** — the gradual erosion of project intent through repeated re-explanation.

SDD solves this by externalising intent into version-controlled files. The constitution, specs, plans, and task lists travel with the code. Any agent in any session picks up full context in seconds by reading those files.

### Problem 2: The Ambiguity Tax

Vague prompts force agents to fill gaps with their best guess — statistically probable but often wrong. The **ambiguity tax** is the rework cost of those wrong guesses discovered during or after implementation.

SDD moves ambiguity resolution to the *earliest possible stage* — at spec time, before any plan or code is written. Clarifying questions get asked and answered when the cost is near zero (updating a markdown file) rather than high (rewriting implemented code).

---

## The Three-File Standard

Most SDD platforms (Kiro/AWS, GitHub Spec Kit, Tessl) converge on the same three-file structure per feature:

```
specs/
  [feature-name]/
    spec.md      ← WHAT: requirements, user story, acceptance criteria
    plan.md      ← HOW: technical approach, architecture, data models
    tasks.md     ← STEPS: ordered, atomic coding tasks
constitution.md  ← ALWAYS: binding principles for the whole project
```

Each file is committed alongside code. A PR is incomplete without the spec that drove it.

---

## 1. `/constitution` — Core Binding Principles

**Run once per project.** When you run `/constitution`, the agent reads the principles you have written down and produces or updates a formal `constitution.md` file. This file is the root document of the entire project. Every other command runs in its context.

### What the constitution is

The constitution is equivalent to a **CLAUDE.md** file — the project-wide standing instruction that an agent reads at the start of every session before doing anything else. It defines the rules that cannot be overridden by any individual feature. It is the boundary.

When you run `/constitution` and tell it "the application should have WCAG accessibility standards", the agent doesn't just note that — it encodes it as a binding principle with specific rules and checkable criteria. It may also update related templates (e.g. the spec template to include an accessibility acceptance criterion section, or a checklist that runs on every feature).

### What it contains

```markdown
# Project Constitution

## Mission
One or two sentences: what the application does and what it must never do.
e.g. "Deliver accurate fund data to wealth advisors. Never fabricate financial figures."

## Tech Stack
Locked choices: language, framework, database, versions.
e.g. "Python 3.12 · FastAPI 0.111 · PostgreSQL 16 · React 18 · Docker"

## Architectural Constraints
Module boundaries, dependency rules, folder structure conventions.
e.g. "No circular imports. Business logic lives in /services, not /routes."

## Security Rules
Non-negotiable security requirements.
e.g. "No PII in logs. All endpoints require JWT. No secrets in source."

## Performance Budgets
Hard limits.
e.g. "API p95 response < 500ms. Frontend LCP < 2.5s."

## Accessibility Standards
e.g. "All UI must meet WCAG 2.1 AA. All interactive elements keyboard-navigable."

## Coding Standards
Style, test coverage, documentation requirements.
e.g. "Black formatter, 80% test coverage minimum, no `any` in TypeScript."

## Out of Scope (Global)
What will never be built in this project.
```

### Best practices for constitution files

- Keep it concise (150–200 lines is the practical limit before agents start missing things)
- Use headers for organisation, prose for intent, code snippets for unambiguous examples
- Do **not** document file paths — they change. Document patterns and capabilities instead
- Cross-tool: the same principles live in `CLAUDE.md` (Claude Code), `.cursorrules` (Cursor), and `.github/copilot-instructions.md` (GitHub Copilot). A pre-commit hook can keep them in sync

### What the agent does when you run `/constitution`

1. Reads the principles you have provided
2. Produces or updates `constitution.md` with those principles encoded formally
3. Updates any related templates (e.g. the default spec template, checklists) to reflect the new principles
4. Flags anything underspecified: "You mentioned 'clean code' — do you want to specify a formatter, linter, or style guide?"

---

## 2. `/specify` — Feature Specification

**Run once per new feature.** The specify command creates a structured specification for a single feature. This is intentionally high-level — it captures *what* the feature does, *who* it serves, and *what success looks like*, without prescribing implementation details.

### What the spec file contains

Running `/specify` produces `specs/[feature]/spec.md` with these sections:

**User story**
Who the user is, what they want to do, and why.
```
As a wealth advisor, I want to see a dashboard of my client's portfolio stats
so that I can quickly answer questions about allocation and performance without
opening a PDF.
```

**Functional requirements**
What the system must do — concrete, observable behaviours.
```
- Display total portfolio value in AED, updated daily
- Show asset allocation breakdown: equity / bond / cash as percentages
- List top 5 performing funds this month by absolute return
- Show a 30-day performance sparkline (% return)
```

**Key entities**
The domain objects involved.
```
Portfolio, Fund, AssetAllocation, PerformanceHistory
```

**Edge cases**
Boundary conditions the implementation must handle.
```
- Portfolio with zero funds → show empty state, not an error
- API returns stale data (>24h old) → show a staleness warning
- All funds are in the same asset class → allocation shows 100%
```

**Assumptions**
Dependencies and constraints that are taken as given.
```
- /api/portfolio endpoint exists and returns data in the agreed schema
- User is already authenticated before reaching the dashboard
- Performance data is available for all funds in the portfolio
```

**Success criteria**
The checklist that defines "done". Each item is measurable and binary.
```
- [ ] Dashboard renders all 4 components when portfolio data loads
- [ ] Empty state shown when portfolio has no funds
- [ ] Staleness warning shown when data is >24h old
- [ ] All interactive elements keyboard-navigable (WCAG AA)
- [ ] Page load < 1.5s on a simulated 4G connection
- [ ] All 12 unit and integration tests pass
```

**Quality gates (built into specify)**

The specify command also runs a built-in checklist before finalising the spec:

| Gate | Check |
|------|-------|
| Requirement completeness | Are all functional requirements specific and testable? |
| Feature readiness | Are all dependencies (APIs, data) confirmed to exist? |
| Validation coverage | Does each requirement have at least one acceptance criterion? |
| Edge case coverage | Are failure states and boundary conditions addressed? |
| Constitution alignment | Does this feature violate any constitution rule? |

If any gate fails or has outstanding doubts, the spec is flagged — not silently passed.

---

## 3. `/clarify` — Resolve Ambiguities *(optional, runs after `/specify`)*

**Run after `/specify`, before `/plan`.** The clarify command reads the current spec and identifies anything ambiguous, underspecified, or open to more than one interpretation. The agent asks you targeted questions. You answer. The spec is updated with your answers encoded as hard requirements.

### Why this matters

Without clarify, ambiguities carry forward silently. The plan picks one interpretation, the tasks inherit it, and the implementation is based on an assumption you never approved. You discover the divergence late — in review, or in production.

Clarify is cheap (a conversation at spec time). Finding the same issue in code review is expensive.

### Example — clarify running on the dashboard spec

```
Agent (running /clarify on specs/dashboard/spec.md):

I found 4 areas that need clarification before planning begins:

Q1. Sparkline metric
    The spec says "30-day performance sparkline (% return)" but does not
    specify the baseline. Should this be:
    (a) % change from 30 days ago to today
    (b) cumulative daily return over the 30-day period
    (c) return relative to the fund's benchmark index

Q2. "Top 5 performing funds" definition
    Performing relative to what? Options:
    (a) Absolute return in AED over the current month
    (b) Percentage return over the current month
    (c) Return vs category average (alpha)

Q3. Dashboard visibility
    The spec assumes the user is authenticated, but does not state
    what happens if the session expires mid-session on the dashboard.
    Should the page: (a) redirect to login, (b) show a session timeout
    modal with re-login option, (c) show cached data with a warning?

Q4. Staleness threshold
    The spec mentions a "staleness warning if data is >24h old" but the
    constitution performance budget requires data to be refreshed daily.
    Are these two constraints in conflict, or is 24h the correct threshold?

Please answer each question. I will update spec.md accordingly.
```

You answer the four questions. The agent updates the spec with your answers as explicit requirements. The plan starts from a complete foundation.

---

## 4. `/plan` — Technical Implementation Plan

**Run after `/specify` (and `/clarify` if used).** The plan command reads the feature spec and the constitution and produces a detailed technical plan for *how* to implement the feature. This is where the agent applies its technical knowledge — it designs the architecture, selects the components, defines the data flow, and records the rationale.

You do not write the plan. You review it. Your job is to check that it aligns with the constitution and the spec — not to prescribe implementation details.

### What you can direct in `/plan`

You can give the agent specific technical preferences or constraints at plan time. These are things that matter for how it is built, not what is built:

```
/plan - use Tailwind for theme colours, reference tokens from tailwind.config.js
        - use react-query for data fetching and caching
        - use date-fns for date formatting, not moment.js
        - use recharts for the sparkline (already installed)
        - no tests needed for this feature (we will add them in Phase 2)
```

These preferences are encoded into the plan and cascade to the task list automatically.

### What the plan produces

Running `/plan` creates multiple files in the specs folder:

**`specs/[feature]/plan.md`**
The main plan document.
```markdown
## Technical Approach
How the feature will be built: component tree, data flow, state management.

## Principles
Which constitution rules apply and how this plan respects them.

## Tech Stack
Libraries and versions used for this specific feature.

## Project Structure
New files to create, existing files to modify.

## Prerequisites / Setup
Anything that must exist before implementation begins.

## Research
Why specific decisions were made — rationale for library choices, architecture tradeoffs.
```

**`specs/[feature]/contracts.md`**
The machine-readable interface contracts: API response schemas, TypeScript interfaces, function signatures. These act as the reference while modifying code — both human reviewers and the implementing agent check their output against contracts.

```typescript
// contracts.md
interface PortfolioSummary {
  totalValueAED: number;
  lastUpdated: string;           // ISO 8601
  allocation: AllocationBreakdown;
  topFunds: FundPerformance[];
  sparkline: SparklinePoint[];
}

interface AllocationBreakdown {
  equity: number;   // percentage 0-100
  bond: number;
  cash: number;
}
```

**`specs/[feature]/datamodel.md`**
Entity definitions, relationships, validation rules, and database schema changes if any.

---

## 5. `/tasks` — Actionable Coding Task List

**Run after `/plan`.** The tasks command reads the full plan and breaks it into an ordered list of discrete, atomic coding tasks. Each task is small enough to be implemented and verified in a single agent session.

The task list is what the implementing agent works from. Every task has an ID, a clear objective, and defined inputs/outputs.

### What a task list looks like

```markdown
# Dashboard Feature — Tasks

## Wave 1: Foundation (no dependencies)
- [ ] T01 — Create `src/hooks/usePortfolio.ts` · fetches from `/api/portfolio`,
             returns { data, isLoading, isError, isStale } via react-query
- [ ] T02 — Create `src/types/portfolio.ts` · TypeScript types from contracts.md
- [ ] T03 — Add `/dashboard` route to `src/router.tsx`

## Wave 2: Components (depends on T01, T02)
- [ ] T04 — Build `PortfolioValueCard` · displays totalValueAED, staleness warning
- [ ] T05 — Build `AssetAllocationChart` · recharts pie chart, equity/bond/cash
- [ ] T06 — Build `TopFundsList` · ranked list, fund name + monthly % return
- [ ] T07 — Build `PerformanceSparkline` · recharts LineChart, 30-day % return

## Wave 3: Integration (depends on Wave 2)
- [ ] T08 — Assemble `Dashboard.tsx` · wire usePortfolio, render all 4 components
- [ ] T09 — Handle loading state · skeleton loaders for each card
- [ ] T10 — Handle empty state · EmptyPortfolio component when fund list is empty
- [ ] T11 — Handle error state · error boundary with retry button

## Wave 4: Quality (depends on Wave 3)
- [ ] T12 — Accessibility pass · aria-labels on all chart elements, keyboard nav
- [ ] T13 — Performance check · verify LCP < 1.5s with Chrome DevTools
```

### Key properties of a good task list

| Property | Why it matters |
|----------|---------------|
| **Atomic** — one clear objective per task | Agent can complete in one session without re-reading entire plan |
| **Ordered by dependency** — later tasks build on earlier | No task starts before its inputs exist |
| **Grouped into waves** — independent tasks in the same wave | Tasks in the same wave can run in parallel (multiple agents) |
| **Traceable** — each task maps to a spec requirement | Easy to verify nothing was missed |
| **Sized for one session** — not too large, not trivial | Avoids context overflow and reduces WIP |

---

## 6. `/analyze` — Cross-Artifact Alignment Check *(optional, runs after `/tasks`)*

**Run after `/tasks`, before `/implement`.** The analyze command reads all artefacts produced so far — constitution, spec, plan, contracts, data model, and task list — and performs a **read-only consistency audit**. It makes no changes. It only reports issues.

Think of it as a pre-flight check before any code is written.

### What analyze looks for

| Check | Example issue found |
|-------|-------------------|
| **Duplication** | Task T04 and T08 both claim to render the staleness warning |
| **Gaps** | Acceptance criterion 4 (WCAG) has no corresponding task |
| **Contradictions** | Plan uses `localStorage` for caching; constitution prohibits browser storage for portfolio data |
| **Underspecified items** | T07 references "30-day sparkline data" but contracts.md has no `SparklinePoint` type defined |
| **Unresolved ambiguities** | Plan says "% return" but clarify session answer said "cumulative daily return" — these need to be reconciled |
| **Constitution violations** | A new dependency introduced in the plan is not on the approved tech stack |

### Example analyze output

```
/analyze results — specs/dashboard/

CONFLICTS (must fix before implement):
  1. Plan.md proposes caching portfolio data in localStorage.
     constitution.md §4 prohibits storing portfolio data in browser storage.
     → Update plan.md to use in-memory react-query cache only.

GAPS (acceptance criteria with no task):
  2. spec.md criterion 4: "All interactive elements keyboard-navigable (WCAG AA)"
     No task in tasks.md addresses this.
     → Add task T12: Accessibility audit pass

UNDERSPECIFIED:
  3. T07 references SparklinePoint[] but contracts.md defines no such type.
     → Add SparklinePoint interface to contracts.md before T07 runs.

NO ISSUES FOUND IN: user story, functional requirements, entity model,
tech stack selection, dependency graph.

Resolve 3 items above, then run /implement.
```

Analyze is read-only. It never modifies files. It is a safety net, not an autocorrect.

---

## 7. `/implement` — Execute the Task List

**Run after `/tasks` (and `/analyze` if used).** The implement command instructs the agent to work through the task list in order and produce real code. The agent reads the constitution, spec, plan, contracts, and task list before starting. It does not improvise — it executes the spec.

### How implement runs

```
/implement

Agent:
Reading: constitution.md, specs/dashboard/spec.md, specs/dashboard/plan.md,
         specs/dashboard/contracts.md, specs/dashboard/tasks.md

Starting T01 — Creating src/hooks/usePortfolio.ts
[implements T01]
✓ T01 complete

Starting T02 — Creating src/types/portfolio.ts
[implements T02]
✓ T02 complete

Starting T03 — Adding /dashboard route
[implements T03]
✓ T03 complete

Wave 1 complete. Proceeding to Wave 2.
...
```

### Your role during implement

You are the supervisor. You:
- Set checkpoints ("stop after Wave 2 so I can review")
- Review each completed wave against the spec's acceptance criteria
- Accept the output, or update the spec/task and re-run that task
- Do not re-prompt with new requirements — update the spec instead

The agent never has "creative licence" during implement. If it encounters ambiguity, it stops and asks — it does not guess. If you find yourself correcting the agent with new instructions mid-implement, that is a signal that the spec needed more detail. Update the spec. Re-run.

---

## Full Workflow Summary

### Initial project build

```
Step  Command         Output                          Who produces it
────  ─────────────   ──────────────────────────────  ────────────────────
1     /constitution   constitution.md                 You define principles,
                                                      agent formalises them
2     /specify        specs/[feat]/spec.md            You describe feature,
                                                      agent structures it
3     /clarify*       updated spec.md                 Agent asks, you answer
4     /plan           plan.md, contracts.md,          Agent (you review)
                      datamodel.md
5     /tasks          tasks.md                        Agent (you review)
6     /analyze*       conflict report → you fix       Agent reads, you resolve
7     /implement      working code                    Agent builds, you supervise

* optional quality gates
```

### Adding a new feature (project already exists)

Skip `/constitution` — it is already written. Start at `/specify`. All 4 remaining steps are the same. The constitution constrains every new feature automatically.

---

## The Mental Model: Supervisor and Builder

| Role | Who | Responsibilities |
|------|-----|-----------------|
| **Supervisor / Architect** | You | Write principles (constitution), describe features (specify), answer clarifications, review plans, review tasks, accept or reject output |
| **Builder / Agent** | Coding agent | Formalise the constitution, structure the spec, write the plan, decompose tasks, implement code — all faithfully against the spec |

The agent does not co-decide on architecture. It executes your intent and flags when the spec is unclear or contradictory.

> Your leverage is not writing code faster. It is writing better specs that produce correct code the first time — eliminating the ambiguity tax before it accumulates.

---

## Why Each Command Exists (One-Line Summary)

| Command | Exists because... |
|---------|------------------|
| `/constitution` | Without binding principles, every feature re-invents architecture differently |
| `/specify` | Without a written spec, intent stays implicit and the agent guesses |
| `/clarify` | Ambiguities found at spec time cost nothing to fix; found in code review, they cost everything |
| `/plan` | The agent needs to decide *how* before *doing* — and you need to review that decision |
| `/tasks` | A plan is too large for one session; tasks make implementation incremental and reviewable |
| `/analyze` | Silent contradictions between spec, plan, and tasks produce bugs that are hard to trace |
| `/implement` | The agent executes the task list faithfully — no improvisation, no scope creep |
