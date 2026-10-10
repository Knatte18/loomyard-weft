---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: scope
    location: internal/loomcli/integration_driverbootstrap_test.go:59
    summary: Card 17 emptied driverSkills, but the tmux-tier driver-bootstrap test's stub still waits for typed skill-load and pointer lines, so TestIntegrationDriverBootstrap_ReturnsWithoutWaitingOnTheDriver fails Publish
---

### [BLOCKING] The driver-bootstrap integration stub still follows the skill-load launch flow card 17 removed

**Location:** `internal/loomcli/integration_driverbootstrap_test.go:59` (`integrationWriteStubDriverScript`), and its header and doc comments at lines 46-58 and 91-103.

**Issue:**
This is the failing test the Publish section names, and its cause is in the branch's own range, not in the merge-in.
The merge-in commit `fec3d1d42` brought in nothing that touches this path.
The branch's own range is `main..HEAD`, since `main` is `abe2aa50e`, the merge's second parent.

Card 17 sets `driverSkills` to empty (`internal/loomcli/driverspec.go:18`).
`claudeengine.Prepare` puts the prompt pointer on the launch line when the spec names no skills (`internal/shuttleengine/claudeengine/claudeengine.go:172-176`).
It types the pointer afterwards, through `Launch.PromptLine`, only when skills are named.
So shuttle now types nothing after the optional `/color` line.

The stub script was written for the old flow.
After `/color` it does `IFS= read -r line` twice, once for the skill-load message and once for the pointer, and takes the `prompt.md` path from that second line.
Neither line ever arrives, so it blocks on stdin and never writes the drive report.
`waitForDriveReport` then fails at line 178 after 10s: `no single drive-report-*.md file appeared ... (matches=[])`.
This matches `.lyx/verify/publish-failure.log` (`integration_driverbootstrap_test.go:178`, 11.92s).

Card 17 lists `driverprompt_test.go` and `driverspec_test.go` under `Edit:`, but not this test.
The plan's `## verify:` section runs only the untagged and `integration` tiers, never `tmux`, so the card's own Verify could not have caught it.
The file's `tmux` build tag is the reason it was missed.
It is a `design`-adjacent scope gap: the card changed a shared input (the driver spec's skill list) and missed a consumer that depends on its old value.
I filed it as `scope` because the work missed a case.
No recurring method gap is visible in this range.

The test's explanatory comments are also stale.
They say "The driver spec names a skill, so the claude engine's launch line carries no prompt pointer" (lines 47-48) and describe the skill-load answer (lines 53-55).
`integrationDriverShuttleConfig`'s comment about Start's paced typing of "the skill message and the pointer" has the same problem, as does the `stubSettleDelay` comment at line 124.

**Fix:**
Rewrite the stub for the launch-line flow.
It should take the `prompt.md` path from its own argv (the pointer argument that contains `prompt.md`) instead of from a typed line.
It should drop the skill-load read and the events.jsonl Stop append, keeping only the `/color` skip if shuttle still types it.
The test body keeps its three assertions.
Update the comments above to match: the pointer rides the launch line, and nothing but the optional color line is typed.
Re-run `lyx gate test --tags tmux ./internal/loomcli` over this test to confirm it passes.

## Reviewed with no finding

- **Range:** `product.parent` is `main` and no rework round exists, so the range is `git merge-base main HEAD..HEAD` (`abe2aa50e`).
  The merge-in commit's `^1` diff is `main`'s own changes and is not the subject.
- **Per-card mechanical checks:**
  The Delete group of card 2 holds: `internal/gatecli/child_{linux,unix,windows}.go` are gone with no remaining references.
  The Delete group of card 16 holds: `removeStatusStrands` and `TestRemoveStatusStrands` have no remaining references.
  All four Rename groups (cards 9 and 20) hold: the old test names are absent and the new ones exist in the named packages.
  `GOOS=windows go vet` is clean over `internal/proc`, `internal/gatecli`, `internal/pairteardown`, `internal/shedverbs` and `internal/shedrun`, as the `windows-bodies-cross-vetted` decision requires.
- **Constraints:**
  `PATTERN.md` is 16137 bytes, under the 16200 cap the `pattern-overview-cap` decision sets.
  No new production file spells a `_lyx` or `.lyx` literal outside `internal/lyxdirs`.
  The refusals the diff adds each have a `refusal-spec.md` row in the same range.
  The waiter's poll floor is one second and backs off to 30s, which is within `PATTERN-daemon-wakeups`.
- **Tests:**
  `lyx gate test` over `shedverbs`, `pairteardown`, `proc`, `shedengine`, `loomcli`, `stencilstore`, `shedcli`, `shedrun`, `logger`, `gatecli`, `contracts/stencils` and `cmd/lyx` passes on the untagged tier.
  I did not re-run the `integration` or `tmux` tiers; for the one failing test I relied on the Publish log and on reading the code path above.
- **Code read in full:** `shedverbs/loop.go`, `loopwait.go`, `loopenvelope.go`, `pairteardown/loop.go` and its `teardown.go` diff, `shedengine/failedstop.go`, `proc/tied.go` and `tied_linux.go`.
  I found no defect in them.
  I did not read the Windows bodies, `loopwatchdog.go`, `looppid.go`, the stencil rewrite text or the `stencilstore` drift change line by line.
