```yaml
lens: fix-way-forward
outcome: reported
```

Lens: fix-way-forward. I checked that each refusal and next step the fix added can be carried out and leads to progress.

Ways forward the fix introduced:
1. D4: the pre-run covered refusal exits with its own code, dobby reads it as covered, and the run is passed at the lift. Reachable.
2. D4: a hand pass under a covering pause keeps the refusal naming `lyx resume`. This is existing behaviour and reachable.
3. D4: the origin-owed marker keeps passes full until a full pass that reached origin exits 0. A refused `--continue-pass` while the marker stands leads to a full pass. Reachable.
4. D4: a marked run whose owner pause failed gets a full pass on each probe tick. A repeat refusal re-runs the owned-elsewhere handling, its pause included; exit 0 clears the mark. Reachable.
5. D2: `lyx adopt` of a task or beam refuses naming `lyx adopt &lt;parent&gt;` or `lyx adopt --prime`, unless the same call adopts the parent. Reachable: `adopt [&lt;slug&gt;...]` takes several slugs, and `--prime [&lt;slug&gt;...]` exists.
6. D5: the release and the start-guard refusal name `lyx board set-status '{"slug":"&lt;slug&gt;","status":null}'`. It runs as typed: the verb's help shows the `slug` key and an explicit `null`.

Findings:

- MEDIUM. D2's changed text: "The test is made against the entries the call records an owner for, so a parent the same call stops before its owner write ... does not count as adopted."
  The owner write is one batched call, so a parent lost inside it slips past the refusal and leaves a child owned here with no dobby to drive it.
  Evidence: `internal/adoptcli/adopt.go` collects every slug that passes step 1 into `adoptable`, then makes one `deps.SetOwners(... Slugs: adoptable ...)` call.
  In that call, `write.Lost` can hold the parent while the child lands in `write.Recorded`; the lost parent then gets only "`lyx adopt &lt;parent&gt;` takes it over again".
  So the refusal can only be decided against the entries the call is about to record.
  A parent lost to a race in the same write leaves the child owned here under a parent owned elsewhere, with nothing that refuses it or names that state's way forward.
  The text should either say the test is made against `adoptable` before the write and name what happens to a child whose parent is in `Lost`, or accept that race as a stated bound.

Further tool calls:
- `grep` of `internal/adoptcli` for `Use:`, `--prime` and step-1 stops: checks that one call can name several slugs and the prime, which my context did not show.
- Lines 100–200 of `internal/adoptcli/adopt.go`: checks whether the owner write happens per slug or in one batch after step 1, which my context did not show.
