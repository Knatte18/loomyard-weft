# Round 2 fixer-report — plan

## Changes

- **F1** (BLOCKING), fixed as proposed: card 8 lists `cmd/lym#testOnlyCodeAllowed` under `**Edit:**`.
  Its Intent deletes the `internal/seatengine.Prompts` entry, which `shuttlecli` now reaches through `shuttleparity.Report`, `seatsOf` and `seatengine.Specs`.
  Premise checked: the entry sits at `cmd/lym/testonlycode_linux_test.go:56`.
- **F2** (BLOCKING), fixed as proposed: card 6's `(*Engine).SeatTables` builds the fix table as `ProbeFix` does, with `fixSeatValues(p, fixFilesIn(e.probeDirOf()))` (`internal/burlerengine/fix.go:254`).
  The card says why: the fixer stencil's markers come from the seat's values, and `Prompts` refuses an unfilled marker.
  `fixSeatValues`, `fixFilesIn` and `Engine.probeDirOf` were added to card 6's `**Uses:**`.
- **F3** (MEDIUM), fixed as proposed:
  - Card 6's fixture writes a hub `language.yaml` layer listing a `<plugin>:<name>` skill.
  - `PrepareEach` expands `Skills` with `cfg.Agents.Skills` for a spec with a `ConfigRole`, as `Runner.start` does at `internal/shuttleengine/run.go:576`, and the parity test collects names after that expansion.
  - `addSkills` writes the Codex plugin-cache layout `skillPath` resolves, and a name without a colon is a fixture error.
  - Card 7's producer test collects names the same way, and its stand-in `LanguageSkills` are the fixture's list.
- **F4** (MEDIUM), fixed as proposed: in card 8, an engine that cannot start the seat is named with its `Unmet` requirements on a line of the `output.Ok` report, and the exit stays zero.
  A missing compared item is the only failing result.
  The `TestManifestCmd` row now expects `ok` and exit zero, and the overview's every-seat decision says the same.
- **F5** (LOW), fixed as proposed: card 6 and the overview gate the fork deny on the set's `codex` engine's `RequirementSupporter` answer for `RequirementForks`, not on `Unmet` or a spec.
  Both state that Claude's manifest holds the item only for a `ForkSubagents` spec.
- **F6** (LOW), fixed as proposed: card 2's two warnings now cover every cause.
  The copy write's way forward also covers making `<root>/manifests` a writable directory with free space.
  The unreadable-record way forward separates a file that does not decode from one that cannot be read.

`lym loom validate-plan` answers `ok: true`, with informational findings only.

## Self-check

- fix-consistency, LOW: card 8's `ImpactSummary` omitted the allowlist edit. Fixed as proposed.
- fix-consistency, LOW: `TestCompare` had no row for `PrepareEach`'s skill expansion. Merged with the fix-affected-set LOW and the fix-proposal MEDIUM below.
- fix-affected-set, LOW: the same `PrepareEach` expansion row, plus the unstated expansion error. Merged.
- fix-proposal, MEDIUM: the expansion's error path, and where the parity test's `cfg` comes from, were unstated. Fixed as proposed:
  - `PrepareEach` returns the expansion error naming the role.
  - `TestCompare` gains rows for a `ConfigRole` spec reaching `Prepare` with the role's skills, and for a role with no list being refused.
  - The parity test loads `cfg` with `shuttleengine.LoadConfig`, so `cfg.Agents` holds every seat role.
- fix-proposal, LOW: the Claude half of `addSkills` named a flat config-dir layout that `claudeengine`'s `skillListing` lists without a `<plugin>:` prefix (`internal/shuttleengine/claudeengine/usersetup.go:280-283`). Fixed with my own change, since the lens proposed none: Claude's copy goes under an installed fixture plugin's `skills/<name>/SKILL.md`, which `skillListing` lists as `<plugin>:<name>`.
- fix-claims: no findings.

## Disputed

None.

## Deferred

None.

## Rulings

None.
