---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: handler
    location: internal/shuttleengine/codexengine/home.go:109
    summary: "The engine derives the weft writable root itself by reading the anchor's .lym and _lym link targets, where D6 and PATTERN-cwd-resolution say it is told an absolute path"
    fix: "Stop deriving the root inside codexengine. Remove the link reading from `writableRoots` in `internal/shuttleengine/codexengine/home.go` (and its `fslink.IsLink` and `fslink.PointsTo` calls). Hand the engine the absolute weft checkout path as told geometry: add a field for it to the value `NewFromConfig` receives alongside `dirs` (or to `configengine.Dirs` if that is where the other told paths live), fill it in `internal/shuttleengines/shuttleengines.go`'s `New` from the caller's hub geometry (`hubgeom`'s records worktree path for the worktree), and leave it empty outside a hub so no writable root is added. `Prepare` in `codexengine.go` then passes that one path to `launchLine.writableRoots` when it is non-empty. Update `TestPrepare_Launch` and the fixture to supply the path through the told value instead of through symlinks, and update the `codexengine.go` Prepare doc comment, `doc.go` and `_lym/landing/summary.md`'s Design decisions entry that says Prepare is not told the weft checkout. If the parent instead rules the derivation acceptable, the commit that makes it must carry a `Pattern-Departure: PATTERN-cwd-resolution` trailer giving that reason."
  - id: F2
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: internal/modelspec/template.go:14
    summary: ConfigTemplate doc comment still lists only sonnet, opus, haiku and fable though the seed template now also holds sol, astra and luna
    fix: "In `internal/modelspec/template.go`, reword the `ConfigTemplate` doc comment so it no longer enumerates the aliases, for example `ConfigTemplate returns the seed content for models.yaml: the live-keys registry with its effort defaults that configreg materializes when models.yaml is absent.`"
  - id: F3
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/shuttleengine/codexengine/prepare_test.go:40
    summary: Prepare has no test for the three launch shapes the diff adds code for outside a linked worktree with real link targets, namely real .lym and _lym directories (no writable root), a plain clone (one trust root), and a rules directory under an anchor that differs from the worktree root
    fix: "In `internal/shuttleengine/codexengine/fixture_test.go` add a worktree builder variant with real `.lym` and `_lym` directories and a plain clone with no main checkout, and add rows to `TestPrepare_Launch` (launch line carries no `sandbox_workspace_write.writable_roots`), `TestPrepare_ReadiesHome` (only the worktree root is trusted) and `TestPrepare_RefusesBeforeWriting` (a `.codex/rules` directory under an anchor that is a subfolder of the worktree root refuses). Add them as rows of the existing tests, not as new test functions."
---

### [BLOCKING] Weft writable root is derived inside the engine, not told to it

**Location:** `internal/shuttleengine/codexengine/home.go:109`, `internal/shuttleengine/codexengine/codexengine.go:101`
**Issue:** D6 says the writable root is the worktree's weft checkout, "which the engine is told as an absolute path", and the parent decision "Weft checkout as a Codex writable root" asks for the checkout the `.lym` and `_lym` links point into.
`PATTERN-cwd-resolution` (Told geometry) says an engine is handed the absolute paths it operates on and derives none of its own.
The diff instead has `writableRoots` read the anchor's `.lym` and `_lym` symlinks with `fslink.IsLink` and `fslink.PointsTo` and take `filepath.Dir` of each target; `_lym/landing/summary.md` records this as a choice ("Prepare is not told the weft checkout").
No commit carries a `Pattern-Departure:` trailer, so the derivation is not licensed.
The derivation also differs from the decision in effect: fabric's `WarpJunctions` links `<anchor>/.lym` to `<records worktree>/<AnchorRel>/.lym`, so for any anchor with a non-`.` `AnchorRel` the root is the weft's `<AnchorRel>` subfolder and not the weft checkout.
The write the session needs still works, but the root is narrower than the decision names, and the engine hard-codes a fabric layout that only fabric and `hubgeom` should know.

