---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    location: internal/hubgeom/hubgeom.go:29
    summary: ReedGeometry swallows a PrimeName failure and tells a prime geometry, against card 6, and its git spawn reaches untagged wiring tests
  - id: F2
    severity: LOW
    location: internal/hubgeom/hubgeom.go:16
    summary: Comment-convention breaches in new and changed Go comments, which are fixed-column wrapped mid-phrase instead of using semantic line breaks
  - id: F3
    severity: LOW
    location: internal/loomengine/driver.go:18
    summary: Driver and status strand docs still claim a second add under the same name appends a second pane, but reed now refuses a held explicit name
  - id: F4
    severity: LOW
    location: internal/reedengine/namerepair.go:132
    summary: paneIsLive and directoryRows restate the existing aliveIDSet helper instead of calling it
  - id: F5
    severity: LOW
    location: internal/reedcli/list.go:1
    summary: list.go file doc calls the verb the diagnostic for the session-name mirror too, but its rows carry pane-title drift only
  - id: F6
    severity: LOW
    location: docs/overview.md:302
    summary: The fabric entry's new --code clause separates reconcile backfills that binding from the warp binding it refers to
  - id: F7
    severity: NIT
    location: internal/reedcli/remove.go:98
    summary: remove's Long help has one overlong line and its exactly-one-of-the-two parenthetical now trails the name examples
  - id: F8
    severity: NIT
    location: internal/agentname/agentname.go:146
    summary: Resolve builds a Name literal with an explicit empty Role just to call prefix, where n.prefix() says the same
---

Review range: `git diff $(git merge-base main HEAD)..HEAD` (`product.parent` is `main` in `_lyx/shed/agent-naming/status.json`; merge base `9ae5f9405`), cards 1–16 plus `9eb0fa3e5`, `4ddf3665e` and `d7a344670`.

Evidence gathered: the committed HEAD, exported with `git archive` into a scratch directory, passes `go build ./...`, `go vet ./...`, `go test ./...` (apart from two `websterengine` tests that need a `.git` the export lacks; they pass in the worktree), the full `-tags integration` package list from the plan's `verify:` block, and `go vet -tags smoke ./...`.

The worktree also carries an uncommitted edit to `internal/hubgeom/hubgeom.go`, which is outside the review range.
It makes `ReedGeometry` return the `PrimeName` error, and with it applied 33 untagged wiring tests fail in `burlercli`, `loomcli` and `webstercli` (`wire() = hubgeom: resolve main worktree: …`).
F1 covers the design question it was reaching for.

### [MEDIUM] ReedGeometry swallows a PrimeName failure and spawns git on every hub wiring path

**Location:** `internal/hubgeom/hubgeom.go:29` (also `internal/hubgeom/doc.go:14`)
**Issue:** Card 6 says "A `PrimeName` failure is the function's only error".
The committed `ReedGeometry` logs a warning instead, sets `prime = l.WorktreeName`, and returns the geometry of a prime.
In a task worktree where `git worktree list` fails, every strand is then born `<code>:<role>` with no `ParentName`.
That is a prime-shaped full name, and since the name is a birth attribute it stays wrong for the strand's lifetime.
It collides with the prime's own strands in `lyx reed list` and in any full-name SendMessage address.
The stated reason, that the Tier 1 wiring tests pass fictional git-less locations, also exposes a second problem.
`PrimeName` spawns `git worktree list` on every hub wiring path, so the untagged wiring tests in `loomcli` and `webstercli` now spawn git: their failure text under the uncommitted edit is git's own `fatal: not a git repository`.
That breaks the Test Tier Purity Invariant ("Untagged test files perform no expensive spawns"), even though its grep-based enforcement cannot see an indirect spawn.
`hubgeom/doc.go` lists what `ReedGeometry` reads but leaves out the git spawn.
**Fix:** Return an error whenever prime cannot be told from task, as the card requires, and decide it without spawning git.
A worktree's `.git` is a directory in the main worktree (the prime) and a `gitdir:` file in a linked one (a task worktree), and that is the same fact `PrimeName` derives.
Any other state is the error.
Give the three Tier 1 `hubLocation` helpers a `.git` directory in their fictional worktree, so they stay offline and are told a prime.
Update `ReedGeometry`'s doc and `hubgeom/doc.go` to name the stat and the error.

