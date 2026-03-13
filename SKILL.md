---
name: session-manager
description: Diagnose session health and lifecycle risk for active or recent sessions. Use when a user asks to inspect session state, detect bloated sessions, find orphaned sessions, identify abandoned recovery attempts, assess growing verbosity or repeated answers, distinguish provider/server errors from session-structure problems, or decide whether to continue, summarize, hand off, restart, archive, or reassign a session.
---

# Session Manager

Inspect session health, classify lifecycle risk, and recommend the next operating action.

This skill is for **session health management**, not simple session listing.

## Core goal

Treat sessions as things that need lifecycle management.

Do not optimize for “keep the session alive forever.”
Optimize for:
- keeping healthy sessions healthy
- catching early warning signs
- summarizing before degradation gets worse
- restarting or handing off when needed
- preserving canonical state when recovery got messy

## Operating principle

A session is healthy when it can still:
- respond proportionately
- stay clear about ownership and next action
- avoid unnecessary repetition
- remain efficient relative to the task
- preserve a clear source of truth

A long-lived session is **not automatically** a healthy one.

## Default workflow

### 1) Inspect
Check the relevant session(s) and gather signals.

Look for:
- owner
- current task
- current stage (`plan`, `exec`, `verify`, or `fix`) when it can be inferred
- current status (`idle`, `running`, `waiting`, `blocked`, or `done`) when it can be inferred
- recent activity
- next step clarity
- message volume
- response length trend
- repetition pattern
- recovery attempts
- provider/tool/server error signs

Use available session/status/history tools as needed.
Keep inspection lightweight unless the user asks for deep diagnosis.

### 2) Diagnose
Classify the session into one or more states.

Prefer evidence over intuition.

If multiple risks apply, say so explicitly.

Always distinguish:
- **external failures**
from
- **session structure / health failures**

### 3) Recommend
Return an action recommendation, not just a description.

Typical recommendation types:
- continue
- summarize_and_continue
- summarize_and_restart
- mark_orphan
- reassign_owner
- choose_canonical_session
- archive_reference
- retry_after_external_error
- ask_user_to_switch_session

### 4) Report
Return a short structured report.
If useful, also provide a handoff block.

---

## Health signals

### A. Size signals
Treat these as “session is getting heavy” signals:
- high message count
- growing reliance on long prior context
- larger and larger replies over time
- repeated re-explanation of background
- token/context utilization crossing practical warning thresholds

#### Token threshold heuristics
Apply these heuristics to any session type unless a channel-specific rule overrides them:
- **<70% context used** → usually healthy on size alone
- **70-85%** → size `early_warning` candidate; inspect for repetition, verbosity growth, or slowing progress
- **85-92%** → strong `early_warning`; default toward `summarize_and_continue` unless the task is nearly done
- **>92%** → `bloated` candidate by default; if user-visible repetition or degraded responsiveness is also present, prefer `summarize_and_restart`

Do not use thresholds mechanically. Combine them with quality, repetition, and progress signals.

### B. Cost signals
Treat these as efficiency warnings:
- token usage appears disproportionate to task size
- small questions produce large responses
- repeated context drag reduces cost/performance efficiency

### C. Activity signals
Treat these as lifecycle/ownership and visibility signals:
- session still exists but has little real work happening
- no recent meaningful update
- owner is unclear
- next action is unclear
- completion/interruption was never reported
- current stage is unclear when it should be knowable (`plan`, `exec`, `verify`, `fix`)
- current working status is unclear when it should be knowable (`idle`, `running`, `waiting`, `blocked`, `done`)

### D. Recovery signals
Treat these as “repair flow got messy” signals:
- a recovery attempt clearly started
- temporary or retry session exists
- recovery completion is missing
- canonical/source-of-truth session is unclear

### E. Quality signals
Treat these as user-visible early warnings:
- replies become increasingly verbose
- action content shrinks while explanation grows
- simple questions trigger large context reuse
- same point is repeated in slightly different wording
- decision/execution lags behind narration

### F. Repetition signals

Treat these as strong health indicators:
- same answer structure repeats across turns
- new input produces little meaningful update
- the agent restates prior conclusions without progress
- retry attempts keep regenerating the same content loop

This is often an `early_warning` sign and can support `bloated` classification.

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

Goal:
Intervene before the session becomes bloated.

### `bloated`
Use when:
- session size/cost is clearly excessive
- response quality or speed is degraded
- repeated background explanation dominates
- repetition and verbosity are no longer minor

Recommended action:
- `summarize_and_restart`

Optional secondary action:
- keep old session as reference only

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
- but it still contains useful context/history
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

## Compound-state rule

Do not stop at a single label when the evidence suggests a compound risk.

Examples:
- `external_error + early_warning`
- `external_error + bloated`
- `external_error + recovery_abandoned`

Use compound assessment when:
- provider errors recur
- replies are getting repetitive
- verbosity is growing
- responses are slowing down
- recovery state is unclear

Rule of thumb:

> Classify provider/server failure separately first.  
> If repeated errors occur together with repetition, growing verbosity, slower responses, or recovery confusion, also evaluate session bloat or abandoned recovery risk.

---

## Action mapping

Map states to actions like this by default:

- `healthy` → `continue`
- `early_warning` → `summarize_and_continue`
- `bloated` → `summarize_and_restart`
- `orphaned` → `mark_orphan` or `reassign_owner`
- `recovery_abandoned` → `choose_canonical_session`
- `stale_reference` → `archive_reference`
- `external_error` → `retry_after_external_error`

If the user asks what to do next, answer directly.

Do not only say what the state is.
Always say what action is best.

---

## Guardrails

- Do not classify a session as bloated just because it is long.

