```yaml
lens: fix-affected-set
outcome: reported
```

Task: check that the fix covers every member of each set-premised finding (F1, F3, F4, F8, F9; F2 is the parent's).

Findings left unchecked: none.

1. **LOW**: `internal/loomcli/prreview_mail_integration_test.go`, the rows' checks (F3).
   The fix restores the push and whole-record checks, but not the rest of the shared loop the three moved rows ran under.
   - **Evidence:** at `c391c9f35`, the loop in `internal/landingshed/prreview_wait_test.go` also asserts that:
     - every sleep is at least 1s and at most 1min;
     - the wait mark is set to `PR #7` at the start and is `""` on the way out;
     - `w.deps.Lock.Store.List()` holds nothing, "since the wait takes no parent lock".
   - The new test asserts none of these. Its `pr.Sleep` takes `d` without checking it, and it records no mark and reads no lock store.
   - No commit message carries a `Loosened-Check:` line for them.
   - Rows still in the old table keep these checks on other paths, including gate failures that mail nothing, but not on the mailing path.

2. **LOW**: `_lym/landing/summary.md`, the new Design decisions bullet, and `internal/testkit/envkit/envkit.go` (F4).
   The bullet accounts for `DriverStrand` and `ReviveStrands` only, but `FullEnv` has other fixed answers of the same kind.
   - **Evidence:** `InnerRun.ArbiterInFlight` still returns `battenshed.ChildArbiter{}, false, nil`. In production it reads the child pair's reed directory (`readReedDirectory`, `internal/battencli/wire.go`, around lines 1395-1409).
   - `InnerRun.DriverAlive` returns `false, nil` beside it (production: `childDriverAlive`; I did not read what that reads).
   - Neither is filled from reed or named in the bullet. The review named only the two, but the finding's premise is envkit's hand-written reed strand answers.

F1, F8 and F9 cover their sets:
- **F1:** `Mode`, `Origin` and `TaskHead` all come from the wired deps.
- **F8:** `loom-landing.md` lines 103 and 200 are the rows for the removed `envkit` and `prreview_wait_test.go` fakes. The `noopSamplerSessions` at `webster-shed.md:95` belongs to `shedrecipe/fixture_test.go`, which this branch does not touch.
- **F9:** the summary's new sentences cover the production files changed and the packages building on `envkit` and `tmuxrec`.

Tool calls (5):
- Grep of the docs for the removed stand-in and row names: my context had only `loom-landing.md` slices.
- Shared table loop at `c391c9f35`: my context had only the diff hunks of the removed rows.
- Packages importing `envkit` or `tmuxrec`, plus the branch's changed files: needed for F9's set after the new commits.
- A range of the wait test at the merge base: looking for the loop's tail, which it did not reach.
- Tail of the loop at `c391c9f35`: the first read stopped partway through it.
