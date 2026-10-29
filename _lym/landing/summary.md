# Delete CLAUDE.md after moving the rules only it held
The repository's `CLAUDE.md` is deleted.
LoomYard adds no instruction file to a repository, so the rules agents follow reach them through PATTERN, the stencils and the plugin skills.
Each rule that stood only in `CLAUDE.md` now has a home that reaches the agents needing it.
The opening line about LoomYard developing itself and the conversational-shorthand rule get no new home.

## What changes

- `darn-template-write.md`: step 4.3 states the docs a change carries in its commits, in generic terms.
  These are the package doc of each package whose behavior changes (`doc.go` for Go), the repository overview when a module table or execution stack it shows changes, and a `PATTERN.md` entry for a cross-cutting invariant when the repository has `PATTERN.md`.
  The step-1 "read `CLAUDE.md`" line goes.
  The stencil ships to every hub, so the rule holds in a repository without `docs/overview.md` or `PATTERN.md`.
- `drift-template-baseline.md`: step 1 reads the repository's README and test setup for how it is built and tested, in place of `CLAUDE.md`.
  The docs rule is not added here: the baseline agent changes no code, and the old line named the build and test route, not the doc list.
- `edit-directive.md` and `internal/editdirective/doc.go`: the directive, rendered into every spawned role, adds the own-worktree rule.
  A role never edits, commits or pushes in another worktree and never creates one, unless its parent or the operator says so for that case; work that belongs elsewhere is reported to the parent.
- `orch-template-role.md`: the placement rule for a new rule names `lym orch rules` for orch-only workarounds, the stencils and skills lym ships for what spawned agents need, and lym's code for what lym enforces.
  `CLAUDE.md` is no longer named.
- Stale citations of `CLAUDE.md` as a live rule are fixed in `cmd/lym/crosscompile_test.go`, `internal/fabricengine/doc.go` and `tools/mdreflow/reflow.go`.
  `CLAUDE.md` leaves the vocabulary scan's root-file set in `internal/lymcwd/enforcement_test.go` and the scope list in `pattern/PATTERN-fabric-vocabulary.md`, which change together.
- `PATTERN-parent-directive.md` and the `testtiming:keep` reason in `contracts/stencils/parentdirective_test.go` say the `edit-directive` stencil states the edit directive's rules, and the test enforces the edit rule only.
- `merriamFixedContext` in `internal/websterengine/merriambase.go` keeps its value, `20600 - 392*12`.
  Only its comment changes: the fixed figure is the system prompt, tools and skills, which deleting the file does not alter, and the base drops the file's lines on its own.

## Outside the diff

The production rule ("never run `update-plugins.sh` unasked, never install `lym` elsewhere", with its exception after the orch's own pushed fix on a clean `main`) and the hand-push gate (`lym gate test --upto 2` green as its own Bash call before a push to `main`, with the docs-only exemption) go into the hub-wide orch rules file.
That file lives in the hub at `_board/_lym/orch/rules.md`, not in this repository, and is written through `lym orch rules --from`, so no hunk of this diff carries it.

## To check by hand

- Read the hub's orch rules file: it states the production rule with its exception and the hand-push gate, and every other section is unchanged from before.
  The verb replaces the whole file, so a draft that dropped a section would not show in this diff.

Closes nothing: the board entry names no issue.
