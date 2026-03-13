---
name: session-manager
description: Diagnose session health and lifecycle risk for active or recent sessions. Use when a user asks to inspect session state, detect bloated sessions, find orphaned sessions, identify abandoned recovery attempts, assess growing verbosity or repeated answers, distinguish provider/server errors from session-structure problems, decide whether to continue, summarize, restart, archive, or hand off, or choose the canonical session among multiple related sessions.
---

# Session Manager

Inspect session health, classify lifecycle risk, and recommend the next operating action.

This skill is for **session health management**, not simple session listing.

## Core goal

Treat sessions as things that need lifecycle management.

Optimize for:
- keeping healthy sessions healthy
- catching early warning signs before degradation compounds
- summarizing before context usage becomes dangerous
- restarting or handing off when needed
- preserving a single clear source of truth

A long-lived session is **not automatically** a healthy one.

## Default workflow

### 1) Inspect
Check the relevant session(s) and gather signals.

Look for:
- owner
- current task
- recent activity
- next-step clarity
- message volume
- response length trend
- repetition pattern
- recovery attempts
- provider/tool/server error signs
- approximate context usage when available

Use available session/status/history tools as needed.
Keep inspection lightweight unless the user asks for deep diagnosis.

### 2) Diagnose
Classify the session into one or more states.

Prefer evidence over intuition.
If multiple risks apply, say so explicitly.
Always distinguish:
- **external failures**
- **session structure / health failures**

### 3) Recommend
Return an action recommendation, not just a description.

Typical recommendation types:
- continue
- summarize_and_continue
- prepare_handoff
- summarize_and_restart
- choose_canonical_session
- mark_reference_only
- archive_reference
- retry_after_external_error
- ask_user_to_switch_session

### 4) Report
Return a short structured report.
If useful, append a handoff block.

---

## Percent-based operating rules

Apply these rules conservatively. Do not use them mechanically.

- **0-69%** → normal operation; usually `healthy` on size alone
- **70-84%** → inspect for early drift; likely `early_warning` if repetition, verbosity growth, or context drag is visible
- **85-89%** → prepare handoff/summary now; default to `prepare_handoff` or `summarize_and_continue`
- **90-94%** → prefer fresh-session transition; default to `summarize_and_restart` unless the task is about to finish
- **95%+** → treat as effectively overfull; avoid continuing normal work unless there is a strong reason

When context usage is high **and** outputs are repetitive, escalate faster.

## Health signals

### A. Size signals
Treat these as “session is getting heavy” signals:
- high message count
- growing reliance on long prior context
- repeated re-explanation of background
- token/context utilization crossing warning thresholds

### B. Cost signals
Treat these as efficiency warnings:
- token usage is disproportionate to task size
- small questions produce large responses
- repeated context drag reduces cost/performance efficiency

### C. Activity signals
Treat these as lifecycle/ownership signals:
- session still exists but little real work is happening
- no recent meaningful update
- owner is unclear
- next action is unclear
- completion/interruption was never reported

### D. Recovery signals
Treat these as “repair flow got messy” signals:
- a recovery attempt clearly started
- temporary or retry sessions exist
- recovery completion is missing
- canonical/source-of-truth session is unclear

### E. Quality signals
Treat these as user-visible early warnings:
- replies become increasingly verbose
- action content shrinks while explanation grows
- simple questions trigger large context reuse
- the same point is repeated with little new progress
- decision/execution lags behind narration

### F. Repetition signals
Treat these as strong health indicators:
- the same answer structure repeats across turns
- new input produces little meaningful update
- retry attempts keep regenerating the same content loop

### G. External error signals
Treat these as provider/tool-side indicators first:
- `server_error`
- provider failure
- tool backend instability
- temporary remote processing errors
- repeated request-id-bearing failures

Do **not** assume these are session health problems by default.

---

## Session states

### `healthy`
Use when:
- owner is clear
- recent work is real and active
- response size feels proportionate
- source of truth is clear
- no material repetition or degradation

Recommended action:
- `continue`

### `early_warning`
Use when:
- verbosity is rising
- repetition is starting
- context reuse feels too heavy
- quality is slipping, but not severely yet

Recommended action:
- `summarize_and_continue`

### `handoff_ready`
Use when:
- the session is still workable
- but context usage is high enough that transition preparation should happen now
- continuing without a summary would likely create recovery pain later

