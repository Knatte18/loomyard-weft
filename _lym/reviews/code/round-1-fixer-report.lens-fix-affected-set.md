```yaml
lens: fix-affected-set
outcome: reported
```

no findings

Lists I checked (each came back with no remaining members):
- F1: the only remaining `owns` call over dirty paths is `testmaptree.go:88`, now fed by `CodeWorktreeChanges()`. No `fabric.Status()` call is left in `internal/gatecli`.
- F3: no `--tags` mention tied to `--map` remains in any Go or Markdown file outside `docs/research`. That covers `run.go`, `cli.go`, `doc.go` and `cli_test.go`.
- F2: restoring the sentences and examples touches one file, `cli.go`, which has no other site.

Calls made:
1. One Bash call with three greps. I ran the F1 and F3 searches fresh because my context held only the pre-fix state, not the repo-wide matches. The F2 grep was a leftover-mention check.