### [LOW] New and changed Go comments are fixed-column wrapped

**Location:** `internal/hubgeom/hubgeom.go:16`, `internal/reedengine/strand.go:22`, `internal/fabricengine/codebinding.go:1`, `internal/fabricengine/warpprobe.go:42`, `internal/shedrun/seed.go:37`, `internal/battencli/wire.go:291`, `internal/fabriccli/code.go:1`, `internal/fabriccli/clone.go:130`, `internal/fabricengine/clone.go:58`, `internal/hubforge/hub.go:33`, `internal/reedengine/byname.go:23`, `internal/standalonegeom/reedgeom.go:69`, `internal/hubgeom/doc.go:14`, `internal/reedengine/doc.go:152`, `internal/loomcli/driverspec.go:22`, `internal/shuttleengine/spec.go:91`, `internal/cliwire/doc.go:28`, and several new test files (`codebinding_clone_integration_test.go`, `reedgeom_integration_test.go`, `strand_test.go`, the two `testmain_integration_test.go`)
**Issue:** CONSTRAINTS.md's Comment Line-Break Convention requires semantic line breaks in new and changed Go comments: one sentence per line, with a further break only at an independent-clause boundary.
Many comments this diff adds or rewrites break mid-phrase at a fixed column instead.
Examples: `numbered `-2`, `-3` against the worktree's` / `strands when taken`, `the wiring functions' Tier 1 tests hand` / `ReedGeometry fictional, git-less locations`, and `reports whether a usable one` / `was found`.
Several changed lines inside legacy fixed-wrapped paragraphs (`driverspec.go`, `spec.go`, `cliwire/doc.go`, `reedengine/doc.go`) also keep the column wrap, though the rule governs every changed line.
**Fix:** Reflow each new or changed comment to one sentence per line, breaking a long sentence only at an independent-clause boundary.
Leave untouched legacy lines as they are.

### [LOW] Driver and status strand docs describe the retired duplicate-add behaviour

**Location:** `internal/loomengine/driver.go:18`, `internal/loomcli/bootstrap.go:19`, `internal/loomcli/bootstrap.go:293`
**Issue:** `LoomDriverStrandName`'s doc says a second add under the same role "would append a second pane rather than replace the first".
`statusStrandDisplayName`'s doc and `resolveStatusStrandAction`'s doc say the same.
Since card 7, an add whose explicit name is already held is refused, not appended.
Both strands are added with `NameOverride`, and the same diff's `driverSpec` doc (`driverspec.go:22`) states the refusal, so the docs now contradict each other.
**Fix:** Say that reed refuses an add whose explicit name is already held, so the add and every lookup must use the one constant, and that the bootstrap removes or replaces an existing strand before it adds.

### [LOW] paneIsLive and directoryRows restate aliveIDSet

**Location:** `internal/reedengine/namerepair.go:132`, `internal/reedengine/directory.go:56`
**Issue:** `paneIsLive` re-implements "present and not dead", which `aliveIDSet` (`apply.go:37`) already provides.
`directoryRows` repeats the same test inline.
The Create card type's check ("nothing equivalent exists first") applies to both new files.
**Fix:** Build `aliveIDSet(live)` once and test membership in both places, then drop `paneIsLive`.

### [LOW] list.go doc overstates what list diagnoses

**Location:** `internal/reedcli/list.go:1`
**Issue:** The file doc calls `list` "the diagnostic for the two display mirrors of a strand's name (pane title and session name)".
`DirectoryRow.Drift` covers only the pane title, and nothing in the verb reads a provider session name.
**Fix:** Say the verb shows each strand's pane title and whether it has drifted from the name.

### [LOW] overview.md: "that binding" lost its antecedent

**Location:** `docs/overview.md:302`
**Issue:** The new text puts "takes `--code` … `.lyx-code` on weft:main; `code [<code>]` prints the recorded code …" between "derived from the warp binding" and "`reconcile` backfills that binding".
Read in place, "that binding" now points at the code record, but `reconcile` backfills the warp binding.
**Fix:** Name the warp binding explicitly in the `reconcile` clause, or move the `--code` and `code` sentences after it.

### [NIT] remove's help text layout

