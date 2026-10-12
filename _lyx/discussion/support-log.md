# Support log — board-field-drop

## Interview

Autonomous run: no operator answered; every pick below is an auto-pick recorded in the Question ledger.

1. Read the board entry: two bundled items, the board field drop and four PR #574 follow-ups.
   Read the other board entries; none plans board-schema or lyxtemp work (`board-viewer` and `doctor` read the board but do not change its write path).
2. Explored `internal/boardengine`: `storeFile`/`loadedStoreFile` decode entries into `Task`/`storedEntry`, so unknown keys drop on save; `ApplyPatch`/`NewTask` round-trip through JSON; the run lock compares named fields.
   Picked keeping unknown keys over a schema refusal (Q1, Q2).
3. Explored PR #574's "To check by hand" list and the `test-tmp-cleanup` records in loomyard-weft.
   - The cleanup refusal: `internal/cleanupcli/cleanup_integration_test.go` runs `cleanup --apply` from a task pair and asserts an error naming the prime and that every leftover remains; it landed in `bc9ee7a02`, the PR's own commit.
   - The batten-cleanup skip: `internal/battencli/cleanup_test.go` row "deleted run with a committed seed but no committed status is skipped and keeps its aged scratch" wants the reason "tracked run directory with no committed status"; also from `bc9ee7a02`.
   - The testtiming item: Plan-Review round-1 finding F3 (`testtiming-resign-arrow-missing`) of `test-tmp-cleanup`, a plan card whose Edit bullets for `measurePackageResources` and `leftoverProcesses` lacked re-sign arrows.
     The landed signatures match the fix F3 proposed, and their doc comments and the `cmd/testtiming` package doc describe the marker, not a temp directory.
   All three need no change (Q4).
4. Explored `internal/lyxtemp/sweep.go`: `merge-review` (`314816cde`) added a three-attempt removal retry against a peer's lock probe recreating the lock file mid-removal.
   First pick: the retry covers two sweepers, add a deterministic test only.
5. Spawned the six lens forks and wrote the record.
6. Lens review; all six forks returned.
   - scenarios, MEDIUM, D7 assumed two sweepers while every `newRun` sweeps: fixed; D7 now retries for a one-second budget, a code change (Q5 revised).
   - scenarios, MEDIUM, case-insensitive key matching could keep a duplicate of a known field: fixed in D1 with an acceptance line.
   - scenarios, LOW, exhausted-retry and vanished-path rows: fixed in Acceptance criteria.
   - scenarios, LOW, empty board with a top-level unknown key: fixed in Acceptance criteria.
   - scenarios, LOW, kept values can go stale: fixed in Open risks and D5's doc text.
   - scenarios, LOW, a newer binary retiring a field this binary knows: fixed in Open risks.
   - premises, MEDIUM, `list-full` also prints whole `Task`s: fixed in D4, Scope and Acceptance criteria.
   - premises, LOW, D1's write-path list omits the legacy fold: rejected; legacy files carry no kept keys, as the finding says.
   - premises, LOW, promote shares the kept-key map between copies: rejected; nothing mutates it (D4).
   - pattern-fit, MEDIUM, retry test bypassed the public surface (`PATTERN-test-economy`): fixed; the seam sits on `Root` and tests drive `Root.Sweep`.
   - pattern-fit, LOW, leaf import set unstated: fixed in D7's bound.
   - pattern-fit, LOW, `PATTERN-sole-parsers` cited for board.json: fixed, constraint dropped.
   - pattern-fit, LOW / goal-scope, LOW, `boardcli/doc.go` for the `get` output change: fixed in Scope and Acceptance criteria.
   - pattern-fit, LOW / leanness, LOW, `storedEntry` embedding is a requirement, not a risk: fixed, moved to Constraints.
   - goal-scope, MEDIUM, the reason for excluding other record files was wrong: fixed; Scope gives the title-and-Idea reason, and the hazard is in Open risks.
   - goal-scope, LOW, retiring a field later needs an explicit drop: fixed in D3.
   - goal-scope, LOW, pin "no race remains" with a repeated concurrent run: rejected; a repeated run is chance-based, and the time-budget retry plus deterministic rows pin the behavior.
   - board-fit, LOW, which rendered views show kept keys: fixed in D4.
   - board-fit, LOW, the Windows risk has no owner: fixed; Open risks says no entry tracks it.
   - board-fit, LOW, restoring the dropped dates: fixed, listed under Out.
   - leanness, MEDIUM, testtiming evidence and signatures in the record: fixed, moved to turn 3 here.
   - leanness, LOW, assumptions repeating decisions: fixed, cut to the new facts.
   - leanness, LOW, Open risks repeating D5: fixed, cut.
   - leanness, LOW, constraints with nothing to act on: fixed, cut.
   - leanness, LOW, race reasoning in D7: fixed by the D7 rewrite, which keeps only what directs the plan.
   - leanness, LOW, "existing subtest still passes" criterion: fixed, cut.
   - leanness, LOW, Windows both Out and Open risk: fixed, Open risks only.

## Rejected alternatives

- Schema-version refusal (a writer refuses a board whose schema is newer): halts a long run's status writes, which is worse than the loss, and requires a version bump per field.
- Status writes patch only `status` in place instead of rewriting the entry: fixes set-status alone; keeping unknown keys covers every write path.
- Both keeping keys and a refusal: the refusal adds the halt without adding protection.
- Extending unknown-key keeping to shed `status.json` and other record files: outside the entry's title and Idea.
- Exact-name matching of known keys: a case-variant key would be decoded into the field and also kept, and the kept copy would override it on the next load.
- Keeping the fixed three-attempt retry with only a test: many concurrent sweepers can recreate the lock file more often than three attempts absorb.
- A probe that never creates the lock file (a non-creating try-lock in `internal/lock`): removes the race at its root but changes a shared package.
- Renaming a dead directory out of the run prefix before removal: still races probes in flight at rename time, so it needs the retry anyway.
- Testing `removeRunDirectory` directly through a function parameter: tests an unexported helper instead of the `Sweep` surface.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 (auto-pick, resolved): keep unknown keys, or refuse a newer schema? Keep keys (D1, D6); a refusal halts runs.
- Q2 (auto-pick, resolved): keep top-level unknown keys too? Yes (D2), same mechanism, same failure.
- Q3 (auto-pick, resolved): cover other whole-file record writers? No; the entry names board writers. The hazard is in Open risks.
- Q4 (auto-pick, resolved): do the three non-race PR items need work? No; each is already met (turn 3).
- Q5 (auto-pick, resolved, revised after lens review): fix the sweep code or the test? Both: a time-budget retry, pinned by deterministic rows through `Sweep` (D7).
- Q6 (auto-pick, resolved): should `get` and `list-full` show kept keys? Yes, since `Task`'s encoding carries them; list, README and rendered views stay on known fields.
- Q7 (auto-pick, resolved): the removal budget? One second total, retrying while the directory exists.
