```yaml
lens: fix-consistency
outcome: reported
```

Task: fix-consistency check of the fixer's change set against its surroundings (read-only).

1. **MEDIUM**: `internal/driftcli/cli.go`, the `defer` verb's Long text (twin of the changed `amend` Long).
   - It still says "only the run's parent or the operator's own shell runs it".
   - `defer` shares `checkParentCaller`, which now admits every orch above the pair. `amend.go` and `driftcli/doc.go` were updated; this text was not.

2. **MEDIUM**: `internal/loomcli/parentreview.go:371`, `parent-review answer` Long (twin of the changed `decision add`/`store` Long texts).
   - It says "The callers are the run's parent orch, resolved at call time, the orch strand the ask went to, and the operator's own unnamed shell".
   - The code admits every orch on the run's beam chain, as `internal/loomcli/doc.go` states.

3. **LOW**: `internal/shedarbiter/prompt.go:52` and `internal/sheddriver/tacklerprompt.go:27` (twins of the F3 rewording).
   - Both still say "ParentName is the orch the run hands its stops back to".
   - F3 removed that claim from the stencil, `HandBack`, `parentdirective/doc.go` and `stencils.go`, because a recorded parent whose pair is gone receives no stop.

4. **LOW**: stale "nearest live orch" wording for routing that F1 changed.
   - A start worktree that records an orch strand now takes the notice even when it reads as down.
   - `docs/overview.md:858` still says dobby sends "the nearest live orch" its notices, and dobby's `NearestOrch` and post seams now go through `Destination`.
   - The same phrase remains in `internal/dobbyengine` (`doc.go`, `describe.go`, `tidy.go`, `starts.go`, `loop.go` and others), `internal/battencli/wire.go`, `internal/loomcli/{prepush,drivewiring,landingdeps}.go`, `internal/adoptcli`, `internal/beamcli/{cli,dobbytrace}.go`, and the banners of `contracts/stencils/beam/beam-template-{describe,board-tidy}.md`.

5. **LOW**: `internal/loomcli/doc.go`, the `decisionCallerRefusal` paragraph (around lines 48-56).
   - It states only the chain rule.
   - F6 added that a chain that cannot be read still admits the resolved parent (`admissibleOrchs`, used by `add`, `replace`, `store` and `parent-review answer`). The `driftcli` docs mention the matching fallback; this one does not.

Tool calls made (3 of 6):
- Grep for other statements of the down-orch routing rule (no hits): my context lacked a repo-wide view.
- Grep for sibling caller-rule and hand-back statements: my context held only the files the fixer opened.
- Read `loomcli/doc.go`, `parentreview.go` and `driftcli/cli.go` spans and counted "nearest live orch" files: those spans were not in my context.
