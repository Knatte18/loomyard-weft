```yaml
lens: code-correctness
outcome: reported
```

Task: code-correctness lens over the amendment, where the only change since the settle is the merge into `docs/overview.md`.

**List (function or branch, case run, result)**
- No Go function, branch or error path changed in the amendment. The parent's overlap with the task is `docs/overview.md` alone, and the parent's hunks there are the darn-recipe and regression-row sentences. Nothing is left for step-through execution, and I settled the settled code's correctness out of scope.
- `docs/overview.md:311`, the `internal/creelcli` row: names the hidden `daemon` and `hook-read`. `internal/creelcli/hookread.go:18` defines the verb as `hook-read`, so it is correct.
- `docs/overview.md:312`, the `internal/creelengine` row: names the per-strand mail flags and the tool-boundary claim, which matches the decision record's flag file and claim verb. Correct.
- `docs/overview.md:472`, the creel module bullet: lists `lym creel hook-read` among the hidden verbs. Correct against the verb's code.
- `docs/overview.md:473`, the new creel line, run against the decision record:
  - "Claude strand in a hub" matches the Claude-only hook, which `claudeengine/settings.go:301` omits for a session in no hub.
  - "mail, agent mail included, in full at its next tool call" matches the record (an 8 KiB budget, and at least one message whole).
  - "tests the strand's flag file ... only then runs `lym creel hook-read`" matches the record.
  - "each message delivered once, by that hook or by the idle pane path" matches the single-reservation decision.
  - The line says nothing about hold and pause, so it contradicts nothing. Omitting them is not a defect.
- `docs/overview.md:499` and `:565` (the parent's hunks): the `Drift-Await` text is intact. No conflict markers, duplicated lines or mangled sentences, and the grep for conflict markers found none. The task's hunks at 311, 312 and 472-473 do not touch the parent's lines.

**Findings:** none.

**Tool calls:** none. The fork's context already held the diffs, the grep for conflict markers and the creelcli, claudeengine and overview greps.
