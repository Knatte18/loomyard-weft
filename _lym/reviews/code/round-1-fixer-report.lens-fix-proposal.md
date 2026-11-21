```yaml
lens: fix-proposal
outcome: reported
```

Task: fix-proposal lens, challenging each proposal the fixer carried out as proposed.

Two findings, both LOW. The other proposals I tried to break (F2, F3, F4, F6, F8, F10, F12, F14, F15) held.

1. **LOW** — F1, `bindCards, _ := withoutDroppedCreates(in.Cards, dropped)` in `PostBatchChecks` (`internal/runplan/postbatch.go`).
   - Case: the parent drops a handle-declared Create, but the batch built that symbol anyway, as the `recover-batch` "build the target as the card names it" route allows.
   - The delta holds the symbol, but the dropped handle is no longer owned by any card, so `BindHandles` never rewrites it.
   - A later card that references `plan:internal/foo#Dropped` stays an unbound `plan:` handle.
   - The comment at `postbatch.go:265` says this leaves later cards referencing unbound handles, and drift detection cannot see them.
   - The same drop with a card that has no sibling handle also leaves that handle unbound, because `OwnedHandles` is then empty.
   - Evidence: `BindHandles` (`planglyph/handle.go`) builds `subs` only from the handles it is given.

2. **LOW** — F5, `namedByAmendment` (`internal/burlerengine/roundcheck.go`).
   - The new check `filepath.Dir(path) == planparser.PlanDirRel()` compares against the literal `_lym/plan`.
   - `locationPath` only relativises an absolute path that lies under `WorktreeRoot`.
   - `_lym` is a link out of the worktree, as the `validate.go` comment says.
   - A reviewer location written through the link target, such as an absolute `.../_lym/plan/14-card-slug.md` resolved elsewhere, now fails the plan-card test and is refused where the old substring match admitted it.
   - The bare form `14-card-slug.md` fails the same way.
   - The review's proposal ("match only plan card files") does not say which spellings count.

No tool calls made; my context already held everything this lens needed.
