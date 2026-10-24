---
verdict: APPROVED
findings:
  - id: F1
    origin: lens:affected-set
    severity: MEDIUM
    class: consistency
    location: _lyx/plan/23-batten-guard.md
    summary: Card 23 rewords the batten prime-only refusal, but two integration steps that assert the old text are not among its targets, so its own gate fails on them.
    fix: Add `internal/battencli#stepNonPrimeRefusal` and `internal/battencli#stepRecordsPrimeRefusal` under card 23's Edit list, and say in the Intent that they assert the new text. For the task-pair step that text names the prime. For the records-sibling step it is the drivable-worktree refusal, wrapped in the new guard's wording instead of "prime worktree only".
  - id: F2
    origin: lens:affected-set
    severity: MEDIUM
    class: consistency
    location: _lyx/plan/19-orch-in-beam.md
    summary: Card 19 rewords orch's refuseNonPrime, but the integration helper refusalCase still asserts "prime worktree only" and is not on the card.
    fix: Add `internal/orchcli#refusalCase` under card 19's Edit list. Say in the Intent that it asserts the new refusal, which names `lyx beam create` and `lyx beam promote`, and no longer asserts the prime-only wording.
  - id: F3
    origin: lens:guard-reach
    severity: MEDIUM
    class: design
    location: _lyx/plan/35-beam-promote.md
    summary: The start guard's Promoted check does not reach a promote stopped after step 2, because the run is still claimed and its status file still exists, so a resume restarts the ended run in a marked pair.
    fix: In card 35's Intent, state that the start guard's Promoted check runs whether or not the run's status file is found, unlike the left-pair check, which runs only with no status file. Add a TestStartGuard_Refuse row with found true and a marked pair, refused naming `lyx beam promote <slug>`.
  - id: F4
    origin: lens:guard-reach
    severity: MEDIUM
    class: consistency
    location: _lyx/plan/09-board-project-kind.md
    summary: The upsert test rows meant to reach normalizeTask's refusal of order or beam on a task or note never reach it. The upsert key allowlist refuses those keys first at card 9, and from card 11 checkBeamPayload refuses beam first.
    fix: In card 9's Intent, replace the TestKindDependencyRules rows for order and beam on a task or note with TestFabricReplay rows that replay a board.json entry carrying them, refused by normalizeTask naming the field. Note that an upsert never reaches this branch.
  - id: F5
    origin: handler
    severity: MEDIUM
    class: consistency
    location: _lyx/plan/23-batten-guard.md
    summary: Card 23 lists internal/battencli/cli.go under Extends, but it must rewrite text there. The batten Command's Long says every verb runs from the prime only, and cleanupCmd's doc and help say prime-only.
    fix: In card 23, remove the Extends entry `internal/battencli/cli.go`. Add `internal/battencli#Command` and `internal/battencli#battenCLI.cleanupCmd` under Edit. Say in the Intent that the Command Long's "Every verb runs from the hub's prime worktree only" and cleanupCmd's "prime-only" are rewritten to say the verbs run from the prime and from any beam.
  - id: F6
    origin: handler
    severity: MEDIUM
    class: design
    location: _lyx/plan/24-batten-window-site.md
    summary: Card 24 adds a routing edge with no stated bound. It brings up a reed session in a beam's worktree on demand, but its Intent never says what that session lets through or what ends it.
    fix: Add a Bound sentence to card 24's Intent. It says the session brought up holds no orch strand, so OrchLive and HubOrchs read no orch there and no notice, tender or hibernate read changes. It also names what ends the session, the beam's pair teardown, or the beam's orch reusing it.
  - id: F7
    origin: lens:way-forward
    severity: MEDIUM
    class: consistency
    location: _lyx/plan/08-weft-reset-executor.md
    summary: The missing-origin-record refusals of MarkBeam (card 2) and ResetRecordsToParent (card 8) name lyx fabric reconcile, which recreates a missing records worktree but never restores a record file deleted from a present one, the very input card 8's test plants.
    fix: In cards 2 and 8, tell the two causes apart. A missing records worktree names `lyx fabric reconcile`. A record file absent from a present records worktree names restoring it from the records branch in that worktree with `git checkout HEAD -- <record path>`, then re-running the verb. The card 8 test row asserts the second text.
  - id: F8
    origin: lens:way-forward
    severity: MEDIUM
    class: consistency
    location: _lyx/plan/05-add-adopt-local.md
    summary: The adopt-local refusal of a records branch checked out in another records worktree names that worktree but no way forward, against the record's constraint that every new refusal names one.
    fix: In card 5's Intent, give that refusal a way forward that releases the records branch, such as removing the stray records worktree, and have TestAdd_AdoptLocal's row assert it.
  - id: F9
    origin: handler
    severity: LOW
    class: consistency
    location: _lyx/plan/24-batten-window-site.md
    summary: Card 24 moves the --window site to the run's parent worktree's session, but the run verb's help text in battenVerbTexts and the --window flag help in Command still say the prime's reed session.
    fix: Add `internal/battencli#battenVerbTexts` and `internal/battencli#Command` under card 24's Edit list. Say in the Intent that both texts now name the run's parent worktree's reed session, the prime's or a beam's.
  - id: F10
    origin: lens:way-forward
    severity: LOW
    class: consistency
    location: _lyx/plan/35-beam-promote.md
    summary: A promote of a root pair, made by adopt-local over the default branch, refuses only at step 4, after the run has been ended, marked and unclaimed. Every re-run refuses again there, and the start guard sends the operator back to promote.
    fix: In card 35, add a refusal before step 1 for a pair whose origin record has an empty ParentBranch. It names the pair as a root pair that cannot become a beam. TestPromote reaches it over a fake origin with an empty parent.
  - id: F11
    origin: lens:way-forward
    severity: LOW
    class: consistency
    location: _lyx/plan/04-add-from-parent.md
    summary: The refusal of a detached or unborn parent names no way forward, and the refusal of the prime's own name points at lyx fabric pairs, when the step that works is dropping --parent.
    fix: In card 4's Intent, have the detached or unborn parent refusal name `lyx fabric checkout <parent branch>` in the parent's worktree. Have the prime-name refusal name dropping `--parent` to fork from the prime.
  - id: F12
    origin: lens:way-forward
    severity: LOW
    class: consistency
    location: _lyx/plan/13-board-slug-seam.md
    summary: The slug rule's suggested admitted slug is checked neither against the rule nor against the board, so it can fail again, or name an existing entry that an upsert would then edit.
    fix: In card 13's Intent, have checkNewSlugs run the suggested slug through the validate seam and the board's slugs before it offers it, and offer none when the suggestion fails. TestSlugRule gains a row whose suggestion collides with an existing entry.
  - id: F13
    origin: lens:way-forward
    severity: LOW
    class: consistency
    location: _lyx/plan/11-board-beam-flag.md
    summary: A payload setting beam false on a beam gets the same refusal naming lyx beam create or lyx beam promote, but neither verb clears the flag.
    fix: In card 11's Intent, word the refusal of a false value on a beam to say the flag stays set until the beam lands with `lyx beam land <slug>`. TestUpsertFieldAllowlist gains that row.
  - id: F14
    origin: lens:guard-reach
    severity: LOW
    class: consistency
    location: _lyx/plan/23-batten-guard.md
    summary: Card 23 says only a beam at its own landing can reach the self-pair refusal, but at its landing the beam's run-held refusal fires first. The refusal is reached before a landing claims.
    fix: In card 23's Intent, replace "only a beam can reach this, at its own landing" with a statement that a beam reaches it before its landing claims, for example `lyx batten run <beam>` from inside the beam while its entry is open, the state the TestArmAt_StartGuardRefusesBeforeAnyWrite row sets up.
  - id: F15
    origin: handler
    severity: LOW
    class: decision
    location: _lyx/plan/34-beam-create.md
    summary: The record's constraint wires lyx beam through internal/cliwire, but card 34 never says how beamcli meets or departs from it.
    fix: Add one sentence to card 34's Intent. beamcli is a hub-only CLI like adoptcli and battencli, so PATTERN-cliwire-sole-wiring's standalone wiring does not apply. It resolves its location through battencli.ResolveBattenPrime and declares no cliwire.Module.
