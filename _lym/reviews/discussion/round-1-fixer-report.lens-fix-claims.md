```yaml
lens: fix-claims
outcome: reported
```

Task: check every claim the fix added or rewrote against the code at HEAD (fix-claims lens).

**MEDIUM: D4 step 2 makes the "no run holds it" test narrower than the code**

- **Changed text:** "A project qualifies when it is not done and no run holds it: its status is empty, or, for a beam, its producer is a hold-free row".
- **Problem:** Only a run-form status `<state> · <producer>` can hold a project. A project whose status is not run-form also qualifies, but the step rejects it. Examples are `draft` or any other plain status that is not done. Following the step, the orch would stop a move into such a project even though the board accepts the move.
- **Evidence:**
  - `internal/boardengine/runlock.go:30-38`: `RunStatusProducer` returns false for any status without the run separator.
  - `RunHolds` returns false whenever `RunStatusProducer` returns false.
  - `internal/boardengine/project.go:96-99`: the board refuses only a project that is done or that `RunHolds` reports as held.
- **Fix:** Replace "its status is empty" with "its status is not a run status (`<state> · <producer>`)".

**LOW: D4 step 3 assumes the archived tip is always the one the orch recorded**

- **Changed text:** "The code archive tag is `archive/<slug>/<first 12 hex of the tip>`, named from the tip recorded in step 1", then confirmed on origin with `ls-remote`.
- **Problem:** Suppose origin's tip strictly contains the local tip, for example after another machine pushed. The teardown child then pushes only origin's tip tag. The recorded tip's tag is created locally but never reaches origin, so the `ls-remote` check finds no tag. The step says nothing about this case.
- **Evidence:** `internal/fabricengine/taskbranchdiscard.go`, `archiveDiscardedTips`: when `localInOrigin` is true, it calls `archiveTaskTip(..., originTip, ...)` only. This should not happen under the hold, since only the task worktree pushes the branch, so the severity is LOW.

**Claims checked and found correct:**
- `cancel.go:299-304`: cancel refuses only a hub pause; the slug's own pause entries are set aside per the verdict comment at `cancel.go:86-87`.
- `cancel.go:572-576`: cancel lifts the slug's pause entries when it completes.
- The rerun text in the way forward has the form `"lym batten cancel <slug> --remove"` (`cancel.go:259`, `%q`).
- `ArchiveTagName` builds `archive/<slug>/<first 12 hex>` (`archive.go:78-79`).
- Remote-teardown failures are mailed to the orch with the manual tag push as the way forward (`remoteteardown.go:184-207`, `:243-255`).
- `runlock.go:44` holds `holdFreeProducers`, and the rows listed match.
- `project.go:99` is the "held by a run" refusal.
- `role-beams.md:61` says a member filed after `lym beam ready` never starts.
- `childHold.finish` returns before the remote half runs (`remoteteardown.go:319-333`), so `archive_tags` holds only the local tag.

**Tool calls:**
1. `cancel.go:284-320`, the rerun definition, `ArchiveTagName` and `NoticeRemoteTeardownEnded`: to check the pause-refusal scope, the rerun text, the tag format and the notice mailing, none of which were in context.
2. `RunStatusProducer` in `runlock.go`: to check what counts as a run status, which decides the D4 step 2 test; not in context.
