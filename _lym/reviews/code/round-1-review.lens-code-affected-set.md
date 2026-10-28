```yaml
lens: code-affected-set
outcome: reported
```

Lens: code-affected-set. Scope is what each change forces outside the lines it touches. I found no defect in that set, so I report no findings.

**Per change, the set found:**

1. **`CLAUDE.md`, the PATTERN-overview-inlined sentence removed.**
   - Still stated: `contracts/stencils/pattern/pattern-directive-*.md` line 6 says the directive carries the `PATTERN.md` content.
   - `internal/websterengine/merriambase.go:22` mentions "its inlined PATTERN overview 104". It is a measured constant pinned "at the commit the constant was taken at". Its `CLAUDE.md 59` figure was already off against the pre-diff 64 lines, and the diff does not make the constant wrong.
   - Covered.
2. **`CLAUDE.md`, the Batten Land `deploy` sentence removed.**
   - Still stated: `internal/battenshed/doc.go:198` and `land.go` enforce it, and `contracts/stencils/orch/orch-template-role.md:142` tells the orch.
   - Covered.
3. **`CLAUDE.md`, the notes section replaced by the kind-split section.**
   - Orch-rules routing: it stays in `contracts/stencils/orch/orch-template-role.md:232` (hub-wide file through `lym orch rules`, thread file through `lym fabric commit`).
   - Memory-store rule: added at `orch-template-role.md:233`.
   - References to the old heading: no stencil, spec, `pattern/` file, `PATTERN.md` or Go code cites "Notes go in git" or the removed `_lym/raddle/` sentence outside `docs/research/` bench snapshots. Those snapshots are records of past runs.
   - Tests on the section: no Go test in `contracts/stencils` or `internal/stencilstore` references "Scratch and notes" or "memory store".
   - Covered.
4. **`CLAUDE.md`, the Docs section trimmed (unbuilt design in body, run-lock rule).**
   - Run-lock rule still stated in the discussion stencil (`loom-template-discussion.md:87-88`), the plan stencil (`loom-template-plan.md:45`), the darn write stencil (`darn-template-write.md:96`), and the discussion, plan and code-review rubrics. The board code enforces it (`boardengine/doc.go` run lock).
   - Design-in-body rule: added at `plugins/ly/skills/board/SKILL.md:172-173`, under "Writing a body" (heading at line 164, the last section).
   - The skill's claim that `lym board prune` removes a done entry is borne out by `boardengine/board.go:375` (`Prune` removes every done entry).
   - `darn-template-write.md:26` and `:76` send agents to the repository's `CLAUDE.md` for its docs rule. They name no removed text, so nothing is stale.
   - Covered.
5. **`CLAUDE.md`, the board terminology bullet shortened.**
   - The "only through `lym board`" rule is carried by the board skill's description. No other document quotes the old bullet.
   - Covered.
6. **Rules in the repo's rule documents (`PATTERN.md` and `pattern/`).** The declared-changes list is empty and the diff touches neither, so nothing is owed.
7. **Commit message.** It carries no `Pattern-Departure:` trailer and needs none. Its claim that the other removed rules were already stated or enforced holds for items 1 to 5. The `_lym/raddle/` sentence is covered by D3 as dropped rather than relocated, and the message omits that exception. The message is thereby slightly imprecise, but D3 licenses the drop, so I raise no finding.

**Tool calls (3):**
1. `grep` over the board skill's headings, repo-wide `CLAUDE.md` and "Notes go in git" references, and "durable note"/`memory/` in stencils and plugins. My context lacked reverse references to the removed `CLAUDE.md` text.
2. `sed`/`grep` on `merriambase.go` and `wc -l CLAUDE.md`, to read the pinned line-count comment, which my context lacked.

That is two calls, not three. The first Bash call was the combined one above, and the second is the one that follows it.
