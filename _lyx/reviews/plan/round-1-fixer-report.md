# Round 1 plan fixer report — umbrella-daemon

Every non-decision finding was validated against the code at HEAD; each premise held.
`lyx loom validate-plan` reports `ok: true` with informational findings only, before and after the self-check edits.

## Changes

- F1 (fixed as proposed): cards 33 (`SpawnPass`) and 28 (`StartFirstPass`) start `lyx batten run <slug>` in the prime's anchor, citing batten's pre-run, `OpenWindowIn` and orchcli's `cmd.Dir`; the hub root stays the dobby's own working directory.
  Card 37's `hubDobbyDeps` sentence, which said every seam runs "from the hub root and never from inside a worktree", now says the pass spawn alone starts in the prime's anchor.
- F2 (fixed as proposed, one addition): card 13 lists `battenshedAllowedImports` (gains `internal/state`), `wireForHub` and the three battencli tier1 steps, each with its rewrite; card 28 lists `registerSeededRun` (loses its `windowFlag` check) and the three tests that set the window fields; card 4 creates `dobbyengineAllowedImports` holding only its own imports; cards 35 (`boardengine`) and 36 (`shuttleengine`, `stencilstore`, `segmentcolor`) edit it.
  Addition: card 27 edits it too, for `internal/lock` (`Live`'s `TryAcquireWriteLock`), an import the review's card list missed; cards 32 to 34 bring in no import beyond the list, their seams being told.
- F3 (fixed as proposed): card 13's `Call` takes `begun` from the persisted episode state before `observeState` and ends at once with the state-change Stuck when the state read differs; `TestInnerRun_MemoryOutlivesTheProcess` gains the running-then-done row.
- F4 (fixed as proposed): card 25 states `CheckGotoTarget`'s bound and what a back goto would re-run, and wires a `GotoGuard` that refuses any target but the halted row, naming `lyx batten goto <slug> --to <row>`; the recipe description and `TestBattenDriverDeps` follow; the ImpactSummary names the guard.
- F5 (fixed as proposed, then widened by the self-check): card 4's `ReadState` and `UpdateState` read an undecodable record as absent with one Warn and rewrite it fresh, with a test row; card 31's `dobbyReading` reads it as no dobby, adding no Warn of its own; card 32 reads a pid-less record met on a tick mid-run as a start; card 36 counts the Land Done already there as seen on such a record and states the lost tidy as an accepted bound.
- F7 (fixed as proposed): card 23 writes the re-sign arrows for `arbiterVerb` and `loomCLI.arbiterDepsFor`.
- F8 (fixed as proposed): card 14 runs the owner check on a due probe whenever `ownedBy` stands, covered or not, then yields with the pause or owner reason; `TestInnerRun_OwnerCheck` gains the covered owned-here row.
- F11 (fixed differently after the self-check): card 37 states `dobbyCmd`'s bound and adds `TestDobbyCmd_RefusesAPathItCannotPlace`.
  The proposed refusal, "a path `DobbyGeometry` cannot place", was not kept, since card 27's `DobbyGeometry` places any worktree with a directory, a task pair's included; the refusal reads the beam mark from the pair's origin record as `DobbyStart` does, and the test covers a task pair's root.
- F12 (fixed as proposed, first branch): card 24 drops the landing-halt row and states that batten has no `LandingHalt`, since `landingshed.HandsBack` knows only Merge-In and Finalize; with the parent's pause-honoured entry, card 24 names that Land stop as D8's batten landing halt the acceptance criteria keep from the arbiter.
- F13 (fixed as proposed, the row moved by the self-check): card 26's `TestPassExitCode` wraps only the owned-elsewhere sources the pre-run reaches; the row asserting a row error keeps the driver envelope's code sits in the `driveRun` harness of `TestBattenRun_RefusesWhileAPauseCoversTheSlug`, since `passExitCode` itself maps any `OwnedElsewhereError` to 77.
- F14 (fixed as proposed): card 28's `ensureParentDobby` gains an `ownerElsewhere` parameter and a `parentOwner` seam, spawns nothing for a parent another machine owns and returns an error naming `lyx adopt <parent>` or `lyx adopt --prime`; `TestStartFirstPass` gains the row.
- F15 (fixed as proposed): card 7 lists `battenCLI.preDriveCheck` and `TestPreDriveCheck`, adds the release step to the left-pair and no-seed texts, and updates `TestStartGuard_Refuse`'s left-pair rows.
- F16 (fixed differently): the proposed `lyx tender pause <slug>` was applied, then replaced after the self-check, since adopt's `LivePair` is wired with `hubOnlyPaused` (`internal/adoptcli/deps.go:114-126`), which counts only a hub entry, so a slug pause leaves the pair live and the named call refused again.
  Card 39 names `--keep-local` for a live pair, which `takeOverSlug`'s step-1 arm passes and its own way forward offers for "a live driver"; the `TestAdopt` row asserts it.
- F17 (fixed differently after the self-check): the proposed `lyx loom resume` was not kept, since `resume.go`'s help says it never spawns an LLM driver and re-steps a go-seeded run; card 15's failed-revive text names `lyx loom start`, as the cap text on the same card does, and the test row asserts it.
- F18 (fixed as proposed): card 36 keeps a timed-out tidy's slugs for the next tidy, words the no-report line to name them, and adds the row.
- F19 (fixed as proposed): card 28 rewords `adoptcli/doc.go` and `beamcli/doc.go`; card 13 rewords `pattern/PATTERN-shed-driver.md`'s "Outside it"; cards 4 and 22 append `dobbyengine` and `shedarbiter` to `pattern/PATTERN-cwd-resolution.md`'s bound packages.
- F20 (fixed as proposed): card 36's `TidySpec` sets `Segment` to `segmentcolor.Coordinator` and `Skills` to `ly:board`, asserted in `TestTidyPrompt` beside `internal/shedarbiter#TestSpec`'s precedent.
- Parent's Added-after-Discussion entries of 2026-10-25, applied as the parent's instruction: 00-overview.md's records-stop Shared Decision cites its entry (cards 17 and 42 unchanged); card 24 refuses a pause-honoured Land stop with a `TestEligibility_BattenRule` row; card 34 reads the owner from the local Bolt with no pull; card 27 reports and skips a failed prime claim with a `TestEnsureLive` row.

## Self-check

- fix-claims, BLOCKING (card 39, slug pause leaves `LivePair` live): fixed with my own change, `--keep-local` (see F16).
- fix-proposal, BLOCKING (same card 39 finding, merged with the above): fixed with my own change.
- fix-proposal, MEDIUM (card 15, `lyx loom resume` brings no driver to a halted child): fixed with my own change, `lyx loom start`.
- fix-proposal, MEDIUM (card 37, `DobbyGeometry` places a task pair): fixed as proposed, the beam-mark read.
- fix-proposal, MEDIUM (card 4, a mid-run fresh record drops pid, counted landings and tidy): fixed as proposed in cards 4, 32 and 36, the lost tidy stated as an accepted bound ending at `TidySpec`'s timeout.
- fix-affected-set, MEDIUM (cards 13 and 16, envkit's seam inventory): fixed as proposed, `internal/testkit/envkit#nilLegal` gains `InnerRun.Sleeps` and `InnerRun.ReviveAPIError`, both nil-legal.
- fix-affected-set, LOW (shedrecipe embeds `InnerRunDeps`): no defect reported; no change.
- fix-consistency, MEDIUM (card 26, the row error row cannot be a `passExitCode` row): fixed as proposed, moved to the `driveRun` harness.
- fix-consistency, LOW (card 31, a double Warn): fixed as proposed, the Warn is `ReadState`'s.
- fix-consistency, LOW (card 25's ImpactSummary): fixed as proposed.

## Disputed

None.

## Deferred

None.
