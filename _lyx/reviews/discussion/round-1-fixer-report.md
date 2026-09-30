# Discussion-Review round 1 — fixer report

All ten findings fixed; `lyx loom validate-discussion` exits 0 afterwards.

## Changes

- **F1** — D4 now names the identity as the pair (`vcs.revision`, `vcs.modified`) and compares both fields.
  The dev-build Auto-mode assumption is rewritten to match: successive dev builds from a still-modified tree at one revision compare equal; moving the revision or flipping the flag re-steps.
- **F2** — D1 retitled "Go-classified; the driver never text-matches".
  Its Decision now allows the Go failure site to inspect the failed call's own output (git stderr, HTTP status), with unrecognised output not transient, and bars text matching in the skill.
  The Open risks error-text entry now points at D1.
- **F3** — D8 gains the in-flight resume rule: no second step, and the line's report path and budget reset apply to the next stop.
  The Open risks race entry now points at that rule.
- **F4** — D5 scopes the guard to the task's code worktree (the warp), never the weft `_lyx` or `.lyx`, and states that a skipped re-step waits for the next binary change or a resume.
- **F5** — D6 starts the watch only when the stop is D5-eligible, matching D10.
- **F6** — D8's live-driver-without-marker branch spelled out: no `Send`, no spawn, then the usual handover.
- **F7** — Constraints gains a Buildinfo Leaf Invariant entry.
- **F8** — The `SKILL.md` acceptance criterion enumerates the skill-only behaviours from D6, D7, D8, D9, D10 and D11.
- **F9** — `manifest/HANDOFF.md` moved from Open risks to Scope Out; `support-log.md` Q15 updated to say so.
- **F10** — Notes write `shuttleengine.Runner.Send` without its parameter list.

## Deferred

None.
