# Round 2 fixer-report — discussion

Every finding's premise was checked against the code (`internal/driftengine/records.go`, `capture.go`, `paths.go`, `verify.go`, `internal/hubreconcile/walk.go`, `internal/gateslot/tiers.go`, `contracts/recipes/darn-recipe.yaml`, `internal/runproc/end.go`) and holds.
All edits are in `_lym/discussion/decision-record.md`, with a Round 2 entry and a Q5 revision in `_lym/discussion/support-log.md`.

## Changes

- F1 — fixed differently. D7 records a baseline digest when Drift-Await counts the capture. Every check recomputes it first, in either mode, at Drift-Verify, Drift-Recheck, the drift entry and `lym drift check`. A mismatch or missing directory names `lym drift defer`. A capture counted before landing gets its digest on Drift-Expect's seal commit. D8's bound and a D7 criterion were added.
  Not made as proposed:
  - The digest is not kept in `scope.json`. It sits in a `Drift-Baseline` trailer on Drift-Await's weft commit, read as the oldest such commit, because the writer can edit `scope.json` as easily as the baseline. The support log rejected seal fields in `scope.json` for the same reason (self-check, fix-proposal 2).
  - At the drift entry and `lym drift check`, the stop is reported without failing, not as a failure. No writer or Merriam fixer can run `defer`, as D6 already rules for seal failures (self-check, fix-way-forward 2).
- F2 — fixed as proposed. D6 uses an `expect_session` mark in `scope.json`, and D9 a `judge_session` mark. D14 says where `runs.json`'s mark moved.
- F3 — fixed as proposed. In D3, Drift-Baseline records `diff` and `key_patterns`, and later rows, the entry and the check read them. A run seeded before landing records config's values at its next drift row; the entry and check read config until then, since they write no tracked record. The D3 criterion was extended.
- F4 — fixed differently. D2 gives `command` mode a per-case expectation: the case list only, one `why` per case, no `keys` or `lines`, plus `added` and `removed`. The parser checks no case name against the baseline at all; amendments follow the same rules (D11). D5 and D16 point to it, and the D2 criterion was extended.
  Not made: "case names checked only when the capture writes one file per case". Code cannot tell when that holds, since `command` mode exists for captures that break D1's contract.
- F5 — fixed as proposed. D2 stores a `does not match` or `unexpected` ruling keyed by (warp head, case), and a resumed row at that head with no amendment is a Stuck without a judge. D2's bound, D10 and the D2 criterion were updated.
- F6 — fixed as proposed. D9's snapshot covers both checkouts' HEAD, tracked tree and untracked paths. The baseline is guarded by D7's digest. The D9 criterion was updated.
- F7 — fixed differently. In D9, Drift-Verify removes `judgement.yaml` before each spawn and hands the judge a spawn id. The file carries the id, and the entry refuses any other. An interrupted judge is probed, resumed with the same id, or respawned. D16 and the D9 criterion were updated.
  Not made: the "cell ids prefixed by spawn id" example. One field in the file covers `lines` and `command` mode alike.
- F8 — fixed differently. The D9 guard Stuck names the changed paths and moved HEADs, and the way forward restores the worktree to the snapshot, then `lym loom resume`. A resumed row compares against the recorded snapshot.
  Not made as proposed:
  - The snapshot is not stored in `scope.json`. It goes in `.lym/drift/judge-snapshot.json`, outside both trees, so the guard still catches the judge's edits to `scope.json` and `judged.json` (self-check, fix-consistency 1).
  - The way forward is wider than `git checkout <head> -- <path>`. It also resets a moved branch and deletes added untracked paths (self-check, fix-proposal 3 and fix-way-forward 4).
