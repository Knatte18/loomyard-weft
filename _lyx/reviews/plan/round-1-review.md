---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: consistency
    location: _lyx/plan/02-board-cli-kept-keys.md:14
    summary: Card 2 adds kept-key rows for get, list-full, upsert, promote and merge to stepLookupContract, whose doc comment limits the step to the slug-or-id lookup contract on get, set-status, remove and set-deps, and the card leaves that comment unchanged, so it goes stale
    fix: In card 2's Intent, after the sentence that introduces the new stepLookupContract rows, add that stepLookupContract's doc comment (internal/boardcli/cli_test.go, above func stepLookupContract) gains a line saying the step also pins that get, list-full and the task envelope of upsert, promote and merge print an entry's kept keys with exact digits. stepLookupContract is already an Edit target, so no target changes.
  - id: F2
    severity: NIT
    class: consistency
    location: _lyx/plan/01-board-unknown-keys.md:90
    summary: The new TestRunLock row set-status on an entry with unknown keys repeats the act of the existing accepted row set-status to another value, SetStatus locked paused X, and differs only by the new unknown field
    fix: In card 1's Intent, replace the sentence that adds the new row with one that sets unknown to "zz_scalar":9007199254740993 on the existing accepted row set-status to another value, so that row pins both the status write and the kept key passing the run lock, and add no new row.
  - id: F3
    severity: NIT
    class: consistency
    location: _lyx/plan/01-board-unknown-keys.md:81
    summary: The set-status row's saveOrder anchors on the title member, so it only pins that kept keys follow title, not D1's rule that they come after every known field
    fix: In card 1's Intent, change the saveOrder list of the set-status row to start with the status member the row writes, the last known key entry a carries after that write, followed by "zz_nested" and "zz_scalar", so the order pins kept keys after the last known field in sorted key order.
---

### [LOW] stepLookupContract's doc comment goes stale under card 2's rows

**Location:** `_lyx/plan/02-board-cli-kept-keys.md:14`

**Issue:** `internal/boardcli/cli_test.go`'s `stepLookupContract` comment reads "covers the slug-or-id lookup contract on get, set-status, remove and set-deps".
Card 2 adds rows for `get`, `list-full`, `upsert`, `promote` and `merge`, all pinning a different behavior: whether a printed `Task` carries its kept keys.
The card's Intent changes the table and its runner but never mentions the step's doc comment, so after card 2 the comment no longer describes what the step covers.
`stepContract`, whose comment names the JSON envelope shape per happy-path verb, would also fit these rows, but moving them changes targets.
Updating the comment of the step the card already edits is the smaller fix.

### [NIT] The TestRunLock kept-key row duplicates an existing row's act

**Location:** `_lyx/plan/01-board-unknown-keys.md:90`

**Issue:** `internal/boardengine/runlock_test.go`'s `TestRunLock` already has the accepted row `set-status to another value`, which acts with `b.SetStatus("locked", strPtr("paused · X"))`.
Card 1's new row "set-status on an entry with unknown keys" runs the same act on the same seed and differs only in setting `unknown`.
Setting `unknown` on the existing row meets the acceptance criterion "a status write on a run-held entry that carries unknown keys passes the run lock" without a second row for the same write (`PATTERN-test-economy`).

### [NIT] saveOrder's anchor does not pin "after the known fields"

**Location:** `_lyx/plan/01-board-unknown-keys.md:81`

**Issue:** D1 says encoding writes kept keys "after the known fields, in sorted order".
The set-status row's `saveOrder` is `"title":"A"`, `"zz_nested"`, `"zz_scalar"`, so it pins only that the kept keys follow `title` and sort among themselves.
An encoding that put them between `title` and `body` would pass.
After `SetStatus("a", …)`, entry `a`'s last known member is `status`, since the seeded body carries no dates or run settings.
Anchoring on that member pins the D1 rule the lens review says this row covers.

## Rejected

- Focus 1, granularity of card 1's Create list: not a defect.
  `jsonKeys`, `splitUnknown` and `appendUnknown` are private helpers, and the five JSON methods implement one behavior, unknown-key keeping.
  All of them sit on card 1 beside the types they serve, and no symbol has a card of its own.
  Granularity governs cards, not Create bullets, and the format requires one `plan:` handle per new member.
- Focus 2, re-sign arrows and member glyphs on card 1: not a defect.
  `Store.Load`, `Store.Save`, `ApplyPatch`, `cloneTasks`, `checkRunLocks`, `scopeChanged` and `TaskWithLayer` keep their parameters, results and receivers, so none takes an arrow.
  The Intent changes their bodies, comments or fields only.
  Every symbol the Intent names is a member glyph, and `NewTask` and `validateUpsertFields` are named as unchanged.
  Card 3's re-signed `removeRunDirectory` carries its arrow, and its only caller, `Root.Sweep`, is on the same card.
- Focus 3, test economy of `keptValue`, `addEntryKeys` and the rows: no redundancy beyond F2.
  No existing boardengine test reads a saved member's exact value: `TestLoad` matches whitespace-free substrings only.
  `addEntryKeys` is needed because `Board.UpsertTask` refuses an unknown key, and `TestRunLock` seeds through it.
  No boardcli test helper writes a raw board.json, so `seedBoardFile` duplicates nothing.
  The card 3 subtests cover retry behavior that `TestSweep`'s rows and its concurrent subtest do not reach.
- Focus 4, fidelity: every decision from D1 to D7 and every constraint is carried.
  D1 to D3 and D5 to D6 are on card 1, D4's printing is on card 2, and D7 is on card 3.
  The `storedEntry` constraint is carried by `storedEntry.UnmarshalJSON` and the extended tier-and-type row.
  The older-binary constraint is met by plain members and the byte-identical encoding of an entry without unknown keys.
  The D7 seam is `retryNow` and `sleep` beside the grace `now`.
  No card adds unlicensed scope.
  The `TaskWithLayer` comment and the run-lock comment edits are forced by the change.
- Focus 5, attack surface and refusal reachability: not a defect.
  Card 3's Intent states the retry's bound, and card 1's Intent carries D4's bound into `doc.go`.
  The unknown-key upsert refusal is unchanged, and `TestUpsertFieldAllowlist`'s `upsert_typo_key_errors` reaches it through `Store.UpsertTask`.
  Go's `os.RemoveAll` reports the final not-empty `unlinkat` with the full joined path (`os/removeall_at.go`), and `syscall.ENOTEMPTY` satisfies `fs.ErrExist` (`syscall/syscall_unix.go`).
  So `isRemovalSymptom` matches the real symptom on Linux, and its inside-path row reaches the non-symptom branch.
- Focus 6, card order and verify: not a defect.
  Card 2 depends only on card 1's `Task.MarshalJSON`, and card 3 is independent.
  `## verify:` covers `boardengine/...` (including `boardtest`), `boardcli/...` and `lyxtemp/...` in the template's order.
  The integration-tagged `cli_test.go` runs under the integration tier.
- Focus 7, ImpactSummary: not a defect.
  Card 1's states a blast-radius conclusion: `ApplyPatch` callers are all in `store.go`, and no struct outside boardengine embeds `Task`, which I confirmed by grep.
  Card 3's names the single caller and the unchanged public surface.
