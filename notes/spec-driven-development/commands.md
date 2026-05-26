# SDD Commands & Workflow

**Topic:** The five core commands that drive a spec-driven development workflow — and two optional quality gates

---

## Overview

In SDD, you work through a set of commands in a deliberate sequence. Each command is a prompt you run against a coding agent. Each builds on the output of the previous one. The pipeline moves from high-level principles down to actual code.

```
INITIAL BUILD (first time for a new project)
─────────────────────────────────────────────
constitution → specify → [clarify] → plan → tasks → [analyze] → implement

ADDING A NEW FEATURE (project already exists)
──────────────────────────────────────────────
specify → [clarify] → plan → tasks → [analyze] → implement
```

The bracketed commands — `clarify` and `analyze` — are optional quality gates. Run them when the feature is complex or the spec is long.

---

## 1. `constitution` — Core Binding Principles

**Run once per project.** The constitution command instructs the coding agent to take the high-level principles you have outlined and flesh them out into a full, formal `constitution.md` file for the application.

The constitution is the root document of the entire project. Every other command (specify, plan, tasks, implement) is run *in the context of* the constitution. It defines what the application *must always be* — the rules that cannot be overridden by any individual feature.

### What goes in a constitution

- Application mission and north star
- Core architectural decisions (tech stack, folder structure conventions)
- Non-negotiable constraints (performance budgets, security rules, accessibility standards, compliance requirements)
- Coding standards (language versions, style guides, test coverage requirements)
- What the application must never do

### Example

You might write in your notes:

```
Principles to encode in the constitution:
- Application must meet WCAG 2.1 AA accessibility standards
- All API responses must be under 500ms at p95
- No PII may be written to logs
- All endpoints must be authenticated via JWT
- Tech stack: FastAPI, React 18, PostgreSQL, Docker
```

You then run the constitution prompt:

```
Prompt to agent:
"Read the principles below and produce a full constitution.md for this application.
The constitution must define binding rules under these headings: Mission, Tech Stack,
Architectural Constraints, Security Rules, Performance Budgets, Accessibility Standards,
Coding Standards, Out of Scope. Flag anything underspecified.

Principles: [paste your notes]"
```

The agent produces `constitution.md`. You review it, adjust anything that was mis-stated, and lock it. From this point on, every other agent session starts by reading this file.

---

## 2. `specify` — High-Level Feature Specification

**Run once per new feature.** The specify command produces a high-level specification document for a single feature. This is intentionally high-level — it captures *what* the feature does, *who* it serves, and *what success looks like*, without prescribing implementation details.

### What a spec covers

- Feature name and one-line description
- User story / goal
- Inputs and outputs (what the user provides, what they receive)
- Acceptance criteria (the checklist that defines "done")
- Constraints specific to this feature
- What is explicitly out of scope for this feature

### Example

```
Prompt to agent:
"I want to add a dashboard page that shows a user's portfolio stats:
total portfolio value, asset allocation breakdown (equity/bond/cash), 
top 5 performing funds this month, and a 30-day performance sparkline.
The data comes from our existing /api/portfolio endpoint.

Produce a feature spec for this dashboard page following our constitution.
Include: goal, user story, inputs/outputs, acceptance criteria, constraints, out of scope."
```

The agent produces `features/dashboard/spec.md`. You review it. If anything is off, you correct the spec — not the agent in a back-and-forth. The spec is the artefact.

---

## 3. `clarify` — Resolve Underspecified Areas *(optional)*

**Run after `specify`, before `plan`.** The clarify command instructs the agent to read the spec you just produced and identify any areas that are ambiguous, underspecified, or could be interpreted in more than one way. The agent asks you targeted questions. You answer them. The spec is updated.

This command exists because specs written quickly will have gaps. It is better for the agent to surface those gaps *before* a plan is written than to discover them during implementation.

### What clarify does

- Reads the feature spec
- Lists every ambiguity or missing detail it finds
- Asks you specific questions, one per gap
- You answer; the spec is updated with your answers
- Result: a tighter, more complete spec before any planning begins

### Example interaction

```
Agent (running clarify on the dashboard spec):

Questions:
1. The sparkline shows 30-day performance — should this be based on NAV change,
   percentage return, or absolute return in AED?
2. "Top 5 performing funds this month" — performing relative to what benchmark?
   Absolute return or return vs category average?
3. Should the dashboard be visible to all user roles, or only users with
   a linked portfolio? What should unauthenticated users see?
4. Is the asset allocation chart interactive (clickable to drill down)
   or purely visual?
5. What should the page show if the /api/portfolio endpoint returns an error?

Please answer each question and I will update the spec.
```

You answer the five questions. The agent updates `spec.md` with your answers baked in as hard requirements. Now the spec has no gaps.

---

## 4. `plan` — Technical Implementation Plan

**Run after `specify` (and `clarify` if used).** The plan command instructs the agent to read the feature spec and the constitution, and produce a concrete technical plan for *how* to implement the feature. This is where the agent's technical knowledge is applied — it decides the architecture, the components, the data flow, the file structure.

You do not write the plan. The agent writes it based on your spec and constitution.

### What a plan covers

- Technical approach and architecture decisions
- Files to create or modify, with their purpose
- Data flow from input to output
- API contracts (if new endpoints are needed)
- State management approach (if frontend)
- External dependencies introduced
- Testing approach

### Example

```
Prompt to agent:
"Read constitution.md and features/dashboard/spec.md.
Produce a detailed technical implementation plan for the dashboard feature.
Include: architecture decisions, file list (create/modify), data flow diagram,
component breakdown, API changes if any, and testing approach."
```

