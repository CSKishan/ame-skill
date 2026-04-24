# AME + EMA — Agent Pipeline Documentation

## What Is This?

Two VS Code Copilot Chat skills that work as a pipeline:

```
/ame  →  single compiled interview  →  .ame/spec.md
/ema  →  layer analysis             →  .ame/plan.md  →  chunked execution
```

**`/ame` (Ask Me Exhaustively)** closes the requirement gap before any code is written. It compiles all applicable interview questions into **one message** across five quality dimensions — tech stack, architecture, security, quality, and edge cases — and produces a structured spec file from your answers. Total exchanges: 2–3.

**`/ema` (Enclose My Analysis)** reads that spec, identifies which architectural layers the project spans, and generates a dependency-ordered, chunked implementation plan. It offers chunk-by-chunk execution (with per-chunk confirmation) or a "run all" mode for one-shot execution.

---

## Requirements

- VS Code with GitHub Copilot Chat
- **Agent mode** — both skills write files to the workspace; Ask mode is not sufficient
- Optional: [context7 MCP server](https://github.com/upstash/context7) for live library documentation during the interview

---

## Install

### Manual

1. Clone or download this repository
2. Copy `skills/ame/SKILL.md` to your agents skills folder:
   - Windows: `%USERPROFILE%\.agents\skills\ame\SKILL.md`
   - macOS/Linux: `~/.agents/skills/ame/SKILL.md`
3. Copy `skills/ema/SKILL.md` to:
   - Windows: `%USERPROFILE%\.agents\skills\ema\SKILL.md`
   - macOS/Linux: `~/.agents/skills/ema/SKILL.md`
4. Restart VS Code

### Via skills CLI

```bash
npx skills add github:{your-github-username}/ame-skill
```

---

## Usage

### Full workflow

```
1. Open VS Code Copilot Chat in Agent mode
2. Describe what you want to build, then invoke:

   /ame I want to build a REST API for managing IoT device telemetry

3. AME detects scope, compiles all applicable dimension questions into
   one message. Answer everything at once — free-form or labelled.

4. AME processes your answers, writes .ame/spec.md, and presents a
   plain-English summary. Confirm it or correct anything in one reply.

5. When done, invoke:

   /ema

6. EMA reads .ame/spec.md, generates .ame/plan.md,
   shows a chunk summary, and waits for your confirmation to execute.
```

### Shortcuts

| Trigger | Behaviour |
|---------|-----------|
| `/ame` with a description | Starts interview immediately |
| `/ame` without a description | AME asks for a one-sentence intent |
| "done" / "plan it" / "proceed" during `/ame` | Finalises spec and shows handoff message |
| `/ema` during `/ame` | AME finalises spec, EMA takes over immediately |
| "review" during `/ema` | Shows full `.ame/plan.md` before execution |
| "run all" during `/ema` | Executes all chunks back-to-back; pauses only on validation failure |
| "adjust {description}" during `/ema` | Updates plan before executing |

---

## Scope Levels

AME automatically estimates scope from your opening description:

| Scope | Criteria | Dimensions asked |
|-------|----------|-----------------|
| `micro` | Single file, script, quick fix | [E] only (3 questions total) |
| `small` | <5 files, no auth, no external systems | [A], [B], [E] |
| `full` | Multiple systems, auth, integrations, or compliance | [A], [B], [C], [D], [E] |

---

## Interview Dimensions

| Dimension | Covers |
|-----------|--------|
| **[A] Tech Stack** | Language, runtime, framework, platform, team |
| **[B] Features & Architecture** | Core features, actors, data flow, architecture patterns |
| **[C] Security & Compliance** | Auth, data sensitivity, regulations, trust boundaries _(full only)_ |
| **[D] Quality & Operations** | Accessibility, performance, testing, CI/CD _(full only)_ |
| **[E] Edge Cases & Gaps** | Failure modes, degraded state, assumptions, open questions |

---

## Spec File — `.ame/spec.md`

AME writes this file after receiving your answers. It lives in your project root and is the contract between AME and EMA.

**Commit it to version control.** It documents the decisions made before coding began — valuable for onboarding, audits, and revisiting requirements.

A blank template lives at `.ame/spec-template.md` in this repository.

---

## Plan File — `.ame/plan.md`

EMA writes this file after reading the spec. It contains:

- Plan risks (LOW-confidence spec dimensions)
- Project summary
- Per-chunk breakdown: files, implementation steps, security/accessibility checklists, validation gate, rollback instructions
- Post-execution checklist

**Commit this alongside your code** for traceability.

---

## Layer Model

EMA determines chunk boundaries by identifying which architectural layers the project spans:

| Layer | Standard | Embedded / Firmware | Data Pipeline |
|-------|----------|---------------------|---------------|
| 0 — Foundation | Data models, auth, config | BSP, HAL, MCU init | Source connectors, schema |
| 1 — Core Logic | Business logic, APIs, services | RTOS tasks, drivers, protocols | Transform, enrichment, validation |
| 2 — Delivery / IO | UI, integrations, wiring | Application, host comms | Sinks, serving layer |

Layer 2 is absent for headless, pure CLI, pure backend microservice, and standalone firmware projects. EMA reduces the chunk count accordingly.

---

## Context7 Integration

AME uses the [context7 MCP server](https://github.com/upstash/context7) to fetch live library documentation when you name a specific framework. It fires **once, before presenting the interview questions**, so constraints inform how AME compiles the questions.

- Capped at **3 call-pairs per session**
- If context7 is unavailable, AME logs it in the spec and continues — the interview is never blocked

To enable context7, add this to your VS Code MCP configuration:

```json
"context7": {
  "command": "npx",
  "args": ["-y", "@upstash/context7-mcp@latest"],
  "type": "stdio"
}
```

---

## Contributing

Pull requests are welcome. Please keep skill changes backward-compatible with the `.ame/spec-template.md` format — EMA's parser depends on consistent section headers ([A]–[E] and [6]).

When changing the spec format:
1. Update `.ame/spec-template.md`
2. Update AME's Step 2 spec write template
3. Update EMA's Step 1 spec read references

---

## Licence

MIT