Recommended action:
- `prepare_handoff`

### `bloated`
Use when:
- session size/cost is clearly excessive
- response quality or speed is degraded
- repeated background explanation dominates
- repetition and verbosity are no longer minor

Recommended action:
- `summarize_and_restart`

Secondary action:
- `mark_reference_only`

### `orphaned`
Use when:
- session is still around
- owner is unclear or absent
- no next step exists
- no completion/stop state was properly recorded

Recommended action:
- `mark_orphan`
- optionally `reassign_owner`

### `recovery_abandoned`
Use when:
- recovery was attempted
- retry/temporary sessions were created
- the recovery flow stopped halfway
- canonical session is unclear

Recommended action:
- `choose_canonical_session`

Also identify:
- canonical session
- secondary sessions
- what to keep as reference only

### `stale_reference`
Use when:
- session is no longer active
- but still contains useful context/history
- it should be preserved as reference, not kept active

Recommended action:
- `archive_reference`

### `external_error`
Use when:
- provider/server/tool failure is the primary issue
- there is not enough evidence yet to call it a session health failure

Recommended action:
- `retry_after_external_error`

---

## Canonical-session rules

Use these whenever multiple related sessions exist.

- Prefer **one canonical session per task**.
- Treat all non-canonical sessions as **secondary** or **reference-only**.
- If a new session takes over, say so explicitly.
- Do not leave ownership ambiguous.
- If recovery or branching happened, always name:
  - canonical session
  - secondary/reference sessions
  - next action
  - blocker

## Fresh-session rules

When recommending or preparing a new session:
- do **not** tell the next session to ingest the full transcript by default
- prefer minimal restore context only
- carry forward only:
  - canonical summary
  - done
  - remaining
  - next action
  - blocker
  - key decisions

## Reference-only rules

When a session is superseded:
- mark it as `reference-only`
- do not recommend continuing active work there unless necessary
- preserve it for lookup, not execution

---

## Compound-state rule

Do not stop at a single label when the evidence suggests compound risk.

Examples:
- `external_error + early_warning`
- `external_error + bloated`
- `external_error + recovery_abandoned`

Rule of thumb:

> Classify provider/server failure separately first.  
> If repeated errors occur together with repetition, growing verbosity, slowdown, or recovery confusion, also evaluate session bloat or abandoned recovery risk.

---

## Output format

Use this default report shape:

```md
Session Health Report
- Target:
- Status:
- Context usage band:
- Confidence:

Signals
- ...
- ...

Assessment
- ...

Recommended Action
- ...

Next Step
- ...
```

If handoff, branching, or recovery clarification is needed, append:

```md
Session Handoff
- Task:
- Canonical session:
- Secondary/reference sessions:
- Owner:
- Stage:
- What is done:
- Remaining:
- Blocker:
- Recommended next action:
```

Keep the report concise.
Prefer practical action over essay-length explanation.

---

## Decision heuristics

When unsure:
- favor `early_warning` over `bloated`
- favor `handoff_ready` over `bloated` when the session is still coherent but close to the edge
- favor `external_error` over structural blame when only server/tool errors are visible
- favor “summarize first” over “restart immediately”
- favor “choose canonical session” when recovery ambiguity exists
- favor “archive reference” over “delete context”

## Operating rules

Use this skill with a conservative operating posture.

- Default to **manual, symptom-based invocation** rather than always-on monitoring.
- Do **not** run a full session-health check before every normal message.
- Prefer summarize/continue or summarize/restart over destructive cleanup.
- Escalate faster when both are true:
  - context usage is high
  - recent outputs are materially repetitive or add little new progress
- If one related session is unhealthy but another is healthy, explicitly recommend the healthy session as the continuation target.

## Example user requests

- "Can you run a health check on the sessions?"
- "Are any of these sessions getting too bloated?"
- "Should I move this over to a fresh session?"
- "Which session is the canonical one now?"
- "Mark the old one as reference only."
- “세션 상태 점검해줘”
- “너무 길어진 세션 있나?”
- “이 작업 새 세션으로 넘기는 게 좋을까?”
- “정본 세션이 어디야?”
- “이전 세션은 참고용으로만 둬야 하나?”

## Final definition

This skill diagnoses **session health, efficiency, source-of-truth clarity, and lifecycle risk**, then recommends the right operating move:
**continue, summarize, hand off, restart, archive, reassign, or retry**.