---

### [MEDIUM] Card 23 misses two tests that assert the reworded batten refusal

**Location:** `_lyx/plan/23-batten-guard.md`

**Issue:** `ResolveBattenPrime` replaces `RefuseUnlessPrime`'s text, which wraps the drivable-worktree refusal in "this verb runs from the hub's prime worktree only" (`internal/battencli/refusal.go:43`).
`stepRecordsPrimeRefusal` asserts "prime worktree only" (`internal/battencli/lifecycle_integration_test.go:1509-1525`), and `stepNonPrimeRefusal` asserts the non-prime refusal (`:1480-1505`).
Neither is on card 23, so the card's own gate fails on the stale assertions, against the writer stencil's rule that a rewording card carries every test asserting the old text.

### [MEDIUM] Card 19 misses orch's refusal integration helper

**Location:** `_lyx/plan/19-orch-in-beam.md`

**Issue:** `refusalCase` asserts "prime worktree only" (`internal/orchcli/refusal_integration_test.go:118-128`), and card 19 rewords `refuseNonPrime` to admit beams and name the beam verbs.

### [MEDIUM] Promoted start-guard check misses a promote stopped after step 2

**Location:** `_lyx/plan/35-beam-promote.md`

**Issue:** `startGuard.refuse` checks pair state only when no status file is found (`internal/battencli/startguard.go:78`).
After promote step 2 the claim and status file still stand, so `lyx batten run <slug>` resumes the ended run inside a marked pair.
D10 requires the guard to refuse a task whose pair carries the mark, and the card claims the check covers a stop after step 2.