- Do not mistake provider/tool errors for structure problems without supporting signs.
- Do not assume a recovery session is failed merely because it exists.
- Do not mark a session orphaned without checking owner, activity, next action, and reporting state together.
- Do not perform destructive cleanup or termination without explicit user approval.
- Prefer summarize/handoff/archive recommendations before kill/close recommendations.

---

## Output format

Use this default report shape:

```md
Session Health Report
- Target:
- Status:
- Confidence:
- Stage:
- Work status:

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

If handoff or recovery clarification is needed, append:

```md
Session Handoff
- Task:
- Canonical session:
- Secondary sessions:
- Owner:
- Stage:
- Work status:
- What is done:
- Blocker:
- Recommended next action:
```

When recovery, branching, or multi-session ambiguity exists, explicitly name:
- the **canonical session** that should be treated as the source of truth
- any **secondary/reference sessions** that should be kept only for context
- the current **stage** (`plan`, `exec`, `verify`, `fix`) when inferable
- the current **work status** (`idle`, `running`, `waiting`, `blocked`, `done`) when inferable
- the **next action** required to move work forward
- the main **blocker** if progress is currently stalled

Keep the report concise.
Prefer practical action over essay-length explanation.

---

## Decision heuristics

When unsure:
- favor `early_warning` over `bloated`
- favor `external_error` over structural blame when only server/tool errors are visible
- favor “summarize first” over “restart immediately”
- favor “choose canonical session” when recovery ambiguity exists
- favor “archive reference” over “delete context”

---

## Operating rules

Use this skill with a conservative operating posture.

- Default to **manual, symptom-based invocation** rather than always-on monitoring.

- Do **not** run a full session-health check before every normal message.
- When a session starts to feel off, prefer `early_warning` over `bloated` unless there is clear evidence of severe degradation.
- Treat repeated or near-identical answers as a strong signal that session fatigue, context drag, or recovery-loop behavior may be present.
- Prefer exposing a small amount of state visibility (`stage`, `work status`, `next action`, `blocker`) over forcing another agent or future session to reconstruct progress from scratch.
- When long-running work exists, try to distinguish whether the session is planning, executing, verifying, or fixing rather than treating all active work as one undifferentiated state.
- Classify provider/server/tool failures as `external_error` first, then upgrade to a compound assessment only when repetition, verbosity growth, slowdown, or recovery confusion also appears.
- Prefer summarize/continue or summarize/restart over destructive cleanup.
- Consider periodic checks only after enough real-world usage establishes what counts as a useful signal vs a false positive.

### Cross-session monitoring guidance
Apply the same health logic across **all session types**, not only Discord:
- direct chats
- group/channel sessions
- thread-bound sessions
- ACP or subagent sessions when their history is available

Do not assume the problem is platform-specific. A Telegram direct session, Discord channel session, or long-lived subagent can all become `early_warning` or `bloated`.

### Lightweight proactive checks
When asked to be proactive, use a lightweight pass instead of deep inspection:
- inspect active or recently updated sessions first
- flag sessions above threshold even if the user has not complained yet
- prefer `summarize_and_continue` at 70-92% when quality is still acceptable
- prefer `summarize_and_restart` above 92% when repetition, context drag, or loss of proportionality is visible
- if one session is unhealthy but another related session is healthy, explicitly recommend the healthy session as the continuation target

### Repetition escalation rule
Escalate faster when both are true:
- context usage is high
- recent outputs are materially repetitive or add little new progress

In that case, do not keep recommending indefinite continuation. Prefer a concrete restart/handoff recommendation.

## Example user requests

- "Can you run a health check on the sessions?"
- "Are any of these sessions getting too bloated?"
- "Look for any 'ghost' or orphaned sessions."
- "Are there any sessions stuck mid-recovery?"
- "Why are the answers getting longer and longer?"
- "It's stuck in a loop. Is something wrong with the session?"
- "Should I move this over to a fresh session?"
- "Can you tell if this server_error is coming from the session or something else?"
- "Give me some guidelines on how to handle sessions that eat up too many tokens."
- “세션 상태 점검해줘”
- “너무 길어진 세션 있나?”
- “고아 세션 찾아줘”
- “복구하다 만 세션 있는지 봐줘”
- “이 작업 새 세션으로 넘기는 게 좋을까?”
- “왜 답변이 점점 길어지는지 봐줘”
- “같은 답변 반복하는데 세션 문제야?”
- “server_error가 세션 문제인지 외부 문제인지 구분해줘”
- “토큰 많이 먹는 세션 관리 기준 정리해줘”
- “Check whether this session is getting bloated and tell me if I should summarize, restart, or hand it off.”

## Handoff and recovery rules

When a session is split, resumed, handed off, or partially recovered, do not only label the health state.
Also produce a minimal recovery structure that another agent or later session can immediately use.

Always prefer these seven elements when relevant:
- **Canonical session** — identify the one session that is the current source of truth
- **Secondary/reference sessions** — list any sessions that still matter for context but should not drive new work
- **Stage** — say whether the work is currently in `plan`, `exec`, `verify`, or `fix`
- **Work status** — say whether it is `idle`, `running`, `waiting`, `blocked`, or `done`
- **Next action** — state the single most useful next step in concrete terms
- **Blocker** — state what is currently preventing progress, if anything
- **Handoff/resume block** — provide a compact structured handoff when another agent, another session, or a future restart may need to continue

Do not force this structure into every healthy session report.
Use it when recovery ambiguity, branching, stalled progress, or handoff value is present.

## Final definition

This skill does not merely list sessions.

It diagnoses **session health, efficiency, source-of-truth clarity, and lifecycle risk**, then recommends the right operating move:
**continue, summarize, hand off, restart, archive, reassign, or retry**.
