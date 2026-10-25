```yaml
lens: code-pattern-fit
outcome: reported
```

**Lens: code-pattern-fit. Rules checked against the diff, held or broken:**

- Held: `PATTERN-cwd-resolution`.
  - `internal/dobbyengine/view.go` imports only `slices`, `strings` and `time`, so there is no new non-stdlib import and the allowlist needs no change.
  - Neither `lyxcwd` nor a CLI package is imported.
  - `battencli` and `beamcli` already import `lyxcwd`, and `NewDobbyViewReader` resolves its worktree through it.
- Held: `PATTERN-tolerant-records`.
  - `StartState.Refusal` is `omitempty` (`state.go:64`).
  - It is written inside `refusedStart`'s `updateStart`, which goes through `UpdateState`.
  - The delete-when-empty case includes `Refusal == ""` (`starts.go`, `updateStart`), so the record clears with the rest.
- Held: `PATTERN-pair-parent`. The parent is read only through `DobbyParents.Of`, which calls `hubgeom.ParentOf` (`dobbyreads.go:53`). Owners resolve through the same cache.
- Held: `PATTERN-cli-cobra`. `statusCmd()` is added to the existing beam tree (`beamcli/cli.go`, `beam.AddCommand`). It is tagged operator audience.
- Held: `PATTERN-daemon-wakeups`.
  - The section adds no wake.
  - Times render through `durfmt.FormatMinutes`, so the text changes at most once a minute.
  - The poll code is untouched.
  - The read-failure log in `overviewDobbyLines` is Debug level.
- Held: `PATTERN-config-layers`. `LoadConfig` is the same strict load the dobby uses.
- Held: `PATTERN-test-speed`.
  - `view_test.go` and `overviewdobby_test.go` are tier0 with `t.Parallel`.
  - The hub-needing `status_integration_test.go` is tier1 and uses `hubforge`.
  - `passes_test.go` extends an existing file.
- Held: `PATTERN-spawn-observability` and `PATTERN-gitrepo-client-boundary`. The diff spawns no process and runs no git.
- Held: docs in the same commit.
  - `dobbyengine/doc.go`, `beamcli/doc.go`, `battencli/cli.go` and `docs/overview.md` are all updated.
  - No `PATTERN.md` entry is needed: the diff adds no cross-cutting invariant, and the directive declares no PATTERN changes.
- Held: stencil change. The single added line in `orch-template-role.md` drops or weakens no rule.
- Held: commit message. It has no `Pattern-Departure:` trailer, and none is needed.
- Held: the `Log` seam is wired to the dobby's stdout, which `SpawnDetached` points at `dobby.log` (`start.go:203`).
- **Broken:** the acceptance criterion and auto-mode assumption that a task pair's frame carries no dobby section (finding F1).

**F1 — MEDIUM, design.** `internal/battencli/overview.go`, the `overviewCmd` block that calls `NewDobbyViewReader(location, acting)`, next to `acting := location.WorktreeName`.
- The overview draws a dobby section for whatever worktree it runs in, with no check that it is the prime or a beam.
- `hubgeom.DobbyGeometry` succeeds for any existing worktree directory (`internal/hubgeom/dobbygeom.go:25-36`). It sets `Slug` to the worktree name for anything but the prime.
- Run in a task pair, the pane therefore shows `dobby <pair>: not running; way forward: lyx beam start`. A pair has no dobby, so that way forward is wrong.
- The decision record's acceptance criterion says "a task pair's frame carries none", and its auto-mode assumption says a pane in any other worktree shows no section.
- Nothing in `overviewdobby_test.go` or elsewhere covers the pair case, because the test sets `source.dobby` directly.
- The check belongs where the reader is built. Skip the section unless `acting` is the prime's name or a name in `hubgeom.DobbyStart(prime).Beams()`. Add a test row for the pair case.

Tool calls (2 commands in 1 call):
1. `grep` of `hubgeom.DobbyGeometry`, to see whether it rejects a non-beam worktree. The manifest did not show its body.
2. `grep` of `dobby.log` and `Stdout` in `dobbyengine`, to confirm `SpawnDetached` points stdout at the log. The manifest had only the doc claim.
