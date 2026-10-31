```yaml
lens: way-forward
outcome: reported
```

Findings (way-forward lens: per refusal, stop or recovery step, does the step it names succeed from the state it leaves):

1. MEDIUM — Card 19 (D2/D11), `refuseUnservedRecipe` on a `beam` seed: the refusal sends the operator to `lym shed status|step|goto <run-id>`, but card 2 makes `lym shed` refuse that same seed as an unknown recipe. The step does not progress. Evidence: card 19 says the refusal "covers batten and a seed recording `beam`"; `internal/shedcli/table.go:80` returns `shedcli: unknown recipe %q; available recipes: %v` for any name not in the table. For a seed whose recipe names no embedded recipe, the way forward must name re-seeding (remove the run's seed and seed it again), not the `lym shed` verbs.

2. MEDIUM — Card 2 (D11), `lym shed`'s refusal of a `beam` seed: the card adds a test row that pins this refusal, but the message has no `way forward:` clause, which PATTERN-refusal-way-forward requires for a shed refusal. Evidence: `internal/shedcli/table.go:80` text above; card 2 says the row asserts "the unknown-recipe refusal naming the available recipes". Naming the available recipes is not a next step for a write-once seed.

3. MEDIUM — Card 16 (D4), `decision add` on a darn run with no record and current row empty (no status file): the way forward is `lym loom resume, then re-run the verb`, but resume refuses a run with no status file. Evidence: `internal/loomcli/resume.go:60-61` refuses `no status file at …; run "<start>" … to begin the run`. With no status the next step is `lym loom start`. Only Preflight or Record-Write should name resume.

4. MEDIUM — Card 5 (D13), the retired-key refusal of a local `loom.yaml`/`darn.yaml` and of `run_settings`: the shared `RetiredWayForward` text says "set the key under the shed module instead". Following it leaves the refused key in place, so the same refusal fires again. The text never says to remove the key from the refused local file, profile block or `run_settings` object. Evidence: card 5's text "set the key under the shed module instead: in a local file, in the pair's local shed.yaml; in a profile block or run_settings, in the shed object".

5. LOW — Card 13 (D5), Darn-Write guard halt: the way forward names `lym fabric commit` with no actor. The existing `decisionWayFabricCommit` notes that a session lym refuses the verb from reports FAILED and the orch runs it. A task-session reader of the halt reason may try it and be refused. Evidence: `internal/loomcli/decision.go:88` versus card 13's guard text.

6. LOW — Card 13 / card 15, a support log missing beside a present record: card 15 checks only the record (`RecordPath`) before spawning. The damage surfaces only in card 13's post-session guard, after a full writer session. The restore it then names works, but the session spent before it is wasted. Evidence: card 15 "`darnWrite.Call` first stats `RecordPath`"; card 13 runs the guard "once the session returns".

7. LOW — Card 15 (D4), `CommittedRecord`-true halt: "restore the decision record from the _lym weft repository's HEAD" names no command. The step can succeed (a `git checkout <head> -- <path>` is not denied per `pattern/PATTERN-refusal-way-forward.md`), but the reason gives the reader nothing to run. The same applies to card 16's matching restore texts.

8. LOW — Card 19, `resolveReviewTarget`'s refusal for `decision add` or `circling` on a batten target: the way forward names `lym shed status|step|goto`, none of which records a decision or settles a circling. For those verbs the honest way forward is that the verb does not apply to that run. Evidence: card 19's refusal text is reused "prefixed with the group and verb".

Tool calls:
- One grep of `internal/loomcli/resume.go` (missing-status refusal), `internal/configsync` (`moveKeys` layers) and `internal/shedcli/table.go` (unknown-recipe text): my context lacked resume's no-status branch, whether reconcile moves keys in the local layer (it does, so card 6's local plan keys raise no finding) and the exact `lym shed` refusal text.