**Location:** `internal/reedcli/remove.go:98`
**Issue:** The rewritten Long text has one line of about 120 columns among roughly 80-column lines, and "(exactly one of the two)" now trails the name examples, so it no longer clearly refers to `<guid>` and `--name`.
**Fix:** Rewrap the paragraph to the surrounding width and put "exactly one of the two" straight after the two alternatives.

### [NIT] Resolve's prefix construction

**Location:** `internal/agentname/agentname.go:146`
**Issue:** `Name{Code: n.Code, Slug: n.Slug, Role: ""}.prefix()` builds a copy with an explicit empty Role.
`prefix` never reads Role, so `n.prefix()` says the same.
**Fix:** Use `n.prefix()`.

## Focus items settled without a finding

- `9eb0fa3e5` stays inside the cards.
  The new `testmain_integration_test.go` is what the Hermetic Git Test Environment Invariant requires for card 10's git-spawning `standalonegeom` integration test.
  The other three hunks reword comments only ("weft" to "Board", "warp worktree" to "task worktree and the prime").
  It adds no behaviour.
- Card 7: `validateNaming` runs before `ensureSessionLocked` in `AddStrand` and before `requireSessionLocked` in `ReplaceStrand`.
  Formation (`strandNameLocked`) runs inside `addStrandLocked` under the op lock in both paths.
  `ReplaceStrand` removes the old strand before the add, so a replacement under the same name is not refused.
  Concurrent adds serialize on the op lock's file lock, so `-N` numbering and the held-name refusal see a consistent table.
- Card 7 deletions: no reference to `resolveStrandName`, `FormatStrandName`, `Config.StrandName`, `AddSpec.Round` or the `<ROLE>`/`<ROUND>`/`<SHORT_GUID>` tokens remains outside the deliberate `config_test.go` fixture and the `doc.go` retirement note.
  `TestLoadConfig_RetiredStrandNameKeyIsIgnored` loads a `reed.yaml` carrying `strand_name`.
- Card 8: every by-name lookup (`guidByName`, `findStatusStrand`, `removeStatusStrands`, `orchStrands`, batten's `driverAliveFrom`, loomcli's `resolveDriverStrandAction`) goes through `agentname.Matches`.
  Both driver lookups go through `loomengine.IsDriverStrand`, which also accepts the `loom-driver` literal.
  A role query never matches a `-N` sibling (`worker` does not match `…:worker-2`), so role matching cannot pick the wrong numbered strand.
  Two strands share a role segment only when a legacy literal sits beside a new full name, and that case is refused as ambiguous.
- Cards 12 and 13: `WatchNames` is started only in the daemon's `enterSession`.
  A nil namer or an empty `SessionID` skips the session check, and nothing is typed unless `SessionIdle` passes on a fresh capture taken under the op lock.
  Under that lock, shuttle's `SendText` cannot interleave with the rename.
- Cards 11 and 13: both launch and resume lines carry `--name` plus `sh.EnvRef(…)`, giving `"${LYX_STRAND_NAME}"` on POSIX and `"$env:LYX_STRAND_NAME"` on pwsh.
  A transcript line that does not parse is skipped, and a missing or unreadable transcript reports no drift.
  A transcript without a `custom-title` line reports drift, which the card requires and the file doc records as a risk.
- Cards 9 and 10: every launch, including replace and resume, funnels through `launchStrandLocked`.
  That function sets `allow-set-title off` and the title, then composes the name and parent exports.
  `4ddf3665e` strips both keys from the server env so an unparented pane cannot inherit a stale parent.
- Cards 2 and 3: `freshBind` is `!Found && !AnchorFound`, with a stale marker counting as bound.
  A bound weft without a code returns a warning, not a refusal.
  `--code` is validated before use, and a malformed record reads as absent with a named warning.
- Card 14: an uncoded hub lists normally, since `Directory` needs no code.
  A worktree with no state file yields no rows, and a corrupt state file is skipped with a named warning.
- Cards 5 and 15: `WriteSeed` compares recipe, driver and params only, so `Parent` is write-once across batten run, Seed-Child and loom start.
  With `LYX_PARENT` unset, the skill and the prompt fall back to the stop report alone.
- Per-card mechanical checks: the plan has no Delete or Rename group.
  The assert-no-callers check over card 7's in-Edit deletions holds (see above).
