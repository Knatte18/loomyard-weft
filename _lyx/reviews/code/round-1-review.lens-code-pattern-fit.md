```yaml
lens: code-pattern-fit
outcome: reported
```

**Lens code-pattern-fit: each rule checked, held or broken**

- `PATTERN-dobby-starts-runs`: held. The diff only frees slots. Starts still go from an empty status through `startFirstPass`.
- `PATTERN-owner-machine`: held. The filter runs before the planner, so no pass is added or removed for runs that were not done.
- `PATTERN-test-speed`: held. Both new subtests sit in `internal/dobbyengine/starts_test.go`, which has the `//go:build tier0` tag, and use `t.Parallel()`. They use the existing `newPassHarness` and fake spawner, and no real time or sleep.
- `PATTERN-cwd-resolution`: held. No new import and no path resolution were added.
- `PATTERN-tolerant-records`: held. `state.json` is still rewritten through `UpdateState`, with no new key.
- `PATTERN-daemon-wakeups`: held. The diff adds no wait or poll.
- Repo `CLAUDE.md` docs rule: held. The `doc.go` passes and starts bullets and the `startReady` doc comment are updated in the same commit.
- Repo `CLAUDE.md` rule against touching a board entry or its scope: held. The diff touches no board file.
- `scribe:golang-comments` (the doc comments in the diff): held. The new constant and the `drivePasses` doc comment sit above the code and carry no end-of-line comments. Line breaks are the `lint-comments` gate's to judge, not mine.
- Declared PATTERN changes landed: held. No entry is declared, and `PATTERN.md` and `pattern/` are untouched.
- Commit message and `Pattern-Departure:` trailer: held. There is one commit and it carries no trailer, so no departure is claimed.
- `CLAUDE.md` markdown rule (semantic line breaks): held. The new `doc.go` lines are one sentence per line.

No findings from this lens.

Tool calls (1):
1. `head` of the two test files, a grep of `PATTERN.md` and `pattern/PATTERN-owner-machine.md` for "done", the `PATTERN.md`/`pattern` diff stat, and a grep of the `passes_test.go` harness. My context lacked the build tags, the PATTERN text for done runs, and whether the PATTERN files changed.
