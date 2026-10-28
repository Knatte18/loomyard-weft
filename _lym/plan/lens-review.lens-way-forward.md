```yaml
lens: way-forward
outcome: reported
```

Task: review the plan through the way-forward lens, covering refusals, stops and recovery steps and whether each named next step can succeed.

1. MEDIUM — Card 1 (D5 bound). Card 1 makes a new refusal reachable: a pair's local `code_reviewer_fan` naming a fan `lenses.yaml` lacks. No planned row reaches it. D5's bound relies on that refusal naming "the key and the fix". The only planned fan-name refusal row is the existing base-file one, "unknown code_reviewer_fan" in `internal/loomengine#TestLoadConfig_Refuses` (config_test.go:610). Card 1 adds only an admitted local row, `code_reviewer_fan: ""` in `TestModules_LocalLayerRefusals`. Nothing pins that a local `code_reviewer_fan: nosuchfan` fails `LoadConfig` through `ValidateFanKeys` with the way forward. The next step, `set the entry to ""`, can succeed from that state, because the local file is open after card 1 and the agent can edit it. It is simply unpinned. Proposed fix: add a row to `TestLoadConfig_Refuses`' local-fan subtest that writes `code_reviewer_fan: nosuchfan` to the local file and wants `loom config key "code_reviewer_fan" entry 1` and `set the entry to ""`.

2. LOW — Card 1 (D5 way forward). After card 1, `code_reviewer_fan` can come from the pair's local layer or a task's `run_settings` (via Seed-Child). The `ValidateFanKeys` refusal, `define the fan in lenses.yaml, or set the entry to ""` (internal/lenses/fankeys.go, the `ValidateFanKeys` body), names neither the layer nor the file holding the bad value.
   - An operator who reads "set the entry" may edit the hub's `loom.yaml` or the task's `run_settings`. Neither reaches the running pair, since D5 says `run_settings` reach a pair only at Seed-Child. So the first fix tried may not clear the stop.
   - The other branch, "define the fan in lenses.yaml", cannot be done from the pair's local layer, because `lenses.yaml` is closed to it (internal/lenses/doc.go, Guard keys section).
   - Card 1's Intent calls the refusal one that "names the key and the fix" without noting this. A clause in the Intent, or in the refusal, naming the pair's local file (`configengine.LocalFile`) as the place to set the entry would make the next step land.

3. LOW — Cards 3 and 5 (open risk, recovery). A hub whose `lenses.yaml` fan still names `goal-scope`, `board-fit` or `code-pattern-fit` gets the `ResolveFan` undefined-lens refusal, wrapped by `ValidateFanKeys` with `define the fan in lenses.yaml, or set the entry to ""`.
   - The fan is defined; it is a lens it names that is missing. The named next step either misdirects or drops the whole fan, when the right step is to remove the lens name from the fan.
   - `internal/lenses#TestResolveFan`'s row "a fan naming an undefined lens" asserts no way forward.
   - Neither card names the operator's real recovery step: drop the name with `lym config lenses --set` or the editor. That step lives only in the record's Open risks.
   - The refusal predates this plan and changing it is out of the record's scope, so the finding is that cards 3 and 5 leave the recovery path to an error text that does not name it. At most, the Intents can note this.

Per refusal:
- Local wrong `code_reviewer_fan` name: next step is setting the entry to `""` in the local file. It progresses but no row pins it (1), and the refusal does not name the file (2).
- Hub fan naming a deleted lens: next step is "define the fan or set entry to `""`". It misdirects; the real step is removing the lens name (3).
- Hand-edited local `webster_fan` or `webster_reviewer_fan`: card 1 leaves this as it is; the next step is a reconcile, which progresses.
- `TestProfilesCheckLoad` failing without card 1: the ordering in the overview's Shared Decisions prevents it. Not a runtime refusal.

Further tool calls: none.
