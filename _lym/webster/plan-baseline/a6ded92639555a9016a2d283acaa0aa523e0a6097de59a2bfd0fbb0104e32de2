# Card 15 — beam-status-adopt

**Edit:**
- `internal/beamcli#statusResult`
- `internal/beamcli#dobbyStatus`
- `internal/beamcli#statusCmd`
- `internal/beamcli_test#TestStatus_PrintsThePrimesDobbyAndEachLocalBeams`
- `internal/beamcli/doc.go`
- `docs/overview.md`

**Create:**
- `internal/adoptcli#CheckView`
- `internal/adoptcli#ReadCheckView`
- `internal/adoptcli#checkView`
- `internal/adoptcli#TestCheckView`

**Extends:**
- `internal/adoptcli/doc.go`

**Uses:**
- `internal/adoptcli#ReadCheck`
- `internal/adoptcli#checkAlive`
- `internal/adoptcli#CheckRecord`

**Intent:**
Decision 11: `lym beam status` shows the adopt check.

In `adoptcli`:
- `CheckView` holds the check's `State` as shown (`running`, `green`, `failed` or `ended without a verdict`), the tier in flight, the tiers done with their outcomes, the end time, the paths of `suite.log` and `check.out`, and `Lines`, the rendered section.
- `checkView` builds it: a running record shows the tier in flight and the tiers done; a green or failed record shows its end time, and a failed one its failing tier; a running record that `alive` reads dead shows `ended without a verdict`, naming `suite.log`, `check.out` and the rerun `lym gate test --gate adopt` in the prime, and that the record clears at the next `lym adopt` with an owner write, which starts a fresh check, since the rerun writes no record. Every view's lines name the log path.
- `ReadCheckView` reads `check.json` under `fabricengine.AdoptScratchDir(hubPath)` with `ReadCheck`, which on an absent record reports false having created no directory or lock file, and returns nil then; judges a running record with `checkAlive` (a pid that is not alive, or a live process whose start ticks differ from the record's, as after a reboot that reused the pid).

In `beamcli`: `statusResult` gains `AdoptCheck *adoptcli.CheckView` with the tag `json:"adopt_check,omitempty"`; `dobbyStatus` fills it from `ReadCheckView(prime.HubPath)`, a read error joining the returned error like a dobby's, the dobbies still read; `statusCmd`'s Long text names the adopt check section.

Tests:
- `TestCheckView` (tier0): a running record alive shows the tier in flight and the tiers done; a green and a failed record show their end time, the failed one its failing tier; a running record read dead shows `ended without a verdict` naming `suite.log`, `check.out`, `lym gate test --gate adopt` and that the next `lym adopt` with an owner write starts a fresh check; the start-ticks case reaches `checkView` as the same dead reading, which card 14's `TestCheckAlive` pins.
- `TestStatus_PrintsThePrimesDobbyAndEachLocalBeams` asserts the envelope has no `adopt_check` with no record and that the hub's adopt scratch directory is still absent after the call, then writes a green `check.json` under the hub's adopt scratch directory and asserts `adopt_check` shows `green` and the log path; a hub with no record shows nothing.

Docs: `internal/beamcli/doc.go` and `docs/overview.md`'s beam status sentence name the adopt check section; `internal/adoptcli/doc.go` gains the view.

**ImpactSummary:** One additive envelope key on `lym beam status`, read from one record; the dobby and orch entries are unchanged.

**Commit:** 15: Show the adopt check in lym beam status
