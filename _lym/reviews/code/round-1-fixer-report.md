---
dispositions:
  - id: F1
    disposition: fixed
  - id: F2
    disposition: fixed
  - id: F3
    disposition: fixed-differently
    reason: "The proposed Finalize comment says the gate runs with no lock and the landing needs the gated tip, with no condition; that is false after the lost-race bound is spent and over a parent already holding the work, so the comment adds that sentence. The three header rewordings are as proposed."
  - id: F4
    disposition: fixed
  - id: F5
    disposition: fixed
  - id: F6
    disposition: fixed
  - id: F7
    disposition: fixed-differently
    reason: "Reading the whole notice once before the loop takes the parent values before the parent pair's sync, so a key the parent already holds upstream would be named. The task's changes and base are read before the call's first merge of the parent, and the parent values are read again under the lock (internal/landingshed/finalize.go, readConfigBeforeMerge and configChangeNotice). The proposed deps.go sentence claims a fork-point base, which an earlier Parent-Align or Finalize merge makes false, so the doc says so."
  - id: F8
    disposition: fixed
  - id: F9
    disposition: fixed
  - id: F10
    disposition: fixed
  - id: F11
    disposition: fixed
---

## What changed

- F1: `gateUnlocked` runs the owner checks before it merges anything.
  Rows for a run owned elsewhere and an ended run assert the halt and that nothing was merged.
- F2: `parentMergeTip` takes `front`; Finalize passes its loop's value and Parent-Align passes false.
- F3: the Finalize comments and headers in the darn, sampler, tapestry and hardener recipes describe the new lock order.
  `internal/loomrecipe/testdata/tapestry-rows-before-motif.yaml` keeps the old comments, which its header says are the recipe's own as they stood.
- F4: a row for a local parent behind its upstream, asserting the fast-forward runs before the landing merge.
- F5: `TestFinalize_FixerInTheLockFreePass`.
- F6: the compare with the gated tip is skipped when the parent already holds the work.
- F7: see the disposition; a row in `TestFinalize_ConfigChangeNotice` pins which read supplies which value.
- F8: `TestFinalize_LockFreePassStops` and `TestFinalize_QueuesAtTheFrontAfterALostRace`.
  The compare's head read is the first `HeadSHA` call of that pass, so its row fails every call.
- F9: the fake gate skips a tree it already passed.
- F10: the `Finalize` type doc and the `freeLock` doc.
- F11: `internal/loomcli/prepush.go` is unchanged, since F1 makes its sentence true again.
- `docs/overview.md`, `internal/landingshed/doc.go` and `deps.go` follow the changes.

A comment check then refused wrapped and over-long comment lines.
Those in `deps.go`, `finalize.go` and `finalize_unlocked_test.go` are broken at sentence boundaries or reworded, with no code change.

Commits: f293071a1, bfd261ba2, efb221c4f and 779dfd162.
f293071a1 carries a `Loosened-Check` trailer for F9.

Gate: `lym gate test ./internal/landingshed ./internal/loomrecipe` passed with no failures on 779dfd162's tree.
`lym gate test --tier 1 ./internal/landingshed` passed on efb221c4f's tree; only comments changed since.

## Self-check

- fix-proposal, LOW (the gate-order rows no longer pinned a gate call under the lock): fixed as proposed.
- fix-consistency, MEDIUM (`ConfigChanges` doc's base is false for a re-entered Finalize): fixed as proposed.
- fix-consistency, LOW (`Call` doc and overview leave out the lock-free pass after a fixer): fixed as proposed.
- fix-consistency, LOW (`doc.go` sentence order): fixed as proposed.
- fix-consistency, LOW (comment lines over the limit): fixed as proposed.
- fix-affected-set, LOW (same twins of the `freeLock` rewording): fixed as proposed.
- fix-affected-set, LOW (long recipe comment line): fixed as proposed.
- fix-code-correctness, LOW (notice built before the sync reports stale parent values): fixed as proposed.
- fix-code-correctness, LOW (`ConfigChanges` doc's base): fixed as proposed.

The self-check fixes are in efb221c4f; no fork reviewed that commit.

## Disputed

## Deferred

## Rulings
