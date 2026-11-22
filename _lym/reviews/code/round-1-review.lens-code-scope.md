```yaml
lens: code-scope
outcome: reported
```

Task: the code-scope lens. I traced the diff's changes to the decision record's goal points and checked that each goal point is built.

**Decisions** (decision record alone, no `Added after Discussion` entries):
- **G1:** a run pushes nothing to origin before it starts, so the start rows from Seed-Child to Run-Shed act without a records push.
- **G2:** only those start rows stop pushing, and every later transition keeps its push, Land and Worktree-Teardown included.
- **G3:** a takeover of a run whose start rows were not pushed redoes those rows and does not break. The entry hands this check to Discussion.

**Per change:**
- **C1** `internal/battencli/commitstatus.go:224-247,264-266`: the push-free `startCore`, the shared `writeMarker`, the `isStartRow` predicate and the dispatch condition. Traces to G1 (start rows commit without a push) and G2 (Worktree-Create, the rows after Run-Shed and any non-running or marker-less call still go through `core`).
- **C2** `commitstatus.go:7-8,209-215` (file header and `newCommitStatusSeam` doc): forced by C1, which made the old doc stale. The takeover sentences there are the diff's account of G3.
- **C3** `internal/battencli/commitstatus_test.go:32-33,51-64,100-101` (`wantPushes` and the four new rows): forced by C1, as its test.
- **C4** `PATTERN.md:33`, `pattern/PATTERN-run-fsm.md:4,38-47`, `pattern/PATTERN-pair-remote.md:20`: forced by C1. The rule "a transition commits and pushes before the run acts" became stale, and the run-fsm section states G3's takeover behaviour.

**Per decision:**
- **G1:** built by C1, with C2, C3 and C4 forced. One part is narrowed; see finding 1.
- **G2:** built by C1. The `StartRowsCommitWithoutPush` row (test file lines 58-61) pins that Land and Worktree-Teardown keep their push.
- **G3:** no code change, which matches the entry: it asks for a check and that the takeover not break. The diff states the outcome in C2 and C4 and relies on the existing `pairteardown.PushRun` → `HandOver` path (`internal/pairteardown/handover.go:85-87`). I filed no scope finding, since the entry asks for no code. Whether the existing path does hold is for the correctness and tests lenses.

**Findings:**

1. **LOW · scope** at `internal/battencli/commitstatus.go:264` (the dispatch condition `marker.State == running … && st == running`).
   The diff narrows G1's exemption to an *uninterrupted* start, and the entry does not license that.
   G1 says the start rows from Seed-Child to Run-Shed act without a records push. G2 says only those rows stop pushing. Neither carves out a start row resumed after a pause or a halt.
   Under the diff, the first resumed start row still pushes through `core` because its marker holds `paused` or `failed`. A Run-Shed resumed after a pause pushes too.
   The `ResumedStartRowPushes` row pins this (test file line 63). The PATTERN text bakes in "uninterrupted" (`pattern/PATTERN-run-fsm.md:4,40,43`; `PATTERN.md:33`).
   The halt transition's own push is defensible, since a halt is not a start row acting. The push on resume is the writer's own design call, from the summary's "Design decisions" (`_lym/landing/summary.md:34-36`), and no `Added after Discussion` entry records it.
   Fix: either make a resumed start row commit without a push, or have the parent record the narrowing with `lym loom decision add`. Recording it is outside the fixer's write surface, so defer it to the parent if the narrowing is kept.

**Not a finding:**
- Seed-Child's own seed push (`internal/battenshed/seamchild.go:123`, wired as a detached spawn at `internal/battencli/wire.go:1666`) still pushes during a start row, against G1's literal "pushes nothing to origin". It already existed, it is detached so the row never waits on it, and the entry's measurement and scope name the transition push.
- Worktree-Create's push stays. It sits outside the Seed-Child to Run-Shed range G1 names.

Tool calls: none.
