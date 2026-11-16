```yaml
lens: existing-paths
outcome: reported
```

Lens: existing-paths — trace each decision's state against the code paths that already write, reset or read it.

Paths traced:
- D1 manifest record: written by `Prepare` into the run directory. The run-dir readers (attach, wait, stop) never read it, so it survives. A reed relaunch re-types no prompt and does not re-Prepare, so the record stays as first written. Survives.
- D3 edit directive: `editdirective.Directive` callers are sheddriver tackler, treadle targeting and judge, sampler, webster render (three sites), landingshed prsession, loomcli driverprompt, gatefixer, shedarbiter, friction, dobby tidy and mergeresolve. Each gains an engine argument. Stencilstore seeding is the second path: a Bolt copy whose hash no longer matches is never overwritten. Breaks in part (finding 3).
- D4 prompt engine: a respawn or relaunch reuses the spec or launch line. An unset value passes, so the bound holds. Survives.
- D5 developer instructions: the launch line carries them, so a reed relaunch replays them. Codex resume (`buildResumeCmd`) still exits, which is out of scope. Two more readers: the skill-load turn order (load turn first, then the pointer to the prompt file) is unchanged, and the creel idle wake (`WakeIdle` / `codex queue`) does not read the prompt file. Survives.
- D6 thread cap and effort: `-c` keys sit beside the existing `-c` hooks on the same line. Survives.
- D7 shipped entries move to `agents-file`: the file targets render only through the seam's `Made`/`TookIn`, the heddle verbs and a hand render. The hub reconcile renders only `HEDDLE.md`. Breaks for live worktrees (finding 1).
- D7 `agents-prompt`: read by `Renderer.Passages` through `promptTargetsFor`, and `lym heddle show` refuses a shared target. Finding 4.
- D8 home `AGENTS.md`: written under the home lock next to `config.toml`. The other Codex calls under that home (sandbox probe, `codex queue`) never read it as instructions that matter. Survives.
- D2 scan: it covers all of `contracts/stencils/`, orch stencils included. Finding 2.
- D9: AGENTS.override.md is rendered only by heddle's file render (the same paths as D7). Finding 1 applies to it as well.

Findings:

1. MEDIUM — D7 and D9: a shipped-entry or override-file change never reaches existing worktrees after a lym upgrade.
   Evidence: `internal/heddleengine/seam.go` renders file targets only on `Made` and on a `TookIn` that touches a re-render path. Its doc says "A change to the store itself is never an event". `internal/hubreconcile/walk.go` renders only the overview (`renderHeddleOverview`). The shipped layer changes with the lym build, so every live worktree keeps an `AGENTS.override.md` without `lym-batching` and `lym-plan-edits-before-gate`, and without the AGENTS.md+CLAUDE.md merge, until someone runs `lym heddle set`/`remove` or a hand render. The parity test runs on fixtures and cannot see this, and the record does not mention it.
   Fix: decide that a lym build re-renders file targets in live worktrees (for example in the hub reconcile that already runs once per build), or state the bound and the way forward (`lym heddle render`).

2. MEDIUM — D2 against Scope (orch out): the scan's reach includes the orch stencils, which are Claude-only by design.
   Evidence: `contracts/stencils/orch/orch-template-compact.md` is the focus text typed after `/compact`, and `orch-template-handoff-soft.md` speaks of a "subagent or fork". D2 fails a closed-list term found anywhere in `contracts/stencils/` outside `engine/claude/`. So the orch stencils would need Codex copies of text no Codex session ever renders, or the scan must exclude them, and the record says neither.
   Fix: exclude `contracts/stencils/orch/` from the scan, with the Scope's reason.

3. LOW — D3: removing or demoting `contracts/stencils/edit/edit-directive.md` meets stencilstore's no-overwrite rule.
   Evidence: PATTERN-stencil-ownership: "A hash-mismatched file is never overwritten". A hub whose Bolt copy of the shared directive was edited locally loses that edit without notice once `Directive` reads `engine/<engine>/edit-directive.md`. The PATTERN background's "Edit directive" line ("The `edit-directive` stencil is the only statement of the edit directive's rules") also goes stale, and the record's PATTERN change rewrites only the overview line.
   Fix: name the background edit in `## PATTERN changes`, and say what happens to a locally edited seeded copy.

4. LOW — D7 `agents-prompt`: the record does not say whether the new target reaches orch sessions.
   Evidence: `Renderer.Passages(boltDir, engine, role)` gathers `promptTargetsFor(engine, role)`. The orch session runs on Claude, so an `agents-prompt` entry that renders into `claude-prompt` would also reach the orch prompt, unless `claude-prompt` already excludes the orch role. The record states neither. `lym heddle show agents-file` is refused as a shared target, and `agents-prompt` needs the same refusal, which the record does not decide.

Tool calls beyond context:
- grep for heddle render callers and Claude terms in the orch/shed/beam stencils: needed to find who re-renders file targets. The pattern matched nothing.
- grep of the render call sites and an ls of the orch stencils: needed the list of orch stencil files.
- Listed the `heddleengine/render.go` functions and the heddleengine importers, and grepped the orch stencils for slash commands: needed the render entry points and confirmation of orch Claude terms.
- grep of `hubreconcile/walk.go` and the `RenderRequest` sites: needed whether a lym build re-renders file targets.
- Read the doc lines of `heddleengine/seam.go`: needed which events trigger a file render.
