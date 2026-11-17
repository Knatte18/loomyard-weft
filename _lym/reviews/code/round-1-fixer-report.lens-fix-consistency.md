```yaml
lens: fix-consistency
outcome: reported
```

Task: check the change set through the fix-consistency lens, looking for twins and cross-references left stale.

**Findings**

1. **LOW**: the dobbyengine package doc still says the first sight "posts one notice … and records the time".
   - Changed text: the orphan bullet at `internal/dobbyengine/doc.go:292-294`, against the new `dueOrphans` in `internal/dobbyengine/orphan.go`.
   - `dueOrphans` now records the first sight only after the notice posts, and a failed post leaves the orphan unrecorded and retried at the next probe tick.
   - The doc keeps no hint of that ordering or of the retry, though the `dueOrphans` comment states both.
   - It also lists only the failed read and failed write as logged failures, while a failed notice now goes through `orphanFailures` too.

2. **LOW**: three descriptions of kept-cancel marking cover only the no-intent case, though `MarkKeptCancels` now also re-commits a finished keep intent this machine wrote.
   - Changed text: `internal/dobbyengine/loop.go:118-122` (`Deps.MarkKeptCancels`), `internal/dobbyengine/doc.go:309-311`, and `internal/beamcli/doc.go:444-445`.
   - Each describes only a run "cancelled with keep before cancel intents existed" that gets its intent.
   - The log line in `markKeptCancels` (`internal/dobbyengine/leftover.go:217`) says "a cancel that kept the pair before the upgrade now records its finished keep cancel intent". It now also prints for a retried, previously uncommitted intent.
   - The returned slugs now mean "committed", not "marked", which the `MarkKeptCancels` comment updated but these siblings did not.

3. **LOW**: `HandOver` still calls `RemovePair` directly with no session end, which is the defect F3 fixed in `RemoveHandedOverPair`.
   - Evidence: `internal/pairteardown/handover.go:147` is `td.RemovePair(Request{Slug: slug, Local: true, Force: true, Verdict: fabricengine.EntryOwnedElsewhere})`.
   - The package doc (`doc.go:309`) says `HandOver` ends only "the processes this machine runs for the run".
   - That path covers a caller outside the pair, whose reed session can still be live.
   - The review did not name this call site, but the fix made the two removals of the same pair differ in whether they end the session.
   - Using `td.Run` there would give the same ordering.

**Checked and consistent**
- `RemoveHandedOverPair`'s new `Result` return still fits its caller, which discards it (`internal/beamcli/dobbydeps.go:443`).
- `internal/pairteardown/doc.go` "Leftover pairs", `internal/beamcli/doc.go:439-440`, and the `tearDownLeftover` comment all agree with the new behaviour.
- The doc.go kept-pairs bullet reorder (F16) agrees with its neighbours.

**Further tool calls**
- Bash grep/sed over the repo for `MarkKeptCancels`, "first sight", `RemoveHandedOverPair`, `RemoveOrphanPair`, plus `handover.go` lines 135-150: my context lacked every sibling and twin site outside the files the fixer edited.
- Read `internal/dobbyengine/doc.go` 286-313: to see the orphan and kept-cancel bullets as they stand.
- Read `internal/dobbyengine/leftover.go` 190-230: to see the `markKeptCancels` caller and its log line.
- Read `internal/beamcli/doc.go` 434-449: to see the beamcli package doc's wording of the same items.
