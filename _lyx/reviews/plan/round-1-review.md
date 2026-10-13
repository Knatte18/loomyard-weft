---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/plan/07-focus-rules.md:52
    summary: The one test named for D5's failed-client-listing guard cannot reach it, because the ListClientsError row's fixture has no agent or overview window, so no select-window is appended with or without the guard
    fix: In card 7's Intent, rewrite the sentence on TestAttachArgv_MultiClientWarning so its ListClientsError row seeds the state with a live own-window strand (or an overview strand) and scripts list-windows and the session pane listing (answerWindowPanes) the way TestAttachArgv_SelectsTheRelevantWindow does, so relevantWindow would return that window, and then asserts that no select-window is appended. Also state that without the failure the same fixture appends the selection, so a missing guard fails the row. Keep the row's existing warning assertion.
  - id: F2
    severity: MEDIUM
    class: consistency
    location: _lyx/plan/05-window-order-and-shell.md:58
    summary: Card 5 makes Go placement and the shell check read @lyx_hidden through sessionWindowFormat, but internal/reedengine/doc.go still says no Go decision reads @lyx_hidden, and the card does not retire that sentence
    fix: In card 5's Intent, extend the internal/reedengine/doc.go sentence to say that the OpenWindow paragraph's line '@lyx_hidden is display only ... no Go decision reads it' is rewritten. The new wording says the status bar skips a hidden window, and window placement and the shell check read the flag to find the last non-hidden window. No focus rule reads it, which keeps card 7's wording true.
  - id: F3
    severity: MEDIUM
    class: design
    location: _lyx/plan/05-window-order-and-shell.md:35
    summary: newWindowLocked composes new-window -d plus the placement's -a/-b -t plus the caller's own arguments, and both callers' own arguments carry -t on the exact session target today, so a placed window gets two -t flags and tmux keeps the last one, the session target, which drops the placement
    fix: In card 5's Intent, state that a caller passes its arguments to newWindowLocked without its own -t. newWindowLocked adds -t on the exact session target (exactSessionWindowTarget) only when the placement is empty, the plain form and the psmux retry, so every new-window carries exactly one -t. Also state that TestOpenWindow's new row asserts exactly one -t in the argv.
  - id: F4
    severity: MEDIUM
    class: design
    location: _lyx/plan/06-overview-anchor.md:40
    summary: Card 6 introduces the overview anchor with no stated bound, and one path escapes D1's bound that the anchor places a single strand only in the first window. createOwnWindowPaneLocked sends a display with a window group through planOwnWindowPlacement before openOwnWindowLocked, so an overview display that carries Window would join or open a group window and skip overviewWindowInsertArgs
    fix: In card 6's Intent, have validateDisplay refuse an overview display that carries a window group (Display.Window non-empty or WindowSize non-zero). The refusal names the way forward, drop the group or use own-window. Add a Bound sentence, - Bound - the overview anchor places one strand per session, alone in a window of its own that opens at the lowest index, and it never joins a group. Extend the TestAddStrandLocked subtest that calls validateDisplay directly with an overview display carrying Window, which wants the refusal, since that is the input that reaches it.
  - id: F5
    severity: MEDIUM
    class: design
    location: _lyx/plan/07-focus-rules.md:50
    summary: The pure TestWindowOrder rows for newAgentWindowSelected repeat case for case the new-agent-window rows card 7 adds to TestWindowOrder_RealTmux (selected from the overview, from an all-dead agent window and for Focus; not selected from a live agent window or the shell), and the decision record's Constraints put focus at the tmux tier
    fix: In card 7, delete the sentence that gives TestWindowOrder rows for newAgentWindowSelected and drop the Edit bullet plan:internal/reedengine#TestWindowOrder, since the card then adds no row to it. TestWindowOrder_RealTmux's new-agent-window rows stay the single site. validate-plan --overlay with that bullet removed reports no new blocking finding.
  - id: F6
    severity: LOW
    class: design
    location: _lyx/plan/05-window-order-and-shell.md:60
    summary: TestWindowOrder's row 'the hidden placement targets the highest window' pins the same hiddenWindowInsertArgs choice that card 5's new TestOpenWindow row AHiddenWindowOpensAfterTheHighestWindow asserts by argv, which contradicts the card's own claim that TestWindowOrder holds the placement no engine row asserts
    fix: In card 5's Intent, drop the TestWindowOrder row 'the hidden placement targets the highest window even when it is hidden'. Script TestOpenWindow's AHiddenWindowOpensAfterTheHighestWindow row with a highest-index window that carries the hidden flag, so the engine row pins the hidden-included case.
  - id: F7
    severity: LOW
    class: consistency
    location: _lyx/plan/05-window-order-and-shell.md:30
    summary: Card 5 says the new members live in windoworder.go and later says newWindowLocked lives in spawn.go because of the new-window allowlist, so the two sentences disagree about one Create target
    fix: In card 5's Intent, change the sentence at line 30 to say the new members live in internal/reedengine/windoworder.go except newWindowLocked, which lives in spawn.go beside openOwnWindowLocked, since panebin_enforcement_test.go's windowCreationAllowlist admits the new-window literal only there and in window.go.
  - id: F8
    severity: NIT
    class: consistency
    location: _lyx/plan/06-overview-anchor.md:38
    summary: Card 6 says replacing another strand with an overview strand is never refused, but the one-per-session refusal does refuse it while a third strand holds the overview
    fix: In card 6's Intent, reword line 38 to say ReplaceStrand removes the old strand before its add, so replacing the overview strand itself is never refused. Replacing another strand with an overview strand is refused only while a third strand holds the overview.
