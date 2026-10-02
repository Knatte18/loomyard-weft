---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    location: _lyx/plan/08-callers-address-roles.md:22
    summary: Renaming the driver role to driver leaves a live or dormant pre-task loom-driver strand invisible to the bootstrap and batten lookups, so an in-flight run spawns a second driver and keeps an unremovable corpse
  - id: F2
    severity: MEDIUM
    location: _lyx/plan/06-geometry-naming.md:59
    summary: No card carries the standalone acceptance criterion; nothing spawns a strand through standalonegeom's geometry and checks the code-role name, LYX_STRAND_NAME set and LYX_PARENT unset
  - id: F3
    severity: LOW
    location: _lyx/plan/08-callers-address-roles.md:32
    summary: Card 8's ImpactSummary claims behaviour is unchanged apart from the role text, hiding the in-flight driver consequence F1 describes
  - id: F4
    severity: LOW
    location: _lyx/plan/07-reed-forms-names.md:39
    summary: A malformed recorded .lyx-code refuses with way forward lyx fabric code, but cards 2 and 3 refuse any code that differs from the record, so the named way forward is a dead end
  - id: F5
    severity: LOW
    location: _lyx/plan/01-agentname-package.md:25
    summary: Resolve's foreign-prefix refusal names both prefixes but no way forward, against the record's Refusal Way-Forward constraint that every new refusal names its remedy
  - id: F6
    severity: LOW
    location: _lyx/plan/11-claude-name-flag.md:20
    summary: Card 11's ImpactSummary restates Intent instead of concluding on the blast radius, such as resume lines stored before deploy and the dependence on reed's export
---

### [MEDIUM] The driver rename strands every pre-task `loom-driver` strand

**Location:** `_lyx/plan/08-callers-address-roles.md:22` (with `_lyx/plan/01-agentname-package.md:26`)

**Issue:**
Card 8 moves `loomengine.LoomDriverStrandName` from `loom-driver` to `driver`, and every driver lookup queries `driver` through `agentname.Matches`.
Card 1's `Matches` reaches a name that fails `Parse` only by exact equality, so `Matches("loom-driver", "driver")` is false.
The Shared Decision's own claim, that a strand recorded before this task stays reachable, therefore holds only for a caller that still queries the old literal.
For the driver only the old driver's own baked-in teardown (`lyx reed remove --name loom-driver`) does; `loom-status` and `orch` keep their literals and are unaffected.
Consequences for a run in flight across the deploy:
- `resolveDriverStrandAction` (`internal/loomcli/bootstrap.go:319`) reports `driverStrandNone` for a live `loom-driver`.
  Between `lyx shed step` calls the run lock is free, so `mustSpawnDriver` returns true and a re-entered `lyx loom start` launches a second driver beside the live one.
- A dead `loom-driver` corpse is never found, so the bootstrap never removes it.
  Reed's resume replays every strand after a server rebirth, which relaunches the old driver next to the new one.
- `driverAliveFrom` (`internal/battencli/wire.go:56`) reports the live old driver as dead, so batten's inner-run wait (`internal/battenshed/innerrun.go:385`) finishes at once instead of granting the driver its exit grace for the stop report.

**Fix:** Card 8 keeps the pre-task literal as a named legacy constant in `loomengine` (`LegacyLoomDriverStrandName = "loom-driver"`), and both driver lookups (`resolveDriverStrandAction` and `driverAliveFrom`) also match a strand named exactly that.
That lets a live old driver be seen as live and lets the bootstrap remove a dead one by GUID.
Card 8's tests add a legacy-`loom-driver` case to the bootstrap and batten lookups.

### [MEDIUM] The standalone acceptance criterion has no carrying card

**Location:** `_lyx/plan/06-geometry-naming.md:59`

**Issue:**
The decision record's acceptance criteria require that a standalone run with no `.lyx-code` spawn strands named `<code>:<role>`, with the code derived from the hash8, `LYX_STRAND_NAME` set and `LYX_PARENT` unset.
Card 6 tests only `standalonegeom.ReedGeometry`'s field value, card 1 only the code grammar, and cards 7 and 10 test an empty slug only on hub-shaped geometries (`newTestEngine`, the reedcli `hubforge` hub).
The path itself works: `webstercli/wiring.go:242` and `burlercli/wiring.go:178` build their engine from `standalonegeom.ReedGeometry`.
The derived code `s` plus five lowercase hex digits (`hex.EncodeToString` in `standalonestate`) passes card 7's `ValidateCode` check, and the empty slug skips the slug check.
Still, no test spawns through that geometry, so the criterion is unverified.
A `lyx webster run --target-dir` test that spawns real producers would need a live Claude, which is out of reach for the integration tier.
The engine the standalone CLIs build is the testable seam.

