```yaml
lens: fix-consistency
outcome: reported
```

Task: check the round-1 change set for agreement with its sibling decisions, criteria and cross-references (fix-consistency lens).

**1. MEDIUM: the new D6 criterion contradicts the beam file's generated mark.**
- Changed text: the D6 acceptance criterion "a regeneration with no workspace-content hook installed writes workspace files holding only `folders` and `settings`".
- Summary: a beam workspace file always carries a third top-level key, so the criterion fails for every beam file even when no hook is installed.
- Evidence: `internal/fabricengine/hubworkspace.go:42` declares `beamWorkspaceMark = "lym.generated"`, and `markBeamWorkspace` at `hubworkspace.go:227-230` writes it as the first top-level key of every beam file. The D6 bullet's own wording, "with no hook installed a file carries no extra member", is the claim the criterion should mirror.

**2. LOW: the Notes bullet leaves out two of the hook inputs D6 defines.**
- Changed text: Notes for the plan writer, bullet 2: "The hook's inputs are settled: the prime's and each beam's anchor … and the binary path".
- Summary: D6 hands the hook the hub path, the beam slug (empty for the hub's own file) and the anchor path. The Notes list only the anchor and the binary.
- Evidence: D6 bullet 3 says "The installed hook is handed the hub path, the beam slug a workspace file belongs to (empty for the hub's own file) and that worktree's anchor path". The hook needs the beam slug to choose between `lym beam start root` and `lym beam start <beam>`.

**3. LOW: the D6 Bound does not cover the strip case the new seam introduces.**
- Changed text: the D6 bullet "with no hook installed a file carries no extra member", read against the D6 Bound's mixed-version sentence.
- Summary: the Bound covers an older binary stripping `beam start`, but not a current binary regenerating in a process that installed no hook. That process rewrites the files without `beam start` too, and the Bound states no recovery for it, unlike its twin case.
- Evidence: `cmd/lym/main.go:175` installs the existing namer only inside the armed branch of the root pre-run, beside where D6 puts the new hook. `regenerateAfterVerb` (`hubworkspace.go:130`) calls `RegenerateHubWorkspace` from any fabric verb's process.

No further tool calls made.
