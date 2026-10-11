# Card 5 — The bootstrap's disagreeing-seed refusal names `rm` of the seed and a re-seed

**Edit:**
- `internal/loomcli#loomCLI.seedAndCommitBootstrap`
- `contracts/specs/refusal-spec.md`
- `internal/loomcli/smoke_prebootstrap_integration_test.go`
- `internal/loomcli/smoke_bootstrapwiring_test.go`

**Create:**
- `internal/loomcli#disagreeingSeedRefusal`

**Uses:**
- `internal/shedrun#ErrDisagreeingSeed`
- `internal/shedrun#SeedFile`
- `internal/shedrun#WriteSeed`

**Intent:**
Decision D4, its disagreeing-seed half.
When `lyx loom step` or `lyx loom start` finds a seed that disagrees with the one its bootstrap writes, such as a driverless seed typed with the wrong parent branch, `shedrun.WriteSeed`'s refusal says "keep the existing seed and drive it", which loops: driving that seed is what was just refused.

In `seedAndCommitBootstrap`, when `shedrun.WriteSeed` returns an error wrapping `shedrun.ErrDisagreeingSeed`, the bootstrap returns `disagreeingSeedRefusal(shedrun.SeedFile(c.location, shedrun.SelfRunID), existingSeed, parent)` at `bootstrapStageSeed` instead of the raw error.
`disagreeingSeedRefusal` wraps `shedrun.ErrDisagreeingSeed` with `%w` and renders a `loom:` message naming the existing seed's recipe, driver and params and the recorded parent the bootstrap would write, ending in `way forward: rm <seed path>, then lyx shed seed self --recipe <recipe> --driver <driver> --param parent=<parent>`, the path, the existing seed's recipe and driver, and the recorded parent all rendered from what the bootstrap read.
It never holds "keep the existing seed".
Every other `WriteSeed` error keeps propagating unchanged, and `lyx shed seed`'s own disagreeing-seed refusal in `shedrun` and its `seed disagrees` row in the shed section stay as they are.

The input that reaches the branch after every upstream check passes: a pair whose origin record resolves and whose `seed.json` exists with params, driver or recipe other than what `loomSeedFor` builds from the recorded parent, for example `parent=wrong-branch` on a go-driver loom seed; `TestLoomFailedReedUpRefuses` reaches it.

What the change lets through and its bound: the `rm` names one file, the addressed run's `seed.json`, and a plain `rm <path>` is not denied (`PATTERN-no-denied-recovery`), never `rm -rf`.
The re-seed must still agree with the recorded parent, which the next bootstrap enforces again, so the remedy can clear a wrong seed but never record a different parent.
`lyx shed seed` commits nothing and the bootstrap commits only after its seed stage succeeds, so a wrong seed reaches no history.

In the same commit (`PATTERN-refusal-way-forward`), the loom section of `contracts/specs/refusal-spec.md` gains a `start/step: seed disagrees` row: trigger, the bootstrap of `lyx loom start` or `lyx loom step` finds the run already seeded with values other than the recorded parent's seed; class correctness halt; way forward quoting the new clause with `<seed path>`, `<recipe>`, `<driver>` and `<recorded parent>` placeholders.

Coverage extends `TestLoomFailedReedUpRefuses` in `smoke_prebootstrap_integration_test.go`, with no new top-level test: a `wrong-parent seed` subtest builds `newBadReedUpFixture` with a seed function writing a go-driver loom seed whose `parent` param is not the recorded parent.
Its `lyx loom step` exits 1 with envelope kind `unseeded`, and its error holds `rm ` followed by the seed's path, `lyx shed seed self --recipe loom --driver go --param parent=` followed by the recorded parent, and not `keep the existing seed`.

Following the way forward to a driven producer needs a tmux server, so it extends `TestSmokeStepBootstrapWiring` in `smoke_bootstrapwiring_test.go`, the tmux tier Publish runs, with no new top-level test: a `wrong-parent seed is cleared and re-seeded` subtest, after the existing three, builds its own `newWiredPairFixture` with `registerBootstrapTeardown` and writes a go-driver loom seed whose `parent` param is not the recorded parent.
Its first `lyx loom step` fails; the subtest then removes the seed file, runs the built binary's `lyx shed seed self --recipe loom --driver go --param parent=<recorded parent>`, and runs `lyx loom step` again, which succeeds and dispatches Preflight, the pure-Go row the test's first subtest drives.
The test's doc comment gains the wrong-parent pair beside the one go-seeded pair it already names.

**ImpactSummary:** `seedAndCommitBootstrap` is shared by `lyx loom start` and `lyx loom step`; only its disagreeing-seed refusal's text changes, the stage and the sentinel it wraps stay, and `lyx shed seed`'s own refusal is untouched.

**Commit:** 5: loom: the bootstrap's disagreeing-seed refusal names rm and a re-seed
