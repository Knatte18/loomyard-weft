---
dispositions:
  - id: F1
    disposition: fixed
  - id: F2
    disposition: fixed
  - id: F3
    disposition: fixed
  - id: F4
    disposition: fixed-differently
    reason: The proposal lists such a function apart as new since the cross-package maps. Every new function in a package that another package links meets that condition, since the diff itself changes the package's hashes. A separate list would therefore hide every untested new function in a linked package from the reach turn and from D5. D4 keeps the function in the unreached list with a caveat naming the packages whose older maps may reach it, so it is never listed as plainly unreached.
  - id: F5
    disposition: fixed
  - id: F7
    disposition: fixed
  - id: F8
    disposition: fixed
  - id: F9
    disposition: fixed
  - id: F10
    disposition: fixed
  - id: F11
    disposition: fixed
  - id: F12
    disposition: fixed
  - id: F13
    disposition: fixed
  - id: F14
    disposition: fixed
  - id: F15
    disposition: fixed
---

# Round 1 fixer report — discussion

Every finding's premise held against the code: `internal/loomshed/verifygate.go`, `internal/weaverengine/strand.go`, `internal/gatecli/failurereport.go`, `internal/gatecli/scope.go`, `internal/testcover/blockkey.go` and `pattern/PATTERN-test-speed.md`.
F6 is `decision`-class and lies with the parent.

## Changes

All changes are in `_lym/discussion/decision-record.md` unless named otherwise.

- F1: D3 gains the reach turn, the first path the fix key offered.
  After a passing verify, the loomshed verify gate (Darn-Write, Code-Burler-Fix) and the weaver's batch gate read a `reach.json` that matches HEAD.
  When it lists an unreached function that no earlier turn listed, the gate returns one failing result carrying the note.
  The turn spends one attempt, and a gate one failure short of its budget passes without it.
  Scope, the D3 rationale and its bound, and a D3 criterion were updated to match.
- F2: D7's line-cap bullet is replaced: a triage line never causes a cut, and triage lines are dropped before any test output is cut.
  The parse criterion now covers a report at the cap.
- F3: D3 refreshes only when neither the package nor a linked in-module package is dirty, citing `PATTERN-test-speed`, and the note names a dirty linked package.
  The criterion and the batch Open risk were updated to match.
- F4: fixed differently (see frontmatter).
  D4 gives an unreached new function, whose package other packages' older maps link, a caveat naming those packages; the D4 criterion matches.
- F5: D7 gains a no-change-set bullet and a criterion: the line says no change set exists and why, with no D8 command.
- F7: D9 counts a hub map whose hashes match the worktree as fresh in the prime, and D11 and the D9 criterion say the same.
- F8: D1 stamps the set of map files; the criterion covers an added map file.
- F9: D2 matches by package and name across files and matches `init` functions in order; the criterion covers a verbatim move.
- F10: D5 never matches a red-tier, out-of-hub or other-scoper report; the criterion matches.
- F11: D6 also refuses `--since` or `--branch` without `--test`; the criterion matches.
- F12: D2 compares two given sets of contents, base and head; D8 hands it a commit's first parent and the commit through fabric.
- F13: the batch Open risk is narrowed to a hand-run `lym gate test --gate batch`; the D3 rationale names the weaver's batch gate.
- F14: the D8 command sits on its own four-space line under the D7 line, and D8's last bullet says so.
- F15: the repeated auto-mode assumption is deleted.
- `_lym/discussion/support-log.md`: a Round 1 entry under Review rounds.

## Self-check

The four lenses all returned a report.

- fix-claims, MEDIUM: a gate cannot know it is on its last attempt. This is merged with the fix-way-forward MEDIUM below. Fixed as proposed: each gate is told its entry's attempt budget and counts its own failing results since its last pass, which never undercounts the engine's consecutive count (`internal/shuttleengine/wait.go:647-652`).
- fix-way-forward, MEDIUM: duplicate of the fix-claims finding above; fixed there.
- fix-way-forward, LOW: tests added but left uncommitted hit the dirty-tree check. Fixed as proposed: the findings now say to commit them.
- fix-way-forward, LOW: the darn form has no skip path, so "the next arrival passes as the verify did" was false. Fixed as proposed: the next arrival runs the verify and passes or fails as it does.
- fix-consistency, LOW: D2 named only the reach step as the caller handing in the merge base. Fixed as proposed: it now also names D7 triage and D9.
- fix-proposal, MEDIUM: the separate "new since those maps" list hid every new function in a linked package. Fixed as proposed: it is now a caveat on the unreached entry (F4 above).
- fix-proposal, MEDIUM: matching by package and name is ambiguous for build-constrained variants. Fixed as proposed: a name defined in more than one file is matched by file, and the D2 criterion covers it.
- fix-proposal, MEDIUM: the reach turn re-fires on every new tree for the same functions. Fixed as proposed: the once-only record is kept per unreached function and its text hash, and the bound and criterion were updated.
- fix-proposal, LOW: a package with no map has no source for its linked set. Fixed as proposed: its linked set comes from the Go toolchain's dependency list of its test binary.
- fix-proposal, LOW: the file-set stamp makes every reader rebuild after an interrupted map run. Fixed as proposed: D1 states that bound.

## Disputed

None.

## Deferred

None.

## Rulings

None reached this round.