The agent produces `features/dashboard/plan.md`. You review it for soundness — does it align with the constitution? Does it introduce anything that violates a constraint? If yes, you update the spec or the plan accordingly. You are not debugging code — you are reviewing a blueprint.

---

## 5. `tasks` — Actionable Coding Task List

**Run after `plan`.** The tasks command instructs the agent to take the full technical plan and break it down into a logical, ordered list of discrete, actionable coding tasks. Each task is small enough to be implemented and verified independently.

The task list is what actually drives implementation. It is an ordered checklist — each item is a unit of work the agent can execute in a single session without needing to re-read the entire plan.

### What a task list looks like

```markdown
# Dashboard Feature — Task List

## Setup
- [ ] T01: Create `src/pages/Dashboard.tsx` with placeholder component
- [ ] T02: Add `/dashboard` route to `src/router.tsx`
- [ ] T03: Create `src/hooks/usePortfolio.ts` — fetches from `/api/portfolio`

## Components
- [ ] T04: Build `PortfolioValueCard` component (displays total value in AED)
- [ ] T05: Build `AssetAllocationChart` component (pie chart, equity/bond/cash)
- [ ] T06: Build `TopFundsList` component (top 5 funds, name + monthly return)
- [ ] T07: Build `PerformanceSparkline` component (30-day line chart, % return)

## Integration
- [ ] T08: Wire `usePortfolio` hook into Dashboard page, handle loading/error states
- [ ] T09: Pass portfolio data as props to all four sub-components

## Testing
- [ ] T10: Unit test `usePortfolio` hook — mock API success and error cases
- [ ] T11: Unit test `AssetAllocationChart` renders correct segments
- [ ] T12: Integration test: Dashboard renders all 4 components when data loads

## Polish
- [ ] T13: Add skeleton loaders for each card while data is fetching
- [ ] T14: Add error boundary with fallback UI if API fails
- [ ] T15: Verify WCAG 2.1 AA: all chart elements have aria-labels
```

### Example prompt

```
Prompt to agent:
"Read constitution.md, features/dashboard/spec.md, and features/dashboard/plan.md.
Turn the technical plan into an ordered list of small, actionable coding tasks.
Each task should be completable in one focused session. Format as a markdown checklist.
Group tasks by: Setup, Components, Integration, Testing, Polish."
```

---

## 6. `analyze` — Cross-Artifact Alignment Check *(optional)*

**Run after `tasks`, before `implement`.** The analyze command instructs the agent to read all artefacts produced so far — constitution, spec, plan, task list — and verify they are fully aligned. It looks for contradictions, gaps, tasks that reference things not defined in the plan, or plan decisions that violate the constitution.

Think of it as a consistency audit before any code is written.

### What analyze checks

- Do all tasks in the task list map to something in the plan?
- Does the plan introduce any patterns that contradict the constitution?
- Are there acceptance criteria in the spec that have no corresponding task?
- Are there any implicit assumptions in the plan that were never approved in the spec?
- Are there conflicting instructions across any of the documents?

### Example output

```
Agent (running analyze):

Alignment issues found:

1. CONFLICT: spec.md (section 3) requires authentication check before rendering the
   dashboard. plan.md makes no mention of an auth guard. T01–T15 have no auth task.
   → Add a task: "T00: Add auth guard HOC to /dashboard route"

2. GAP: spec.md acceptance criterion 4 states "page must show empty state if
   portfolio has zero funds." No task covers this empty state.
   → Add a task: "T09b: Build EmptyPortfolio component and render when fund list is empty"

3. CONSTITUTION VIOLATION: plan.md proposes using localStorage to cache portfolio data.
   constitution.md section 4 (Security Rules) prohibits storing portfolio data in
   browser storage.
   → Remove caching from plan or discuss an alternative (e.g. in-memory React state)

No other issues found. Resolve the 3 items above before running implement.
```

You fix the three issues — update the spec, plan, or task list as needed — then proceed to implement with confidence.

---

## 7. `implement` — Execute the Task List

**Run after `tasks` (and `analyze` if used).** The implement command instructs the agent to begin executing the task list. The agent works through the tasks in order, implementing each one, checking it off, and moving to the next.

You do not write code. You supervise. After each task or group of tasks, you review the output against the spec and acceptance criteria.

### What implement looks like in practice

```
Prompt to agent:
"Read constitution.md, features/dashboard/spec.md, features/dashboard/plan.md,
and features/dashboard/tasks.md. Begin implementing the tasks in order, starting
at T01. After completing each task, confirm it is done and state which task you
are moving to next. Stop after completing T07 so I can review."
```

The agent works through T01–T07, producing real code. You review. If a task's output does not match the spec, you do not re-prompt with new instructions — you update the spec or task, and re-run implement for that task. The artefacts stay authoritative.

---

## Full Workflow Summary

### Initial project build

| Step | Command | Output | Run by |
|------|---------|--------|--------|
| 1 | `constitution` | `constitution.md` | You + agent |
| 2 | `specify` | `features/[name]/spec.md` | You + agent |
| 3 | `clarify` *(optional)* | Updated `spec.md` | Agent asks, you answer |
| 4 | `plan` | `features/[name]/plan.md` | Agent |
| 5 | `tasks` | `features/[name]/tasks.md` | Agent |
| 6 | `analyze` *(optional)* | Conflict report → fixes | Agent audits, you resolve |
| 7 | `implement` | Working code | Agent builds, you supervise |

### Adding a new feature (project exists)

Skip `constitution` — it is already written. Start at `specify`:

```
specify → [clarify] → plan → tasks → [analyze] → implement
```

The constitution is always in scope. Every new feature is constrained by it automatically.

---

## Key Principle

> You are not writing code. You are writing documents that produce code.
>
> The quality of your output is determined by the quality of your spec — not by how well you can prompt in the moment.
