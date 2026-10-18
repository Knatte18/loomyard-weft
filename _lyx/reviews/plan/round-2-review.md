---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: decision
    location: _lyx/plan/15-command-engine-replaces-switched-rows.md:102
    summary: "The declared PATTERN-recipe-setting-lookup text makes a 321-character entry line, one over internal/pattern's 320-character cap, so go test ./internal/pattern fails after card 15 and no card handles it"
    fix: "No fix touching the overview or the targets is admissible, because the declared text is fixed by the decision record and the plan gate's pattern-changes-mismatch check binds the overview's `## PATTERN changes` to it. Add to card 15's Docs paragraph a stated resolution: the line `- `PATTERN-recipe-setting-lookup` — <declared text>` is 321 characters against `MaxEntryLineChars` 320 in `internal/pattern/check.go`, so card 15 writes the declared text minus the single word 'told' (`the owner's executable-key lookup`, 316 characters), names that one-word departure in its Intent with the cap as the reason, and asks `ly:orch` to amend the decision record's declared text to match; leave the overview's `## PATTERN changes` line as it is. Card 10's byte arithmetic then drops 5 bytes. If the fixer finds the departure not admissible, it defers this finding naming the constraining check (pattern-changes-mismatch) and the cap."
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/plan/10-status-seams-spawn-detached-push.md:78
    summary: Card 10 obliges a PATTERN.md cut of the whole plan's growth but names no entry or wording to cut, so nothing in the plan shows the cut exists and the fallback is a mid-card question to ly:orch
    fix: "In card 10, replace the open obligation with a named cut: list the entries outside the three declared ones and the shortened text of each, chosen so that `wc -c PATTERN.md` after card 15's additions is at most the merge-base size 15556 (the headroom line, `HeadroomBytes`, is 15564, so only 8 bytes of growth are free). The growth to cover is the `PATTERN-push-both-sides` lengthening (70 bytes), the `PATTERN-recipe-setting-lookup` lengthening (117 bytes) and the new `PATTERN-command-row` line (about 190 bytes, plus about 49 more if a background link is kept, see F4), so a cut of at least 370 bytes without the link. Each shortened entry keeps its rule and its `(test)` marker and background link, and `go test ./internal/pattern` stays green. Drop the 'ask ly:orch' fallback or make it an explicit stop before any edit."
  - id: F3
    severity: MEDIUM
    class: scope
    location: _lyx/plan/15-command-engine-replaces-switched-rows.md:98
    summary: Card 15 says nilLegal drops the switched paths, but envkit's nilLegal holds none, while the two edits that matter are missing, so TestFullEnv_FillsEverySeam and TestWire_EverySeamFilled fail on the nil Command.Setting and Command.ExecutableSetting
    fix: "In card 15's last test bullet, replace 'nilLegal and loom's intentionallyNil drop the switched paths' with: `envkit.nilLegal` gains `Command.Setting` and `Command.ExecutableSetting` (commandEntry fills both from `Env.Setting` and `Env.ExecutableSetting` at build, so `Env.Command` never holds them), plus any other `Command` field that a wired env leaves nil by design, such as `Command.Forced` on an unforced run; loom's `intentionallyNil` drops its six `OpenWindow.*`, `SeedIDE.*` and `OpenIDE.*` entries and gains the `Command.*` paths that loom's wire leaves nil; battencli's `wire` fills the rest of `Env.Command` so `TestWire_EverySeamFilled` passes with no new `intentionallyNil` entry. State which `Command` fields `FullEnv` fills."
  - id: F4
    severity: LOW
    class: consistency
    location: _lyx/plan/10-status-seams-spawn-detached-push.md:78
    summary: Card 10 counts a background link for the new PATTERN-command-row entry, but card 15 writes the entry without one and no card creates pattern/PATTERN-command-row.md, which a link would need
    fix: "In card 10, drop 'with its background link' from the growth sentence and the 'about 425 bytes' tally, or have card 15 add `plan:pattern/PATTERN-command-row.md` to its Create list and write the link into the entry. Prefer dropping the link, since the declared text carries no background and `internal/pattern`'s Check refuses a link whose file is missing."
  - id: F5
    severity: LOW
    class: scope
    location: _lyx/plan/00-overview.md:44
    summary: The verify block omits internal/loomrecipe, whose tests parse contracts/recipes/batten-recipe.yaml, a file card 15 edits
    fix: "Add `./internal/loomrecipe` to the end of both `go test` lines of the `## verify:` block in 00-overview.md, after `./internal/lyxcwd`. The overlay run of `lyx loom validate-plan --overlay` with that change reports no finding beyond the baseline."
  - id: F6
    severity: LOW
    class: decision
    location: _lyx/plan/08-push-lock-pending-flags.md:36
    summary: Card 8 says a pending flag holds the JSON of a PushSpawn but does not say whether it carries WorkDir, and its bound validates only the side's path, not the slugs or the working directory a respawn trusts
    fix: "In card 8's Intent, state that a flag stores the side's path and the two slugs only, and that a holder's respawn sets `WorkDir` itself to `BoardDir` of the hub (the parent of the records worktree the lock directory belongs to), ignoring any stored value; add a `TestCoalesceFlagged_PendingFlags` assertion that the respawned request's `WorkDir` is the board directory even when the flag file holds another one. Without it a foreground pusher's flag, which carries no WorkDir, is respawned with the holder's own working directory, possibly a pair worktree, against D1."
  - id: F7
    severity: LOW
    class: scope
    location: _lyx/plan/15-command-engine-replaces-switched-rows.md:24
    summary: Card 15 promises a way forward clause on every commandEntry refusal, but a non-string switch or command value reaches configString's error, which carries none, and no TestCommandEntry row reaches it
    fix: "In card 15's `TestCommandEntry` bullet, add a row with a non-string `switch` (for example `3`) and assert the message carries the row, the key and a `way forward:` clause, and say in the Intent that commandEntry wraps the typed-accessor errors for `switch`, `command` and `timeout_s` with that clause."
