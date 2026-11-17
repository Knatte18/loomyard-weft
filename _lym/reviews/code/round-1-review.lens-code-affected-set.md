```yaml
lens: code-affected-set
outcome: reported
```

Lens: code-affected-set. For each change, I listed what reads, calls or describes it and checked whether the diff left any of it wrong.

**Per change, the set found:**
- **`orchcli.BeamDown` now ends the orch's process last; `BeamDownReport` gains `orch_process_ended`.**
  - Callers: `beamcli/cli.go:628` `stop`, `battencli/cancelcmd.go:384` StopBeam, `loomcli/pawlwiring.go:166` Down. All three ignore or pass through the report, so none is left wrong.
  - Docs: `orchcli/doc.go:45-50` and the `orchcli/stop.go` doc are updated; `orchengine/doc.go:15` and `pawlengine/beamdown.go:1` still hold.
  - `pawl-recipe.yaml:75`, `shedrecipe/entries_pawl.go:100-112` and `loomshed/interruptpolicy.go:87` (Reinvoke) stay valid, since a second call is idempotent.
  - Test `beamdown_test.go` is updated. Covered.
- **`hibernateengine`: confirmation constants exported, `awaitConfirmation` signature changed, `PauseAndConfirm`, `HubPause` and `PauseConfirmRequest` added.**
  - No reference to the old lower-case names or the old signature remains outside `watch.go`.
  - `hibernateengine/doc.go:116+` and the `watch.go` header are updated, and `hibernatecli/doc.go` gains a section.
  - The tree line at `docs/overview.md:385` for `hibernatecli` is not updated (NIT below).
  - The module line at `docs/overview.md:850-856` still holds. Covered.
- **`hibernatecli.PauseHub` (new).** Its caller is `beamcli/hubstop.go`, and `hibernatecli/doc.go` documents it. Covered.
- **`reedcli.down` extracted, `DownWorktree`, `DownReport` and `wireLocation` added.**
  - The verb's envelope keys `session`, `ended_processes` and `abandonedSession` are unchanged.
  - `reedcli/doc.go` is updated, and the `downCmd` Long text still holds.
  - Covered.
- **Reed templates gain `stop` in the prime group.**
  - Both `template_posix.yaml:77-79` and `template_windows.yaml:77-79` carry identical text, and `config_test.go` is updated.
  - No other doc lists the prime group's entries. Covered.
- **`lym beam stop --all` and the hidden `--detached` flag.**
  - `beamcli/cli.go` Long and Example, `beamcli/doc.go:9` and `:325-352`, and `docs/overview.md:800` are updated.
  - No stencil, spec or skill under `contracts/` or `plugins/` mentions `beam stop`, so none goes stale.
  - The `beamcli` row of `pattern/PATTERN-cli-cobra.md:63` is not updated (LOW below).
- **Repository rules.**
  - PATTERN-child-processes: the spawn uses `selfexec.LymBinary`, is logged, and is waited on through `proc.Reap`. Covered.
  - PATTERN-refusal-way-forward: `beamcli` is not a bound module, and the new refusals still carry a `way forward:` clause.
  - Declared PATTERN changes: the directive declares none, so there is nothing to land.
  - Commit messages: no `Pattern-Departure:` trailers, and no message contradicts the diff.

**Findings:**

1. **LOW**, class consistency. Location: `pattern/PATTERN-cli-cobra.md:63`.
   - The `beamcli` deviation row still lists only `fabricengine`, `boardengine`, `hubgeom` and `battencli`.
   - The diff makes `beamcli` call `hibernatecli`, `hibernateengine`, `reedcli`, `tendercli` and `tenderengine` too. Evidence: the imports at `internal/beamcli/hubstop.go:146-158`.
   - So the rule document's description of the `cli` package's dependencies grows more stale.
   - The row already omitted `orchcli`, `adoptcli` and `dobbyengine` before this diff.
2. **NIT**, class consistency. Location: `docs/overview.md:385`.
   - The tree line for `hibernatecli` lists `request`, `cancel`, `status`, the hub wiring, the carrier and the `systemctl` spawn.
   - It does not name the new `PauseHub` that `lym beam stop --all` calls. Evidence: `internal/hibernatecli/deps.go:69-79`.

Outside my lens, for the handler: `summary.md` says the shipped-buttons test pins `stop` "in both templates". The test reads the template for the running GOOS (`config_test.go:117` region, redundancy note "per GOOS"), so on Linux only the posix template is pinned.

Tool calls:
1. grep of the repo for `beam stop`, `BeamDown` and prime-button mentions in md, go and yaml, to find callers, docs and stencils beyond the manifest.
2. grep of the docs and `orchcli`, `reedengine` and `reedcli` docs for `EndRole` and prime-group wording, plus the `orchcli/stop.go` and `pawlengine/beamdown.go` headers and the old hibernate symbol names, to check doc staleness and leftover references.
3. grep of `docs/overview.md` for buttons and hibernate, of the PATTERN files for `beamcli`, and of `contracts` and `plugins` for `beam stop`, to check the overview lines, the cli-cobra deviation row and stencil wording.
