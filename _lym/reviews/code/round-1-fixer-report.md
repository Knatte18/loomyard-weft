---
dispositions:
  - id: F1
    disposition: fixed-differently
    reason: "The baseline is read through a new git-CLI method, EnclosingCommit, not the existing HeadSHA seam: the records directory is a subdirectory of its repository, which the go-git read behind HeadSHA cannot open. A base the records repository does not hold is unverifiable too, and the ignore check was dropped because the blob comparison already decides every case it covered."
  - id: F2
    disposition: fixed-differently
    reason: "The persisted pre-fix head is taken over at the gate's first done arrival, and only while it is on HEAD's history, no batch has landed above it and every commit above it passes the commit check. Taking it unconditionally makes a hand merge between runs fail every later gate terminally, and keeps a head that later batches have passed."
  - id: F3
    disposition: fixed-differently
    reason: "Recording the closure's pre-fix head alone refuses an earlier run's vetted fix commits when a later run also fixes. A pass whose pre-fix head is the earlier FixedHead, or only clean parent merges above it, keeps the earlier FixedBase."
  - id: F4
    disposition: fixed-differently
    reason: "Line 308 also says 'when one is named', since the proposed wording names a note in a prompt with friction off."
  - id: F5
    disposition: fixed
  - id: F6
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
    disposition: fixed-differently
    reason: "The proposed text puts the words 'Friction note' in every prompt, which TestRenderPrompts_FrictionDirective (internal/websterengine/template_test.go:1072) refuses with friction off. The line says 'the friction note named near the top of this prompt, when one is named'."
  - id: F13
    disposition: fixed-differently
    reason: "The removed sentences are not restored verbatim: they put accept-audit's long help at 2056 bytes, over the 2000-byte cap of cmd/lym/clitree_test.go:413. The dropped fact, that a repeated call is harmless, is back in shorter wording."
  - id: F14
    disposition: fixed
---

# Round 1 fixer report

## Changes

- F1: `State.MasterRecordsBase` holds the records side's HEAD from before the Master spawn.
  A records-side path clears only when its content, absence included, matches that commit, and the restore step names it.
  Without the commit the path takes the reset route.
- F2: a restarted verify gate takes over the persisted `PreFixHead`, within the limits the disposition gives.
- F3: `State.FixedBase` records where the vetted range starts, and accept-audit allows only clean parent merges below it.
  Reset clears it.
- F6: a head refusal is a `headMismatchError`; any other git error is returned as it is.
- F4, F11, F12: the Master stencil and the orch role name the friction note.
- F13: the accept-audit help says again that a repeated call is harmless.
- F5, F7, F8, F9, F10, F14: the rows and assertions the findings ask for.
  F5 has both the run-exit row and the record-batch row.

Gate repair: the comments the gate listed as wrapped mid-sentence in `internal/websterengine` are rewritten one sentence per line.
Only comments changed.

Tier 0 passed for `internal/websterengine`, `internal/webstercli` and `contracts/...`, and tier 1 for the two webster packages.
After the gate repair, `go vet` and tier 0 passed again for `internal/websterengine`.

Second gate repair: the verify command failed on the help length F13's first fix caused,
since the earlier test runs left out `cmd/lym`.
After shortening the help, tier 0 passed for `cmd/lym`, the two webster packages and `contracts/...`.

Third gate finding: `TestGateTestMap/NamedPackageIsRebuiltWhenFresh` in `internal/gatecli` failed in tier 1.
No code was changed for it.
The branch touches nothing in `internal/gatecli` or `internal/testcover`.
The failure was one extra `PassedAlone: alpha.TestAlphaTwo` entry: that fixture test passed only when run by itself,
under the test's shortened per-test limit.
Run alone through `lym gate test --tier 1 ./internal/gatecli -- -run '^TestGateTestMap$'`,
all 38 targets passed with none skipped.
I read it as a load-dependent flake and left the test's timing as it is.
It can fail again in a later loaded run.

One consequence of F2 remains: after a restart, `reset --to pre-fix` targets the earlier run's head,
so it also drops commits made between the runs.

## Self-check

- fix-consistency 1, MEDIUM, `NewVerifyGate` doc on recording the pre-fix head: fixed as proposed.
- fix-consistency 2, LOW, `State.PreFixHead` comment: fixed as proposed.
- fix-consistency 3, LOW, verify-fix body on listed fix commits: fixed as proposed.
- fix-consistency 4, LOW, `runlevel.go` comment on Master's writes: fixed as proposed.
- fix-consistency 5, LOW, stencil line 308 unconditional: fixed as proposed.
- fix-consistency 6, LOW, records-side comments omit the no-base case: fixed as proposed.
- fix-affected-set 1, LOW, same as fix-consistency 4: fixed as proposed.
- fix-affected-set 2, LOW, same as fix-consistency 2: fixed as proposed.
- fix-code-correctness 1, MEDIUM, run-exit note row passes with the defect: fixed as proposed (asserts no `parent-write` warning).
- fix-code-correctness 2, MEDIUM, takeover dead end on a rejected merge: fixed with own change (the head is dropped when the commit check would reject a commit above it).
- fix-code-correctness 3, LOW, dead ignore check: fixed with own change (check removed; the `Ignored` row stays, pinning that such a write is not cleared).
- fix-proposal 1, MEDIUM, same as fix-code-correctness 2: fixed with own change.
  Its note on `reset --to pre-fix` is the consequence stated under Changes.
- fix-proposal 2, MEDIUM, ranges join only on exact equality: fixed with own change (clean parent merges also join).
- fix-proposal 3, LOW, same as fix-code-correctness 3: fixed with own change.
- fix-proposal 4, LOW, same as fix-consistency 5: fixed as proposed.

## Disputed

## Deferred

## Rulings
