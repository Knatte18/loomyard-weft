# Fixer report — plan, round 1

## Changes

- **F1** (fixed as proposed): card 8's Intent states `fanShapeError.Error()`'s text with its way forward, `set the key to a fan name, or to a list of fan names, with "" for a solo or single round`.
  Its refusal-spec paragraph adds the `config: fan key not a fan list` row over the four fan keys, and the mapping-valued `discussion_fix_fan` row of `TestLoadConfig_Refuses` now asserts the way forward.
- **F2** (fixed as proposed): card 7's `bouncerEntry` paragraph passes `RowClusterFans` as `BouncerConfig.ClusterFans` only when the list holds a non-empty entry, so a recipe-fan row keeps the told `cluster_excludes` bool.
  `TestBouncerEntry_ClusterExcludes` gains a subtest with `cluster_excludes: true` beside a `[""]` entry that still asks for `exclude_lenses`.
  Card 6's `ClusterFans` sentence names the partner review row's effective per-round list, never the raw wiring value.
- **F3** (fixed as proposed, shape (a)): card 3's `fixTable` leaves the fixer seat's `ModelKey` zero in every round, so `opts.FixKey` is read only by `twillEngineRefusal` and a single fixer's message is unchanged.
  `<refusal>` is the start error cut at its last `; way forward: `, which drops the clause the zero key's `ModelKey.Wrap` appends, so the message carries exactly one `way forward:`, D7's step.
  Shape (a) was taken over (b) because D7 asks for "move that fixer key to an engine with forks and resume", and `Wrap`'s keyed clause says "re-run the verb, or under the role's row re-step the row".
  The `TestEngine_Fix_Outcomes` start-error rows run through `seatengine.RunnerShuttle` over a real `shuttleengine.Runner` with a `shuttlefake` engine that supports no requirement, the setup `internal/seatengine`'s `realRunnerShuttle` uses, and assert `way forward:` appears once.
  The twill row of `TestEngine_Fix_TableShapeAndFiles` asserts the seat's zero model key, and `internal/seatengine#RunnerShuttle` joins card 3's Uses.
  The refusal-spec row wording names the two steps as the row's only way forward.
- **F4** (fixed as proposed): card 5's `BurlerFixProducer.Call` paragraph reads the names before the probe, sets a recorded `FixFan` before the probe, and defers the archive and refusal for missing names until the probe finds no live fixer.
  A live fixer over a nameless record is probed and resumed as the single fixer.
  `TestBurlerFixProducer_Call_RefusesARoundWithNoAcceptedReview` gains the flagless-record-with-live-fixer row, which resumes and archives nothing.

## Disputed

- **F5**, premise false.
  The finding's stale state needs a gate-failed twill fix to have written `fix_lenses` and `fix_forks`.
  `fixedExit` (`internal/shedadapters/burlerfix.go:229-237`) archives the round and returns on a failed gate before any `mergeRoundUsage` call.
  Its fix-half merge (`burlerfix.go:241-245`) runs only past that return, and card 5 puts the two new keys in that merge ("beside the fix half").
  A gate-failed fix therefore writes neither key.
  A passed fix completes the round, so `highestCompleteRound` moves both producers to round N+1, and no later single fixer runs over round N's record.

## Deferred

None.
