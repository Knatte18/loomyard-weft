```yaml
lens: way-forward
outcome: reported
```

Way-forward lens review of the plan in `_lym/plan/` (overview plus cards 1–10): for each refusal, stop or recovery step a card adds, does the next step it names work from the state it leaves.

**Findings**

1. MEDIUM — Card 8 (D10 verb) and Card 6 (`Seats`): a failure in any one table builder blocks the verb for every seat, and no next step is named.
   - The verb calls `Seats(env)` and only then `LookupSeat`.
   - Card 6 says `Seats` "builds every table", with stand-in inputs and "paths under `env.Dir`".
   - At least two builders need files on disk:
     - `loomengine#PartsPlanTable` reads the parts file (`readPartsFile`).
     - `landingshed#DescribeTable` reads its records.
   - Card 6 never says that `Build` writes those stand-in files. Card 8 never says what `env.Dir` is for the verb.
   - `Report` makes its temporary directory after `Seats` has run. So any stand-in files `Seats` writes either:
     - land outside the removed directory, breaking D10's bound "writes nothing outside that temporary directory but the Codex home"; or
     - are not written, so the builder errors.
   - The same failure hits a worktree whose stencils directory is missing.
   - In both cases the verb prints a builder error with no way forward, for any seat argument, and an operator cannot get past it.
   - Card 8's `TestManifestCmd_RefusesAMissingOrUnknownSeat` runs in a fixture worktree. It reaches the unknown-seat refusal only if `Seats` succeeds there, and no card makes that happen.
   - Evidence: Card 6 Intent (catalog paragraph, "`Seats(env)` builds every table"); Card 8 Intent (verb paragraph, order `Seats` → `LookupSeat` → `Report`; `Report` "makes a temporary directory"); the builder inventory (`loomengine/partsplan.go:60` needs the parts file on disk; `landingshed/describe.go:119` reads records).

2. MEDIUM — Card 8 (D10, "exits non-zero naming a missing item"): the refusal names the missing items but not what to do about them.
   - The message prints `<channel> <key>` with `output.Err` and stops there.
   - The existing parity-style refusals carry a "way forward" clause. Examples are `CheckPromptEngine` and `TestPromptEngineRule`'s RealModule message.
   - This exit is the gate step in `codex-writer-bench` ("check with `lym shuttle manifest` …"). A bench operator who sees it is not told:
     - that the gap is a missing Codex delivery in `codexengine` that needs a code fix; and
     - that the bench must not run until that fix is in.
   - Evidence: Card 8 Intent ("a non-empty one prints `output.Err` naming each missing item as `<channel> <key>`"); `internal/shuttleengine/promptengine.go` (CheckPromptEngine's way-forward form).

3. LOW — Cards 6 and 7 (D1 test and scan failures): the failure messages name no next step.
   - `TestSeats_CodexManifestHoldsClaudes` and `TestProducers_CodexManifestHoldsClaudes` "fail naming the seat/function and each item".
   - The `RealModule` row of `TestLaunchSpecLiterals_AreListed` "fails naming each unlisted literal". It should point the reader to:
     - add an entry to `producers` or to `seatTables`; or
     - set no `Prompt` when the spec is a probe.
   - The existing `TestPromptEngineRule` RealModule message shows the expected pattern ("…; way forward: …").
   - Evidence: Card 6 Tests bullet 3; Card 7 Tests bullet 2; `internal/shuttleengine/promptengine_enforcement_test.go` (RealModule `t.Errorf` carries a way forward).

4. LOW — Card 6 (`PrepareEach`): an engine that writes no record is "an error naming the engine", with no next step.
   - This state is reachable when an engine is registered without manifest support, for example a future engine, or a fake in `TestReport`.
   - Neither the verb nor the test tells the reader that the engine's `Prepare` must write a record through `shuttleengine.WriteManifest`.
   - Evidence: Card 6 Intent (`PrepareEach`: "a `Prepare` error or an absent record is an error naming the engine").

5. LOW — Cards 3 and 4: a failed manifest write "refuses the launch with the path" and names no next step.
   - A raw I/O error is exempt as transient, so this is a note only.
   - What the refusal leaves behind is sound: `Runner.launch` removes the run directory after a `Prepare` error (`internal/shuttleengine/run.go:649-652`), so retrying the launch can succeed.
   - Evidence: Card 3 Intent ("A write failure of the manifest refuses the launch with the path"); Card 4 Intent (the same).

6. LOW — Card 6/7 skip versus Card 8: the skip rule is not shared with the verb.
   - The tests skip a seat that requires forks "since `Start` refuses it on Codex".
   - Card 8's `Report` has no matching rule. For such a seat the verb would print a Codex manifest and a pass verdict for a seat whose Codex launch `Start` refuses, and give no sign that it cannot run on Codex.
   - Evidence: Card 6 Tests bullet 3 (skip clause); Card 8 Intent (`Report` runs `PrepareEach` for each engine with no requirement check).

**Per refusal: the next step named, and whether it gets past the refusal**

| Card | Refusal or step | Next step named | Gets past it? |
|---|---|---|---|
| 2 | `copyManifest` failure is logged at Warn | the launch proceeds | yes |
| 2 | sweep and lookup skip `manifests` | "nothing removes the copies" (bound stated) | yes |
| 3, 4 | manifest write failure | none (I/O, exempt) | the launch is retryable |
| 6 | `PrepareEach` error or absent record | none | no (finding 4) |
| 6, 7 | parity and scan test failures | none | no (finding 3) |
| 8 | unknown seat | way forward names the catalog's seats | only once `Seats` succeeds (finding 1) |
| 8 | missing items | none | no (finding 2) |
| 8 | `Report` with a failing `Prepare` | the engine's own error, temp dir removed | yes, through the engine's own way forward |
| 10 | board upsert | not a code refusal | n/a |

No further tool calls were made.
