# Card 2 — reed.yaml's window_share_max and window_split keys and their validation

**Create:**
- `internal/reedengine#windowSplitLayout`
- `internal/reedengine#validateWindowConfig`

**Edit:**
- `internal/reedengine#Config`
- `internal/reedengine#ConfigSpec`
- `internal/reedengine#Engine.validateBootConfig`
- `internal/reedengine#Engine.AddStrandUnless`
- `internal/reedengine#Engine.ReplaceStrand`
- `internal/reedengine/template_posix.yaml`
- `internal/reedengine/template_windows.yaml`
- `internal/reedengine#TestUp_BootValidation`
- `internal/reedengine#TestAddStrand_UnformableName_RefusesBeforeAnyTmuxCommand`
- `internal/reedengine_test#TestLoadConfig_TemplateDefaultsResolve`

**Uses:**
- `internal/reedengine#Engine.withBootOpLockKeepingZoom`
- `internal/reedengine#Engine.validateNaming`
- `internal/reedengine#hiddenSpec`

**Intent:**
D3's two settings, as ordinary per-worktree `reed.yaml` keys resolved through `configengine`'s layers (`PATTERN-config-layers`, `PATTERN-config-strictness`), so a hub-layer value reaches every worktree and `reed` stays a non-hub-wide module.

`Config` gains:

- `WindowShareMax int \`yaml:"window_share_max"\``: the largest declared group size whose strands share one window (card 5 reads it).
- `WindowSplit string \`yaml:"window_split"\``: the direction a shared window splits in.

`ConfigSpec` classifies both keys `configengine.CategoryNone`.

Both templates gain the two keys after `operator_quiet_s`, each with a comment in the house style:

- `window_share_max: 2  # the largest group of strands, as their spawner declares it, that share one window; a larger group gives each strand a window of its own, and 0 gives every grouped strand its own; an integer of at least 0, and a negative value fails "lyx reed up", "lyx reed resume" and every add, naming the key; a group keeps the choice its first strand made, so an edit reaches only groups started after it; an already-materialized reed.yaml lacking this key takes this default until "lyx config reconcile" writes it`
- `window_split: width  # how a shared window lays out its strands: width puts them side by side, height stacks them; any other value fails "lyx reed up", "lyx reed resume" and every add, naming the key, and a remove still removes and reports it; an already-materialized reed.yaml lacking this key takes this default until "lyx config reconcile" writes it`

In `internal/reedengine/config.go`:

- `windowSplitLayout(raw)` maps `width` to the split flag `-h` and the layout `even-horizontal`, and `height` to `-v` and `even-vertical`.
  The empty string reads as `width`, the template default, so a `Config` built in code without the key (every test fixture that constructs one) keeps working; see the Shared Decision `empty-window-split-is-width`.
  Any other value returns `invalid window_split value %q: want "width" or "height"`.
- `validateWindowConfig(cfg)` returns `windowSplitLayout`'s error, then, for `cfg.WindowShareMax < 0`, `invalid window_share_max value %d: want an integer of at least 0`.
  Both messages name their key, so the operator knows which line to fix.

Where it is checked:

- `validateBootConfig` calls `validateWindowConfig` after the segment colors, so `Up`, `Resume` (through `withBootOpLockKeepingZoom`, which calls `validateBootConfig` before its zoom read) and every boot via `ensureServerAndSessionLocked` refuse an invalid value before any tmux round trip.
  Its doc comment names the two keys among those it validates.
- `AddStrandUnless` and `ReplaceStrand` call `validateWindowConfig` right after `validateNaming`, before the session pre-flight, so an add refuses without booting a server or issuing any tmux command.
  `reed.yaml` is loaded per invocation, so an edit on a live session surfaces at the next add.
  The check covers every add, window 0 adds included (the driver, the orch, the runs and status strands), as D3 names add without narrowing it: a `reed.yaml` typo blocks every spawn until it is fixed, which the refusal names the key for.
- Remove never calls it (card 6 reports an invalid value instead).

Tests:

- `TestUp_BootValidation` gains rows `WindowSplitInvalid` (`cfg.WindowSplit = "diagonal"`, `wantErr: "window_split"`), `WindowShareMaxNegative` (`cfg.WindowShareMax = -1`, `wantErr: "window_share_max"`), `WindowSplitHeightPasses` (`notErr: "window_split"`) and `WindowShareMaxZeroPasses` (`notErr: "window_share_max"`).
  Its `wantErr` branch also calls `e.Resume()` and asserts that error contains `wantErr` with no tmux call, beside `Up()` and `ensureServerAndSessionLocked()`.
  The doc comment names the two keys and the Resume entry.
- `TestAddStrand_UnformableName_RefusesBeforeAnyTmuxCommand` gains a `configure func(cfg *Config)` row field (nil for the existing rows) and rows `InvalidWindowSplit` and `NegativeWindowShareMax`, with a formable shortname and slug, asserting that `AddStrand` and `ReplaceStrand` both fail with the key named and no tmux command issued.
  Its doc comment says it pins every up-front refusal of add and replace.
- `TestLoadConfig_TemplateDefaultsResolve` asserts `WindowShareMax == 2` and `WindowSplit == "width"`, and its `//testtiming:keep` reason names the two keys among the defaults it pins.

**ImpactSummary:** Adds two config keys and one validator called from the boot path, add and replace; a test fixture building `Config` without the keys keeps passing because the zero values are valid.

**Commit:** 2: reed.yaml gains window_share_max and window_split, validated at up, resume and add