### [LOW] ConfigTemplate comment lists four aliases

**Location:** `internal/modelspec/template.go:14`
**Issue:** The doc comment of `ConfigTemplate` still names `sonnet/opus/haiku/fable` as the seed registry.
The diff added `sol`, `astra` and `luna` to `template.yaml` and reworded the neighbouring comments in `load.go`, `registry.go` and `template_test.go`, but left this one.

### [LOW] Launch shapes outside the fixture's linked worktree are unpinned

**Location:** `internal/shuttleengine/codexengine/prepare_test.go:40`
**Issue:** Every fixture worktree has symlinked `.lym` and `_lym` and a main checkout (`fixture_test.go`, `addWorktree`).
The code paths for real `.lym` and `_lym` directories (`writableRoots` returns none, so the launch line omits the writable-roots flag; `home.go:117`), for a plain clone (`trustRoots` returns one root; `home.go:49`, `home.go:72`) and for a `.codex/rules` directory under an anchor that differs from the worktree root (`home.go:85`) run in no `Prepare` test.
The decision record and the summary both state the first two behaviors, so a regression there would pass the suite.

## Rejected

- Lens code-correctness F1, `TurnStartInterrupt` fallback reads a stale abort: the unbounded case needs both a missing turn id and a zero stamp time, while the spike payloads carry turn ids and a missing stamp time is the fail-open path the code documents; with a nonzero time, events older than the turn start are skipped.
- Lens code-correctness F2, hook stamps lost when two hooks interleave: the hooks run one at a time per turn event, the lens itself finds sequential hooks correct, and a lost stamp only yields a zero signal time, which the parser documents.
- Lens code-tests F4, no test for an invalid `codex.yaml` or a relative anchor in Prepare: both are operator-config and caller-wiring failures that propagate unchanged, and `TestLoadConfig_Template` and `TestExpandHomePath` pin the code that raises them.
- Lens code-tests F5, a configured `binary` reaching the launch line: a one-line pass-through of a string with no branch logic beyond the empty default, which `TestPrepare_Launch` covers.
- Lens code-tests F6, `CheckAfterStartup` failure paths: each only logs a Warn and returns, and the one behavior that matters, warning on a lost root, is pinned.
- Lens code-tests F7, `ensureLink` kept-link path asserts no warning on a re-run: `TestPrepare_ConcurrentLaunchesLoseNoRoot` re-runs Prepare over an existing home and a regression would surface in the Warn-bearing `TestPrepare_Warnings` rows.
- Lens code-tests F8, constant-returning methods have no row: these return fixed values from a documented capability contract, and a per-getter test would only restate the constants.
- Lens code-tests F9, Windows-only assertions and the link-before-write ordering: the Windows refusal is pinned by `TestCreateFileLink` where it can run, and the ordering cannot be observed on Linux without injecting a failing link, which the code does not offer.

## Checks that held

- D4: `codex` is registered with `codex.yaml`, the `CODEX_` prefix is scrubbed and mirrored in `tmuxkit.ServerEnviron`, and `sol`, `astra` and `luna` resolve from the built-in fallback and the seeded template at effort `medium`; an effort outside `efforts` is refused naming the key.
- D5: the home's links, the lock, the atomic render from the record, the pruning, the Warns and the rules refusal match the Sketch; the check after startup is wired through the optional `StartupChecker` capability, which existing engines do not implement.
- D6 and D7: the launch line carries the spike's flags, the writable-roots setting, the hooks and the shell exit, the resume line starts no Codex, and the recording hooks and rollout reading match the Sketch.
- D8: the pane readings, trust accept, paste path and interrupt guard match the Sketch.
- D13 and `PATTERN-fs-links`: `CreateFileLink` links on Linux and refuses first on Windows, and `PATTERN.md` and `pattern/PATTERN-fs-links.md` land the declared entry text.
- The diff touches no decision outside D4 to D8 and D13; the prompt file leaves D10's notice and the skill-load turn to the other parts, as the summary says.
- No commit carries a `Pattern-Departure:` trailer.
