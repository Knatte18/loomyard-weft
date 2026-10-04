# Card 4 — pattern-judge-role

**Edit:**
- `internal/pattern/pattern.go`
- `internal/pattern/pattern_test.go`
- `internal/pattern/doc.go`
- `contracts/stencils/stencils.go`
- `contracts/stencils/bouncer/bouncer-template-judge.md`
- `internal/shedadapters/bouncer.go`
- `internal/shedadapters/bouncer_judge_test.go`
- `internal/shedrecipe/entries_bouncer.go`

**Create:**
- `contracts/stencils/pattern/pattern-directive-judge.md`

**Intent:** Give the Bouncer judge the PATTERN directive through a new `RoleJudge` (D5).

Add `RoleJudge` with its own stencil-name constant and `Directive` case.
The new stencil carries the `{{.pattern_overview}}` marker and is worded for judging findings, not reviewing.
A finding that cites a `PATTERN-*` entry violation is never downgraded below blocking on the judge's own say-so.
The judge still never opens the artifacts.
It reads background files under the literal `pattern/` only to weigh a cited entry.
Register it in `stencils.go`.

`bouncer-template-judge.md` gains an optional `{{.pattern_directive}}` marker before the judge's first work instruction.
`BouncerConfig` gains a `WorktreeRoot` field, which `entries_bouncer.go` fills from its env's worktree root.
`judgeCall` calls `pattern.Directive(b.cfg.WorktreeRoot, b.cfg.StencilsDir, pattern.RoleJudge)` and fills the template through `stencil.FillOptional` with `pattern_directive` optional.
A directive error degrades like an unreadable judge template, with its own log message.
An empty `WorktreeRoot` yields no directive (Directive's empty-path case), so existing struct-literal test configs stay valid.

`internal/pattern/doc.go`'s "Why three roles, not one" is rewritten for the five roles, one sentence per role on why its wording differs.
Rewrite it without pinning the count in the heading.

Tests:
- `pattern_test.go` adds the judge role to its every-role cases.
- `bouncer_judge_test.go` proves the judge prompt carries the directive when `PATTERN.md` exists at the configured worktree root and omits it otherwise.
- It also proves a failed directive read degrades rather than spawning.

**ImpactSummary:** Adds one Role value, one stencil and one BouncerConfig field filled at the single bouncer recipe entry; the judge prompt gains an optional section.
