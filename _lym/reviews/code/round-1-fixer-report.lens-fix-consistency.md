```yaml
lens: fix-consistency
outcome: reported
```

Task: check the F1–F3 change set for siblings or twins left stale (lens fix-consistency).

**Finding 1: LOW**
- **Changed text:** `internal/dobbyengine/starts.go:89`, the hold branch now applies only when the task is not owned elsewhere, has no pending dependencies and is not covered by a pause.
- **Summary:** the sibling statement in `internal/dobbyengine/doc.go:97-99` still says the hold is unconditional.
- **Evidence:** `doc.go` reads "A released task whose slug has a finished run record carries the refusal every start of that slug gets… Dobby spawns no pass for it and records the refusal… with the same one notice per refusal text". Nothing there says the record and the notice apply only to a task this dobby would otherwise start. A task owned elsewhere, with open dependencies or under a covering pause now gets no record and no notice.
- **Fix:** add that qualifier to the `doc.go` bullet.

**Finding 2: LOW**
- **Changed text:** the `heldStart` doc comment at `internal/dobbyengine/starts.go:212` and the `startReady` doc comment at lines 55-59.
- **Summary:** neither comment names the new gating, and `startReady`'s comment doesn't mention the hold at all.
- **Evidence:** the `heldStart` comment says only "records the refusal task carries… and posts one notice per refusal text". The condition that decides when it runs now lives only in the call site at line 89. This is minor, since the `doc.go` fix would cover the behavior.

**Checked and consistent**
- The `beamcli/cli.go` help text now matches the `battencli/cli.go` package doc (a start batten refuses, and the last refused start on a ready task).
- `overviewdobby.go:123` still renders the reason.
- `view.go` `WaitingTask`/`IsReady` still includes `StartRefusal`.
- The new `view_test.go` row matches the `heldByEntry` and `BuildView` behavior.
- No twin of the old help sentence exists elsewhere (grep found only `cli.go:74`).

**Tool calls**
1. `grep` for "refused start and its retry" across `.go` and `.md` files. I ran it because my context did not show whether a twin of the old help sentence exists elsewhere. It found none.
