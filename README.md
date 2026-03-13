# session-manager

Agent skill for diagnosing session health and session-lifecycle risk.

`session-manager` helps an agent decide whether a session should:
- continue normally
- be summarized and continued
- prepare a handoff
- move to a fresh session
- become `reference-only`
- be treated as the canonical session for a task

## What it solves

Long-running sessions often degrade in predictable ways:
- verbosity grows
- repetition appears
- context cost becomes disproportionate
- recovery attempts create multiple competing sessions
- the source of truth becomes unclear

This skill turns those symptoms into a practical recommendation.

## Core operating model

The skill now includes explicit percent-based session rules:

- **0–69%**: normal operation
- **70–84%**: inspect / likely `early_warning`
- **85–89%**: prepare handoff / canonical summary
- **90–94%**: prefer fresh-session transition
- **95%+**: avoid extending unless absolutely necessary

It also enforces:
- **one canonical session per task**
- non-canonical sessions become **secondary** or **reference-only**
- fresh sessions should load **minimal restore context**, not the full transcript

## Key states

- `healthy`
- `early_warning`
- `handoff_ready`
- `bloated`
- `orphaned`
- `recovery_abandoned`
- `stale_reference`
- `external_error`

## Recommended actions

Depending on diagnosis, the skill recommends actions such as:
- `continue`
- `summarize_and_continue`
- `prepare_handoff`
- `summarize_and_restart`
- `choose_canonical_session`
- `mark_reference_only`
- `archive_reference`
- `retry_after_external_error`

## Output shape

Typical output includes:
- target session
- status
- context-usage band
- confidence
- observed signals
- recommended action
- next step

When needed, it also emits a handoff block containing:
- canonical session
- secondary/reference sessions
- owner
- stage
- done / remaining
- blocker
- next action

## Example prompts

- "Run a health check on this session."
- "Should I move this to a fresh session?"
- "Which session is canonical now?"
- "Mark the old one as reference-only."
- "Is this bloated or just a temporary provider error?"
- “세션 상태 점검해줘”
- “이 작업 새 세션으로 넘기는 게 좋을까?”
- “정본 세션이 어디야?”
- “이전 세션은 참고용으로만 둬야 하나?”

## Install

```bash
npx -y skills add https://github.com/RichardHojunJang/session-manager
```

Or explicitly:

```bash
npx -y skills add https://github.com/RichardHojunJang/session-manager --skill session-manager
```

## Files

- `SKILL.md` — main diagnosis and session-lifecycle rules
- `README.md` — overview and install instructions
