---
dispositions:
  - id: F1
    disposition: fixed
  - id: F2
    disposition: fixed
  - id: F3
    disposition: fixed
  - id: F4
    disposition: ruling
    reason: "The parent's decision notice (Added after Discussion, parent, 2026-11-19, \"Hub gate keys are operator writes after landing\") settles F4; the Shared Decision hub-gate-keys-are-a-hub-write now cites it and names the three lym config gate --set writes for the change description's landing section."
  - id: F5
    disposition: fixed-differently
    reason: "The proposed signal, the round's fixer report or gate-findings file, does not hold: a fix row whose verify gate ran out archives the round's fixer report away (internal/shedadapters/burlerfix.go:128, called at :327), and the gate-findings path is a told row path, not a run-directory file. LiveRed instead keys on harden-commits-<round>.json, which card 9 writes on every nil-error fix-row call, the counted Stuck of an exhausted gate included. The proposed extra TestBouncer_AutoAccept row is not added: that test drives LiveRed through a fake, so the row would repeat card 10's existing red-LiveRed row; the new case is covered by a new TestCampaignLiveRed."
  - id: F6
    disposition: fixed
  - id: F7
    disposition: fixed
  - id: F8
    disposition: fixed
  - id: F9
    disposition: fixed
  - id: F10
    disposition: fixed
  - id: F11
    disposition: fixed
  - id: F12
    disposition: fixed
  - id: F13
    disposition: fixed
  - id: F14
    disposition: fixed
  - id: F15
    disposition: fixed
---

# Round 1 fixer report — plan

## Changes

- **F1** (card 8): the prescribed `PATTERN.md` line keeps "a pinned or spawned lym is named via `selfexec.LymBinary`" beside the new orphans clause; its rule text is exactly 280 characters, which `pattern-entry-line-cap` admits (`internal/pattern/check_test.go:89` refuses only `n > MaxEntryTextChars`).
- **F2** (cards 7 and 16): `SandboxDeps.Seed` and `Clone` name their parameters `codeBare` and `recordsBare`; card 7 states that `hardenshed`'s doc and code name the bares by role, and card 16 that `hardencli`'s code does too. `fabricengine.SeedLocalBares` keeps its signature.
- **F3** (card 16): `pattern/PATTERN-cli-cobra.md` joins the Edit list, and the Intent adds the Package naming deviation bullet. The self-check widened the bullet to every package card 16 has `hardencli` call, matching its sibling bullets. Not raised with the parent: the bullet is a deviation listing, not a rule change, as are the PATTERN table and list rows F7 and F14 add.
- **F5** (card 11), fixed differently, see the frontmatter: `LiveRed` reads an absent live report as red once the round's `harden-commits-<round>.json` exists, and `RoundFacts` renders such a report red too. A new tier0 `TestCampaignLiveRed` covers the red, green, absent-after-fix and absent-before-fix cases.
- **F6** (cards 6 and 14): `LiveRun` gains `Refused`. A refused `harden-live` call is returned as an error carrying its refusal text and writes no report; a refused copies call is recorded in the report as refused and leaves the mark alone. Card 14 adds `hardenLiveRun`, which reads the call's JSON envelope into `LiveRun`. New rows cover it in `TestLiveGate` and `TestHardenLiveArgs`.
- **F7** (card 2): `internal/gateslot/doc.go` moves to Edit and `pattern/PATTERN-test-speed.md` joins it. `harden-burler-fix` carries `code-burler-fix`'s reports and the two live keys carry none. The card rewrites the two `doc.go` sentences and the `reach` sentence, and adds the hardener rows to the doc's site table and to PATTERN-test-speed's key table.
- **F8** (cards 3 and 16): each card creates `TestMain` in its package's `testmain_test.go`. The self-check made `hardencli`'s call `gitkit.HermeticGitEnv()` before `tmuxkit.Main`, since its tier1 test spawns git. `hardenshed`'s calls `tmuxkit.Main` alone, its tests running git through fakes.
- **F9** (card 5): refusal 3 also refuses a copies key naming more than one tier when copies run, and `TestCheckCampaign` gains a row for it and a passing row with `llm_driving` true.
- **F10** (card 8): reap step 4 skips the caller's pid and its ancestors, read from the listing's parent pids, and `TestReap` gains the ancestor row.
- **F11** (card 19): `checkEfforts` joins the Edit list, and its unknown-role refusal names the same layer way forward as `skillLists`'. `TestLoad_RefusesUnusableContent` gains an efforts row. The premise's "names no way forward" is loose: the refusal already appends the generic "fix it with lym config agents from the prime" (`internal/agentconfig/config.go:77`). But that way forward is wrong for a local layer, so the finding holds.
- **F12** (card 19): `internal/mergeresolve#mergeresolveAllowedImports` joins the Edit list and the reworded comments.
- **F13** (card 8): `internal/runproc/doc.go` joins Extends, and the Intent names both `EndProcess` users.
- **F14** (card 3): `pattern/PATTERN-config-layers.md` joins Edit, adding `hardenshed` to the strict callers.
- **F15** (card 13): `internal/boardengine#normalizeTask`, `internal/boardcli#storeVerbs` and `internal/boardengine_test#TestNewTask` join Edit, and both texts name `hardener`. `TestNewTask`'s task-pawl row asserts the new list. `lym loom validate-plan` then asked for the two packages' `doc.go` files, which name no task recipe list, so they are listed under Uses.

`lym loom validate-plan` passes with informational findings only.

## Self-check

- fix-consistency, MEDIUM: a commits file written by the burler fix producer's unjudged hand-on would read an absent live report red. Disputed, premise false. The hand-on returns before any fixer runs only when the highest complete round, one holding its fixer report, is unjudged (`internal/shedadapters/burlerfix.go:170-176`). A round whose gate ran out has its fixer report archived (`burlerfix.go:128`, `:327`), so it is not complete. A completed fix ran its `live` gate, which wrote the report or returned an error. So the hand-on never meets an absent report.
- fix-consistency, LOW: card 11's `RoundFacts` sentence and `TestCampaignRoundFacts` did not cover the absent-red case. Fixed: the case moved into `RoundFacts`' own bullet, and the test gained the row.
- fix-consistency, LOW: card 6's report fields and card 11's facts did not carry a refused copies run. Fixed: the report field and the facts' rendering were added, with a `TestCampaignRoundFacts` row.
- fix-claims: no findings.
- fix-proposal, MEDIUM: `hardencli`'s `TestMain` needs `gitkit.HermeticGitEnv()`. Fixed as proposed in card 16; card 3 states that `hardenshed`'s tests spawn no git.
- fix-proposal, LOW: F3's deviation bullet left out packages `hardencli` calls. Fixed as proposed.
- fix-affected-set, LOW: `pattern/PATTERN-gate-slot.md:104-105` and `internal/loomshed/doc.go:113-114` name each fix row's own verify key without `harden-burler-fix`. Fixed: PATTERN-gate-slot joins card 11's Edit list, and `loomshed/doc.go` moves from Extends to Edit in card 13, each naming the new key.

## Disputed

None.

## Deferred

None.

## Rulings

- **F4**: the parent recorded, through the decision notice "Hub gate keys are operator writes after landing", that the three hub gate values are operator writes after landing. No card writes them, the change description's landing section names the three `lym config gate --set` commands, and the Shared Decision cites the entry. Changes: the overview's Shared Decision `hub-gate-keys-are-a-hub-write` cites the entry by title and lists the three exact writes for the landing section. It also states that until the operator sets `harden-burler-fix`, the `verify` gate runs tier 0 alone. Nothing left undone, and no flag sent to the parent.
