```yaml
lens: fix-affected-set
outcome: reported
```

Task: lens fix-affected-set, checking whether the F4, F1 and F6 fixes cover every site in their sets.

No findings from the F1, F3 and F6 searches. The F4 search found stale twins of the clause that F4 rewrote.

**F4: twins of the rewritten clause**

- **LOW, `internal/loomcli/rownotice.go:49`.** It says "A re-call after an escalation's answer or a resumed halt is a new visit, since it spawns a fresh agent." After the diff, a resumed halt whose agent ended early resumes that agent's session, so "spawns a fresh agent" is stale. The visit count still holds.
- **LOW, `docs/overview.md:1510`.** It repeats the "re-call after an escalation's answer or a resumed halt" clause for the row-reached mail.
  - Whether this is stale depends on the lines around it, which I didn't read.
  - The mail claim itself is still correct.
- **No finding, `internal/shedrun/rownotice.go:17`, `internal/loomengine/config.go:566`, `internal/loomengine/template.yaml:62`.** They describe only when mail is sent, not what the agent is, so they stay correct.

**F1: sites using `spec`, `resuming`, `resumeWarning`, `transcriptCopied` in `start`**

- Every site is covered: `launch` at `run.go:661`, the notice guard at `run.go:666` and the reset at `run.go:675`.
- `takesPlace` depends only on `pred` and the original `spec.Unawaited` and `OutputFiles`, which the resume does not change.
- `startRetryable` sets `resuming` back to false only on the failed-resume branch, so the success path still skips the carried notice only for a resumed start.

**F3 and F6: sites calling `stopRecorded` / `Interrupted`**

- The production callers are `attach.go:240` (running outcome), `stepstop.go:158` (true) and `stop.go:158` (false).
- `Run.Stop` in `stop.go` never sets `Interrupted`, so the F6 assertion in the handle form is also right.

**Unchecked**

- F7: it names the one test.
- F8, F9: they are a single header and a single line.
- F2: it is a decision-class finding held by the parent.

Calls made:
1. `grep -rniE "resumed halt|fresh agent"` over md, go and yaml, because my context held only the one-phrase grep for the F4 twins.
2. `grep -rn "stopRecorded(\|resuming\|resumeWarning\|freshSpec" internal`, because my context lacked the full caller set for the F1, F3 and F6 checks.
