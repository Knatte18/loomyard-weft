```yaml
lens: fix-consistency
outcome: reported
```

Task: check the change set for twins of the changed text left stale (fix-consistency lens).

no findings

The one search I ran for the old wording came back empty. It covered `.go`, `.md` and `.yaml` files outside `_lym/`, for "unknown or has no hooks", "unknown or hookless", "engine is unknown" and "hookless". So no copy of the old rule is left in `cli.go`, `verbs.go`, `docs/` or any doc comment.

The new wording in `cli.go` for `creel` and `creel post` also matches the one in the `verbs.go` refusal: any strand without the hook mark gets the typed notice or reminder.

Tool calls made:
1. `grep -rnE` for the old phrases over `.go`, `.md` and `.yaml` files, to find twins of the reworded help and refusal text that my context lacked.
