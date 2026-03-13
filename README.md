# session-manager
Agent skill for session health diagnosis: detect bloated, orphaned, or recovery-abandoned sessions and recommend summarize, restart, handoff, or archive actions.

## Install

If you're installing it directly from GitHub with the skills CLI, the simplest command is:

```bash
npx -y skills add https://github.com/RichardHojunJang/session-manager
```

If you want to be explicit about the skill name, use:

```bash
npx -y skills add https://github.com/RichardHojunJang/session-manager --skill session-manager
```

After install, the skill should be available under your global skills directory.

## What it does
`session-manager` helps an agent decide whether a session is still healthy or whether it should be summarized, handed off, restarted, archived, or otherwise managed.

It focuses on practical session-health problems such as:
- bloated sessions
- orphaned sessions
- abandoned recovery attempts
- growing verbosity
- repeated answers
- external provider errors vs session-structure problems

## Key states

- `healthy`
- `early_warning`
- `bloated`
- `orphaned`
- `recovery_abandoned`
- `stale_reference`
- `external_error`

## Recommended actions

Depending on the diagnosis, the skill recommends actions such as:
- `continue`
- `summarize_and_continue`
- `summarize_and_restart`
- `mark_orphan`
- `reassign_owner`
- `choose_canonical_session`

## Example prompts

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
- “같은 답변 반복하는데 세션 문제야?”
- “server_error가 세션 문제인지 외부 문제인지 구분해줘”
- “Check whether this session is getting bloated and tell me if I should summarize, restart, or hand it off.”

## Notes
This skill is designed as a session-health and lifecycle management skill, not just a session-listing utility.
