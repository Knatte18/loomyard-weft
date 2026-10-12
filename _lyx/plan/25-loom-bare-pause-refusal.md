# Card 25 — loom-bare-pause-refusal

**Create:**
- `plan:internal/loomcli#loomCLI.loomBarePause` -> `func (c *loomCLI) loomBarePause(st shedengine.Status) error`
- `plan:internal/loomcli#TestPauseCmd_BareLoomPauseRefusesNamingBothWaysForward` -> `func TestPauseCmd_BareLoomPauseRefusesNamingBothWaysForward(t *testing.T)`

**Edit:**
- `internal/shedverbs#Hooks`
- `internal/shedverbs#pauseCmd`
- `internal/shedverbs#TestPauseCmd_Conditions`
- `internal/loomcli#loomCLI.specFor`
- `internal/loomcli#loomVerbTexts`
- `internal/loomcli#loomCLI.loomPreStepUnmarked`
- `internal/loomcli#TestStepCmd_BusyRefusal_BeforeBootstrap`
- `internal/shedcli#shedVerbTexts`
- `internal/shedcli#Command`
- `internal/shedcli#parityLoomPauseSeeded`
- `contracts/specs/refusal-spec.md`
- `contracts/specs/loom-status-spec.md`
- `contracts/stencils/orch/orch-template-role.md`
- `tools/sandbox/SANDBOX-CORE-SUITE.md`
- `docs/skills.md`
- `docs/overview.md`
- `cmd/lyx/testdata/test-budget.yaml`

**Intent:**
Retire the bare boundary pause for a loom run (D8): `lyx loom pause` with no flag, and `lyx shed pause` against a loom run, refuse with both ways forward, while `--before`, `--after` and `--clear` work as before and `lyx batten pause` keeps its bare request.

- `Hooks` gains `BarePause func(st shedengine.Status) error`; `pauseCmd`, after its absent-file check, returns the hook's error for a flagless pause when the hook is set and writes nothing, and sets `pause_requested` as today when it is nil. A status file carrying `pause_requested` still loads, and batten, which sets no hook, is unchanged.
- loom's `specFor` sets `Hooks.BarePause` to `loomBarePause`, which refuses with `loom: a bare pause no longer stops a loom run; way forward: run "lyx pause <slug>" to stop its agents now, or "lyx loom pause --after <current_producer>" to halt it at its next producer boundary`, filling the pair's slug, the status's current producer and the run-id when it is not `self`.
  `lyx shed pause` against a loom run arms loom's own spec, so it refuses the same way and the shed parity check holds.
- `loomPreStepUnmarked`'s busy refusal names `lyx loom pause --after <current_producer>` in place of the bare pause.
- `loomVerbTexts`' and `shedVerbTexts`' pause help say a bare pause refuses for a loom run, naming `lyx pause` and `--after`, and drop the bare examples; `shedcli`'s group `Long` example uses `--after`.

Reachability: a seeded loom run with a status file reaches the refusal after the absent-file check; the absent-file refusal still comes first, so the never-bootstrapped pause of `TestLoomStatusAndPauseOnNeverBootstrappedPair` keeps naming `lyx loom start`.

Docs and records: `refusal-spec.md`'s loom section gains `bare pause on a loom run` (correctness halt; way forward: `lyx pause <slug>` to stop the agents, or `lyx loom pause --after <current_producer>` to halt at the next boundary); `loom-status-spec.md` names `--before`/`--after` as the loom pause verb's forms; the orch role stencil's "`lyx loom pause` while Plan-Write runs" becomes `lyx loom pause --after Plan-Write`, a boundary halt a goto can move; S8 of the core sandbox suite checks that a bare `lyx loom pause` refuses naming both ways forward and that `--after <row>` records `pause_after`; `docs/skills.md`'s `mill:mill-pause` row names `lyx pause <slug>`; `docs/overview.md`'s loom bullet says `pause` records a `--before` or `--after` condition and refuses bare, naming `lyx pause` for stopping the agents.

Tests: `TestPauseCmd_Conditions` gains a `barePause` hook field and two rows: a flagless pause under a hook refuses with the hook's error and leaves `pause_requested` false, and `--after` under the same hook records the condition.
`TestPauseCmd_BareLoomPauseRefusesNamingBothWaysForward` runs loom's pause verb on a hand-populated receiver over a running status at `Plan-Write` and asserts the refusal names `lyx pause <slug>` and `lyx loom pause --after Plan-Write`, with the file unchanged.
`TestStepCmd_BusyRefusal_BeforeBootstrap` asserts the busy remedy names `lyx loom pause --after`; `parityLoomPauseSeeded` drives `pause --after Discussion-Write` through both verbs, comparing their success envelopes as before.
The loomcli budget row grows by one.

**ImpactSummary:** One optional shedverbs hook that loom fills; loom's and shed's pause help, loom's busy text and four records change, and batten's pause is untouched.
