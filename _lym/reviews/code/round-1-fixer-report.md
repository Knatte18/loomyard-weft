---
dispositions:
  - id: F1
    disposition: fixed
  - id: F2
    disposition: fixed
  - id: F3
    disposition: fixed
  - id: F4
    disposition: deferred
    reason: "Whether the narrowing of the webstercli ask stands is the run's parent's ruling, as the finding says. No code changed for it. If the parent wants the XDG_STATE_HOME, LOCALAPPDATA, LYM_TRACE and t.Chdir tests made parallel, that is a board note for a new task."
  - id: F5
    disposition: disputed
    ground: premise false
    rebuttal: "No test the branch moved onto the pair template only reads its hub as SharedHub counts reading. TestPairSlugOf, the one the finding names, writes into the hub: switched to SharedHub, its cleanup failed with `shared fixture \".+pair\" was written to`, and the poisoned template then failed the later tests that copy it. The origin read takes a shared file lock, which can create the lock file. The other moved tests commit, write files, upsert board entries, add pairs, run CLI verbs or spawn git, and a spawned git subcommand counts as a write (TestCreateBeamFromParent and TestWiredStartPusher_AsksForThePairsPusher reach `git worktree list`)."
    evidence:
      - internal/hubforge/doc.go:26
      - internal/hubforge/doc.go:28
      - internal/hubforge/template.go:501
      - internal/loomcli/gatefixer_integration_test.go:34
      - internal/state/state.go:112
      - internal/lock/lock.go:85
      - internal/fabricengine/worktreelist.go:171
  - id: F6
    disposition: fixed
  - id: F7
    disposition: fixed-differently
    reason: "The proposed sentence says the way-forward tests reach fabricSync through a failing opener. TestBracketVerbs_NoRunInProgressWayForward is a way-forward test that builds with newSkipGitVerbsFixture and uses no failing opener, so the header names the tests of a verb's fabric-sync refusal instead."
  - id: F8
    disposition: fixed-differently
    reason: "The proposed hubforge.CopyHub(t, pairHubShape) does not compile: ownercheck_integration_test.go is package loomcli_test and pairHubShape is unexported in package loomcli. export_test.go now re-exports it as PairHubShape, and the test copies loomcli.PairHubShape."
---

# Fixer report, Code-Review round 1

## Changes

- F1 (`204d20239`): `lym loom fmt` on the nine `internal/webstercli` files; `gofmt -l internal/webstercli internal/loomcli` prints nothing.
- F2 (`9eb940372`): `TestWire_ModeHubSelectsHubMode` and `TestWire_StandaloneMode` set `FABRIC_SKIP_GIT`, `FABRIC_SKIP_PUSH`
  and the strand-name variable, and assert `syncOptions` and `strandName` after wiring.
- F3: replaced the false `loomcli` measurement sentence in `_lym/landing/summary.md` (not committed).
- F6 (`842697169`): restored the space in the `seedLoomConfig` doc comment.
- F7 (`462560414`, `d9ef714d3`), fixed differently: rewrote the `verbs_test.go` header sentence.
  The second commit corrects the first after the self-check.
- F8 (`c039151e1`), fixed differently: `export_test.go` exports `PairHubShape`, and `TestRunOwnerVerdict` copies it.
- From the self-check, in `_lym/landing/summary.md`: the "Still serial" bullet names the two wiring tests,
  and the F3 sentence is broken to fit the line limit.

Tier0 and tier1 of `internal/webstercli` and `internal/loomcli` passed on the tree through `c039151e1`.
Tier0 and tier1 of `internal/webstercli` passed again after `d9ef714d3`.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency, LOW, F3 sentence over the line limit: fix as proposed.
- fix-consistency, LOW, "Still serial" bullet stale after F2: fix as proposed.
- fix-proposal, MEDIUM, `verbs_test.go` header false for some way-forward tests: fix with your own change.
  The sentence is scoped to a verb's fabric-sync refusal.
  `TestFabricSyncWayForward_NextSyncCommitsSavedState` calls `fabricSync` directly and no verb,
  so the header's first sentence, about a test whose verb reaches it, needs no change for that test.
- fix-proposal, LOW, F3 sentence over the line limit: fix as proposed (same as the first fix-consistency finding).
- fix-affected-set, LOW, `verbs_test.go` header covers too many tests: fix as proposed (same as the fix-proposal MEDIUM).

## Disputed

- F5, premise false: see the frontmatter entry.
  The `SharedHub` switch was made, failed the tier1 run and was reverted, so nothing of it is committed.

## Deferred

- F4: the run's parent rules whether the narrowing stands.

## Rulings

None.