---

### [MEDIUM] The failed-client-listing guard has no test that reaches it

**Location:** `_lyx/plan/07-focus-rules.md:52`

**Issue:** D5 and the acceptance criteria require that an attach whose client listing fails leaves the current window unchanged.
Card 7 names `TestAttachArgv_MultiClientWarning`'s "ListClientsError" row as the one test for it.
That row builds its engine with `newAttachTestEngine(t, goodAttachStrands())` (`internal/reedengine/attach_test.go:461`).
`goodAttachStrands` holds a single `below-parent` strand (`attach_test.go:78-82`), and the fixture scripts no `list-windows`.
`relevantWindow` therefore finds no agent window and no overview, and `attachWindowSelectionLocked` returns nothing whether or not the listing failure drops the selection.
The new "no `select-window` is appended" assertion passes against an implementation with no guard at all.

### [MEDIUM] The doc's claim that no Go decision reads `@lyx_hidden` goes stale

**Location:** `_lyx/plan/05-window-order-and-shell.md:58`

**Issue:** `internal/reedengine/doc.go:310` reads "@lyx_hidden is display only: the status bar skips a window carrying it, and no Go decision reads it."
Card 5's `sessionWindowFormat` lists `#{?#{@lyx_hidden},1,0}`.
`ownWindowInsertArgs` and `planShellWindowFix` both decide on "the highest-index non-hidden window", which D2 and D10 license.
Card 5 edits `doc.go` but names only the topics it adds.
It never retires that sentence, so the shipped doc would contradict the code.

### [MEDIUM] Placed `new-window` gets two `-t` flags

**Location:** `_lyx/plan/05-window-order-and-shell.md:35`

**Issue:** `openOwnWindowLocked` passes `-t exactSessionWindowTarget(e.SessionName())` today (`spawn.go:293-294`), and so does `OpenWindow` (`window.go:80-81`).
Card 5 has `newWindowLocked` build `new-window -d` plus the placement (`-a -t <id>` or `-b -t <id>`) plus "the caller's own arguments".
It never says the caller drops its session `-t`.
tmux's option parser keeps the last `-t`.
With the order the card gives, the session target overrides the placement window, and `-a`/`-b` then insert relative to the session's current window instead of the shell or highest window.
`TestWindowOrder_RealTmux` would catch this late, at Publish, so the card should say how the target is composed.

### [MEDIUM] The overview anchor states no bound, and a grouped overview display escapes the first-window placement

**Location:** `_lyx/plan/06-overview-anchor.md:40`

**Issue:** The rubric's attack-surface rule asks a card that adds a routing edge to state what it lets through and what bounds it.
D1 states "Bound: the anchor places a single strand per session, only in the first window."
Card 6 states the one-per-session refusal but no bound.
The placement it describes does not hold for every valid overview display.
`createOwnWindowPaneLocked` (`spawn.go:218-262`) takes the group branch whenever `s.Display.Window` is non-empty, before it ever reaches `openOwnWindowLocked`.
Neither `render.ValidateDisplay` (`render/policy.go:14-23`) nor card 6's `validateDisplay` rejects a window group on an overview display.
An overview display with a group would therefore be planned by `planOwnWindowPlacement` and could join an existing group window.
Card 6's `isOwnWindow` change routes overview strands into `createOwnWindowPaneLocked` (`spawn.go:107-108`).

### [MEDIUM] newAgentWindowSelected is pinned twice

**Location:** `_lyx/plan/07-focus-rules.md:50`

**Issue:** Card 7 gives `TestWindowOrder` pure rows for `newAgentWindowSelected`: selected from the overview, from an all-dead agent window and with `Focus`, and not selected from a live agent window or the shell.
The same card gives `TestWindowOrder_RealTmux` rows for the same cases at the public surface.
They cover a new agent window selected from the overview and from an all-dead agent window, not selected from a live agent window or the shell (where it carries `@lyx_unseen`), and a `Focus: true` strand selected from the shell.
`PATTERN-test-economy` and the decision record's Constraint ("window order and focus are tested at the tmux tier") make the tmux rows the site.
The pure rows test a private helper's matrix a second time.
The lens review applied the same reasoning to `relevantWindow` but not here.

