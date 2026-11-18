```yaml
lens: fix-consistency
outcome: reported
```

Task: check the change set against its surrounding docs and comments for stale twins.

Findings:

1. **LOW**, `internal/shuttleengine/claudeengine/doc.go:543-544`.
   - The python row paragraph still says "`shuttle.yaml`'s `deny_python` switches it off for a Python target repo".
   - Lines 548-553 of the same paragraph now say the hook is installed in every run and only answers live.
   - This is the same stale claim as F7 and F8, fixed in `rows.go` and `configcli.go` but not here.
   - Suggested wording: `deny_python: false` lets Python through for a Python target repo.

2. **LOW**, `internal/shuttleengine/engine.go:316` (`PythonAllowedHookVerb` doc) and the same wording in both engine `doc.go` files.
   - Evidence: `engine.go:316` says "the hub's shuttle config", `claudeengine/doc.go:550` and `codexengine/doc.go:106` say "the hub's `deny_python`", and `docs/overview.md:683` says "the hook reads on each…".
   - `hookpython.go` was changed (F2) to read the launch's own config, which is the state directory's config in a standalone run.
   - The verb's `Long` text says this, but these twin descriptions still say hub only.
   - They should say "the shuttle config the launch read".

3. **LOW**, `internal/shuttleengine/claudeengine/settings.go:332-336` and `codexengine/guards.go:31-36` (the `pythonAllowedCheck` comments).
   - They say the verb "asks the hub" and "resolves the session's own worktree".
   - After F2 it resolves the standalone target's state directory when there is no hub board.
   - The comments are not wrong for a hub, but they omit the standalone path.
   - The codex hook runs from "the hook's working directory", and the verb derives the standalone state directory from that cwd or the worktree root.
   - This is an existing assumption the F2 fix leans on, so the comment should state it.

4. **LOW**, `internal/shuttlecli/cli.go:61`.
   - The comment lists `hook-python-allowed` as "which resolves its own config".
   - The sentence and the sibling comment at `cli.go:16` are still accurate.
   - But `hook-batching` and `hook-edit` keep the plain-text stderr error (existing debt), while the new verb now emits JSON.
   - Nothing in the change set records that departure for the siblings. The review called it existing debt, so this is a note only.

No BLOCKING or MEDIUM findings. The tests, `rows.go`, `configcli.go`, the `guards.go` panic text and the keep markers agree with each other.

Calls made:
1. `grep -rn "hook-python-allowed\|deny_python"` across the non-test `.go`, `.md` and `.yaml` files: finds twins of the changed text, which my context lacked.
2. `sed` of `engine.go:312-318`, `claudeengine/doc.go:540-554`, `codexengine/doc.go:103-109`, `spec.go:76-82`, `template.yaml:14`, `docs/overview.md:683` and `shuttlecli/cli.go:58-63`: gets the exact wording of the twins found in call 1.
