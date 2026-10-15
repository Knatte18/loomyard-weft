# Card 1 — doc-lifecycle-wording

**Edit:**
- `CLAUDE.md`
- `docs/overview.md`

**Intent:** Carry D1: rewrite the documentation-lifecycle wording so it no longer says or implies that a design moves out of, or is cut from, the task's own board entry.

In `CLAUDE.md`'s `## Docs land in the same commit` section, replace the sentence "An unbuilt design lives in its board entry's body, and a built design lives in its package's `doc.go`: a design whose code has landed moves from the board entry into `doc.go`." with sentences saying:

- an unbuilt design lives in its board entry's body, and a built design lives in its package's `doc.go`;
- the task that builds a design writes it into the package's `doc.go` and never changes its own board entry's body or scope, since the landing marks the entry done and the board prune removes it, so no pointer is left behind;
- no decision, card or step of a run changes the running task's own board entry's body or scope, because the run's board lock refuses such a change.

Scope the rule to body or scope only: a status write, a `run_settings` write, a write to another board entry (such as filing a follow-up entry) and a change to the board module's code stay allowed, and the text forbids none of them.

In `docs/overview.md`'s `## Documentation lifecycle` section, rewrite the "Module designs" bullet's second line ("when the module lands, the entry's design moves into the package's `doc.go` ...") to say that the task building the module writes the design into the package's `doc.go`, leaves its own board entry's body and scope unchanged, and the landing marks the entry done and the board prune removes it; the implementation and tests then become the source of truth.
Keep the heading `## Documentation lifecycle` byte-identical, since other docs link to its anchor (`PATTERN-markdown-link-integrity`, pinned by `internal/lyxcwd`'s docs link test).

`PATTERN.md`'s `PATTERN-documentation-lifecycle` line and `pattern/PATTERN-documentation-lifecycle.md` stay unchanged: they say only where a built design lives, and the record puts a `PATTERN.md` entry out of scope.
Write one sentence per line in both files.
No test is planned: the change is static prose, which `PATTERN-test-economy` exempts; the docs link-integrity test already pins the anchor.

**ImpactSummary:** Two prose files; one anchor (`#documentation-lifecycle`) kept unchanged, no code or test reads the reworded sentences.
