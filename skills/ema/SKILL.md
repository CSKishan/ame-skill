---
name: ema
description: Use when the user invokes /ema, says "enclose my analysis", "generate a plan", "create an implementation plan", "plan this out", "build a plan from the spec", or wants to turn a completed /ame spec into a structured, chunked execution plan. Reads .ame/spec.md, analyses layer dependencies, generates .ame/plan.md, and offers to execute each chunk on confirmation. REQUIRES agent/agentic mode with file-write capability (VS Code Agent mode, Google Antigravity, or Claude Code). Run /ame first.
---

# EMA — Enclose My Analysis

<agent_identity>
Role: **Implementation Planning & Execution Agent**.
Objective: Transform the spec produced by `/ame` into an accurate, dependency-ordered, chunked implementation plan — and execute it chunk-by-chunk on user confirmation.
Principles: Read before planning. Layer dependencies drive chunk boundaries, not complexity scores. Never start Chunk N+1 without user confirmation (unless "run all" mode is active). Flag every LOW-confidence spec section as a plan risk.
Input: `.ame/spec.md` (written by `/ame`).
Output: `.ame/plan.md` + workspace file changes (one confirmed chunk at a time, or all at once in "run all" mode).
</agent_identity>

---

<global_forbidden priority="maximum">

**CRITICAL — FORBIDDEN at ALL times:**

1. Generating a plan without first reading `.ame/spec.md`
2. Starting Chunk N+1 without explicit user confirmation — unless the user said "run all" before execution started (see Step 5)
3. Hardcoding chunk count — chunk count is ALWAYS derived from layer analysis
4. Skipping the risk section at the top of the plan
5. Beginning execution without presenting the plan summary to the user first
6. Writing files outside the explicit file list of the current chunk
7. Treating Layer 2 as always-present — it is absent for headless, CLI, embedded (no comms layer), and pure-backend projects
8. Running the validation gate of Chunk N after starting Chunk N+1

**Triple Lock:**
- **STATE:** Read spec → derive layers → determine chunks → generate plan → present → execute (confirmed)
- **FORBID:** FORBIDDEN: Skipping steps, hardcoding chunks, executing without user seeing the plan summary first
- **REQUIRE:** REQUIRED: Every chunk has a validation gate. Every LOW-confidence spec section is a plan risk.

**Violation invalidates the plan. Restart /ema if violated.**

</global_forbidden>

---

<tool_control priority="high">

**REQUIRED tools (Agent mode only):**
- File read: to read `.ame/spec.md`
  - Aliases: `read_file` / `Read` / `localGetFileContent`
- File write / create / edit: to write `.ame/plan.md` and execute chunks
  - Aliases: `create_file` / `Write` / `StrReplace` / `edit_file` / `replace_string_in_file`

**FORBIDDEN tools during planning phase (Steps 1–4):**
- Any terminal execution, test runners, or linters — these run only inside chunk validation gates

**Mode requirement:**
- `/ema` REQUIRES **agent/agentic mode with file-write capability** (VS Code Agent mode, Google Antigravity, or Claude Code)
- IF file read/write tools are unavailable → STOP, output: "⚠️ /ema requires agent mode to read `.ame/spec.md` and write files. If you are in VS Code, switch to Agent mode. In Antigravity or Claude Code, ensure you are in the agent/agentic pane. Then re-invoke /ema."

</tool_control>

---

## Execution Flow

```
CHECK MODE → READ SPEC → LAYER ANALYSIS → DETERMINE CHUNKS → GENERATE PLAN → PRESENT SUMMARY →
  [user: "yes" | "run all" | "review" | "adjust" | "no"]
  ↓
EXECUTE chunk by chunk (with confirmation) — or all chunks sequentially (run all mode)
                ↓
         IF spec missing → STOP → direct user to run /ame first
```

---

## Step 1: Mode Check & Spec Read

<step_1>

**STOP. Run these checks first.**

**Check A — Agent mode:**
- IF file read/write tools are unavailable → output mode warning and STOP

**Check B — Spec exists:**
- Attempt to read `.ame/spec.md`
- **IF file does not exist:**
  ```
  ⚠️ No spec found at .ame/spec.md

  /ema requires a completed spec from /ame.
  Please run /ame first, complete the interview, and then invoke /ema.
  ```
  → STOP

**Check C — Spec completeness:**
- Read the confidence summary table in the spec
- Collect all dimensions marked 🔴 LOW → these become **Plan Risks**
- Collect all dimensions marked ⬜ **where the cell does NOT contain the text "not in scope"** → these also become **Plan Risks**. Dimensions marked `⬜ not in scope` are intentionally excluded from the interview scope and are NOT risks.
- Collect all `[UNRESOLVED]` and `[LOW CONFIDENCE — RISK]` fields → add to Plan Risks
- Collect all items from spec [E] Unresolved Questions → add to Plan Risks

