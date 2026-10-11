# Card 6 — round-gate-headroom

**Create:**
- `plan:internal/loomshed#headroomFindings` -> `func headroomFindings(f pattern.Finding) string`

**Edit:**
- `internal/loomshed#NewVerifyGate` -> `func NewVerifyGate(worktreeRoot, verifyDir, siteLabel string, command, mergeBase func() (string, error), wholeDiff bool, slots *gateslot.Pool) shuttleengine.Gate`
- `internal/loomshed#TestVerifyGate_Scenario`
- `internal/loomshed#roundGateFixture.runGate`
- `internal/loomshed#roundGateFixture.wholeDiffGate`
- `internal/loomshed#TestVerifyGate_RoundScenario`
- `internal/loomshed/doc.go`
- `internal/shedrecipe#Env`
- `internal/shedrecipe#resolveGateSpec`
- `internal/shedrecipe#resolvePlainGateClosure`
- `internal/webstercli#TestGateParity_VerifyGate`
- `internal/loomcli#loomCLI.wire`
- `internal/loomcli/darnwiring.go`
- `internal/loomcli#TestWire_DarnRecipe`
- `contracts/specs/refusal-spec.md`
- `pattern/PATTERN-gate-self-check-parity.md`
- `pattern/PATTERN-test-speed.md`

**Uses:**
- `plan:internal/patternlint#Lint`
- `plan:internal/patternlint#WayForward`
- `plan:internal/loomcli#pairMergeBase`

**Intent:** Run the headroom lint in the Webster-Burler round gate from the task branch's merge base with its recorded parent branch to HEAD, before any test, the second gate site of decision D3, without switching a loom run's gate to the whole-diff form.
Today a told merge base means the whole-diff form, so the form becomes its own told flag: `NewVerifyGate` gains `wholeDiff bool`, and `mergeBase` means only the reader of the merge base.
With `wholeDiff` false the gate keeps the round form: the comment lint's base stays the told command's latest recorded pass and the round command stays the impacted-set derivation; when `mergeBase` is non-nil the gate additionally calls `patternlint.Lint(worktreeRoot, base)` with the merge base it reads, after the guard scan and before the comment lint and any test, and a finding fails the gate with `headroomFindings`, which renders the finding's message and `patternlint.WayForward` plus the instruction to re-step the row.
With `mergeBase` nil the headroom lint is skipped, which is the standalone run that records no parent branch.
With `wholeDiff` true the gate keeps its present whole-diff behavior and the one reader serves both lints, headroom first; `wholeDiff` true with a nil reader is a returned error naming the missing reader, as the empty told command is.
A merge base read error stays a returned gate error in both forms.
`NewVerifyGate`'s doc comment describes the three cases, and `internal/loomshed/doc.go`'s verify gate paragraphs say the round form runs the headroom lint from the merge base when told one and that the form is now a told flag rather than the reader's presence.
`shedrecipe.Env` gains the field `VerifyWholeDiff bool`, documented beside `VerifyMergeBase`; `resolvePlainGateClosure` passes it, and `resolveGateSpec`'s doc comment on the `verify` gate names the flag and the reader's new meaning.
`loomCLI.wire` tells `env.VerifyMergeBase` the `pairMergeBase` reader for every pair run: it reads the fabric origin record once at wire time through `fabricengine.ReadOrigin` and sets the reader only when a record with a parent branch is found, leaving it nil otherwise, so a standalone run skips the lint and a read error at wire time is wire's error.
`wireDarn` no longer sets the reader, since wire did, and sets `env.VerifyWholeDiff` to true; `darnwiring.go`'s header comment no longer claims the merge base reader is darn-only.
`TestWire_DarnRecipe`'s loom-seed step asserts the reader is set and `VerifyWholeDiff` is false, and its darn-seed step asserts both are set.
`roundGateFixture.runGate` takes an optional merge base reader and passes `wholeDiff` false; `wholeDiffGate` passes true; `TestVerifyGate_Scenario` and `TestGateParity_VerifyGate` pass nil and false.
`TestVerifyGate_RoundScenario` gains ordered steps over a `PATTERN.md` fixture sized with `pattern.HeadroomBytes`: told the merge base and the round form, a branch that grew the overview past the line fails naming the bytes to cut and the way forward before any command runs, which the verify log count shows, and a second arrival after the growth is cut passes and records the narrowed command, so the round form was kept; a branch that did not grow it passes; after a merge-in of a parent commit whose overview sits above the line, a branch that never grew it passes and one that grew it on top fails naming only its own growth; told no reader, the same growth passes the lint.
The whole-diff steps pass true and keep their assertions, and the read-error table gains the nil-reader-with-whole-diff case.
`contracts/specs/refusal-spec.md`'s "Webster-Burler verify gate exhausted" row adds the cause that the branch grew `PATTERN.md` past its headroom line, and the way forward to cut the bytes the finding names from `PATTERN.md`, moving detail into the entries' background files, then re-step the row.
`pattern/PATTERN-gate-self-check-parity.md`'s headroom pair gains the round gate as its third caller, and `pattern/PATTERN-test-speed.md`'s gate-per-tier table names the headroom lint first in the Webster-Burler round gate row.
Bound: the lint can block only a change that grew `PATTERN.md` since its merge base and left it above the line; a standalone run's growth is checked by no lint; both are bounded by the hard cap every full verify runs.

**ImpactSummary:** Re-signs NewVerifyGate, whose four callers are on this card; every pair run's gate now reads the origin record and the merge base, and the darn gate's form is told explicitly instead of inferred.
