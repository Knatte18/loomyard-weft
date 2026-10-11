# Card 6 — round-gate-headroom

**Create:**
- `internal/loomshed#headroomFindings`

**Edit:**
- `internal/loomshed#NewVerifyGate` -> `func NewVerifyGate(worktreeRoot, verifyDir, siteLabel string, command, mergeBase func() (string, error), wholeDiff bool, slots *gateslot.Pool) shuttleengine.Gate`
- `internal/loomshed#TestVerifyGate_Scenario`
- `internal/loomshed#roundGateFixture.runGate` -> `func (f roundGateFixture) runGate(t *testing.T, mergeBase func() (string, error)) (passed bool, findings string)`
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
- `contracts/specs/shed-recipe-spec.md`
- `contracts/recipes/loom-recipe.yaml`
- `pattern/PATTERN-gate-self-check-parity.md`
- `pattern/PATTERN-test-speed.md`

**Uses:**
- `internal/patternlint#Lint`
- `internal/patternlint#WayForward`
- `internal/loomcli#pairMergeBase`
- `internal/fabricengine#OriginRecordPath`

**Intent:** Run the headroom lint in the Webster-Burler round gate from the task branch's merge base with its recorded parent branch to HEAD, before any test, the second gate site of decision D3, without switching a loom run's gate to the whole-diff form.
Today a told merge base means the whole-diff form, so the form becomes its own told flag: `NewVerifyGate` gains `wholeDiff bool`, and `mergeBase` means only the reader of the merge base.
With `wholeDiff` false the gate keeps the round form: the comment lint's base stays the told command's latest recorded pass and the round command stays the impacted-set derivation; when `mergeBase` is non-nil the gate additionally calls `patternlint.Lint(worktreeRoot, base)` with the merge base it reads, after the guard scan and before the comment lint and any test, and a finding fails the gate with `headroomFindings`, which renders the finding's message and `patternlint.WayForward` plus the instruction to re-step the row.
With `mergeBase` nil the headroom lint is skipped, which is the standalone run that records no parent branch.
With `wholeDiff` true the gate keeps its present whole-diff behavior and the one reader serves both lints, headroom first; `wholeDiff` true with a nil reader is a returned error naming the missing reader, as the empty told command is.
A merge base read error stays a returned gate error in both forms.
`NewVerifyGate`'s doc comment describes the three cases, and `internal/loomshed/doc.go`'s verify gate paragraphs say the round form runs the headroom lint from the merge base when told one and that the form is now a told flag rather than the reader's presence.
`shedrecipe.Env` gains the field `VerifyWholeDiff bool`, documented beside `VerifyMergeBase`; `resolvePlainGateClosure` passes it, and `resolveGateSpec`'s doc comment on the `verify` gate names the flag and the reader's new meaning.
`VerifyMergeBase`'s field comment, which says nil selects the round form and set selects the whole-diff form, is rewritten to say the reader is the merge base of the headroom lint, or of both lints when `VerifyWholeDiff` is set, and that nil skips the headroom lint; `VerifyWholeDiff`'s comment says it alone selects the form.
`contracts/specs/shed-recipe-spec.md`'s `verify` sentence, which says the gate takes the whole-diff form when `Env.VerifyMergeBase` is set, is rewritten the same way: `Env.VerifyWholeDiff` selects the whole-diff form, and `Env.VerifyMergeBase`, when set, is the base of the headroom lint, which runs before the comment lint, or of both lints under the whole-diff form.
`contracts/recipes/loom-recipe.yaml`'s comment on the Webster-Burler row's `gates` key, which says the gate first lints the comments the round added, says the gate first runs the headroom lint from the merge base with the parent branch, then lints the comments; the comment is the only change to that file.
`loomCLI.wire` tells `env.VerifyMergeBase` the `pairMergeBase` reader for every pair run, deciding at wire time with a mutation-free check: it stats `fabricengine.OriginRecordPath(location)` and sets the reader only when the record file exists, leaving it nil otherwise, so a standalone run, which records no parent branch, skips the lint.
Wire never calls `fabricengine.ReadOrigin`, which creates the weft lock directory and seeds the weft exclude, so wire still opens and writes nothing, and a wire fixture with no weft repository and no origin record wires as before with the reader nil.
The reader stays lazy as darn's is: it reads the origin record and the merge base on each call, so an unreadable record or a merge base read error is a returned gate error, never a wire error; a stat error other than not-exist is wire's error, as any other unreadable wire input is.
`wireDarn` keeps setting the reader unconditionally, so a darn run's gate is told the reader exactly as today, and additionally sets `env.VerifyWholeDiff` to true; `darnwiring.go`'s header comment no longer claims the merge base reader is darn-only.
`TestWire_DarnRecipe`'s loom-seed step asserts the reader is nil and `VerifyWholeDiff` is false over its fixture, which writes no origin record, and gains an assertion that a loom seed whose fixture writes an origin record file at `fabricengine.OriginRecordPath` gets the reader with `VerifyWholeDiff` false; its darn-seed step asserts both are set.
No other wire test changes: the stat answers not-exist in every fixture built by `hubLocation`, so their wiring is unchanged.
`roundGateFixture.runGate` takes an optional merge base reader and passes `wholeDiff` false; `wholeDiffGate` passes true; `TestVerifyGate_Scenario` and `TestGateParity_VerifyGate` pass nil and false.
`TestVerifyGate_RoundScenario` gains ordered steps over a `PATTERN.md` fixture sized with `pattern.HeadroomBytes`: told the merge base and the round form, a branch that grew the overview past the line fails naming the bytes to cut and the way forward before any command runs, which the verify log count shows, and a second arrival after the growth is cut passes and records the narrowed command, so the round form was kept; a branch that did not grow it passes; after a merge-in of a parent commit whose overview sits above the line, a branch that never grew it passes and one that grew it on top fails naming only its own growth; told no reader, the same growth passes the lint.
The whole-diff steps pass true and keep their assertions, and the read-error table gains the nil-reader-with-whole-diff case.
`contracts/specs/refusal-spec.md`'s "Webster-Burler verify gate exhausted" row adds the cause that the branch grew `PATTERN.md` past its headroom line, and the way forward to cut the bytes the finding names from `PATTERN.md`, moving detail into the entries' background files, then re-step the row.
`pattern/PATTERN-gate-self-check-parity.md`'s headroom pair gains the round gate as its third caller, and `pattern/PATTERN-test-speed.md`'s gate-per-tier table names the headroom lint first in the Webster-Burler round gate row.
Bound: the lint can block only a change that grew `PATTERN.md` since its merge base and left it above the line; a standalone run's growth is checked by no lint; both are bounded by the hard cap every full verify runs.

**ImpactSummary:** Re-signs NewVerifyGate, whose call sites are all on this card; wire gains one stat of the origin record path and no write, every pair run's gate reads the origin record and the merge base lazily, and the darn gate's form is told explicitly instead of inferred.