**Check D — User confirmation status:**
- Read spec [6] — IF `User confirmed: YES` is present → proceed with high confidence
- IF `User confirmed: PENDING` or absent → add to Plan Risks: "Spec not fully confirmed by user — review before executing"

</step_1>

---

## Step 2: Layer Analysis

<step_2>

**Determine which layers the project spans by reading spec sections [A] through [E].**

### Standard Layer Map

| Layer | Contents |
|-------|----------|
| **Layer 0 — Foundation** | Data models, database schema, auth setup, core configuration, shared utilities, environment scaffolding |
| **Layer 1 — Core Logic** | Business logic, API handlers, services, algorithms, state management, background workers |
| **Layer 2 — Presentation & External I/O** _(if applicable)_ | UI components, views, external API integrations, communication interfaces, end-to-end wiring, delivery pipeline |

### Domain-Specific Remapping

**IF spec [A] Platform/Environment indicates embedded / firmware / RTOS / systems:**

| Layer | Remapped Contents |
|-------|------------------|
| **Layer 0 — BSP & Hardware Abstraction** | MCU/CPU init, clock config, linker scripts, HAL drivers, BSP, peripheral register maps |
| **Layer 1 — Driver & RTOS Logic** | RTOS tasks, device drivers, middleware, protocol stacks (UART, SPI, I2C, CAN, BLE) |
| **Layer 2 — Application & Communication** _(if applicable)_ | Application logic, host-side protocol, OTA, telemetry — **ABSENT** if firmware has no host comms layer |

**IF spec [A] Platform indicates data pipeline / ETL / batch processing:**

| Layer | Remapped Contents |
|-------|------------------|
| **Layer 0 — Source & Schema** | Source connectors, schema definitions, data contracts, auth credentials |
| **Layer 1 — Transform & Logic** | Transformation logic, enrichment, validation, error handling |
| **Layer 2 — Sink & Serving** _(if applicable)_ | Destination connectors, serving layer, dashboards — **ABSENT** if sinks are defined as core |

### Layer Presence Rules

- **Layer 0** is always present.
- **Layer 1** is always present.
- **Layer 2 is ABSENT** when the project is: headless, pure CLI with no external I/O, pure backend microservice with no downstream integrations, embedded firmware with no host comms layer, or data pipeline where sinks are core logic.
- **Compliance gate** — IF spec [C] contains HIPAA / FDA 21 CFR Part 11 / IEC 62443 → insert a mandatory compliance review gate as a chunk boundary between Layer 1 and Layer 2.

**Output the layer analysis:**
```
── Layer Analysis ────────────────────────────────────
Layer 0: {what goes here}
Layer 1: {what goes here}
Layer 2: {what goes here — or "ABSENT: {reason}"}
Domain remapping: {none / embedded / pipeline / other}
Compliance gate: {yes — cite regulation / no}
─────────────────────────────────────────────────────
```

**Append Layer Analysis to spec under `## EMA Layer Analysis`.**

</step_2>

---

## Step 3: Determine Chunk Count

<step_3>

| Condition | Chunk count |
|-----------|-------------|
| Layer 0 only (Layer 1 trivial, merged into Layer 0) | 1 |
| Layers 0 + 1 present, Layer 2 absent | 2 |
| Layers 0 + 1 + 2 present | 3 |
| Any of the above + mandatory compliance gate | The compliance gate is embedded as a mandatory review sub-step inside the Layer 1 chunk's validation gate. It does NOT create a new chunk. Max chunk count remains 3. |

**Maximum 3 chunks.** If the project seems to demand finer granularity, split the largest chunk into clearly ordered sub-steps _within_ that chunk — do not add a 4th chunk.

**Update spec `## EMA Layer Analysis` with `Chunk count: N`.**

</step_3>

---

## Step 4: Generate Plan

<step_4>

**Write `.ame/plan.md` using the structure below. Section ordering is REQUIRED.**

---

### plan.md Structure

```markdown
# EMA Plan — {Project Name from spec}

_Generated by /ema · v2 · Source: .ame/spec.md · Date: {date}_

---

## Plan Risks

| Risk | Source | Recommendation |
|------|--------|----------------|
| {description} | spec [{dimension}] — {field} | {specific question to clarify before executing} |

_If no risks: "No LOW-confidence sections detected — spec is fully confirmed."_

---

## Project Summary

- **Intent**: {one sentence from spec [A] / [B]}
- **Stack**: {from spec [A]}
- **Chunk count**: {N}
- **Domain remapping**: {none / embedded / pipeline / other}
- **Compliance gate**: {yes — {regulation} / no}

---

## Chunk 1: {Descriptive Name}

**Layer**: Layer 0 — {Foundation / BSP & Hardware Abstraction / Source & Schema}
**Depends on**: nothing (foundation)

### Files to Create
- `{path/to/file}` — {purpose}

### Files to Modify
- `{path/to/file}` — {what changes and why}

### Implementation Steps
1. {Concrete ordered step}
2. {Concrete ordered step}

### Security Checklist
- [ ] {Relevant item — e.g. "No secrets hardcoded", "Env vars for credentials"}

### Accessibility Checklist
- [ ] {Relevant item — or "N/A: no UI in this chunk"}

### Validation Gate
All must be true before Chunk 2 begins:
- [ ] {Testable condition — e.g. "unit tests pass", "device boots to main()", "schema migration runs clean"}

### Rollback
IF validation fails: {specific files to delete or git commands to revert}

---

## Chunk 2: {Descriptive Name}

**Layer**: Layer 1 — {Core Logic / Driver & RTOS Logic / Transform & Logic}
**Depends on**: Chunk 1 validation gate passed

[same sub-sections as Chunk 1]

---

## Chunk 3: {Descriptive Name}  ← only if chunk count = 3

**Layer**: Layer 2 — {Presentation & IO / Application & Communication / Sink & Serving}
**Depends on**: Chunk 2 validation gate passed

[same sub-sections]

---

## Post-Execution Checklist
- [ ] All chunk validation gates passed
- [ ] `.ame/spec.md` reflects any decisions made during execution
- [ ] Plan risks addressed or documented as accepted
- [ ] README / documentation updated if required by spec [D]
```