**Fix:** Card 10 creates `internal/standalonegeom/naming_integration_test.go` (`integration` tag, real tmux).
On a plain temp git checkout with no `.lyx-code`, it builds the reed engine exactly as the standalone wiring does, from `standalonestate.Derive` and `standalonegeom.ReedGeometry`.
It adds a `worker` strand running `sh`, and asserts the state name and pane title are `<agentname.StandaloneCode(hash8)>:worker`, the process sees `LYX_STRAND_NAME` with that value, and `LYX_PARENT` is unset.
`./internal/standalonegeom/...` is already in the verify integration list.

### [LOW] Card 8's ImpactSummary hides the in-flight driver consequence

**Location:** `_lyx/plan/08-callers-address-roles.md:32`

**Issue:** "behaviour is unchanged apart from the driver's role reading `driver`" is false for a worktree whose driver predates the deploy (F1), so the summary omits the one blast-radius effect the card has.

**Fix:** State the conclusion: every lookup keeps finding a full-name strand, and pre-task `loom-status` and `orch` strands are still reachable by equality.
A pre-task `loom-driver` is reached only through the legacy constant F1's fix adds.

### [LOW] A malformed recorded code names a way forward that refuses

**Location:** `_lyx/plan/07-reed-forms-names.md:39` (with `_lyx/plan/02-fabric-code-record.md:36` and `_lyx/plan/03-fabric-code-verb.md:25`)

**Issue:**
Card 7 refuses a recorded code that fails `ValidateCode` with the way forward `lyx fabric code <code>`.
Card 2's `ReadCode` returns any non-blank value as found, though, and card 3 refuses a code that differs from the record.
Card 2's clone refuses a differing `--code` too.
A hand-damaged `.lyx-code` therefore leaves no `lyx` verb that fixes it, against the Refusal Way-Forward constraint the record cites.

**Fix:** Card 2's `ReadCode` reports a value failing `agentname.ValidateCode` as not found, and logs a named warning.
Every reader then treats the hub as uncoded, so `lyx fabric code <code>` records over the damaged value and clone takes `--code`.
Card 7's one remaining refusal is the empty code.
A told code failing the grammar becomes a wiring guard, since neither teller can produce one.
`codebinding_test.go` gains the malformed-record case.

### [LOW] Resolve's foreign-prefix refusal names no way forward

**Location:** `_lyx/plan/01-agentname-package.md:25`

**Issue:** The record's constraints say every new refusal names its remedy, but card 1's `Resolve` refusal names only the two prefixes.

**Fix:** The refusal names its way forward: pass the role segment alone, or run the command from the worktree the full name belongs to.

### [LOW] Card 11's ImpactSummary restates Intent

**Location:** `_lyx/plan/11-claude-name-flag.md:20`

**Issue:** "Adds one flag … every Claude strand launch and resume carries it" restates the Intent; it gives no blast-radius conclusion.

**Fix:** Conclude on the effect.
Only reed-launched panes run these lines, and reed exports `LYX_STRAND_NAME` first (card 9), so no line expands to an empty name.
A resume line stored in reed state before the deploy lacks the flag, so its session comes back unnamed until the watchdog repairs it.

## Focus verdicts that raised no finding

- **Standalone code check (focus 2, second half):** the in-process standalone engine reaches card 7's check with the derived code, which fits the grammar, so a standalone spawn is not refused. Only the test coverage is missing (F2).
- **Scope license (focus 3):**
  - Reed's default role `strand` gives every strand a name, as the Goal requires, without a new refusal.
  - `Matches`'s legacy equality keeps existing state addressable.
  - `Resolve`'s foreign-prefix refusal implements the decided `--name` reading of a full name. Its missing way forward is F5.
  - `WatchNames` is the decided watchdog tick, and `Watch` is a signal-driven resize loop with no repair cadence to share.
  - The two refusal-spec rows are what the Refusal Way-Forward Invariant requires for a refusal reachable through shed, loom and webster verbs.
  
  All of these are licensed.
- **Cards 12 and 13 (focus 4):** the session-name repair can work as specified.
  - `reedengine.Strand` carries `SessionID`, and shuttle sets it in `AddSpec` (`internal/shuttleengine/run.go:319`).
  - In hub mode `hubgeom` sets `PaneCwd` to `l.AnchorPath()`, the same directory shuttle's `NewRunner` uses as `paneCwd` for its own transcript audit through `claudeProjectDirFor`.
  - `reedcli` does not import `cliwire` today, and neither `reedengine` nor `claudeengine` depends on `reedcli` or `cliwire`, so card 13 closes no import cycle. The allowlist grows by exactly the two packages the card names.
- **Granularity (focus 5):** card 7's shuttle and reedcli edits are compile-forced by `AddSpec.Round`'s removal, and its refusal-spec rows are invariant-forced into the same commit.
  Card 9 has its own unit test over the composed launch line.
  Card 10's real-tmux tests are cumulative over cards 7, 9 and 10 and can only be written after all three, so they sit where they belong.
- **ImpactSummary (focus 6):** cards 6, 12 and 15 each state a conclusion (every hub-mode wiring caller plus the tagged tests; a second goroutine per watched worktree with the resize loop untouched; prompt text only). Only cards 8 and 11 fall short (F3, F6).
