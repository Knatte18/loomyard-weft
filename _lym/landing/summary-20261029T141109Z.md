# Delete the loomyard CLAUDE.md after moving the rules only it held

The repository-root `CLAUDE.md` is deleted.
Each rule that stood only there now lives where the agents needing it receive it, as the operator decided from `docs/research/claude-md-rule-map-2026-10-29.md`; the rows that report marks `covered` are left as they stand.

## Where each rule went

- **Docs and PATTERN in the same commit**: `contracts/stencils/darn/darn-template-write.md` step 4.3 now states the rule itself: the package doc of each changed package (`doc.go` in Go), the repository's overview doc when a module table or execution stack it shows changes, and the `PATTERN.md` entry for a new or changed cross-cutting invariant when the repository has one.
  Step 1's "read the repository's `CLAUDE.md`" line is gone.
- **Drift baseline**: step 1 reads the repository's README and test setup for how it is built and tested, in place of `CLAUDE.md`.
- **Own worktree only**: `contracts/stencils/edit/edit-directive.md`, rendered into every spawned role, adds "work only in your own worktree: never edit, commit or push in another, and never create one, unless your parent or the operator says so", and tells the role to say so to its parent when work belongs elsewhere; `internal/editdirective/doc.go` follows.
- **Where a new rule goes**: the Where-a-finding-goes paragraph of `contracts/stencils/orch/orch-template-role.md` names the orch rules file for orch-only workarounds and the stencils and skills lym ships for rules spawned agents need, keeps the "never in a package's `doc.go`" clause, and says lym adds no instruction file to a repository.
- **Production and hand pushes**: the hub-wide orch rules file on Bolt, written by the parent through `lym orch rules`, now states that `update-plugins.sh` is the only route to production, never run unasked, with no other `lym` install, keeping the orch's existing exception; a new "Pushing `main` by hand" section carries the `--upto 2` gate, the own-Bash-call rule and the docs-only exemption.
  That file lives outside this diff.

The opening background line and the conversational-shorthand rule are dropped.
The interactive-tmux and filesystem-links rules already stand as `PATTERN-interactive-spawn` and `PATTERN-fs-links`.

## Pointers to the deleted file

- `internal/websterengine/merriambase.go`: `merriamFixedContext` keeps its value `20600 - 392*12`.
  The fixed figure is the system prompt, tools and skills with every base text subtracted, so it does not depend on `CLAUDE.md`; the base itself drops the file's lines because an absent instruction file adds nothing.
  The comment now says so.
- `internal/lymcwd/enforcement_test.go` and `pattern/PATTERN-fabric-vocabulary.md`: the vocabulary scan's root-file set is `README.md` alone.
- Comments in `cmd/lym/crosscompile_test.go`, `tools/mdreflow/reflow.go` and `internal/fabricengine/doc.go` cite README, `scribe:prose` and `PATTERN-fs-links` instead.

## Design decisions

- The darn docs rule is phrased generically, since the stencil ships to every hub and `docs/overview.md` is LoomYard's own path.
- The drift baseline gets no docs rule, though the board entry names it as a target: that agent changes no code, and its `CLAUDE.md` line named the build and test route, not the doc list.
- The orch rules file was written by the parent, because `lym orch rules` runs only from the prime or a beam and refuses a task worktree.
- `docs/skills.md`, `docs/research/` and `crucible/` still mention `CLAUDE.md` as plan or historical records and are left as they are, as is the engine's instruction-file list in `internal/shuttleengine/claudeengine/capabilities.go`, which serves other repositories.
