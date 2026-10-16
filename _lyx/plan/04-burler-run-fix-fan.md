# Card 4 — burler-run-fix-fan

**Edit:**
- `internal/burlercli#profileYAML`
- `internal/burlercli#decodeProfile`
- `internal/burlercli#burlerCLI.runCmd`
- `internal/burlercli#TestDecodeProfile`
- `internal/burlercli#TestRunCmd`
- `internal/burlercli/doc.go`

**Intent:**
Let `lyx burler run` run the twill fixer standalone (D7): it has no shed usage record, so it reads the fixer fan from its own profile.

`profileYAML` gains `FixFan` under the kebab-case key `fix-fan`, and `decodeProfile` maps it onto `burlerengine.Profile.FixFan`.
`runCmd` sets `opts.FixKey` to `shuttleengine.ModelKey{DefaultEngine: true}`, as it does `ReviewKey`, since the command resolves no key, and `opts.FixFanKey` to the profile field's name, `fix-fan`, so a twill fixer refused by the default engine names `engine: in shuttle.yaml` and the profile's `fix-fan` through card 3's wording.
The verb's long help shows `fix-fan` in its example profile and says a profile naming a fixer fan runs the twill fixer, whose forks check its fix before the fixer-report.
An unknown fan is refused by the engine's `validateFix` before any spawn, as an unknown `cluster-fan` is refused by `validateReview` today; there is no new CLI refusal.

Tests:
`TestDecodeProfile`'s `FullValid` row sets `fix-fan` and asserts `FixFan`.
`TestRunCmd` gains a subtest over its `halfShuttle`: a profile naming `fix-fan: fix-plan` over a blocking review runs the fixer with `ForkSubagents` set on its recorded spec, while the existing blocking-review subtest asserts the fixer's spec does not fork.
The halfShuttle's done result carries no fork audit, so the twill fixer ends with card 3's count warning and the verb still succeeds.
`internal/burlercli/doc.go` names the `fix-fan` profile key.

**ImpactSummary:** the profile decode and the run verb's options gain one field each; a profile without `fix-fan` runs exactly as before, and no other package calls these members.
