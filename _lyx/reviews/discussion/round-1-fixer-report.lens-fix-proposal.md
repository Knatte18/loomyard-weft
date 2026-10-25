```yaml
lens: fix-proposal
outcome: reported
```

Lens: fix-proposal. I challenged the proposals the fixer carried out as written: F6, F8, F9, F10, F11, F12, F13, F14, F15 and F16.

- **MEDIUM** — D7 refusal bullet: "…or `lyx tender pause <child>` first so no new full pass of it starts."
  - Summary: the way forward taken from F8's proposal does not clear the refusal on an active beam. A pause stops only full passes, the dobby is still live while the refusals run, and it keeps starting continue passes. A child sitting at Run-Shed can therefore have a pass in flight at every attempt, so the verb is refused indefinitely and "wait and re-run" is not guaranteed to make progress.
  - Evidence: `internal/dobbyengine/doc.go` says "While a pause covers the run, only continue passes run" and "A continue pass needs the run at Run-Shed with no full pass owed". It also says a pass is triggered "by a probe tick once per probe interval".
  - In the changed D7, the dobby is ended only in step 2, after the refusals. Nothing stops it starting passes between attempts.

Checked and dismissed:
- F10: a detached push for a local-only pair is not a problem. Pushes already skip local-only branches (`internal/fabricengine/commit.go:112`, `coalesce.go:60-69`, `spawn.go:150`).
- F6, F12, F13, F14, F15 and F16: I found no concrete case where the proposed change is wrong.
- F9: the clause "the replay result in `settleUnmovedWeft`" is subsumed by the added "upstream has commits the local tip lacks" clause. It is redundant but harmless.

Tool calls:
1. grep of `internal/fabricengine` for local-only push handling, to test whether F10's reconcile push could push a local-only pair. My context had no push-side local-only rule.
