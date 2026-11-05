```yaml
lens: walkthrough
outcome: reported
```

Walkthrough lens: I ran each Decision's mechanism end to end on one concrete case and checked the end state against the Decision's stated purpose.

**Decisions walked**

- D1, the `cs` line `Re = 127069, dP = 7925.267 Pa` with `dP` moving: one `changed` cell, identity `Re`, whose value holds the whole rest of the line. Purpose met. D8's rationale contradicts this (F5).
- D2, a `cs`-style hub under `diff: command` whose capture writes no file per case: Drift-Expect still validates against D1 identities and baseline case files, which this capture doesn't have. Purpose not met (F3).
- D3, `run_settings` `{scope: partial, skip: [a]}`: `skip` is a retired `CategoryNone` key, so `ValidateLocal` admits it (`internal/configengine/layers.go:291-299`) and the load drops it, and `partial` is recorded as `full`. Met. For an in-flight run whose record holds `partial`, D3 reads `ScopeRecord.Skip`, which D14 does not keep (F4).
- D4, a darn run where the perch makes no code change: Drift-Recheck finds the head of the last passing check and is Done at once. With an amendment recorded since, it runs the full pass. Met.
- D5, the expect agent wants to cover `dP` but D1 names the cell `Re`: the parser refuses `dP` unless it is marked `added: true`. If it is marked, the entry covers nothing and the cell is unexpected, which stops the row. Met: the result is a strict failure, not a leak.
- D6, Drift-Expect re-entered after its agent wrote a valid file: the decision rests on whether "the expect session ended normally", read through `AgentSessionOpen`. That mark lives in `runs.json`, which D14 removes. Purpose not met (F1).
- D7, a Webster fork's `lym drift check` running beside a `drift` entry inside Merriam's run-level gate: the existing capture lock refuses rather than waits. Make it blocking and the two take lock and slot in opposite orders. Purpose not met (F2).
- D8, a Darn-Write entry finding one unexpected cell: the writer fixes or exhausts the entry, then the row is Stuck and amend is admitted. Met.
- D9, three expected cells judged at Drift-Verify, then two new cells at Drift-Recheck: the second judge's `judgement` entry can be met by the file the first spawn left. Purpose not met (F6).
- D10, a cell reproduced at Drift-Recheck: it reuses the stored ruling. Met.
- D11, Stuck on `(X, fluid, water, oil)` followed by `amend` with key `fluid`: the entry validates, the cell becomes expected, the judge rules and the check passes. Met.
- D12, a counted capture whose `.lym` was cleaned: Drift-Expect is Stuck, `defer` runs, and every drift row is then Done with the deferral stated. Met.
- D13, a verdict with an open range: the range's `why` and the uncovered entries come from the lists D4 records in `scope.json`. Met.
- D14, removing the `drift-verify` gate key: a hub `gate.yaml` that holds it is not shown to be pruned (F7).
- D15, `cs` with `compare` set, `diff` unset and `{skip_file}` in its capture: flagged once, capture refused until the operator edits it, then `lines` mode. Met.
- D16, the stencils: met, since they are a review check.

**Findings**

F1 — MEDIUM, consistency, D6 against D14. D6's re-entry rule has no record to read once D14 lands. D6 seals an unsealed file "only when the expect session ended normally, recorded as today's drift producers record an open agent session (`AgentSessionOpen`)". That mark is one per-run flag in `runs.json`, and only Drift-Verify's `ResetRuns` and `CloseAgentSession` write it. D14 removes "the run budget and `runs.json`". So the expect seat needs a session mark of its own, kept apart from the judge's, and the record should say where it lives.
Evidence: `internal/driftengine/records.go:144-151` and `records.go:247-281`; `internal/driftengine/verify.go:565` and `verify.go:570`; D14 at `_lym/discussion/decision-record.md:278`.