---

### [BLOCKING] Declared PATTERN-recipe-setting-lookup line exceeds the entry line cap

**Location:** `_lyx/plan/15-command-engine-replaces-switched-rows.md:102`, `_lyx/plan/00-overview.md:41`

**Issue:** `internal/pattern/check.go` caps an entry line at `MaxEntryLineChars` = 320 runes, and `TestOverview_RealFilePassesCheck` runs `Check` over the real `PATTERN.md`.
The current `PATTERN-recipe-setting-lookup` line is 204 characters.
The declared text adds 117, which makes 321 characters with the `- ` prefix, the entry name and the em dash.
Card 15 prescribes exactly the declared text, so its gate and the `internal/pattern` package in `## verify:` fail.
The plan gate does not see this, because the line is in prose and not in a fenced block that `pattern-entry-line-cap` scans.
The other two declared lines fit: `PATTERN-push-both-sides` with its link is 214 characters and `PATTERN-command-row` is 188.
The text is the decision record's, so the plan cannot simply change it; that is why this is a `decision` finding.

### [MEDIUM] Card 10's PATTERN.md cut is unnamed

**Location:** `_lyx/plan/10-status-seams-spawn-detached-push.md:78`

**Issue:** This settles the first focus question.
`PATTERN.md` is 15556 bytes at the merge base, `HeadroomBytes` is 15564, and the three declared changes add about 378 bytes (70, 117 and 191 with the newline).
Card 10 correctly puts the cut before the growth and makes it cover the whole plan's growth, so card 15 stays under the line.
But it names no entry, no wording and no byte count of the cut, and offers a question to `ly:orch` mid-card if no cut exists.
The decision record declares only three PATTERN changes, so a cut across other entries is scope the answer key does not license, and the plan leaves its feasibility to the implementer.
No other card cuts bytes.
The longest entry outside the three is 235 bytes and most are 190 to 200, so a cut of 370 bytes is plausible but unshown.

### [MEDIUM] Card 15's NilSeams edit is wrong and incomplete

**Location:** `_lyx/plan/15-command-engine-replaces-switched-rows.md:98`

