```yaml
lens: fix-proposal
outcome: reported
```

Lens: fix-proposal. I challenged each proposal the fixer carried out as proposed: F1 import, F2 runproc edge, F3 seeding, F4 lock, F5 log-only, F6 BoltDir, F8, F9 tag shape, F10, F11, F13 and F14. I found one case where a proposal is wrong; the rest held.

**LOW** — D3 `claude-file` row ("an `@AGENTS.md` import line when the worktree's root holds an `AGENTS.md` and no `CLAUDE.md`, there or in its `.claude` directory").
The import condition checks only the root, so it can load `AGENTS.md` into a session that the team's own setup keeps it out of.
Case: a repository has a root `AGENTS.md`, and a `CLAUDE.md` in a subdirectory that is the session's anchor, with no root `CLAUDE.md`. Without heddle, the anchor's `CLAUDE.md` counts as a `CLAUDE.md` in the working directory, so Claude reads no `AGENTS.md`. With heddle, the root render adds `@AGENTS.md`, and Claude now reads it.
Evidence: the record's own sentence, "Claude Code reads `AGENTS.md` only when no `CLAUDE.md` … sits in the working directory or above it". Also D3's line "A repository with its own `CLAUDE.md` already keeps its `AGENTS.md` from Claude, so the render adds no import there", which assumes that `CLAUDE.md` sits at the root.
The new Open risk covers an `AGENTS.md` between the root and the anchor, but not a `CLAUDE.md` there.

Cases I built that held, so no finding:
- **F9:** the replayer tags do not collide with run tags. `SnapshotTagged` reads `set-aside/<slug>/` in the weft repository and counts only annotated tags. Replay set-asides are lightweight tags in Bolt.
- **F3/F4:** the lock order cannot deadlock. The exclude lock is taken and released inside each mutation (`internal/fabricengine/gitexclude.go:63-67`), so fabric holds no exclude lock when it fires an event.
- **F2:** the new edge closes no import cycle. Every in-package `lymcwd` test reaches `runproc` only through test imports, and `lymcwd`'s production code does not import `runproc`. `runproc`'s allowlist test (`internal/runproc/leaf_enforcement_test.go:22`) is what the PATTERN change widens.

Tool calls:
1. Checked the package clauses of `internal/lymcwd`'s tests and the import-allowlist tests, for F2's cycle and allowlist. My context did not show the test files' packages.
2. Grepped for readers of set-aside tags, for F9's collision case. My context showed only `setAsideTag`.
3. Read `SnapshotTagged` in `internal/fabricengine/snapshotuncommitted.go`, to see whether its prefix read could match a replayer tag. My context lacked that file.
4. Grepped for the exclude lock's scope in `internal/fabricengine`, for a lock-order case between F3 and F4. My context lacked `gitexclude.go`.