### [MEDIUM] Card 9's order and beam refusals have no reaching test

**Location:** `_lyx/plan/09-board-project-kind.md`

**Issue:** Card 9 adds no upsert key, so the upsert allowlist refuses `order` and `beam` before `normalizeTask` runs, and from card 11 `checkBeamPayload` refuses `beam` first.
The planned `TestKindDependencyRules` rows therefore pass on another refusal's text.
`normalizeTask` also runs on replay, which is the path that reaches the branch.

### [MEDIUM] Card 23 extends a file whose text it rewrites

**Location:** `_lyx/plan/23-batten-guard.md`

**Issue:** Card 23 lists `internal/battencli/cli.go` under `**Extends:**` "as batten's command help adds".
The help it changes is `Command`'s Long, "Every verb runs from the hub's prime worktree only" (`internal/battencli/cli.go:299`), and `cleanupCmd`'s doc and help say "prime-only" (`internal/battencli/cleanup.go:403`).
Rewriting existing content belongs under `**Edit:**`, and a Go file with symbols to name belongs there as member glyphs, per the writer stencil.
The combined fix overlay of F1, F2, F5 and F9 passes `lyx loom validate-plan --overlay` with no new blocking finding.

### [MEDIUM] Card 24 brings up a beam reed session with no stated bound

**Location:** `_lyx/plan/24-batten-window-site.md`

**Issue:** For a beam site, the window opener calls the reed engine's `Up`, which creates a session in a beam that may have no orch.
This new routing edge reaches readers that look at a worktree's session and strands: orch liveness, tender's reads and teardown.
The Intent states neither what the edge lets through nor what bounds it, which the rubric's attack-surface rule requires.

### [MEDIUM] The missing-origin-record refusals name a step that cannot restore a deleted record

**Location:** `_lyx/plan/08-weft-reset-executor.md`, `_lyx/plan/02-origin-beam-mark.md`