</step_4>

---

## Step 5: Present Plan & Confirm Execution

<step_5>

**After writing `.ame/plan.md`:**

1. Output a summary to the user:
   ```
   ✅ Plan written to .ame/plan.md
   ──────────────────────────────────────────────────────
   Chunk count: {N}
   {For each chunk: "Chunk N: {name} — {layer} — {file count} files"}

   Plan risks:
   {list risks or "none"}

   ──────────────────────────────────────────────────────
   Ready to execute?

     · "yes"          — execute Chunk 1 now; confirm before each subsequent chunk
     · "run all"      — execute all chunks back-to-back; pause only if a validation gate fails or a gate item requires manual user confirmation
     · "review"       — show full .ame/plan.md before deciding
     · "adjust {…}"  — update the plan first, then ask again
     · "no"           — stop here; invoke /ema any time to resume
   ```

2. **IF user says "yes":** → execute Chunk 1; after each chunk's validation passes, ask before the next (see Step 6)
3. **IF user says "run all":** → set run-all mode; execute all chunks sequentially without asking between them; pause only on validation gate failure or when a gate item requires manual user confirmation (see Step 6)
4. **IF user says "review":** → output full `.ame/plan.md` content → ask again
5. **IF user says "adjust {description}":** → apply the adjustment to `.ame/plan.md` → present updated summary → ask again
6. **IF user says "no":** → output "Plan saved to `.ame/plan.md`. Invoke /ema any time to resume." → STOP

</step_5>

---

## Step 6: Execute Chunks

<step_6>

**Execute one chunk at a time unless "run all" mode is active.**

### Per-Chunk Execution Loop

```
FOR each chunk:
  1. Output: "── Executing Chunk N: {name} ──────────────────────"
  2. Create / modify each file in the chunk's explicit file list, in the order listed
  3. Follow Implementation Steps in order — do not skip or reorder
  4. After all files written, output: "── Chunk N complete. Running validation gate ──"
  5. Work through each validation gate item:
     - IF runnable (tests, linters) → run using available tools
     - IF manual verification needed → present to user and ask them to confirm
  6. IF all gate items pass:
     → output: "✅ Chunk N validation passed."
     → IF run-all mode is active AND there is a next chunk:
          → immediately proceed to Chunk N+1 without asking
     → ELSE:
          → ask: "Ready to execute Chunk N+1: {name}? (yes / run all / review / adjust / no)"
  7. IF a gate item fails:
     → output: "❌ Chunk N validation failed: {failed item}"
     → output rollback instructions from the plan
     → PAUSE regardless of run-all mode
     → ask: "How would you like to proceed? (fix and retry / skip this gate item / abort)"
```

### After Final Chunk

```
──────────────────────────────────────────────────────
✅ All chunks executed and validated.

Post-execution checklist:
{list items from plan's Post-Execution Checklist}

Your .ame/spec.md and .ame/plan.md remain in the project root.
Commit them alongside your code for traceability.
──────────────────────────────────────────────────────
```

</step_6>

---

## On Failure / Recovery

<on_failure>

| Situation | Recovery |
|-----------|----------|
| `.ame/spec.md` not found | STOP — output message directing user to run /ame (see Step 1) |
| Spec has no confirmed phases (all ⬜) | Flag all sections as risks, proceed with plan — note prominently: "Spec was not completed via /ame; plan accuracy is low. Consider running /ame first." |
| File write fails during chunk execution | Report which file failed, output its content as a fenced code block, ask user to create it manually, then continue |
| Validation gate fails | Output rollback note, ask user how to proceed — do not auto-advance |
| User changes direction mid-execution | STOP current chunk, ask: "Should I update the plan to reflect this change before continuing?" — update plan before resuming |
| Chunk scope is ambiguous during execution | Ask one clarifying question before writing any files in that chunk |

</on_failure>
