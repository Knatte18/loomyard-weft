# Support log — plan-gate-gaps

## Interview

Autonomous run: no operator answered; every pick below is an auto-pick recorded in the question ledger.

### Turn 1 — exploration

- Read the board entry (#484) and the related entries `glyph-use-audit`, `discussion-premise-cites`, `dotnet-scoped-verify` and `lighter-test-rules`; none plans work on webster's done-checks, batch boundaries or card-gate scope.
- Read tender's record-batch refusal note for batch 07 (`tender-weft/_lyx/loom/friction/webster-refusal-record-batch.md`): every finding is `DoneChecks`' create-not-done or rename-not-done-new on a `plan:internal/pausecli#…` or `plan:internal/pauseengine#…` handle.
- Read tender's card 12: a Rename group of file pairs moving every file of both packages into `internal/tendercli` and `internal/tenderengine`.
- `DoneChecks` resolves through quarry's `Resolve` against the working tree; quarry v0.2.1 has no revision-scoped resolve, only `DeltaGit` between revisions.
- `gateScopeOf` unions the cards' target directories, filtered by a `.go` check on disk at render time; `CardTargetDirs` takes a Rename group's new side only, so the stale directory in batch 07's gate came from cards 8–10's Create targets.
- `internal/impactset` already scans `//lyx:guard` markers and runs marked tests by name in the Webster-Burler-Fix round gate; both tests named in the board entry carry the marker.
- Measured: `lyx gate test ./cmd/lyx` took about 8 s wall; running every marked test across its packages with `--tags integration` and `-run` took about 7 s wall.
- The glyph scope guard's deviations are informational, so a fork editing a pinned test outside its card is not blocked.

### Turn 2 — design picks

- Gaps 1 and 2: picked a batch boundary before a card that renames or deletes an earlier same-batch card's target (D1), over rename-following through every batch-end consumer and over per-commit resolution.
- Gap 3: picked a guard step in every card gate (D2), with a degrade-to-line rule for a failed scan (D3) and a fork instruction for out-of-card fixes (D4).

### Turn 3 — lens review

Every fork returned a report.

- premises MEDIUM, guard scan's directories come from a `go list` spawn inside `Derive`: fixed, D2 walks the worktree instead and spawns nothing.
- pattern-fit MEDIUM, same `go list` source, spawn logging and its failure under D3: fixed with the walk; D3 covers a walk read error (merged with premises).
- premises LOW, error policy differs between impactset and webster: fixed, D3 states it is per caller.
- premises LOW, `DetectDrift` handles later cards naming old glyphs: rejected, behaviour is the same as under the identity batcher today and D1 does not change it.
- premises LOW, cost DP early break stays sound: no change; one-card feasibility added to D1.
- pattern-fit MEDIUM, `PATTERN-ref-shape-registry` not named for D1's classification: fixed in D1 and Constraints.
- pattern-fit MEDIUM, spec's per-card gate text missed in docs: fixed in Scope.
- pattern-fit LOW, rule should be one shared batcher helper: fixed in D1.
- pattern-fit LOW, rebaseline not covered: fixed, D1 applies at first init and rebaseline.
- pattern-fit LOW, fixture temp dirs: fixed in Constraints (`t.TempDir`).
- scenarios MEDIUM, guard step's directory list goes stale under a batch's rename: fixed, D2 drops a guard directory a card of the batch renames out of or deletes; acceptance row added.
- scenarios MEDIUM, unmappable refs and same-card comparison: fixed in D1 with acceptance rows.
- scenarios LOW, command length: fixed, over-long step renders D3's line.
- scenarios LOW, resume keeps partition: fixed, acceptance row added.
- scenarios LOW, slot pressure: fixed, Open risks entry.
- scenarios LOW, D4 untested: fixed, acceptance row added.
- goal-scope MEDIUM, guard step covers only marked tests: fixed, D2 bound states marked tests run and guard packages compile, unmarked pins stay late; Open risks entry.
- goal-scope LOW, D1 answers board item 1 by its third route: fixed, one clause in Scope.
- goal-scope LOW, directory-level old side undefined: fixed in D1.
- board-fit LOW, `weaver` reuses partitioning: rejected, the rule is profile-independent and needs no mention.
- board-fit LOW, guard step is Go-only against `dotnet-scoped-verify`: fixed, Open risks entry.
- board-fit LOW, `lighter-test-rules` wording pins: fixed, Constraints says pin the marker or presence.
- board-fit LOW, `glyph-use-audit` replays partition differently: rejected, it affects only reading that bench.
- leanness MEDIUM, Auto-mode assumptions restate decisions: fixed, cut to the three bare assumptions.
- leanness LOW, rejection reason in Scope Out: fixed, reason dropped.
- leanness LOW, `./cmd/lyx` 8 s figure in D2: fixed, cut.
- leanness LOW, incident narrative in Goal: fixed, one clause per gap.
- leanness LOW, filler Constraints (test budget, leaf packages): fixed, both cut.

## Rejected alternatives

- **Resolve each card's glyphs at that card's commit.** Quarry resolves only the working tree, so this needs a checkout per card or a quarry change in a separate module; it also leaves gap 2 open.
- **Follow the plan's renames.** Rewriting every card's done-check keys, handle bindings, drift and scope-guard inputs and gate directories through later cards' Rename pairs touches four consumers in two packages and needs a directory map derived from file pairs, which is ambiguous when a package splits; the batch split reaches the same result with one rule.
- **Plan-Review or `validate-plan` flags a rename after use.** It pushes the work onto the plan writer, who would reorder or split cards by hand to get what the batchifier can do mechanically.
- **Run `./cmd/lyx` in each batch gate.** Catches the break a batch late instead of at the card, and covers only `cmd/lyx` while guard tests live in other packages too.
- **A plan-time check for symbols pinned by name.** Needs per-test knowledge of what each scan pins; the guard step catches the same break generically for seconds per card.
- **Refuse begin-batch on a `GuardScanError`.** Stalls the batch on a test-file fault the plan-level verify already reaches.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 (resolved, auto-pick): Which fix for done-checks against a later rename? — Batch split (D1). Recommended because it closes gaps 1 and 2 together, in one package, and renames are rare.
- Q2 (resolved, auto-pick): Card gate or batch gate for repo-wide scans? — Card gate (D2), at about 7 s per card, so the break is attributed to its card.
- Q3 (resolved, auto-pick): Run all of `./cmd/lyx` or only marked guard tests? — Marked guard tests, reusing impactset's scan, since guards also live outside `cmd/lyx`.
- Q4 (resolved, auto-pick): Nested-module guard tests? — Root module only (A3).
- Q5 (resolved, auto-pick): Guard-scan failure behaviour? — Render a prompt line, no refusal (D3).
- Q6 (resolved, auto-pick): Should a by-hand batch accept verb be built? — No; the board's scope list does not ask for it and D1 removes the case that needed it.
- Q7 (resolved, auto-pick, lens review): How does webster list the root module's package directories for the guard scan? — A directory walk skipping nested modules, so no `go list` spawns at every begin-batch.
