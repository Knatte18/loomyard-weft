---
verdict: APPROVED
rationale: "All seven round-1 findings are fixed on HEAD 1225c84b4 (F1 dirty-sibling remedy names the way out and the overview drops its false distinction; F2 archive remedy points at the wrapped cause; F3 an unparseable done-seen marker restarts the grace, with a test; F4 overview history-cap sentence; F5 teardown step asserted done; F6 a token-level check shows 42 files changed only comment line breaks; F7 recipe header), the plan's full verify block passes, and a fresh pass finds no new defect"
---

# Webster-Review round 1: Bouncer verdict

The round-1 report raised one MEDIUM, five LOW and one NIT finding against the diff over merge base `083e992cf` (`product.parent` is `main`).
The fixer report claims all seven fixed, one commit each.
I checked each fix against its commit and against the code on HEAD `1225c84b4`.

- **F1 (MEDIUM), commit-records remedy for dirt it cannot clear.**
  `049c7a931`: `teardownRefusal` keeps `lyx loom commit-records` as the first step.
  It then names the exit when the resumed teardown refuses again: the leftover is not a run record, so commit or remove it by hand, never `--force`.
  The doc comment states why: fabric's refusal covers the whole sibling.
  docs/overview.md no longer claims that "other uncommitted content keeps fabric's refusal".
  It now places that refusal on the task worktree itself, which is what the code does.
  `wire_test.go` asserts the new wording.
  Resolved.
- **F2 (LOW), archive remedy assumes an unreachable remote.**
  `23da48ba1`: the arm now reads "fix the failure named here (most often an unreachable remote)" over the `%w`-wrapped cause, and the test asserts it.
  Resolved.
- **F3 (LOW), an unparseable done-seen marker wedges the run.**
  `e226003d7`: `callDone` keeps a non-absent read failure as a hard error.
  A marker that reads but does not parse now logs a warning naming the path and parse error, is rewritten with `now`, and the wait continues.
  `TestInnerRun_UnparseableDoneSeenMarkerRestartsTheGrace` starts from an empty marker, asserts it is rewritten and that `Done` follows a full grace later.
  Resolved.
- **F4 (LOW), stale history-cap sentence.**
  `ecf54ecca`: the overview now says the self-bounce appends per poll while the child runs, awaits approval or waits for its driver.
  Resolved.
- **F5 (LOW), records end-to-end test.**
  `1a053cdf1`: the teardown step's producer, outcome and state are asserted as `Worktree-Teardown`, `done` and `done`.
  Resolved.
- **F6 (LOW), fixed-column comment wraps.**
  `1225c84b4`: I compared each of the 42 Go files before and after with `go/scanner`.
  Every non-comment token is identical, and the comment words match in order, so the commit changes only comment line breaks.
  A scan of every added `//` line pair in the branch diff finds no break after a line that ends without punctuation.
  Three breaks remain inside a sentence at a comma: `loomcli/commitrecords.go` before "and the Fabric Git Invariant…", `loomcli/wiring.go` before "where a friction-only entry…", and `loomcli/wiring_commitstatus_test.go` before "including only a timestamped archive sibling…".
  The first is an independent-clause break the convention asks for.
  The other two are deliberate phrase-level breaks, not fixed-column wraps, and I do not raise them.
  Resolved.
- **F7 (NIT), change-narrating recipe header.**
  `0b6b7ba16` drops the sentence.
  CONSTRAINTS.md's Comment Line-Break Convention governs Go comments only, so the YAML header's own wrapping is not a finding.
  Resolved.

The report has no `## Focus departures` section, so there is nothing to ratify or reject.
I agree with its settled focus items, which found no defect: the unattributed commit `a25c57c0d`, card 10's resume through `lyx loom start --no-attach`, card 1's budget arm, and the card 12/13 ordering.

## Fresh pass

- **Verify.**
  On HEAD, `go build ./...` passes, both `go test` lines exit 0 with no `FAIL`, and `go vet -tags smoke` passes.
- **Per-card mechanical checks.**
  The cards carry only `Create`, `Edit` and `Uses` groups, with no `Rename` or `Delete` group, so no AST-script or `assert-no-callers` check is due.
  The report's no-prior-equivalent check covers the `Create` targets.
  The two in-card removals, `reviewsHoldFile` and `awaitingChildRemedy`, have no reference in code, tests, docs or recipes outside `_lyx/`.
- **Fix-induced regressions.**
  The fix commits touch only the lines their findings name, apart from F6's comment-only rewrap, and none changes behavior beyond its finding.
