# Conversation Continuity

Produce a self-contained Markdown block structured as follows. It must be complete enough to paste into a brand new conversation and resume with zero loss.

---

## Output format

~~~markdown
## Conversation Snapshot

### Objective
<!-- One sentence: what we are building / solving. -->

### Context & Decisions Made
<!-- Bullet list of every key decision, constraint, chosen approach, rejected alternative (with reason). -->

### Current State
<!-- Exact step we stopped at. What is done. What is NOT done yet. -->

### Next Action
<!-- The very next concrete step to execute. Be surgical. -->

### Artifacts
<!-- For each file / snippet produced: filename, full content or diff, purpose. -->

### Environment & Stack
<!-- Languages, frameworks, versions, OS, tooling, env vars, paths — anything non-obvious. -->

### Blockers / Open Questions
<!-- Unresolved issues, pending decisions, things to watch out for. -->

### Resume Prompt
<!-- A ready-to-paste first message for the new conversation. Must include: role/persona to adopt, objective, current state, next action. -->
~~~

---

## Rules

- No summarization. Full fidelity. If in doubt, include it.
- Code blocks must be complete and runnable, not truncated.
- The **Resume Prompt** section must be standalone — assume the new session has zero memory.
- Output the snapshot in a single fenced block, ready to copy.