```yaml
lens: fix-consistency
outcome: reported
```

Task: fix-consistency lens over the round-1 fix commits (agreement with siblings and twins).

Drift's `CommitWithTrailers` is a different function, so the force-empty change doesn't reach it.

**Findings**

1. **MEDIUM** — `pattern/PATTERN-run-fsm.md`, the new paragraph "`webster reset` marks a transition too…".
   - The paragraph says the commit is sent "after the reset moved the code branch on origin". That puts the code push before the records push.
   - The pattern's own Rule line (line 20) and the sentence just above say "records side first, then the code branch".
   - So the pattern now contradicts itself, and nothing says the reset is a deliberate exception.
   - It also leaves the old `Held by` wording lead-in unchanged, which still describes `statuscommit` as the only writer.
   - Fix: say reset is the one writer whose code push goes first, and why that is safe. Or drop the "after…origin" ordering claim.

2. **LOW** — `internal/fabricengine/doc.go:110-112`, the `CommitWithTrailers` paragraph.
   - It was not updated for the new behaviour. The `commit.go` and `recordsgit.go` docs say a weft commit always lands, empty if need be, on a born warp. The package doc says nothing of it.
   - Fix: add a line that a trailer commit lands even when the files are unchanged.

3. **LOW** — `contracts/specs/loom-plan-checks-spec.md`, the new `overview-frame-edited` entry: "A change to its work is a follow-up card…".
   - The text was copied from `done-card-edited`. An overview has no "work", and the gate's Detail text says "carry the change in a follow-up card".
   - Fix: reword to "Carry the change in a follow-up card after the last begun batch, with its Card Index line."

4. **LOW** — `_lym/landing/summary.md`, Design decisions.
   - It still says both notices go only "while batten's hold ends without the arbiter's hand-on". The new `driveNotices` and `doc.go` text says both go at every watched judged blocked stop.
   - The fixer left this file alone because it is under `_lym`. The twin is now stale and must be recorded as left undone.
   - Its Tests section also lacks the new rows: the `judgedKey` row, the Card-Index-only row, the Start row, and the unborn-warp and no-op trailer tests.

No further tool calls beyond the two below.

- `grep` for `CommitWithTrailers`, `no-op commit` and `records side first`: to find sibling docs and callers of the changed function that my context lacked.
- `sed` on `internal/fabricengine/doc.go` and `internal/driftcli/wiring.go`: to check the package doc wording and whether the drift caller reaches the changed function.