- F9 — fixed as proposed. D9's and D10's `does not match` Stucks name amend and the code-fixing route. An exhausted `judgement` entry is a Stuck whose resume respawns the judge. D7's failed current capture is a Stuck naming its log, `lym loom resume`, the code-fixing route and `lym drift defer`; the last two were added after self-check, fix-way-forward 1, and D12 admits `defer` there. The criteria were updated.
- F10 — fixed as proposed. In D7 every capture, the main one included, takes the slot before the lock, and a held lock is waited on. The rationale was corrected and states the bound that a waiting check holds an idle slot. A D7 criterion was added.
- F11 — fixed as proposed. On entry, D4 ends chairs under the roles `drift-baseline` and `drift-verify` through `runproc.EndRole`. The new seats take `drift-expect` and `drift-judge`, so the sweep never ends them. Open risks and the D4 criterion were updated.
- F12 — fixed as proposed. In D12, `WaivedAt` makes both compare rows Done, with the waiver rendered. An uncounted capture is a Stuck at Drift-Verify naming `defer`, which is admitted there. Open risks and the D12 criterion were updated.
- F13 — fixed as proposed. D14 keeps `partial` and `skip` read-only.
- F14 — fixed as proposed: "with its `skip` list" was dropped from D15.
- F15 — fixed as proposed. D6 records `unsealed`, written only by Drift-Expect, Drift-Verify and Drift-Recheck, never by the entry or check (self-check, fix-proposal 1). A later seal is ignored. The bound names the remaining window, and the D6 criterion was extended.
- F16 — fixed as proposed. D6 refuses to seal over a moved warp, naming `lym drift defer` or an operator reset. D12 admits `defer` there, and the D6 criterion was extended.
- F17 — fixed as proposed. D4's Done-at-once rule includes scope `off`, a deferral and an old waiver, and the criterion was extended.
- F18 — fixed differently. D8 reports a missing baseline naming `lym drift defer`, and the D8 criterion was extended.
  Not made: a failing entry. The entry reports without failing, as F1 explains.
- F19 — fixed as proposed. D14 leaves the `gate.yaml` entry as one nothing reads and drops the prune claim.
- F20 — fixed as proposed: D8's rationale now names key `Re`.
- F21 — fixed as proposed: D11 now says "one of three things".

## Self-check

- fix-claims, MEDIUM — main capture takes lock before slot: fixed as proposed (D7: every capture, the main one included, takes the slot first).
- fix-consistency, MEDIUM — D9 exemption lets the judge edit `scope.json` and `judged.json`: fixed with own change (snapshot moved to `.lym/drift/judge-snapshot.json`; only `judgement.yaml` exempt; D9 criterion names the records).
- fix-consistency, MEDIUM — command-mode "passes" twins after the digest failure: fixed (the entry and check report baseline failures without failing; D2 and D8 name the baseline check).
- fix-consistency, LOW — D4 row 3 omits digest backfill: fixed as proposed.
- fix-consistency, LOW — D16 judge stencil omits the spawn id: fixed as proposed.
- fix-consistency, LOW — support-log Q5 contradicts D2: fixed as proposed.
- fix-consistency, LOW — D4 criterion "checks again" under command mode: fixed (adds "reusing stored rulings").
- fix-way-forward, MEDIUM — failed current capture can loop on resume: fixed (code-fixing route and `defer` named; D12 admits `defer`).
- fix-way-forward, MEDIUM — baseline failure at the entry goes to fixers that cannot clear it: fixed (reported without failing, merged with fix-consistency's command-mode item).
- fix-way-forward, MEDIUM — the entry and check would write `unsealed` and `diff` to `scope.json`: fixed (rows only; merged with fix-proposal 1).
- fix-way-forward, LOW — the revert cannot remove added untracked paths: fixed (merged with fix-proposal 3).
- fix-way-forward, LOW — moved-warp reset names no verb: fixed with own change (the reset is the operator's; the parent's way forward is `defer`).
- fix-proposal, MEDIUM — the entry and check write tracked records: fixed (merged above).
- fix-proposal, MEDIUM — a digest in `scope.json` is editable: fixed with own change (weft trailer, oldest commit wins).
- fix-proposal, MEDIUM — the revert cannot undo a moved HEAD: fixed (the restore resets the branch, restores files and deletes added paths).
- fix-proposal, MEDIUM — the `EndRole` sweep could end a judge with a shared role: fixed (new seats take `drift-expect` and `drift-judge`).
- fix-proposal, LOW — a waiting check holds an idle slot: fixed (a bound was added to D7's rationale).

Every fork returned a report.

## Disputed

None.

## Deferred

None.
