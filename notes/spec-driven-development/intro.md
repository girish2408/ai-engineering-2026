# Spec-Driven Development

**Topic:** What it is, why it matters, how it differs from vibe coding, and how to structure a full SDD workflow with an AI agent

---

## 1. What Is Spec-Driven Development?

Spec-Driven Development (SDD) is the practice of writing a detailed, structured specification *before* giving a coding agent any implementation task.

Instead of typing a short instruction and waiting to see what the agent produces, you write one or more documents — sometimes called a **spec**, **brief**, or **constitution** — that describe *exactly* what you want to build: the goal, the constraints, the tech choices, the acceptance criteria, the edge cases.

The agent's job is to execute against that spec. Your job is to write and refine the spec.

> Think of it like an architect drawing blueprints before any concrete is poured. The builder (agent) is skilled — but they need a complete drawing, not a verbal sketch.

### What Goes in a Spec?

A spec is **not code**. It is context. It answers:

- What problem does this solve?
- Who is it for?
- What does success look like?
- What constraints apply (tech stack, performance, security, style)?
- What should it *not* do?
- What are the edge cases?

The spec can be a single long markdown file, a folder of files per feature, or a structured template — the format matters less than the completeness.

---

## 2. Three Core Benefits of Spec-Driven Development

### 2.1 Control Large Code Changes with Small Changes to the Spec

In a large codebase, a single poorly-worded prompt can trigger cascading changes across dozens of files. With a spec, the scope is locked. When requirements change, you make a *small, deliberate edit to the spec* — one line, one constraint, one updated acceptance criterion — and the agent re-executes against the updated spec.

The spec is the single source of truth. The code is the output of the spec.

```
Without SDD:  change requirement → re-prompt → hope the agent infers the scope correctly
With SDD:     change requirement → edit one line in spec.md → agent re-runs scoped task
```

### 2.2 Eliminate Context Decay Between Sessions

AI agents have no persistent memory across sessions. Every new conversation starts cold. If your only record of intent is a chat history, you lose it. The agent loses it. Even *you* lose it after a few days.

A spec is a durable, version-controlled document that preserves:
- The original intent
- The decisions made and why
- The constraints agreed upon
- The definition of done

Next session, you hand the agent the spec. It has full context in seconds. No re-explaining, no drift.

```
Without SDD:  session 2 → "so as I was saying last time..." → agent has no memory → drift
With SDD:     session 2 → attach spec.md → agent has complete context → continues cleanly
```

### 2.3 Define Success Criteria, Improve Intent Fidelity, and Set Constraints

A spec forces you to define *what done looks like* — before a single line of code is written. This has a compounding benefit: the agent can also help you improve the spec. You describe your intent in rough terms, the agent asks clarifying questions or proposes a tighter spec, and you end up with a crisper definition of success than you would have written alone.

The spec captures:
- **Success criteria** — what must be true for this to be "done"
- **Constraints** — what must not happen (no vendor lock-in, must be under 200ms, no PII in logs)
- **Intent** — the *why*, not just the *what*, so the agent can make sensible decisions in ambiguous situations

---

## 3. Why Spec-Driven Development? Vibe Coding vs Agentic Coding

### Vibe Coding

Vibe coding is prompt-and-react. You describe something loosely, see what you get, then iterate through a long back-and-forth dialogue to get closer to what you wanted.

**Example — vibe coding a button:**

```
You:    "create me an HTML button"
Agent:  [produces a plain grey button]
You:    "make it red"
Agent:  [makes it red]
You:    "round the corners"
Agent:  [rounds corners]
You:    "the text should say Submit not Button"
Agent:  [changes text]
You:    "the font is wrong, make it bold"
Agent:  [makes it bold]
You:    "actually center it on the page"
...     [5 more exchanges]
```

Each exchange consumes tokens, introduces small regressions, and depends entirely on the agent remembering earlier decisions. By turn 8 the agent has lost the beginning of the thread.

### Spec-Driven / Agentic Coding

You write the spec upfront. The agent executes once (or with minimal iteration) against a complete description.

**Example — spec-driven button:**

```markdown
## Button Spec

- Element: HTML <button>
- Background color: #D0021B (red)
- Border radius: 8px (rounded corners)
- Text: "Submit"
- Font: 600 weight, 16px, color #FFFFFF (white)
- Width: 160px, height: 48px
- Centered horizontally and vertically on the page
- On hover: background darkens to #A50115, cursor: pointer
- Accessible: aria-label="Submit form", keyboard focusable
```

