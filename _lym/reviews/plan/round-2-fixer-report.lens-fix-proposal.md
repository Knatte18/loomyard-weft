```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge each review proposal the fixer carried out (fix-proposal lens).

**F5 (card 3, `Verdict.Refusal`, terminal `Landed` clause):**

1. MEDIUM. For a landed beam, the new wording names a step that cannot be carried out.
   - Changed text: "then releasing the entry with `lym board set-status '{"slug":"<slug>","status":null}'`".
   - Case: a beam is a project, and its landing sets its status to `done`.
   - Evidence: `internal/boardengine/projectrelease.go:16`, together with `setStatusReleasingProject`, refuses a null status on a project whose status is `done` ("A null status on a project whose status is a run status or done is refused").
   - So `lym batten run <beam>` on a landed beam gets a way forward that the board refuses.
   - The clause should either limit the release to a task, or name the beam's own path.

2. MEDIUM. When a batten record still stands, following the new wording leads straight into another refusal.
   - Changed text: "removing its pair … releasing the entry … deleting the leftover task branch … as the done-slug refusal of `lym batten run` words it".
   - Case: `Landed` from batten found and `done` (`Derive` rule 1, first arm), reached through a verb other than card 10's `lym batten run`, such as `lym batten step` or `lym loom resume`.
   - The text never names removing the run directory. The re-run therefore finds batten's `done` record and hits `doneSlugRefusal`.
   - Evidence: `internal/battencli/arm.go:848-852`. `doneSlugRefusal` names `lym batten cleanup --apply` (to remove the run directory), deleting the task branch, or using another slug. It names neither `lym fabric remove` nor a board release.
   - So the claim "as the done-slug refusal of `lym batten run` words it" is false too: the two texts name different steps.

**F1, F2, F3, F4:** no case found against the proposals.
- F1: keeping `CancelMarker` in both gates restores today's behaviour in the lock window.
- F2: rule 4 alone reads `PastParentMerge`, and only under `Held`, which is false in this part. So the claim that false "only withholds the act a hold would otherwise yield" holds.
- F3: the internal `TestReadRun` needs no `hubforge`.
- F4: the glyphs cover the const block.

Tool calls:
- grep of `internal/boardengine` for done/status rules: to learn whether a `done` entry can be released, which my context lacked.
- read of `internal/boardengine/projectrelease.go`: for the null-status refusal on a done project, which my context lacked.
