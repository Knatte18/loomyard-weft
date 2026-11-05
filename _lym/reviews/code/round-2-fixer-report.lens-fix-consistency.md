```yaml
lens: fix-consistency
outcome: reported
```

No findings.

Task: check the `PATTERN.md` change to the `PATTERN-durable-vs-ephemeral-state` line against its siblings and cross-references.

- The new clause "bar heddle's rendered files and `.heddle` at a warp worktree's anchor" matches background line 16 ("Heddle's rendered instruction files … and the `.heddle` temporary directory sit at a warp worktree's anchor").
- "bar a `tools/` script's variables" still points at the exception in the background's "Single declarer" section ("a `tools/` script's variable block"). It is only a softer wording of that exception.
- "untracked under `.lym`" is the same meaning as the background's "never-tracked". The background and the `PATTERN-tracked-file-locks` entry were left unchanged, which is not a conflict.
- The `PATTERN-tracked-file-locks` line and the link target are unchanged and agree with background line 6.

Tool calls:
1. Read `pattern/PATTERN-durable-vs-ephemeral-state.md`, because my context lacked the background file the changed line summarises.