### [LOW] Hidden-window placement is pinned by both a pure row and an engine row

**Location:** `_lyx/plan/05-window-order-and-shell.md:60`

**Issue:** Card 5 calls `TestWindowOrder` "the placement no engine row asserts by argv".
Yet `TestOpenWindow`'s new row "AHiddenWindowOpensAfterTheHighestWindow" (line 62) asserts `-a -t <highest id>` in the argv, the same `hiddenWindowInsertArgs` choice as the pure row.
Only the "even when it is hidden" case differs, and the engine row can carry it.

### [LOW] Card 5 names two homes for newWindowLocked

**Location:** `_lyx/plan/05-window-order-and-shell.md:30`

**Issue:** Line 30 places every new member in `windoworder.go`.
The placement paragraph then says `newWindowLocked lives in spawn.go`.
That is the right call, since `windowCreationAllowlist` in `panebin_enforcement_test.go:47-60` admits `"new-window"` only in `spawn.go`, `window.go` and `probe.go`.
The two sentences still disagree.

### [NIT] ReplaceStrand's never-refused claim is too broad

**Location:** `_lyx/plan/06-overview-anchor.md:38`

**Issue:** "replacing another strand with one, is never refused" is false when a third strand already holds the overview, which is the case the one-per-session refusal exists for.

## Rejected

- Focus 1, re-sign arrows on card 5. `createOwnWindowPaneLocked(st *ReedState, s *Strand)` already takes `st` (`spawn.go:218`).
  The card's "three calls pass `st` along" changes the calls to `openOwnWindowLocked` inside it, and that member carries the arrow.
  `markResolvedStrandWindowLocked(st *ReedState)` (`paneoptions.go:65`) and `OpenWindow(name string, lyxArgs []string)` (`window.go:52`) keep their signatures.
  No arrow is missing.
- Focus 2, cards 6 and 7. Card 7 changes neither `killStrandPanes`' nor `openOwnWindowLocked`'s signature.
  `selectOnEnd` and `endedCurrent` arrive with card 6's re-sign, and card 7 only changes what the `selectOnEnd` branch calls.
  Card order holds: `sessionWindow` is created in card 5, `windowPane` exists (`ownwindow.go:25`), and `selectOnEnd` exists from card 6.
- Focus 3, overlapping rows. Card 5's `TestEnsureShellWindow` rows for `AttachArgv(0, 0)` and a Selvage-only session pin the shell fix.
  Card 7's rows with the same inputs pin the selection, which is a different behavior, so they are not duplicates.
  The duplicates found are F5 and F6.
- Focus 4, reachability. Card 6's second-overview refusal is reached by its own subtest: `addStrandLocked` runs `validateDisplay` and then the name checks before launching (`strand.go:325-372`).
  A different name passes the name check, and a second overview under the same name is refused earlier, as the card says.
  Card 5's unlisted-shell branch has its `TestEnsureShellWindow` row.
  Card 7's failed current-window read in `openOwnWindowLocked` is a logged degrade on a tmux read error, not a refusal or input guard, so it needs no row.
  The failed client listing is F1.
- Focus 5, attack surface. Card 7 bounds the attach `select-window`, card 5 bounds the shell rename and `move-window`, and card 1 bounds the re-exec.
  Card 8 removes the `@lyx_strands` gate on Alt+Up/Down. Outside that window the current binding already swallows the key (`bindings.go:71-72`, an `if-shell` with no else branch), so removing the gate lets nothing new through.
  The overview anchor is F4.
- Focus 6, fidelity. Each of D1 to D10 and each Constraint has a carrying card:
  - D10's once-per-episode logging, with its session option, is in card 5.
  - D5's psmux caveat is in card 7.
  - D8's never-exec under `go test` and the Windows no-op are in card 1.
  - D6's hidden-window step is in card 8.
  - The leaf-package section and leaf test are in card 1.
  - The `statuspane/doc.go` import list is in card 3.
  - The docs are spread across cards 1, 3, 5 to 9.
  No card's scope goes beyond the record.
- Focus 7, verify. `go test ./...` and `go test -tags integration ./...` run every targeted package.
  The `tmux` tier is compiled by `go vet -tags tmux ./...`, which the rubric accepts.
  The section follows the template's order.
- Focus 8, granularity. `sessionWindowFormat`, `parseSessionWindows`, `planShellWindowFix` and `hiddenWindowInsertArgs` are separate `Create` declarations on the one card that owns the surface they serve, which is how the format lists new members.
  None is split onto a card of its own, so the bundling rule is met.