```
You:    [paste spec]  "implement this button"
Agent:  [produces correct button in one pass]
```

### Side-by-Side Comparison

| | Vibe Coding | Spec-Driven / Agentic Coding |
|---|---|---|
| Starting point | Rough idea, verbal | Written spec with details |
| Agent input | Short, ambiguous prompt | Complete context document |
| Iteration style | Long dialogue, back-and-forth | Refine spec, re-run once |
| Context decay | High — earlier intent gets lost | None — spec is always attached |
| Regressions | Common — each fix can break earlier fix | Rare — spec is the constraint |
| Suitable for | Tiny one-off experiments | Real features, multi-file changes |
| Collaboration | Hard to hand off | Spec is readable by any human or agent |

---

## 4. The Spec as a Universal Contract

A spec is not just an instruction to an AI. It is a **contract** between everyone involved in building the thing:

- Between **you and the agent** — the agent knows exactly what to build and can flag when reality diverges from spec
- Between **you and your future self** — returning to a project after two weeks, the spec tells you what was intended
- Between **you and teammates** — a spec is readable, reviewable, and debatable before any code is written

### Intentions Into Spec

The most common failure mode in agentic coding is *implicit intent* — you know what you want but you haven't written it down. The agent fills the gaps with its best guess, which is often wrong.

Spec-Driven Development forces you to make implicit intent explicit:

```
Implicit:  "build a search feature"
Explicit:  "semantic search over fund descriptions using cosine similarity,
            returns top 5 results, response time < 300ms, no exact keyword
            matching, results include fund name + similarity score + excerpt"
```

The act of writing the spec often reveals ambiguities you didn't know existed — *before* the agent writes a line of code.

---

## 5. Benefits — Consolidated

| Benefit | Description |
|---------|-------------|
| **Control via small spec changes** | Changing a constraint in the spec is safer and more precise than re-prompting with new wording |
| **Eliminate context decay** | The spec travels with the project — any agent in any session has full context |
| **Improve intent fidelity** | Writing the spec forces you to resolve ambiguities; the agent can help tighten it further |
| **Reusability** | The spec documents the "why" — future changes are made in the context of original intent |
| **Reviewability** | A spec can be reviewed by a human *before* any code is written — catch mistakes early |
| **Scope control** | A well-scoped spec prevents the agent from doing too much or too little |

---

## 6. How Development Is Done — Two Levels of Detail

SDD operates at two levels. You write top-down: project first, then features.

### Level 1 — Project Constitution

Written once per project. Covers the big picture. The agent reads this at the start of every session.

```markdown
# Project Constitution: ENBD Wealth AI Agent

## Mission
Build a conversational AI agent that answers questions about ENBD mutual funds
using verified fund data. Accuracy over speed. Never hallucinate fund data.

## Tech Stack
- Backend: Python 3.12, FastAPI, LangGraph
- Vector DB: Qdrant (local for dev, cloud for prod)
- LLM: Claude 3.5 Sonnet (Anthropic)
- Embeddings: text-embedding-3-small (OpenAI)
- Tracing: LangSmith

## Constraints
- All fund data must come from the Qdrant vector store — no model hallucination
- No PII stored in logs
- API response time < 2s for 95th percentile
- All endpoints must have OpenAPI docs

## Roadmap
- [x] Phase 1: Data ingestion pipeline (fund PDFs → Qdrant)
- [ ] Phase 2: RAG retrieval + answer generation
- [ ] Phase 3: LangGraph stateful conversation
- [ ] Phase 4: Voice interface
```

### Level 2 — Feature Specification

Written per feature. Detailed enough that an agent can implement it without asking clarifying questions.

```markdown
# Feature Spec: Fund Search Endpoint

## Goal
Expose a REST endpoint that accepts a natural language query and returns the
top 5 most relevant fund descriptions from the Qdrant collection.

## Endpoint
POST /api/v1/search
Request:  { "query": string, "top_k": int = 5 }
Response: { "results": [{ "fund_name": string, "score": float, "excerpt": string }] }

## Implementation Details
- Embed the query using text-embedding-3-small
- Query Qdrant collection "enbd_funds" with cosine similarity
- Return top_k results with fund_name, similarity score, and a 200-char excerpt

## Acceptance Criteria
- [ ] Returns 200 with correct schema for a valid query
- [ ] Returns 400 if query is empty string
- [ ] Score values are between 0 and 1
- [ ] Response time < 300ms for a single query (measured via LangSmith)
- [ ] Unit test covers empty query edge case

## Out of Scope
- Authentication (handled by Phase 3)
- Re-ranking (future feature)
```

