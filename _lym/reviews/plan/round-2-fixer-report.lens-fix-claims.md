```yaml
lens: fix-claims
outcome: reported
```

I checked the fix's claims against the code at HEAD.

**[LOW] Card 3, the new `Verdict.Refusal` terminal `Landed` clause**
- **Summary:** "as the done-slug refusal of `lym batten run` words it" says the refusal words something it does not.
- **Evidence:** `doneSlugRefusal` (`internal/battencli/arm.go:848-853`) names `lym batten cleanup --apply` to remove the run directory, plus deleting the task branch locally and on the remote, or another slug. It names neither `lym fabric remove <slug>` nor `lym board set-status ... null`. So the two texts share only the branch deletion and the other slug.
- **Second gap:** the clause also covers a `Landed` run whose batten record stands and is `done`. For that run, removing the pair and releasing the entry leaves the done record in place. A re-run would derive `Landed` again, because `Derive` rule 1 is "batten found and `done`". The clause names no removal of the run directory, which is the `lym batten cleanup --apply` step the done-slug refusal carries.

These claims check out:
- **Card 5, both seams need the task worktree:** `FinalizePastMerge` (`internal/battenshed/innerrun_owner.go:192`) calls `restepped` only for a transient Finalize failure and calls `parentHolds` otherwise. `OriginParentHoldsTaskWork` takes `pairLocation` (card 4). The `childRestepped` source chain matches `internal/battencli/cancelcmd.go:400-411`.
- **Card 5, "false only withholds the act a hold would otherwise yield":** `Act` checks the owned-elsewhere rule (rule 2) before the landing-past-merge rule (rule 4). Rule 4 differs from the fall-through only under `Held`.
- **Card 11, `CancelMarker` claims:** they match `passes.go:124` and `:203` and the review's account of card 8.
- **Board release of a done entry:** `Store.SetStatus` (`internal/boardengine/store.go:656`) clears a status subject only to `validateWrite`. Nothing refuses a `done` entry.

Further tool calls:
1. grep of `FinalizePastMerge` and card 4's origin reads: my context did not show the seam signatures.
2. grep for the board `SetStatus` and `lym fabric remove` verbs: to confirm the commands the new `Landed` text names exist.
3. Read of `internal/boardengine/board.go:200-249`: to see whether clearing a done entry's status is refused.
4. Read of `internal/boardengine/store.go:645-689`: the same check one level down.