F2 — MEDIUM, design, D7 and D8. The capture lock refuses rather than serialising, and a blocking version would deadlock.
- D7 runs every current capture "under the capture lock", and its rationale says "The lock serialises concurrent checks, such as a Webster fork's `lym drift check` beside Merriam's gate".
- Today `RunCapture` takes that lock through `AcquireWriteLockWithin` with a bounded retry, then refuses with `lockHeldRefusal`. A second check therefore fails instead of waiting.
- Make it blocking and the orders invert. A check inside Merriam's gate holds its slot, then waits for the lock. A fork's check holds the lock, then waits, with no deadline, for a slot from the pool. With no free slot, neither moves.
- The Decision should name one lock order or say how the second check waits.

Evidence: `internal/driftengine/capture.go:54-56` and `capture.go:70-75`; the driftengine package doc ("waits for one gate slot as a non-tier holder with no deadline"); D7 at `_lym/discussion/decision-record.md:181` and `:184`.

F3 — MEDIUM, design, D2 with D5 and D11. Under `command` mode the expectation is validated against identities the capture doesn't produce.
- `command` mode is for a capture that cannot meet D1's contract. Yet Drift-Expect still hands the agent D1's identity listing.
- D5 refuses a case missing from the baseline and a key D1 does not read unless it is marked `added: true`. D11 amendments go through the same rules.
- A capture that is not one file per case, or not line-keyable, cannot name the cases the judge reads in the compare's log.
- The record doesn't say what the listing, the validation or an amendment's shape is in `command` mode.

Evidence: D2 at `_lym/discussion/decision-record.md:64-72`; D5 at `:122` and `:132`; D11 at `:234`.

F4 — LOW, consistency, D3 and D13 against D14. D3 and D13 need the recorded skip list that D14 doesn't keep.
- For a scope record holding `partial`, D3 leaves out "the cases its recorded skip list names", and D13 renders those cases.
- D14 removes "`partial`, `skip`". It keeps read-only only "the old compare, acceptance and waiver fields" of `scope.json`.
- `ScopeRecord.Skip` is in neither list as something to keep. D14 should name `scope` and `skip` among the fields kept read-only.

Evidence: `internal/driftengine/records.go:71-74`; `_lym/discussion/decision-record.md:88`, `:265` and `:278-280`.

F5 — LOW, consistency, D8's rationale. D8's example names a key D1 never produces. It says the writer meets "key `dP` of case X moved". Under D1's own default patterns the `cs` line keys as `Re`, so code never reports a key `dP`. The example should name `Re`.
Evidence: `_lym/discussion/decision-record.md:58` and `:199`.

F6 — MEDIUM, design, D9. A later judge spawn can pass its gate on the file an earlier spawn left.
- `judgement.yaml` stays under `_lym/drift/` between checks, and the judge's tree guard exempts that file.
- D9 doesn't say Drift-Verify removes it before a spawn, or that cell ids are unique across spawns.
- Drift-Recheck's judge can therefore meet the `judgement` entry with the Drift-Verify judge's file when the ids coincide, for example ordinal ids over the same count of cells. That would store rulings no judge made for these cells.
- The same gap covers a judge that died mid-write: the record says nothing about a partial or stale file at re-entry.

Evidence: D9 at `_lym/discussion/decision-record.md:204-212`.

F7 — LOW, design, D14. A hub's `drift-verify` gate entry may outlive its removal. D14 says `lym config reconcile` prunes a hub `gate.yaml` entry for `drift-verify`. But `gates` is a per-name map, whose names belong to the operator, and reconcile's `Prune` is passed `PerNameMaps`. That suggests an operator-set key there is kept, not pruned as unknown, so the claim is unproven. The entry would remain as a key nothing reads. The record should settle whether reconcile prunes it or the entry stays as harmless.
Evidence: `internal/gateslot/config.go:100-101`; `internal/gateslot/template.yaml:30`; `internal/configsync/configsync.go:688` and `:696`; D14 at `_lym/discussion/decision-record.md:278`.

**Tool calls**

1. A grep of `internal/gateslot` for the `drift-verify` key and `ConfigPerNameMaps`, and of `capture.go` for how the capture lock is taken. My context lacked whether `gates` is a per-name map and how the existing lock behaves.
2. A grep of `internal/configsync` for how per-name maps are pruned, and of `capture.go` for the lock's refusal. My context lacked how reconcile prunes a per-name map and the refusal text.
