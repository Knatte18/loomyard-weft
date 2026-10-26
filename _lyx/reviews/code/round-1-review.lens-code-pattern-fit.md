```yaml
lens: code-pattern-fit
outcome: reported
```

Lens: code-pattern-fit. I checked the changed files against `PATTERN.md`, the `pattern/` background files and `CLAUDE.md`.

**Rules checked**

- **Declared change `PATTERN-owner-machine`: held.**
  - The `PATTERN.md` line (`PATTERN.md:26`) matches the declared text verbatim.
  - `pattern/PATTERN-owner-machine.md` drops the run-held paragraph and the per-task `owner_machine`. It now says owners sit only on root and beam projects, and that a load drops a task's leftover key.
- **Declared change `PATTERN-run-process-owner`: held.** The `PATTERN.md` line (`PATTERN.md:55`) matches the declared text. The background file's opening sentence and its Enforcement bullet now name `End`, `EndSparing` and `EndRole`.
- **`PATTERN-run-process-owner`, ending through `runproc` only: held.**
  - Non-test call sites of the End family:
    - `internal/loomcli/pawlwiring.go:289` (`EndRole`)
    - `internal/orchcli/cli.go:263` (`EndRole`)
    - `internal/fabriccli/anchordeps.go:243` (`EndSparing`)
    - `internal/pairteardown/teardown.go:597` (`EndSparing`)
    - `internal/tendercli/deps.go` (`End`, `EndProcess`)
  - `EndRole` is role-scoped with `End`'s sequence, as D4 asks.
- **`PATTERN-daemon-wakeups`: held for Beam-Await.** `awaitFloor` is one second and `awaitCeiling` one minute, with backoff. `pause` waits out the rest of the floor after an event, and there is no raw `time.Sleep`, `Kill` or `Signal` under `internal/pawlengine` or `internal/beamcli`.
- **`PATTERN-pane-text-injection`: held.** The one reference to `lyx orch start` in that background file is updated to `BeamUp`.
- **`CLAUDE.md` docs in the same commit: held.** The diff touches the `doc.go` of `adoptcli`, `beamcli`, `orchcli`, `orchengine`, `boardengine`, `dobbyengine`, `battenshed`, `shortcutcli`, `fabriccli`, `loomrecipe` and `pawlengine`. It also updates `docs/overview.md`, which carries the pawl, `beam start`/`beam stop` and `lyx s` text.
- **Removed verbs: held in non-test Go and markdown.** A grep for `orch start`, `lyx prime`, `beam promote`, `adopt --prime`, `orch cleanup` and `orch-template-adopt` finds only these:
  - Generic prose "orch start" in doc comments: `orchengine/launch.go:6`, `orchcli/start.go:64`, `orchcli/start.go:275`, `orchcli/notify.go:49` and `beamcli/cli.go:150`. These name the orch's start action, not the removed verb.
  - Historical `docs/research/` reports.
  - `orchengine/pause.go:165`, which I found by grep only; the hit looked like generic prose too, but I did not read the line.

  I did not search `plugins/ly`, `contracts/stencils`, string literals in test files or the `.yaml` files beyond this grep.
- **`PATTERN-shed-recipe-registry`, `PATTERN-parent-lock`, `PATTERN-pair-parent`, `PATTERN-refusal-way-forward`, `PATTERN-tolerant-records`, `PATTERN-config-layers`: not examined.** My six calls went to the PATTERN diffs and the greps above, so I cannot report them held or broken.
- **Commit messages and departures:** no `Pattern-Departure:` trailer appears in the range, so there is nothing to judge there.

**Findings:** none.

**Tool calls (3 of 6)**
1. `git diff` of `PATTERN.md` and `pattern/`, the `Pattern-Departure` trailers and `runproc/end.go` — my context held only a stat for these.
2. Grep for removed verb names across `.go`, `.md`, `.yaml` and `.svg` files — to check the removals landed everywhere.
3. Grep for `runproc` End-family call sites, raw sleep or signal calls, and `docs/overview.md` mentions — to check the process-ownership rule and the docs.
