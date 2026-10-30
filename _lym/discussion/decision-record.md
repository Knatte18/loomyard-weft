# Decision record — dobby posts the can-land notice only once the beam is flagged ready

## Goal

A beam's dobby no longer reads an empty new beam as a finished one.
The beam carries a ready flag that its orch or the operator sets with `lym beam ready <slug>`;
the can-land notice, `lym beam land` and the landing's re-check at Beam-Await all wait for it.

## Scope

- New verb `lym beam ready <slug>`, which writes and commits the beam's ready record.
- The ready record: a JSON record in the beam worktree's `_lym/landing/` directory, beside the change description, its lock at the mirrored `.lym/landing/` path.
- `lym beam land` refuses a beam whose ready record is absent.
- Beam-Await's landing re-check refuses the same, so a land request written past the verb cannot land an unflagged beam.
- The dobby's can-land reading passes only for a flagged beam.
- Docs: the package docs of `internal/beamcli`, `internal/dobbyengine` and `internal/loomcli` where they describe the land refusals, the can-land reading and the re-check, and `docs/overview.md` where it lists the beam verbs.

Out of scope: a verb that clears the flag, and dobby landing the beam itself; see D6.

## Decisions

### D1 — The ready record lives in the beam's `_lym/landing/` directory and names its beam

The record is `beam-ready.json` in `loomengine.LandingDir` of the beam's pair, holding the beam's `slug` and the `set_at` time.
Its lock is the mirrored `.lym/landing/beam-ready.json.lock`.
`internal/loomengine` declares both paths, as it declares the landing directory.
`internal/pawlengine` owns the record type and its write and read, through `internal/state`.

Rationale: the operator placed the flag in the beam's `_lym`, beside the landing's other information.
`_lym` is tracked, so the flag survives a machine change, and weft content never merges into the parent, so a landed beam does not mark its parent ready.
A pair forked from a flagged beam inherits the beam's weft content, the record included;
a record whose `slug` is not the reading beam's own reads as no flag, which bounds that inheritance.

### D2 — `lym beam ready <slug>` sets the flag and commits it

The verb runs anywhere in the hub, as `lym beam land` does.
It refuses what `lym beam land`'s entry check refuses: an entry that is not an open beam project with a marked pair on this machine.
It writes the record and commits the landing directory through `fabricengine.CommitAnchoredPaths`, as the beam's Describe row commits it.
A beam already flagged for its own slug is reported as already ready, and nothing is written.
Dobby never runs it.

Bound: the verb is a new edge that lets a beam land.
It lets nothing land by itself: the landing still needs `lym beam land`, and every other land check stands.

### D3 — `lym beam land` refuses an unflagged beam

The refusal comes after the owner checks and before the work-remaining refusal, and names `lym beam ready <slug>` then `lym beam land <slug>` as the way forward.

Rationale: the operator asked for it.
Placing it after the run and owner checks keeps their ways forward first, since those are fixed before the flag matters.

### D4 — Beam-Await's re-check refuses an unflagged beam

The landing re-check that Beam-Await runs before anything lands adds one line for a beam with no ready record of its own slug, naming `lym beam ready <slug>`.

Rationale: the re-check re-runs every check of `lym beam land` but the run-at-Beam-Await one; a land request is only a file, and the flag must hold whatever wrote it.

### D5 — The can-land reading passes only for a flagged beam

`beamLandReading` in `internal/beamcli` reads the ready record first, and an unflagged beam reads as not passing, whatever works under it.
`dobbyengine.LandReading.Passes` then means: the beam is flagged ready and nothing works under it.
The notice episode logic in `internal/dobbyengine` is unchanged.

Rationale: the dobby's notice waits for the flag, and the reading is the one seam that already says whether the beam can land.

### D6 — Dobby gates the notice; it does not land the beam itself

The entry says both that dobby starts the landing once the flag is set and that dobby waits with the notice until it is set.
This run builds the notice gate, option (A), the reading its title states.
The parent took the choice between (A) and dobby writing the land request itself, (B), to the operator and asked for the parts both share to be built meanwhile; an `Added after Discussion` entry that picks (B) overrides this decision.

## Constraints

- `PATTERN-durable-vs-ephemeral-state`: the record is tracked under `_lym`, its lock under `.lym` at the same subpath, both named through `internal/lymdirs` in `internal/loomengine`.
- `PATTERN-tolerant-records`: the record is written and read through `internal/state`.
- `PATTERN-fabric-git`: the commit goes through `internal/fabricengine`.
- `PATTERN-refusal-way-forward`: each new refusal names its way forward, and a test reaches it.
- `PATTERN-cli-cobra`: the verb is a cobra subcommand of the beam command.

## Auto-mode assumptions

- A darn run holds no interview; the parent's answer on D6 is still open.
- No verb clears the flag: a beam flagged too early lands only once nothing works under it, which the work-remaining refusal still guards.

## Open risks

- The parent's answer may pick (B), which adds dobby writing the land request on top of this change.
- A beam made before this change has no record, so its landing needs `lym beam ready` first.

## Acceptance criteria

- D1: a written record reads back as flagged for its own slug, an absent record reads as not flagged, and a record naming another slug reads as not flagged.
- D2: `lym beam ready` on a landable beam writes the record and commits it; on a beam already flagged it writes nothing and says so; on an entry that is not a landable beam it refuses with the entry check's way forward.
- D3: `lym beam land` of an unflagged beam refuses naming `lym beam ready <slug>`, writes no land request, and a flagged beam lands as before.
- D4: the re-check lines of an unflagged beam name `lym beam ready <slug>`, and a flagged beam with nothing else wrong yields no lines.
- D5: the can-land reading of an unflagged beam with nothing under it does not pass, so the dobby posts no notice; a flagged beam with nothing under it passes.
- D6: dobby never writes a land request, and the can-land notice text is unchanged.
