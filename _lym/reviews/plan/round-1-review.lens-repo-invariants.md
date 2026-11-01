```yaml
lens: repo-invariants
outcome: reported
```

Repo-invariants lens over cards 1 to 9: test seams, new-file tier tags, doc text a card falsifies, and card order.

**Per card, each invariant checked:**

- **Card 1** (`shell-detached-forms`):
  - Seam: held. Only `posixShell` and `pwshShell` implement `Shell`. A grep for `DiscardLineKey()` outside `internal/shell` finds only callers (`reedengine/spawn.go:121`, `spawn_test.go:500`), no implementer.
  - Tier: held. The tests stay in `shell_test.go`, which is `tier0`.
  - Doc: broken (finding 1). The `shell.go` package header lies outside the targeted members. The `shell_test.go:4` header is falsified (finding 2).
  - Order: held.
- **Card 2** (`buttons-config-keys`):
  - Seam: held. I found no reflective seam test over reed's or desktop's `Config`. The comparability change is handled in the card.
  - Doc: held. `docs/overview.md` and `reedengine/doc.go` are listed.
  - Order: held.
- **Card 3** (`button-entries`):
  - Tier: held. One new `tier0` file.
  - Doc: held. `doc.go` is extended.
  - Order: held. Its uses are `Config` (card 2) and `Shell`.
- **Card 4** (`geometry-button-context`):
  - Seam: held. No `//lym:seam` or reflective test over `reedengine.Geometry` in `reedengine`, `hubgeom` or `standalonegeom`.
  - Doc: broken (finding 1). The `geometry.go` file header is edited, but only the `#Geometry` member is targeted.
  - Order: held. `ButtonPlace` comes from card 3.
- **Card 5** (`third-status-line`):
  - Doc: broken (finding 1). The `bar.go` and `windowsize.go` file headers are edited, but only members of those files are targeted.
  - Order: held. `TestAttachArgv` goes 12 → 13 (`attach_test.go:236` reads `12 + wantShellPins`). The pin counts of 8 and 9 match `windowsize_test.go:251` and `:310`.
- **Card 6** (`render-button-row`):
  - Tier: held. The new file is `tier0 && !windows`, which is one tier tag plus an OS build constraint, not a skip.
  - Doc: held. `bar.go`, `follow.go` and `doc.go` are listed as files.
  - Order: held. 13 → 14. The row pin is one `set-option` invocation. It uses cards 3, 4 and 5.
- **Card 7** (`strand-name-pane-option`):
  - Doc: broken (finding 1). Its Intent rewrites the `paneoptions.go` header (`paneoptions.go:1-3`, "No Go decision reads them back"), but its only target in that file is `#Engine.setStrandPaneOptionsLocked` (`paneoptions.go:28`).
  - Order: held. 14 → 15.
- **Card 8** (`reed-button-verb`):
  - Tier: held. One `tier0` file and one `tier2 && linux` file, like the existing integration tests.
  - Doc: `reedcli/doc.go` and the `stencilseedgate` header are covered. The `cli.go` header is NIT-level stale (finding 3).
  - Order: held. It uses cards 3, 6 and 7.
  - Unverified: `cmd/lym/spawnobservability_test.go` exists and enforces `PATTERN-spawn-observability`. I ran out of tool calls before reading it, so I can't say whether a new `exec` site in `reedengine` (`runButtonScript`) needs an entry that no card lists.
- **Card 9** (`button-click-binding`):
  - Doc: broken (finding 1). The `bindings.go` file header is edited ("The file's header comment names the button branch"), but only members are targeted.
  - Order: held. 15 → 16. Every caller of the two re-signed functions is listed.

**Findings:**

1. **MEDIUM, design. Cards 1, 4, 5, 7 and 9: file header comments edited outside the declared targets.**
   - What: each card's Intent rewrites a file's header comment, but its targets name only members of that file.
     - Card 1: the `internal/shell/shell.go` package header.
     - Card 4: the `internal/reedengine/geometry.go` header.
     - Card 5: the `bar.go` and `windowsize.go` headers.
     - Card 7: the `internal/reedengine/paneoptions.go` header.
     - Card 9: the `internal/reedengine/bindings.go` header.
   - Effect: the header edit falls outside every target span, so a target-scoped executor or drift check either skips it, leaving the header false, or flags it as out of scope.
   - Evidence: `paneoptions.go:1-3` against its function at line 28; `bindings.go`'s binding doc at lines 103-109; the card 7 and card 9 Intent texts.
   - Fix: none admissible as a plain pairing. Adding the file beside a member of it on one card is refused by `redundant-file-target`, and swapping members for the bare file draws the member-glyph rule. The fix needs a shape the plan gate admits, such as moving the header edits to a card that already targets the file, as card 6 does for `bar.go`.
2. **NIT, consistency. Card 1: the `shell_test.go:4` file header says "One test per Shell method".**
   - The new `TestShell_DiscardOutputAndCommandFlag` covers two methods, and card 1 does not update the header.
   - Mitigation: `TestShell_ScriptExtAndDiscardLineKey` (`shell_test.go:268`) already breaks the header's claim.
3. **NIT, consistency. Card 8: `internal/reedcli/cli.go:5-6` header names the `switch` verb as the one pre-run exception.**
   - Card 8 adds `button` to the skip list but targets only `#Command`.
   - Mitigation: the header was already stale before this plan, since `follow` and `window` are also exempt (`cli.go:142`).

**Tool calls:**
1. Grepped for `lym:seam` and reflection tests and for `Shell` implementers. The manifest named no seam tests and no `Shell` implementers outside the package.
2. Read the `DiscardLineKey` and `Shell` use sites in `reedengine` and the `Geometry` literals in the tests, to confirm no second `Shell` implementer and no field-reflecting test.
3. Grepped the docs, `PATTERN.md`/`pattern/` and `hubgeom/doc.go` for bar and pane-option prose, and read the `paneoptions.go` header. The manifest did not include the pattern docs or that header.
4. Grepped for enumerations of the reed verbs outside `reedengine` (CLI and stencil-seed tests), to find the full set of verb lists the card must update.
5. Read the `reedcli/cli.go` header, `reedcli/doc.go` and the current pin counts in `attach_test.go`/`windowsize_test.go`, to check the card-order count chain.
6. Listed `PATTERN.md`/`pattern/` and the spawn-observability enforcement tests, to check for an exec-site allowlist. I located `cmd/lym/spawnobservability_test.go` but did not read it.