**Issue:** `envkit.NilSeams` recurses into nested structs and reports every nil func, map and pointer field not in `nilLegal`.
`Env.Command` is a `CommandDeps` whose `Setting` and `ExecutableSetting` are filled by `commandEntry` at build, never in `Env`, so they are always nil in `FullEnv` and in battencli's wired env.
`TestFullEnv_FillsEverySeam` and `TestWire_EverySeamFilled`, both in `## verify:`, would report `Command.Setting` and `Command.ExecutableSetting`.
The card instead says `nilLegal` drops the switched paths, and `nilLegal` has none (the switched seams are listed only in loom's `intentionallyNil` and set in `FullEnv`).
The rest of the `Command` struct (`Forced`, and the func-typed facts) has the same exposure on an unforced or loom-wired env.

### [LOW] Background link counted but never created

**Location:** `_lyx/plan/10-status-seams-spawn-detached-push.md:78`

**Issue:** Card 10 sizes the cut for "its background link", while card 15's Docs paragraph adds the `PATTERN-command-row` entry with its declared text and no link, and no card lists `pattern/PATTERN-command-row.md`.
An implementer who follows card 10 and writes the link fails the link check in `internal/pattern`; one who follows card 15 writes none and over-cuts by about 49 bytes.

### [LOW] Verify block misses a reader of the edited recipe

**Location:** `_lyx/plan/00-overview.md:44`

**Issue:** This settles the fifth focus question: the block covers every package whose tests read `PATTERN.md` (`internal/pattern`, `internal/lyxcwd`, `cmd/lyx`, `internal/fabriccli`, `internal/fabricengine`, `internal/loomcli`, `internal/shedadapters`) and `docs/` (`internal/lyxcwd`'s link scan), and `cmd/lyx`'s `sandbox_coverage_test.go` covers the suite file card 4 edits.
The one gap is `internal/loomrecipe`, whose `TestRowNames_FollowEachRecipesRowOrder` and entry-number tests parse `recipes.BattenRecipe`, the embedded `contracts/recipes/batten-recipe.yaml` that card 15 edits.
The row names do not change, so the tests should pass, but the package is neither targeted nor run.

### [LOW] Flag content bound leaves WorkDir and slugs open

**Location:** `_lyx/plan/08-push-lock-pending-flags.md:36`

**Issue:** This settles the third focus question.
The Intent does state what the try-lock and flags let through (a hung holder delays later pushes; a holder respawns whatever request a flag holds) and bounds the flag's side path to the lock directory's own worktree.
It does not bound the flag's slugs or `WorkDir`: the content is "the JSON of the single-side `PushSpawn`", whose fields include `WorkDir`, while the same sentence lists only the path and the slugs.
`SpawnStatusPush` always sets `WorkDir` to the board directory, but the foreground `CoalescePushBothAt` writes flags with no slugs and no `WorkDir`, and a holder that respawns one starts a child with an empty `cmd.Dir`, which inherits the holder's directory.

### [LOW] An untested commandEntry refusal path

**Location:** `_lyx/plan/15-command-engine-replaces-switched-rows.md:24`

**Issue:** This settles the fourth focus question.
Every refusal the card names ends in a `way forward:` clause and `TestCommandEntry` asserts the row, key and clause for a missing `switch`, a missing `command`, a reference with no dot, `timeout_s: 0`, a non-int `timeout_s` and an unknown key.
`configString` (used through `settingRef` for the references) returns "must be a string, got %T" for a non-string value with no clause, and `configInt` does the same for a non-int; the card wraps the latter case but names no wrap for the former and has no row that reaches it.

## Rejected

- Pause gate, log sink and hub on a subpath-anchored hub (second focus question): sound as planned.
  `lyxcwd.ResolveWorktree` over the `_board` root skips the cwd gate that `Resolve` applies against the recorded anchor, `BoardLyxPresent` and `PauseGeometry` then work from that location, and card 9 sets the sink directory explicitly with `logger.SetDurableSinkDir`.
  A push with an empty slug carries `--pause-slug=`, `pauseSlugOverride` returns a pointer to the empty string, and the gate's geometry slug is replaced with `""`, the prime's own entry, which is what a prime push uses today; card 7's `TestInstallNetworkGate_...` rows pin both the empty slug and the `Shape{Anchor: "backend"}` hub.
- The `PATTERN-push-both-sides` and `PATTERN-command-row` declared lines fit the line cap (214 and 188 characters), and the growth of `PATTERN.md` is the only headroom exposure; the headroom lint compares against the merge base, so it is met when the plan's net growth is at most the 8 free bytes.
- `tools/sandbox` tests embed the suite files that card 4 edits and compare them to the same on-disk file, so they are unaffected and need no verify entry.
- Card order: every card's `Uses` of a `plan:` symbol points at an earlier card (7 uses 2 and 4; 8 uses 7; 9 uses 7 and 8; 14 uses 11; 15 uses 12 to 14).
