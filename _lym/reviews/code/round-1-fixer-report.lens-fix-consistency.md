```yaml
lens: fix-consistency
outcome: reported
```

Task: check the change set through the fix-consistency lens.

**Findings**

- **LOW**: `internal/gatecli/doc.go:1051` still reads "It reads the uncommitted and untracked files through `Fabric.Status`."
  - After F1, `dirtyOwnedFile` calls `r.fabric.CodeWorktreeChanges()`, and `dirtyModuleFile` already did.
  - The module doc is a twin of the changed code that was left unchanged.
  - It should name `Fabric.CodeWorktreeChanges` and say that only the code side counts.

**Checked, no finding**

- No `--map` text mentions `--tags` any more. My grep for `--map` near `--tags` found nothing outside `docs/research`.
  - The help line in `cli.go`, the refusal list and message in `run.go`, `doc.go:1027` and the `cli_test.go` row all agree.
- `doc.go:81` and `doc.go:147` still describe the generic `--tags` refusal, which now also covers `--map --tags`. They stay correct.
- The restored help sentences and examples in `cli.go` match the original wording in the `git diff` I read.

Calls made:
1. Bash grep for `--map` near `--tags`, and for `Fabric.Status`, `dirtyOwnedFile` and "records side" in `doc.go` and `testmap.go`. My context lacked any search for twins of the changed text.
2. Read `doc.go:1046-1055`, to see the `Fabric.Status` mention in context.