**Issue:** `lyx fabric reconcile` recreates a missing weft worktree, re-points junctions and adopts raw worktrees (`internal/fabricengine/reconcile.go:158-163`, `internal/fabriccli/fabric.go:270-273`).
It never rewrites a record file deleted from a present records worktree, which is the input card 8's test plants.

### [MEDIUM] The adopt-local records-branch refusal names no way forward

**Location:** `_lyx/plan/05-add-adopt-local.md`

**Issue:** The refusal names the other records worktree and stops there.
The record's constraints require every new refusal to name its way forward, and the sibling warp refusal's `lyx fabric checkout` acts on the warp side only.

### [LOW] The run verb's --window help still names the prime's session

**Location:** `_lyx/plan/24-batten-window-site.md`

**Issue:** `battenVerbTexts` says "its own tmux window of the prime's reed session" (`internal/battencli/cli.go:172`), and the `--window` flag help says the same (`internal/battencli/cli.go:337`).
Card 24 changes that behaviour but lists neither member.

### [LOW] Promote of a root pair dead-ends after step 3

**Location:** `_lyx/plan/35-beam-promote.md`

**Issue:** Card 8 refuses a pair with an empty `ParentBranch` at the reset, which promote runs as step 4, after it has ended the run, written the mark and cleared the claim.
A re-run refuses again at step 4, and the start guard's `Promoted` refusal names promote, so no named step progresses.

### [LOW] Card 4's parent refusals name weak or no ways forward

**Location:** `_lyx/plan/04-add-from-parent.md`

**Issue:** The detached or unborn parent bullet names no way forward, and the refusal of the prime's own name sends the operator to list pairs instead of dropping `--parent`.

### [LOW] The slug rule's suggested slug is unchecked

**Location:** `_lyx/plan/13-board-slug-seam.md`

**Issue:** The suggestion, the slug with its suffix dropped or with `-task` appended, may itself fail the rule.
It may also name an existing entry, which an upsert under it would edit instead of create.

### [LOW] The refusal of beam false names verbs that cannot clear the flag

**Location:** `_lyx/plan/11-board-beam-flag.md`

**Issue:** D3 says nothing sets the flag back.
A payload clearing it on a beam is still refused naming `lyx beam create` or `lyx beam promote`, and neither verb clears it.

### [LOW] Card 23 misstates which state reaches the self-pair refusal

**Location:** `_lyx/plan/23-batten-guard.md`

**Issue:** During its landing the beam's entry is run-held, so `ResolveBattenPrime`'s run-held refusal fires before the self-pair one.
The planned test row reaches the self-pair refusal before a claim, so the test is right and only the Intent's sentence is wrong.

### [LOW] Card 34 is silent on the record's cliwire constraint

**Location:** `_lyx/plan/34-beam-create.md`

**Issue:** The record's constraints say `lyx beam` is wired through `internal/cliwire`.
`PATTERN-cliwire-sole-wiring` binds the standalone-capable CLIs, and hub-only CLIs such as `adoptcli` and `battencli` do not import `cliwire`.
Card 34 neither carries the constraint nor records why it does not apply.

## Rejected

- way-forward finding 7, which says card 35's no-local-pair refusal names only `lyx adopt`: the card words it exactly as D10 decides, and the answer key is never the subject of a finding.
- Card order across the board, hubgeom and fabric cards: every symbol a card uses is created on the same or an earlier card. `boardengine`'s `Config.ParentOf` seam already exists (`internal/boardengine/config.go:47`), and `internal/hubgeom` already depends on `boardengine`, with no import cycle.
- Fidelity: D1 through D16, the PATTERN changes and the docs constraints each have a carrying card. Cards 29 and 30 are licensed by D11's never-rerouting Finalize and its describe producer. Card 27's widened `RecordsPresent` is what makes D16's named cancel progress.
- Board and task rules: no card changes this task's own board entry or files or promotes a task. No card has a `**Verify:**` line.
