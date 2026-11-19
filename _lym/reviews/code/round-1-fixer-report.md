---
dispositions:
  - id: F1
    disposition: fixed
  - id: F2
    disposition: fixed
  - id: F3
    disposition: fixed
  - id: F4
    disposition: fixed
  - id: F5
    disposition: fixed
  - id: F6
    disposition: fixed
  - id: F7
    disposition: fixed
  - id: F8
    disposition: fixed
  - id: F9
    disposition: disputed
    ground: every fix worse
    rebuttal: "Restoring the reworded lines puts the lym gate test Long text at 2117 bytes, above the 2000-byte cap that TestCLIForm_EveryCommand enforces. The rewordings are what keep the page under the cap beside the added --copies line, and trimming 117 bytes elsewhere would reword other lines unasked."
    evidence:
      - cmd/lym/clitree_test.go:413
  - id: F10
    disposition: fixed
  - id: F11
    disposition: fixed-differently
    reason: "The proposed assertion on exe.green after a copies call cannot hold: the call setup sets exe.green, and runWithSlots never touches it. I extracted the rule as greenForCall in internal/gatecli/copies.go, unit-tested it, and added a tier-1 scenario in internal/gatecli/test_integration_test.go that runs a --copies call and expects no green record. The forwarding row, the merged-launch test and the single-prebuild test were added as proposed."
  - id: F12
    disposition: fixed
  - id: F13
    disposition: fixed-differently
    reason: "selfexec.LymBinary refuses the running test binary, so no stub lym can reach clearWorktreeSandbox. I added a lymBinary seam to Teardown, set to selfexec.LymBinary in New, and drive the real spawn over the system true, false and ls binaries: success, no root, and a failing exit whose output reaches the error. The hardencli test is as proposed, except the orphan names the root on its command line, since its working directory resolves through the worktree's _lym link."
  - id: F14
    disposition: disputed
    ground: every fix worse
    rebuttal: "Restoring the two wordings puts the lym board upsert Long text at 2007 bytes and the lym config Long text at 2003, both above the 2000-byte cap. The shorter wordings are what keep them under it."
    evidence:
      - cmd/lym/clitree_test.go:413
  - id: F15
    disposition: fixed-differently
    reason: "The proposed fix also adds the Harden-Burler-Fix live gate to the Verify sites sentence. Only the verify gate runs through Verify, as PATTERN-gate-slot.md line 104 states; the live gate is NewLiveGate in internal/hardenshed/live.go, which spawns lym gate test. I added Harden-Burler-Fix to the sentence and left the live gate out."
---

## What changed

Each finding has its own commit or commits, named `<target>: fix <id> — …`.

- F1: `ArchiveDeps.WorktreeRoot` is now `AnchorRoot`, and the archive and the final ledger land under it.
  `Campaign` gained `AnchorPath`, set from `location.AnchorPath()` in `hardenCampaign` and in the envkit and shedrecipe test campaigns.
  `TestArchiveRound` and the row fixture use an anchor root of their own, and `TestCampaignArchivesUnderTheAnchorPath` shows `Campaign.ArchiveFinal` writing under `AnchorPath` and nowhere in the worktree.
- F2: `TestHardenRoundKey` gained a Bouncer auto-accept pair (no live report accepts; a red `harden-live-2.json` escalates cause budget) and a halt row (the campaign's reap runs at `ReapHalt`).
  `TestSettingLookup` gained a row reading `hardener.auto_accept` and `hardener.reviewer_fan` from the board's `hardener.yaml`.
  `TestHardenerRecipe` gained a subtest pinning each ring row's `*_setting` reference to a `hardener.` key.
- F3: the fix-row test reads the archived `harden-strays-1.json` and expects the start entry and the `reap-end` entry.
- F4: a red mark whose output names no failing test defers one line naming the `harden-live` run and its report path.
  Two `TestLiveGate` rows changed to expect it, and one row was added.
- F5: `CheckCampaign` refuses an empty reviewer or fixer list before the repeat check, with rows for both.
- F6: a fix row records the head it first started at in `harden-head-<round>.json` and reuses it on resume.
  `TestWrapRow` gained a resume row, and the package doc names the file.
- F7: `runWithSlots` leaves vet-step and guard-step runs out of a copies call, with a `TestRunTierCopies` row.
- F8: `hardener` joins the recipe lists in `docs/overview.md` (the `Run-Shed` sentence and the `shedcli` arming-table line) and `contracts/orchrole/orch-role.md`.
- F9: disputed. The restoration was committed and then reverted, because it broke the `Long` byte cap in `TestCLIForm_EveryCommand`.
- F10: the five `//testtiming:keep` marks carry the reasons the review gave.
- F11: described under the disposition above.
- F12: `TestHardenLiveKey` covers the `covers` read and an absent gate config.
  `TestHardenSegmentSettings_RefusesAnUnresolvableModelSpec` covers a bad reviewer and a bad fixer entry.
  `TestWire_HardenerRecipe` checks the fixer rotation for rounds 1 to 9 and the `judge` and `seed` overrides.
  A bad `judge` or `seed` spec has no row, because resolving it needs a registry and a loom config the unit test does not build.
- F13: described under the disposition above.
- F14: disputed, for the same cap. The restoration was committed and then reverted.
- F15: described under the disposition above.

A gate then flagged three comments in the new tests (a fixed-column wrap and two over-long lines).
They are rewritten one sentence per line in `internal/gatecli/copies_test.go`, `internal/pairteardown/sandboxclear_integration_test.go` and `internal/shedrecipe/entries_harden_test.go`, with no code change.
A second gate run still flagged the `continueSeamShuttle` comment as 120 characters wide, and it is split again.

`lym gate test` over the touched packages passes at tier 0 and, for `hardencli`, `pairteardown`, `loomcli` and `gatecli`, at tier 1.

## Self-check

Forks run: fix-code-correctness, fix-consistency, fix-proposal and fix-affected-set. Every fork returned a report.

- fix-code-correctness, LOW, fan settings unpinned: fixed. The recipe test pins each ring row's `reviewer_fan_setting`, `fixer_fan_setting` and `auto_accept_setting`.
- fix-code-correctness, LOW, `greenForCall` call site unpinned: fixed. The tier-1 scenario runs a copies call and expects no green record.
- fix-consistency, LOW, package doc silent on the head file: fixed in `internal/hardenshed/doc.go`.
- fix-consistency, LOW, the `shedcli` arming-table line in `docs/overview.md` omits hardener: fixed. `internal/shedcli/table.go` arms the hardener recipe.
- fix-consistency, LOW, `docs/overview.md:1146` omits hardener: disputed, premise false. The sentence reads "among them" and is not a closed list (`docs/overview.md:1146`).
- fix-consistency, LOW, `contracts/orchrole/role-beams.md:73` omits hardener: disputed, premise false. The sentence names a beam task run's rows, and it omits arras too, so it is no closed recipe list (`contracts/orchrole/role-beams.md:73`).
- fix-proposal, LOW, a recorded head goes stale if the first call's commits are rewritten: disputed, every fix worse.
  Revalidating the head needs a new git ancestry seam through `RowDeps` (`internal/hardenshed/round.go:75-80`), `Campaign` and `hardenCampaign`, and every fixture, for a fix row that rewrites its own commits.
  The finding's own fix key names reuse of the recorded head.
- fix-affected-set: no findings.

## Disputed

Disputed review findings: none.
The fork disputes above are listed in the Self-check section.

## Deferred

Nothing deferred.

## Rulings

No parent ruling reached this round.
