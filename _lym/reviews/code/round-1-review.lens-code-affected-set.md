```yaml
lens: code-affected-set
outcome: reported
```

List by change (set found, covered or not):

1. **Skill-load hold cap (`loadturn.go`).**
   - `internal/shuttleengine/doc.go`: updated, covered.
   - `internal/shuttleengine/engine.go:34-37`, the `OutcomeAPIError` doc, is not covered. It lists when api-error applies: the re-send limit spent, or a rejected model "a skill-load turn's included". It omits a start whose skill-load hold reached the cap.
   - `docs/overview.md:650-652` describes the api-error outcomes and also omits the cap.
   - The hold notice text (`loadturn.go:272`) is asserted nowhere else. The grep found no test on the old text, so covered.
   - The `Result.APIError` and `OutcomeAPIError` consumers that cause strings come from, such as `contracts/orchrole/orch-role.md:185`, need no change.
2. **`HistoryEntry.StartedAt`.**
   - Writers: `run.go` and the three `reroute.go` paths carry it, covered.
   - `goto.go:140` and `goto.go:263` write goto entries with no StartedAt, as intended, and `entry.go:55` writes a skipped entry the same way. `idleGapsOf` handles both through the `At` branch, covered.
   - `appendOrFold`: the fold keeps the last element's StartedAt, and its doc is updated, covered.
3. **`idle_gaps` envelope key and `idle` human line.**
   - `docs/overview.md`, `shedverbs/doc.go` and `human.go` are updated, covered.
   - `loomcli/status_test.go` key set is updated, covered.
   - `battencli/arm.go:942` comment says the generic body supplies "current_producer, state, error and activity, plus history_length", which omits `idle_gaps`. It is a comment, not a rule document, so I count it covered.
   - The `internal/shedverbs/step.go` and `drive.go` envelopes do not carry history gaps and need no change.
4. **Commit messages.** There is no `Pattern-Departure:` trailer, and `PATTERN.md` and `pattern/` have no diff.
   - The declared-changes list in the explore step is empty, so nothing is owed, covered.
   - `PATTERN-shuttle-provider-seam` does not touch this change.
   - Nothing in the diff contradicts either commit-message claim.

Findings:

**F-a — LOW, consistency**
- Location: `internal/shuttleengine/engine.go:34-36`
- Summary: the `OutcomeAPIError` doc still lists only the re-send limit and a rejected model, so a start whose skill-load hold reached `loadHoldMaxAPIErrors` is missing from it.
- Evidence: `engine.go:34-37`. The diff updated `shuttleengine/doc.go` for the same fact.

**F-b — LOW, consistency**
- Location: `docs/overview.md:650-652`
- Summary: the api-error description omits the new capped-hold path, although this diff edits `docs/overview.md` elsewhere (line 1127).
- Evidence: `docs/overview.md:650-652`.

Tool calls:
1. Grepped for `history_length` consumers, the old notice text and `last_at` docs, because the manifest lacked the other status-envelope readers and doc mentions.
2. Read `engine.go:26-42`, `battencli/arm.go:938-948` and grepped api-error docs, because the manifest lacked the `OutcomeAPIError` doc, battencli's envelope comment and the doc mentions of api-error.