---

## 7. Level of Detail in a Spec

The more detail in the spec, the less the agent needs to guess. Cover these dimensions:

| Dimension | Questions to Answer | Example |
|-----------|-------------------|---------|
| **Goal** | What problem does this solve? For whom? | "Wealth advisors need instant answers about fund NAVs without reading PDFs" |
| **Mission** | What is the north star? What is explicitly not the mission? | "Accurate answers over fast answers — never fabricate data" |
| **Target audience** | Who uses this? What is their technical level? | "Non-technical wealth advisors using a chat UI" |
| **Constraints** | What must be true regardless of implementation? | "< 2s response, no PII in logs, Shariah-compliant funds tagged separately" |
| **Success criteria** | How will you know it's done? | "RAGAS faithfulness ≥ 0.80, all 5 acceptance criteria checked" |
| **Tech stack** | What tools, frameworks, versions? | "FastAPI 0.111, LangGraph 0.2, Qdrant 1.9" |
| **Out of scope** | What explicitly is NOT being built? | "No multi-language support in this phase" |

---

## 8. The SDD Workflow

```
┌─────────────────────────────────────────────────────────────┐
│                    PROJECT CONSTITUTION                       │
│  Mission · Tech Stack · Constraints · Roadmap                │
│  (written once, lives in /docs/constitution.md)              │
└────────────────────┬────────────────────────────────────────┘
                     │  drafted → reviewed → locked
                     ▼
┌─────────────────────────────────────────────────────────────┐
│                   FEATURE LOOP (per feature)                 │
│                                                              │
│   1. SPECIFICATION                                           │
│      Write feature spec: goal, endpoint, acceptance criteria │
│      Agent can help refine: "does this spec have gaps?"      │
│                                                              │
│   2. IMPLEMENTATION                                          │
│      Agent reads constitution + feature spec                 │
│      Agent implements — no free-form guessing                │
│                                                              │
│   3. VALIDATION                                              │
│      Run acceptance criteria as a checklist                  │
│      Automated tests + manual review                         │
│      Pass → merge · Fail → update spec → re-implement        │
└─────────────────────────────────────────────────────────────┘
```

### The Three Phases Per Feature

| Phase | Who does it | What happens |
|-------|------------|--------------|
| **Specification** | You (+ agent as consultant) | Write the feature spec; agent flags gaps, ambiguities, missing edge cases |
| **Implementation** | Agent (you supervise) | Agent codes against the spec; no creative licence beyond what's specified |
| **Validation** | You (+ automated tests) | Check each acceptance criterion; accept or send back with spec update |

---

## 9. You Are the Supervisor — The Agent Is the Builder

This is the most important mental model in SDD.

**You are the supervisor.** Your job is to:
- Write and refine the spec (design)
- Assign tasks to the agent (supervise)
- Review the output against the spec (review)
- Accept when criteria are met, or update the spec and re-assign (iterate)

**The agent is the builder.** The agent's job is to:
- Read and execute the spec faithfully
- Flag when the spec is ambiguous or contradictory
- Implement without inventing requirements that weren't specified

```
You (Supervisor)          Agent (Builder)
─────────────────         ────────────────────────────
Write spec          →     Read spec
                          Ask: "Is section 3 correct?"
Clarify spec        →     Implement
Review output       →     
Accept / revise     →     Re-implement if needed
```

The agent is highly capable — it knows how to write clean code, handle edge cases, structure files. You don't need to tell it *how* to do things. You need to tell it *what* to build and *what success looks like*. The how is the builder's domain. The what is yours.

> Your leverage as a developer in the agentic era is not writing code faster — it is writing *better specs* that produce correct code the first time.

---

## Summary

| Concept | One-line summary |
|---------|-----------------|
| What is SDD | Write a detailed spec first; agent executes against it |
| Spec content | Context, not code — goals, constraints, acceptance criteria |
| vs Vibe coding | Vibe = dialogue iteration; SDD = spec once, execute cleanly |
| Universal contract | Spec is readable by humans and agents alike; source of truth |
| Two levels | Constitution (project-wide) + Feature spec (per feature) |
| Workflow | Constitution → Feature loop: Spec → Implement → Validate |
| Your role | Supervisor: design, assign, review, accept or refine |
| Agent's role | Builder: read spec, implement faithfully, flag ambiguity |
