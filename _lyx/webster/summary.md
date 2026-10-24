# gate-runs-once: tier-disjoint gates and test retag

Every batch in the plan reached `status: done`.

Batches 01 through 20 were recorded before this session resumed.
Batch 22 (tier-plan-docs) had died in an earlier recovery and finished in its second and last recovery, at head `00d1b470b`.

The remaining batches ran as one fork each, and each recorded `done`:

- 24 retag-fabricengine, head `aa3ad57c8`: the fork retagged the fabricengine, loom, reed and shuttle test files.
  It moved shared helpers out of tier 0 files into untagged helper files, and tagged two support-only files in `internal/shuttleengine` `tier0`.
- 27 retag-webster-landing-batten, head `51fc8c9a7`: retagged the batten, landing and webster packages and moved tier 1 helpers into untagged helper files.
- 28 retag-shed-plan-testkit, head `25c04d488`: retagged the shed, plan and testkit packages.
  It tagged five support-only files `tier0` where the retag rule leaves them untagged, because their packages have no higher-tier tests.
- 29 retag-internal-a-f, head `54cc33e7b`: retagged the internal packages a through f and moved helpers to untagged files.
- 30 retag-internal-g-o, head `58b268d18`: retagged the internal packages g through o with no helper moves.
- 31 retag-internal-p-z, head `0bafac17b`: retagged the internal packages p through z and added the one-tier-tag-per-test-file scan in `cmd/lyx/tiertags_test.go`.
  It added the `PATTERN-tier-tags` line to `PATTERN.md` and filed the `drop-transitional-tag-spellings` board follow-up.

Scope guard: batch 31 reported one deviation, `cmd/lyx/tiertags_test.go`, the scan the card adds.
No other batch reported deviations, and no batch reported `Pattern-Departure:` trailers.

## Verify gate findings

The plan-level verify gate failed on two tier 0 tests:

- `github.com/Knatte18/loomyard/internal/shuttleengine/claudeengine.TestBuildSettings_MatchesGolden`
- `github.com/Knatte18/loomyard/cmd/lyx.TestCLIForm_EveryCommand`

The findings file named cards 22-tier-plan-docs and 24-retag-fabricengine as touching a failing package.


## Verify gate fix commits

The verify-gate fixers' commits above the pre-fix head, and the PATTERN departures their messages declare.

- `094e2ab1bf1fa6ff406f04bb08ce7373a6a47cc9` fix: settings golden carries the tier2 and tier3 shell-guard alternation
- `621fe202e60f9f5c03334f68558fe829bc931383` fix: lyx gate test and lyx batten run Long pages stay within the 2000-byte cap
